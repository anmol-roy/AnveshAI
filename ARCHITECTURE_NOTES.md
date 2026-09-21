# Application Architecture Notes

## 1. Overview

This project is a multi-layer AI application for intellectual-property guidance, with a Next.js frontend and a Python backend.

The system is designed around a RAG (Retrieval-Augmented Generation) architecture, with:
- document ingestion
- vector search
- hybrid retrieval
- formulation classification
- legal / patent / TK evidence retrieval
- knowledge graph enrichment
- final answer generation with citations

---

## 2. High-Level Architecture

The system combines:
- Frontend UI for user interaction
- Backend API layer for orchestration
- Retrieval pipeline for legal and technical knowledge
- Knowledge graph for relationships among formulations, laws, and traditional knowledge
- LLM-based reasoning for evidence-grounded answers

The conceptual flow is:

1. User asks a question
2. Query is processed and classified
3. Relevant sources are retrieved
4. Evidence is fused and ranked
5. Knowledge graph is enriched
6. Final answer is generated with citations and risk indicators
7. Human escalation may happen when confidence is low or the issue is complex

---

## 3. Frontend Stack

The frontend is located in the `client` folder and uses:

- Next.js 16.3.3
- React 19
- TypeScript 5.7.3
- Tailwind CSS 4.3.3
- PostCSS
- Clerk authentication
- Vercel Analytics
- Lucide React icons
- shadcn-style component structure

Main frontend files include:
- `client/package.json`
- `client/app/layout.tsx`
- `client/next.config.mjs`
- `client/components/...`

The frontend is mainly a user-facing application shell and UI for interacting with the backend.

---

## 4. Frontend-to-Backend Connection

The frontend and backend are designed to be connected through an HTTP API layer.

### Backend API entry point
The Python backend exposes FastAPI routes from:
- `server/src/api/main.py`

The app is intended to run with Uvicorn, for example:
- `uvicorn src.api.main:app --reload --host 0.0.0.0 --port 8000`

### Main API flow
The frontend should call the backend using HTTP requests to the API endpoints defined in the FastAPI app, such as:
- `GET /`
- `GET /health`
- `POST /query`
- `POST /ask`
- `POST /analyze-invention`
- `POST /patentability-check`
- `POST /jurisdictional-query`

These routes are defined in `server/src/api/main.py` and are the main integration points for the client UI.

### Typical frontend-backend interaction model
1. User types a question in the Next.js app
2. Frontend sends a request to the backend
3. Backend runs query processing and retrieval
4. Backend returns structured JSON response
5. Frontend renders the answer, citations, risk indicators, and follow-up steps

### Recommended integration pattern
For a clean connection, the client should:
- keep API URLs in environment variables
- use a centralized API client or fetch wrapper
- separate UI state from backend response parsing
- handle loading, error, and empty-result states
- map backend response fields into frontend components

