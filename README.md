# 📚 MCP Starter Kit — PDF QA + Arithmetic Agent

A full-stack AI application that lets you **chat with your PDF documents** and perform **arithmetic calculations** — all powered by a LangGraph ReAct agent, Groq LLM, LlamaIndex RAG, and a FastMCP backend communicating over SSE.

Built with a clean **microservices architecture** using Docker Compose: a FastMCP backend server and a Streamlit frontend, connected via Server-Sent Events (SSE).

---

## ✨ Features

- 📄 **PDF RAG** — Upload any PDF and ask questions about its content using LlamaIndex + local HuggingFace embeddings
- ➕ **Arithmetic Tools** — Ask the agent to add, subtract, multiply, or divide via MCP tools
- 🤖 **ReAct Agent** — LangGraph agent with native tool-use (Groq `llama3-groq-8b-8192-tool-use-preview`)
- 🔌 **MCP over SSE** — Backend exposes tools via Model Context Protocol using Server-Sent Events
- 🐳 **Docker Compose** — One command to spin up the entire stack
- 🎨 **Dark Glassmorphism UI** — Custom Streamlit theme

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│                  User Browser                   │
│              http://localhost:8501              │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│         Frontend Container (Streamlit)          │
│  • LlamaIndex RAG (PDF ingestion + query)       │
│  • LangGraph ReAct Agent                        │
│  • langchain-mcp-adapters client                │
│              Port 8501                          │
└────────────────────┬────────────────────────────┘
                     │ SSE (http://backend:8000/sse)
┌────────────────────▼────────────────────────────┐
│         Backend Container (FastMCP)             │
│  • add / subtract / multiply / divide tools     │
│              Port 8000                          │
└─────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- A [Groq API Key](https://console.groq.com/keys) (free)

### 1. Clone the repository
```bash
git clone https://github.com/mananjp/MCP-starter.git
cd MCP-starter
```

### 2. Set up environment variables
```bash
cp .env.example .env
```
Open `.env` and add your Groq API key:
```env
GROQ_API_KEY=gsk_your_groq_api_key_here
```

### 3. Build and run
```bash
docker compose up --build
```

### 4. Open the app
Visit **http://localhost:8501** in your browser.

> The frontend waits for the backend healthcheck to pass before starting — you may see a short delay on first launch.

---

## 💬 Usage

1. **Upload a PDF** using the sidebar file uploader
2. **Ask questions** in the chat input — the agent will:
   - Search your PDF for relevant answers using RAG
   - Perform arithmetic calculations when asked
   - Combine both in a single response if needed

### Example Queries
```
"What are the main topics covered in this document?"
"What is 15% of 2400?"
"Summarize chapter 3 and multiply 48 by 7"
```

---

## 📁 Project Structure

```
MCP-starter/
├── streamlit_app.py          # Frontend — Streamlit UI + LangGraph agent
├── mcp_arithmetic_server.py  # Backend — FastMCP SSE server with arithmetic tools
├── requirements.txt          # Frontend Python dependencies
├── frontend.Dockerfile       # Frontend container build
├── backend.Dockerfile        # Backend container build
├── docker-compose.yml        # Multi-container orchestration
├── .streamlit/
│   └── config.toml           # Streamlit theme config
├── .env.example              # Environment variable template
├── .env                      # your secrets (gitignored, never commit)
└── .gitignore
```

---

## 🔧 Tech Stack

| Layer | Technology |
|---|---|
| **LLM** | Groq — `llama3-groq-8b-8192-tool-use-preview` |
| **Agent** | LangGraph `create_react_agent` |
| **MCP Client** | `langchain-mcp-adapters` |
| **MCP Server** | `FastMCP` over SSE transport |
| **RAG** | LlamaIndex + PyMuPDF + HuggingFace BGE embeddings |
| **Frontend** | Streamlit |
| **Containerization** | Docker Compose |

---

## ⚙️ Configuration

| Environment Variable | Description | Default |
|---|---|---|
| `GROQ_API_KEY` | Your Groq API key | **Required** |
| `MCP_SERVER_URL` | Backend SSE endpoint | `http://backend:8000/sse` |

---

## 🛠️ Development

### Run without Docker (local dev)

**Terminal 1 — Backend:**
```bash
pip install fastmcp
python mcp_arithmetic_server.py
```

**Terminal 2 — Frontend:**
```bash
pip install -r requirements.txt
MCP_SERVER_URL=http://localhost:8000/sse streamlit run streamlit_app.py
```

### Useful Docker Commands
```bash
# View all logs
docker compose logs -f

# View only backend logs
docker compose logs -f backend

# Check container status
docker compose ps

# Test backend SSE endpoint
curl http://localhost:8000/sse

# Stop all containers
docker compose down

# Rebuild after code changes
docker compose up --build
```

---

## 🧩 Adding New MCP Tools

Open `mcp_arithmetic_server.py` and add a new decorated function:

```python
@mcp.tool()
def power(base: float, exponent: float) -> str:
    return f"Result: {base ** exponent}"
```

Rebuild the backend container:
```bash
docker compose up --build backend
```

The new tool is automatically discovered by the frontend agent — no frontend changes needed.

---

## ☁️ Deployment (Railway)

1. Push your repo to GitHub (ensure `.env` is gitignored)
2. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub**
3. Add **two services** from the same repo:
   - **Backend**: Dockerfile = `backend.Dockerfile`, add env var `PORT=8000`
   - **Frontend**: Dockerfile = `frontend.Dockerfile`, add env vars:
     ```
     GROQ_API_KEY=your_key_here
     MCP_SERVER_URL=https://your-backend.up.railway.app/sse
     ```
4. Generate a public domain for the frontend service
5. Every `git push` to `master` triggers an automatic redeploy

---

## 🔒 Security Notes

- **Never commit your `.env` file** — it is gitignored by default
- **Rotate your Groq API key** if it was ever accidentally pushed to GitHub
- Use `.env.example` as a template — it contains only placeholder values

---

## 📄 License

MIT License — feel free to fork, extend, and build on top of this starter kit.

---

## 🙏 Acknowledgements

Built with love using [LlamaIndex](https://www.llamaindex.ai/), [LangChain](https://langchain.com/), [FastMCP](https://github.com/jlowin/fastmcp), [Groq](https://groq.com/), and [Streamlit](https://streamlit.io/).
