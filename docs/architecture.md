# Architecture & Tools Walkthrough

This guide walks through the **important files**, the **workflow**, and the **tools** used in this repository so you can understand how to implement changes and extend the project.

![Open Deep Research Workflow Diagram](open-deep-research-workflow-diagram.png)

## 1) End-to-end Workflow (Purpose → Code)

1. **User enters a research query**  
   - CLI entry point in `main.py`.
2. **Planner creates a research plan**  
   - `planner.generate_research_plan()` produces a structured plan from the user query.
3. **Plan is split into explicit subtasks**  
   - `task_splitter.split_into_subtasks()` returns JSON subtasks that do not overlap.
4. **Coordinator spawns sub-agents**  
   - `coordinator.run_deep_research()` uses a tool to create a sub-agent per subtask.
5. **Sub-agents research with Firecrawl tools**  
   - Each sub-agent uses Firecrawl MCP tools for search/crawl/retrieval.
6. **Coordinator synthesizes everything**  
   - The coordinator merges sub-agent reports into the final markdown output.
7. **Final report is written to disk**  
   - `main.py` writes the output to `research_result.md`.

## 2) Important Files (What Each One Does)

| File | Purpose |
| --- | --- |
| `main.py` | CLI entry point. Loads env vars, prompts for a query, runs the pipeline, writes `research_result.md`. |
| `planner.py` | Uses Hugging Face Inference API to generate a high-level research plan from the user query. |
| `task_splitter.py` | Turns the plan into structured subtasks using a JSON schema (Pydantic). |
| `coordinator.py` | Orchestrates the workflow, initializes sub-agents, and aggregates their reports. Also wires Firecrawl MCP tools. |
| `prompts.py` | Central location for all system prompts/templates used by planner, splitter, sub-agents, and coordinator. |
| `pyproject.toml` | Project metadata, Python version, and dependencies. |
| `.env.example` | Required environment variables (`HF_TOKEN`, `FIRECRAWL_API_KEY`). |
| `docs/open-deep-research-workflow-diagram.png` | Visual workflow diagram used in docs. |
| `docs/blog-post.md` | Narrative tutorial explaining the workflow in a story-like format. |

## 3) Tooling (What’s Used & Why)

- **smolagents**  
  - Provides `ToolCallingAgent`, `MCPClient`, and model wrappers used for orchestration.  
  - Why: simplifies multi-agent workflows, tool calling, and prompt-driven coordination.

- **Firecrawl MCP**  
  - Exposed via `MCPClient` in `coordinator.py`.  
  - Why: gives agents powerful, ready-made search/crawl/retrieval tools without hand-writing scraping logic.

- **Hugging Face Inference API**  
  - `planner.py` uses `huggingface_hub.InferenceClient`.  
  - `coordinator.py` uses `InferenceClientModel` from `smolagents`.  
  - Why: access open models from HF providers while keeping the workflow model-agnostic.

- **Pydantic**  
  - Defines `Subtask` and `SubtaskList` schemas in `task_splitter.py`.  
  - Why: enforces structured JSON so tasks stay consistent and machine-parseable.

- **python-dotenv**  
  - `main.py` loads environment variables from `.env`.  
  - Why: keeps API keys out of code and easy to configure.

- **uv / pip**  
  - `uv sync` is recommended for dependency installation.  
  - Why: reproducible installs and fast environment setup.

## 4) How to Implement Changes (Where to Code)

Use this section as a map when you want to implement features or adjust the workflow.

### Change the planning behavior
- Edit `PLANNER_SYSTEM_INSTRUCTIONS` in `prompts.py`.  
- Update `MODEL_ID`/`PROVIDER` in `planner.py` if you want a different model.

### Change how tasks are split
- Edit `TASK_SPLITTER_SYSTEM_INSTRUCTIONS` in `prompts.py`.  
- Adjust schema or constraints in `task_splitter.py`.

### Change sub-agent behavior or report format
- Update `SUBAGENT_PROMPT_TEMPLATE` in `prompts.py`.  
- Add or remove tool usage rules in the prompt.

### Change how the coordinator orchestrates
- Edit `COORDINATOR_PROMPT_TEMPLATE` in `prompts.py`.  
- Adjust the flow in `coordinator.py` (e.g., add extra steps before/after sub-agent calls).

### Add new tools for agents
- Extend the MCP tool configuration in `coordinator.py`.  
- Any tools added there become available to sub-agents.

## 5) Quick Run Checklist

1. Set env vars in `.env` (see `.env.example`).
2. Install deps: `uv sync`
3. Run: `uv run main.py`
4. Enter a query → check `research_result.md`
