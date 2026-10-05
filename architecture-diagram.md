# Architecture Diagram

InsightForge AI is a document knowledge-intelligence platform. It ingests authorized documents, indexes searchable chunks, produces citation-backed answers, and records the evidence and operational details for each AI request.

## Application Architecture

<!-- mermaid-checked: no \n, no em-dash/en-dash, no {} in labels, subgraphs are id["label"], arrows are -->|"label"|, all subgraphs closed by end, ids unique -->
~~~mermaid
flowchart TD
    subgraph Client1["Client Layer"]
        User1["Knowledge Worker"]
        Docs1["Swagger UI"]
    end
    subgraph Edge1["Access Layer"]
        Gateway1["API Gateway"]
        Auth1["Authentication and Access Policy"]
    end
    subgraph Api1["FastAPI Application"]
        DocumentApi1["Document API"]
        SearchApi1["Search API"]
        ChatApi1["Chat and Compare API"]
        AgentApi1["Research Agent API"]
    end
    subgraph Process1["Processing Services"]
        Ingest1["Validation and Ingestion"]
        Chunk1["Chunking Service"]
        Embed1["Embedding Service"]
        Retrieve1["Hybrid Retrieval"]
        Rerank1["Reranking Service"]
        Rag1["RAG Orchestrator"]
        Agent1["Constrained Research Agent"]
    end
    subgraph Store1["Data Stores"]
        Object1[("Document Storage")]
        Meta1[("Metadata Database")]
        Vector1[("Vector Store")]
        Eval1[("Evaluation Data")]
    end
    subgraph External1["External Services"]
        Llm1["LLM Provider"]
        Trace1["Tracing and Metrics"]
    end

    User1 -->|"uses"| Gateway1
    Docs1 -->|"calls"| Gateway1
    Gateway1 -->|"enforces access"| Auth1
    Auth1 -->|"routes request"| DocumentApi1
    Auth1 -->|"routes request"| SearchApi1
    Auth1 -->|"routes request"| ChatApi1
    Auth1 -->|"routes request"| AgentApi1
    DocumentApi1 -->|"uploads"| Ingest1
    Ingest1 -->|"stores original"| Object1
    Ingest1 -->|"stores metadata"| Meta1
    Ingest1 -->|"sends text"| Chunk1
    Chunk1 -->|"creates chunks"| Embed1
    Embed1 -->|"writes vectors"| Vector1
    SearchApi1 -->|"searches"| Retrieve1
    ChatApi1 -->|"retrieves evidence"| Retrieve1
    Retrieve1 -->|"filters vectors"| Vector1
    Retrieve1 -->|"loads metadata"| Meta1
    Retrieve1 -->|"candidates"| Rerank1
    Rerank1 -->|"best chunks"| Rag1
    ChatApi1 -->|"asks answer"| Rag1
    Rag1 -->|"generates answer"| Llm1
    AgentApi1 -->|"runs limited tools"| Agent1
    Agent1 -->|"uses retrieval"| Retrieve1
    Agent1 -->|"uses generation"| Llm1
    Rag1 -->|"records evidence"| Trace1
    Agent1 -->|"records tool trace"| Trace1
    Eval1 -->|"checks quality"| Rag1
~~~

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---:|---|
| Client | Swagger UI initially; optional web client later | TBD | Upload documents and call search, chat, comparison, and agent APIs. |
| Access | API gateway plus application authorization | TBD | Authenticate callers, enforce rate limits, and pass allowed document-access levels to services. |
| Application | Python and FastAPI | Python 3.11+ | Typed REST APIs, request validation, response contracts, and service lifecycle management. |
| Persistence | SQLite locally; PostgreSQL in production | TBD | Store document metadata, ingestion state, access levels, audit information, and workflow records. |
| Document storage | Local storage initially; object storage in production | TBD | Store original uploaded documents separately from searchable representations. |
| Retrieval | FAISS initially; Qdrant or pgvector later | TBD | Store embeddings and return access-filtered semantic-search candidates. |
| AI | Embedding model and LLM provider | TBD | Embed chunks and queries; produce structured analysis, comparisons, and grounded answers. |
| Operations | Structured logs; optional OpenTelemetry, Langfuse, or Phoenix | TBD | Trace retrieval evidence, model versions, token use, cost, latency, failures, and feedback. |
| Delivery | Docker Compose and GitHub Actions | TBD | Reproducible local runtime and automated lint, tests, evaluation, and image-build checks. |

### Data Storage & External Services

Original files live in document storage, while the relational database owns document metadata, content hashes, access levels, ingestion state, and audit records. The vector store contains chunk embeddings plus minimal retrieval metadata; it is not the source of truth for document authorization. The LLM provider is called only after authorized retrieval produces bounded evidence. Evaluation data is versioned separately so retrieval and answer quality can be measured without mixing it into production documents.

### Key Architectural Decisions

- Authorization is enforced before and during retrieval. The client never decides which chunks it may access.
- Ingestion, chunking, embedding, and indexing are distinct stages so errors are visible, reprocessing is idempotent, and stale vectors can be replaced.
- RAG responses require evidence and citations. When evidence is missing or weak, the system returns an explicit insufficient-evidence response rather than fabricating an answer.

