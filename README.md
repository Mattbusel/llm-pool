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

| Repo | Purpose |
|------|---------|
| [llm-stream](https://github.com/Mattbusel/llm-stream) | SSE streaming |
| [llm-cache](https://github.com/Mattbusel/llm-cache) | Response caching |
| [llm-cost](https://github.com/Mattbusel/llm-cost) | Token cost estimation |
| [llm-retry](https://github.com/Mattbusel/llm-retry) | Retry + circuit breaker |
| [llm-format](https://github.com/Mattbusel/llm-format) | Markdown/code formatting |
| [llm-embed](https://github.com/Mattbusel/llm-embed) | Embeddings + cosine similarity |
| [llm-pool](https://github.com/Mattbusel/llm-pool) | Connection pooling |
| [llm-log](https://github.com/Mattbusel/llm-log) | Structured logging |
| [llm-template](https://github.com/Mattbusel/llm-template) | Prompt templates |
| [llm-agent](https://github.com/Mattbusel/llm-agent) | Tool-use agent loop |
| [llm-rag](https://github.com/Mattbusel/llm-rag) | Retrieval-augmented generation |
| [llm-eval](https://github.com/Mattbusel/llm-eval) | Output evaluation |
| [llm-chat](https://github.com/Mattbusel/llm-chat) | Multi-turn chat |
| [llm-vision](https://github.com/Mattbusel/llm-vision) | Vision/image inputs |
| [llm-mock](https://github.com/Mattbusel/llm-mock) | Mock LLM for testing |
| [llm-router](https://github.com/Mattbusel/llm-router) | Model routing |
| [llm-guard](https://github.com/Mattbusel/llm-guard) | Content moderation |
| [llm-compress](https://github.com/Mattbusel/llm-compress) | Prompt compression |
| [llm-batch](https://github.com/Mattbusel/llm-batch) | Batch processing |
| [llm-audio](https://github.com/Mattbusel/llm-audio) | Audio transcription/TTS |
| [llm-finetune](https://github.com/Mattbusel/llm-finetune) | Fine-tuning jobs |
| [llm-rank](https://github.com/Mattbusel/llm-rank) | Passage reranking |
| [llm-parse](https://github.com/Mattbusel/llm-parse) | HTML/markdown parsing |
| [llm-trace](https://github.com/Mattbusel/llm-trace) | Distributed tracing |
| [llm-ab](https://github.com/Mattbusel/llm-ab) | A/B testing |
| [llm-json](https://github.com/Mattbusel/llm-json) | JSON parsing/building |

## License

MIT — see [LICENSE](LICENSE).
