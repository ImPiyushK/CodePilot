# CodePilot: Autonomous Software Engineering Agent

> A multi-agent AI system that plans, generates, tests, validates, and iteratively improves Python code using **LangGraph, Gemini, RAG, and automated execution**.

## 🚀 Overview

**CodePilot** is an autonomous software engineering agent designed to transform natural-language programming tasks into tested and executable Python solutions.

Instead of relying on a single LLM call:

```text
User → LLM → Code
```

CodePilot uses a **multi-agent, closed-loop workflow**:

```text
                    ┌─────────────────┐
                    │    User Task    │
                    └────────┬────────┘
                             ↓
                    ┌─────────────────┐
                    │  Planner Agent  │
                    │ Task Decompose  │
                    └────────┬────────┘
                             ↓
                    ┌─────────────────┐
                    │   Coder Agent   │◄──────────┐
                    │ Code Generation │           │
                    └────────┬────────┘           │
                             ↓                    │
                    ┌─────────────────┐           │
                    │   Tester Agent  │           │
                    │ Pytest Creation │           │
                    └────────┬────────┘           │
                             ↓                    │
                    ┌─────────────────┐           │
                    │Security Scanner │           │
                    └────────┬────────┘           │
                             ↓                    │
                    ┌─────────────────┐           │
                    │ Code Executor   │           │
                    │  + Pytest       │           │
                    └────────┬────────┘           │
                             │                    │
                     ┌───────┴────────┐           │
                     ↓                ↓           │
                   PASS              FAIL         │
                     │                │            │
                     ↓                └────────────┘
                  Complete          Self-Correction
```

The system maintains workflow state using **LangGraph**, retrieves relevant programming documentation using **RAG**, validates generated code through security checks, and uses execution feedback to improve failed implementations.

## ✨ Features

### 🤖 Multi-Agent Architecture

- **Planner Agent** — decomposes the user's task into subtasks.
- **Coder Agent** — generates and modifies Python code.
- **Tester Agent** — generates pytest test cases.
- **Security Scanner** — checks generated code for potentially dangerous operations.
- **Code Executor** — executes the generated solution and tests.
- **Human Escalation** — stops autonomous retries after repeated failures.

### 🔄 Self-Correction

```text
Generate Code
      ↓
Generate Tests
      ↓
Security Check
      ↓
Execute
      ↓
   ┌──┴──┐
   ↓     ↓
 PASS   FAIL
   │     │
   ↓     ↓
 DONE   Error
          ↓
     Coder Agent
          ↓
     Regenerate
```

When execution fails, the error output is stored in the shared agent state and supplied to the Coder Agent during the next attempt.

## 🧠 LangGraph Workflow

CodePilot uses a **LangGraph state machine**.

```text
Planner
   │
   ▼
Coder
   │
   ▼
Tester
   │
   ▼
Security Scanner
   │
   ├──────── Security Failure ───────► Coder
   │
   ▼
Executor
   │
   ├──────── Test Failure ───────────► Coder
   │
   ├──────── Success ────────────────► Next Subtask
   │
   └──────── Max Retries ────────────► Escalation
```

## 📚 Retrieval-Augmented Generation

CodePilot uses **Retrieval-Augmented Generation (RAG)** to provide the Coder Agent with relevant programming documentation.

### Indexing

```text
Documentation
     ↓
Chunking
     ↓
Gemini Embeddings
     ↓
Pinecone Vector Database
```

Current documentation includes topics such as:

- Python collections
- JSON
- OS and pathlib
- Regular expressions

### Retrieval

```text
Current Subtask
      ↓
Gemini Embedding
      ↓
Pinecone Similarity Search
      ↓
Top-K Documentation
      ↓
Coder Agent
```

## 🛡️ Security Validation

CodePilot performs a lightweight static security scan before execution.

The scanner checks for potentially dangerous patterns including:

```text
os.system()
subprocess.call()
subprocess.Popen()
eval()
exec()
__import__()
shutil.rmtree()
rm -rf
```

> **Security disclaimer:** The current scanner is a lightweight pattern-based defense, not a complete security sandbox. Production deployment should use container or microVM isolation, resource limits, network controls, and filesystem restrictions.

## 🧪 Automated Testing

The Tester Agent generates pytest tests for the generated solution.

The Executor runs:

```bash
python -m pytest tests_sol.py -v --tb=short
```

Failed test output is passed back to the Coder Agent for correction.

## 📊 Observability

CodePilot records basic LLM execution metrics:

- Agent execution latency
- Token usage
- Retry count
- Execution status

## 🌐 Web Interface

CodePilot includes a Flask-based web interface supporting:

- Task submission
- Agent execution visualization
- Real-time status updates
- Generated code display
- Test results
- Error messages
- Execution metrics

Real-time updates use **Server-Sent Events (SSE)**.

## 🏗️ Project Structure

