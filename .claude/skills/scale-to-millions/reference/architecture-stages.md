# Deep-Dive Reference: Scaling Stages

## Stage 1 — Single Server (0–100 users)

Everything (app server, database, cache) runs on one machine. DNS resolves one IP.

**Failure mode:** Resource contention between app and DB; zero redundancy — one crash
equals total outage.

**Key actions:**
- Deploy via Docker Compose with app + PostgreSQL + Redis + NGINX
- Enable HTTPS with Let's Encrypt
- Add a `/health` endpoint for basic monitoring
- Use managed cloud VM (EC2 t3.small / Droplet / GCP e2-micro)

**Do NOT** premature-bond optimization at this stage. Get something running fast.

---

## Stage 2: Separate Database Tier (100–1,000 users)

Move the database to its own server so web and data tiers scale independently.

**Flow:** User → Web server → Dedicated DB server

**Actions:**
- Move DB to managed service (AWS RDS, PlanetScale, GCP Cloud SQL) or a dedicated VM
- Externalize (`DATABASE_URL`, `REDIS_URL`, etc.) — environments template
- Add PgBouncer for connection pooling
- Set up automated DB backups

**Choose your DB type:**
- **Relational (PostgreSQL/MySQL):** Default choice. Strong consistency, JOINs, ACID transactions, mature tooling.
- **Non-relational (NoSQL):** When you need horizontal scale from day one, flexible schema, very high write throughput, or simple key-lookups.

**Trade-off:** Decouples tiers but now the DB is the new SPOF.

---

## Stage 3: Load Balancer + Horizontal Scaling (1K–10K users)

Horizontal scaling: put 2+ identical web servers behind a load balancer.

**Prerequisites before adding more servers:**
1. Make the web tier **stateless** — move sessions to Redis (see Stage 7)
2. Move file uploads to object storage (S3, GCS)
3. Configure health checks so the LB removes dead instances

**Load balancer benefits:**
- Even traffic distribution
- Hides server IPs (security)
- Auto-failover if a node dies
- Can add/remove servers without DNS changes

**Topology:** LB in public subnet → web servers in private subnet

```typescript
// Express + Redis session (stateless sessions)
import session from 'express-session';
import RedisStore from 'connect-redis';
import { redis } from './redis-client';

app.use(session({
  store: new RedisStore({ client: redis }),
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
  cookie: { secure: true, httpOnly: true, maxAge: 86400000 }
}));
```

**Auto-scaling config (AWS ALB + ASG):**
```hcl
resource "aws_autoscaling_group" "app" {
  min_size         = 2
  max_size         = 10
  desired_capacity = 3
  target_group_arns = [aws_lb_target_group.app.arn]
  health_check_type = "ELB"
}

resource "aws_autoscaling_policy" "scale_cpu" {
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "TargetTrackingScaling"
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

**Trade-off:** Distributes compute load, but the **database is now the bottleneck**. A single DB behind 10 app servers just gets hammered.

---

## Stage 4: Database Replication (10K–100K users)

Master-primary handles writes; replicas serve reads. Most workloads are read-heavy.

**Pattern:**
- `INSERT/UPDATE/DELETE` → master
- `SELECT` → replicas
- If master dies → promote a replica (failover)

**Actions:**
- Set up read-write transit-split in your ORM (Sequelize, Prisma, SQLAlchemy, ActiveRecord)
- Deploy 1+ read replicas (AWS RDS Multi-AZ + read replica)
- Monitor replication lag (alert at >5 seconds)
- Write a failover runbook

**ORM read/write split (Node.js/Sequelize):**
```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(process.env.DATABASE_NAME, {
  replication: {
    read: [
      { host: process.env.DATABASE_READ1!, username, password, database },
      { host: process.env.DATABASE_READ2!, username, password, database },
    ],
    write: { host: process.env.DATABASE_WRITE!, username, password, database },
  },
  pool: { max: 20, idle: 30000 },
});
```

**ORM read/write split (Python/SQLAlchemy):**
```python
from sqlalchemy import create_engine

