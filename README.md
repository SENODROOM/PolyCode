# PolyCode

PolyCode is a learn-to-code platform: 100+ interactive courses across many languages, an in-browser code playground, daily challenges, certificates, an AI coding mentor (**PolyMentor**) and an AI security scanner (**PolyGuard**).

This repository is the **umbrella repo**. It holds no app code itself — each part lives in its own repository and is pulled in as a git submodule.

---

## Repositories

| Folder | Repo | Stack | What it does |
| --- | --- | --- | --- |
| [`frontend/`](frontend) | [SENODROOM/PolyCode-Frontend](https://github.com/SENODROOM/PolyCode-Frontend) | React 19 (CRA), Monaco | Learner app — courses, playground, daily challenge, profiles, certificates, PolyMentor chat, PolyGuard scanner |
| [`backend/`](backend) | [SENODROOM/PolyCode-Backend](https://github.com/SENODROOM/PolyCode-Backend) | Node.js, Express, MongoDB | REST API — auth (JWT + Google), documents, playground execution, challenges, chat/assistant, certificates |
| [`PolyMentor/`](PolyMentor) | [QuantumLogicsLabs/PolyMentor](https://github.com/QuantumLogicsLabs/PolyMentor) | Python, FastAPI, Groq | AI coding tutor API + MLOps pipeline (MongoDB prompt export, LoRA fine-tuning, eval) |
| [`PolyGuard/`](PolyGuard) | [QuantumLogicsLabs/PolyGuard](https://github.com/QuantumLogicsLabs/PolyGuard) | Python, FastAPI, PyTorch | AI code security analyzer (ML + static analysis), hosted on Hugging Face Spaces |

Each submodule has its own README with full details.

## Architecture

```text
Browser
  └─ frontend (React, :3000)
       ├─ REST ──────────────▶ backend (Express, :5000/api) ──▶ MongoDB
       │                          └─ /api/chat/assistant
       │                               └─ ASSISTANT_PROVIDER:
       │                                    polymentor → PolyMentor FastAPI ──▶ Groq
       │                                    groq       → Groq API directly
       │                                    custom     → your own OpenAI-compatible model
       └─ PolyGuard scans: rules in the browser + PolyGuard API (Hugging Face)
```

### Live deployments

| Service | URL |
| --- | --- |
| Frontend | https://poly-code-frontend-tau.vercel.app |
| Backend API | https://poly-code-backend.vercel.app/api |
| PolyMentor API | https://poly-mentor-one.vercel.app |
| PolyGuard API | https://muhammadsaadamin-polyguard-api.hf.space |

---

## Getting started

### Prerequisites

- Git
- Node.js 18+ and npm
- Python 3.12 (only if you run PolyMentor or PolyGuard locally)
- A MongoDB instance (local `mongod` or MongoDB Atlas)
- A [Groq API key](https://console.groq.com/keys) for the AI mentor

### 1. Clone with submodules

```bash
git clone --recurse-submodules https://github.com/SENODROOM/PolyCode.git
cd PolyCode
```

Already cloned without them? Run `git submodule update --init --recursive`.

**Contributors:** run the setup script to point each submodule at your own fork (`origin`) with the org repo as `upstream`. It can auto-fork for you, or set up plain clones.

```bash
./setup.sh        # macOS / Linux / Git Bash
.\setup.ps1       # Windows PowerShell
```

The setup script only configures git remotes — install dependencies in the next step.

### 2. Install dependencies

```bash
npm install            # root tooling (concurrently)
npm run install:all    # backend + frontend
```

### 3. Configure environment

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
```

Minimum settings:

| File | Variable | Value |
| --- | --- | --- |
| `backend/.env` | `MONGODB_URI` | Your MongoDB connection string |
| `backend/.env` | `JWT_SECRET` | Any long random string |
| `backend/.env` | `ASSISTANT_PROVIDER` | `polymentor` (default), `groq` or `custom` |
| `backend/.env` | `POLYMENTOR_API_URL` | PolyMentor URL — deployed above, or `http://127.0.0.1:8000` locally |
| `backend/.env` | `GROQ_API_KEY`, `GROQ_MODEL` | Needed when `ASSISTANT_PROVIDER=groq` (e.g. `openai/gpt-oss-120b`) |
| `frontend/.env` | `REACT_APP_API_URL` | `http://localhost:5000/api` |
| `frontend/.env` | `REACT_APP_GOOGLE_CLIENT_ID` | Same Google OAuth client ID as the backend `GOOGLE_CLIENT_ID` |

See each `.env.example` for optional features (Google Drive prompt export, Pinecone RAG, PolyGuard remote scanning).

> **Groq models:** Groq has retired `llama-3.3-70b-versatile` and `llama-3.1-8b-instant`. Use `openai/gpt-oss-120b` (main) and `openai/gpt-oss-20b` (fallback). An old model name in any `.env` — or in a deployment's environment variables — causes a `404 model_not_found` error in chat.

### 4. Run

```bash
npm run dev
```

This starts both apps:

- Frontend — http://localhost:3000
- Backend — http://localhost:5000 (API docs at http://localhost:5000/api-docs)

Or run them separately with `npm run dev:backend` and `npm run dev:frontend`.

### 5. (Optional) Run PolyMentor locally

```bash
cd PolyMentor
python -m venv .venv
source .venv/bin/activate          # Windows: .\.venv\Scripts\Activate.ps1
pip install -e .
cp .env.example .env               # set GROQ_API_KEY
uvicorn src.api.app:app --reload --port 8000
```

Then set `POLYMENTOR_API_URL=http://127.0.0.1:8000` in `backend/.env`. See [PolyMentor/README.md](PolyMentor/README.md) for the training pipeline.

---

## Root scripts

| Command | What it does |
| --- | --- |
| `npm run install:all` | `npm install` in `backend/` and `frontend/` |
| `npm run dev` | Backend (nodemon) + frontend dev server together |
| `npm run dev:backend` | Backend only |
| `npm run dev:frontend` | Frontend only |
| `npm run build` | Production build of the frontend (`frontend/build/`) |
| `npm start` | Start the backend in production mode |

## Working with submodules

```bash
# Pull the latest commit of every submodule
git submodule update --remote --merge

# Commit the new submodule pointers in the umbrella repo
git add frontend backend PolyMentor PolyGuard
git commit -m "chore: update submodules"
```

Make code changes **inside** the submodule folder, commit and push them there first, then commit the updated pointer here.

The `.github/workflows/update-submodules.yml` workflow also bumps submodule pointers automatically on a schedule.

## Deployment

- **Frontend** and **backend** deploy separately on Vercel (each has its own `vercel.json`).
- **PolyMentor** deploys to Vercel (`src.api.app:app`).
- **PolyGuard** deploys to Hugging Face Spaces.

Set each service's environment variables in its hosting dashboard — local `.env` files are not uploaded. After changing a variable, redeploy for it to take effect.

## License

See [LICENSE](LICENSE).
