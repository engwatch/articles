# How I Replaced the Finance Department with a Chain of AI Agents

I have a simple rule: if a task repeats more than twice — it's time to delegate. I used to delegate to people. Now — to AI agents that work with databases, spreadsheets, and task trackers directly.

Over three months, I built a system of 17 AI skills that took over all financial reporting for a B2B platform with 200+ partners. Without a single line of traditional code.

---

## Context: A B2B Platform and Financial Routine

I work as a PO at a B2B platform at the intersection of insurance and finance. We aggregate financial products for partners: banks, lending institutions, marketplaces. Each partner wants to see their own funnel — applications, conversions, disbursements, commissions.

With 200+ active partners, reporting becomes a conveyor belt:

- **Daily reports** — every morning a partner expects yesterday's numbers
- **Monthly reconciliations** — hundreds of rows of deals broken down by vendors, with different revenue-split percentages
- **Ad-hoc requests** — "show me the conversion for this product over the last two weeks"
- **Legal documents** — generating standard contracts for new partners

2–3 hours a day, two people. A textbook case for delegation — but there was nobody to delegate to. Then a tool appeared that can work with SQL, APIs, and spreadsheets on its own.

## Not One Agent — a Chain of Agents

An important distinction: this isn't "one smart bot that does everything." It's a **system of specialized agents**, each solving a specific task. In Claude Code terminology, each agent is a skill (slash-command) with clearly defined logic.

The agents are connected through shared infrastructure:

- **MCP server for the database** (MS SQL, read-only) — the agent builds SQL queries and retrieves data
- **Google Sheets API** — creating spreadsheets, writing data, formatting, formulas
- **Google Drive** — file storage, uploading vendor exports
- **Task tracker** — syncing with team tasks
- **GitLab API** — analyzing development team productivity

One agent generates financial reports. Another runs monthly reconciliations. A third collects dev team statistics. A fourth transcribes meeting recordings and produces structured reports. They don't know about each other, but they operate within a single ecosystem.

## How the Most Frequently Used Skill Works: The Financial Report

A manager enters a single command — the partner name and a date range. 40 seconds later, a complete report appears in Google Sheets. Here's what happens during those 40 seconds:

**1. Counterparty Resolution**
The agent maintains a mapping of human-readable aliases to internal identifiers. No need to remember UUIDs — a short name is enough.

**2. Database Query**
A SQL query is generated that builds the funnel for a financial product: applications → transitions → disbursements → commissions. The query accounts for the correct deal statuses — that's a story in itself, because the database documentation and the actual enum values don't always match.

**3. Aggregation**
Data is grouped by day, conversion rates between stages are calculated, commission totals are summed. Different partners have different revenue-sharing terms — the agent handles that.

