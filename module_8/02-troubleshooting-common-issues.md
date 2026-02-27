---
name: Troubleshooting Common Issues
dependsOn:
  - architecture-review-and-integration
tags: [chatbots, debugging, troubleshooting, integration]
learningOutcomes:
  - Diagnose and resolve the five most common integration failures in chatbot systems.
  - Apply a systematic debugging strategy that isolates problems by component.
  - Test chatbot systems across happy path, edge case, error, and performance scenarios.
---

## Learning Outcomes

- Diagnose and resolve the five most common integration failures in chatbot systems.
- Apply a systematic debugging strategy that isolates problems by component.
- Test chatbot systems across happy path, edge case, error, and performance scenarios.

Integration is where individually working components reveal their incompatibilities. A RAG engine that retrieves documents perfectly in a test script may fail when connected to the chat endpoint because the document format doesn't match what the prompt expects. A Docker container that runs locally may crash in the cloud because an environment variable is missing. This section catalogues the most common integration failures, explains their root causes, and provides concrete solutions.

## RAG Not Retrieving Relevant Documents

The symptom is that the chatbot doesn't reference the correct documents, or its answers are generic despite having a document corpus. The bot seems to ignore the knowledge base entirely, or retrieves documents that are tangentially related but not useful.

The most common cause is a chunk size mismatch. If documents are chunked into very large pieces (say, 3,000 tokens), each chunk contains too many topics and the embedding becomes a blurred average that doesn't match specific queries well. If chunks are too small (50 tokens), they lack enough context to be meaningful. The fix is to experiment with chunk sizes — try 500, 1,000, and 1,500 tokens and compare retrieval quality.

Another frequent cause is an embedding model mismatch. If you embed documents with one model (say, `text-embedding-ada-002`) but embed queries with a different model or a different version, the vectors live in incompatible spaces and similarity search produces poor results. Verify that the same embedding model and version is used for both document ingestion and query embedding.

Poor vector search configuration also contributes. If the similarity threshold is too high, only near-exact matches are returned, missing relevant documents that use different phrasing. If it's too low, irrelevant documents flood the context. Adjust the threshold and the number of results (`top_k`) based on your corpus.

The best debugging approach is to test retrieval independently. Before testing the full chat flow, print the retrieved documents for a set of test queries and manually evaluate their relevance:

```python
# Test retrieval in isolation
test_queries = [
    "How do I reset my password?",
    "What are the pricing plans?",
    "How does the API authentication work?"
]

for query in test_queries:
    results = rag_engine.retrieve(query, top_k=5)
    print(f"\nQuery: {query}")
    for i, doc in enumerate(results):
        print(f"  {i+1}. Score: {doc['score']:.3f} | {doc['text'][:100]}...")
```

If retrieval quality is poor for most queries, the issue is systemic (chunk size, embedding model). If it's poor for specific queries, the issue is likely in how those particular documents were processed.

## Context Window Overflow

The symptom is an error message like "This model's maximum context length is X tokens" from the LLM API. The request is rejected because the combined size of the system prompt, conversation history, retrieved documents, and user message exceeds the model's context limit.

This usually happens in longer conversations. Each turn adds to the conversation history, and without management, the history grows until it overflows. It can also happen when the RAG engine retrieves too many documents or documents that are too large.

The solutions fall into three categories. First, implement conversation summarisation (covered in Module 4) — replace older conversation turns with a compressed summary, keeping the token count bounded. Second, limit retrieved documents to the top 3–5 most relevant, and impose a token budget on the total context size. Third, count tokens before making the API call and truncate or summarise if the count exceeds the limit:

```python
import tiktoken

def count_tokens(messages, model="gpt-3.5-turbo"):
    """Count the total tokens in a message list."""
    encoding = tiktoken.encoding_for_model(model)
    total = 0
    for message in messages:
        total += len(encoding.encode(message["content"]))
        total += 4  # overhead per message
    return total

def ensure_within_limit(messages, max_tokens=3500):
    """Trim conversation history to stay within token limit."""
    while count_tokens(messages) > max_tokens and len(messages) > 2:
        # Remove the oldest non-system message
        messages.pop(1)
    return messages
```

A sliding window that keeps the most recent N turns plus a summary of earlier turns is the most robust approach for production systems.

## Slow Response Times

The symptom is that responses take longer than 10 seconds, making the chatbot feel unresponsive. Users expect conversational latency — 2–3 seconds is acceptable, 5 seconds is tolerable, and anything beyond 10 seconds feels broken.

The first step is to identify the bottleneck. Response time is the sum of retrieval time (searching the vector database), generation time (the LLM API call), and overhead (network latency, serialisation, middleware). Profile each component independently:

