# Task 4: Evaluation & Advanced Reasoning (45 points)

**Covers:** Chapter 8 (Advanced Reasoning — ReAct, Tree-of-Thoughts, Plan-and-Execute, Self-Consistency), Chapter 9 (Evaluation — Golden Sets, LLM-as-Judge, Regression Testing), Chapter 10 (Observability — Tracing, Replay, Failure Analysis)

## Scenario

You are building an **Agentic Reasoning Lab** — a system that solves hard, multi-step problems (math word problems, logical puzzles, multi-hop QA, or code-generation tasks) using **multiple competing reasoning strategies**, and then **measures them rigorously** against a held-out benchmark.

Unlike Tasks 1–3, the emphasis here is on **scientific rigor**: you should be able to answer questions like *"Does ReAct beat Plan-and-Execute on GSM8K with this model, and is the difference statistically meaningful?"* with numbers and traces to back it up.

This is the capstone task of the course — it ties together everything from Tasks 1–3 (chaining, routing, parallelization, reflection, RAG, safety) by making you **measure** the systems you've been building.

## What You Need to Build

### 1. Advanced Reasoning Strategies (16 points)

Implement **at least 3 distinct reasoning strategies** on the same problem set so they can be compared head-to-head. Each must be a real implementation of the pattern — not a one-line prompt tweak.

Required (pick **any 2**):
- **ReAct** — interleaved Reason–Act–Observe loop, with at least one external tool (calculator, web search, code executor, retriever from Task 3 — your choice).
- **Plan-and-Execute** — a planner LLM produces a step-by-step plan upfront; an executor (separate prompt / persona) runs each step.
- **Self-Consistency** — sample `N ≥ 3` reasoning paths in parallel, majority-vote (or judge-vote) on the final answer.

