# CLAUDE.md — llm-pool

## Build & Run
```bash
cmake -B build && cmake --build build
./build/basic_pool
./build/priority_queue
./build/rate_limited
```

## THE RULE: Single Header
`include/llm_pool.hpp` is the entire library. No source files to compile separately.

## API
- `Pool(PoolConfig)` — construct, starts `max_concurrent` worker threads
- `submit(PoolRequest, on_complete)` — async, returns immediately
- `submit_sync(PoolRequest)` → `PoolResult` — blocks until complete
- `drain()` — wait for queue empty + `in_flight == 0`
- `shutdown()` — stop + join all workers (destructor calls this automatically)
- `stats()` → `Stats` — snapshot of current pool metrics

## PoolConfig fields
| Field | Default | Description |
|---|---|---|
| `max_concurrent` | 10 | Worker thread count |
| `max_queue_size` | 1000 | Max pending requests |
| `requests_per_minute` | 500 | Rate limit (requests) |
| `tokens_per_minute` | 100000 | Rate limit (tokens) |
| `queue_timeout_ms` | 5000 | Max queue wait before rejection |

## Common mistakes
- Forgetting `#define LLM_POOL_IMPLEMENTATION` in exactly one `.cpp`
- Pool destructor calls `shutdown()` — do not call it twice
- `drain()` waits for queue AND in-flight to reach zero; call before inspecting stats
