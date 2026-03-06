# AGENTS.md — llm-pool

## Purpose
Single-header C++ thread pool + priority queue + token-bucket rate limiter for LLM API calls.

## Architecture
Everything in `include/llm_pool.hpp`. Guard: `#ifdef LLM_POOL_IMPLEMENTATION`. No external deps.

## Build
```bash
cmake -B build && cmake --build build
```

## Constraints
- Single header, no external deps, C++17, namespace `llm`
- Thread-safe via mutex + condvar
- Token bucket rate limiting
- Priority queue via `std::priority_queue<Entry>` (max-heap on Priority enum)