### Environment variables to define
The frontend will likely need values such as:
- `NEXT_PUBLIC_API_BASE_URL`
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`

The backend may also use runtime environment variables for model providers and credentials such as:
- `GROQ_API_KEY`
- any other service-specific secrets

---

## 5. Backend Stack

The backend is located in the `server` folder and uses:

### Core runtime
- Python
- FastAPI
- Uvicorn
- Pydantic
- python-dotenv

### AI / retrieval stack
- LangChain
- LangChain Community
- LangChain Core
- LangChain Text Splitters
- LangChain Qdrant
- LangChain MistralAI
- Mistral AI models (`mistral-large-latest`)
- sentence-transformers
- rank-bm25
- Qdrant vector DB
- scikit-learn / lightweight ML helpers where needed

### Data ingestion / document processing
- pypdf
- PyMuPDF / document parsing utilities
- JSON and structured corpus ingestion pipelines

### Multilingual support
- langdetect
- Bhashini / ULCA translation integration
- Indian-language translation pipeline support
- language routing and multilingual answer generation

### External services / infrastructure
- Clerk for frontend auth (client-side)
- Vercel Analytics / hosting for frontend
- Qdrant local/vector persistence for retrieval and knowledge indexing
- Mistral API for LLM inference
- Bhashini translation APIs for Indian language workflows

Main backend files include:
- `server/src/api/main.py`
- `server/src/ingest.py`
- `server/src/retrieval/retriever.py`
- `server/src/routing/orchestrator.py`
- `server/src/agents/orchestrator.py`
- `server/src/graph/graph_store.py`
- `server/src/multilingual/bhashini.py`

---

## 5. Actual App Flow

This app does not only do one-side retrieval. It has both:

### A. Ingestion / Indexing Side
This is the data preparation layer:
- raw sources are loaded
- documents are parsed
- text is chunked
- embeddings are generated
- embeddings and metadata are indexed in Qdrant
- BM25 indexes are built
- a knowledge graph is created and persisted

Evidence of this:
- `server/src/ingest.py`
- `server/src/qdrant_singleton.py`
- `server/src/graph/graph_store.py`
- `server/src/graph/graph_builder.py`

### B. Query / RAG Side
This is the runtime response layer:
- user query is received
- query is classified by intent and domain
- relevant sources are retrieved
- legal / patent / TK / ABS / international evidence is collected
- evidence is ranked and fused
- LLM produces a grounded answer with citations

Evidence of this:
- `server/src/api/main.py`
- `server/src/routing/orchestrator.py`
- `server/src/retrieval/retriever.py`
- `server/src/agents/orchestrator.py`
- `server/src/evidence/fusion.py`
- `server/src/generation/report_generator.py`

---

## 6. Knowledge Graph Implementation

The app contains a custom in-memory knowledge graph implementation rather than a full Neo4j-based graph database.

Files:
- `server/src/graph/graph_store.py`
- `server/src/graph/graph_builder.py`

The graph stores:
- nodes such as formulation, legal section, patent, traditional knowledge, jurisdiction
- relationships such as:
  - contains
  - appears_in
  - documented_in
  - relevant_to
  - belongs_to
  - has_jurisdiction

This graph is used to connect related entities and improve contextual reasoning.

---

## 7. Multi-Agent / Tool-Orchestration Design

The architecture is closer to tool orchestration than fully autonomous independent agents.

The main orchestrator is:
- `IPSaktiOrchestrator`

The orchestrator calls tools such as:
- `legal_search`
- `patent_search`
- `tk_search`
- `formulation_classifier`
- `abs_check`
- `international_search`

This is defined in:
- `server/src/agents/orchestrator.py`

This is a strong modular design, but not a fully distributed multi-agent system with independent agent workers and asynchronous role-based collaboration.

---

## 8. What the Architecture Gets Right

The architecture correctly includes:
- user query intake
- domain routing
- formulation classification
- vector/hybrid retrieval
- legal + TK + patent retrieval
- evidence fusion
- citation-based answer generation
- graph-based relationship modeling
- system-level orchestration

This is a valid RAG + knowledge-graph architecture for the current app.

---

## 9. What Is Missing or Needs Enhancement

The main gap is the feedback / learning loop.

### Missing or weak areas
- human feedback capture
- expert correction workflow
- automatic quality improvement after user feedback
- retraining or prompt tuning loop
- formal evaluation dashboard
- more explicit multi-agent role division

The app has escalation and audit modules, but they are not yet a complete closed-loop learning system.

Files related to this:
- `server/src/escalation/facilitator.py`
- `server/src/audit/logger.py`

---

## 10. Architecture Verdict

### Current status
The app architecture is mostly correct and matches the intended system design.

### Best description
This is a:
- RAG-based AI system
- knowledge-graph enriched backend
- tool-orchestrated legal/technical analysis engine
- frontend dashboard for user interaction

### Needed enhancement to be more complete
Add a dedicated feedback loop:
- Human review
- Correction capture
- Quality scoring
- Knowledge refresh
- Retrieval and prompt refinement

---

## 11. Final Conclusion

The architecture is broadly valid for the current implementation, but it is still incomplete if the goal is a production-ready, continuously learning AI system.

The strongest version of the architecture should include:
- ingestion + indexing
- retrieval + reasoning
- knowledge graph
- response generation
- human escalation/feedback loop

This makes it complete and realistic for a next-stage product architecture.
