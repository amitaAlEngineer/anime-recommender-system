# 🎌 Anime Recommender System

> LLM-Powered Anime Recommendations using RAG, Streamlit, Docker & Kubernetes

![Python](https://img.shields.io/badge/Python-3.12-blue)
![LangChain](https://img.shields.io/badge/LangChain-0.2.16-green)
![ChromaDB](https://img.shields.io/badge/ChromaDB-0.5.5-orange)
![Streamlit](https://img.shields.io/badge/Streamlit-1.37.1-red)
![Docker](https://img.shields.io/badge/Docker-Enabled-2496ED)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Enabled-326CE5)

---

## 📖 Overview

The Anime Recommender System is an end-to-end LLM-powered application that uses **Retrieval-Augmented Generation (RAG)** to recommend anime titles based on natural language user preferences.

- 🔍 **Semantic Search** via ChromaDB + HuggingFace Embeddings
- 🤖 **LLM Response** via Groq (LLaMA / Mixtral)
- 🧠 **RAG Orchestration** via LangChain
- 🖥️ **Interactive UI** via Streamlit
- 🐳 **Containerized** with Docker
- ☸️ **Deployable** on Kubernetes

---

## 🏗️ Architecture

```
User Query
    │
    ▼
Streamlit UI (app/app.py)
    │
    ▼
AnimeRecommendationPipeline
    │
    ├──► ChromaDB Retriever (semantic search over anime dataset)
    │
    └──► Groq LLM (generates 3 structured recommendations)
```

### Tech Stack

| Component        | Technology                        |
|------------------|-----------------------------------|
| Frontend UI      | Streamlit 1.37.1                  |
| LLM Backend      | Groq (LLaMA3 / Mixtral)           |
| Orchestration    | LangChain 0.2.16                  |
| Vector Store     | ChromaDB 0.5.5                    |
| Embeddings       | HuggingFace all-MiniLM-L6-v2      |
| Containerization | Docker                            |
| Deployment       | Kubernetes                        |
| Package Manager  | uv                                |

---

## 📁 Project Structure

```
anime-recommender-system/
├── app/
│   └── app.py                  # Streamlit UI
├── pipeline/
│   ├── pipeline.py             # Main RAG pipeline
│   └── build_pipeline.py       # Vector store builder script
├── src/
│   ├── vector_store.py         # ChromaDB vector store logic
│   └── recommender.py          # LangChain RAG chain
├── config/
│   └── config.py               # Config & environment variables
├── utils/
│   ├── logger.py               # Logging utility
│   └── custom_exception.py     # Custom error handling
├── data/                       # Anime CSV dataset
├── chroma_db/                  # Persisted vector store (auto-generated)
├── logs/                       # Application logs
├── Dockerfile
├── llmops-k8s.yaml             # Kubernetes manifests
├── pyproject.toml
├── requirements.txt
└── setup.py
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.12+
- `uv` package manager (recommended) or `pip`
- Docker (for containerized deployment)
- Kubernetes + `kubectl` (for k8s deployment)
- Groq API key — get one free at [console.groq.com](https://console.groq.com)

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/anime-recommender-system.git
cd anime-recommender-system
```

### 2. Set Up Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
MODEL_NAME=llama3-8b-8192
```

### 3. Install Dependencies

**Using uv (Recommended):**

```bash
uv pip install -r requirements.txt
uv pip install -e .
```

**Using pip:**

```bash
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
pip install -e .
```

### 4. Build the Vector Store

Run this **once** to embed the anime dataset into ChromaDB:

```bash
python pipeline/build_pipeline.py
```

### 5. Run the App

```bash
# With uv
uv run streamlit run app/app.py

# With pip venv
streamlit run app/app.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## 🐳 Docker Deployment

### Build the Image

```bash
docker build -t anime-recommender:latest .
```

### Run the Container

```bash
docker run -p 8501:8501 \
  -e GROQ_API_KEY=your_groq_api_key \
  -e MODEL_NAME=llama3-8b-8192 \
  anime-recommender:latest
```

### Sample Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
RUN pip install -e .

# Build vector store at image build time
RUN python pipeline/build_pipeline.py

EXPOSE 8501

CMD ["streamlit", "run", "app/app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

---

## ☸️ Kubernetes Deployment

### 1. Push Image to Registry

```bash
docker tag anime-recommender:latest your-registry/anime-recommender:latest
docker push your-registry/anime-recommender:latest
```

### 2. Create Secrets

```bash
kubectl create secret generic anime-recommender-secrets \
  --from-literal=GROQ_API_KEY=your_groq_api_key \
  --from-literal=MODEL_NAME=llama3-8b-8192
```

### 3. Apply Manifests

```bash
kubectl apply -f llmops-k8s.yaml
```

### 4. Check Deployment

```bash
kubectl get pods
kubectl get services
kubectl logs -f deployment/anime-recommender
```

### Sample `llmops-k8s.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anime-recommender
spec:
  replicas: 2
  selector:
    matchLabels:
      app: anime-recommender
  template:
    metadata:
      labels:
        app: anime-recommender
    spec:
      containers:
      - name: anime-recommender
        image: your-registry/anime-recommender:latest
        ports:
        - containerPort: 8501
        env:
        - name: GROQ_API_KEY
          valueFrom:
            secretKeyRef:
              name: anime-recommender-secrets
              key: GROQ_API_KEY
        - name: MODEL_NAME
          valueFrom:
            secretKeyRef:
              name: anime-recommender-secrets
              key: MODEL_NAME
---
apiVersion: v1
kind: Service
metadata:
  name: anime-recommender-service
spec:
  selector:
    app: anime-recommender
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8501
  type: LoadBalancer
```

---

## ⚙️ How It Works

### RAG Pipeline

1. The anime CSV dataset is loaded and split into chunks using LangChain's `CharacterTextSplitter`
2. Chunks are embedded using HuggingFace's `all-MiniLM-L6-v2` model and stored in ChromaDB
3. On each user query, the retriever fetches the most semantically similar anime entries
4. The retrieved context + user query are passed to the Groq LLM via a structured prompt
5. The LLM returns exactly **3 anime recommendations** with title, summary, and match reason

### Prompt Design

The system prompt strictly:
- Restricts responses to anime-related questions only
- Returns a hard refusal for off-topic queries (no pivoting)
- Always recommends exactly 3 titles in numbered list format
- Never fabricates information outside the provided context

---

## 📦 Requirements

```
langchain==0.2.16
langchain-community==0.2.16
langchain_groq==0.1.9
chromadb==0.5.5
streamlit==1.37.1
pandas==2.2.2
python-dotenv==1.0.1
sentence-transformers==3.0.1
langchain_huggingface==0.0.3
```

---

## 🛠️ Troubleshooting

| Error | Fix |
|-------|-----|
| `ModuleNotFoundError: No module named 'pipeline'` | Run `uv pip install -e .` from project root |
| `CustomException: Error '_type'` | Delete `chroma_db/` and rebuild: `python pipeline/build_pipeline.py` |
| `streamlit: command not found` | Use `uv run streamlit run app/app.py` |
| `GROQ_API_KEY` missing | Ensure `.env` file exists with a valid key |
| Pod `CrashLoopBackOff` | Run `kubectl logs -f deployment/anime-recommender` |

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

> Built with ❤️ using LangChain · ChromaDB · Groq · Streamlit · Docker · Kubernetes