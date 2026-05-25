# Advanced ReAct AI Agent

A production-ready AI agent built on the **ReAct (Reason + Act)** framework using LangGraph, LangChain, and Groq LLMs. The agent reasons through problems step by step, selects the right tools, chains them together when needed, and maintains memory across conversations.

This project is shared to document a working, end-to-end ReAct implementation — the kind of practical setup that's often missing from tutorials.

---

## What This Agent Can Do

- **Multi-tool reasoning** — chains tools together across multiple steps to solve complex problems
- **Math** — delegates arithmetic and expressions to safe, validated math tools instead of hallucinating answers
- **Web & academic search** — Tavily for real-time web results, ArXiv for research papers, Wikipedia for general knowledge
- **Persistent memory** — conversations are stored in SQLite and survive restarts; in-memory option available for fast, session-scoped use
- **Multi-turn dialogue** — the agent remembers context within a thread, so follow-up questions just work
- **Graceful error handling** — bad inputs, division by zero, API failures — none of these crash the agent
- **Streaming support** — observe the agent's reasoning and tool calls as they happen

---

## Architecture

```
User Query
    │
    ▼
┌──────────────────────────────────────────┐
│           ReAct Agent Graph              │
│                                          │
│  ┌────────────┐   tool_calls?  ┌───────────────┐ │
│  │  LLM Node  │ ─────────────► │   Tool Node   │ │
│  │(w/ system  │ ◄───────────── │  (10 tools)   │ │
│  │  prompt)   │  tool results  └───────────────┘ │
│  └────────────┘                                  │
│        │ no tool_calls                           │
│        ▼                                         │
│       END                                        │
└──────────────────────────────────────────┘
         │
         ▼
  SQLite Memory (persistent across restarts)
```

The graph is built with **LangGraph's StateGraph** — a node for LLM reasoning, a node for tool execution, and conditional routing between them. This loop continues until the LLM produces a final answer with no further tool calls.

---

## Tools Available

| Tool | Purpose |
|---|---|
| `tavily_tool` | Real-time web search (requires `TAVILY_API_KEY`) |
| `arxiv_tool` | Search academic papers on ArXiv |
| `wiki_tool` | Wikipedia lookups |
| `add` / `subtract` / `multiply` / `divide` | Basic arithmetic |
| `power` | Exponentiation |
| `calculate` | Safe evaluation of math expressions (`sqrt`, `sin`, `log`, etc.) |
| `get_current_datetime` | Current UTC date and time |

---

## Tech Stack

- [LangGraph](https://github.com/langchain-ai/langgraph) — agent graph and state management
- [LangChain](https://github.com/langchain-ai/langchain) — tool abstractions and LLM integrations
- [Groq](https://groq.com/) — LLM inference (`llama-3.3-70b-versatile` as primary, `gemma2-9b-it` as fallback)
- [Tavily](https://tavily.com/) — web search API
- SQLite — persistent conversation memory

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/Yashraj2007/Advanced-ReAct-AI-Agent.git
cd Advanced-ReAct-AI-Agent
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Set environment variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key       # optional — web search degrades gracefully without this
LANGCHAIN_API_KEY=your_langsmith_key     # optional — enables LangSmith tracing
```

> Get a free Groq API key at [console.groq.com](https://console.groq.com). Tavily offers a free tier at [tavily.com](https://tavily.com).

### 4. Run

Open the notebook in Jupyter or Google Colab and run cells sequentially. The agent, tools, and memory are all initialised within the notebook.

---

## Usage Examples

```python
# Single query
run_agent("What is the square root of 1764, and what is today's date?")

# Multi-turn conversation (agent remembers context)
run_agent("What is 17 multiplied by 13?", thread_id="my_thread")
run_agent("Now add 500 to that result.", thread_id="my_thread")

# Persistent memory (survives restarts)
run_agent("Remember my budget is $50,000.", graph=persistent_agent, thread_id="project_session")
run_agent("What is 15% of my budget?",     graph=persistent_agent, thread_id="project_session")

# Streaming
stream_agent("Find recent papers on Retrieval Augmented Generation.")
```

---

## Project Structure

```
Advanced-ReAct-AI-Agent/
├── react_agent.ipynb       # Main notebook — all code, demos, and explanations
├── requirements.txt        # All dependencies
├── agent_memory.db         # SQLite memory file (auto-created on first run)
├── .env                    # API keys (not committed)
├── .gitignore
└── README.md
```

---

## Key Design Decisions

**Why LangGraph over vanilla LangChain agents?**
LangGraph gives explicit control over the agent loop — you define the graph, the routing logic, and the state schema. This makes it far easier to reason about, debug, and extend compared to black-box agent executors.

**Why delegate math to tools?**
LLMs hallucinate arithmetic. Every numeric operation goes through a validated Python tool, so the output is reliable.

**Why SQLite for memory?**
It's zero-infrastructure — no external server, no config. For a project focused on agent architecture, SQLite keeps the memory layer simple and self-contained.

**Why a fallback LLM?**
API outages happen. The agent falls back to `gemma2-9b-it` automatically if the primary model fails, without crashing the whole session.

---

## What Could Be Added Next

- A custom RAG tool querying a vector database (Chroma, Pinecone)
- Human-in-the-loop approval before tool execution
- Async execution with `graph.ainvoke()`
- A FastAPI wrapper for deployment as a REST service
- Multi-agent setups where specialised agents are composed as sub-graphs

---

## License

MIT — use it, modify it, build on it.

---

## Author

[Yashraj2007](https://github.com/Yashraj2007) — built and shared as a learning resource. If something is unclear or could be explained better, feel free to open an issue.
