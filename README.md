# 🛡️ Automated Threat Intelligence & CTI Reporting Platform
### A Professional Case Study

> ⚠️ **Note on Source Code**
> This platform was developed during my internship at **Cybercommand Private Limited**
> (Feb 2026 – Apr 2026) under a confidentiality agreement. The source code is
> proprietary and cannot be shared publicly. This repository serves as a
> **detailed technical case study** — documenting my architecture decisions,
> engineering challenges, and personal learnings from building this system
> end-to-end from scratch.
>
> *I designed and built every component of this system independently,
> on my own hardware, using open-source tools and public APIs.*

---

<div align="center">

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Google Gemini](https://img.shields.io/badge/Gemini_AI-4285F4?style=for-the-badge&logo=google&logoColor=white)

**Status:** Deployed in Production &nbsp;|&nbsp;
**Role:** Sole Developer &nbsp;|&nbsp;
**Duration:** Feb 2026 – Apr 2026

</div>

---

## 📋 Table of Contents

- [What This System Does](#-what-this-system-does)
- [The Problem I Was Solving](#-the-problem-i-was-solving)
- [System Architecture](#-system-architecture)
- [The Full Pipeline — How It Actually Works](#-the-full-pipeline--how-it-actually-works)
- [Tech Stack & Why I Chose Each Tool](#-tech-stack--why-i-chose-each-tool)
- [Key Engineering Decisions](#-key-engineering-decisions)
- [The Hardest Problems I Solved](#-the-hardest-problems-i-solved)
- [Security Design](#-security-design)
- [What I Learned](#-what-i-learned)
- [What I Would Do Differently](#-what-i-would-do-differently)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🔍 What This System Does

This platform is a **fully automated, end-to-end Cyber Threat Intelligence (CTI)
pipeline** that:

1. Automatically ingests threat data from **multiple live sources** — cybersecurity
   blogs, threat feeds, and open-source intelligence APIs
2. Uses **Google Gemini LLM** to extract structured Indicators of Compromise (IOCs)
   from unstructured article text
3. Enriches every IOC with **VirusTotal reputation data** automatically
4. Stores, deduplicates, and manages IOCs in **MongoDB**
5. Routes IOCs through an **analyst approval workflow** before they reach reporting
6. Generates **professional PDF threat intelligence reports** automatically
7. Presents everything through a **real-time React dashboard** with JWT authentication

The goal: take a process that previously required hours of manual analyst work and
reduce it to a **single button click** — with a human review checkpoint built in.

---

## 🎯 The Problem I Was Solving

Before this platform existed, threat intelligence gathering at the organization
was entirely manual:

```
Old Process (Manual):
Analyst opens browser → reads 10-15 security blogs → manually copies IOCs
→ checks each one on VirusTotal one by one → pastes into a Word document
→ formats a report → emails it → repeat tomorrow

Time cost: 2-4 hours per day per analyst
Error rate: High (human copy-paste errors, missed IOCs)
Consistency: Zero (different analysts, different formats)
```

```
New Process (Automated):
Analyst clicks "Run Pipeline" → reviews flagged IOCs → approves → clicks
"Generate Report" → report is emailed automatically

Time cost: 10-15 minutes per day
Error rate: Near zero
Consistency: 100% standardized
```

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     ANALYST (Web Browser)                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTPS
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   REACT FRONTEND (Vite)                          │
│   LoginPage  ──►  JWT Token  ──►  DashboardPage                 │
│                                      │                           │
│   • Pipeline trigger button          │ Axios HTTP               │
│   • IOC review & approval table      │                           │
│   • Report generation & download     │                           │
└──────────────────────────────────────┼──────────────────────────┘
                                       │ REST API calls
                                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                   FASTAPI BACKEND                                │
│                                                                  │
│   /auth/login  ──► JWT generation & validation                  │
│   /api/v1/ingest  ──► triggers background pipeline task         │
│   /api/v1/iocs  ──► returns IOCs from MongoDB                   │
│   /api/v1/approve  ──► updates IOC status                       │
│   /api/v1/report  ──► triggers PDF generation                   │
│                                                                  │
└────────────────────┬────────────────────────────────────────────┘
                     │ Background Task (non-blocking)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                   INGESTION PIPELINE                             │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │  Scraper    │  │  OTX Client │  ← Parallel collection        │
│  │ (BS4+HTTP)  │  │  (REST API) │                               │
│  └──────┬──────┘  └──────┬──────┘                               │
│         └────────┬────────┘                                      │
│                  ▼                                               │
│  ┌───────────────────────────┐                                   │
│  │  Gemini AI Client         │  ← IOC extraction from text      │
│  │  + Regex Fallback Engine  │                                   │
│  └──────────────┬────────────┘                                   │
│                 ▼                                                │
│  ┌───────────────────────────┐                                   │
│  │  VirusTotal Enrichment    │  ← Reputation scoring            │
│  └──────────────┬────────────┘                                   │
│                 ▼                                                │
│  ┌───────────────────────────┐                                   │
│  │  MongoDB + Dedup Engine   │  ← Store unique IOCs only        │
│  └──────────────┬────────────┘                                   │
│                 ▼                                                │
│  ┌───────────────────────────┐                                   │
│  │  PDF Report Generator     │  ← ReportLab output              │
│  └───────────────────────────┘                                   │
└─────────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│              EXTERNAL THREAT INTEL SOURCES                       │
│                                                                  │
│   🌐 Threat Intel Blogs    │  Scraped via BeautifulSoup4        │
│   🔴 AlienVault OTX        │  REST API — global indicators      │
│   🟠 VirusTotal API        │  IOC reputation & malware data     │
│   🤖 Google Gemini API     │  LLM text analysis                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 The Full Pipeline — How It Actually Works

### Step 1 — Trigger
An analyst logs into the React dashboard and clicks **"Run Pipeline."**
The frontend sends a `POST /api/v1/ingest` request with a JWT bearer token.

### Step 2 — Background Task Starts
FastAPI receives the request and immediately returns `202 Accepted` — the pipeline
runs as a **background task** so the API never blocks. The analyst can continue
using the dashboard while ingestion runs.

### Step 3 — Parallel Collection
Two collectors run:
- **Web Scraper** fetches the latest articles from targeted cybersecurity blogs
  using `requests` + `BeautifulSoup4`. It extracts article titles, body text,
  and publication dates.
- **OTX Client** calls the AlienVault OTX REST API to pull recently published
  threat indicators — IPs, domains, file hashes associated with active campaigns.

### Step 4 — AI-Powered IOC Extraction
The raw article text is chunked and sent to **Google Gemini** with a carefully
engineered prompt instructing it to extract:

```
IOC Types extracted:
→ IP Addresses (IPv4 and IPv6)
→ Domain Names
→ File Hashes (MD5, SHA1, SHA256)
→ CVE identifiers
→ Malware family names
→ Threat actor names
```

If Gemini returns an unexpected format or fails, a **regex fallback engine**
catches common IOC patterns automatically. This dual-layer approach means the
pipeline never silently fails.

### Step 5 — VirusTotal Enrichment
Every extracted IOC is sent to the VirusTotal API to retrieve:
- Community detection score (X of Y engines flagging as malicious)
- Associated malware families
- Last analysis date
- Threat category

This enrichment data is stored alongside the IOC so analysts have context
during review.

### Step 6 — Deduplication & Storage
Before saving, the system checks MongoDB using a **unique index on the IOC value
field.** If the IOC already exists in the database, it is silently skipped.
New IOCs are saved with `status: "pending"` — ready for analyst review.

### Step 7 — Analyst Review
The dashboard displays all pending IOCs in a sortable table. Analysts can:
- Read the enrichment data
- Add investigation notes
- Mark each IOC as **Approved** or **Rejected**

Only approved IOCs flow into the final report.

### Step 8 — PDF Report Generation
When the analyst triggers report generation, **ReportLab** builds a structured PDF
containing:
- Executive summary of the pipeline run
- Table of all approved IOCs with enrichment data
- Threat landscape summary
- Timestamp and analyst attribution

---

## 🛠️ Tech Stack & Why I Chose Each Tool

| Component | Tool | Why I Chose It |
|:----------|:-----|:---------------|
| **Backend Framework** | FastAPI | Native async support was critical for running the pipeline as a non-blocking background task. Flask would have blocked the API thread during ingestion. |
| **Frontend** | React + Vite | Vite's HMR made development fast. React's component model was ideal for the real-time dashboard state (pipeline status, IOC table, approval controls). |
| **Database** | MongoDB | IOC data is semi-structured and the schema evolves as new IOC types are added. MongoDB's flexible document model means no migration overhead when adding new fields. |
| **AI Layer** | Google Gemini | Tested against GPT-3.5 and Gemini — Gemini's structured output was more consistent for IOC extraction from cybersecurity text. |
| **Web Scraping** | BeautifulSoup4 | Lightweight, well-documented, and sufficient for structured blog HTML. |
| **PDF Generation** | ReportLab | The only Python PDF library that gave me full programmatic control over layout — essential for professional threat intelligence report formatting. |
| **Authentication** | JWT + Passlib | Stateless authentication fits well with FastAPI's architecture. JWTs mean no session storage needed on the backend. |
| **Styling** | Tailwind CSS | Utility-first approach let me build the dashboard UI rapidly without switching between CSS files. |

---

## ⚙️ Key Engineering Decisions

### Decision 1 — Background Tasks Over Synchronous Processing

**Problem:** The ingestion pipeline takes 30–120 seconds depending on how many
articles are scraped and how many VirusTotal API calls are needed. A synchronous
endpoint would time out or block all other requests.

**Decision:** Used FastAPI's `BackgroundTasks` to run the pipeline asynchronously.
The endpoint returns `202 Accepted` immediately and the pipeline runs independently.

**Tradeoff:** The frontend needed a polling mechanism to check pipeline status.
I implemented a `/api/v1/status` endpoint that the dashboard polls every 5 seconds
during a run.

---

### Decision 2 — Dual-Layer IOC Extraction (LLM + Regex Fallback)

**Problem:** Gemini occasionally misses IOCs that are embedded in unusual sentence
structures, or returns them in inconsistent JSON formats.

**Decision:** Built a regex fallback engine that runs after Gemini. It catches
common IOC patterns (IPv4 regex, domain regex, hash length detection) as a safety
net. The results from both layers are merged and deduplicated before storage.

**Result:** Zero silent failures. Even when the LLM underperforms, the regex
catches the obvious IOCs.

---

### Decision 3 — Database-Level Deduplication

**Problem:** Running the pipeline daily means the same IOC (e.g., a known malicious
IP) will appear in multiple articles over time. Storing duplicates wastes space
and pollutes analyst dashboards.

**Decision:** Instead of checking for duplicates at the application level before
every insert, I applied a **unique index on the IOC value field in MongoDB.**
Any duplicate insert raises a `DuplicateKeyError` which the application catches
and silently skips.

**Why this is better than app-level deduplication:** It's atomic. If two pipeline
runs overlap (which can happen), app-level checks create race conditions. The
database-level constraint is always enforced regardless of concurrent access.

---

### Decision 4 — Analyst Approval Workflow Before Reporting

**Problem:** LLMs and regex both produce false positives. A domain name mentioned
in an article as an example might get extracted as an IOC. Pushing false positives
directly into threat intelligence reports destroys analyst trust in the system.

**Decision:** All IOCs land in a `pending` state. Nothing reaches the final report
until a human analyst reviews and approves it. This keeps the human in the loop
for the judgement call while automation handles the grunt work.

---

## 🧩 The Hardest Problems I Solved

### Problem 1 — VirusTotal API Rate Limiting

VirusTotal's free API tier allows 4 requests per minute. With 50–100 IOCs
extracted per pipeline run, naive sequential requests would hit the rate limit
within seconds and the pipeline would fail.

**Solution:** Implemented **exponential backoff with jitter** — when a 429 rate
limit response is received, the client waits, then retries with increasing delays.
I also batched IOC enrichment and added configurable sleep intervals between
requests to stay within quota. The pipeline slows down but never fails due to
rate limiting.

---

### Problem 2 — Gemini Returning Inconsistent JSON

When prompting Gemini to return structured IOC data, the response format was
inconsistent — sometimes valid JSON, sometimes JSON wrapped in markdown code
blocks, sometimes plain text with JSON embedded in it.

**Solution:** Built a response parser that:
1. Strips markdown code block wrappers if present
2. Finds the first `{` and last `}` and extracts the JSON substring
3. Falls back to regex extraction if JSON parsing still fails

This made the AI layer robust regardless of Gemini's output formatting quirks.

---

### Problem 3 — Keeping the Dashboard in Sync With Pipeline State

The pipeline runs asynchronously in the background, but the analyst needs to
see real-time progress — is it scraping? Is it running AI analysis? Is it done?

**Solution:** Maintained a pipeline state object in memory on the server
(current stage, IOC count, errors encountered, completion status). A
`/api/v1/status` polling endpoint exposes this state. The React dashboard
polls every 5 seconds and updates a progress indicator in real time.

---

## 🔐 Security Design

Even though this was an internal tool, I implemented proper security from the start:

```
Authentication:   JWT-based with expiry — no session storage needed
Secrets:          All API keys stored in .env — never hardcoded
API Security:     All endpoints except /auth/login require valid JWT
Data:             .env files excluded from all commits via .gitignore
Input:            IOC values sanitized before MongoDB insertion
```

---

## 📚 What I Learned

**Technical learnings:**

- How to structure a production FastAPI application with proper separation of
  concerns — routes, services, database layer, background tasks all in separate modules
- The practical reality of working with LLM APIs in production — they are powerful
  but need robust fallback logic and output parsing
- Why database-level constraints are always safer than application-level checks
  for data integrity problems
- How to handle third-party API rate limits gracefully without failing the pipeline
- The full React component lifecycle in a real dashboard — state management,
  polling, protected routes, JWT handling in Axios interceptors

**Professional learnings:**

- How a real SOC workflow looks from the inside — what analysts actually need
  from a threat intelligence tool
- Why the human-in-the-loop approval step is not optional in security tooling —
  false positives from AI are real and consequential
- How to scope a project that starts as an idea and becomes a working production
  system in under 3 months

---

## 🔁 What I Would Do Differently

**1. Containerize with Docker from Day 1**
Running the backend, frontend, and MongoDB as separate processes during development
was manageable, but setting it up on a new machine required following setup steps
carefully. A `docker-compose.yml` from the start would have made this one command.

**2. Add CERT-In as an Additional Feed**
The platform uses AlienVault OTX as its primary threat feed, but CERT-In advisories
are specifically relevant for Indian organizations. I would integrate the CERT-In
advisory RSS feed as a third collection source.

**3. Replace Manual Polling With WebSockets**
The dashboard polls the `/api/v1/status` endpoint every 5 seconds for pipeline
progress. This works but is inefficient. WebSockets would push updates from the
server the moment state changes — lower latency, less unnecessary network traffic.

**4. Use Antigravity Agents for Orchestration**
The pipeline stages (scrape → AI extract → enrich → store) are currently
orchestrated manually with sequential function calls in a background thread.
Using a proper multi-agent framework like Antigravity Agents would make each
stage independently retryable, observable, and easier to extend with new sources.

**5. Add MITRE ATT&CK Technique Mapping**
Currently the system extracts IOCs but does not map them to MITRE ATT&CK
techniques. Adding this layer would make the threat intelligence reports
significantly more actionable for SOC analysts.

---

## 🎯 Skills Demonstrated

```
Security Domain:
✓ Threat Intelligence pipeline design and implementation
✓ IOC extraction, classification, and enrichment
✓ Analyst workflow design (approval gates, reporting)
✓ Understanding of CTI lifecycle in a SOC context

Backend Engineering:
✓ FastAPI — async endpoints, background tasks, middleware, routing
✓ MongoDB — document design, indexing, deduplication strategy
✓ JWT authentication implementation
✓ REST API design and documentation
✓ Third-party API integration (Gemini, OTX, VirusTotal)
✓ Rate limit handling and retry logic
✓ PDF generation (ReportLab)

Frontend Engineering:
✓ React + Vite — component architecture, routing, state management
✓ Axios with JWT interceptors
✓ Real-time dashboard with polling
✓ Tailwind CSS

AI / LLM Engineering:
✓ Prompt engineering for structured data extraction
✓ LLM output parsing and error handling
✓ Regex fallback design for production robustness

General Engineering:
✓ End-to-end system design from scratch
✓ Security-first development practices
✓ Production deployment considerations
✓ Documentation and architectural thinking
```

---

## 🔗 Related Work

| Link | Description |
|:-----|:------------|
| [🌐 Portfolio](https://sushmanth-cybersecurity.vercel.app) | Full portfolio with project showcase |
| [💼 LinkedIn](https://www.linkedin.com/in/sushmanth-j-163335373/) | Professional profile |
| [🔍 OSINT Toolkit](../osint-investigation-framework) | My OSINT methodology repo |
| [🛡️ SOC Notes](../soc-analyst-toolkit) | Alert triage and SOC methodology |

---

<div align="center">

*Built with curiosity, caffeine, and a genuine belief that security automation*
*should make analysts faster — not replace their judgment.*

**Sushmanth** · SOC Analyst · Cyber Forensics · OSINT
`Andhra Pradesh, India`

</div>
