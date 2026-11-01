# Multi-Agent Learning Accelerator

[![CI](https://github.com/fedy-benhassouna/multi-agent-learning-accelerator/actions/workflows/ci.yml/badge.svg)](https://github.com/fedy-benhassouna/multi-agent-learning-accelerator/actions/workflows/ci.yml)
![Python](https://img.shields.io/badge/python-3.11%2B-blue)
![LangGraph](https://img.shields.io/badge/orchestration-LangGraph-green)
![LLM](https://img.shields.io/badge/LLM-Mistral%20AI-orange)
[![License: MIT](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

A production-style multi-agent AI system built with **LangGraph**, **MCP**,
**A2A**, **CrewAI** and **Mistral AI**, with Langfuse tracing and DeepEval
quality tests.

Based on the freeCodeCamp handbook companion code by
[Sandeep Bharadwaj Mannapur](https://github.com/sandeepmb), adapted to run
on the Mistral AI API instead of a local Ollama model.

---

## What this builds

A **Learning Accelerator**: a four-agent system that plans a study curriculum,
explains topics from your own notes, quizzes you, and adapts based on results.
The use case is the teaching vehicle. The architecture is the real subject.

```
Goal: "Learn Python closures and decorators"
  │
  ▼
Curriculum Planner  →  structured study roadmap
  │
  ▼ (you approve the plan)
Explainer           →  reads your notes via MCP, explains each topic
  │
  ▼
Quiz Generator      →  tests understanding, grades answers with LLM-as-judge
  │
  ▼
Progress Coach      →  adapts roadmap, calls CrewAI Study Buddy via A2A
  │
  └── loops back to Explainer for next topic
```

The same architecture pattern runs in production for sales enablement,
compliance training, customer support onboarding, and engineering ramp-up.

---

## Architecture

| Layer | Technology | What it does |
|---|---|---|
| Orchestration | LangGraph 1.1.0 | Stateful agent graph with checkpointing |
| Tool integration | MCP (mcp 1.26.0) | Standardised agent-to-tool protocol |
| Agent coordination | A2A (a2a-sdk 0.3.25) | Cross-framework agent-to-agent protocol |
| LLM provider | Mistral AI | Hosted LLM API (api.mistral.ai) |
| Cross-framework | CrewAI 1.13.0 | Study Buddy agent (called via A2A) |
| Observability | Langfuse 4.0.1 | Full trace of every agent and LLM call |
| Evaluation | DeepEval 3.9.1 | LLM-as-judge quality metrics |

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for a full deep-dive.

---

## Requirements

- Python 3.11+
- A [Mistral AI](https://console.mistral.ai/) API key
- Docker Desktop (for Langfuse observability, optional)

---

## Quick start

```bash
# 1. Clone and set up
git clone https://github.com/fedy-benhassouna/multi-agent-learning-accelerator
cd multi-agent-learning-accelerator
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 2. Configure
cp .env.example .env
# Edit .env: set MISTRAL_API_KEY (and optionally MISTRAL_MODEL)

# 3. Run
python main.py
```

For the full system with A2A services and observability, see [Running all services](#running-all-services).

---

## Project structure

```
multi-agent-learning-accelerator/
│   ├── agents/                 # LangGraph agent nodes
│   │   ├── curriculum_planner.py
│   │   ├── explainer.py
│   │   ├── quiz_generator.py
│   │   ├── progress_coach.py
│   │   └── human_approval.py
│   ├── graph/
│   │   ├── state.py            # Shared AgentState TypedDict
│   │   └── workflow.py         # LangGraph graph definition
│   ├── mcp_servers/            # MCP tool servers
│   │   ├── filesystem_server.py
│   │   └── memory_server.py
│   ├── a2a_services/           # A2A protocol services and client
│   │   ├── quiz_service.py     # Quiz Generator as A2A service
│   │   └── a2a_client.py       # Client for calling A2A services
│   ├── crewai_agent/
│   │   └── study_buddy.py      # CrewAI agent served via A2A
│   └── observability/
│       └── langfuse_setup.py   # Langfuse callback handler
├── tests/
│   ├── conftest.py             # Shared fixtures and markers
│   ├── test_state.py           # 24 tests
│   ├── test_curriculum_planner.py  # 11 tests
│   ├── test_mcp_servers.py     # 36 tests
│   ├── test_explainer.py       # 14 tests
│   ├── test_quiz_and_coach.py  # 17 tests
│   ├── test_checkpointing.py   # 20 tests
│   ├── test_observability.py   # 16 tests
│   ├── test_a2a.py             # 19 tests
│   ├── test_crewai_interop.py  # 25 tests
│   └── test_eval.py            # 12 eval tests (requires MISTRAL_API_KEY)
├── study_materials/
│   └── sample_notes/           # Markdown files the agents read
├── docs/
│   ├── ARCHITECTURE.md
│   └── MODEL_SELECTION.md
├── data/                       # SQLite checkpoint DB (created at runtime)
├── main.py                     # Entry point
├── docker-compose.yml          # Langfuse self-hosted stack
├── Makefile                    # One-command startup
├── requirements.txt
└── .env.example
```

---

## Running all services

The full system has three processes. Open three terminal tabs.

**Tab 1: Quiz Generator A2A service:**
```bash
source .venv/bin/activate
python src/a2a_services/quiz_service.py
# Serves at http://localhost:9001
```

**Tab 2: CrewAI Study Buddy A2A service:**
```bash
source .venv/bin/activate
python src/crewai_agent/study_buddy.py
# Serves at http://localhost:9002
```

**Tab 3: Main LangGraph application:**
```bash
source .venv/bin/activate
python main.py
```

Or use the Makefile:
```bash
make services   # starts both A2A services in background
make run        # runs the main application
```

---

## Session resume

Every session is checkpointed to `data/checkpoints.db` after each agent node.
To resume a stopped session:

```bash
python main.py --resume <session-id>
```

The session ID is printed at the start of every run.

---

## Observability

Start Langfuse locally:
```bash
docker compose up -d
# Open http://localhost:3000
```

Add your Langfuse API keys to `.env` (from the Langfuse project settings), then run
as normal. For Langfuse Cloud set `LANGFUSE_BASE_URL=https://cloud.langfuse.com`;
for the local stack leave it unset (defaults to `http://localhost:3000`). Every
agent call, LLM completion, and tool call appears in the trace UI automatically.

---

## Testing

```bash
# Fast unit tests, run during development (~3 seconds)
# 184 unit tests across 9 test files
pytest tests/ -m "not eval" -v

# Quality evaluation tests, run before releases (~90 seconds, requires MISTRAL_API_KEY)
# 12 LLM-as-judge tests, these call the Mistral API and use credits
pytest tests/test_eval.py -v -s -m eval
```

---

## Adding your own study materials

Replace or add Markdown files in `study_materials/sample_notes/`.
The Explainer agent reads every `.md` file in that directory automatically
via the MCP filesystem server. No configuration changes needed.

---

## Configuration reference

See `.env.example` for all available settings.

Key toggles:
| Variable | Default | Effect |
|---|---|---|
| `MISTRAL_API_KEY` | (required) | Mistral AI API key |
| `MISTRAL_MODEL` | `mistral-small-latest` | Model for all agents |
| `USE_A2A_QUIZ` | `true` | Route quiz tasks to A2A service |
| `USE_STUDY_BUDDY` | `true` | Call CrewAI Study Buddy for low scores |
| `CHECKPOINT_DB` | `data/checkpoints.db` | SQLite path for checkpoints |

---

Maintained by [@fedy-benhassouna](https://github.com/fedy-benhassouna).
Released under the [MIT License](LICENSE).
