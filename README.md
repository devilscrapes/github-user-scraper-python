<p align="center"><a href="https://apify.com/DevilScrapes/github-user-scraper"><img src="assets/banner.svg" alt="GitHub User Scraper — Profiles, Repos &amp; Orgs — Python example" width="100%"></a></p>

# GitHub User Scraper in Python: profiles and organizations to pandas, JSON and CSV

<p align="center">
<a href="https://apify.com/DevilScrapes/github-user-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/github-user-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/github-user-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

This repo is a tested Python starter for the **[GitHub User Scraper](https://apify.com/DevilScrapes/github-user-scraper)** on Apify. You give it account names, for example `apify` and `microsoft`, and it hands back one row per account: display name, kind of account, follower and following totals, number of public repositories and gists, declared location, company and website, creation date and, when you switch it on, a short list of the repositories that changed most recently. The lookups run on Apify, so your notebook or cron job only waits for the run and reads `results.json` or `results.csv`. It is the kind of table you would otherwise assemble by clicking through profiles one by one, which is a special kind of purgatory we would rather automate.

> **Actor:** [GitHub User Scraper — Profiles, Repos & Orgs](https://apify.com/DevilScrapes/github-user-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [github-user-scraper-javascript](https://github.com/devilscrapes/github-user-scraper-javascript)

## 🔥 What you get

- **People and organizations in one schema**: both kinds of account come back with identical keys, and `type` says `User` or `Organization` so you can split them later.
- **Paste logins or links**: entries in `usernames` may be bare slugs or full profile URLs; the host part is removed before lookup.
- **Counts as integers**: `followers`, `following`, `public_repos` and `public_gists` are numbers, ready to sort, sum or chart.
- **Recent repositories on request**: set `includeRepos` and each row gains a `repos` list of the account's most recently updated public repositories, each with name, description, language, stars, forks, fork and archived flags and last push time. `maxReposPerUser` caps it at up to 100.
- **Bring a token if you need volume**: without one, GitHub allows 60 requests an hour; an optional personal access token lifts that to 5,000.
- **Back-off built in**: on throttling or server errors the Actor waits and retries, and it reports a clear status message instead of finishing with a silently empty dataset.
- **Only what is public**: `email` is filled only when the account owner chose to publish one, and private repositories or memberships are never touched.

## 🎯 Use cases

### Benchmark open-source presence across companies

List the GitHub organizations of a set of vendors, load the rows into a DataFrame and compare `followers` and `public_repos` side by side. Repeat it each quarter from a scheduled job and you have a simple trend line of developer reach.

### Find out what an organization actually works on

Turn on `includeRepos`, then count the `language` values and sum `stargazers_count` across the repositories you got back. A notebook cell tells you whether a company's recent public work is mostly TypeScript, Go or Python.

### Clean up a list of GitHub links in a spreadsheet

Export the profile URLs from a CRM or conference sheet, pass them straight in as `usernames` and merge the result back on `login`. Accounts that no longer resolve stand out because they are missing from the output.

### Track developer-relations metrics over time

Snapshot your own organization and a few peers daily with cron, write each CSV to a dated folder and plot follower growth with matplotlib. Because `scraped_at` is on every row, concatenating snapshots needs no extra bookkeeping.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/github-user-scraper-python.git
cd github-user-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [GitHub User Scraper — Profiles, Repos & Orgs](https://apify.com/DevilScrapes/github-user-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Underneath the CLI wrapper, the integration fits in a few lines. The official `apify-client` package starts the run, blocks until it is done and iterates over the dataset for you.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/github-user-scraper").call(run_input={'usernames': ['apify', 'microsoft'], 'includeRepos': True, 'maxReposPerUser': 5, 'proxyConfiguration': {'useApifyProxy': False}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

The script ships `input.json` to the Actor without changes. The example looks up two organizations and asks for the five most recently updated repositories of each, so the run returns two rows and finishes quickly. `usernames` is the only required field. Add a `githubToken` in a private input file, never in a committed one, once your list grows past a few dozen accounts.

```json
{
  "usernames": [
    "apify",
    "microsoft"
  ],
  "includeRepos": true,
  "maxReposPerUser": 5,
  "proxyConfiguration": {
    "useApifyProxy": false
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `usernames` | array | yes | `["torvalds", "apify"]` | — |
| `githubToken` | string | no | — | — |
| `includeRepos` | boolean | no | `false` | — |
| `maxReposPerUser` | integer | no | `30` | — |
| `concurrency` | integer | no | `6` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": false}` | — |

## 📤 Output

You get one record per account. `login`, `type` and `html_url` are guaranteed; profile fields that the owner left blank arrive as null. `repos` is null unless `includeRepos` was true, and in that case it is a list of small dictionaries rather than a flat column, which is why the recipe below unpacks it.

| Field | Type | Always present |
|---|---|---|
| `login` | string | yes |
| `type` | string | yes |
| `name` | string or null | no |
| `company` | string or null | no |
| `blog` | string or null | no |
| `location` | string or null | no |
| `email` | string or null | no |
| `bio` | string or null | no |
| `twitter_username` | string or null | no |
| `public_repos` | integer | no |
| `public_gists` | integer | no |
| `followers` | integer | no |
| `following` | integer | no |
| `html_url` | string | yes |
| `avatar_url` | string | no |
| `hireable` | boolean or null | no |
| `created_at` | string | no |
| `updated_at` | string | no |
| `repos` | array or null | no |
| `scraped_at` | string | no |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.003 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/github-user-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

This pandas snippet (`pip install pandas`) builds an account summary: it computes how long each account has existed, unpacks the nested `repos` lists, totals stars per account and picks the most common language, then saves `accounts.csv`.

```python
import json

import pandas as pd

accounts = json.load(open("results.json", encoding="utf-8"))
profiles = pd.DataFrame(accounts)[["login", "type", "followers", "public_repos", "created_at"]]
created = pd.to_datetime(profiles["created_at"], utc=True)
profiles["years_on_github"] = ((pd.Timestamp.now(tz="UTC") - created).dt.days / 365.25).round(1)

repo_rows = [{**repo, "owner": acc["login"]} for acc in accounts for repo in (acc.get("repos") or [])]
repos = pd.DataFrame(repo_rows)
if not repos.empty:
    per_owner = repos.groupby("owner").agg(
        sampled_repos=("name", "count"),
        stars=("stargazers_count", "sum"),
        top_language=("language", lambda s: s.mode().iat[0] if not s.mode().empty else None),
    )
    profiles = profiles.join(per_owner, on="login")

profiles.to_csv("accounts.csv", index=False)
print(profiles.to_string(index=False))
```

## ❓ FAQ

### Do I need a GitHub token?

Not for small lists. Without a token GitHub allows 60 requests per hour from one address, and every account costs one request, plus one more when `includeRepos` is on. For hundreds of accounts, put a personal access token in `githubToken` and the allowance rises to 5,000 an hour.

### Why is `email` almost always null?

GitHub hides addresses unless the account owner chose to make one public. The Actor returns the public field as it is and does not try to recover addresses from commits or anywhere else.

### Can I get contribution graphs or commit counts?

No. The data comes from the public profile, not from activity history, so contribution calendars, sponsorship tiers, starred repositories and commit statistics are not part of a row. For repository-level detail, look at the related repo scraper below.

### Is it fine to use this data for recruiting outreach?

The Actor only returns fields people chose to publish, such as the self-declared `hireable` flag. How you use them, and whether that use meets GDPR and GitHub's acceptable use rules, is your responsibility. Keep personal data to what you actually need.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/github-user-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [GitHub User Scraper — Profiles, Repos & Orgs on Apify Store](https://apify.com/DevilScrapes/github-user-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/github-user-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [GitHub Organization Scraper](https://apify.com/DevilScrapes/github-org-scraper) — organization metadata plus its public repository list, including member counts.
- [GitHub Repo Scraper](https://apify.com/DevilScrapes/github-repo-scraper) — deep metadata for individual repositories, such as topics, license and latest release.
- [GitHub Trending Scraper](https://apify.com/DevilScrapes/github-trending-scraper) — what is climbing GitHub's trending page today.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/github-user-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with GitHub or Microsoft.
