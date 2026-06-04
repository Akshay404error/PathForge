# 🛤️ PathForge

> **AI-powered conversational learning roadmap platform**

PathForge turns vague learning goals into structured AI-generated roadmaps with milestones, curated resources, capstone projects, and streak-based progress tracking. Mentors and institutions get a dashboard to track learners. Everything runs on **local Ollama** — your data never leaves your machine.

---

## ✨ Features

- 💬 **Conversational AI Coach** — Natural chat that elicits your learning goal
- 🗺️ **Personalized Roadmaps** — AI generates milestones → lessons → projects
- 📚 **Curated Resources** — YouTube, MDN, official docs auto-attached to lessons
- 🔥 **Streaks & XP** — Daily streak tracking + GitHub-style activity heatmap
- ✅ **Progress Tracking** — Lesson completion, milestone auto-completion, XP rewards
- 🔗 **Shareable Public Profile** — `/u/yourname` like a "learning resume"
- 👥 **Mentor Dashboard** — Mentors and institutions track multiple learners
- 🔒 **100% Local & Private** — Powered by Ollama, no cloud LLM calls
- 📧 **Email OTP Auth** — 6-digit code on signup AND login (Gmail SMTP or console mode)

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | FastAPI · SQLAlchemy · Pydantic v2 |
| AI Engine | Ollama (Llama 3.2) — local |
| Database | SQLite (dev) → PostgreSQL (prod-ready) |
| Auth | Bcrypt + JWT, role-based |
| Frontend | Vanilla JS + HTML + CSS (zero build step) |

---

## 📁 Project Structure

```
pathforge/
├── backend/
│   ├── app/
│   │   ├── core/          # config, security (JWT, bcrypt)
│   │   ├── db/            # SQLAlchemy engine + session
│   │   ├── models/        # User, Roadmap, Milestone, Lesson, Project, Progress
│   │   ├── schemas/       # Pydantic request/response models
│   │   ├── services/      # ai_service (Ollama), roadmap_service
│   │   ├── api/           # auth, chat, roadmaps, progress, profile routes
│   │   └── main.py        # FastAPI entry point
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── css/style.css      # Premium dark theme
│   ├── js/api.js          # API client + auth + UI helpers
│   ├── index.html         # Landing
│   ├── login.html, register.html
│   ├── dashboard.html     # Stats, heatmap, roadmap grid
│   ├── chat.html          # Conversational AI coach
│   ├── roadmap.html       # Milestones, lessons, projects
│   ├── profile.html       # Public shareable profile
│   └── mentor.html        # Mentor/institution dashboard
├── PathForge_BusinessModel.pptx   # 14-slide pitch deck
└── README.md
```

---

## 🚀 Quick Start (VS Code)

### Prerequisites

1. **Python 3.10+**
2. **Node.js** (only needed if you want to rebuild the deck)
3. **Ollama** — https://ollama.com/download

### Step 1 — Install Ollama and pull a model

```bash
# Install Ollama from https://ollama.com (Windows / Mac / Linux)
# Then pull the recommended model:
ollama pull llama3.2

# Verify it is running:
ollama list
# Ollama auto-starts a server on http://localhost:11434
```

### Step 2 — Backend setup

Open the project in VS Code, then in the terminal:

```bash
cd backend

# Create a virtual environment
python -m venv venv

# Activate it
# Windows (PowerShell):
.\venv\Scripts\Activate.ps1
# Windows (cmd):
venv\Scripts\activate
# macOS / Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Copy environment file
# Windows:
copy .env.example .env
# macOS / Linux:
cp .env.example .env

# Run the API
uvicorn app.main:app --reload --port 8000
```

The API is now live at **http://localhost:8000**
Auto-generated docs: **http://localhost:8000/docs**

You should see: `Ollama is reachable at http://localhost:11434 (model: llama3.2)`

### Step 3 — Frontend setup