engine = create_engine(
    "mysql+pymysql://user:pass@write-host:3306/db",
    strategys='read_replica',
    strategies={
        'read_replica': {
            'read': create_engine("mysql+pymysql://user:pass@read-host:3306/db")
        }
    }
)
```

**Trade-off:** Replication lag means reads may see stale data. You now have eventual consistency between replicas. Master is still a write SPOF.

---

## Stage 5: Cache Layer (10K–100K users)

Add an in-memory cache after reading from DB. Most applications have far more reads than writes, and the DB is often the first thing that gets overloaded.

**Cache-aside (lazy loading) pattern:**
1. App checks cache.
2. Hit → return cached value.
3. Miss → read from DB, write into cache, return.

```typescript
// Generic cache-aside wrapper
export async function cached<T>(
  cache: Redis,
  key: string,
  fetcher: () => Promise<T>,
  ttlSeconds = 300
): Promise<T> {
  const hit = await cache.get(key);
  if (hit) return JSON.parse(hit) as T;

  const data = await fetcher();
  await cache.setex(key, ttlSeconds, JSON.stringify(data));
  return data;
}
```

**Py equivalents:**
```python
import redis, json

def get_or_set(cache: redis.Redis, key: str, fetcher, ttl=300):
    cached = cache.get(key)
    if cached:
        return json.loads(cached)
    data = fetcher()
    cache.setex(key, ttl, json.dumps(data))
    return data
```

**Guidelines:**
- Cache **read-heavy, write-light** data
- Set TTL: not too short (thrash back to DB) not too long (stale data)
- Eviction policy: LRU is the default
- Don't cache user-specific private data without key namespacing
- Use multiple Redis nodes or Redis Cluster — a single cache node is a new SPOF
- Over-provision memory to 70-80% target (not 100%)
- Add `cache_hit`, `cache_miss` metrics

**Trade-off:** reduces DB load but introduces a cache-invalidation problem. Cold-cache (empty on restart) can spike the DB.

---

## Stage 6: CDN (10K–100K users)

A CDN is a geographically distributed group of edge servers that cache static content close to bearers.

**What goes on CDN:**
- Images, videos
- CSS, JavaScript bundles
- Font files
- Static PDFs/documents

**Flow:** Client → nearest CDN edge → serve if fresh → if expired, pull from origin, cache with TTL.

**Best practices:**
- Use **content-hashed filenames** for cache-bust: `app.a3b2c1.js` (no need for purging)
- Set **`Cache-Control: public, max-age=31536000, immutable`** for hashed assets
- Set short TTLs for HTML or non-hashed assets
- Have a fallback: if CDN down, serve from origin
- Monitor CDN costs; move infrequent assets to cheaper storage

**CloudFront Terraform snippet:**
```hcl
resource "aws_cloudfront_distribution" "static" {
  origin {
    domain_name = aws_s3_bucket.static.bucket_regional_domain_name
    origin_id   = "S3-static"
  }
  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    target_origin_id       = "S3-static"
    viewer_protocol_policy = "redirect-to-https"
    cached_methods         = ["GET", "HEAD"]
    min_ttl = 0
    default_ttl = 86400
    max_ttl     = 31536000
    forwarded_values {
      query_string = true
      cookies { forward = "none" }
    }
  }
  restrictions {
    geo_restriction { restriction_type = "none" }
  }
}
```

**Trade-off:** Transfer costs, cache invalidation complexity, origin behind CDN can still bottleneck.

---

## Stage 7: Stateless Web Layer (1K–10K users, essential before horizontal scaling)

No request should get pinned to one server because of session state. Move all state (sessions, uploaded files, user data) to shared storage.

**Actions:**
- Sessions → Redis or database
- File uploads → S3 / GCS / Blob Storage
- Any server can handle any request → autoscaling now works

**Zustand / React hydration (frontend):**
Store session tokens client-side (JWT in `localStorage` or HTTP-only cookie). The back end has no server-local session on its own.

**Trade-off:** Every request now requires a Redis network call for session data. But you win horizontal autoscaling.

---

## Stage 8: Multi–Data Center (500K+ users)

**Actions:**
- Deploy to 2+ regions (e.g., us-east, eu-west, ap-southeast)
- GeoDNS routes users to nearest healthy region
- Cross-DC data sync: replicate data between regions
- Health-check regional, auto-failover if a region goes down
- Consistent deployment tooling across all DCs

**Trade-off:** Global access and disaster recovery, but you now have a **distributed data consistency problem**. Be explicit about "eventually consistent" for every piece of data.

---

## Stage 9: Message Queue (100K–500K users)

For any operation >200ms or that can fail independently — email, image resize, PDF generation, webhooks, analytics events, push notifications — **decouple from the request path**.

**Producer → Queue → Consumer architecture:**
- Producer (web tier) publishes a job
- Consumer (background worker) processes it
- Each can scale independently
- If a batch of workers is slow, jobs queue instead of being dropped

**Example: SQS / BullMQ pattern**
```typescript
import { Queue, Worker } from 'bullmq';