## Component Relationships

<!-- mermaid-checked: no \n, no em-dash/en-dash, no {} in labels, subgraphs are id["label"], arrows are -->|"label"|, all subgraphs closed by end, ids unique -->
~~~mermaid
flowchart LR
    subgraph Presentation2["Presentation"]
        cDocRoute["Document Routes"]
        cSearchRoute["Search Routes"]
        cChatRoute["Chat Routes"]
        cAgentRoute["Agent Routes"]
    end
    subgraph CrossCutting2["Cross Cutting"]
        cAuth["Access Context"]
        cValidate["Pydantic Validation"]
        cTrace["Request Tracing"]
        cErrors["Error Handler"]
    end
    subgraph Business2["Business Logic"]
        cIngest["Ingestion Pipeline"]
        cClassifier["Document Classifier"]
        cChunker["Chunking Strategy"]
        cRetriever["Hybrid Retriever"]
        cReranker["Reranker"]
        cRag["RAG Service"]
        cCompare["Comparison Workflow"]
        cAgent["Research Agent"]
    end
    subgraph Data2["Data Access"]
        cDocRepo["Document Repository"]
        cVectorRepo["Vector Store Adapter"]
        cObjectRepo["Object Storage Adapter"]
        cEvalRepo["Evaluation Repository"]
    end
    subgraph Infra2["Infrastructure"]
        cDb[("Metadata Database")]
        cVectors[("Vector Store")]
        cFiles[("Document Storage")]
        cLlm["LLM Client"]
        cEmbed["Embedding Client"]
        cTelemetry["Telemetry Platform"]
    end

    cDocRoute -->|"validates request"| cValidate
    cSearchRoute -->|"validates request"| cValidate
    cChatRoute -->|"validates request"| cValidate
    cAgentRoute -->|"validates request"| cValidate
    cAuth -.->|"authorizes"| cDocRoute
    cAuth -.->|"authorizes"| cSearchRoute
    cAuth -.->|"authorizes"| cChatRoute
    cAuth -.->|"authorizes"| cAgentRoute
    cDocRoute -->|"ingests"| cIngest
    cDocRoute -->|"classifies"| cClassifier
    cSearchRoute -->|"retrieves"| cRetriever
    cChatRoute -->|"answers with evidence"| cRag
    cChatRoute -->|"compares"| cCompare
    cAgentRoute -->|"executes task"| cAgent
    cIngest -->|"writes metadata"| cDocRepo
    cIngest -->|"stores original"| cObjectRepo
    cIngest -->|"chunks text"| cChunker
    cChunker -->|"embeds chunks"| cEmbed
    cChunker -->|"stores vectors"| cVectorRepo
    cRetriever -->|"queries vectors"| cVectorRepo
    cRetriever -->|"loads records"| cDocRepo
    cRetriever -->|"ranks candidates"| cReranker
    cRag -->|"gets evidence"| cRetriever
    cRag -->|"generates grounded output"| cLlm
    cCompare -->|"gets evidence"| cRetriever
    cCompare -->|"creates comparison"| cLlm
    cAgent -->|"uses tools"| cRetriever
    cAgent -->|"uses tools"| cCompare
    cAgent -->|"uses tools"| cRag
    cDocRepo -->|"reads and writes"| cDb
    cVectorRepo -->|"reads and writes"| cVectors
    cObjectRepo -->|"reads and writes"| cFiles
    cEvalRepo -->|"provides cases"| cRag
    cTrace -.->|"observes"| cIngest
    cTrace -.->|"observes"| cRetriever
    cTrace -.->|"observes"| cRag
    cTrace -.->|"observes"| cAgent
    cTrace -->|"exports"| cTelemetry
    cErrors -.->|"formats failures"| cDocRoute
    cErrors -.->|"formats failures"| cSearchRoute
    cErrors -.->|"formats failures"| cChatRoute
    cErrors -.->|"formats failures"| cAgentRoute