The frontend is pure HTML/CSS/JS — no build step. Use VS Code's **Live Server** extension:

1. Install the **Live Server** extension by Ritwick Dey in VS Code
2. Right-click `frontend/index.html` → **Open with Live Server**
3. Browser opens at `http://127.0.0.1:5500/frontend/index.html`

Alternatively from the terminal:

```bash
cd frontend
python -m http.server 5500
# Then open: http://localhost:5500
```

---

## 📧 Email OTP Setup

PathForge requires email OTP verification on **both signup and login**. There are two modes, switchable via `.env`:

### Mode 1: Console (default — perfect for hackathon demo)

```bash
# In backend/.env:
EMAIL_MODE=console
```

The OTP is **printed to the terminal** where uvicorn is running, in a nicely-formatted box:

```
╔════════════════════════════════════════════════════════╗
║  📧  EMAIL OTP (console mode — no email sent)          ║
╠════════════════════════════════════════════════════════╣
║  To:      akshay@example.com                           ║
║  Subject: Verify your PathForge account                ║
║  Code:    539771                                       ║
║  Expires: 10 minutes                                   ║
╚════════════════════════════════════════════════════════╝
```

This is **ideal for hackathon demos** — you don't need internet/SMTP, just paste the code from the terminal into the browser.

### Mode 2: Gmail SMTP (real emails)

Use this for live deployments or impressing judges with real emails.

**Step 1 — Enable 2-Step Verification on your Google account**
Go to https://myaccount.google.com/security → Turn on "2-Step Verification".

**Step 2 — Generate an App Password**
Visit https://myaccount.google.com/apppasswords → Create one for "Mail" → Copy the 16-character password (no spaces).

**Step 3 — Configure `backend/.env`:**

```bash
EMAIL_MODE=gmail
GMAIL_USER=youremail@gmail.com
GMAIL_APP_PASSWORD=abcdwxyzabcdwxyz   # 16 chars, no spaces
EMAIL_FROM_NAME=PathForge
```

Restart the backend. Real emails will now be sent with a polished HTML template.

### OTP Security

- 6-digit codes, **cryptographically random** (`secrets.randbelow`)
- **Bcrypt-hashed** before storage — plaintext never touches the DB
- 10-minute expiry (configurable)
- **Max 5 verification attempts** per code → locks the OTP
- **Max 3 OTP requests per email per 15 minutes** → rate limit
- Single-use — verified codes are immediately consumed
- Purpose-bound — a `signup` OTP cannot be used on the `login` endpoint

---

## 🎮 Demo Flow

1. **Register** at `/register.html` — pick role: `learner`, `mentor`, or `institution`
   - You'll be sent a 6-digit OTP. In console mode, **check your uvicorn terminal** for the code.
2. **Verify your email** by entering the 6 digits → account created, JWT issued
3. **Log in** at `/login.html` — enter password, then verify OTP again (sent fresh)
4. **Chat** with the AI Coach at `/chat.html`
   - Try: *"I want to become a backend engineer in 6 months. I know Python basics."*
   - The AI asks clarifying questions, then offers to generate a roadmap
5. **Generate roadmap** — adjust duration/hours, hit Generate (takes ~30-60s with local Ollama)
6. **Open the roadmap** — click lessons to mark complete, watch XP and streaks build
7. **View dashboard** — see streak heatmap, total XP, all roadmaps
8. **Public profile** — share `http://localhost:5500/profile.html?u=yourname`
9. **Mentor view** (only mentor/institution roles) — `/mentor.html` shows all learners

---

