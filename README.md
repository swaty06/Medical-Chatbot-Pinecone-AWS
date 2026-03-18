# 🏥 RAG Medical Chatbot

A production-ready **Retrieval-Augmented Generation (RAG)** medical chatbot powered by **Pinecone** vector database, containerized with **Docker**, and deployed on **AWS** with a complete **CI/CD pipeline**.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Environment Variables](#environment-variables)
  - [Run Locally](#run-locally)
  - [Run with Docker](#run-with-docker)
- [Pinecone Setup](#pinecone-setup)
- [AWS Deployment](#aws-deployment)
  - [CI/CD Pipeline](#cicd-pipeline)
- [Cost Note](#cost-note)
- [Screenshots](#screenshots)
- [License](#license)

---

## 🧠 Overview

This project is an intelligent medical chatbot that uses **Retrieval-Augmented Generation (RAG)** to answer medical queries accurately. A medical book in **PDF format** is parsed, chunked, and embedded into a **Pinecone** vector store. When a user asks a question, relevant chunks are retrieved and passed as context to a **Groq-powered LLM** (Llama / Mixtral) — delivering fast, grounded, domain-specific responses with minimal hallucination.

---

## 🏗️ Architecture

```
User Query
    │
    ▼
┌─────────────────────┐
│   Chatbot Frontend  │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   FastAPI / Flask   │  ← Application Backend
└────────┬────────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐  ┌─────────────────────────┐
│  LLM  │  │  Pinecone Vector Store  │
│(OpenAI│  │  (Medical Knowledge Base│
│/etc.) │  │   as Embeddings)        │
└───────┘  └─────────────────────────┘
         │
         ▼
    RAG Response
    (Retrieved context + LLM generation)
```

**AWS Deployment Architecture:**

```
GitHub Push
    │
    ▼
AWS CodePipeline
    │
    ├──► AWS CodeBuild  (Build Docker Image)
    │
    ├──► Amazon ECR     (Push Docker Image)
    │
    └──► AWS EC2 / ECS  (Deploy Container)
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Language** | Python |
| **Framework** | FastAPI / Flask / Streamlit |
| **LLM** | Groq API (Llama / Mixtral via Groq) |
| **Embeddings** | HuggingFace Sentence Transformers |
| **Vector DB** | Pinecone |
| **Knowledge Source** | Medical Book (PDF) |
| **Containerization** | Docker |
| **Cloud** | AWS (EC2 / ECS / ECR) |
| **CI/CD** | AWS CodePipeline + CodeBuild |
| **Source Control** | GitHub |

---

## ✨ Features

- 🔍 **RAG Pipeline** — Retrieves relevant chunks from a medical PDF before generating answers
- 📄 **PDF Ingestion** — Medical book parsed, chunked, and embedded automatically
- ⚡ **Groq-Powered LLM** — Ultra-fast inference via Groq API (Llama / Mixtral)
- 🧬 **Pinecone Vector Store** — Efficient semantic search over embedded medical content
- 🐳 **Dockerized** — Fully containerized for consistent environments
- ☁️ **AWS Deployed** — Scalable deployment on AWS infrastructure
- 🔄 **CI/CD Pipeline** — Automated build, test, and deploy on every push to main
- 💬 **Conversational Interface** — Clean chat UI for querying medical information
- 🛡️ **Disclaimer Aware** — Bot includes medical disclaimer on responses

---

## 📁 Project Structure

```
rag-medical-chatbot/
│
├── src/
│   ├── app.py                  # Main application entry point
│   ├── chatbot.py              # Chatbot logic and RAG chain
│   ├── embeddings.py           # Embedding generation
│   ├── pinecone_utils.py       # Pinecone index operations
│   └── prompts.py              # LLM prompt templates
│
├── data/
│   └── medical_book.pdf        # Source medical book PDF for ingestion
│
├── scripts/
│   └── ingest.py               # Data ingestion & embedding script
│
├── Dockerfile                  # Docker image definition
├── docker-compose.yml          # Local multi-service setup
├── buildspec.yml               # AWS CodeBuild build specification
├── requirements.txt            # Python dependencies
├── .env.example                # Example environment variables
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.9+
- Docker & Docker Compose
- Pinecone account & API key
- Groq API key — get one free at [console.groq.com](https://console.groq.com)
- AWS account (for cloud deployment)

### Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```env
# Groq
GROQ_API_KEY=your_groq_api_key

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_ENVIRONMENT=your_pinecone_environment
PINECONE_INDEX_NAME=medical-chatbot

# App
APP_PORT=8080
```

### Run Locally

```bash
# Clone the repository
git clone https://github.com/your-username/rag-medical-chatbot.git
cd rag-medical-chatbot

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your keys

# Place your medical PDF in data/ folder, then run:
python scripts/ingest.py

# Start the application
python src/app.py
```

Visit `http://localhost:8080` in your browser.

### Run with Docker

```bash
# Build the Docker image
docker build -t rag-medical-chatbot .

# Run the container
docker run -p 8080:8080 --env-file .env rag-medical-chatbot
```

Or using Docker Compose:

```bash
docker-compose up --build
```

---

## 📦 Pinecone Setup

1. Sign up at [pinecone.io](https://www.pinecone.io/)
2. Create a new index:
   - **Index Name:** `medical-chatbot`
   - **Dimensions:** `384` (HuggingFace `all-MiniLM-L6-v2`) or match your embedding model
   - **Metric:** `cosine`
3. Copy your **API Key** and **Environment** into `.env`
4. Place your medical book PDF inside the `data/` folder and run the ingestion script to parse, chunk, embed, and upload to Pinecone:

```bash
python scripts/ingest.py
```

---

## ☁️ AWS Deployment

### Services Used

| AWS Service | Purpose |
|---|---|
| **EC2 / ECS** | Host and run the Docker container |
| **ECR** | Store Docker images |
| **CodePipeline** | Orchestrate the CI/CD workflow |
| **CodeBuild** | Build Docker images automatically |
| **IAM** | Manage access roles and permissions |

### CI/CD Pipeline

The pipeline triggers automatically on every push to the `main` branch:

```
1. GitHub push to main
       ↓
2. AWS CodePipeline detects change
       ↓
3. AWS CodeBuild runs buildspec.yml
   - Installs dependencies
   - Builds Docker image
   - Pushes image to Amazon ECR
       ↓
4. Deploy updated container to EC2 / ECS
       ↓
5. Application is live ✅
```

**`buildspec.yml` overview:**

```yaml
version: 0.2
phases:
  pre_build:
    commands:
      - aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_URI
  build:
    commands:
      - docker build -t $IMAGE_NAME .
      - docker tag $IMAGE_NAME:latest $ECR_URI/$IMAGE_NAME:latest
  post_build:
    commands:
      - docker push $ECR_URI/$IMAGE_NAME:latest
```

---

## 💸 Cost Note

> ⚠️ **AWS resources were intentionally shut down after deployment to avoid ongoing charges.**
>
> This project was fully deployed and verified on AWS. To keep costs at zero, EC2 instances and associated services have been stopped/terminated. To redeploy, simply push to `main` — the CI/CD pipeline will rebuild and redeploy automatically once instances are restarted.

---

## ⚠️ Medical Disclaimer

> This chatbot is intended for **informational purposes only** and does **not** constitute medical advice. Always consult a qualified healthcare professional for medical decisions.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 🙋‍♂️ Author

Built with ❤️ — feel free to connect or raise issues!
