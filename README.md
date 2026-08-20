# ops-watchdog

Public repo whose only job is to run scheduled health sweeps against Mark Studios
production services.

It lives here, and not inside each service's repo, for one reason: **GitHub Actions
is free and unmetered on public repositories.** Private-repo Actions minutes come out
of a single shared monthly pool for the whole account, and when that pool ran dry on
2026-08-19 the account-level `$0` Actions budget ("stop usage: yes") hard-blocked
*every* private repo's workflows at once. The Strategy Generator pipeline watchdog
went down 14.5 hours with no code change and no alert.

Nothing secret lives in this repo. Endpoint URLs are already public; bearer tokens are
GitHub Actions secrets, which are never exposed to forks or pull requests.

## Sweeps

| Workflow | Target | Cadence |
|---|---|---|
| `strategy-pipeline-watchdog.yml` | `strategy.markstudios.com/api/pipeline/cron` | `*/5` (GitHub throttles to ~30 min in practice) |

## Cadence caveat

GitHub's scheduler drops scheduled runs under load. A `*/5` cron on a free account
really fires about every 30-40 minutes. Treat this as the always-on **floor**, not the
SLA. The fast layer is the launchd job on Mark's Mac
(`com.markksantos.strategy-watchdog`, every 5 min, real cadence) plus the app's own
self-fire chain.

## Heartbeat

GitHub disables scheduled workflows in any repo with 60 days of no activity, silently.
The sweep pushes a dated line to `heartbeat.txt` once a day so that never happens.
