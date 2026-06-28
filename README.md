# 🧠 NipunGPT - Agentic AI Chatbot

NipunGPT is a premium-designed, feature-rich agentic chatbot built on top of **FastAPI**, **LangGraph**, and the **Gemini 2.5** suite of models. It is designed to act as an advanced AI assistant similar to ChatGPT, capable of executing complex workflows, accessing long-term memory, performing math calculations, conducting web searches, and running RAG (Retrieval-Augmented Generation) on uploaded documents.

---

## 📸 Output Showcase

![NipunGPT Screenshot](screenshot.jpg)

---

## 🖼️ Architecture & Workflow

The architecture is split into a lightweight, streaming frontend, a FastAPI routing layer, and an agentic orchestrator built on LangGraph that leverages SQLite checkpointing.

```mermaid
graph TD
    A[Frontend: index.html] <-->|HTTP / SSE Streaming| B[FastAPI Backend: app.py]
    B <-->|Load/Save History| C[(SQLite DB: chatbot_memory.db)]
    B -->|Build/Execute Agent| D[LangGraph Orchestrator: agent.py]
    D <-->|Read/Write Checkpoints| E[(SQLite Checkpointer: langgraph_checkpoints.sqlite)]
    
    subgraph Agent Tools
        D -->|Use Calculator| F[Calculator Tool: tools.py]
        D -->|Search Web| G[Tavily Search API]
        D -->|Long Term Memory| H[SQLAlchemy Memory Helpers: database.py]
        D -->|Query RAG| I[RAG Retrieval Engine: rag.py]
    end

    B -->|Upload Document| J[Document Loader & Chunking: rag.py]
    J -->|Generate Embeddings| K[Google AI Embedding API]
    K -->|Persist Embeddings| L[(Chroma Vector DB)]
    I <-->|Fetch Chunks| L
```

---

## 📂 Project Structure

```text
Nipungpt/
│
├── app.py                  # FastAPI server and streaming chat endpoints
├── agent.py                # LangGraph agent setup and tool orchestration
├── database.py             # Conversation, history, and memory persistence logic
├── rag.py                  # Document ingestion, text chunking, and RAG logic
├── tools.py                # Agent tools (web search, memory, RAG, calculator)
├── requirements.txt        # Python dependencies
├── Dockerfile              # Docker image configuration
├── .dockerignore           # Docker ignore rules
│
├── templates/
│   └── index.html          # Responsive frontend UI
│
├── uploads/                # Directory for uploaded documents
├── data/                   # SQLite database and app data checkpoints
└── chroma_db/              # ChromaDB vector database storage
```

---

## ⚙️ Implementation & How It Works

### 1. The Frontend (Client Side)
* Communicates with FastAPI endpoints using `fetch` (for uploads/history) and standard HTTP requests.
* Implements **SSE (Server-Sent Events)** to stream chatbot replies word-by-word.
* Utilizes CSS micro-animations to indicate when a tool (e.g. Web Search or Calculator) is actively processing.
* Built-in browser-native **SpeechRecognition** for voice dictation.

### 2. The Backend Routing (`app.py`)
* Serves the HTML frontend and processes user queries, document uploads, and thread histories.
* Handles file uploads, stores them securely in `uploads/`, calls `add_document_to_rag` to chunk text, and updates the vector database.
* Converts LangGraph messages to an SSE generator payload using custom filters to prevent raw JSON/tool calls from rendering to the user.

### 3. The Orchestration Layer (`agent.py`)
* Standardizes on a unified System Prompt instructing the agent to choose tools intelligently based on user intent.
* Compiles a LangGraph `StateGraph` containing `chatbot` and `tools` nodes.
* Connects a `SqliteSaver` to automatically handle thread checkpointing.

---

## 🚀 Run Locally

### 1. Set Up Environment Variables
Create a `.env` file in the root folder:
```env
GOOGLE_API_KEY="your-gemini-api-key"
TAVILY_API_KEY="your-tavily-api-key"
GOOGLE_MODEL="gemini-2.5-flash"
```

### 2. Run the App
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

The application will be available at: **`http://127.0.0.1:8080`**

---

## 🐳 Docker Deployment

1. **Build the Docker Image:**
   ```bash
   docker build -t nipungpt .
   ```
2. **Run the Docker Container:**
   ```bash
   docker run -d \
     --name nipungpt \
     --restart always \
     -p 8080:8080 \
     --env-file .env \
     nipungpt
   ```

The application will be available at: **`http://localhost:8080`**

