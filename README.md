# gha-uptime-probe-issues

A scheduled workflow that checks a URL and keeps **one GitHub issue** in step with its health: opened when the check fails, commented on while it keeps failing, and closed when it recovers. Failure becomes visible and tracked without any monitoring service.

## What ran (2026-10-16)

Three manual runs of the same workflow (`workflow_dispatch` takes the URL as an input) showed the whole life cycle:

| Run | URL | Result |
|---|---|---|
| 1 | `https://example.invalid/` | check failed, **issue #1 opened** |
| 2 | same bad URL | **no second issue**; a "Still failing" comment was added |
| 3 | default (`https://api.github.com/zen`) | HTTP 200, a "Recovered" comment was added and **the issue was closed** |

The daily schedule (`0 6 * * *`) has **not fired yet**; only the manual runs were observed. Issue #1 is left closed as the record.

## Decisions

- **One open issue, not one per failure:** an alert that repeats every run gets ignored.
- **Close on recovery** so open issues mean "currently broken".
- **Everything untrusted goes through `env:`** (the URL is an input).
- **`curl --max-time` and `--retry`** so one slow response is not an outage.
- **Least privilege:** `contents: read` and `issues: write` only.

## Honest limits

- **Scheduled runs are best-effort:** they can start late, and GitHub disables the schedule in a public repository after 60 days without activity.
- A real probe needs a tighter interval, several locations, and paging or chat notifications, not just an issue.
- The issues are public here; do not put internal URLs or response bodies in an issue on a public repository.
- A 2xx or 3xx status counts as healthy; no body or latency check.

MIT licensed.
