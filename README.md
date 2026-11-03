# Daily Job Search Automation (n8n + Claude API)

I built this during my own job search. Every morning it looks for new jobs, scores them against my profile, writes cover letters for the best matches and emails me everything ready to apply. I still press the apply button myself. The system only does the preparation.

## What it does

1. Runs every day at 8:00 (Asia/Riyadh time).
2. Sends five search queries to the JSearch API (RapidAPI), mixing local searches with remote roles.
3. Merges the results and drops anything already saved in my Google Sheet.
4. Filters in code by location, contract type and title rules, so the AI never sees obvious noise.
5. Scores each remaining job from 0 to 10 with Claude Haiku, using a prompt that describes my background and target roles.
6. Adds a small bonus for government and non-profit employers.
7. Logs every job to Google Sheets with its score and status.
8. For the strong matches (score 5 and above, up to 10 a day), Claude Sonnet writes a tailored cover letter.
9. Gmail sends me two emails: a digest of the day's jobs, and the cover letters with my CV attached from Google Drive.

## Stack

- n8n (self-hosted), with Schedule, HTTP Request, Code, Google Sheets, Gmail and Google Drive nodes
- JSearch API on RapidAPI for job listings
- Claude API: Haiku for scoring, Sonnet for cover letters
- JavaScript in the Code nodes for filtering, de-duplication and parsing the model output

## Design decisions

- **Cheap model for scoring, stronger model for writing.** Only jobs that pass the filters get scored, and only strong matches get a cover letter, so the cost per day stays small.
- **Filter before the model.** Location, full-time only, seniority and title exclusions and duplicate title and employer pairs are handled in plain code. It is faster, free and predictable.
- **Two scoring lanes.** The prompt treats automation, BI and data roles as the main lane and digital marketing as a second lane, so I do not miss jobs from either side.
- **Resilient runs.** The search step retries and continues on error, so one failed API call does not stop the whole day.
- **Semi-automatic on purpose.** It does not auto-apply. Many job portals block bots, and I want to read each job before sending anything.

## Limits and what I would improve

- The free JSearch plan gives 200 requests a month and each daily run uses 5, so a paid plan or a second source (for example Google Alerts feeds) is needed for higher volume.
- Track which jobs I actually apply to and use that history to tune the scoring prompt.
- Move the search queries into a Google Sheet so I can edit them without opening the workflow.

## Setup

1. Run n8n and create credentials for Google Sheets, Gmail, Google Drive, the JSearch header key and the Claude API key. Never put keys directly in a node.
2. Create a Google Sheet with the columns: job_id, job_title, employer_name, job_location, job_apply_link, date_posted, status and score.
3. Upload your CV to Google Drive and set its file ID in the Download node.
4. Edit the search queries, the city filter and the scoring prompt in the workflow to match your own profile.
5. Publish the workflow and set the timezone in the workflow settings.

## Author

Omar Al-Majnuni, automation and business systems. LinkedIn: linkedin.com/in/omar-almajnuni-ba1903172