**4. Writing and Formatting**
A spreadsheet is created or updated: headers, data, summary rows, number formats, column widths. Formatting is applied strictly after data is written — if you do both simultaneously, the API sometimes applies styles to empty cells (the data hasn't arrived yet).

**5. Self-Verification**
The agent re-reads what it wrote and compares it against the source data. This isn't paranoia — Google Sheets API actually drops rows during batch writes. Without verification, errors are inevitable.

## Monthly Reconciliations: When One Agent Isn't Enough

The daily report is a relatively straightforward task. Monthly reconciliation is a different league entirely.

A single partner's traffic might flow through 3–4 vendors. Each has its own export format, its own rules for matching deals, its own commission rates. You need to:

1. Download vendor files from cloud storage
2. Match each deal against data in our database
3. Calculate commissions with different splits for different partners
4. Generate a summary sheet with formulas
5. All in one document, with separate tabs for each counterparty

This used to take a full workday. Now — a command with two parameters and 3–5 minutes of waiting.

## What Else the System Covers

Over three months, the agents expanded well beyond reporting:

| Skill | Task | Time |
|-------|------|------|
| Financial Report | Funnel for a single partner | 40 sec |
| Daily Digest | Report across all active partners | 2–3 min |
| Monthly Reconciliation | Vendor reconciliation with formulas | 3–5 min |
| Contract Generator | Standard contract for a new partner | 1–2 min |
| Team Tracker | Product team management | interactive |
| GitLab Analytics | Developer activity comparison across 17 repos | 2–3 min |
| Meeting Report | Meeting transcription (600+ segments) → structured report | 3–5 min |
| Recruiting | Candidate search by parameters | interactive |

17 skills, 9 MCP servers (SQL, Google Sheets, Drive, Calendar, Gmail, GitLab, Tracker, Telegram, GitHub). Not a single line of Python, Go, or TypeScript. All logic lives in markdown files with instructions for the agent.

## Why Claude Code, Not LangChain

I tried several approaches:

- **ChatGPT + Zapier** — too primitive, no direct database access
- **LangChain/LangGraph** — requires a developer, long iteration cycles
- **Custom API + GPT-4** — works, but every change = rebuild and redeploy

Claude Code won because of one thing: **a 30-second feedback loop**. Write an instruction → test → fix → it works. No deployment, no CI/CD, no Docker. MCP servers give the agent direct access to infrastructure, and skills are just text files containing business logic.

Architecture:

- **Claude Code** — runtime, the brain
- **MCP servers** — bridge to external systems (database, Google, GitLab, Tracker)
- **Skills** — markdown files with instructions (business logic)
- **Google Sheets/Drive** — presentation layer for users

## Pitfalls Worth Knowing About

**Google Sheets API is not atomic.** Write 100 rows — you might find only 97 in the spreadsheet. The rule: always re-read and verify after every write. Without this, bugs will appear regularly and silently.

**Database documentation lies.** The MCP server for SQL returns a glossary with table descriptions. But actual enum values may not match the documentation. We lost an entire day before we started joining reference tables directly instead of trusting the glossary.

**Formatting is a separate step.** If you write data and apply styles simultaneously, the API may format empty cells (the data hasn't landed yet). Data first → verification → then visuals.

**Context is not infinite.** When processing large volumes (200+ rows), the agent can lose its initial context. The solution: the agent keeps a log of every step and periodically re-reads it. This sounds odd, but it works — essentially, the agent creates its own "external memory."

**Merge commits inflate statistics.** When analyzing productivity via the GitLab API, it turned out that both feature commits and merge commits are counted separately. And auto-generated code (ORM migrations) can bloat statistics by 20,000+ lines. Without filtering, the numbers are meaningless.

**People's aliases don't match across systems.** The same developer can appear under different names in GitLab, the task tracker, and corporate email. The agent maintains an alias mapping — without it, reports for a team of 11 simply won't come together.

## The Numbers

| Metric | Before | After |
|--------|--------|-------|
| Daily reports | 2–3 hours/day | 5–10 minutes |
| Monthly reconciliation | 8 hours | 5 minutes |
| Contract generation | 2–3 hours | 2 minutes |
| Errors in reports | 3–5/month | 0 (verification) |
| People on financial routine | 2 | 0 (PO handles it) |

~60 person-hours saved per month. Cost of Claude Code — $200/month on the Max plan.

## Three Things I Learned

**AI agents are a manager's tool, not a developer's.** A PO who understands the process builds working automation faster than a team going through the classic cycle of specs → sprint → review → deploy. Because there's no translation from "business language" to "code" and back.

**MCP servers are what turn a chatbot into an operator.** Without access to databases and APIs, an agent can only generate text. With MCP, it becomes a full participant in the workflow — operating the same systems a human employee would.

**Verification matters more than generation.** 80% of the effort went not into prompts, but into the verification system. Every step is checked: did the data get written? Do the formulas add up? Is the formatting correct? An agent that doesn't check its own work is an agent that will eventually fail silently.

---

*If you're a manager spending hours on data routine — start with one skill. Automate the most painful report. Once you see a three-hour task completed in a minute — the other 16 skills will follow naturally.*

---

**About the author:** Georgii Motrenko — PO / Head of Product Growth in a B2B InsurTech/FinTech platform. Building chains of AI agents to automate financial and product operations.

**Follow me:**
- [LinkedIn](https://www.linkedin.com/in/geo-m-551b69349/)
- [Telegram](https://t.me/danielspe_chanel) — Geo in IT channel
- [Threads](https://www.threads.com/@geo.m.ru)
