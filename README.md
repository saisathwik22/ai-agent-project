# Autonomous Sales Intelligence Agent
### A Multi-Agent AI System for End-to-End Lead Research & Personalised Outreach

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Problem Statement](#problem-statement)
3. [Solution](#solution)
4. [Key Features](#key-features)
5. [How It Works](#how-it-works)
6. [Agent Architecture](#agent-architecture)
7. [Technology Stack](#technology-stack)
8. [Course Concepts Demonstrated](#course-concepts-demonstrated)
9. [MCP Server & Tools](#mcp-server--tools)
10. [Security Design](#security-design)
11. [Project Structure](#project-structure)
12. [Setup & Requirements](#setup--requirements)
13. [Use Cases & Business Value](#use-cases--business-value)
14. [Submission Track](#submission-track)

---

## Project Overview

The **Autonomous Sales Intelligence Agent** is a production-grade, multi-agent AI system built with **Google Agent Development Kit (ADK)** that automates the end-to-end process of B2B sales lead research and personalised outreach generation.

Given just a company name and an Ideal Customer Profile (ICP), the system deploys a coordinated team of specialised AI sub-agents that work in parallel — researching the company's web presence, tracking recent news signals, scoring the lead against ICP criteria, and monitoring job postings — before synthesising all findings into a rich intelligence brief and a highly personalised cold email draft, which a human sales rep reviews and approves before anything is sent.

This project was built as a capstone submission for **Kaggle's 5-Day AI Agents: Intensive Vibe Coding Course with Google**, under the **Agents for Business** track.

---

## Problem Statement

Sales Development Representatives (SDRs) at B2B companies spend an estimated **4–6 hours per day** on manual lead research tasks:

- Browsing company websites to understand what a prospect does
- Searching for recent news — funding rounds, product launches, leadership hires
- Cross-referencing the lead against ideal customer criteria
- Looking up open job roles to understand the company's growth signals
- Writing a personalised email that references specific, recent context

This process is:
- **Slow** — deeply time-consuming and difficult to scale
- **Inconsistent** — quality varies heavily between SDRs
- **Expensive** — senior SDR time is wasted on research instead of conversations
- **Error-prone** — stale or incorrect information leads to irrelevant outreach

The result is low email open rates, wasted pipeline, and frustrated sales teams.

---

## Solution

The Autonomous Sales Intelligence Agent solves this by replacing the manual research workflow with a **coordinated multi-agent pipeline** that completes in under 2 minutes what previously took hours.

A single CLI command triggers the full pipeline:

```bash
python cli.py --company "Stripe" --icp "fintech, 500+ employees, US-based"
```

The system then:

1. Spins up four specialised sub-agents that run **in parallel**
2. Each agent researches a specific dimension of the lead
3. A synthesis agent merges all findings into a structured intelligence brief
4. An outreach agent drafts a highly personalised cold email grounded in the research
5. A **human approval UI** surfaces the draft for review — the human can edit, approve, or discard
6. On approval, the email is sent and the lead is logged to the CRM database

No email is ever sent without human sign-off. The agent assists; the human decides.

---

## Key Features

### Multi-Agent Parallelism
Four sub-agents execute simultaneously, cutting total research time from hours to under 2 minutes.

### Human-in-the-Loop Approval
A FastAPI-powered web review panel lets the sales rep read the intelligence brief, edit the draft email, and approve or reject — before any action is taken.

### MCP Server with Custom Tools
A dedicated Model Context Protocol (MCP) server exposes clean, reusable tools for web search, CRM logging, and email dispatch — decoupled from agent logic.

### ICP Scoring
The ICP scorer evaluates each lead against configurable criteria (industry, company size, geography, tech stack signals) and outputs a numeric fit score with reasoning.

### News & Trigger Signal Detection
The news analyst identifies buying signals — recent funding rounds, executive hires, product launches, expansions — that make outreach timely and relevant.

### Job Posting Intelligence
The jobs watcher reads open roles to infer company priorities, pain points, and growth areas — context that makes cold emails genuinely useful.

### Local CRM with Audit Trail
All leads, intelligence briefs, email drafts, and outcomes are stored in a local SQLite database — providing a complete audit trail with no external CRM dependency.

### Security-First Design
API keys are managed via environment variables, PII is redacted before logging, and no credentials are ever hardcoded in source code.

### Fully Deployable
The system is Dockerized and deployable to any cloud platform (Railway, Cloud Run, Render) with a single command.

---

## How It Works

### Step-by-Step Flow

```
User runs CLI command
        |
        v
Coordinator Agent receives company + ICP input
        |
        |-----> Web Research Agent    (parallel)
        |-----> News Analyst Agent    (parallel)
        |-----> ICP Scorer Agent      (parallel)
        |-----> Jobs Watcher Agent    (parallel)
        |
        v
All results collected and passed to Synthesis Agent
        |
        v
Synthesis Agent builds Lead Intelligence Brief
        |
        v
Outreach Agent drafts personalised cold email
        |
        v
Human Approval UI opens in browser
        |
    [Approve] ---------> Email sent via SMTP
        |                Lead logged to CRM DB
    [Edit + Approve] --> Edited email sent
        |
    [Discard] ---------> Lead marked as discarded in DB
```

### What Each Agent Does

**Coordinator Agent**
- The root agent built with Google ADK
- Receives the user's input (company name + ICP string)
- Spawns the four sub-agents and runs them in parallel
- Collects all results and passes them downstream
- Manages error handling if any sub-agent fails

**Web Research Agent**
- Uses the `search_web` MCP tool to query the company's website, LinkedIn page, and Crunchbase profile
- Extracts: company description, industry, founding year, employee count, headquarters, products/services, key executives
- Returns a structured company profile

**News Analyst Agent**
- Searches for recent news about the company (last 90 days)
- Identifies trigger events: funding announcements, acquisitions, product launches, leadership changes, awards, expansions
- Ranks signals by relevance and recency
- Returns a list of news events with source URLs and dates

**ICP Scorer Agent**
- Compares the company profile against the user-defined ICP criteria
- Scores across dimensions: industry match, size match, geography match, tech stack signals, growth indicators
- Outputs a score from 0–100 with a plain-English explanation of fit and gaps

**Jobs Watcher Agent**
- Searches for the company's open job postings
- Identifies roles that signal priorities and pain points (e.g. "Head of Data Engineering" = data infrastructure investment)
- Returns a list of relevant roles with inferred business context

**Synthesis Agent**
- Combines outputs from all four sub-agents into a single Lead Intelligence Brief
- Structures the brief with: company overview, trigger signals, ICP score + reasoning, inferred pain points, recommended talking points
- This brief is saved to the CRM database

**Outreach Agent**
- Reads the Lead Intelligence Brief
- Drafts a personalised cold email that:
  - References a specific recent trigger event
  - Connects the prospect's inferred pain point to your solution
  - Has a clear, low-friction call to action
  - Is concise (under 150 words)
- Returns subject line + email body

**Human Approval Gate**
- A lightweight FastAPI web interface that opens automatically
- Displays the full intelligence brief alongside the draft email
- Allows the human to edit the email inline before approving
- Approve → triggers email send + CRM log
- Discard → marks lead as discarded in CRM

---

## Agent Architecture

The system uses a **hierarchical multi-agent architecture** built on Google ADK:

```
Root Agent (Coordinator)
├── Sub-Agent: Web Researcher
│   └── Tools: search_web, fetch_page
├── Sub-Agent: News Analyst
│   └── Tools: search_web, filter_by_date
├── Sub-Agent: ICP Scorer
│   └── Tools: score_lead
└── Sub-Agent: Jobs Watcher
    └── Tools: search_web, parse_jobs

Synthesis Agent
└── Tools: build_brief

Outreach Agent
└── Tools: draft_email

MCP Server (shared tool layer)
├── search_web        → Serper API
├── log_to_crm        → SQLite DB
├── send_email        → SMTP
└── score_lead        → Scoring logic
```

All agents share a common MCP server, meaning tools are defined once and reused across agents — a clean separation of concerns between reasoning (agents) and capability (tools).

---

## Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| Agent framework | Google ADK (`google-adk`) | Multi-agent orchestration |
| LLM | Gemini 1.5 Flash | Reasoning backbone for all agents |
| Tool protocol | MCP (`mcp` Python SDK) | Standardised tool exposure |
| Web search | Serper API | Google search results via API |
| Approval UI | FastAPI + Uvicorn | Human-in-the-loop review panel |
| CRM storage | SQLite + `sqlite-utils` | Local lead and audit database |
| Email dispatch | SMTP (smtplib) | Sending approved outreach |
| CLI | Python argparse + `rich` | Terminal entrypoint with live output |
| Config & secrets | `python-dotenv` | Environment variable management |
| Containerisation | Docker | Packaging and deployment |
| Deployment | Railway / Cloud Run | Cloud hosting |
| Data validation | Pydantic | Structured agent output schemas |
| HTTP client | httpx | Async HTTP requests |

---

## Course Concepts Demonstrated

This project demonstrates all six key concepts from the course:

| Concept | Where Demonstrated | How |
|---|---|---|
| **Multi-agent system (ADK)** | Code — `agents/` | Coordinator + 4 parallel sub-agents + synthesis + outreach agents, all built with Google ADK |
| **MCP Server** | Code — `mcp_server/` | Custom MCP server exposing `search_web`, `log_to_crm`, `send_email`, `score_lead` tools |
| **Antigravity** | Video | Project built entirely inside Antigravity IDE, demonstrated live in the 5-minute video |
| **Security features** | Code + Video | API keys via `.env`, PII redaction before CRM logging, no hardcoded credentials, audit trail |
| **Deployability** | Video | Dockerfile included, deployed to Railway, one-command setup shown in video |
| **Agent skills (CLI)** | Code — `cli.py` | Full CLI entrypoint with `--company` and `--icp` flags, rich terminal output showing agent progress |

---

## MCP Server & Tools

The MCP server is the shared capability layer for all agents. It is defined in `mcp_server/server.py` and exposes the following tools:

### `search_web(query: str, num_results: int) -> list[dict]`
Calls the Serper API to run a Google search. Returns a list of results with title, URL, and snippet. Used by web researcher, news analyst, and jobs watcher agents.

### `score_lead(company_profile: dict, icp_criteria: str) -> dict`
Evaluates a company profile against ICP criteria. Returns a score (0–100), a breakdown by dimension, and a plain-English fit summary.

### `log_to_crm(lead_data: dict) -> str`
Writes a lead record to the local SQLite database. Includes the company name, intelligence brief, email draft, ICP score, timestamp, and approval status. PII fields are redacted before storage.

### `send_email(to: str, subject: str, body: str) -> bool`
Sends an email via SMTP using credentials from environment variables. Only callable after human approval is recorded in the database.

---

## Security Design

Security is a first-class concern in this system:

- **No hardcoded secrets** — all API keys and credentials live in `.env`, loaded via `python-dotenv`. The `.env` file is in `.gitignore`.
- **`.env.example`** — a safe template with placeholder values is committed to the repo so others can set up without exposing real credentials.
- **PII redaction** — before any lead data is written to the CRM database, email addresses and phone numbers are redacted using regex patterns.
- **Human approval gate** — the system cannot send any email without an explicit human approval action in the UI. The `send_email` tool checks for an approval record in the database before executing.
- **Audit trail** — every pipeline run is logged with timestamp, agent outputs, human decision, and outcome — providing full accountability.
- **Input validation** — all agent outputs are validated against Pydantic schemas before being passed downstream, preventing prompt injection from propagating through the pipeline.

---

## Project Structure

```
sales-intelligence-agent/
├── agents/
│   ├── __init__.py
│   ├── coordinator.py       # Root ADK agent, orchestrates all sub-agents
│   ├── web_researcher.py    # Researches company website and LinkedIn
│   ├── news_analyst.py      # Finds recent news and trigger signals
│   ├── icp_scorer.py        # Scores lead against ICP criteria
│   └── jobs_watcher.py      # Reads open job postings for signals
├── mcp_server/
│   ├── __init__.py
│   ├── server.py            # MCP server definition and tool registry
│   └── tools/
│       ├── __init__.py
│       ├── search.py        # Web search tool (Serper API)
│       ├── crm.py           # CRM logging tool (SQLite)
│       └── email_tool.py    # Email dispatch tool (SMTP)
├── api/
│   ├── __init__.py
│   └── approval_ui.py       # FastAPI human-in-the-loop review panel
├── cli.py                   # CLI entrypoint with argparse + rich output
├── .env.example             # Safe credentials template
├── .env                     # Real credentials (gitignored)
├── .gitignore
├── requirements.txt
├── Dockerfile
└── README.md
```

---

## Setup & Requirements

### Prerequisites
- Python 3.11+
- Docker (optional, for containerised deployment)
- A Serper API key (free tier available at serper.dev)
- A Gemini API key (free via Google AI Studio)
- An SMTP account for email sending (Gmail works)

### Environment Variables

```env
GEMINI_API_KEY=your_gemini_api_key_here
SERPER_API_KEY=your_serper_api_key_here
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password_here
CRM_DB_PATH=./crm.db
```

### Installation

```bash
git clone https://github.com/yourusername/sales-intelligence-agent
cd sales-intelligence-agent
pip install -r requirements.txt
cp .env.example .env
# Fill in your credentials in .env
```

### Running

```bash
python cli.py --company "Stripe" --icp "fintech, 500+ employees, US-based"
```

### Docker

```bash
docker build -t sales-agent .
docker run --env-file .env sales-agent --company "Stripe" --icp "fintech, 500+ employees"
```

---

## Use Cases & Business Value

### Who benefits
- **B2B SaaS companies** with outbound sales motions
- **Sales development teams** doing high-volume prospecting
- **Founders** doing early-stage outbound without a sales team
- **Agencies** running outreach campaigns for clients

### Measurable impact
- Reduces per-lead research time from **4–6 hours to under 2 minutes**
- Increases email personalisation quality and relevance consistently
- Enables a single SDR to process **10x more leads** per day
- Creates a searchable, auditable record of all outreach activity

### Why agents, specifically
This problem is uniquely suited to agents because it requires:
- **Multi-step reasoning** across different information sources
- **Parallel execution** of independent research tasks
- **Tool use** (web search, database write, email send)
- **Human oversight** before consequential actions (sending emails)
- **Structured output** that flows between pipeline stages

A simple LLM prompt cannot do this reliably. An agent system can.

---

## Submission Track

**Track:** Agents for Business

**Rationale:** This project directly addresses a high-cost, high-frequency business problem — lead research and outreach — with a measurable impact on sales efficiency and pipeline quality. The use of agents is central and non-trivial: the multi-agent architecture is what makes the parallelism, specialisation, and human-in-the-loop design possible.

---

*Built for Kaggle's 5-Day AI Agents: Intensive Vibe Coding Capstone — Agents for Business track.*
*Developed using Antigravity IDE, Google ADK, Gemini, and the MCP protocol.*
