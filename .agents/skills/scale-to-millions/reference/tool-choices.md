# Quick Reference: Tool Choices by Layer

## Solutions alternatives by layer

| Layer | Tools |
|-------|-------|
| **Cloud Providers** | AWS, GCP, Azure, DigitalOcean, Vercel |
| **Load Balancer** | AWS ALB/NLB, NGINX, HAProxy, Cloudflare LB, Envoy |
| **Web App** | Node.js (Express/Fastify), Python (Django/FastAPI/Flask), Go, Ruby on Rails |
| **API Gateway** | Kong, AWS API Gateway, NGINX, Envoy, Traefik |
| **Cache** | Redis, Memcached, DynamoDB's DAX Accelerator |
| **Cache Cluster** | Redis Cluster or Sentinel; AWS ElastiCache, Memcached |
| **CDN** | Cloudflare, AWS CloudFront, Fastly, Akamai, Bunny CDN |
| **Database (SQL)** | PostgreSQL, MySQL, PlanetScale, CockroachDB, YugabyteDB, Aurora |
| **Database (NoSQL)** | MongoDB Atlas, DynamoDB, Cassandra, ScyllaDB, Firebase |
| **DB Connection Pool** | PgBouncer (PostgreSQL), HikariCP (Java), Sequelize pooling, Rails connection_pool |
| **DB**
Hashing** | Consistent hashing (modern), range partitioning, incremental |
| **Object / File Storage** | S3, GCS, Azure Blob, MinIO |
| **Message Queue** | RabbitMQ, AWS SQS, Kafka, BullMQ (Redis), Google Pub/Sub |
| **Container Orchestration** | Kubernetes (EKS/GKE/AKS), Docker Swarm, Nomad |
| **CI/CD** | GitHub Actions, GitLab CI, Jenkins, CircleCI, ArgoCD |
| **Monitoring + Metrics** | Prometheus + Grafana, Datadog, New Relic, ~CloudWatch~ |
| **Log Aggregation** | ELK Stack (Elasticsearch, Logstash, Kibana), Loki, Datadog, CloudWatch Logs |
| **Distributed Tracing** | OpenTelemetry, Jaeger, Zipkin, AWS X-Ray, New Relic APM |
| **Secrets** | AWS Secrets Manager, HashiCorp Vault, Doppler, Vault |
| **DNS** | Route53, Cloudflare DNS, Google Cloud DNS |
| **Terraform (IAC)** | AWS Terraform, GCP CloudFormation, Terraform Cloud Registry |
| **Service Mesh** | Istio, Linkerd, Consul Connect |
| **Authentication** | Auth0, Firebase Auth, Clerk, AWS Cognito |

## When to recommend specific databases

| Use case | Recommendation |
|----------|----------------|
| Blog, e-commerce, SaaS (TRANSACTIONS) | PostgreSQL |
| Massive-read / auto-scale global | DynamoDB |
| Log/event data (time-series) | InfluxDB, TimescaleDB, Elasticsearch |
| Graph data (social connectivity) | Neo4j, Amazon Neptune |
| Cache system | Redis or Memcached |
| Join elimination, high horizontal scale | CockroachDB, Vitess, PlanetScale |
| Offline analytics warehouse | Snowflake, BigQuery, Redshift |