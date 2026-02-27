---
name: Capstone Architecture and Troubleshooting
dependsOn:
  - alternative-llm-providers
tags: [chatbots, architecture, integration, debugging, troubleshooting, capstone]
learningOutcomes:
  - Map the complete reference architecture of a production chatbot system and evaluate a project against it.
  - Diagnose and resolve the five most common integration failures in chatbot systems.
  - Apply a systematic debugging strategy that isolates problems by component.
  - Prioritise remaining implementation work using a structured priority framework.
---

## Learning Outcomes

- Map the complete reference architecture of a production chatbot system and evaluate a project against it.
- Diagnose and resolve the five most common integration failures in chatbot systems.
- Apply a systematic debugging strategy that isolates problems by component.
- Prioritise remaining implementation work using a structured priority framework.

The capstone project brings together every component built across this course — LLM integration, RAG pipelines, memory management, deployment infrastructure, analytics, and production hardening — into a single, cohesive system. Integration is where most projects stumble. Each piece works in isolation, but connecting them reveals mismatches in data formats, assumptions about state, and subtle timing issues. This section provides the reference architecture your capstone should target, a checklist to evaluate your progress, and practical guidance for diagnosing and resolving the most common integration failures.

## Reference Architecture

A production chatbot system is organised into layers, each with a clear responsibility. The reference architecture below represents the target state — not every capstone will implement every component, but the architecture shows how the pieces fit together.

```
┌─────────────────────────────────────────────────────────┐
│                      FRONTEND LAYER                      │
│  ┌──────────────┐         ┌──────────────┐              │
│  │   Web UI     │         │   Mobile     │              │
│  │  (React)     │         │    App       │              │
│  └──────┬───────┘         └──────┬───────┘              │
│         └────────────┬───────────┘                       │
└──────────────────────┼───────────────────────────────────┘
                       │ HTTPS/WSS
┌──────────────────────┼───────────────────────────────────┐
│                 API GATEWAY                               │
│     (Load Balancer, Rate Limiting, Auth)                 │
└──────────────────────┼───────────────────────────────────┘
                       │
┌──────────────────────┼───────────────────────────────────┐
│                 BACKEND LAYER                             │
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │           FastAPI Application Servers             │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │    │
│  │  │ Server 1 │  │ Server 2 │  │ Server 3 │      │    │
│  │  └─────┬────┘  └─────┬────┘  └─────┬────┘      │    │
│  └────────┼─────────────┼─────────────┼────────────┘    │
│           │             │             │                   │
│  ┌────────┼─────────────┼─────────────┼────────────┐    │
│  │  ┌─────▼──────┐ ┌───▼────┐  ┌────▼──────┐     │    │
│  │  │  Chatbot   │ │  RAG   │  │  Memory   │     │    │
│  │  │   Core     │ │ Engine │  │  Manager  │     │    │
│  │  └────────────┘ └────────┘  └───────────┘     │    │
│  └────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────┘
                       │
┌──────────────────────┼───────────────────────────────────┐
│                 DATA & STORAGE LAYER                      │
│  ┌────────────┐  ┌──────────┐  ┌──────────┐             │
│  │ PostgreSQL │  │  Redis   │  │ ChromaDB │             │
│  │(User Data) │  │ (Cache)  │  │(Vectors) │             │
│  └────────────┘  └──────────┘  └──────────┘             │
└──────────────────────────────────────────────────────────┘
                       │
┌──────────────────────┼───────────────────────────────────┐
│                 EXTERNAL SERVICES                         │
│  ┌────────────┐  ┌──────────┐  ┌──────────┐             │
│  │  OpenAI    │  │ Analytics│  │  Logs    │             │
│  │    API     │  │(Datadog) │  │ (ELK)   │             │
│  └────────────┘  └──────────┘  └──────────┘             │
└──────────────────────────────────────────────────────────┘
```

The **frontend layer** provides the user interface — a web application built with React or Streamlit, potentially with a mobile client. It handles real-time message display, file uploads for document ingestion, and conversation history views.

The **API gateway** sits between the frontend and backend. It handles load balancing across multiple backend instances, enforces rate limits, and validates authentication tokens. In a simple deployment this might be NGINX; in a cloud environment it could be a managed API gateway.

