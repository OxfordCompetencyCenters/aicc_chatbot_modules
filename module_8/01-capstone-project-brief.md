---
name: Capstone Project Brief
dependsOn:
  - alternative-llm-providers
tags: [chatbots, capstone, project, architecture, evaluation]
learningOutcomes:
  - Define a domain-specific chatbot project with a clear goal, objectives, and methodology.
  - Build and integrate a production-ready chatbot following a phased methodology from data preparation through deployment.
  - Design and execute experiments that evaluate retrieval quality, response accuracy, robustness, and performance.
  - Analyse and report results with quantitative metrics, and reflect on lessons learned.
---

## Learning Outcomes

- Define a domain-specific chatbot project with a clear goal, objectives, and methodology.
- Build and integrate a production-ready chatbot following a phased methodology from data preparation through deployment.
- Design and execute experiments that evaluate retrieval quality, response accuracy, robustness, and performance.
- Analyse and report results with quantitative metrics, and reflect on lessons learned.

The capstone project is a structured engineering project, not a tutorial exercise. Over Modules 1–7, you built individual components — LLM integration, RAG pipelines, memory management, deployment infrastructure, analytics, and production hardening. The capstone brings them together into a single system that solves a real information-retrieval problem for a specific audience. This section defines the project, provides a structured process for executing it, and explains how to evaluate and report your results.

## Project Goal

Build and deploy a **domain-specific chatbot** that retrieves information from a curated document corpus, maintains conversation context across turns, and operates with production-grade reliability, safety, and observability.

The emphasis on "domain-specific" is deliberate. A generic chatbot that wraps an LLM API adds little value — users can already access ChatGPT or Claude directly. A domain-specific chatbot adds value by grounding its responses in a particular corpus of documents that the base LLM does not have access to, applying domain-appropriate constraints (what to answer, what to refuse, how to handle ambiguity), and serving a defined audience with specific information needs.

For example, a legal research assistant answers questions grounded in case law documents. A company onboarding bot answers new-hire questions from HR policies and IT setup guides. A medical literature chatbot synthesises findings from PubMed papers. Each solves a problem that a general-purpose LLM cannot solve as well, because the chatbot has access to specialised documents and is tuned for the domain.

Your project should identify a domain, a target audience, and a document corpus, then build a chatbot that serves that audience using that corpus.

## Objectives

The capstone must satisfy the following objectives. Each is testable — you should be able to demonstrate it working.

1. **Retrieval-augmented generation.** The chatbot retrieves relevant documents from a vector database and grounds its responses in those documents. Responses cite or reference the source material rather than relying solely on the LLM's training data.

2. **Conversation memory.** The chatbot maintains context across multiple turns within a conversation. A follow-up question like "tell me more about that" correctly refers to the previous response. Memory management prevents token overflow in long conversations.

3. **Deployed web application.** The chatbot is accessible via a web interface (Streamlit or React frontend) backed by a FastAPI API, containerised with Docker, and deployed to a cloud provider with a public URL.

4. **Error handling and safety.** The system handles invalid inputs, API failures, and rate limiting gracefully. Content moderation filters unsafe inputs and outputs. The chatbot does not crash, hang, or return unhelpful error messages under normal or adversarial use.

5. **Performance and cost tracking.** The system tracks response latency, token usage, and cost per conversation. These metrics are accessible via a metrics endpoint or dashboard.

6. **Documentation.** The project includes a README with setup instructions and usage examples, architecture decision records for key technical choices, and an architecture diagram showing how components connect.

## Methodology

The project follows a four-phase methodology. Each phase builds on the previous one, and the integration checklist at the end of this section serves as a quality gate to verify completeness before moving forward.

### Phase 1: Domain and Data

Choose a domain and assemble a document corpus. A good domain choice has three properties: the documents are available (you can collect or access them), the questions are non-trivial (they require searching across multiple documents, not just looking up a single fact), and the audience is defined (you can describe who would use this chatbot and what they need).

Prepare the corpus for RAG ingestion. Chunk documents using the strategies from Module 2 — experiment with chunk sizes (500, 1,000, 1,500 tokens) to find the best trade-off between granularity and context. Generate embeddings and store them in a vector database (ChromaDB for development, Pinecone if scale requires it). Verify that similarity search returns relevant results for a sample of test queries before proceeding to the next phase.

### Phase 2: Core Pipeline

Build the end-to-end chat flow: the user sends a message, the RAG engine retrieves relevant documents, the prompt is constructed with the retrieved context and conversation history, the LLM generates a response, and the response is returned to the user. This is the minimum viable product.

