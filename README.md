# Scale to Millions — AI Agent Skill

An [Agent Skills](https://github.com/anthropics/skills) standard skill that turns the **ByteByteGo "Scale From Zero to Millions of Users"** framework into a reusable, progressive system-design playbook for AI coding agents.

When loaded, the agent acts as a Senior Distributed Systems Architect — analyzing your current architecture, identifying bottlenecks, and producing **incremental, actionable scaling plans with real code and configs** — not just theory.

## Supported Agents

| Agent | Path |
|---|---|
| **Claude Code** | `.claude/skills/scale-to-millions/` |
| **OpenCode** | `.opencode/skills/scale-to-millions/` or reads `.claude/skills/` |
| **Codex CLI** | `.agents/skills/scale-to-millions/` |
| **Cursor / OpenClaw** | `.claude/skills/scale-to-millions/` |
| **Global (all)** | `~/.claude/skills/scale-to-millions/` |

## Quick Install

```bash
mkdir -p .claude/skills/scale-to-millions/reference
# Copy all files from this repo's .claude/skills/scale-to-millions/ into the directory above
```

Or globally (available in every project):

```bash
mkdir -p ~/.claude/skills/scale-to-millions/reference
# Copy skill files into ~/.claude/skills/scale-to-millions/
```

## How It Works

The skill auto-activates when you say things like:

- *"Help me scale this app for more traffic"*
- *"Design a URL shortener for millions of users"*
- *"My database is slow at 50K users"*
- *"Should I add caching or read replicas?"*
- *"Review this architecture for scaling bottlenecks"*
- *"Prepare me for a system design interview"*

### Progressive Disclosure

Uses the standard Agent Skills progressive disclosure pattern:

| File | When loaded |
|------|-------------|
| `SKILL.md` | Always — scanned at startup for trigger phrases + loaded on activation |
| `reference/architecture-stages.md` | On demand — when the agent needs stage-specific actions |
| `reference/estimation-cheatsheet.md` | On demand — for QPS math, storage estimates, latency numbers |
| `reference/interview-framework.md` | On demand — when the user is preparing for an interview |
| `reference/tool-choices.md` | On demand — when the agent needs to recommend specific tools |

## The 12 Scaling Stages

| # | Stage | User scale | What it adds |
|---|-------|------------|---------------|
| 1 | Single server | 0–100 | Web app + DB + cache on one box |
| 2 | Separate DB tier | 100–1K | Dedicated DB server; SQL vs NoSQL |
| 3 | Load balancer | 1K–10K | Multiple web servers behind LB; stateless tier |
| 4 | DB replication | 10K–100K | Master (writes) + read replicas |
| 5 | Cache layer | 10K–100K | Redis/Memcached cache-aside |
| 6 | CDN | 10K–100K | Edge-cached static assets |
| 7 | Stateless web tier | 1K–10K | Shared session store; autoscaling |
| 8 | Multi–data center | 500K+ | GeoDNS; cross-region |
| 9 | Message queue | 100K–500K | Async producers/consumers |
| 10 | Observability | All stages | Logging, metrics, CI/CD |
| 11 | DB sharding | 1M+ | Horizontal partitioning |
| 12 | Microservices | 1M+ | Split by domain; independent scaling |

## Key Principles

- **Iterative, not a big redesign.** Solve the current bottleneck, then move to the next.
- **Start with the simplest thing that works.** Don't jump to microservices + sharding for 100 users.
- **Every scaling step trades cost, consistency, or complexity for capacity.** Name the trade-off explicitly.
- **Prefer horizontal over vertical scaling** once a single machine hits its ceiling.

## File Structure

```
.claude/skills/scale-to-millions/
├── SKILL.md
└── reference/
    ├── architecture-stages.md
    ├── estimation-cheatsheet.md
    ├── interview-framework.md
    └── tool-choices.md
```

## License

MIT — see [LICENSE](LICENSE).

## Credits

Built on the system design methodology from [ByteByteGo](https://bytebytego.com/) by Alex Xu — specifically the "Scale From Zero to Millions of Users" chapter from *System Design Interview – An Insider's Guide, Volume 1*.