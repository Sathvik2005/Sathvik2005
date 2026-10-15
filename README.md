# Kanithi Satya Sathvik

Building software at the intersection of AI systems, backend engineering, and developer tooling.

I work on systems where the boundary between probabilistic model output and deterministic application behaviour matters — evidence grounding, structured validation, human-in-the-loop controls, and testable AI pipelines. I'm a final-year B.Tech student in Computer Science (AI & ML) at VIT-AP University.

---

## About

My engineering interests centre on problems where AI alone is not enough: where you need explicit contracts between the model and the rest of the system, where incorrect output must be caught rather than propagated, and where a reviewer needs to trace exactly why the system reached a conclusion.

In practice this means:

- LLM applications with retrieval, structured outputs, and deterministic validation layers
- Backend systems and APIs where persistence and failure behaviour are tested explicitly
- Agent workflows with observable state, bounded retries, and human approval gates
- Full-stack products where the AI component is one audited layer, not the whole architecture

I'm looking for roles in software development engineering, AI/LLM engineering, backend and full-stack engineering, and AI evaluation or QA.

---

## Technical Focus

**Languages** — Python, TypeScript, SQL  
**Backend** — FastAPI, Node.js / Express, PostgreSQL, SQLAlchemy / Alembic, psycopg3  
**AI & LLM** — LangGraph, Anthropic SDK, Amazon Bedrock Converse API, Tavily, pgvector, Titan Text Embeddings V2  
**Frontend** — React 19, Next.js, Vite, strict TypeScript  
**Testing** — pytest, Vitest, React Testing Library, embedded PostgreSQL (pytest-postgresql), chdb  
**Infrastructure** — Vercel (Python Functions + static), Docker Compose, Cloud Run  
**Data** — ClickHouse, pgvector / HNSW indexing  

---

## Selected Projects

### [EvidenceOS](https://github.com/Sathvik2005/evidenceos)
*A claim-research system that enforces provenance by construction and uses deterministic rules to override model output.*

**Stack:** Python 3.11 · FastAPI · LangGraph · Anthropic · Tavily · PostgreSQL · React 19 · Vite · TypeScript · pytest · Vitest

- A five-node LangGraph workflow (Claim Decomposer → Research Agent → Deterministic Validation → Evidence Analyst → Evaluator) where the validation step applies hard rules that override the evaluator's score — the LLM cannot promote a claim past a `CONFLICTING` state on its own authority.
- Provenance is enforced at retrieval: the model may only quote verbatim text that was actually retrieved; it cannot supply URLs, titles, or source dates. Any citation referencing an ID outside the retrieved set is rejected before it reaches persistence.
- Append-only claim history is enforced by PostgreSQL database triggers — state transitions cannot be silently overwritten in application code.
- 171 backend tests run against a real embedded PostgreSQL 16 instance; 26 UI tests via Vitest + Testing Library; 10 failure-injection cases covering partial failures, retries, and `REVIEW_REQUIRED` outcomes.

> **Status:** Experimental MVP. Full automated test suite and seeded demo pass locally. The Python backend has not yet been run against a hosted PostgreSQL or deployed to Vercel. Live-provider smoke test is the next step. No accounts or rate limits yet.

---

### [MemoryOps for Schools](https://github.com/Sathvik2005/memoryops-for-schools)
*A persistent-memory AI assistant for school clubs and councils, built on pgvector and Amazon Bedrock with a grounding step that strips model-hallucinated evidence.*

**Stack:** TypeScript · Node.js · Express · PostgreSQL + pgvector · Amazon Bedrock Converse API · Titan Text Embeddings V2 · React · Vitest

- Five memory types (Episodic, Semantic, Procedural, Decision, Task) with seven relationship types (`CAUSED_BY`, `RESOLVED_BY`, `SIMILAR_TO`, `SUPERSEDES`, `CONFIRMS`, `CONTRADICTS`, `GENERATED_TEST`), each memory carrying `confidence`, `verified`, `last_verified_at`, and `source_type/source_id` provenance.
- Retrieval is a weighted blend of similarity (0.6) + confidence (0.2) + verified (0.1) + recency (0.1), computed in code (`src/memory.ts`, `rankScore`). Memories flagged `SUPERSEDES` are excluded from evidence; `CONTRADICTS` pairs are surfaced as explicit caveats rather than resolved silently.
- `groundAnalysis()` in `src/agent.ts` runs after every Bedrock call: any record ID referenced in the model's output that was not in the retrieved set is stripped out in code, and reliability/staleness/contradiction caveats are computed deterministically — never left to the model to self-report. Unit-tested directly with analyses that reference IDs that were never retrieved.
- Human approval is required before any action that mutates state (`create_task`); every action is logged to `agent_actions` with its approval state.