```text
CodePilot/
│
├── app.py                         # Flask web application
├── main.py                        # CLI entry point
├── requirements.txt               # Python dependencies
│
├── context/
│   └── Agent_Graph.md             # Agent workflow documentation
│
├── docs/                          # RAG knowledge base
│   ├── collections.txt
│   ├── json.txt
│   ├── os_pathlib.txt
│   └── re_regex.txt
│
├── graph/
│   ├── __init__.py
│   ├── graph.py                   # LangGraph workflow
│   ├── nodes.py                   # Agent nodes
│   └── state.py                   # Shared agent state
│
├── prompts/
│   ├── planner.md                 # Planner prompt
│   ├── coder.md                   # Coder prompt
│   └── tester.md                  # Tester prompt
│
├── rag/
│   ├── __init__.py
│   ├── indexer.py                 # Build Pinecone index
│   └── retriever.py               # Retrieve documentation
│
├── tools/
│   ├── __init__.py
│   ├── executor.py                # Code/test execution
│   └── scanner.py                 # Security scanner
│
└── templates/
    ├── index.html                 # Web UI
    └── index_backup.html
```

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| Language | Python |
| Agent Orchestration | LangGraph |
| LLM | Google Gemini |
| LLM Framework | LangChain |
| Vector Database | Pinecone |
| Embeddings | Gemini Embeddings |
| Testing | Pytest |
| Backend | Flask |
| Streaming | Server-Sent Events |
| Configuration | python-dotenv |

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd CodePilot
```

### 2. Create a virtual environment

#### Windows

```bash
python -m venv .venv
.venv\Scripts\activate
```

#### Linux / macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

## 🔑 Environment Variables

Create a `.env` file in the project root:

```env
GEMINI_API_KEY=your_gemini_api_key
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_HOST=your_pinecone_index_host
```

> **Do not commit `.env` or API keys to GitHub.**

## 📚 Build the RAG Index

Before using documentation retrieval:

```bash
python rag/indexer.py
```

The indexer:

1. Reads documentation files.
2. Splits documents into chunks.
3. Generates embeddings.
4. Stores vectors and metadata in Pinecone.

## ▶️ Running CodePilot

### CLI Mode

```bash
python main.py
```

Then enter a programming task:

```text
Create a Python function that determines whether
a string is a palindrome.
```

CodePilot automatically:

```text
Plan
 ↓
Generate Code
 ↓
Generate Tests
 ↓
Security Scan
 ↓
Execute
 ↓
Correct if Necessary
 ↓
Return Result
```

### Web Mode

```bash
python app.py
```

Open:

```text
http://localhost:5000
```

## 🔬 Example

For a request such as:

```text
Create a Python function that calculates the Fibonacci
sequence up to n terms.
```

the workflow is:

```text
User Request
     │
     ▼
Planner Agent
     │
     ▼
Coder Agent
     │
     ▼
Tester Agent
     │
     ▼
Security Scanner
     │
     ▼
Code Executor
     │
 ┌───┴────┐
 ↓        ↓
PASS     FAIL
 │        │
 ↓        ↓
Done    Feedback
           │
           └──────► Coder
```

## 🎯 Why CodePilot?

Traditional LLM-based coding:

```text
Prompt → LLM → Code
```

provides no guaranteed verification mechanism.

CodePilot introduces an engineering feedback loop:

```text
Plan
  ↓
Generate
  ↓
Test
  ↓
Validate
  ↓
Execute
  ↓
Observe
  ↓
Correct
  ↓
Verify
```

> **Don't trust generated code—generate, verify, observe, and improve it.**

## 📈 Future Work

- Repository-level code understanding and multi-file modification
- Docker/microVM-based execution sandbox
- Independent code review agent
- Benchmarking using Pass@1 / Pass@k, success rate, retries, latency, and token usage
- Persistent repository-specific agent memory
- Improved AST-based security analysis

## ⚠️ Limitations

CodePilot is currently an **experimental research/engineering prototype**.

- Primarily focused on Python programming tasks.
- Generated code is executed using a local subprocess.
- Security scanning is pattern-based.
- RAG quality depends on indexed documentation.
- Generated tests may contain incorrect assumptions.
- Full repository-level modification is not currently implemented.
- Performance depends on the underlying LLM.

## 🎓 Project Highlights

CodePilot demonstrates practical implementation of:

- Agentic AI
- Multi-agent orchestration
- Stateful workflows
- LangGraph
- LLM-based code generation
- Retrieval-Augmented Generation
- Vector search
- Automated testing
- Self-correction
- Security validation
- Tool execution
- LLM observability
- Human-in-the-loop escalation

## 👨‍💻 Author

**Piyush Kandwal**

M.Tech — Computer Science & Engineering  
IIIT-Delhi

## 📄 License

This project is intended for educational and research purposes.

If you plan to distribute the project for reuse, consider adding an appropriate open-source license such as the MIT License.