## 🔌 API Endpoints (21 total)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | Server + Ollama status |
| POST | `/api/auth/signup/request` | Step 1: send signup OTP |
| POST | `/api/auth/signup/verify` | Step 2: verify OTP, create account |
| POST | `/api/auth/login/request` | Step 1: verify password, send OTP |
| POST | `/api/auth/login/verify` | Step 2: verify OTP, get JWT |
| POST | `/api/auth/otp/resend` | Resend OTP for in-flight flow |
| POST | `/api/chat/` | Conversational AI |
| DELETE | `/api/chat/history` | Clear chat |
| POST | `/api/roadmaps/generate` | Generate via AI |
| GET | `/api/roadmaps/` | List my roadmaps |
| GET | `/api/roadmaps/{id}` | Get roadmap detail |
| DELETE | `/api/roadmaps/{id}` | Delete |
| PATCH | `/api/roadmaps/{id}/visibility` | Toggle public/private |
| POST | `/api/progress/lesson/complete` | Mark lesson done |
| POST | `/api/progress/lesson/uncomplete` | Undo |
| GET | `/api/progress/streak` | Streak + heatmap data |
| GET | `/api/profile/me` | Current user |
| PATCH | `/api/profile/me` | Update profile |
| GET | `/api/profile/u/{username}` | Public profile |
| GET | `/api/profile/u/{username}/roadmaps` | Public roadmaps |
| GET | `/api/profile/mentor/learners` | Mentor view |

Full interactive docs: **http://localhost:8000/docs**

---

## 🗄️ Database Schema

```
User ──┬── Roadmap ──┬── Milestone ──┬── Lesson (with resources JSON)
       │             │               └── Project (with requirements JSON)
       └── ProgressLog (daily activity for heatmap + streaks)

OTPCode (standalone — email + bcrypt-hashed code + purpose + expiry)
```

The DB auto-initializes on first run as `backend/pathforge.db` (SQLite). For PostgreSQL, just change `DATABASE_URL` in `.env`.

---

## 🐛 Troubleshooting

| Problem | Fix |
|---------|-----|
| `Ollama NOT reachable` warning | Run `ollama serve` or check it's running on port 11434 |
| Roadmap generation times out | First call is slow as model loads into memory; subsequent calls are faster |
| Roadmap quality is poor | Try `ollama pull llama3.1:8b` and set `OLLAMA_MODEL=llama3.1:8b` in `.env` |
| CORS errors in browser | The backend allows `localhost:5500` and `127.0.0.1:5500` by default |
| `email-validator` import error | Run `pip install email-validator` |
| Bcrypt errors | Already handled — we use `bcrypt` directly, not passlib |
| Didn't receive OTP email | Check `EMAIL_MODE` in `.env`. If `console`, look at the uvicorn terminal. If `gmail`, check spam folder & verify app password is correct (no spaces). |
| "Too many code requests" | Hit the rate limit (3 OTPs per 15min per email). Wait 15 minutes or use a different email. |
| OTP says "expired" | Codes expire in 10 min. Click "Resend code" on the OTP screen. |

---

## 💼 Business Model

See **`PathForge_BusinessModel.pptx`** — 14-slide deck covering:

1. Title · 2. Problem · 3. Insight · 4. Product · 5. How it works
6. Target users · 7. Market size · 8. Business model · 9. Unit economics
10. Go-to-market · 11. Competitive landscape · 12. Tech & moat
13. Roadmap & ask · 14. Closing

**TL;DR Revenue Streams:**
- 💚 **B2C Freemium** (40%) — ₹299/mo Pro plan
- 💜 **B2B Institutions** (45%) — ₹1,200/seat/year
- 🟠 **B2B2C Marketplace** (15%) — 15-20% mentor commission

LTV/CAC = **6.7x** · Gross margin = **78%** (local LLM = near-zero inference cost)

---

## 👤 Built by

**Akshay D** — AI/ML & ECE Engineer
Team Matsya N · Sri Venkateshwara College of Engineering, Bengaluru
GitHub: [@Akshay404error](https://github.com/Akshay404error)

Built for **Scaler School of Business ProdX Hackathon — Education Domain**

---

## 📄 License

MIT — Free to fork, learn from, and improve.