> **Status:** Working. End-to-end test seeds `INC-005`, submits a demo issue, and asserts against actual database rows — not UI text — then runs a second independent `searchMemory()` call to verify persistence. Offline/hash providers allow the full test suite to run without AWS credentials.

---

### [CareLoad](https://github.com/Sathvik2005/careload)
*A healthcare workload and patient check-in platform with an evidence-backed AI friction classifier and role-separated access control.*

**Stack:** Python · FastAPI · SQLAlchemy · Alembic · Next.js 16 · TypeScript · Tailwind · React Query · Zod · pytest · Vitest · Docker Compose · Vercel

- An AI analysis pipeline (`AIProvider` interface) classifies patient-reported friction across 8 dimensions. Every classification is backed by a quoted excerpt from what the patient actually reported — the system never produces an unexplained score. If the AI cannot extract evidence for a dimension, it records the check-in as unclassified rather than fabricating a result.
- Trend detection compares each new assessment against the patient's prior one per dimension, generating typed insight events (`NEW_FRICTION`, `INCREASING_FRICTION`, `PERSISTENT_FRICTION`).
- Role-based routing enforces that patients and professionals are redirected away from each other's pages — not merely hidden from navigation — and professionals see only patients who have explicitly granted access.
- 48 backend pytest tests including the master-prompt acceptance scenario; 22 Vitest + React Testing Library tests covering the API client error handling, role-based routing, and key components. Vercel deployment verified (frontend and backend as separate projects).

> **Status:** Working prototype with mock AI provider (`AI_PROVIDER=mock`). The `AIProvider` interface is defined and the mock implements it fully; a real LLM provider would plug in via the same interface. Research prototype — not HIPAA-eligible. No real patient data.

---

### [StudioPulse](https://github.com/Sathvik2005/studio-pulse-for-agentic-film-)
*A five-agent streaming analytics system with an explicit state-machine workflow and ClickHouse as the sole source of truth for viewer telemetry.*

**Stack:** Python · FastAPI · ClickHouse · MCP (Model Context Protocol) · Gemini · React · Docker Compose · Cloud Run

- Five agents (Director, Analyst, Content Strategist, Ops/Anomaly, Governance) run a continuous cycle. The Analyst is the only agent with ClickHouse access; it returns validated structured JSON that the rest of the workflow consumes. The LLM is used only for summarisation and action ranking on top of already-determined data — never for arithmetic or as the source of a metric.
- The workflow is an explicit state machine (`agents/workflow.py`), not a freeform agent loop. Every transition is driven by validated structured output.
- ClickHouse integration is real runtime use via `backend/mcp_clickhouse/server.py`, which wraps `clickhouse-connect` behind five typed MCP tools (`query_trending`, `query_qoe`, `query_anomalies`, `execute_readonly_sql`, `write_audit_log`), imported and called by the agents at runtime.
- 15 backend tests pass against a real ClickHouse query engine via `chdb` (not a mock) — see `docs/END_TO_END_GUIDE.md` for the exact run and the 4 bugs caught during testing.

> **Limitation:** Viewer data is synthetic (`clickhouse/generator.py`). The Governance agent's action execution stubs and IAM lookups reference a local `iam_roles.json` rather than live Cloud IAM. Gemini is called at runtime in two summarisation passes only, with a deterministic fallback if not configured.

---

## Engineering Principles

**Evidence grounds; rules control; models interpret.**  
In every project here, the model's output is one input to a deterministic step, not the final answer. Hard rules in code or database constraints override what the model says when they conflict.

**Provenance before confidence.**  
A system that cannot show where a conclusion came from should not surface it. Grounding checks, verbatim quotes, and source IDs in every row are non-negotiable.

**Insufficient is a valid answer.**  
Overconfidence is a defect. A missing classification, a `REVIEW_REQUIRED` state, or a "no verified memory found" response is preferable to a fabricated one.

**Failure paths are first-class.**  
Bounded retries, classified error types, and explicit test cases for failure scenarios — unreachable database, contradictory evidence, rejected agent actions — are part of the implementation, not an afterthought.

**Implementations are distinguished from plans.**  
Mocked providers, synthetic data, and incomplete integrations are labelled as such in source and documentation. The description of what a system does in a test is not the same as what it does in production.

---

## Education

**B.Tech, Computer Science (AI & ML)** — VIT-AP University  
Final year student.

---

## Contact

GitHub: [github.com/Sathvik2005](https://github.com/Sathvik2005)
