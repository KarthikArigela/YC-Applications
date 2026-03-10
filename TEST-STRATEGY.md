# Test Strategy — Voice Agent Intelligence Dashboard

> Companion document to `Bolna-AI-Strategy.md`.
> Covers every layer of the proposed prototype and identifies where tests would have the highest ROI.

---

## Current State

No source code exists yet. This document serves two purposes:
1. Define the test strategy **before** writing the implementation (test-first thinking).
2. Identify the riskiest logic areas that require the most coverage.

---

## Prototype Layer Map

```
┌──────────────────────────────────────────────────────────┐
│  Frontend (Next.js + TailwindCSS + shadcn/ui)            │
│  • Dashboard, call list, drill-down, comparison view     │
├──────────────────────────────────────────────────────────┤
│  Backend API (FastAPI)                                   │
│  • Ingestion endpoints, analysis endpoints, scoring API  │
├──────────────────────────────────────────────────────────┤
│  Core Engine (Python)                                    │
│  • Transcript parser → LLM analyzer → Scorer             │
│  • Synthetic call simulator                              │
├──────────────────────────────────────────────────────────┤
│  Persistence (Supabase / SQLite for local dev)           │
│  • Calls, analyses, agent versions, scores               │
└──────────────────────────────────────────────────────────┘
```

---

## 1. CRITICAL — Production Readiness Scorer

**Why it's the riskiest:** The 0–100 score is the primary value proposition of the entire prototype. If the scoring logic drifts or has edge-case bugs, every recommendation downstream is wrong. Customers will make real production decisions based on this number.

**What to test:**

| Test | Type | Priority |
|------|------|----------|
| Score of 100 when all 5 dimensions are perfect | Unit | P0 |
| Score of 0 when all 5 dimensions are zero | Unit | P0 |
| Each dimension is weighted exactly 20% | Unit | P0 |
| Score is always in [0, 100] range regardless of inputs | Unit | P0 |
| Partial data (missing dimensions) defaults gracefully | Unit | P0 |
| Score with 1 call vs. 100 calls produces stable results | Unit | P1 |
| Before/after comparison delta is signed correctly (V2 better than V1) | Unit | P1 |
| Score degrades correctly when failure rate rises incrementally | Property | P1 |
| Rounding does not cause score to exceed 100 | Unit | P2 |

**Tooling:** `pytest` + `hypothesis` for property-based testing of score invariants.

**Key gap to fill first:** The scoring rubric math. Write the scorer as a pure function of a `ScoredDimensions` dataclass — no I/O, no LLM calls — so it is 100% unit-testable.

---

## 2. CRITICAL — Transcript Parser

**Why it's risky:** Transcripts arrive in multiple formats (Bolna webhook JSON, CSV upload, raw text). Malformed or unexpected input at this layer causes silent data loss — a call analyzed with truncated data will produce a misleadingly high score.

**What to test:**

| Test | Type | Priority |
|------|------|----------|
| Parse well-formed Bolna webhook payload | Unit | P0 |
| Parse CSV with correct `contact_number` header | Unit | P0 |
| Reject CSV missing required columns, return clear error | Unit | P0 |
| Preserve speaker turns (agent vs. customer) correctly | Unit | P0 |
| Parse transcript with code-switching (Hindi + English) without truncation | Unit | P0 |
| Handle empty transcript (zero-turn call) without crashing | Unit | P1 |
| Handle very long transcript (500+ turns) within memory limit | Unit | P1 |
| Parse timestamps and compute call duration | Unit | P1 |
| Gracefully handle malformed JSON (return error, not 500) | Unit | P1 |
| Reject files >50MB with actionable error message | Integration | P2 |

**Tooling:** `pytest` with fixture files (`tests/fixtures/transcripts/`) covering each format variant.

**Key gap:** Need fixtures for edge-case Bolna webhook payloads — especially calls where the agent barged in or where the user was silent. These should be created from the Bolna API docs before writing the parser.

---

## 3. HIGH — LLM Analysis Engine (Mocked)

**Why it's risky:** The LLM pipeline is the most complex unit. Prompts can regress silently — a prompt change that looks reasonable can cause the model to stop returning valid structured JSON, breaking the entire analysis pipeline downstream.

**What to test:**

| Test | Type | Priority |
|------|------|----------|
| LLM response is valid JSON matching the `CallAnalysis` schema | Unit (mocked) | P0 |
| Parser handles LLM returning markdown-wrapped JSON (```json...```) | Unit | P0 |
| Failure when LLM returns plain text (not JSON) raises `AnalysisParseError` | Unit | P0 |
| Retry logic triggers on LLM timeout (max 3 retries) | Unit (mocked) | P1 |
| Correctly extracts `resolution_status: resolved/unresolved/unclear` | Unit (mocked) | P1 |
| Correctly identifies language switch events in transcript | Unit (mocked) | P1 |
| Correctly flags hallucination indicators (fabricated data, wrong numbers) | Unit (mocked) | P1 |
| Correctly identifies interruption handling failures | Unit (mocked) | P1 |
| Analysis of identical transcripts is deterministic (temperature=0) | Integration | P2 |
| Cost per call is tracked and logged | Unit | P2 |

**Tooling:** Mock the Claude API using `pytest-mock` or `respx` (for httpx). Never call real LLM in unit tests. Create a small library of canned LLM responses covering success and failure cases.

**Key gap:** No contract test exists for the `CallAnalysis` JSON schema. The schema should be defined as a Pydantic model first and validated against every LLM response. This prevents silent schema drift.

---

## 4. HIGH — FastAPI Endpoints

