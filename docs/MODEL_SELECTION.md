# Model Selection Guide

All agents in this system use [Mistral AI](https://mistral.ai) models via the
hosted API. Set `MISTRAL_API_KEY` (get one at https://console.mistral.ai/).

---

## Why model choice matters for agents

Agents call tools by generating structured JSON. A model that
hallucinates tool names or produces malformed JSON will fail
silently, the tool call won't execute, the agent will loop,
and you'll hit the `max_iterations` limit without understanding why.

All models below support function calling and JSON mode.

---

## Recommendations

| Model | Best for |
|---|---|
| `mistral-small-latest` (default) | Fast, cheap, good tool calling |
| `mistral-medium-latest` | Better reasoning, higher cost |
| `mistral-large-latest` | Best quality, highest cost |

---

## Agent-specific settings

These settings are used throughout the codebase:

```python
# Structured output (Curriculum Planner, Quiz Generator grader)
ChatMistralAI(temperature=0.1, model_kwargs={"response_format": {"type": "json_object"}})

# Tool-calling (Explainer)
ChatMistralAI(temperature=0.3)

# Creative/coaching (Progress Coach, Quiz Generator questions)
ChatMistralAI(temperature=0.4, model_kwargs={"response_format": {"type": "json_object"}})
```

Low temperature = more consistent JSON. Higher temperature = more
varied explanations and coaching messages. Never use temperature > 0.5
for any agent that produces structured output.

---

## Switching models

Change `MISTRAL_MODEL` in `.env`. No code changes needed.

```bash
# .env
MISTRAL_API_KEY=your-key
MISTRAL_MODEL=mistral-large-latest
```

---

## Eval tests

Eval tests in `tests/test_eval.py` use the same Mistral model as the judge
and make real API calls, so they consume API credits. If they fail
consistently with `mistral-small-latest`, try a larger model before
lowering thresholds.
