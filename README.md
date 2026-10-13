<p align="center"><img src="logo.png" width="96" alt="ATS Job Pulse logo"></p>

# ATS Job Pulse — company jobs & hiring signals for AI agents

**Who is hiring, for what, and what changed since yesterday?**

ATS Job Pulse is a hosted tool (an [Apify Actor](https://apify.com/hirepulse/ats-job-pulse)) that takes a company domain and returns the company's open jobs plus a compact hiring summary, straight from the company's own job board. It works as an **MCP tool** for Claude, Cursor and other AI agents, and as a plain HTTP API.

- **Auto-detects the job board** from a domain: Greenhouse, Lever, Ashby, Workable, Recruitee, SmartRecruiters
- **Hiring signals**: open jobs, new and closed jobs since the last run, breakdown by department, location and technology
- **Per job**: title, department, location, remote flag, salary range (when published), posted date, technologies, apply URL
- **Small output on demand**: filters and a summary-only mode sized for an agent's context window
- **Pay per company**: $0.05 per successfully processed company. Companies that are not found are free

➡️ **Run it:** https://apify.com/hirepulse/ats-job-pulse

## Use it from an AI agent (MCP)

Add the Apify MCP server with this tool enabled to your MCP client:

```json
{
  "mcpServers": {
    "ats-job-pulse": {
      "url": "https://mcp.apify.com/?actors=hirepulse/ats-job-pulse"
    }
  }
}
```

The first connection opens a browser window to sign in to Apify (a free account is enough to start).

Then ask, for example:

- "How many open jobs does stripe.com have, and in which departments?"
- "Which of ramp.com, linear.app and notion.so opened new engineering roles since the last check?"
- "Does figma.com hire for Kubernetes or LLM skills? Show salary ranges."
- "List remote data-engineering jobs at these ten companies."

## Use it over HTTP

```bash
curl -X POST "https://api.apify.com/v2/acts/hirepulse~ats-job-pulse/run-sync-get-dataset-items?token=YOUR_APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"companies": ["stripe.com", "linear.app"], "output_mode": "summary_only"}'
```

## Input

Only `companies` is required. See [`examples/`](examples) for ready-to-use inputs.

| Field | Default | Description |
| --- | --- | --- |
| `companies` | *required* | Domains (`stripe.com`) or board slugs (`stripe`). Prefix to force an ATS: `greenhouse:stripe`, `lever:spotify`, `ashby:linear`, `workable:huggingface`, `recruitee:bunq`, `smartrecruiters:BoschGroup`. |
| `output_mode` | `jobs_and_summary` | `jobs_and_summary`, `summary_only` (smallest) or `jobs_only`. |
| `track_diff` | `true` | Flag each job NEW / EXISTING / CLOSED versus the previous run. |
| `state_id` | `default_state` | Name of the saved history. Runs sharing a `state_id` are compared. |
| `only_changes` | `false` | Return only NEW and CLOSED jobs. |
| `title_keywords`, `departments`, `locations`, `technologies` | | Keep jobs matching any of the given values. |
| `remote_only` | `false` | Keep remote jobs only. |
| `max_jobs_per_company` | `0` | Cap on job records per company (`0` = no limit). |
| `extract_tech_keywords` | `true` | Detect technologies in job titles and descriptions. |

## Output

Two kinds of records, told apart by `record_type`.

**`company_summary`** — one per company:

```json
{
  "record_type": "company_summary",
  "company_domain": "linear.app",
  "ats_type": "ashby",
  "total_active_jobs": 31,
  "remote_jobs": 31,
  "jobs_with_salary": 12,
  "is_first_run": false,
  "new_jobs": 2,
  "closed_jobs": 1,
  "new_job_titles": ["Senior Product Engineer", "Enterprise Account Executive"],
  "closed_job_titles": ["Support Specialist"],
  "jobs_by_department": { "GTM": 15, "Product": 13, "Operations": 3 },
  "top_locations": { "North America": 19, "Europe": 8 },
  "top_technologies": { "React": 8, "TypeScript": 8, "GraphQL": 7 }
}
```

**`job`** — one per job posting:

```json
{
  "record_type": "job",
  "company_domain": "ramp.com",
  "ats_type": "ashby",
  "job_id": "…",
  "title": "Security Engineer, Cloud",
  "department": "Engineering",
  "location": "New York, NY",
  "is_remote": false,
  "posted_date": "2026-09-03T13:30:34+00:00",
  "salary": "$211.4K - $290.6K",
  "apply_url": "https://jobs.ashbyhq.com/ramp/…",
  "detected_technologies": ["AWS", "Kubernetes", "Python", "Terraform"],
  "status": "NEW"
}
```

## Limitations

- Companies on other systems (Workday, iCIMS, Taleo, in-house career sites) are reported as not found and are not charged.
- Salary is returned as the employer published it and is not normalized. Many postings state no range.
- SmartRecruiters publishes no job description in its listing, so technologies there come from the job title only.
- A bare slug can occasionally match a different company with the same name on another ATS. Use a prefix (`lever:acme`) to be exact.

## Support

Questions and bug reports: open an issue here, or use the **Issues** tab on the [Actor page](https://apify.com/hirepulse/ats-job-pulse).