The target architecture for the complete system is shown below. Not every component is needed in Phase 2 — focus on the core pipeline (backend, RAG engine, memory manager, vector database, and LLM integration). The remaining layers are added in Phases 3 and 4.

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
│  ┌──────────────────────────────────────────────────┐    │
│  │           FastAPI Application Servers             │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │    │
│  │  │ Server 1 │  │ Server 2 │  │ Server 3 │      │    │
│  │  └─────┬────┘  └─────┬────┘  └─────┬────┘      │    │
│  └────────┼─────────────┼─────────────┼────────────┘    │
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
│  │    API     │  │          │  │          │             │
│  └────────────┘  └──────────┘  └──────────┘             │
└──────────────────────────────────────────────────────────┘
```

The **frontend layer** provides the user interface. The **API gateway** handles load balancing, rate limiting, and authentication. The **backend layer** contains stateless FastAPI servers coordinating three core services: the chatbot core (conversation orchestration), the RAG engine (document retrieval), and the memory manager (history and summarisation). The **data layer** provides persistence: PostgreSQL for user data, Redis for caching and session state, and ChromaDB for vector embeddings. The **external services layer** includes the LLM provider, analytics, and log management.

### Phase 3: Production Hardening

Add the layers that make the system production-ready. Implement JWT authentication to protect the API. Add content moderation (OpenAI Moderation API) on both inputs and outputs. Implement structured logging with trace IDs so that requests can be traced through the system. Add rate limiting to prevent abuse. Set `max_tokens` on API calls and implement cost tracking. Containerise the application with Docker and docker-compose.

### Phase 4: Deployment and Monitoring

Deploy to a cloud provider (Google Cloud Run, AWS, or equivalent) with a public URL. Add a health check endpoint. Configure metrics tracking for latency, cost, and error rate. Verify that the deployed version behaves identically to the local version — environment differences (missing variables, different URLs, network configuration) are the most common source of deployment failures.

### Integration Checklist

Use this checklist as a quality gate to verify completeness at the end of each phase.

**Core Functionality (Phase 2):**
- Chat API endpoint accepts messages and returns responses
- RAG retrieval fetches relevant documents for a given query
- LLM generation creates responses grounded in retrieved context
- Conversation history is maintained across turns
- Memory management prevents token overflow

**Data Flow (Phase 2):**
- User messages flow through the full pipeline: API → RAG retrieval → LLM generation → response
- Retrieved documents are properly formatted and injected into the LLM prompt
- Conversation history is stored and retrieved correctly between requests

**Error Handling and Security (Phase 3):**
- Invalid inputs return appropriate HTTP error codes and messages
- API failures are caught and handled gracefully with fallback responses
- Authentication is required for protected endpoints
- Content moderation is active on both inputs and outputs
- API keys are stored securely in environment variables

**Deployment and Observability (Phase 4):**
- The application is Dockerised and runs via docker-compose
- Deployed to a cloud provider with a public URL
- Health check endpoint responds to monitoring probes
- Logging uses structured format (JSON)
- Key metrics are tracked: latency, cost, error rate

### Priority Framework

When time is limited, work through priorities in order. Priority 1 must be complete before moving to Priority 2.

**Priority 1: Core Functionality (Must Have).** End-to-end chat with RAG and memory. Basic error handling. This is Phases 1–2.

**Priority 2: Production Readiness (Should Have).** Authentication, moderation, logging, Docker, cloud deployment. This is Phases 3–4.

**Priority 3: Polish (Nice to Have).** Advanced RAG (re-ranking, hybrid search), analytics dashboard, A/B testing, semantic caching.

**Priority 4: Documentation (Important).** README, ADRs, architecture diagram. Allocate dedicated time for this regardless of progress on other priorities.

## Experiments

Once the system is built, evaluate it systematically. The experiments below assess whether the system meets its objectives.

### Retrieval Quality

Prepare a set of 10–20 test queries with known answers that exist in your document corpus. For each query, record which documents the RAG engine retrieves and whether the correct document appears in the top-k results.

```python
test_queries = [
    {"query": "What is the refund policy?", "expected_doc": "returns-policy.md"},
    {"query": "How do I set up two-factor authentication?", "expected_doc": "security-guide.md"},
    # ... more test queries
]

