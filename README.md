<div align="center">

# SaathiCare 🧠
### AI-Powered Alzheimer's Companion & Care Management Platform


> Built at **AfterMath Hackathon** — Top 12 from 36 finalist teams out of 250+ · Team Lead

</div>

---

## What Is SaathiCare?

SaathiCare is a **unified care ecosystem** for Alzheimer's patients, their guardians, and caretakers. It adapts interaction complexity to the patient's cognitive stage (early / moderate / severe) — so the same platform serves a patient who can still hold a full conversation *and* one who needs simple, visual prompts.

The core insight: most Alzheimer's care tools are built for *caretakers*, not patients. SaathiCare puts the patient at the centre with a voice-first 3D AI companion, while giving guardians real-time dashboards and caretakers structured journaling — all from one codebase.

---

## Screenshots

## Screenshots

| AI Companion Chat | Memory Room Therapy |
|-------------------|---------------------|
| ![AI Companion](https://github.com/user-attachments/assets/2e76a4ae-a0cd-477f-add5-300d685b390d) | ![Memory Room](https://github.com/user-attachments/assets/dd85683f-cff9-4e40-88b4-f2b790ba9492) |

| Live GPS Safety Map | Guardian Analytics Dashboard |
|---------------------|------------------------------|
| ![GPS Map](https://github.com/user-attachments/assets/39794fd8-67cb-49f2-b235-a5fa0826b8e0) | ![Dashboard](https://github.com/user-attachments/assets/c162a1ed-fc71-4922-bf7b-a6e1d3ec17e3) |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        Client                           │
│         Next.js App Router  ·  Zustand State            │
└──────────────────┬──────────────────────────────────────┘
                   │  API Routes (Next.js)
        ┌──────────▼──────────────┐
        │      Auth Layer         │
        │   JWT (jose + bcrypt)   │
        │   Role: Patient /       │
        │   Guardian / Caretaker  │
        └──────────┬──────────────┘
                   │
      ┌────────────┼─────────────┐
      ▼            ▼             ▼
 ┌─────────┐ ┌─────────┐ ┌──────────────┐
 │ SQLite  │ │ Gemini  │ │  Web Speech  │
 │  DB     │ │  API    │ │     API      │
 │(better- │ │1.5 Flash│ │ (voice I/O)  │
 │sqlite3) │ │+ warm   │ │              │
 └─────────┘ │fallbacks│ └──────────────┘
             └─────────┘
```


### Why JWT over sessions?

The tri-role architecture (Patient / Guardian / Caretaker) needed **stateless role verification** on every API route without a session store. JWTs carry the role claim directly, so each route handler can gate access in a single decode — no DB lookup per request for auth.

### AI Companion — How the Context Works

Each Gemini call includes:
1. **Patient profile** — name, cognitive stage, known family members, daily routine
2. **Recent conversation history** — last N exchanges (truncated to fit token budget)
3. **Current emotional state** — pulled from the mood log
4. **System persona** — instructs the model to respond as Saathi, with warmth, simplicity, and memory continuity

Prompt optimisation reduced average response latency from ~10–12s to **4–5s** by trimming context to only what's necessary for each turn, rather than sending the full history on every message.

---

## Tri-Role System

| Role | Access | Key Features |
|---|---|---|
| **Patient** | Personal dashboard | AI companion, memory room, games, schedule, QR ID |
| **Guardian** | Full oversight | Analytics, alerts, medication logs, GPS map, health records |
| **Caretaker** | Day-to-day ops | Journal, task completion, medication admin, schedule |

Each role gets a completely different UI layout and data visibility — a Caretaker cannot see the Guardian's analytics dashboard, and a Patient cannot see their own alert history (to avoid anxiety).

---

## Feature Breakdown

### 🤖 AI Companion (Core Feature)
- Voice-first interaction using Web Speech API (STT + TTS)
- 3D animated avatar with speaking/idle state
- Memory-grounded responses — Saathi remembers family names, routines, and past conversations
- Warm fallbacks when Gemini API is unavailable (pre-written empathetic responses)
- Multilingual support

### 🧠 Cognitive Games
- **Card Match** — visual memory pairing
- **Word Recall** — prompted word retrieval
- **Pattern Tap** — sequential pattern recognition
- Scores tracked over time, fed into cognitive trend analytics

### 🖼 Memory Room
- Interactive 3D-style room with clickable memory objects
- Each object triggers a guided memory exercise with audio narration
- Completion awards points, tracked in the engagement system

### 📱 QR Good Samaritan System
- Every patient gets a unique QR code
- Scanning opens a **public safety page** (no login required) with:
  - Patient name and photo
  - Emergency contacts
  - Key medical information
  - One-tap call to guardian
- QR rotates periodically to prevent misuse

### 🔔 Alert System
- Medication missed → push alert to guardian
- Mood drop detected → caretaker notification
- Geofence breach → immediate guardian alert with last known GPS location

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Framework | Next.js 16 (App Router) | Server components + API routes in one repo |
| Language | TypeScript | Type safety across role-specific data shapes |
| Database | SQLite / `better-sqlite3` | Zero-dependency deployment on Render free tier |
| Auth | JWT (`jose` + `bcryptjs`) | Stateless role verification without a session store |
| AI | Google Gemini 1.5 Flash | Fast, cost-efficient, multimodal capable |
| State | Zustand | Lightweight global state for role context + chat history |
| Voice | Web Speech API | Native browser STT/TTS — no third-party voice service |
| Maps | Leaflet | Open-source, no API key required |
| UI Icons | Lucide React | Consistent, accessible icon set |

---

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/netalgupta/Snack-Alzheimer
cd Snack-Alzheimer

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env.local
# Add your GEMINI_API_KEY (optional — fallbacks work without it)

# 4. Start the dev server (seeds the DB automatically)
npm run dev
# → http://localhost:3000
```

### Demo Accounts

| Role | Email | Password |
|---|---|---|
| Patient | `ravi@saathi.care` | `demo123` |
| Guardian | `priya@saathi.care` | `demo123` |
| Caretaker | `anita@saathi.care` | `demo123` |

---

## Project Structure

```
src/
├── app/
│   ├── api/          # All API routes (auth, ai, games, alerts, qr)
│   ├── dashboard/    # Role-gated dashboards
│   ├── companion/    # AI chat + 3D avatar
│   ├── memory-room/  # Interactive memory exercises
│   └── emergency/    # Public QR safety page (no auth)
├── components/       # Shared UI components
├── lib/
│   ├── db.ts         # SQLite connection + query helpers
│   ├── auth.ts       # JWT encode/decode + role middleware
│   └── gemini.ts     # Gemini client + prompt builder
└── store/            # Zustand stores (auth, chat, patient context)
```

---

## Deployment

See **[DEPLOYMENT.md](./DEPLOYMENT.md)** for full instructions, environment variables reference, database schema, and troubleshooting.

---

## Team

Built in 36 hours at **AfterMath Hackathon** · Top 12 / 250+ teams

**Netal Gupta** — Team Lead, Full-Stack, AI Integration, UI/UX  
KJ Somaiya School of Engineering · B.Tech CSBS

---

<div align="center">

Made with ❤️ for the people who forget, and the people who never do.

</div>