~~~

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| Document Routes | Presentation | FastAPI router | Creates, uploads, retrieves, updates, deletes, indexes, classifies, and analyzes documents. |
| Search Routes | Presentation | FastAPI router | Accepts authorized keyword or semantic search queries and returns ranked evidence. |
| Chat Routes | Presentation | FastAPI router | Accepts question and comparison requests and returns citation-backed results. |
| Agent Routes | Presentation | FastAPI router | Starts bounded multi-step research tasks and returns traceable outcomes. |
| Access Context | Cross cutting | Authorization service | Represents the caller and permitted access levels; it is required by every data-access operation. |
| Pydantic Validation | Cross cutting | Schema boundary | Validates untrusted request data and structured LLM output. |
| Request Tracing | Cross cutting | Observability service | Correlates API, retrieval, LLM, tool, cost, latency, and error events. |
| Error Handler | Cross cutting | FastAPI exception layer | Returns explicit safe errors without presenting failures as success. |
| Ingestion Pipeline | Business logic | Orchestrator | Validates files, normalizes text, hashes content, persists state, and triggers chunk/index work. |
| Document Classifier | Business logic | ML inference service | Predicts a document type with a versioned saved model. |
| Chunking Strategy | Business logic | Content processor | Splits documents into stable, metadata-rich chunks while preserving relevant structure. |
| Hybrid Retriever | Business logic | Retrieval service | Combines lexical and vector candidates, filters authorization, and supplies evidence. |
| Reranker | Business logic | Ranking service | Improves precision on a bounded set of retrieved candidates. |
| RAG Service | Business logic | LLM orchestrator | Builds evidence-bound prompts, calls the LLM, validates output, and maps citations. |
| Comparison Workflow | Business logic | Deterministic workflow | Compares authorized evidence from two documents and flags uncertain results. |
| Research Agent | Business logic | Constrained tool workflow | Uses allowlisted tools under call, time, evidence, and permission limits. |
| Document Repository | Data access | Repository | Reads and writes metadata, access levels, ingestion state, and audit data. |
| Vector Store Adapter | Data access | Adapter | Isolates vector-store implementation and supports idempotent index updates. |
| Object Storage Adapter | Data access | Adapter | Stores original documents outside relational metadata. |
| Evaluation Repository | Data access | Dataset provider | Supplies versioned test questions, expected sources, and quality checks. |

## High-Level System Design Quick Reference

<!-- mermaid-checked: no \n, no em-dash/en-dash, no {} in labels, subgraphs are id["label"], arrows are -->|"label"|, all subgraphs closed by end, ids unique -->
~~~mermaid
flowchart TD
    subgraph QuickClient["Client Layer"]
        qUser["Knowledge Worker"]
        qDocs["Swagger UI"]
    end
    subgraph QuickAccess["Access Layer"]
        qGateway["API Gateway"]
        qAuth["Authentication and Access Policy"]
    end
    subgraph QuickApi["FastAPI Application"]
        qDocumentApi["Document API"]
        qSearchApi["Search API"]
        qChatApi["Chat and Compare API"]
        qAgentApi["Research Agent API"]
    end
    subgraph QuickProcessing["AI Processing Services"]
        qIngestion["Validation and Ingestion"]
        qChunking["Chunking Service"]
        qEmbeddings["Embedding Service"]
        qRetrieval["Hybrid Retrieval"]
        qReranking["Reranking Service"]
        qRag["RAG Orchestrator"]
        qAgent["Constrained Research Agent"]
    end
    subgraph QuickStorage["Data Stores"]
        qDocuments[("Original Document Storage")]
        qMetadata[("Metadata Database")]
        qVectors[("Vector Store")]
        qEvaluation[("Evaluation Dataset")]
    end
    subgraph QuickExternal["External Services"]
        qLlm["LLM Provider"]
        qTelemetry["Tracing and Metrics"]
    end

    qUser -->|"uses"| qGateway
    qDocs -->|"calls"| qGateway
    qGateway -->|"enforces access"| qAuth
    qAuth -->|"routes request"| qDocumentApi
    qAuth -->|"routes request"| qSearchApi
    qAuth -->|"routes request"| qChatApi
    qAuth -->|"routes request"| qAgentApi
    qDocumentApi -->|"uploads"| qIngestion
    qIngestion -->|"stores original"| qDocuments
    qIngestion -->|"stores metadata"| qMetadata
    qIngestion -->|"sends text"| qChunking
    qChunking -->|"creates chunks"| qEmbeddings
    qEmbeddings -->|"writes vectors"| qVectors
    qSearchApi -->|"searches"| qRetrieval
    qChatApi -->|"retrieves evidence"| qRetrieval
    qRetrieval -->|"filters vectors"| qVectors
    qRetrieval -->|"loads metadata"| qMetadata
    qRetrieval -->|"candidates"| qReranking
    qReranking -->|"best chunks"| qRag
    qChatApi -->|"asks answer"| qRag
    qRag -->|"generates answer"| qLlm
    qAgentApi -->|"runs limited tools"| qAgent
    qAgent -->|"uses retrieval"| qRetrieval
    qAgent -->|"uses generation"| qLlm
    qRag -->|"records evidence"| qTelemetry
    qAgent -->|"records tool trace"| qTelemetry
    qEvaluation -->|"checks quality"| qRag
~~~

### Key Flows

1. **Document ingestion:** A user uploads a document, the system validates and stores it, then chunks, embeds, and indexes its content.
2. **Search:** An authorized request uses hybrid keyword and vector retrieval, then reranks the best evidence.
3. **RAG chat:** The system answers only from authorized retrieved chunks and returns source citations.
4. **Research agent:** The agent can use only allowed tools, and every tool call is authorized and traced.
5. **Evaluation and monitoring:** Evaluation cases measure retrieval and answer quality; tracing records latency, errors, model usage, and cost.


-------------
RAG explained - 

User asks a question
        |
        v
Find relevant document paragraphs
        |
        v
Give those paragraphs to the LLM
        |
        v
LLM writes a cited answer
        |
        v
User receives an evidence-based response