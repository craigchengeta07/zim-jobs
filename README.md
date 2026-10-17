# Zimbabwe ICT Jobs Agent

Every morning at 06:00 (Zimbabwe time) this project:

1. visits job sites and collects links to new adverts,
2. reads each new advert and uses Claude to pull out the title, employer, deadline, level and requirements, and to decide whether it suits students or graduates in **Software Engineering, Information Technology, Information Security and Assurance, or Computer Science**,
3. updates a website (`docs/index.html`) that students can open on their phones,
4. optionally sends a Telegram message with the new jobs and the ones closing soon.

It runs for free on GitHub (Actions + Pages). The only paid part is the Claude API, which costs very little with the default small model.

## Files

| File | What it does |
|---|---|
| `scripts/update_jobs.py` | The agent: find, read, analyse, save, notify |
| `config.json` | Which sites to read, and keywords for the quick pre-filter |
| `docs/index.html` | The website students use |
| `docs/jobs.json` | The jobs the website shows (rewritten every day) |
| `data/seen.json` | Adverts already checked, so nothing is processed twice |
| `.github/workflows/daily.yml` | The 06:00 schedule |

## Setup (about 15 minutes)

1. **Create a GitHub repository** and upload all these files (keep the folder structure, including the hidden `.github` folder).
2. **Get a Claude API key** at console.anthropic.com. In the repo go to *Settings > Secrets and variables > Actions > New repository secret* and add `ANTHROPIC_API_KEY`.
3. **Turn on the website:** *Settings > Pages*, set Source to "Deploy from a branch", branch `main`, folder `/docs`. Your site will be at `https://YOUR-USERNAME.github.io/REPO-NAME/`.
4. **Allow the bot to save results:** *Settings > Actions > General > Workflow permissions*, choose "Read and write permissions".
5. **Run it once by hand:** open the *Actions* tab, choose "Update jobs daily", click *Run workflow*. After a minute or two, `docs/jobs.json` fills up and the site shows jobs.

### Optional: Telegram messages

1. In Telegram, chat with **@BotFather**, send `/newbot`, and copy the bot token.
2. Make a Telegram channel (or group), add your bot as an admin, and note the channel name (for example `@hit_ict_jobs`).
3. Add two more repository secrets: `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` (the `@channelname`).
4. Share the channel link with students. They only need to join.

## Run it on your own computer

```bash
pip install -r requirements.txt
python scripts/update_jobs.py --dry-run    # test without AI, Telegram or saving
export ANTHROPIC_API_KEY=your-key
python scripts/update_jobs.py              # real run
```

Open `docs/index.html` through a local server (`python -m http.server` inside `docs/`) so the page can load `jobs.json`.

## Adding more job sites

Open `config.json` and copy a source block. You need:

- `listing_urls`: pages that list many jobs (a category or search page),
- `link_regex`: a pattern matching the **path** of an individual job link, for example `/jobs/[a-z0-9-]+-\d+/?$`. Copy one job link from the site and adapt it.

The Classifieds.co.zw entry is switched off until you fill in its `link_regex`. After adding a site, run with `--dry-run` and check that it reports some job links.

## Things to know

- **Check each site's terms** and keep the daily volume small. The agent reads only a few pages once a day and skips adverts it has already seen. Where a site offers an official feed or API, prefer that.
- **Sites change their layout.** If a day's log says `0 job links` for a source, its `link_regex` needs updating. The AI step reads whatever text is on the advert page, so layout changes there do not matter much.
- **The AI can make mistakes.** The website tells students to confirm details on the original advert. Review the first few days of results before sharing widely.
- **Cost control:** `max_new_per_run` in `config.json` limits how many adverts are analysed per day. Change the model with the `ANTHROPIC_MODEL` environment variable.
- **GitHub timing:** scheduled runs can start a few minutes late, and GitHub pauses schedules on repositories with no activity for 60 days. If that happens, re-enable the workflow in the Actions tab.
- Adverts without a closing date disappear after 45 days (`max_age_days_without_deadline`).
