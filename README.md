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
| [llm-stream](https://github.com/Mattbusel/llm-stream) | Stream OpenAI and Anthropic responses via SSE |
| [llm-cache](https://github.com/Mattbusel/llm-cache) | LRU response cache |
| [llm-cost](https://github.com/Mattbusel/llm-cost) | Token counting and cost estimation |
| [llm-retry](https://github.com/Mattbusel/llm-retry) | Retry and circuit breaker |
| [llm-format](https://github.com/Mattbusel/llm-format) | Structured output / JSON schema |
| [llm-embed](https://github.com/Mattbusel/llm-embed) | Embeddings and vector search |
| [llm-pool](https://github.com/Mattbusel/llm-pool) | Concurrent request pool |
| [llm-log](https://github.com/Mattbusel/llm-log) | Structured JSONL logging |
| [llm-template](https://github.com/Mattbusel/llm-template) | Prompt templating |
| [llm-agent](https://github.com/Mattbusel/llm-agent) | Tool-calling agent loop |
| [llm-rag](https://github.com/Mattbusel/llm-rag) | RAG pipeline |
| [llm-eval](https://github.com/Mattbusel/llm-eval) | Evaluation and consistency scoring |
| [llm-chat](https://github.com/Mattbusel/llm-chat) | Conversation memory manager |
| [llm-vision](https://github.com/Mattbusel/llm-vision) | Multimodal image+text |
| [llm-mock](https://github.com/Mattbusel/llm-mock) | Mock LLM for testing |
| [llm-router](https://github.com/Mattbusel/llm-router) | Model routing by complexity |
| [llm-guard](https://github.com/Mattbusel/llm-guard) | PII detection and injection guard |
| [llm-compress](https://github.com/Mattbusel/llm-compress) | Context compression |
| [llm-batch](https://github.com/Mattbusel/llm-batch) | Batch processing and checkpointing |

## License

MIT — see [LICENSE](LICENSE).
