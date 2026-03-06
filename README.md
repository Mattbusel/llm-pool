# llm-pool

Concurrent request pool for LLM API calls. Rate limiting, priority queuing, zero deps.

Single-header C++ library (`include/llm_pool.hpp`) providing a thread pool with a priority queue and token-bucket rate limiter — purpose-built for managing concurrent calls to OpenAI, Anthropic, and compatible LLM APIs.

---

## Quickstart

```cpp
#define LLM_POOL_IMPLEMENTATION
#include "llm_pool.hpp"
#include <iostream>

int main() {
    llm::PoolConfig cfg;
    cfg.max_concurrent      = 5;
    cfg.requests_per_minute = 300;
    cfg.tokens_per_minute   = 90000;

    llm::Pool pool(cfg);

    for (int i = 0; i < 20; ++i) {
        llm::PoolRequest req;
        req.id              = "req-" + std::to_string(i);
        req.priority        = llm::Priority::Normal;
        req.estimated_tokens = 150;
        req.fn = [i]() {
            // your API call here
        };
        pool.submit(std::move(req), [](llm::PoolResult r) {
            std::cout << r.request_id << (r.success ? " OK" : " FAIL") << "\n";
        });
    }

    pool.drain();
    auto s = pool.stats();
    std::cout << "completed=" << s.completed << " rejected=" << s.rejected << "\n";
}
```

---

## API Reference

### PoolConfig

| Field | Type | Default | Description |
|---|---|---|---|
| `max_concurrent` | `size_t` | `10` | Number of worker threads |
| `max_queue_size` | `size_t` | `1000` | Maximum pending request queue depth |
| `requests_per_minute` | `size_t` | `500` | Rate limit: max requests per 60s window |
| `tokens_per_minute` | `size_t` | `100000` | Rate limit: max tokens per 60s window |
| `queue_timeout_ms` | `double` | `5000.0` | Max time a request may sit in queue before rejection |

### Priority enum

```cpp
enum class Priority { Low = 0, Normal = 1, High = 2, Critical = 3 };
```

Higher-priority requests are dequeued first. `Critical > High > Normal > Low`.

### PoolRequest

```cpp
struct PoolRequest {
    std::string           id;               // Unique identifier (user-defined)
    std::function<void()> fn;               // Work to execute
    Priority              priority;         // Default: Normal
    size_t                estimated_tokens; // Used for token-bucket accounting; default: 100
};
```

### Pool methods

| Method | Description |
|---|---|
| `Pool(PoolConfig)` | Construct pool and start `max_concurrent` worker threads |
| `submit(PoolRequest, on_complete)` | Enqueue asynchronously; callback fires with `PoolResult` on completion or rejection |
| `submit_sync(PoolRequest)` | Enqueue and block until `PoolResult` is available |
| `drain()` | Block until queue is empty and all in-flight requests finish |
| `shutdown()` | Stop accepting new work, join all workers. Called automatically by destructor |
| `stats()` | Return a `Stats` snapshot (see below) |

### PoolResult

```cpp
struct PoolResult {
    std::string request_id;  // Echoes PoolRequest::id
    bool        success;     // false on exception, timeout, or rejection
    double      wait_ms;     // Time spent in queue
    double      execute_ms;  // Time spent executing fn()
    std::string error;       // Non-empty on failure
};
```

### Stats

```cpp
struct Stats {
    size_t in_flight;      // Currently executing
    size_t queued;         // Waiting in queue
    size_t completed;      // Total finished (success or error)
    size_t rejected;       // Rejected due to queue-full, rate-limit, or shutdown
    size_t rate_limited;   // Subset of rejected: specifically rate-limited
    double avg_wait_ms;    // Average queue wait across all completed
    double avg_execute_ms; // Average execution time across all completed
};
```

---

## Examples

| File | Description |
|---|---|
| [`examples/basic_pool.cpp`](examples/basic_pool.cpp) | 10 requests, 3 concurrent slots, async callbacks |
| [`examples/priority_queue.cpp`](examples/priority_queue.cpp) | Demonstrates CRITICAL > HIGH > LOW ordering |
| [`examples/rate_limited.cpp`](examples/rate_limited.cpp) | Low RPM limit showing rate-limit rejection path |

---

## Building

Requires CMake 3.16+ and a C++17 compiler (MSVC, GCC, Clang).

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Run examples
./build/basic_pool
./build/priority_queue
./build/rate_limited
```

On Windows with MSVC:
```bash
cmake -B build
cmake --build build --config Release
build\Release\basic_pool.exe
```

---

## See Also

This library is part of the **llm-suite** — a collection of single-header C++ utilities for LLM API integration:

| Repo | Description |
|---|---|
| [llm-stream](https://github.com/Mattbusel/llm-stream) | Stream OpenAI & Anthropic responses |
| [llm-retry](https://github.com/Mattbusel/llm-retry) | Retry + circuit breaker |
| [llm-cache](https://github.com/Mattbusel/llm-cache) | LRU response cache |
| [llm-cost](https://github.com/Mattbusel/llm-cost) | Token counting + cost estimation |
| [llm-format](https://github.com/Mattbusel/llm-format) | Structured output formatting |
| [llm-embed](https://github.com/Mattbusel/llm-embed) | Text embeddings + vector search |
| [llm-pool](https://github.com/Mattbusel/llm-pool) | Concurrent request pool (this repo) |
| [llm-log](https://github.com/Mattbusel/llm-log) | Structured JSONL logging |
| [llm-template](https://github.com/Mattbusel/llm-template) | Prompt templating |
| [llm-agent](https://github.com/Mattbusel/llm-agent) | Tool-calling agent loop |
| [llm-rag](https://github.com/Mattbusel/llm-rag) | Retrieval-augmented generation |
| [llm-eval](https://github.com/Mattbusel/llm-eval) | Evaluation + consistency scoring |

---

## License

MIT License. Copyright 2026 Mattbusel. See [LICENSE](LICENSE).
