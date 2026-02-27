# Chatbot Development Course

Hands-on training in building production-ready chatbots using Large Language Models (LLMs). Participants progress from fundamental concepts through advanced techniques including Retrieval-Augmented Generation (RAG), memory management, deployment, analytics, and production hardening.

**Duration:** 20–25 hours across 8 modules
**Level:** Intermediate
**Delivery:** One module per week over 8 weeks

## Prerequisites

### Required Technical Skills

- **Python Programming:** Intermediate proficiency including classes, functions, error handling, and working with APIs
- **Command Line:** Comfortable with terminal/shell operations, environment variables, and package management (pip)
- **Git & Version Control:** Basic understanding of repositories, cloning, and commits
- **JSON & REST APIs:** Experience making HTTP requests and parsing JSON responses

### Recommended Background Knowledge

- Basic understanding of machine learning concepts (helpful but not required)
- Familiarity with natural language processing terminology
- Experience with any API/backend frameworks (Flask, FastAPI, Django)
- Prior exposure to chatbot concepts or conversational AI (helpful but not required)

## Technical Requirements

### Hardware

| Component | Minimum Requirement |
|-----------|-------------------|
| Computer | Modern laptop or desktop (Windows, macOS, or Linux) |
| RAM | 8 GB minimum (16 GB recommended for RAG modules) |
| Storage | 10 GB free disk space for dependencies and vector databases |
| Internet | Stable broadband connection for API calls |

### Software

- Python 3.9 or higher (3.10+ recommended)
- Code editor: VS Code, PyCharm, or equivalent IDE with Python support
- Terminal: Command Prompt, PowerShell, Terminal, or iTerm2
- Git: latest version for cloning the course repository
- Web browser: Chrome, Firefox, or Edge (for API dashboard access)

### Core Python Dependencies

| Package | Purpose |
|---------|---------|
| `openai` | OpenAI API client for GPT models |
| `tiktoken` | Token counting and management |
| `chromadb` | Local vector database for RAG |
| `sentence-transformers` | Embedding models for semantic search |
| `langchain` | LLM application framework |
| `pandas` | Data manipulation and analysis |
| `python-dotenv` | Environment variable management |

Additional packages (FastAPI, Streamlit, Redis, Docker, etc.) are introduced in later modules as needed.

## Pre-Course Setup

Complete the following before the first session:

1. Install Python 3.9+ and verify: `python --version`
2. Create and activate a virtual environment
3. Register for OpenAI API and generate an API key ([Oxford API access](https://oerc.ox.ac.uk/ai-centre/generative-ai-tools/api-access))
4. Set the `OPENAI_API_KEY` environment variable
5. Install core packages: `pip install openai tiktoken python-dotenv`
6. Clone this repository: `git clone https://github.com/OxfordCompetencyCenters/aicc_chatbot_modules.git`

## Module Overview

### Module 1: Chatbot Foundations & API Integration
**Duration:** 3 hours

Evolution from rule-based to LLM-powered chatbots. LLM fundamentals (tokens, context windows, parameters). OpenAI API integration and best practices. Stateless vs stateful chatbot design. System prompts and conversation management.

### Module 2: RAG — Vector Embeddings & Semantic Search
**Duration:** 4 hours

Retrieval-Augmented Generation paradigm. Vector embeddings and semantic similarity. Document chunking strategies (fixed, semantic, recursive). Vector databases (ChromaDB, Pinecone). Building end-to-end RAG chatbots with source citations.

### Module 3: Advanced RAG & Knowledge Graph Integration
**Duration:** 3.5 hours

Agentic RAG with multi-step retrieval and query expansion. Cross-encoder re-ranking for precision. Hybrid search (dense + sparse retrieval). Knowledge graphs for structured reasoning. Semantic caching for cost reduction. Retrieval quality metrics and evaluation.

### Module 4: Memory & Context Management
**Duration:** 3.5 hours

Short-term vs long-term memory architectures. Conversation window management strategies. Conversation summarisation techniques. Retrieval-based memory systems. Selective memory and importance scoring. User profile building and personalisation.

### Module 5: Deployment Architecture
**Duration:** 3 hours

RESTful API design with FastAPI. Frontend development (Streamlit, React). Docker containerisation and multi-stage builds. Docker Compose for local development. Cloud deployment (Google Cloud Run). JWT authentication and secrets management.

### Module 6: Chatbot Analytics & Observability
**Duration:** 3.5 hours

User engagement metrics (DAU, retention, sessions). Conversation quality analytics. Performance monitoring (latency, cost tracking). Structured logging and distributed tracing with OpenTelemetry. A/B testing and experimentation.

### Module 7: Production Best Practices & Advanced Topics
**Duration:** 2.5 hours

Production prompt engineering patterns. Content moderation and safety (OpenAI Moderation API). PII detection and redaction. Cost optimisation strategies. Horizontal scaling with Redis and load balancing. Rate limiting and backpressure. Alternative LLM providers (Anthropic, open-source). Production deployment checklist.

### Module 8: Capstone Project Integration
**Duration:** 2.5 hours

Complete system architecture review. Component integration and troubleshooting. Professional documentation (README, Architecture Decision Records). Presentation preparation. Code quality standards. Final deliverables and submission.

## Course Outcomes

Upon completion, participants will be able to:

- Design and implement production-ready chatbot systems
- Integrate LLMs with custom knowledge bases using RAG
- Optimise costs and performance at scale
- Deploy chatbots to cloud platforms with Docker
- Implement monitoring and observability
- Apply safety, security, and compliance best practices
- Make data-driven improvements through analytics and A/B testing

## Technologies Covered

**Core:** Python 3.10+, OpenAI API, FastAPI, ChromaDB, Docker
**Frontend:** Streamlit, React
**Data & Caching:** PostgreSQL, Redis, Pinecone
**Observability:** OpenTelemetry, Jaeger, structured JSON logging
**ML/NLP:** GPT-3.5/GPT-4, Anthropic Claude, sentence-transformers, tiktoken
**Cloud & DevOps:** Google Cloud Run, Docker Compose, NGINX

## Repository Structure

```
aicc_chatbot_modules/
├── module_1/          # Chatbot Foundations & API Integration
├── module_2/          # RAG — Vector Embeddings & Semantic Search
├── module_3/          # Advanced RAG & Knowledge Graph Integration
├── module_4/          # Memory & Context Management
├── module_5/          # Deployment Architecture
├── module_6/          # Chatbot Analytics & Observability
├── module_7/          # Production Best Practices & Advanced Topics
├── module_8/          # Capstone Project Integration
└── README.md
```

Each module directory contains numbered markdown files covering individual topics, with YAML frontmatter specifying dependencies, tags, and learning outcomes.