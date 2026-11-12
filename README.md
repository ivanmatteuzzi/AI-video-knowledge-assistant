# AI Video Knowledge Assistant

An AI-powered application that allows users to interact with YouTube video content through natural-language questions and voice input. It retrieves relevant information from video transcripts and presents it in a written, conversational format, reducing the need to rewatch videos to find specific information.

**Live application:** [AI Video Knowledge Assistant](https://ai-video-knowledge-assistant.vercel.app/)

**Backend API:** [Render deployment](https://ai-video-knowledge-assistant.onrender.com)

**LangSmith:** [Project and evaluation workspace](https://eu.smith.langchain.com/o/d3c08405-f466-4174-a589-5e0b4f0822d6/a/ai-video-knowledge-assistant)

---

## 1. Project Overview

The AI Video Knowledge Assistant is a multimodal AI application designed for people interested in wellbeing, health, exercise, and the benefits of physical activity.

The application addresses a common problem: finding specific information within educational videos can require repeatedly watching long sections of content.

Instead, users can select a video and ask questions in natural language. The system searches the corresponding transcript, retrieves relevant passages, and generates an answer grounded in the available video content.

Users can interact with the application through:
- Text-based questions.
- Voice-recorded questions, which are transcribed before processing.
- Conversational follow-up questions about the selected video.

The initial video library contains nine English-language videos.

## 2. Business Case and Objectives

The project explores how large language models, speech recognition, vector search, and AI agents can improve access to information contained in video content.

The main objectives are:

- Enable natural-language question answering over YouTube video transcripts.
- Retrieve relevant information without requiring users to rewatch entire videos.
- Support voice input through speech-to-text processing.
- Implement an AI agent capable of selecting and using dedicated tools.
- Store and retrieve transcript information using vector search.
- Maintain conversational context through persistent checkpoints.
- Evaluate retrieval and answer quality using testing and evaluation tools.

## 3. Key Features

### Video-based Question Answering
Users can ask questions about the content of a selected video. The system retrieves relevant transcript passages and generates context-based answers.

### Voice Input
Users can record questions using their voice. The audio is processed through a transcription endpoint before the resulting text is submitted to the conversational agent.

### Semantic Search
The application uses embeddings and vector similarity search to identify transcript passages relevant to a user's question.

### AI Agent and Tools
The conversational agent can use dedicated tools to:
- Search the transcript.
- Summarize the currently selected video.
- Retrieve relevant video references.

### Conversational Context
The application maintains conversation checkpoints using SQLite and LangGraph's checkpointing functionality.

### Timestamp-based References
Retrieved transcript passages include timestamp information, helping users identify where relevant information appears in the original video.

## 4. Technology Stack

| Component | Technology |
|---|---|
| Backend | Python, FastAPI |
| Frontend | Next.js, React, TypeScript |
| AI framework | LangChain, LangGraph |
| Language model | OpenAI GPT-5.4-mini (default configuration) |
| Embeddings | OpenAI text-embedding-3-small |
| Primary application vector database | Neon PostgreSQL + pgvector |
| Additional vector store component | ChromaDB |
| Conversational checkpointing | SQLite |
| Speech processing | Audio transcription endpoint |
| Evaluation and observability | LangSmith |
| Backend deployment | Render |
| Frontend deployment | Vercel |

## 5. System Architecture

The application is structured into a frontend, backend, AI processing layer, and data storage layer.

### Frontend
The Next.js application provides the user interface for:
- Selecting a video.
- Submitting text questions.
- Recording voice questions.
- Displaying generated answers and references.

### Backend
FastAPI exposes the application endpoints and coordinates the interaction between the frontend, AI agent, retrieval tools, and data services.

### AI Processing
LangChain and LangGraph provide the framework for the conversational agent, tool execution, and conversation checkpointing.

### Data Storage
The deployed application uses Neon PostgreSQL with pgvector for transcript retrieval.

SQLite is used for local agent checkpoint storage.

ChromaDB is also present in the project and is used by the catalog-building script.

## 6. Retrieval and AI Agent Workflow

The main question-answering workflow follows these steps:

1. The user selects a video and submits a question.
2. The backend receives the request.
3. The LangGraph agent processes the question.
4. The agent uses the transcript search tool to retrieve relevant passages.
5. The retrieval layer generates an embedding for the question.
6. Neon PostgreSQL with pgvector performs vector similarity search.
7. The retrieved transcript passages are provided to the language model as evidence.
8. The agent generates a natural-language answer based on the available video content.
9. Relevant references and timestamps can be used to locate the original information.

The agent is instructed to use transcript evidence for factual questions about the selected video and to distinguish video-derived information from general knowledge.

### Agent Tools

| Tool | Purpose |
|---|---|
| `search_transcript` | Searches transcript passages relevant to the user's question. |
| `summarize_current_video` | Produces a summary of the selected video. |
| `retrieve_video_references` | Retrieves relevant transcript references. |

## 7. Database and Embeddings

### Neon PostgreSQL + pgvector

Neon PostgreSQL is the primary vector database used by the deployed application's retrieval workflow.

The database stores transcript chunks and associated metadata, including:
- Video ID.
- Chunk index.
- Transcript content.
- Start and end timestamps.
- Segment information.
- Vector embeddings.

The retrieval process uses OpenAI embeddings generated with `text-embedding-3-small`.

Vector similarity search is performed through pgvector.

### ChromaDB

ChromaDB remains part of the project codebase and dependencies.

The `build_video_catalog.py` script uses ChromaDB to index prepared transcript chunks. It is distinct from the Neon-based retrieval workflow used by the deployed application.

## 8. Video Catalog

The current catalog contains nine English-language videos covering topics related to physical activity, health, recovery, nutrition, sleep, and stress.

| Video | Topic |
|---|---|
| How Playing Sports Benefits Your Body and Your Brain | Benefits of sport |
| The Brain-Changing Benefits of Exercise | Exercise and the brain |
| How the Heart Changes with Exercise | Cardiovascular health |
| Reduce Your Risk of Injury | Injury prevention |
| Recovery Strategies for Athletes | Recovery |
| What is Strength? | Strength |
| Sleep Toolkit: Tools for Optimizing Sleep & Sleep-Wake Timing | Sleep |
| Food is Fuel: Nutrients for Endurance, Optimal Sports Nutrition, and Handling Plateaus | Sports nutrition |
| How to Make Stress Your Friend | Stress management |

The application currently works with this predefined video catalog. It does not provide unrestricted ingestion of arbitrary YouTube videos through the deployed user interface.

The catalog preparation script is available at:

`backend/scripts/build_video_catalog.py`

It retrieves English-language transcripts, cleans transcript segments, creates overlapping chunks, saves the original transcript data locally as JSON files, and indexes the prepared chunks in ChromaDB.

## 9. API Endpoints

The FastAPI backend exposes the following endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/` | Root endpoint |
| GET | `/health` | Backend health check |
| POST | `/videos/ingest` | Retrieves a video from the predefined catalog |
| POST | `/agent-chat` | Processes conversational questions through the AI agent |
| POST | `/transcribe` | Processes audio input for transcription |

## 10. Testing and Evaluation

The project includes automated tests and evaluation components.

### Functional Testing

Functional testing covered:
- Backend API endpoints.
- Video selection and retrieval.
- Multiple questions about the same video.
- Questions targeting different sections of a transcript.
- Voice recording and submission through the deployed application.

### Retrieval and Answer Evaluation

The project includes evaluation scripts and LangSmith integration for inspecting retrieval and answer-generation behaviour.

Evaluation-related files include:
- `evaluation_dataset.py`
- `langsmith_answer_evaluation.py`
- `langsmith_dataset.py`
- `langsmith_evaluation.py`
- `retrieval_evaluation.py`

LangSmith provides an environment for tracing and evaluating application runs.

No overall accuracy percentage is reported here because a complete, verified quantitative evaluation result has not been established.

**LangSmith workspace:**
[Open project and evaluation results](https://eu.smith.langchain.com/o/d3c08405-f466-4174-a589-5e0b4f0822d6/a/ai-video-knowledge-assistant)

## 11. Deployment

The application is deployed using separate frontend and backend services.

| Service | Platform | URL |
|---|---|---|
| Frontend | Vercel | https://ai-video-knowledge-assistant.vercel.app/ |
| Backend | Render | https://ai-video-knowledge-assistant.onrender.com |
| Database | Neon | PostgreSQL with pgvector |
| Evaluation | LangSmith | https://eu.smith.langchain.com/o/d3c08405-f466-4174-a589-5e0b4f0822d6/a/ai-video-knowledge-assistant |

The frontend communicates with the deployed backend API.

## 12. Local Installation

### Prerequisites

- Python
- Node.js and npm
- OpenAI API key
- Neon PostgreSQL database with pgvector
- Optional LangSmith API key for tracing and evaluation

### Backend Setup

From the project root, create a `.env` file using `.env.example` as a template.

Configure the required environment variables:

```env
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=gpt-5.4-mini
NEON_DATABASE_URL=your_neon_database_url
```

LangSmith variables can be configured separately if tracing and monitoring are required.

Install backend dependencies:

```powershell
cd backend
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Start the backend:

```powershell
python -m uvicorn app.main:app --reload
```

The API will be available locally at:

`http://127.0.0.1:8000`

### Frontend Setup

In a separate terminal:

```powershell
cd frontend
npm install
npm run dev
```

The frontend development server will be available at:

`http://localhost:3000`

**Note:** The frontend currently references the deployed Render backend URL in its application code. Running the frontend locally therefore does not automatically configure it to use a locally running backend.

## 13. Environment Variables

The backend uses the following configuration variables:

| Variable | Purpose |
|---|---|
| `OPENAI_API_KEY` | OpenAI API authentication |
| `OPENAI_MODEL` | Language model configuration |
| `NEON_DATABASE_URL` | Neon PostgreSQL connection string |
| `LANGCHAIN_TRACING_V2` | Enables or disables LangSmith tracing |
| `LANGCHAIN_API_KEY` | LangSmith API authentication |
| `LANGCHAIN_PROJECT` | LangSmith project name |
| `APP_ENV` | Application environment |

Never commit real API keys, database credentials, or private environment files to version control.

## 14. Current Limitations and Future Improvements

### Current Limitations

- The application currently supports a predefined catalog of nine English-language videos.
- Video question answering is based on transcript content rather than direct visual analysis of video frames.
- The quality of generated answers depends on transcript availability and retrieval relevance.
- The current implementation does not provide unrestricted YouTube ingestion through the deployed interface.
- Quantitative evaluation results require further systematic validation before reporting an overall performance score.

### Potential Future Improvements

- Expand the video catalog.
- Support additional transcript languages.
- Improve retrieval quality through systematic benchmarking.
- Extend evaluation datasets and automated answer-quality assessment.
- Introduce direct multimodal video analysis.
- Improve deployment configuration and environment management.

## 15. Project Context

This project was developed as part of the Ironhack AI Engineering programme.

It demonstrates the integration of large language models, semantic search, vector databases, conversational agents, speech-to-text processing, evaluation tools, and cloud deployment in a practical application.

**Project:** AI Video Knowledge Assistant
**Programme:** Ironhack AI Engineering
**Application:** https://ai-video-knowledge-assistant.vercel.app/
