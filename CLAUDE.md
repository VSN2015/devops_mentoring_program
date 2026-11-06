# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A 12-week DevOps mentoring program: 84 daily one-hour lessons (Wed 14 Oct 2026 → Tue 5 Jan 2027), practising on [Chatwoot](https://github.com/chatwoot/chatwoot). The plan stays within the Docker ecosystem (Compose → registry → Swarm → CI/CD with GitHub Actions → monitoring), with no Kubernetes.

- `learning-plan.md` is the source of truth for the curriculum. Each day has **Learn** (~20 min), **Do** (~35 min) and **Done when** (a visible check; the day isn't done until it passes).
- The learner tracks progress on a **Notion Kanban** board that mirrors the plan one item per day (see below). "Check the learning status" means query that board.
- Right now the folder holds only `learning-plan.md` and is **not a git repo yet**. On Day 7 the learner runs `git init`, and the folder becomes the `chatwoot-lab` repo (made public on GitHub on Day 57). The planned layout (`chatwoot/`, `stack/`, `status-page/`, `scripts/`, `notes/`, `runbook.md`, `diagrams/`, `meetings/`) is listed in `learning-plan.md` under "Prerequisites". Most of it doesn't exist yet; the learner creates it as the days go by.

## Notion learning board

- Board: "DevOps Learning Plan — Kanban". URL: https://app.notion.com/p/96eb0bc838c2422e943d94f5081f1678?v=3f8c4a0075a780b08a6c000cd292c63d
- Database ID `96eb0bc838c2422e943d94f5081f1678` → single data source `7b9b706c-66a0-4f83-bcd9-7eb089ac595b`
- Access it with the **`ntn` CLI** (`~/.local/bin/ntn`, already logged in to the "Soang Nguyen's Notion" workspace as the "Notion CLI" bot). The user asked to use `ntn` for this; don't use browser automation.

Schema (84 rows, one per day):

| Property | Type | Values |
|---|---|---|
| Name | title | `Day N · <topic>` |
| Day | number | 1–84 |
| Date | date | Day N = 2026-10-13 + N days |
| Month | select | Month 1 / Month 2 / Month 3 |
| Week | select | Week 1 … Week 12 |
| Type | select | Lesson (67) / Buffer (12) / Meeting prep (5) |
| Status | status | Not started / In progress / Done |
| Checked | checkbox | separate from Status; check both when reporting progress |
| Done when | rich_text | the day's completion criterion, copied from the plan |

Commands:

```bash
# All rows, sorted by day: Day, Date, Status, Checked, Name
ntn datasources query 7b9b706c-66a0-4f83-bcd9-7eb089ac595b --limit 100 --json \
  | jq -r '.results[] | .properties as $p | [$p.Day.number, $p.Date.date.start, $p.Status.status.name, $p.Checked.checkbox, ([$p.Name.title[].plain_text]|join(""))] | @tsv' \
  | sort -n

# Items due on or before a date that aren't Done (overdue check)
ntn datasources query 7b9b706c-66a0-4f83-bcd9-7eb089ac595b --limit 100 --json \
  --filter '{"and":[{"property":"Status","status":{"does_not_equal":"Done"}},{"property":"Date","date":{"on_or_before":"YYYY-MM-DD"}}]}'

# Schema / options
ntn api /v1/data_sources/7b9b706c-66a0-4f83-bcd9-7eb089ac595b

# One day's page body as Markdown
ntn pages get <page-id>

# Change a property (a write; ask the user first; not yet run in this repo)
ntn api /v1/pages/<page-id> -X PATCH -d '{"properties":{"Status":{"status":{"name":"Done"}}}}'
```

When reporting status, compare against today's date: which day the learner *should* be on, what's In progress, what's overdue (date passed and not Done), and the next buffer day or mentor meeting.

## Plan structure

| Month | Days | Theme | Ends with |
|---|---|---|---|
| 1 | 1–28 | Docker fundamentals → Chatwoot on Compose → registries (Docker Hub, GHCR) → Swarm intro | Meeting 1 prep (Day 27), meeting ~11 Nov |
| 2 | 29–56 | Own Linux server (Hetzner 4 GB suggested): Linux, networking, Caddy + HTTPS, multi-node Swarm, Trivy, backup/restore, incident drills | Meeting 2 prep (Day 55), meeting ~9 Dec |
| 3 | 57–84 | GitHub Actions CI → push to GHCR → SSH deploy with `docker stack deploy` → verify/rollback → Prometheus/Grafana/Dozzle/alerts → final demo | Meeting 3 prep (Day 83), meeting ~6 Jan |

- Every 7th day (7, 14, 21, …) is a **write-up & buffer** day for catching up. Days 27/28, 55/56 and 81–84 are meeting prep or dress rehearsals.
- Each month ends with a deliverable checklist (end of Days 28, 56, 84). The program-wide "Definition of Done" is at the bottom of `learning-plan.md`.
- The final demo: change `status-page` → push → pipeline → live at `https://<host>/status`. Then a broken build turns the pipeline red, Swarm rolls back, and the site stays up.

## Conventions to keep when editing the plan or helping with labs

- Chatwoot is pinned to **v4.18.0**. Use explicit image tags (`vX.Y.Z`, `sha-<7>`); never deploy `latest`.
- Lab environments only: local, Play with Docker, or the learner's own server. No production, no real customer data.
- Never put `.env`, the Rails master key or credentials in Dockerfiles, images or git. `.env` is created by hand on each machine (`chmod 600` on the server).
- `docker compose down -v` deletes the database volumes. Use `down` without `-v` unless the step says otherwise (only Day 53's local restore drill does).
- When the learner is stuck, point them to the investigation order the plan teaches: `ps` → `logs` → `exec`/connectivity → config. Record findings in `runbook.md` as symptom → cause → fix.
- If you change `learning-plan.md`, keep the Notion board in sync (Name, Date, Done when), and the other way round.
