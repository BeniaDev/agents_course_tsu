# Task 1: Foundation Patterns (20 points)

**Covers:** Chapter 1 (Prompt Chaining), Chapters 2–4 (Routing, Parallelization, Reflection)

## Scenario

You are building an **automated customer support ticket processor** for a fictional company. The system receives incoming support messages and must: classify them, process them through the appropriate pipeline, extract structured information in parallel, and self-check output quality before returning a final response.

## What You Need to Build

### 1. Prompt Chaining (sequential pipeline)
Build a chain of **at least 3** LLM calls where each step's output feeds into the next:
1. **Preprocessing** — clean and normalize the raw user message (fix typos, expand abbreviations, standardize format)
2. **Classification** — determine the ticket category and extract key entities (product name, issue type, urgency)
3. **Response generation** — produce a draft reply to the customer based on the structured data from previous steps

### 2. Routing (dynamic branching)
After classification, route the ticket to one of **at least 3** specialized processing branches, for example:
- **Technical issue** → troubleshooting steps generation
- **Billing/refund** → policy lookup and refund eligibility check
- **General inquiry** → FAQ matching and informational response
- **Complaint/escalation** → empathetic response with escalation flag

Each branch should have its own prompt template and processing logic.

### 3. Parallelization (concurrent sub-tasks)
Within the pipeline, execute **at least 2** independent sub-tasks concurrently. For example:
- Sentiment analysis **+** keyword/entity extraction
- Response drafting **+** similar ticket search
- Priority scoring **+** language detection

Use `asyncio.gather()`, `concurrent.futures`, or any other concurrency mechanism.

### 4. Reflection (self-improvement loop)
Add a reflection step where the LLM evaluates its own generated response and iterates **at least 2 times**:
- First pass: generate draft response
- Evaluation: the LLM critiques the draft (tone, completeness, accuracy)
- Second pass: improve the response based on critique
- Log what changed between iterations so improvement is visible

## Sample Input Data

You can use any of the following datasets for realistic support tickets, or create your own test set of **at least 10 messages** covering all routes:

| Dataset | Link | Notes |
|---------|------|-------|
| Bitext Customer Support | https://huggingface.co/datasets/bitext/Bitext-customer-support-llm-chatbot-training-dataset | 27 intent categories, ready to use |
| Twitter Customer Support | https://huggingface.co/datasets/inaba/TwitterCustomerSupport | Real customer-agent conversations |
| Amazon Reviews | https://huggingface.co/datasets/McAuley-Lab/Amazon-Reviews-2023 | Product reviews, good for sentiment + routing |

## Recommended Models & APIs

Use **any** of the following (pick one as your primary model):

| Provider | Model | Access |
|----------|-------|--------|
| OpenRouter  | Any free api | https://openrouter.ai/models?order=pricing-low-to-high|
| Local / Free | `Qwen2.5-7B`, `Llama-3.1-8B` | Via [Ollama](https://ollama.com) or [HuggingFace Inference API](https://huggingface.co/inference-api) or check this Unsloth guides: https://unsloth.ai/docs/basics/inference-and-deployment | 

Check shit free API from OpenRouter:
- https://openrouter.ai/nvidia/nemotron-3-super-120b-a12b:free
- https://openrouter.ai/qwen/qwen3-next-80b-a3b-instruct:free
- https://openrouter.ai/openai/gpt-oss-20b:free
- https://openrouter.ai/z-ai/glm-4.5-air:free

## Deliverables

1. **Working Python application** with clear separation of the 4 patterns
2. **`README.md`** with:
   - Setup instructions (dependencies, API keys)
   - Architecture diagram or description (text is fine)
   - Example of running the pipeline with sample input/output
3. **Git repository** with proper branch structure:
   - `main` — stable, reviewed version of the project (final submission)
   - `develop` — working branch for active development
   - All feature work should be done in `develop` (or feature branches merged into `develop`), then merged into `main` when ready
4. **Console output or logs** showing:
   - The chain of steps executing sequentially
   - The routing decision and which branch was selected
   - Parallel tasks starting and completing
   - The reflection loop with before/after comparison

## Grading Rubric

| Criterion | Points | Expectations |
|-----------|--------|-------------|
| Prompt Chaining | 5 | Clear sequential steps with proper data flow between stages. Each step adds meaningful transformation. |
| Routing logic | 4 | Correct classification and branching for 3+ categories. Different branches produce noticeably different outputs. |
| Parallelization | 4 | Genuinely concurrent execution (not sequential). Proper result aggregation. |
| Reflection loop | 4 | Self-evaluation with **visible, measurable** improvement between iterations. |
| Code quality & documentation | 3 | Clean, readable code. README with setup instructions and example output. |

## Tips

- Start simple: get a basic chain working end-to-end first, then add routing, parallelization, and reflection one at a time.
- Use structured outputs (JSON mode) between chain steps — this makes data flow much cleaner.
- Keep your prompts in separate variables or files, not buried in code logic.
- Commit often. Your git history should show incremental progress.

## Helpful Links

- [Prompt Chaining Example](https://github.com/ai-tools-reviews/agentic-design-patterns/blob/main/part_1_foundational/prompt_chaining.py)
- [Parallelization Example](https://github.com/ai-tools-reviews/agentic-design-patterns/blob/main/part_1_foundational/parallelization.py)
- [Reflection Example](http://github.com/ai-tools-reviews/agentic-design-patterns/blob/main/part_1_foundational/reflection.py)