The **backend layer** contains the FastAPI application servers, which are stateless and horizontally scalable. Within each server, three core services coordinate: the chatbot core orchestrates the conversation flow, the RAG engine retrieves relevant documents from the vector database, and the memory manager maintains conversation history with summarisation and context management.

The **data and storage layer** provides persistence. PostgreSQL stores user data, session information, and analytics. Redis serves as a cache for semantic caching and distributed session state. ChromaDB (or Pinecone, for production scale) stores vector embeddings for the RAG pipeline.

The **external services layer** includes the LLM provider (OpenAI, Anthropic, or a self-hosted model), analytics platforms for metrics aggregation, and log management systems for structured log storage and search.

## Component Integration Checklist

Use this checklist to evaluate your capstone project against the reference architecture. Each item should be testable — you should be able to demonstrate it working.

**Core Functionality:**
- Chat API endpoint accepts messages and returns responses
- RAG retrieval fetches relevant documents for a given query
- LLM generation creates responses grounded in retrieved context
- Conversation history is maintained across turns
- Memory management prevents token overflow

**Data Flow:**
- User messages flow through the full pipeline: API → RAG retrieval → LLM generation → response
- Retrieved documents are properly formatted and injected into the LLM prompt
- Conversation history is stored and retrieved correctly between requests
- User profiles are created and updated based on conversation data

**Error Handling:**
- Invalid inputs return appropriate HTTP error codes and messages
- API failures (LLM timeouts, database errors) are caught and handled gracefully
- Fallback responses are returned when unexpected failures occur
- Rate limit exceeded returns a proper 429 HTTP status

**Performance:**
- Response time p95 is under 5 seconds
- Semantic caching reduces redundant API calls
- Database queries are optimised with appropriate indexes
- No memory leaks in long-running processes

**Security:**
- Authentication is required for protected endpoints
- API keys are stored securely in environment variables, never in code
- Content moderation is active on both inputs and outputs
- Input validation is applied to all endpoints

**Deployment:**
- The application is Dockerised and runs via docker-compose
- The application is deployed to a cloud provider with a public URL
- A health check endpoint responds to monitoring probes
- Environment variables are configured correctly in all environments

**Observability:**
- Logging uses structured format (JSON) with trace IDs
- Key metrics are tracked: latency, cost, error rate
- A dashboard or metrics endpoint displays system health
- Alerts are configured for critical failures

Not every item needs to be implemented for a successful capstone — the core functionality and error handling sections are essential, while advanced performance and observability features strengthen the project.

## Priority Framework

With limited time, a structured priority framework ensures effort is directed where it has the most impact. Work through the priorities in order.

**Priority 1: Core Functionality (Must Have).** The chatbot must work end-to-end. A user sends a message, the system retrieves relevant documents, generates a response grounded in those documents, and returns it. Memory persists across conversation turns. Basic error handling prevents crashes. The goal is a functional demo — if this priority is not complete, nothing else matters.

**Priority 2: Production Readiness (Should Have).** Authentication protects the API. Content moderation filters unsafe input and output. Structured logging captures request data for debugging. The application runs in Docker and is deployed to a cloud provider. The goal is a production deployment — the system is not just a local prototype.

**Priority 3: Polish (Nice to Have).** Advanced RAG features like re-ranking and hybrid search improve retrieval quality. An analytics dashboard visualises engagement and performance. A/B testing infrastructure supports experimentation. Knowledge graph integration adds structured reasoning. The goal is impressive features — these differentiate your project but are not essential.

**Priority 4: Documentation (Important).** Documentation is complete and professional. An architecture diagram visualises the system design. The README includes setup instructions, usage examples, and API documentation. Architecture Decision Records explain key technical choices. The goal is professional handoff — your project is only as valuable as someone else's ability to understand and run it.

Priority 1 issues must be resolved before moving to Priority 2. A working but unpolished system is far more useful than a broken system with advanced features. Priority 4 (documentation) should receive dedicated time regardless of how far you get on the other priorities.

## Troubleshooting Common Integration Issues

Integration is where individually working components reveal their incompatibilities. The following five issues are the most common failures encountered during capstone integration.

### RAG Not Retrieving Relevant Documents

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

### Context Window Overflow

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

### Slow Response Times

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

### Docker Container Won't Start

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

### Frontend Can't Connect to Backend

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
