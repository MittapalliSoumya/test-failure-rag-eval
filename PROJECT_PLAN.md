# Test Failure RAG + Triage with Eval Harness

**Locked 2026-06-28. Ships 2026-07-25. Public repo target: `github.com/<user>/test-failure-rag-eval`.**

This project replaces the original "Week 7 mini-project" and "RAG v1" as separate artifacts. They are now one project. Synthetic data only — no work artifacts.

---

## What you're building

Given a new test failure (stack trace + log snippet), the system:
1. **Retrieves** the top-k most similar historical failures from a vector index
2. **Classifies** the new failure into a category (flaky / env / regression / data / config) grounded in retrieved context
3. **Suggests a root cause** citing retrieved evidence
4. **Measures itself** — retrieval quality, classification quality, end-to-end quality, cost, latency

The eval harness is the differentiator. Without it this is a bootcamp portfolio piece.

---

## Architecture (the boundaries that matter)

```
[New failure]
     |
     v
[Embed query]  -- deterministic
     |
     v
[Vector retrieval, top-k]  -- deterministic
     |
     v
[Prompt assembly: query + retrieved context]  -- deterministic
     |
     v
[LLM: classify + propose RCA]  -- PROBABILISTIC (the model boundary)
     |
     v
[Schema validation + confidence threshold]  -- deterministic guardrail
     |
     v
[Output: category, RCA, citations, confidence]
```

Parallel eval pipeline:
```
[Ground truth set] --> [Retrieval eval: recall@k, MRR]
                  --> [Classification eval: P/R/F1]
                  --> [End-to-end eval: GEVAL OR CheckEval]
                  --> [Cost/latency tracker]
```

---

## Phase 0 — Skeleton + Synthetic Data (Jul 4–6)

**Goal:** Empty repo runs `pytest` green. Synthetic corpus and ground truth exist on disk.

1. Create repo locally: `rag_triage_eval/`
   - `src/` — application code
   - `tests/` — pytest suite
   - `data/` — synthetic corpus + ground truth (committed for reproducibility)
   - `evals/` — eval scripts and reports
   - `pyproject.toml` or `requirements.txt`
   - `.env.example` (no real keys)
   - `README.md` (stub)
2. Synthetic data generator (`src/synth.py`)
   - Generate ~200 fake test failures with: `id`, `stack_trace`, `log_snippet`, `timestamp`, `service`, `true_category`, `true_root_cause`
   - 5 categories: `flaky`, `env`, `regression`, `data`, `config`
   - Use Claude API to generate realistic-looking traces — script-driven, not by hand
3. Ground truth file (`data/ground_truth.json`)
   - 30–50 labeled records: `{failure_id, expected_category, expected_similar_ids: [...]}`
   - The `expected_similar_ids` is what retrieval eval grades against — this is the hard part to construct, do it carefully
4. Smoke test: `pytest` runs, one test asserts the generator produces 200 records with all required fields

**Architect check at end of phase:** Can you draw the deterministic-vs-probabilistic boundary on a napkin? If not, you haven't internalized the design.

---

## Phase 1 — Indexing + Retrieval + Retrieval Eval (Jul 7–13)

**Goal:** Retrieve top-k similar failures and measure retrieval quality.

1. Embedding pipeline (`src/embed.py`)
   - Pick: `sentence-transformers` local (free, slower) or OpenAI/Voyage API (faster, costs $)
   - Recommend: start local with `all-MiniLM-L6-v2` to avoid cost during dev
2. Vector store (`src/index.py`)
   - Chroma (easiest) or FAISS (fewer deps). Pick one, don't waffle.
   - Index all 200 corpus records
3. Retrieval function (`src/retrieve.py`)
   - `retrieve(query_failure, k=5) -> List[FailureRecord]`
4. Retrieval eval harness (`evals/eval_retrieval.py`)
   - For each ground-truth query, compute:
     - **recall@k** for k=1,3,5,10
     - **MRR** (mean reciprocal rank)
   - Output a markdown report to `evals/reports/retrieval_<date>.md`
5. Pytest coverage for embed/index/retrieve modules

**Phase exit gate:** You have a number that says "retrieval works at X% recall@5." That number is now your baseline. Write it in the README.

