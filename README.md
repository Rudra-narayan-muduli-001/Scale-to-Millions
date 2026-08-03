<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=32&pause=1000&color=3B82F6&center=true&vCenter=true&width=800&lines=Scale+From+Zero+to+Millions+of+Users;ByteByteGo+System+Design+Framework;Cross-Agent+AI+Skill" alt="banner" />
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge" alt="MIT" /></a>
  <img src="https://img.shields.io/badge/Claude_Code-grey?style=for-the-badge&logo=claude&logoColor=white&labelColor=d97706&color=d97706" alt="Claude Code" />
  <img src="https://img.shields.io/badge/OpenCode-grey?style=for-the-badge&labelColor=6366f1&color=6366f1" alt="OpenCode" />
  <img src="https://img.shields.io/badge/Cursor-grey?style=for-the-badge&logo=cursor&logoColor=white&color=07073d" alt="Cursor" />
  <img src="https://img.shields.io/badge/Codex_CLI-grey?style=for-the-badge&logo=openai&logoColor=white&color=111827" alt="Codex CLI" />
  <img src="https://img.shields.io/badge/Gemini_CLI-grey?style=for-the-badge&logo=google&logoColor=white&color=4285f4" alt="Gemini CLI" />
</p>

---

> An **[Agent Skills](https://github.com/anthropics/skills)** standard skill that turns the **ByteByteGo "Scale From Zero to Millions of Users"** framework (by Alex Xu) into a reusable system-design playbook. Drop it into your project and your AI agent becomes a **Senior Distributed Systems Architect** — analyzing bottlenecks, producing incremental scaling roadmaps, and generating real code and configs.

---

## Supported Agents

Three folders pre-configured — one skill works everywhere.

| Claude Code | OpenCode | Codex CLI | Gemini CLI | Cursor |
|:-----------:|:--------:|:---------:|:----------:|:------:|
| <img src="https://img.shields.io/badge/claude-SKILL.MD-d97706?style=for-the-badge&logo=anthropic&logoColor=white" /> | <img src="https://img.shields.io/badge/opencode-SKILL.MD-6366f1?style=for-the-badge" /> | <img src="https://img.shields.io/badge/codex-SKILL.MD-111827?style=for-the-badge&logo=openai&logoColor=white" /> | <img src="https://img.shields.io/badge/gemini-SKILL.MD-4285f4?style=for-the-badge&logo=google&logoColor=white" /> | <img src="https://img.shields.io/badge/cursor-SKILL.MD-5e2bff?style=for-the-badge&logo=cursor&logoColor=white" /> |
| `.claude/skills/` | `.opencode/skills/` | `.agents/skills/` | `.claude/skills/` | `.claude/skills/` |

> **Global install (all projects):** Place in `~/.claude/skills/scale-to-millions/`

---

## Quick Install

```bash
git clone https://github.com/Rudra-narayan-muduli-001/Scale-to-Millions.git
cd Scale-to-Millions
# Done — the .claude/, .opencode/, and .agents/ folders are already in the repo.
# Just copy them into your project or use globally:
# cp -r .claude/skills/scale-to-millions ~/.claude/skills/
```

> **Restart your agent** after adding the skill — it auto-discovers on next launch.

---

## How It Works

The skill **auto-activates** when you naturally talk about architecture or scaling. No special syntax.

```
Agent sees:  "Help me scale this app for more traffic"
          ↓
Agent scans skill descriptions (low token cost)
          ↓
Matches "scale" → loads scale-to-millions SKILL.md
          ↓
Follows 6-step Method (clarify → stage → bottleneck)
          ↓
Loads reference/*.md only when deeper detail needed
          ↓
Produces: roadmap + trade-offs + next bottleneck
```

### Progressive Disclosure (Saves tokens)

| File | When loaded | Lines |
|------|-------------|-------|
| `SKILL.md` | On activation (keyword match) | ~280 |
| `reference/architecture-stages.md` | Stage-specific actions + code | On demand |
| `reference/estimation-cheatsheet.md` | QPS / latency / storage math | On demand |
| `reference/interview-framework.md` | Interview prep mode | On demand |
| `reference/tool-choices.md` | Tool recommendations | On demand |

---

## The 12 Scaling Stages

```
 Users      Stage              What Changes
───────    ─────────           ─────────────────────────────────
    0     Single Server        Monolith — everything on one box
  100     Separate Database    App + DB on different machines
  1K      Load Balancer        Multiple web servers + health checks
 10K      DB Replication       Writes → primary, reads → replicas
 10K      Cache Layer          Redis/Memcached cache-aside
 10K      CDN                  Edge-cached static content
 ~~       Stateless Tier       Sessions → Redis → autoscaling
500K      Multi-Data Center    GeoDNS + cross-region sync
100K      Message Queue        Background workers decoupling
 ~~       Observability        Logs + metrics + alerts + CI/CD
  1M+     DB Sharding          Horizontal data partitioning
  1M+     Microservices        Domain-split, K8s independent
```

---

## Project Structure

```
scale-to-millions/
├── LICENSE
├── README.md
├── .gitignore
├── .claude/skills/scale-to-millions/
│   ├── SKILL.md
│   └── reference/
│       ├── architecture-stages.md
│       ├── estimation-cheatsheet.md
│       ├── interview-framework.md
│       └── tool-choices.md
├── .opencode/skills/scale-to-millions/     ← (same content)
└── .agents/skills/scale-to-millions/       ← (same content)
```

---

## Example Conversations

| You Say | Agent (loaded with skill) does |
|---------|-------------------------------|
| *"My SaaS hits 5K DAU and checkout times out"* | 1. Estimates load <br/> 2. Diagnoses DB bottleneck <br/> 3. Recommends Redis cache + read replicas <br/> 4. Predicts next bottleneck (write path) |
| *"Design a URL shortener for 100M/month"* | 1. QPS ≈ 40/s, storage ≈ 40GB/mo <br/> 2. Builds incrementally: single-server → LB → cache → shard <br/> 3. ASCII diagrams at each stage |
| *"Review my architecture for scaling"* | Audits current state, flags SPOFs, maps to nearest stage |
| *"Prepare me for a system design interview"* | Uses the bottleneck-then-fish pattern, coaching three minutes at a time |

---

## Decision Tree

```
Is the problem the WEB TIER?
  ├─ CPU/RAM capped?        → Add App servers + Load balancer
  ├─ Stateful (sessions)?   → Move to Redis
  └─ Need autoscaling?      → Health checks + Instance groups

Is the problem the DATABASE?
  ├─ READ-heavy?            → Read replicas + Redis cache
  ├─ WRITE-heavy?           → Vertical-first → shard if needed
  └─ Slow queries?          → Indexes, denormalize

Is the problem LATENCY?
  ├─ Static content?        → CDN
  ├─ Geographic spread?     → Multi-region + GeoDNS
  └─ Cold cache startup?    → Pre-warm on deploy

Is the problem RESILIENCE?
  ├─ No redundancy          → Redundancy at every tier
  ├─ No failover            → LB + DB HA + Geo-routing
  └─ Cascading failures     → Circuit breakers + retry + dead-letter queues
```

---

## Security Baseline (All Stages)

> Security is never optional.

- Rate limiting (token bucket, 100 req/min default)
- CORS: whitelist known origins only
- Security headers: CSP, HSTS, X-Frame-Options
- Input validation + parameterized queries (SQL injection prevention)
- Secrets in env vars or Vault — never in code
- `npm audit` / `pip audit` / `bundler-audit` on CI
- HTTPS everywhere (HTTP → 301 HTTPS)

---

## Observability Baseline (All Stages)

- **Structured JSON logging:** Winston / Pino / structlog / zap
- **Prometheus `/metrics` endpoint**
- **OpenTelemetry distributed tracing → Jaeger / Zipkin**
- **Grafana dashboards:** RPS, P50/P95/P99, error rate, DB conns
- **Alerts:** Error > 1%, Latency P99 > 500ms, DB CPU > 80%, Redis mem > 90%

---

## License

[MIT](LICENSE)

---

## Credits

Built on the [ByteByteGo](https://bytebytego.com/) system design methodology by Alex Xu — "Scale From Zero to Millions of Users" from *System Design Interview – An Insider's Guide, Volume 1*.
