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

| Workflow | Target | Cadence | Moved from |
|---|---|---|---|
| `strategy-pipeline-watchdog.yml` | `strategy.markstudios.com/api/pipeline/cron` | `*/5` | `strategy-generator` |
| `quoter-uptime.yml` | `start.markstudios.com` health + landing | `*/15` | `markstudios-quoter` |
| `fablewatch-heartbeat.yml` | `fablewatch.com` health + backup tick | `*/30` | `fablewatch` |

The rule for what belongs here: **an external HTTP call that needs no private code
and no repo write.** Jobs that check out a private repo, build it, or commit back to
it (e.g. fablewatch's `AW autopilot`) must stay where they are — moving them would
mean putting a token with private-repo write access into a public repo, which trades
a billing problem for a security one. Don't.

## Cadence caveat

GitHub's scheduler drops scheduled runs under load. A `*/5` cron on a free account
really fires about every 30-40 minutes. Treat this as the always-on **floor**, not the
SLA. The fast layer is the launchd job on Mark's Mac
(`com.markksantos.strategy-watchdog`, every 5 min, real cadence) plus the app's own
self-fire chain.

## Heartbeat

GitHub disables scheduled workflows in any repo with 60 days of no activity, silently.
The sweep pushes a dated line to `heartbeat.txt` once a day so that never happens.
