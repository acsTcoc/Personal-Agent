# uni-agent

A small, self-hosting agent that runs your job search and study planning once a
day — for free — using GitHub Actions as the "always on" engine. No server to
maintain, no subscription, and the whole thing lives in this repo.

## What it does today

- **Study planner & tutor**: every morning it opens a GitHub issue with today's
  study plan, built from the feedback you left on *yesterday's* issue. Reply to
  the issue with what you covered and how it went — tomorrow's plan adapts from
  that reply. No app, no separate login, just issue comments.
- **Job scouting**: pulls new postings matching your keywords/location from the
  [Adzuna](https://developer.adzuna.com/) and [RemoteOK](https://remoteok.com/api)
  APIs — real APIs, not scraping, so nothing here violates a job board's terms
  of service.
- **Cover letters**: for matches that include a contact email in the posting,
  drafts a tailored cover letter from your resume + the job description.
- **Applications**: stored in the database either way. Turning a draft into an
  actual Gmail draft (never auto-sent) is one optional, separate step — see
  `src/gmail_helper.py`.
- **Daily report**: everything above lands in one GitHub issue per day, which
  doubles as your feedback form for the tutor.

## What it deliberately does *not* do

- No LinkedIn/Indeed scraping or auto-apply bots — most job boards' terms
  explicitly prohibit this, and account bans are common. Job sourcing here is
  limited to sources with a real public API.
- No auto-sending of emails by default (`AUTO_SEND_EMAILS=false`). Cover
  letters are drafted and stored for your review; flip the switch in
  `.env`/repo variables once you trust the output.

## Project layout

```
src/
  config.py          loads all settings from env vars / GitHub secrets
  db.py               SQLite schema + helpers (jobs, applications, study log, feedback)
  job_scout.py        fetches + filters jobs from Adzuna and RemoteOK
  cover_letter.py      generates a tailored cover letter via the Claude API
  tutor.py            generates the daily study plan from recent feedback
  github_helper.py     creates/reads the daily GitHub issue
  gmail_helper.py      optional: turns a drafted letter into a Gmail draft
  daily_workflow.py    entry point — orchestrates all of the above
data/
  uniagent.db          created on first run, committed back to the repo daily
  resume.txt           paste a plain-text version of your resume here
.github/workflows/
  daily.yml            the cron schedule that runs everything
```

## Setup

1. **Fork or clone this repo.**
2. **Get two free API keys:**
   - Anthropic API key — [console.anthropic.com](https://console.anthropic.com)
   - Adzuna app ID + key (free developer tier) — [developer.adzuna.com](https://developer.adzuna.com/)
3. **Add repo secrets** (Settings → Secrets and variables → Actions → New repository secret):
   - `ANTHROPIC_API_KEY`
   - `ADZUNA_APP_ID`, `ADZUNA_APP_KEY`
   - (`GITHUB_TOKEN` is provided automatically — nothing to add)
4. **Add repo variables** (same page, "Variables" tab) for your search criteria:
   - `JOB_KEYWORDS` — e.g. `software engineering intern, data analyst intern`
   - `JOB_LOCATION`, `JOB_EXCLUDE`, `REMOTE_ONLY`
5. **Paste your resume** as plain text into `data/resume.txt`.
6. **Enable Actions** on the repo (Actions tab → "I understand, enable"). The
   workflow runs daily at the time set in `.github/workflows/daily.yml`
   (schedules are in UTC — adjust for your timezone).
7. You can also trigger a run immediately from the Actions tab
   ("Run workflow") instead of waiting for the schedule.

### Testing locally first (recommended)

```bash
cp .env.example .env   # fill in your keys
pip install -r requirements.txt
python -m src.daily_workflow
```

You'll need a personal access token (`repo` scope) as `GITHUB_TOKEN` in `.env`
for this to open a real issue on your repo when testing locally.

## Roadmap

- [ ] Course-catalog / degree-requirement planning module
- [ ] More job sources (anything with a public API — see `job_scout.py`)
- [ ] Wire `gmail_helper.py` into `daily_workflow.py` once you trust the drafts
- [ ] Weekly summary issue in addition to the daily one

## License

MIT — see `LICENSE`.