```python
import time

async def chat_with_profiling(message):
    t0 = time.time()

    # Retrieval
    t1 = time.time()
    documents = await rag_engine.retrieve(message)
    retrieval_time = time.time() - t1

    # Generation
    t2 = time.time()
    response = await llm.generate(messages, documents)
    generation_time = time.time() - t2

    total_time = time.time() - t0
    logger.info("request_profiling", extra={
        "retrieval_ms": round(retrieval_time * 1000),
        "generation_ms": round(generation_time * 1000),
        "total_ms": round(total_time * 1000)
    })

    return response
```

If retrieval is the bottleneck, add semantic caching (Module 3) to serve repeated or similar queries from cache. Optimise vector search by using approximate nearest-neighbour methods rather than exact search, and consider reducing the corpus size or improving index configuration.

If generation is the bottleneck, reduce the prompt size (fewer retrieved documents, shorter system prompt, summarised history). Use response streaming (Module 5) so users see tokens as they're generated rather than waiting for the complete response. Consider routing simple queries to a faster, cheaper model (Module 7).

If overhead is the bottleneck, ensure you're using async/await correctly throughout the request path — a single synchronous call in an async handler blocks the entire event loop.

## Docker Container Won't Start

The symptom is that the container exits immediately after starting, or it starts but the application inside isn't reachable. This is an environmental issue, not a code bug, which makes it frustrating to debug.

The first diagnostic step is to check the container logs:

```bash
docker logs <container_name>
```

The most common cause is missing environment variables. The application tries to read `OPENAI_API_KEY` or `DATABASE_URL` on startup, finds nothing, and crashes. Verify that your `.env` file exists, is loaded correctly by docker-compose, and contains all required variables. A common mistake is having the `.env` file in the wrong directory or having it listed in `.dockerignore`.

Port conflicts are another frequent cause. If port 8000 is already in use on the host, the container starts but isn't accessible. Change the port mapping in your docker-compose file: `-p 8001:8000` maps host port 8001 to container port 8000.

Dependency installation failures happen when a package requires system-level libraries that aren't present in the Docker base image. Check the build logs for errors during `pip install`. The fix is usually to add the missing system package to the Dockerfile:

```dockerfile
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
```

When all else fails, rebuild from scratch with `docker-compose build --no-cache` to eliminate stale cached layers.

## Frontend Can't Connect to Backend

The symptom is CORS errors in the browser console or "connection refused" errors. The frontend is running but can't communicate with the backend API.

CORS (Cross-Origin Resource Sharing) errors occur when the frontend and backend are served from different origins (different ports count as different origins). The fix is to add CORS middleware to FastAPI:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # frontend URL
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

"Connection refused" means the backend isn't running or the frontend is using the wrong URL. Verify the backend is accessible by testing with curl:

```bash
curl http://localhost:8000/health
```

A subtle variant occurs in Docker environments: the frontend container can't reach the backend using `localhost` because each container has its own network namespace. Use the Docker service name instead (e.g., `http://backend:8000`), or configure the Docker network so both services can communicate.

When deploying to the cloud, the backend URL changes from `localhost` to the deployed URL. If the frontend has the backend URL hardcoded, it will fail in production. Use environment variables to configure the backend URL so it can differ between local development and production.

## Systematic Debugging Strategy

When facing an integration failure, resist the urge to make random changes. Instead, follow a systematic process:

1. **Isolate the problem.** Is the issue in the frontend, backend, database, or an external API? Test each component independently.

2. **Test components independently.** Test RAG retrieval separately from the full chat flow. Test LLM generation with fixed, known-good context. Test the database connection with a simple query. This narrows the scope of the problem.

3. **Check logs.** Application logs, Docker logs (`docker logs`), and the browser console each provide different perspectives. Look for error messages, stack traces, and unexpected values.

4. **Add logging liberally.** If the existing logs don't reveal the issue, add temporary logging at key points in the data flow. Log the input and output of each component to find where the data diverges from expectations.

5. **Verify environment variables.** A surprising number of integration failures trace back to a missing or incorrect environment variable. Print them at startup (redacting secrets) to confirm they're set.

## Testing Scenarios

Before considering your capstone complete, test it against four categories of scenarios.

**Happy path testing** verifies the normal conversation flow. Send a greeting, ask a domain-specific question, follow up with a related question (testing memory), and ask a question that requires multiple retrieved documents.

**Edge case testing** probes the boundaries. Send an empty message. Send a very long message (1,000+ words). Send gibberish. Ask an off-topic question that has nothing to do with your domain. Each of these should produce a reasonable response, not a crash or an unhelpful error.

**Error case testing** verifies graceful degradation. Remove the API key and confirm the error message is helpful ("API key missing" not "Error 500"). Disconnect the database and verify the system returns a fallback response. Send requests past the rate limit and confirm the 429 response. Each failure mode should be handled, logged, and communicated clearly to the user.

**Performance testing** verifies the system under load. Send 10 concurrent requests and measure response times. Run a 50-turn conversation and verify memory management keeps the token count bounded. Load a large document corpus and verify retrieval times remain acceptable. These tests reveal bottlenecks that don't appear in single-request testing.
