# llm-pool

[![CI](https://github.com/Mattbusel/llm-pool/actions/workflows/ci.yml/badge.svg)](https://github.com/Mattbusel/llm-pool/actions/workflows/ci.yml)
![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

**Run many LLM requests concurrently with priorities and rate limits.** One header, `llm_pool.hpp`. Needs nothing beyond the C++17 standard library.

Batch jobs and busy services need to fire many LLM calls at once without tripping provider rate limits or letting low-priority work starve urgent requests. llm-pool is a small worker pool that runs your request functions on a fixed number of threads, orders them by priority and enforces requests-per-minute and tokens-per-minute budgets.

## Features

- Fixed number of worker threads (`max_concurrent`)
- Priority queue: `Critical`, `High`, `Normal`, `Low`
- Requests-per-minute and tokens-per-minute budgets using each request's `estimated_tokens`
- Async `submit()` with a completion callback, or blocking `submit_sync()`
- `drain()` and `shutdown()`; exceptions thrown by a request are caught and reported in `PoolResult.error`
- Stats: in flight, queued, completed, rejected, rate limited, average wait and execution time
- Transport agnostic: the pool runs any `std::function<void()>`, so it works with llm-stream, your own HTTP client or anything else

## Quick start

Copy the header into your project:

```bash
curl -fsSLO https://raw.githubusercontent.com/Mattbusel/llm-pool/main/include/llm_pool.hpp
```

Define `LLM_POOL_IMPLEMENTATION` in exactly one `.cpp` file before including it; every other file just includes the header. Save this as `main.cpp` next to the header:

```cpp
#define LLM_POOL_IMPLEMENTATION
#include "llm_pool.hpp"
#include <iostream>

int main() {
    llm::PoolConfig cfg;
    cfg.max_concurrent      = 4;       // worker threads
    cfg.requests_per_minute = 60;
    cfg.tokens_per_minute   = 90000;
    llm::Pool pool(cfg);

    std::mutex out;
    for (int i = 0; i < 8; ++i) {
        llm::PoolRequest req;
        req.id               = "req-" + std::to_string(i);
        req.priority         = (i == 7) ? llm::Priority::Critical : llm::Priority::Normal;
        req.estimated_tokens = 500;
        req.fn = [] { /* call your LLM API here; throw to report failure */ };

        pool.submit(std::move(req), [&out](llm::PoolResult r) {
            std::lock_guard<std::mutex> lk(out);
            std::cout << r.request_id << (r.success ? " ok" : " failed: " + r.error)
                      << " (waited " << r.wait_ms << " ms)\n";
        });
    }
    pool.drain();

    auto s = pool.stats();
    std::cout << s.completed << " completed, " << s.rate_limited << " rate-limited\n";
}
```

```bash
g++ -std=c++17 -O2 main.cpp -pthread -o demo
```

## API at a glance

| Call | Purpose |
|---|---|
| `Pool(PoolConfig)` | Start the workers |
| `submit(PoolRequest, on_complete)` | Queue a request; callback receives a `PoolResult` |
| `submit_sync(PoolRequest)` | Queue and block until done |
| `drain()` / `shutdown()` | Wait for all work / stop accepting new work |
| `stats()` | Counters and average timings |

## Notes and limitations

- When the per-minute budget is exhausted, or the queue is full, new submissions are rejected immediately with an error (`"rate limited"`, `"queue full"`) rather than delayed. Check `PoolResult.success` and resubmit later if you need that.
- Rejection callbacks run on the submitting thread while the pool's lock is held, so do not call `submit()` from inside them.

## Build the examples

The repo builds `examples/basic_pool.cpp`, `examples/priority_queue.cpp`, `examples/rate_limited.cpp` with CMake:

```bash
cmake -B build
cmake --build build
```

## Part of llm-cpp

llm-pool is one of 26 single-header C++ libraries in [llm-cpp](https://github.com/Mattbusel/llm-cpp), a toolkit for building LLM features into native code. Each library stands alone; combine them by giving each `*_IMPLEMENTATION` define its own `.cpp` file. See the [llm-cpp README](https://github.com/Mattbusel/llm-cpp#using-several-together) for the full list and examples of using several together.

## License

MIT. See [LICENSE](LICENSE).