---

## ☁️ AWS CI/CD Deployment with GitHub Actions

This project can be deployed continuously to AWS using **GitHub Actions**, **Amazon ECR**, **Amazon EC2**, and a **GitHub Self-Hosted Runner**.

### 1. Create an IAM User
Create an IAM user for deployment in the AWS Console and attach the following policies:
* `AmazonEC2ContainerRegistryFullAccess`
* `AmazonEC2FullAccess`

*(Save the Access Key ID and Secret Access Key for later).*

### 2. Create an ECR Repository
Create a private repository in Amazon ECR. 
* Example ECR Image URI: `315865595366.dkr.ecr.us-east-1.amazonaws.com/nipungpt`
* Save **only** the repository name for your secrets: `ECR_REPO=nipungpt`

### 3. Create an EC2 Instance
Launch an Ubuntu EC2 instance. In the instance's Security Group, add an Inbound Rule:
* **Type**: Custom TCP
* **Port**: `8080`
* **Source**: `0.0.0.0/0` (or restricted to your IP address)

### 4. Install Docker on EC2
Connect to your EC2 instance via SSH and run:
```bash
sudo apt-get update -y
sudo apt-get upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add the Ubuntu user to the Docker group
sudo usermod -aG docker ubuntu
newgrp docker

# Verify Docker installation
docker --version
```

### 5. Configure EC2 as a GitHub Self-Hosted Runner
1. In your GitHub repository, go to **Settings** → **Actions** → **Runners** → **New self-hosted runner**.
2. Select **Linux** and execute the commands provided in the EC2 terminal.
3. Once configured, start the runner:
   ```bash
   ./run.sh
   ```
4. *Optional (Run as a service for persistence):*
   ```bash
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

### 6. Add GitHub Secrets
Add the following secrets under **Settings** → **Secrets and variables** → **Actions** → **New repository secret**:

| Secret Name | Example Value | Description |
|---|---|---|
| `AWS_ACCESS_KEY_ID` | `AKIAIOSFODNN7EXAMPLE` | AWS Access Key ID |
| `AWS_SECRET_ACCESS_KEY` | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` | AWS Secret Access Key |
| `AWS_DEFAULT_REGION` | `us-east-1` | AWS Region |
| `ECR_REPO` | `nipungpt` | ECR repository name only |
| `GOOGLE_API_KEY` | `AIzaSy...` | Gemini API Key |
| `GOOGLE_MODEL` | `gemini-2.5-flash` | LLM model version |
| `TAVILY_API_KEY` | `tvly-...` | Web search API Key |
| `LANGSMITH_TRACING` | `true` | Enable LangSmith tracing |
| `LANGSMITH_ENDPOINT` | `https://api.smith.langchain.com` | LangSmith API Endpoint |
| `LANGSMITH_API_KEY` | `lsv2_pt_...` | LangSmith API Key |
| `LANGSMITH_PROJECT` | `Nipungpt` | LangSmith Project Name |

---

## 🎯 Usage

After running locally or deploying to AWS:
1. Open the URL in your browser.
2. Select a Gemini model dropdown (e.g. `gemini-2.5-flash` or `gemini-2.5-pro`).
3. Click the attachment paperclip icon `📎` to upload files, then search or ask questions about them.
4. Try using voice dictation via the microphone icon `🎙️`.
5. Ask current event queries or mathematical expressions to watch the agent dynamically switch tools.

### Example Queries to Try:
* *"Remember that my name is Nipun."* (Saves preference to long-term memory).
* *"What is my name?"* (Recalls preference from memory).
* *"Search the web for the latest Gemini 2.5 model news."* (Triggers Tavily Web Search).
* *"Calculate 125 * 48 / 6."* (Triggers the calculator).
* *"Summarize the uploaded file."* (Triggers the document RAG retrieval).

---

## 📝 Important Notes

* **Security**: Never commit your `.env` file or local databases (`chatbot_memory.db`, `langgraph_checkpoints.sqlite`) to GitHub. They are ignored by default.
* **Production Configurations**: For production environments, disable `reload=True` in your uvicorn runner inside `app.py`.
* **EC2 Firewall**: If the page fails to load, ensure Port `8080` is allowed in the Security Group inbound rules.

---

## 🤝 Contributing

1. Fork the repository.
2. Create your feature branch (`git checkout -b feature/AmazingFeature`).
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`).
4. Push to the branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request.

---

## 📄 License

This project is open-source. Refer to the repository license for usage terms.