results = []
for test in test_queries:
    retrieved = rag_engine.retrieve(test["query"], top_k=5)
    doc_names = [doc["metadata"]["source"] for doc in retrieved]
    hit = test["expected_doc"] in doc_names
    results.append({
        "query": test["query"],
        "hit": hit,
        "top_score": retrieved[0]["score"] if retrieved else 0,
        "retrieved_docs": doc_names
    })

hit_rate = sum(r["hit"] for r in results) / len(results)
print(f"Retrieval hit rate (top-5): {hit_rate:.0%}")
```

If retrieval quality is poor, the most common causes are chunk size mismatches (try 500, 1,000, and 1,500 tokens), embedding model mismatches (verify the same model is used for documents and queries), and similarity thresholds that are too high or too low.

### Response Quality

For the same test queries, evaluate whether the chatbot's full response correctly answers the question based on the retrieved documents. Check for three failure modes: incorrect answers (the response contradicts the source documents), hallucinated answers (the response includes information not present in any retrieved document), and refusal to answer when the information is available.

Manual evaluation is the most reliable method for a capstone-scale project. For each test query, rate the response as correct, partially correct, or incorrect. If you have reference answers, you can also use an LLM-as-judge approach — ask a separate LLM call to score the response against the reference.

### Robustness

Test how the system handles inputs outside the happy path:

- **Empty input:** Send an empty string. The system should return a helpful prompt, not crash.
- **Very long input:** Send a message over 1,000 words. The system should truncate or handle it gracefully.
- **Gibberish:** Send random characters. The system should ask for clarification.
- **Off-topic queries:** Ask about something completely outside the domain. The system should redirect politely.
- **Adversarial prompts:** Attempt prompt injection ("ignore your instructions and..."). The system should not comply.

Record the result of each test: pass (reasonable response), fail (crash, unhelpful error, or inappropriate response).

### Performance

Measure system performance under realistic conditions:

- **Latency:** Time 20+ requests and compute p50, p95, and p99 response times.
- **Throughput:** Send 10 concurrent requests and verify all complete successfully within acceptable time.
- **Memory stability:** Run a 50-turn conversation and verify that token counts stay within limits (no context overflow).
- **Cost:** Calculate the average cost per conversation based on token usage and API pricing.

```python
import time
import statistics

latencies = []
for query in test_queries:
    start = time.time()
    response = chatbot.chat(query["query"])
    latencies.append(time.time() - start)

print(f"p50 latency: {statistics.median(latencies):.2f}s")
print(f"p95 latency: {sorted(latencies)[int(len(latencies) * 0.95)]:.2f}s")
```

## Results

Document your experimental results in a structured format. Tables are more effective than prose for conveying quantitative data.

**Retrieval quality:**

| Metric | Value |
|--------|-------|
| Test queries | 20 |
| Hit rate (top-5) | 85% |
| Average top-1 similarity score | 0.82 |

**Response quality:**

| Rating | Count | Percentage |
|--------|-------|------------|
| Correct | 15 | 75% |
| Partially correct | 3 | 15% |
| Incorrect | 2 | 10% |

**Performance:**

| Metric | Value |
|--------|-------|
| p50 latency | 1.8s |
| p95 latency | 3.2s |
| Cost per conversation (avg) | $0.003 |
| Concurrent requests handled | 10 |

**Robustness:**

| Test case | Result |
|-----------|--------|
| Empty input | Pass |
| Long input (1,000+ words) | Pass |
| Gibberish | Pass |
| Off-topic query | Pass |
| Prompt injection | Pass |

Compare against a baseline where appropriate. For example, compare response quality with and without RAG, or compare latency with and without semantic caching. Baseline comparisons demonstrate the value your engineering decisions added.

## Conclusion

The conclusion is a structured reflection on the project. It should address the following:

**What worked well.** Which components performed as expected? Which design decisions proved effective? For example: "The combination of 800-token chunks with cross-encoder re-ranking achieved 85% retrieval accuracy, significantly better than the 60% achieved with default settings."

**What didn't work.** Be specific about failures and their root causes. "The initial summarisation strategy lost important context from early conversation turns. Switching to a hybrid approach (recent turns in full + summary of older turns) resolved this."

**Key technical challenges.** Describe 2–3 significant challenges and how they were resolved. These demonstrate problem-solving ability and depth of understanding.

**Lessons learned.** What would you do differently if starting over? What surprised you about building an LLM application in practice versus in theory?

**Future improvements.** List 3–5 concrete, prioritised improvements. For example: fine-tune a smaller model on domain data to reduce cost and improve accuracy; add function calling so the chatbot can take actions (not just answer questions); implement multi-modal support for image and PDF uploads.
