# MockFlow-AI

<div align="center">

**AI-powered voice interview practice that interviews you, evaluates your responses, and helps you improve.**

[![Python](https://img.shields.io/badge/Python-3.12-blue.svg)](https://www.python.org/downloads/)
[![Flask](https://img.shields.io/badge/Flask-3-000000.svg)](https://flask.palletsprojects.com/)
[![LiveKit](https://img.shields.io/badge/LiveKit-Agents-00ADD8.svg)](https://docs.livekit.io/agents/)
[![OpenAI](https://img.shields.io/badge/OpenAI-LLM%20%2B%20TTS-412991.svg)](https://platform.openai.com/)
[![Deepgram](https://img.shields.io/badge/Deepgram-STT-13EF93.svg)](https://deepgram.com/)
[![Postgres](https://img.shields.io/badge/Neon-Postgres-00E599.svg)](https://neon.tech/)

</div>

---

## What it is

**MockFlow-AI** is a full-stack AI interview platform designed to simulate realistic interview environments.

It combines **real-time voice interaction, adaptive AI questioning, coding interviews, resume awareness, and structured performance evaluation** into a single interview experience.

The AI interviewer can:

* Conduct voice-based interviews in real time
* Ask adaptive follow-up questions
* Conduct technical and coding interviews
* Use your resume and job description for context
* Evaluate communication and technical performance
* Generate structured interview feedback

MockFlow-AI follows a **BYOK (Bring Your Own Keys)** architecture, allowing users to provide their own API credentials for the required AI and voice services.

---

## Features

### 🎯 Four Interview Tracks

| Track          | Focus                                                         |
| -------------- | ------------------------------------------------------------- |
| **Intro**      | General background, motivation, and culture fit               |
| **Behavioral** | STAR-style behavioral interviews with configurable follow-ups |
| **Technical**  | Topic-based conceptual technical questions                    |
| **Coding**     | Coding interviews with curated programming problems           |

---

### 🎙️ Real-Time AI Interviewer

MockFlow-AI provides a conversational interview experience instead of a traditional chatbot-style Q&A flow.

* Real-time speech-to-text using **Deepgram**
* LLM-powered interviewer using **OpenAI**
* Text-to-speech using **OpenAI**
* Real-time communication through **LiveKit WebRTC**
* FSM-driven interview stages
* Adaptive follow-up questions
* Resume-aware interviews
* Job-description-aware questioning
* Explicit stage transitions and fallback handling

---

### 💻 Coding Interview

The coding track provides a dedicated coding interview experience for technical interview practice.

The coding system includes:

* Curated coding problems
* Hidden test cases
* Reference solutions
* Optional code execution through **Piston**
* Objective pass/fail signals for evaluation

The optional Piston integration allows the AI evaluator to ground its feedback in actual code execution results rather than relying only on generated evaluation.

---

### 📊 Interview Feedback

After completing an interview, MockFlow-AI generates structured feedback covering both communication and technical performance.

#### Speech Analytics

* Filler-word detection
* Words per minute
* Speaking pace
* Per-turn analysis

#### Competency Evaluation

* Communication
* Technical depth
* Relevance
* Confidence
* Problem-solving approach
* Edge-case handling
* Time and space complexity
* Coding performance

Feedback can be:

* Exported as PDF
* Copied as Markdown

---

### 📈 Interview Dashboard

The dashboard provides an overview of interview performance, including:

* Total interviews
* Average score
* Track-wise performance
* Interview history
* Interview personality insights
* Recent interview activity

---

## Tech Stack

| Concern            | Technology                     |
| ------------------ | ------------------------------ |
| Web Application    | **Flask 3**                    |
| Voice Agent        | **LiveKit Agents**             |
| Database           | **Neon PostgreSQL**            |
| Authentication     | **Google OAuth + Flask-Login** |
| Speech-to-Text     | **Deepgram**                   |
| LLM + TTS          | **OpenAI**                     |
| API Key Encryption | **Fernet**                     |
| Code Execution     | **Piston**                     |
| Production Server  | **Gunicorn**                   |

---

## Architecture

```text
                         ┌─────────────────────┐
                         │      Browser        │
                         │                     │
                         │  Interview UI       │
                         │  Voice Interface    │
                         │  Coding Interface   │
                         └──────────┬──────────┘
                                    │
                                    │ POST /api/token
                                    ▼
                         ┌─────────────────────┐
                         │       Flask         │
                         │                     │
                         │ OAuth               │
                         │ Token Generation    │
                         │ Worker Management   │
                         │ Feedback APIs       │
                         └──────┬───────┬──────┘
                                │       │
                  ┌─────────────┘       └─────────────┐
                  ▼                                   ▼
        ┌──────────────────┐                 ┌──────────────────┐
        │   Neon Postgres  │                 │   Agent Worker   │
        │                  │                 │                  │
        │ Encrypted Keys   │                 │ LiveKit Agents   │
        │ Interviews       │                 │ FSM              │
        │ Coding Data      │                 │ Voice Pipeline   │
        │ User Data        │                 │ Evaluation       │
        └──────────────────┘                 └────────┬─────────┘
                                                      │
                           ┌──────────────────────────┼─────────────────────┐
                           ▼                          ▼                     ▼
                    ┌─────────────┐            ┌─────────────┐      ┌─────────────┐
                    │   LiveKit  │            │  Deepgram   │      │   OpenAI    │
                    │   WebRTC   │            │    STT      │      │  LLM + TTS  │
                    └─────────────┘            └─────────────┘      └─────────────┘
```

The interview state machine is track-aware and supports:

```text
intro
behavioral
technical_voice
technical_coding
```

Because the BYOK worker architecture tracks agent subprocesses in application memory, the web server runs using a **single Gunicorn worker**.

---

## Project Structure

```text
MockFlow-AI/
│
├── app.py
├── agent_worker.py
├── worker_manager.py
├── fsm.py
├── prompts.py
├── db.py
├── speech_analytics.py
├── document_processor.py
│
├── tracks/
│   ├── intro/
│   ├── behavioral/
│   ├── technical_voice/
│   └── technical_coding/
│
├── migrations/
│   ├── 001_initial_schema.sql
│   └── 002_free_tier_and_stats.sql
│
├── tests/
│   └── e2e/
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── TESTING_E2E.md
│   └── DEPLOYMENT_GCP.md
│
├── static/
│   ├── animations.css
│   ├── mf-rays.js
│   └── animations.html
│
├── templates/
├── requirements-dev.txt
├── env.template
└── runtime.txt
```

### Core Components

| File                    | Purpose                                                         |
| ----------------------- | --------------------------------------------------------------- |
| `app.py`                | Flask server, authentication, API endpoints and worker spawning |
| `agent_worker.py`       | LiveKit agent, voice pipeline and interview logic               |
| `worker_manager.py`     | Spawns and manages interview agent subprocesses                 |
| `fsm.py`                | Multi-track interview state machine                             |
| `tracks/`               | Track-specific stages and configuration                         |
| `db.py`                 | Neon PostgreSQL connection and persistence                      |
| `prompts.py`            | Interview, feedback and evaluation prompts                      |
| `speech_analytics.py`   | Filler-word and speech analysis                                 |
| `document_processor.py` | Resume parsing and document processing                          |

---

## Local Setup

### Prerequisites

Make sure you have:

* Python 3.12
* Neon PostgreSQL database
* Google OAuth application
* LiveKit account
* OpenAI API key
* Deepgram API key

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd MockFlow-AI
```

### 2. Create the environment file

```bash
cp env.template .env
```

Configure the required environment variables.

### 3. Install dependencies

```bash
pip install -r requirements-dev.txt
```

### 4. Run database migrations

```bash
psql "$DATABASE_URL" -f migrations/001_initial_schema.sql
psql "$DATABASE_URL" -f migrations/002_free_tier_and_stats.sql
```

### 5. Start the application

```bash
python app.py
```

The application will be available at:

```text
http://localhost:5000
```

---

## Environment Variables

### Required

| Variable               | Purpose                                            |
| ---------------------- | -------------------------------------------------- |
| `DATABASE_URL`         | Neon PostgreSQL pooled connection string           |
| `GOOGLE_CLIENT_ID`     | Google OAuth client ID                             |
| `GOOGLE_CLIENT_SECRET` | Google OAuth client secret                         |
| `SECRET_KEY`           | Flask session signing key                          |
| `ENCRYPTION_KEY`       | Fernet key used to encrypt stored BYOK credentials |

### Optional

| Variable                 | Purpose                               |
| ------------------------ | ------------------------------------- |
| `FLASK_ENV`              | Production configuration              |
| `CORS_ORIGINS`           | Comma-separated allowed origins       |
| `MAX_CONCURRENT_WORKERS` | Maximum concurrent agent subprocesses |
| `FREE_TIER_*`            | Optional free interview tier          |
| `SYSTEM_*`               | Owner keys for the free tier          |
| `PISTON_*`               | Optional code execution               |

---

## Testing

Run the complete test suite:

```bash
python -m pytest
```

Run linting:

```bash
python -m ruff check .
```

End-to-end testing is available under:

```text
tests/e2e/
```

For the complete manual testing workflow:

```text
docs/TESTING_E2E.md
```

---

## Deployment

The application can be deployed using Gunicorn.

Production command:

```bash
gunicorn app:app --workers 1 --timeout 120
```

The single-worker configuration is important because the current worker-management architecture maintains agent subprocess state in application memory.

### Health Check

```text
GET /health
```

The health endpoint checks the application and database connectivity and reports worker load.

---

## Roadmap

MockFlow-AI is being developed toward a broader **AI-powered interview preparation platform**.

Planned features include:

* Interview and application tracker
* Preparation for real upcoming interviews
* Company-specific interview preparation
* Role-specific interview tracks
* Research-powered interview generation
* Richer interview personality analytics
* Free trial interviews
* Personalized signed-in dashboard
* Upcoming interview management
* Expanded coding support

---

## Why MockFlow-AI?

Most interview-preparation tools focus on giving candidates a list of questions.

MockFlow-AI focuses on recreating the **actual interview experience**.

```text
             ┌─────────────┐
             │    Speak    │
             └──────┬──────┘
                    ↓
             ┌─────────────┐
             │   Listen    │
             └──────┬──────┘
                    ↓
             ┌─────────────┐
             │    Think    │
             └──────┬──────┘
                    ↓
             ┌─────────────┐
             │   Answer    │
             └──────┬──────┘
                    ↓
             ┌─────────────┐
             │    Code     │
             └──────┬──────┘
                    ↓
             ┌─────────────┐
             │  Evaluate   │
             └──────┬──────┘
                    ↓
             ┌─────────────┐
             │   Improve   │
             └──────┬──────┘
                    ↓
                  Repeat
```

The goal is to help candidates practice not only **what to say**, but also **how they perform under realistic interview conditions**.

---

## Built By

<div align="center">

### Satyam Kumar

**AI Systems • Backend • Flutter**

B.Tech CSE @ KIIT · 2023–2027

[![Portfolio](https://img.shields.io/badge/Portfolio-krsatyam.in-000?style=for-the-badge\&logo=vercel\&logoColor=white)](https://krsatyam.in)

</div>

---

## Acknowledgments

* **LiveKit** — real-time voice infrastructure
* **OpenAI** — LLM and TTS
* **Deepgram** — speech-to-text
* **Neon** — serverless PostgreSQL
* **Piston** — sandboxed code execution

---

<div align="center">

**Built with Python, Flask, LiveKit, OpenAI, Deepgram and PostgreSQL.**

[⬆ Back to Top](#mockflow-ai)

</div>
