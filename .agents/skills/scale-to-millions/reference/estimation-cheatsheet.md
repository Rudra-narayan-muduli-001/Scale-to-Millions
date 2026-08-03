# Back-of-Envelope Estimation Cheatsheet

## Traffic → QPS

| Traffic | Average QPS | Peak QPS (2-3×) |
|---------|-------------|------------------|
| 1M requests/day | ~12 req/s | ~35 req/s |
| 10M requests/day | ~115 req/s | ~350 req/s |
| 100M requests/day | ~1,150 req/s | ~3,500 req/s |
| 1B requests/day | ~11,574 req/s | ~35,000 req/s |

**Formula:** QPS = (requests per day) / 86,400. Peak ≈ 2–3× average.

## Storage Rules of Thumb

| Unit | Value |
|------|-------|
| 1 character | ≈ 1 byte (ASCII) |
| 1 KB | 10^3 B |
| 1 MB | 10^6 B |
| 1 GB | 10^9 B |
| 1 TB | 10^12 B |

**Storage estimate formula:**
```
(records/day) × (avg record size) × (retention period in days)
```
Add +20-30% overhead for indexes, replication, and metadata.

**Example:** 10M users, avg user row 1KB → 10 GB raw. With 20% index overhead: ~12 GB.

## Latency Numbers Every Engineer Should Know

| Operation | Approximate time |
|-----------|-----------------|
| L1 cache reference | ~1 ns |
| L2 cache reference | ~10 ns |
| Main memory reference | ~100 ns |
| SSD random read | ~100 µs |
| Redis / in-memory cache read | ~1 ms (network) |
| Roundtrip same data center | ~0.5 ms |
| Disk seek | ~10 ms |
| Roundtrip US ↔ Europe | ~100–150 ms |
| 1 GB over network (1 Gbps) | ~8 s |

**Rule of thumb:** Memory is ~100x faster than SSD; ~100,000x faster than cross‑continent network.

## Availability ("Nines")

| Availability % | Downtime per Year | Downtime per Month | Downtime per Week |
|----------------|--------------------|--------------------|-------------------|
| 99%            | ~3.65 days         | ~7.2 hours         | ~1.68 hours       |
| 99.9%          | ~8.77 hours        | ~43 minutes         | ~10 minutes       |
| 99.99%         | ~52 minutes        | ~4.3 minutes         | ~1 minute         |
| 99.999%        | ~5 minutes         | ~26 seconds         | ~6 seconds        |

## When Answering a Design Question

Always state assumptions explicitly. Example:

> "Assuming 50M DAU, avg 10 requests/user/day, 1KB avg payload → 500M requests/day ≈ 5,800 QPS avg, ~15- 20K QPS peak. Raw ingest ~500 GB/day. Add +20% overhead → 600 GB/day storage growth."

Design against specific numbers, not "web scale."