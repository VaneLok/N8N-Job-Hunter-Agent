# N8N Job Hunter Agent

Automated job discovery, triage, and tracking for Ontario (Canada) with **n8n**, **Google Sheets**, and **OpenAI**.  

This workflow searches for job postings, filters them for location and freshness, evaluates fit with an ATS-style AI model, generates a compact job “snapshot,” and appends the results to a Google Sheet for tracking.

---

## ✨ Features

- **Automated scheduling** — Runs twice daily by default (07:00 & 15:00).
- **Location filter** — Focused on Ontario roles, with option to include remote Canada.
- **AI match evaluation** — ATS-style scoring with decision (`apply` / `skip`).
- **Snapshot card** — Compact summary of the job with comp info, highlights, and flags.
- **De-duplication** — Prevents duplicate entries based on job URL.
- **Google Sheets integration** — Appends structured results to your tracker sheet.
- **Secure config** — All PII, API keys, and IDs are externalized as environment variables.

---

## 📂 Repository Layout

```

.
├─ README.md
├─ workflow/
│  └─ n8n-job-hunter-agent.sanitized.json   # Import into n8n
└─ examples/
└─ sample-tracker-schema.csv              # Suggested tracker schema

````

---

## 🛠 Prerequisites

- **n8n** (Docker or desktop)
- **Google account** with access to Sheets (and Docs if generating resumes/covers)
- **OpenAI** API key
- (Optional) **SerpAPI** key for live job searches

---

## 🔑 Environment Variables

Define these in **n8n → Settings → Environment variables** (or your Docker Compose file).

| Variable | Required | Description |
|----------|----------|-------------|
| `APPLICANT_FULL_NAME` | ✅ | Applicant name |
| `APPLICANT_EMAIL` | ✅ | Applicant email |
| `APPLICANT_PHONE` | ✅ | Applicant phone |
| `APPLICANT_RESUME_TEXT` | ✅ | Resume (plain text for AI match) |
| `GDRIVE_RESUME_TEMPLATE_ID` | ➖ | Google Doc template ID for resume |
| `GDRIVE_COVER_TEMPLATE_ID` | ➖ | Google Doc template ID for cover letters |
| `GDRIVE_TARGET_FOLDER_ID` | ➖ | Target Drive folder ID |
| `SHEET_ID` | ✅ | Google Sheet ID (tracker) |
| `SHEET_RANGE` | ➖ | Defaults to `AppliedMemory!A1:Z1` |
| `SEARCH_LOCATION` | ➖ | Default `Ontario, Canada` |
| `SEARCH_ONTARIO_ONLY` | ➖ | Default `true` |
| `SEARCH_INCLUDE_REMOTE_CA` | ➖ | Default `true` |
| `SEARCH_RECENCY_DAYS` | ➖ | Default `20` |
| `RATE_LIMIT_BATCH_SIZE` | ➖ | Default `6` |
| `RATE_LIMIT_MIN_WAIT_SEC` | ➖ | Default `30` |
| `RATE_LIMIT_MAX_WAIT_SEC` | ➖ | Default `90` |

---

## 🔒 Credentials in n8n

Set up these credentials inside n8n:

1. **OpenAI API** — for AI match & snapshot
2. **Google Sheets** — OAuth2 connection
3. **SerpAPI** — optional (HTTP Header Auth recommended)

---

## 🚀 Setup

1. Clone the repo:
   ```bash
   git clone https://github.com/<your-org-or-user>/n8n-job-hunter-agent.git
   cd n8n-job-hunter-agent```

2. Import workflow:

   * n8n → Workflows → **Import from file**
   * Select `workflow/n8n-job-hunter-agent.sanitized.json`

3. Configure environment variables & credentials.

4. Create a Google Sheet with the required columns:

   * `apply_key`, `title`, `company`, `location`, `link`, `source`, `posted_at`, `captured_at`, `status`, `ai_match_score`, `ai_decision`, `ai_reasons`, `ai_must_have_misses`, `ai_resume_keywords`, `snapshot_card`

5. Run the workflow:

   * Test with **mock job data** first.
   * Verify rows append to your Google Sheet.

6. (Optional) Enable **SerpAPI nodes** for live job searches.

---

## 🔍 Workflow Overview

```
flowchart LR
  A[Schedule Trigger] --> B[Config (env vars)]
  B --> C[Build Searches]
  C --> D[SerpAPI Jobs (optional)]
  C --> E[Mock Job Generator]
  D --> F[Normalize + Freshness]
  E --> F
  F --> G[Provincial Gate (Ontario/Remote CA)]
  G --> H[De-dupe vs Tracker]
  H --> I[Matchmaker AI]
  I --> J[Verdict Vortex (IF: apply & score ≥65)]
  J --> K[Snapshot AI]
  K --> L[Write to Google Sheets]
```

---

## ⚙️ Customization

* **Keywords** — adjust in `Config` or via env vars.
* **Scoring threshold** — modify in `Verdict Vortex (IF)` (default: 65).
* **Location rules** — edit regex in `Provincial Gate`.
* **Output schema** — update **Registry Desk (Prep Rows)**.

---

## 🔐 Security

* No secrets are in this repo.
* All sensitive values come from env vars or n8n Credentials.
* Recommended: disable execution data logging in workflow settings.

---

## 🙏 Acknowledgements

* [n8n](https://n8n.io/) for the workflow automation
* [OpenAI](https://openai.com/) for AI-powered evaluations
* [SerpAPI](https://serpapi.com/) for optional job search integration
