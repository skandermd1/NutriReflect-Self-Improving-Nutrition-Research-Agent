# 🧠 NutriReflect — Self-Improving Nutrition Research Agent

A LangGraph-powered **Reflexion Agent** that answers nutrition questions by generating an initial response, critiquing itself, searching the web for evidence, and iteratively revising its answer — all using a local Ollama LLM.

---

## 📋 Overview

NutriReflect uses the **Reflexion pattern** (Shinn et al., 2023) to simulate how a researcher would approach a question: form an opinion, identify gaps, search for evidence, and revise. The agent alternates between two expert personas:

- **Dr. Paul Saladino (Carnivore MD)** — generates the initial bold, evolutionary-based answer
- **Dr. Peter Attia** — revises it with clinical rigor, citations, and nuance

The result is a self-improving answer that gets more evidence-based with each iteration.

---

## 🏗️ Architecture

```
User Question
      │
      ▼
┌─────────────┐
│   respond   │  ← Initial answer (Dr. Saladino persona)
│             │     Outputs: Answer + Reflexion + Search Queries
└──────┬──────┘
       │
       ▼
┌──────────────────┐
│  execute_tools   │  ← Runs Tavily web searches from queries
│                  │     Returns search results as ToolMessages
└──────┬───────────┘
       │
       ▼
┌─────────────┐
│   revisor   │  ← Revised answer (Dr. Attia persona)
│             │     Adds citations, removes speculation
└──────┬──────┘
       │
   (event_loop)
       ├── iterations < MAX_ITERATIONS → back to "execute_tools"
       └── iterations >= MAX_ITERATIONS → END
```

---

## ⚙️ Prerequisites

- Python 3.10+
- [Ollama](https://ollama.com) installed and running locally
- A Tavily API key — get one free at [tavily.com](https://tavily.com)

---

## 📦 Installation

1. **Clone the repository:**
   ```bash
   git clone <your-repo-url>
   cd NutriReflect
   ```

2. **Install dependencies:**
   ```bash
   pip install langgraph langchain-ollama langchain-openai langchain-community langchain-core pydantic tavily-python
   ```

3. **Pull a model in Ollama:**
   ```bash
   ollama pull llama3.1     # recommended
   # or
   ollama pull mistral
   ollama pull phi3
   ```

4. **Start Ollama:**
   ```bash
   ollama serve
   ```

---

## 🔧 Configuration

Set your Tavily API key as an environment variable:

```bash
# Linux / macOS
export TAVILY_API_KEY="tvly-your-key-here"

# Windows PowerShell
$env:TAVILY_API_KEY="tvly-your-key-here"
```

To change the LLM model, update this line in the script:
```python
llm = ChatOllama(model="llama3.1", temperature=0.2)
#                       ↑ change to mistral, phi3, etc.
```

To change the number of revision iterations:
```python
MAX_ITERATIONS = 2  # increase for more thorough research
```

---

## 🚀 Usage

```bash
python agent.py
```

On Windows without Python in PATH:
```powershell
C:\Users\<you>\AppData\Local\Programs\Python\Python313\python.exe agent.py
```

---

## 🔄 Graph Nodes

| Node | Persona | Description |
|---|---|---|
| `respond` | Dr. Paul Saladino | Generates initial answer + reflexion + search queries |
| `execute_tools` | — | Runs Tavily searches and returns results |
| `revisor` | Dr. Peter Attia | Revises answer with citations and clinical nuance |
| `event_loop` | — | Router — loops or ends based on iteration count |

---

## 🧠 Reflexion Pattern

Each iteration follows this cycle:

```
Generate → Critique (what's missing / superfluous)
        → Search   (Tavily web queries)
        → Revise   (cite sources, remove speculation)
        → Repeat or END
```

This is inspired by the **Reflexion paper (Shinn et al., 2023)** which showed that LLMs improve significantly when given the ability to self-critique and retry with external feedback.

---

## 🧩 State Schema

```python
class AnswerQuestion(BaseModel):
    Answer: str               # Main response
    reflexion: Reflexion      # Missing and superfluous ideas
    search_queries: List[str] # 1-3 Tavily search queries

class ReviseAnswer(AnswerQuestion):
    references: List[str]     # Peer-reviewed citations
```

---

## ⚠️ Known Issues in Original Code

| Issue | Fix |
|---|---|
| `_set_if_undefined("tvly-dev-...")` passes the API key as the variable name | Use `os.environ["TAVILY_API_KEY"] = "tvly-..."` instead |
| `llm.bind_tools([tavily_tool])` — Ollama does not support OpenAI-style tool binding | Use `llm.with_structured_output(AnswerQuestion)` instead |
| `MAX_ITERATIONS` referenced before assignment | Define it before the graph |

---

## 📁 Project Structure

```
NutriReflect/
├── agent.py      # Main script
└── README.md     # This file
```

---

## 📄 License

MIT — free to use and modify.