Pick **at least one more** (the "advanced" one):
- **Tree-of-Thoughts (ToT)** — explicit branching with ≥ 2 alternatives per step, a value/score function over partial states, and a simple search strategy (BFS or beam is fine). Log the tree for at least one example.
- **Reflexion** — failure-driven retry where the agent writes a verbal lesson to itself between attempts (different from Task 2's producer-critic — here the agent reflects on *its own trajectory*, not a draft).
- **Program-of-Thought** — agent emits executable code (Python) and runs it to compute the answer.

Requirements:
- Each strategy lives in its own module behind a **common `Strategy` interface** (e.g. `solve(problem) -> Trace`). This is what makes apples-to-apples comparison possible.
- Tools should be **shared across strategies** where applicable — if ReAct uses a calculator, Plan-and-Execute should use the same one. Otherwise comparisons aren't fair.

### 2. Evaluation Framework (14 points)

Build a **proper eval harness** — not "I ran 5 examples by hand and it looked good."

Requirements:
- **Golden test set of ≥ 20 problems** with ground-truth answers. Use a public benchmark (see below), a subset of one, or hand-crafted problems — but it must be **held-out** (never shown to the model during prompt-tuning) and **version-controlled** in the repo.
- **Programmatic metric** appropriate to the task: exact-match, F1, `pass@k` for code, numerical-tolerance match for math. Document the metric and why it fits.
- **LLM-as-judge** for cases where programmatic match is too brittle (e.g. free-form answers). Sanity-check the judge: hand-grade ≥ 8 examples, report **judge–human agreement as simple accuracy** (Cohen's κ welcome but not required). If the judge agrees with you < 70% of the time, fix the rubric before trusting numbers.
- **Pairwise comparison**: for each problem, run all strategies and record the per-problem outcome. Produce a **strategy × strategy win matrix**.
- **Confidence intervals**: report not just accuracy %, but a 95% CI per strategy (Wilson interval or a 1000-sample bootstrap — both are 3 lines of code). Don't declare a winner on a 2-point delta over 20 examples.
- **Baseline record**: store the current accuracy numbers as a `baseline.json` (or similar) in the repo so future runs can be diff'd against them. A `make eval` command that prints the delta is great; a CI-style auto-fail is bonus, not required.

### 3. Observability & Trace Analysis (11 points)

Instrument every agent run so failures are debuggable **after the fact** — you should not need to re-run with print statements to understand what went wrong.

Requirements:
- **Structured traces**: every LLM call, every tool call, every reasoning step recorded as a structured event (JSONL is fine — no tracing library required). Each event has: timestamp, strategy, problem ID, step type, inputs, outputs, token usage, latency.
- **Cost & latency table** (in the README): per-strategy totals for tokens-in, tokens-out, wall-clock time, and **cost per correct answer** (the metric that actually matters). Using Langfuse / Phoenix / Braintrust is welcome but optional.
- **Failure mode taxonomy**: hand-classify ≥ 8 failures across strategies into categories (e.g. *wrong tool call*, *arithmetic error*, *plan abandoned*, *judge disagreement*, *infinite loop*). Report the distribution. This is qualitative work — show your reasoning, not just a CSV.
- **Replayability**: given a trace ID, your tooling must let you **re-run a single problem** (optionally with a different strategy or model). A diff against the original trace is bonus, not required.
- Pick at least one **observation** from your traces and write it up briefly in the README (e.g. *"ToT spent 40% of its tokens on branches it pruned"* or *"most ReAct failures were due to a single tool returning malformed output"*).

## Suggested Benchmarks / Datasets

Pick a problem domain that lets you actually grade answers. Mixing domains is fine but means more eval plumbing.

| Domain | Dataset | Notes |
|--------|---------|-------|
| Math word problems | [GSM8K](https://huggingface.co/datasets/openai/gsm8k) | Numerical answer, exact-match grading — easiest to start |
| Hard math | [MATH](https://huggingface.co/datasets/hendrycks/competition_math) | Latex answers, harder to grade — use sympy or LLM judge |
| Multi-hop QA | [HotpotQA](https://huggingface.co/datasets/hotpotqa/hotpot_qa) | Pairs naturally with your Task 3 RAG |
| Code | [HumanEval](https://huggingface.co/datasets/openai/openai_humaneval) | Use `pass@k` with a sandboxed executor |
| Reasoning | [BIG-Bench Hard (BBH)](https://huggingface.co/datasets/lukaemon/bbh) | Diverse reasoning tasks, good for showing strategy differences |
| Commonsense | [ARC](https://huggingface.co/datasets/allenai/ai2_arc) | MCQ, easy to grade |

**Pick a subset (≥ 30 examples).** Running 1,000 examples on a free-tier API will burn your quota and your patience.

## Recommended Models & Tooling

| Component | Suggestions |
|-----------|-------------|
| Solver model | Reuse from Tasks 1–3 (OpenRouter free tier, Ollama local) |
| Judge model | **Different** from the solver — same anti-collusion logic as Task 2's critic |
| Tracing / observability | [Langfuse](https://langfuse.com) (self-hostable, free tier), [Arize Phoenix](https://github.com/Arize-ai/phoenix) (local OSS), [OpenTelemetry](https://opentelemetry.io) + JSONL, or a hand-rolled JSONL logger |
| Eval frameworks (optional) | [OpenAI Evals](https://github.com/openai/evals), [Inspect AI](https://inspect.ai-safety-institute.org.uk/), [`promptfoo`](https://promptfoo.dev), [DeepEval](https://github.com/confident-ai/deepeval), [`lm-eval-harness`](https://github.com/EleutherAI/lm-evaluation-harness) |
| Sandboxed code execution (if doing code tasks) | [E2B](https://e2b.dev), Docker with resource limits, or `subprocess` with strict timeouts (last resort) |
| Stats | `scipy.stats` for McNemar, `numpy` for bootstrap CIs |

## Deliverables

1. **Working Python application** with clean module separation: `strategies/` (one file per reasoning strategy + shared `Strategy` interface), `eval/` (golden set, metrics, judge, statistical tests), `observability/` (trace logger, replay tool, analysis notebooks), `tools/` (shared tool implementations).
2. **`README.md`** with:
   - Setup (deps, API keys, `.env`, how to run the eval)
   - Architecture diagram (ASCII fine) showing strategy interface → tools → eval harness → trace store
   - Description of the chosen benchmark and why
   - **Results table**: per-strategy accuracy with 95% confidence intervals
   - **Win matrix** (Strategy × Strategy)
   - **Cost / latency table**: tokens, $, wall-clock, **cost per correct answer**
   - **Judge sanity-check**: judge–human agreement (simple accuracy is fine), on which subset
   - **Failure taxonomy**: distribution table + ≥ 1 worked failure example with a trace excerpt
   - One observation from trace analysis (a paragraph)
3. **Git repository** with branch discipline:
   - `main` — final submission, with passing `make eval`
   - `develop` — active work
   - Feature branches encouraged; one branch per strategy is a natural fit
4. **Console output / logs** clearly showing:
   - A single example run per strategy on the same problem (so the difference in behaviour is visible)
   - If you implemented ToT: the search tree for at least one example
   - The eval harness running end-to-end with progress + final metrics
   - A replay invocation: re-running one problem from a stored trace

## Grading Rubric

| Criterion | Points | Expectations |
|-----------|--------|-------------|
| Advanced Reasoning | 16 | ≥ 3 strategies behind a shared interface; real (not toy) implementations; tools shared across strategies where applicable. |
| Evaluation Framework | 14 | Held-out golden set ≥ 20; appropriate programmatic metric; LLM judge with reported human-agreement sanity check; pairwise win matrix; 95% CIs per strategy; recorded baseline. |
| Observability & Trace Analysis | 11 | Structured traces for every call; cost-per-correct-answer reported; failure taxonomy with ≥ 8 hand-classified examples; working replay tool; at least one observation written up. |
| Code quality & documentation | 4 | Clear module boundaries, prompts externalized, README with diagram + all required tables + worked examples. Reproducible from a clean clone. |

## Tips

- **Pick one benchmark and stick with it.** A great eval on 30 GSM8K problems beats a mediocre eval spanning 4 datasets.
- **Build the eval harness before the strategies.** If you implement ReAct first and then try to retrofit evaluation, you'll bake assumptions into ReAct that the other strategies can't satisfy. Start with the `Strategy` interface and a stub.
- **The judge is a model too — it has bugs.** Always hand-grade a sample. If the judge disagrees with you systematically (e.g. it accepts unsupported reasoning), fix the rubric or switch models *before* trusting any numbers.
- **Self-consistency is the cheapest big win.** N=5 majority vote on a CoT prompt often beats much fancier strategies — include it as a baseline so your fancier work has something honest to beat.
- **Cache aggressively.** Cache LLM calls by `(prompt, model, seed)` — your eval will re-run many times during development, and free-tier rate limits will not love you.
- **Beware of train/test leak.** If you tuned your prompt on problems that are also in your eval set, your numbers are fiction. Split before you start.
- **Cost per correct answer** is the metric leadership cares about. A strategy that's 2% more accurate but 8× more expensive is usually a no-go — surface this tradeoff explicitly.
- **Commit often.** Good git history: `strategy interface + stub` → first strategy → second strategy → third strategy → `eval harness + golden set` → `judge sanity check` → `tracing` → `win matrix + CIs` → `failure taxonomy` → `baseline recorded`.

## Helpful Links

- ReAct paper (Yao et al., 2022): https://arxiv.org/abs/2210.03629
- Tree of Thoughts paper (Yao et al., 2023): https://arxiv.org/abs/2305.10601
- Plan-and-Solve prompting (Wang et al., 2023): https://arxiv.org/abs/2305.04091
- Self-Consistency (Wang et al., 2022): https://arxiv.org/abs/2203.11171
- Reflexion (Shinn et al., 2023): https://arxiv.org/abs/2303.11366
- Program-of-Thoughts (Chen et al., 2022): https://arxiv.org/abs/2211.12588
- Anthropic on evaluating agents: https://www.anthropic.com/engineering/building-effective-agents
- OpenAI on evals best practices: https://cookbook.openai.com/examples/evaluation/getting_started_with_openai_evals
- Hamel Husain on **LLM eval discipline** (must-read): https://hamel.dev/blog/posts/evals/
- Eugene Yan on LLM evals: https://eugeneyan.com/writing/llm-evaluators/
- [Langfuse docs (tracing)](https://langfuse.com/docs)
- [Arize Phoenix (local OSS tracing)](https://github.com/Arize-ai/phoenix)
- [Inspect AI (UK AISI eval framework)](https://inspect.ai-safety-institute.org.uk/)
- McNemar's test (statistical comparison of paired classifiers): https://en.wikipedia.org/wiki/McNemar%27s_test
