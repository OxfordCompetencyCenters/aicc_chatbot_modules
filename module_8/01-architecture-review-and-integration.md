---
name: Architecture Review and Integration
dependsOn:
  - alternative-llm-providers
tags: [chatbots, architecture, integration, capstone]
learningOutcomes:
  - Map the complete reference architecture of a production chatbot system.
  - Evaluate a capstone project against an integration checklist covering functionality, data flow, error handling, performance, security, deployment, and observability.
  - Prioritise remaining implementation work using a structured priority framework.
---

## Learning Outcomes

- Map the complete reference architecture of a production chatbot system.
- Evaluate a capstone project against an integration checklist covering functionality, data flow, error handling, performance, security, deployment, and observability.
- Prioritise remaining implementation work using a structured priority framework.

This is the final module. Over the previous seven modules you built individual components — LLM integration, RAG pipelines, memory management, deployment infrastructure, analytics, and production hardening. Now those components need to work together as a single, cohesive system. Integration is where most projects stumble: each piece works in isolation, but connecting them reveals mismatches in data formats, assumptions about state, and subtle timing issues. This module walks through the full integration process, from architecture review through documentation and presentation.

## Course Journey Recap

Before diving into integration, it is worth stepping back to see the full picture of what you have built across this course.

In Module 1, you established the foundations: connecting to the OpenAI API, managing tokens, and building both stateless and stateful chatbots. Module 2 introduced retrieval-augmented generation — vector embeddings, document chunking, ChromaDB, and semantic search — giving your chatbot access to external knowledge. Module 3 pushed RAG further with query expansion, re-ranking, hybrid search, knowledge graphs, and semantic caching. Module 4 added memory management: conversation history strategies, summarisation, retrieval-based memory, and user profiles.

Module 5 moved from prototype to deployable application: a FastAPI backend, frontend options in Streamlit and React, Docker containerisation, cloud deployment on Google Cloud Run, and JWT authentication. Module 6 introduced observability — engagement metrics, conversation analytics, performance monitoring, structured logging, and A/B testing. Module 7 hardened the system for production with prompt engineering patterns, content moderation, PII protection, cost optimisation, scaling strategies, and multi-provider resilience.

Each module built on the previous one. Your capstone project incorporates skills from all seven modules into a single production-grade system.

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

Not every item needs to be implemented for a successful capstone — the core functionality and error handling sections are essential, while advanced performance and observability features strengthen the project. Use this checklist as both an integration testing plan and a gap analysis tool.

## Priority Framework

With limited time remaining, a structured priority framework ensures effort is directed where it has the most impact. Work through the priorities in order — completing a lower priority before a higher one is a common mistake.

**Priority 1: Core Functionality (Must Have).** The chatbot must work end-to-end. A user sends a message, the system retrieves relevant documents, generates a response grounded in those documents, and returns it. Memory persists across conversation turns. Basic error handling prevents crashes. The goal is a functional demo — if this priority is not complete, nothing else matters.

**Priority 2: Production Readiness (Should Have).** Authentication protects the API. Content moderation filters unsafe input and output. Structured logging captures request data for debugging. The application runs in Docker and is deployed to a cloud provider. The goal is a production deployment — the system is not just a local prototype.

**Priority 3: Polish (Nice to Have).** Advanced RAG features like re-ranking and hybrid search improve retrieval quality. An analytics dashboard visualises engagement and performance. A/B testing infrastructure supports experimentation. Knowledge graph integration adds structured reasoning. The goal is impressive features — these differentiate your project but are not essential.

**Priority 4: Presentation (Important).** Documentation is complete and professional. A demo script is prepared and practised. An architecture diagram visualises the system design. The README includes setup instructions, usage examples, and API documentation. The goal is professional handoff — your project is only as valuable as your ability to explain it.

The key insight is that Priority 1 issues must be resolved before moving to Priority 2, and so on. A working but unpolished demo is far more impressive than a broken system with advanced features. The one exception is Priority 4 (documentation and presentation), which should receive dedicated time regardless of how far you get on the other priorities — a well-presented simple project outperforms a poorly presented complex one.
