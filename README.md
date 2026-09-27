# Langfuse Learning Projects

Small, runnable examples for learning Langfuse observability with LangChain and LangGraph. The notebooks build from a basic chatbot to a traced multi-agent workflow and prompt management.

## What You Will Learn

- Build a chatbot with LangGraph and a `StateGraph`.
- Add Langfuse tracing with the LangChain `CallbackHandler`.
- Add trace context such as a trace name, user, session, tags, and metadata with `propagate_attributes`.
- Build a supervisor that routes work to specialized agents.
- Give agents tools, including Wikipedia lookup and current-time lookup.
- Create and retrieve versioned prompts with Langfuse.
- Run a graph either incrementally with `.stream(...)` or as one completed result with `.invoke(...)`.

## Project Layout

Run the notebooks in this order:

1. [`basic_chatbot.ipynb`](basic_chatbot.ipynb) introduces a single LangGraph chatbot and basic Langfuse callbacks.
2. [`basic_chatbot_v2.ipynb`](basic_chatbot_v2.ipynb) adds richer trace attributes and metadata.
3. [`basic_multi_agent.ipynb`](basic_multi_agent.ipynb) adds a supervisor, researcher and current-time workers, tools, conditional routing, and structured output.
4. [`create_prompt.ipynb`](create_prompt.ipynb) demonstrates creating and retrieving prompts through Langfuse.

[`main.py`](main.py) is the default Python entry point generated for the project; the learning material currently lives in the notebooks.

## Requirements

- Python 3.13 or newer
- An OpenAI API key
- A Langfuse account and API keys
- `uv` or another Python environment/package manager

The dependencies are declared in [`pyproject.toml`](pyproject.toml), including LangChain, LangGraph, Langfuse, LangChain OpenAI integrations, and Wikipedia support.

## Setup

Create an environment and install the project dependencies:

```bash
uv sync
```

Configure the credentials expected by the notebooks in your local environment. A typical setup includes:

```bash
export OPENAI_API_KEY="your-openai-key"
export LANGFUSE_PUBLIC_KEY="your-langfuse-public-key"
export LANGFUSE_SECRET_KEY="your-langfuse-secret-key"
export LANGFUSE_HOST="https://cloud.langfuse.com"
```

Use the Langfuse host for your deployment region when it differs from the default. Never commit API keys or other secrets to the repository.

Open the notebooks in VS Code or Jupyter and run their cells from top to bottom. Model calls may incur provider charges, and the multi-agent research example may call Wikipedia.

## Core Tracing Pattern

The notebooks attach a Langfuse callback to a LangGraph run and scope trace attributes around the call:

```python
from langfuse import propagate_attributes
from langfuse.langchain import CallbackHandler

langfuse_handler = CallbackHandler()

with propagate_attributes(
	trace_name="LangGraph Time Agent",
	user_id="user-1233",
	session_id="session-abcd",
	tags=["langgraph", "multi-agent", "tool-calling"],
	metadata={"example": "multi-agent", "route": "current-time"},
):
	result = graph.invoke(
		{"messages": [HumanMessage(content="What time is it?")]},
		config={
			"callbacks": [langfuse_handler],
			"run_name": "answer-time-question",
		},
	)
```

Use `graph.stream(...)` when you want intermediate node outputs. Use `graph.invoke(...)` when you want the completed graph result without a streaming loop.

## Multi-Agent Architecture

The multi-agent example contains:

- A `Researcher` worker with a Wikipedia tool.
- A `CurrentTime` worker with a datetime tool.
- A supervisor chain that selects the next worker or returns `FINISH`.
- Conditional graph edges that route the supervisor's decision to the selected worker.
- Langfuse callbacks attached to the complete graph execution.

This illustrates a useful separation of responsibilities: the supervisor decides who should act, while each worker performs one focused task.

## Troubleshooting

- Authentication failures usually indicate a missing key or incorrect `LANGFUSE_HOST`.
- If notebook imports fail, run `uv sync` and select the environment created for this project in VS Code.
- If a tool call fails, inspect the tool's returned error and verify the relevant third-party package or network access.
- Keep notebook inputs and metadata free of secrets or sensitive personal information.
