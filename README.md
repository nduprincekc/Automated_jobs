# 🤖 AI Automation Daily Job Feed (n8n)
This project is an **end-to-end automated job-feed pipeline** built with [n8n](https://n8n.io).
It runs daily, scrapes fresh job listings from **JobRight** and **JobberMan**, normalizes and filters them by posting time, and appends only the freshest matches to **Google Sheets** — completely hands-free.

---

## 🧠 Overview

This workflow solves a problem anyone job-hunting across multiple boards runs into: every site reports dates differently, and manually checking two boards every day for "what's new since yesterday" wastes time. This pipeline scrapes both sources in parallel, standardizes their timestamps into one reliable format, keeps only postings from the last 12 hours, and logs them automatically — no manual checking required.

---

## ⚙️ How It Works

### 1. Setup & Trigger
The workflow starts with a **Schedule Trigger** that runs on a fixed daily interval, kicking off two parallel branches at once — one per job board.

---

### 2. Job Scraping (Dual Source)
Each branch runs independently:

- **Run Actor (Apify)** → Executes the JobRight or JobberMan scraper actor.
- **Get Dataset Items** → Pulls the actor's scraped results (title, link, date posted, company details).
- **Edit Fields** → Maps raw scraped fields into a consistent schema across both sources.

---

### 3. Date Normalization
This is the core engineering piece of the workflow. The two sources report dates in incompatible formats:

- **JobRight** → relative text (`"47 minutes ago"`, `"3 hours ago"`)
- **JobberMan** → ISO 8601 timestamps (`2026-07-30T00:00:00.000000Z`)

A **Code (JavaScript)** node parses both formats into a single reliable ISO datetime, using Luxon's `DateTime`, anchored to the workflow's fixed execution time (`$now`) so results stay consistent across the run. It also generates a clean, human-readable display date (e.g. `July 30, 2026`) for the spreadsheet.

---

### 4. Freshness Filtering
- **If Node ("Checking the time")** → Compares each listing's normalized date against `$now.minus({ hours: 12 })`.
  - **True** → Listing is fresh → passed to Google Sheets.
  - **False** → Discarded (JobRight branch) or logged separately (JobberMan branch).

---

### 5. Structured Logging & Deduplication
- **Append or Update Row (Google Sheets)** → Writes each qualifying listing (title, date, source, link, details) to the shared results sheet.
- **Column to match on: `link`** → Prevents duplicate rows across daily runs; if a listing already exists, its row is updated instead of duplicated.

---

## 🧩 Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** | Automation and orchestration |
| **Apify Actors** | Job data source (JobRight, JobberMan scrapers) |
| **JavaScript (Luxon `DateTime`)** | Date parsing and normalization |
| **Google Sheets** | Structured results database with dedup logic |

---

## 🗓️ Automation Flow

1. Trigger → Two parallel scrape branches
2. Scrape → Raw job listings (JobRight + JobberMan)
3. Normalize → Consistent ISO + display dates across sources
4. Filter → Keep only postings from the last 12 hours
5. Save to Sheet → Deduplicated, structured results

---

## 🔒 Security Notice

All OAuth and API credentials have been **removed** from the exported workflow JSON for security reasons.
After importing this workflow into n8n, you'll need to **reconnect your own Apify and Google Sheets credentials**.
