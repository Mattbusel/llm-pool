# llm-pool

Concurrent LLM request pool with priority queue and rate limiting. One header. No deps.

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Single Header](https://img.shields.io/badge/library-single--header-orange)

---

## 30-second quickstart

```cpp
#define LLM_POOL_IMPLEMENTATION
#include "llm_pool.hpp"

int main() {
    llm::PoolConfig cfg;
    cfg.max_concurrent      = 5;
    cfg.requests_per_minute = 200;

    llm::Pool pool(cfg);

    for (int i = 0; i < 20; ++i) {
        llm::PoolRequest req;
        req.id = "req-" + std::to_string(i);
        req.fn = [i]{ /* your llm::stream_openai call here */ };
        req.priority = llm::Priority::Normal;
        pool.submit(req);
    }

    pool.drain(); // wait for all to finish

    auto s = pool.stats();
    std::cout << s.completed << " done, " << s.rejected << " rejected\n";
}
```

---

## Installation

```bash
cp include/llm_pool.hpp your-project/
```

No external dependencies (std::thread, std::mutex, C++17).

---

## API Reference

```cpp
// Submit async
pool.submit(request, [](llm::PoolResult r){ /* callback */ });

// Submit and block
llm::PoolResult r = pool.submit_sync(request);

// Priorities: Low, Normal, High, Critical
req.priority = llm::Priority::Critical;

// Rate limiting (token bucket, resets every 60s)
cfg.requests_per_minute = 500;
cfg.tokens_per_minute   = 100000;

// Stats
auto s = pool.stats();
// s.in_flight, s.queued, s.completed, s.rejected, s.avg_wait_ms
```

---

## Building

```bash
cmake -B build && cmake --build build
./build/basic_pool
./build/priority_queue
```

---

## Requirements

- C++17 (std::thread, std::mutex, std::condition_variable)
- No external dependencies

---

## See Also

| Repo | What it does |
|------|-------------|
| [llm-stream](https://github.com/Mattbusel/llm-stream) | Stream OpenAI & Anthropic responses token by token |
| [llm-cache](https://github.com/Mattbusel/llm-cache) | Cache responses, skip redundant calls |
| [llm-cost](https://github.com/Mattbusel/llm-cost) | Token counting + cost estimation |
| [llm-retry](https://github.com/Mattbusel/llm-retry) | Retry with backoff + circuit breaker |
| [llm-format](https://github.com/Mattbusel/llm-format) | Structured output enforcement |
| [llm-embed](https://github.com/Mattbusel/llm-embed) | Text embeddings + nearest-neighbor search |
| [llm-pool](https://github.com/Mattbusel/llm-pool) | Concurrent request pool + rate limiting |
| [llm-log](https://github.com/Mattbusel/llm-log) | Structured JSONL logger for LLM calls |
| [llm-template](https://github.com/Mattbusel/llm-template) | Prompt templating with loops + conditionals |
| [llm-agent](https://github.com/Mattbusel/llm-agent) | Tool-calling agent loop |
| [llm-rag](https://github.com/Mattbusel/llm-rag) | Full RAG pipeline |
| [llm-eval](https://github.com/Mattbusel/llm-eval) | Consistency and quality evaluation |

---

## License

MIT — see [LICENSE](LICENSE).