**Architect drill:** What would make recall@5 jump from X% to X+15%? Name three levers (chunking strategy, embedding model, query expansion, reranker, etc.). Don't implement — just name them. This is the "how would you improve this" interview question.

---

## Phase 2 — Classification + RCA + Classification Eval (Jul 14–20)

**Goal:** LLM produces structured output grounded in retrieved context, and you measure how often it's right.

1. Prompt template (`src/prompt.py`)
   - Inputs: new failure + top-k retrieved failures
   - Output: JSON with `category`, `root_cause`, `citations` (list of retrieved IDs used), `confidence` (0–1)
   - Use Pydantic for output schema validation
2. Classification function (`src/classify.py`)
   - Calls Claude API (use Haiku for cost during dev, Sonnet for final eval)
   - Schema-validates the response; retry once on parse failure; log and fail loudly on second failure
3. Cost + latency tracker (`src/observability.py`)
   - Log per-call: tokens_in, tokens_out, $cost, latency_ms
   - Aggregate into p50/p95/avg in eval reports
4. Classification eval harness (`evals/eval_classification.py`)
   - For each ground-truth record, run classify, compare predicted vs true category
   - Compute precision/recall/F1 per category + macro-averaged
   - Output report to `evals/reports/classification_<date>.md`
5. Pytest coverage for prompt assembly, schema validation, retry logic

**Phase exit gate:** You have a number for classification F1 AND a number for $/classification. Both in the README.

**Architect drill:** Pick the lowest-F1 category. Why does the model do worst there? Is it a retrieval problem (wrong context being fed in) or a classification problem (right context, wrong call)? This is a real diagnostic skill — don't skip it.

---

## Phase 3 — End-to-End Eval + Polish + Ship (Jul 21–25)

**Goal:** Advanced eval technique implemented, README is interview-ready, repo public.

1. Pick **one** advanced eval — GEVAL or CheckEval. Don't do both.
   - **GEVAL:** LLM-as-judge with chain-of-thought scoring on dimensions like groundedness, completeness, correctness
   - **CheckEval:** decompose "is this RCA good?" into a checklist of yes/no questions, LLM answers each
   - Recommend: **CheckEval** — cheaper, more deterministic, easier to debug, and the technique is less commonly seen so it stands out more
2. Implement chosen eval (`evals/eval_e2e.py`)
   - Run on ground truth set
   - Compare against a "naive LLM-judge" baseline (just ask "is this good 1-10")
   - Report which performed better and why
3. README (`README.md`) — this is the artifact a hiring manager reads
   - One-paragraph problem statement
   - Architecture diagram (the boundary diagram above, cleaned up)
   - Eval methodology section — this is your moat, write it like you're defending it
   - Results table: retrieval metrics, classification metrics, end-to-end metrics, cost
   - "What I'd do next" section: drift detection, online eval, reranker, etc. — shows architect-level thinking
4. Final pytest pass at >80% coverage. `pytest --cov`.
5. Push to public GitHub. Tag `v1.0`.
6. LinkedIn post draft: "How I evaluated my own AI before claiming it works" — actual numbers, actual lessons.

**Ship gate:** Repo public, README readable, eval numbers real. No "I'll polish it later."

---

## Hard rules for this project (do not negotiate)

1. **Synthetic data only.** No work artifacts. Ever. Even anonymized.
2. **Eval before polish.** If you find yourself tweaking prompts before retrieval eval is wired up, stop.
3. **One advanced eval, shipped.** Not two half-done.
4. **README is part of the deliverable**, not an afterthought. A hiring manager reads the README, not the code.
5. **Track cost from day 1.** Architects estimate $/request on a napkin. Don't bolt this on at the end.

---

## What this project teaches you (the architect skills you're building)

- Drawing deterministic-vs-probabilistic boundaries in a real system
- Measuring retrieval quality, not assuming it
- Schema-validated LLM output (the production guardrail)
- Cost/latency observability built in, not bolted on
- One advanced eval technique deeper than "did it sound right"
- Writing a README that signals architect-level thinking

---

*Decision log: see `~/.claude/projects/-Users-smittapalli-python-practice/memory/project_rag_triage_eval.md`*