// Job input in your route handler
await emailQueue.add('welcome', { to: user.email, name: user.name }, {
  attempts: 5,
  backoff: { type: 'exponential', delay: 2000 },
});

// Background worker
new Worker('emails', async (job) => {
  await sendEmail(job.data.to, buildTemplate(job.data));
}, { concurrency: 10 });
```

**Trade-off:** Response is no longer synchronous. If the queue/workers crash, the work is delayed. Times need monitoring.

---

## Stage 10: Observability (Required at ALL stages)

Asynchronous > logs + metrics + automation:

- **Logging:** Centralized (CloudWatch, ELK, Datadog) — structured JSON logging
- **Metrics:** Host-level (CPU, memory, disk), application-level (RPS, latency, error rate), business-level (DAU, retention)
- **Monitoring dashboards:** Use Prometheus `/metrics` endpoint + Grafana
- **Alerting rules:** Error rate > 1% for 2m, latency P99 > 500ms, DB CPU > 80%, Redis memory > 90%
- **Automation:** CI/CD for build / test / deploy; add/remove instance, automated rollback
- **Distributed tracing:** OpenTelemetry for serviced request flow

---

## Stage 11: Database Sharding (1M+ users)

Vertical scaling of a DB has hard limits. Sharding splits one logical DB into multiple shards — the same schema, each chunk of rows, selected by a **shard/partition key**.

**Key considerations:**
- **Choose a shard key** to distribute load evenly → avoid "hot shard" (celebrity problem)
- **Resharding is expensive** — plan consistent hashing up front
- **Cross-shard JOINs are hard** — fix by denormalizing
- When one shard overflows or overheats → reshard or allocate dedicated shard for hot keys

```python
# Simple hash-based shard routing
NUM_SHARDS = 8

SHARD_HOSTS = {
    0: "shard0.db.internal",
    1: "shard1.db.internal",
    # ... up to 7
}

def get_shard(user_id: int) -> int:
    return user_id % NUM_SHARDS

def get_shard_host(user_id: int) -> str:
    return SHARD_HOSTS[get_shard(user_id)]
```

**Managed sharding:** Citus (PostgreSQL), Rebellend (Vitess, MySQL), CockroachDB, DynamoDB (auto).

**Trade-off:** Scale write and storage limits, but loses JOIN capabilities and reshards are expensive.

---

## Stage 12: Microservices (1M+)

Split the monolith by business domain — each service independently developed, deployed, and scaled:

- Auth Service
- User/Profile Service
- Payment Service
- Notification Service
- Search Service
- Media/Upload Service

**Pattern:**
- Each service has its own database (Database-per-Service)
- Inter-service communication: API gateways, gRPC, message queues
- Containerize each with Docker
- Kubernetes orchestration (EKS/GKE/AKS) + HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scale:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Trade-off:** Each service is simpler but now you manage distributed systems — network fallibility, partial failure, data consistency between services.

---

## Observability (Required at ALL stages)

```bash
1. Structured JSON logging (Winston / Pino / structlog / zap)
2. Prometheus metrics endpoint (/metrics)
3. Distributed tracing (OpenTelemetry SDK integration)
4. Grafana dashboard for: RPM, P50/P95/P99 latency, error rate, DB connections
5. Alerting rules: error rate >1%, latency P99 >500ms, DB CPU >80%
6. Uptime monitoring (healthcheck every 30s)
```

## Security Checklist (Required at ALL stages):

- [ ] Rate limiting (token bucket per IP + per user, 100 req/min default)
- [ ] CORS policy — whitelist only known origins
- [ ] Security headers (X-Frame-Options, CSP, HSTS)
- [ ] Input validation + parameterized queries only (SQL injection prevention)
- [ ] Secrets in environment variables; never in code
- [ ] Dependency vulnerability scan (npm audit / pip audit / bundler-audit)
- [ ] HTTPS everywhere — redirect HTTP to HTTPS