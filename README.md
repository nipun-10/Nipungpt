# 🧠 NipunGPT - Agentic AI Chatbot

An advanced, premium-designed AI chatbot powered by **LangGraph** and **Gemini 2.5**. It supports real-time web searches, calculator functions, long-term conversation memory, and RAG document queries.

---

## 📸 Output Showcase

> Add a screenshot or recording of your running chatbot here to showcase the UI!

![NipunGPT Screenshot](https://raw.githubusercontent.com/username/repository/main/placeholder-screenshot.png) 
*(To update, replace this image URL or place a screenshot file in your repository and update the path above, e.g., `![NipunGPT](screenshot.png)`)*

---

## ✨ Features

* **🤖 Smart Agent (LangGraph)**: Automatically decides when to answer directly or when to run tools.
* **🌐 Web Search (Tavily)**: Searches the internet for up-to-date events.
* **📂 Document RAG**: Upload `.pdf`, `.docx`, `.txt`, `.md`, `.py`, or `.csv` files and ask questions about them.
* **🧠 Persistent Memory**: Remembers your preferences and details across sessions.
* **🧮 Calculator**: Solves math queries instantly.
* **🎙️ Voice Dictation**: Speech-to-text dictation in the frontend.
* **🎨 Premium UI**: Beautiful dark-mode interface with streaming responses (SSE).

---

## 🛠️ Tech Stack

* **Frontend**: HTML5, Vanilla CSS3, Javascript (SSE Streaming)
* **Backend**: FastAPI, Uvicorn
* **Orchestration**: LangGraph, LangChain
* **Database**: SQLite (SQLAlchemy), ChromaDB (Vector Store)
* **LLM**: Gemini 2.5 Flash / Pro (via Google GenAI)

---

## 🚀 Quick Start

### 1. Set Up Environment Variables
Create a file named `.env` in the root folder:
```env
GOOGLE_API_KEY="your-gemini-api-key"
TAVILY_API_KEY="your-tavily-api-key"
```

### 2. Run Locally
* **Windows (PowerShell):**
  ```powershell
  .venv\Scripts\Activate.ps1
  python app.py
  ```
* **macOS / Linux:**
  ```bash
  source .venv/bin/activate
  python app.py
  ```

Open **`http://127.0.0.1:8080`** in your browser.

---

## 🐳 Docker Setup

```bash
# Build image
docker build -t nipungpt .

# Run container
docker run -d -p 8080:8080 --env-file .env nipungpt
```