**What to test:**

| Endpoint | Test | Priority |
|----------|------|----------|
| `POST /transcripts/upload` | Returns 200 with job ID for valid CSV | P0 |
| `POST /transcripts/upload` | Returns 422 for missing file | P0 |
| `POST /transcripts/upload` | Returns 400 for malformed CSV | P0 |
| `GET /calls/{call_id}` | Returns full analysis for existing call | P0 |
| `GET /calls/{call_id}` | Returns 404 for unknown call ID | P0 |
| `GET /agents/{agent_id}/score` | Returns readiness score with all 5 dimensions | P0 |
| `POST /simulate` | Returns synthetic scenario list for valid use case | P1 |
| `POST /agents/compare` | Returns delta between two agent versions | P1 |
| `GET /calls` | Pagination works (limit/offset) | P2 |
| `GET /calls` | Filtering by failure type works | P2 |

**Tooling:** FastAPI `TestClient` (synchronous) + `pytest`. Seed test DB with fixtures before each test. Use `pytest-anyio` for async endpoints.

**Key gap:** No auth layer planned for MVP, but the API should reject calls with oversized payloads. Add `Content-Length` guard tests before going public.

---

## 5. MEDIUM — Synthetic Call Simulator

**Why it matters:** The simulator generates fake customer personas and runs them against agent prompts. If the persona generator produces unrealistic scenarios or the simulation loop can enter infinite turns, it wastes money and produces garbage data.

**What to test:**

| Test | Type | Priority |
|------|------|----------|
| Simulator returns exactly N personas when N is requested | Unit (mocked) | P1 |
| Each persona has all required fields (name, intent, language, tone) | Unit (mocked) | P1 |
| Simulation terminates within max_turns limit | Unit | P0 |
| Simulation correctly detects `RESOLVED` / `FAILED` terminal states | Unit (mocked) | P1 |
| Simulator logs cost per simulation run | Unit | P2 |
| Handles LLM mid-simulation failure (network error) gracefully | Unit (mocked) | P1 |
| Two simulation runs with the same seed produce same personas | Unit (mocked) | P2 |

**Key gap:** No max-turn guard exists in the design. The simulation loop must have a hard ceiling (e.g., 30 turns) to prevent runaway LLM calls. This should be enforced in a unit test before the feature ships.

---

## 6. MEDIUM — Frontend Components

**What to test:**

| Component | Test | Priority |
|-----------|------|----------|
| `ReadinessGauge` | Renders "87" when score prop is 87 | Unit | P1 |
| `ReadinessGauge` | Color is red/amber/green based on score range | Unit | P1 |
| `CallList` | Renders correct number of rows from mock data | Unit | P1 |
| `CallList` | Failure-type filter reduces visible rows | Unit | P1 |
| `CallDrillDown` | Highlights failure moments in conversation timeline | Unit | P1 |
| `ComparisonView` | Shows positive delta in green, negative in red | Unit | P2 |
| `UploadForm` | Shows error when non-CSV file is dropped | Unit | P1 |
| `UploadForm` | Disables submit button while upload is in progress | Unit | P2 |
| Full upload → analysis → score flow | E2E | P2 |

**Tooling:** Vitest + React Testing Library for component tests. Playwright for E2E.

**Key gap:** The `ReadinessGauge` score display is the most visible element of the product. It needs snapshot tests so a CSS refactor doesn't accidentally hide the score number.

---

## 7. LOW (but worth noting) — Database Layer

For the MVP, the DB is mostly Supabase-managed. Test coverage here is lower priority but should include:

| Test | Priority |
|------|----------|
| A new call record is persisted after analysis | P2 |
| Updating an agent's prompt version creates a new version record (not overwrite) | P1 |
| Deleting an agent cascades to its calls and scores | P2 |

---

## Test Coverage Targets

| Layer | Current | Target (MVP ship) |
|-------|---------|-------------------|
| Scorer (pure function) | 0% | **100%** |
| Transcript parser | 0% | **90%** |
| LLM analysis engine (mocked) | 0% | **80%** |
| FastAPI endpoints | 0% | **80%** |
| Synthetic simulator | 0% | **70%** |
| Frontend components | 0% | **60%** |
| E2E | 0% | **1 happy path** |

---

## Recommended Test Infrastructure Setup

```
voice-agent-dashboard/
├── backend/
│   ├── tests/
│   │   ├── conftest.py          # Shared fixtures, test DB setup
│   │   ├── fixtures/
│   │   │   ├── transcripts/     # Sample Bolna webhook payloads, CSVs
│   │   │   └── llm_responses/   # Canned LLM responses (success + failures)
│   │   ├── unit/
│   │   │   ├── test_scorer.py
│   │   │   ├── test_parser.py
│   │   │   ├── test_analyzer.py
│   │   │   └── test_simulator.py
│   │   └── integration/
│   │       └── test_api.py
│   └── pytest.ini
└── frontend/
    └── src/
        └── __tests__/
            ├── ReadinessGauge.test.tsx
            ├── CallList.test.tsx
            └── UploadForm.test.tsx
```

**CI:** Run `pytest --cov=app --cov-fail-under=75` and `vitest run` on every PR. Block merge if coverage drops below threshold.

---

## Top 3 Things to Build Tests for First

In order of business risk:

1. **Scorer invariants** — The score is the product. If it's wrong, the whole pitch is wrong.
2. **Transcript parser edge cases** — Silent failures here corrupt every analysis silently.
3. **LLM JSON schema contract** — Schema drift is the most common way LLM pipelines break in production.

---

*Document created: March 2026*
