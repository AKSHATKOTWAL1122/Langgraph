# Iterative LLM Refinement with LangGraph

A LangGraph workflow that generates tweets on a given topic and iteratively
refines them through a **generate → evaluate → optimize** loop until an LLM
judge approves the result or a maximum iteration count is reached.

## How it works

1. **`generate_tweet`** — asks Gemini 2.5 Flash to write a short, funny,
   original tweet on the given topic.
2. **`evaluate_tweet`** — a second Gemini instance acts as a ruthless critic,
   scoring the tweet on originality, humor, punchiness, virality, and format.
   Output is constrained to a structured `TweetEvaluation` (Pydantic) schema:
   `evaluation: "approved" | "needs_improvement"` plus written feedback.
3. **`optimize_tweet`** — if not approved, a third Gemini instance rewrites
   the tweet using the evaluator's feedback.
4. **`route_evaluation`** — a conditional edge sends approved tweets to `END`,
   otherwise loops back through `optimize → evaluate` until approval or
   `max_iteration` is hit.

```
START -> generate -> evaluate --approved--> END
                         ^                    
                         |             needs_improvement
                         +------- optimize <-+
```

## Files

- [`8_iterative_llm_.ipynb`](./8_iterative_llm_.ipynb) — the workflow,
  built with `langgraph`, `langchain-google-genai`, and `pydantic`.

## Setup

```bash
pip install langgraph langchain-google-genai python-dotenv pydantic
```

Create a `.env` file with your Gemini API key:

```
GOOGLE_API_KEY=your_key_here
```

Then run the notebook cells in order. The final cell invokes the compiled
graph with an example `topic` (e.g. `"indian railways"`).
