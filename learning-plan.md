# DevOps Mentoring — Day-by-Day Learning Plan

**Cadence:** 7 days/week, ~1 hour/day · **Start:** Wed 14 Oct 2026 · **End:** Tue 5 Jan 2027 (84 days, 12 weeks)
**Practice app:** [Chatwoot](https://github.com/chatwoot/chatwoot) pinned to **v4.18.0** · **Scope:** Docker ecosystem only (no Kubernetes)

| Month | Days | Dates | Theme | Mentor meeting |
|---|---|---|---|---|
| 1 | 1–28 | 14 Oct – 10 Nov | Docker fundamentals → Chatwoot on Compose → Registry → Swarm intro | ~11 Nov |
| 2 | 29–56 | 11 Nov – 8 Dec | Your server: Linux, networking, HTTPS, Swarm deeper, backup/restore, incidents | ~9 Dec |
| 3 | 57–84 | 9 Dec – 5 Jan | CI/CD to the server, verify, monitor, rollback → final demo | ~6 Jan |

## How to use this plan

- Each day fits in one hour: **Learn** (~20 min reading) → **Do** (~35 min hands-on) → **Done when** (a check you can see). Set a timer.
- **Day 7 of every week is a write-up & buffer day.** If a lab runs over, stop and finish it on the buffer day. Don't let one day push the rest.
- Stuck more than 20 minutes? Paste the exact error into `notes/`, move on, and bring it to the buffer day (or ask me / your mentor).
- Keep everything in git from Day 7: notes, diagrams, runbook, Compose/stack files. **Never commit `.env`.**
- "Done when" is the point of the day. If you can't tick it, the day isn't done — but the buffer day exists for exactly that.

## Lab safety rules (from the brief)

- Lab/local/your own server only. No production, no real customer data.
- No credentials, Rails master key, or secrets in Dockerfiles, images, or git.
- `docker compose down -v` deletes named volumes = deletes your database. Use `down` (no `-v`) unless you mean it.
- Use specific image tags (`v4.18.0`, `sha-abc1234`), never deploy `latest`.

## Costs — and how to keep them near zero

Everything in this plan is free except the server (Months 2–3) and, optionally, a domain. Prices checked 13 Oct 2026 — confirm before buying.

| Item | Cost | Notes |
|---|---|---|
| Docker, Compose, Swarm, Trivy, Caddy, Prometheus, Grafana, Dozzle | $0 | Open source |
| Docker Hub (Personal) | $0 | 100 pulls/hour when logged in — plenty |
| GitHub Actions, GHCR, Environments | $0 | **Keep the lab repo public**: unlimited Actions minutes, free package storage, environment protection rules. (Private: 2,000 min/month, 500 MB packages, no protection rules without Pro.) Nothing secret goes in the repo anyway. |
| Play with Docker | $0 | Multi-node Swarm days (Week 7) |
| Let's Encrypt, DuckDNS, UptimeRobot, healthchecks.io | $0 | Free tiers cover this lab |
| **Server**, Day 29 → Day 84 (~8 weeks) | **≈ €10–13 total** | Options below. Hetzner bills by the hour — you pay only for the weeks you use. |
| **Domain** (optional) | $0–2 | Options below |

**Server options.** Chatwoot's four containers need ~2 GB; with the monitoring stack in Week 11 you want 4 GB. Don't buy 2 GB (constant OOM pain), don't buy 8 GB (wasted).

- **A — Hetzner Cloud, 4 GB, ≈ €4.5–6.5/month incl. IPv4 (recommended).** `CAX11` (2 ARM vCPU, 4 GB, 40 GB) or `CX23` (2 x86 vCPU, 4 GB, 40 GB), location Falkenstein or Helsinki (Singapore costs more). **Add an IPv4 address** (~€0.50) — GitHub Actions runners can't reach IPv6-only servers. ARM is fine: Chatwoot publishes an arm64 image. Card or PayPal; new accounts are sometimes asked for ID verification. ~200 ms latency from Vietnam — irrelevant for a lab. **Delete the server after Meeting 3** (or snapshot it, ~€0.50/month).
- **B — Oracle Cloud Always Free, $0.** ARM `VM.Standard.A1.Flex`, up to 2 OCPU / 12 GB, free for as long as it lasts. Catches: signups get rejected without reason, "Out of host capacity" can block VM creation for days, and the free allowance was cut in June 2026 without notice. Worth 30 minutes on Day 29; if it doesn't work, take A.
- **C — Vietnamese provider (Vietnix, AZDIGI…), ≈ 340–460k VND/month for 4 GB.** 2–3× Hetzner. Only worth it if you need to pay by local bank transfer/MoMo.
- **D — No server, $0.** Multipass VM on your Mac + Tailscale Funnel for a public HTTPS URL. Works for the demo, but your Mac must stay on and CI needs Tailscale on the runner to reach it. Last resort.

**Domain options.** You need a hostname for HTTPS from Day 39.

- **DuckDNS, $0 (recommended):** `yourname.duckdns.org` → your server IP. It's on the Public Suffix List, so Let's Encrypt limits are per-user, and Caddy needs no plugin. Good enough for the mentor demo.
- **Cheap paid, ≈ $1–2 first year:** `.xyz` at Porkbun or Namecheap (renewal $14–19 — let it expire, or transfer to Cloudflare at ~$11/yr). Or `.id.vn` ≈ 60,000 VND/year — **free for 2 years if you're a Vietnamese citizen aged 18–23** (VNNIC program; check it's still open).
- **Avoid `sslip.io` / `nip.io` for HTTPS:** their Let's Encrypt quota is shared by everyone, so certificate requests often fail.

## Prerequisites (do before Day 1, ~30 min)

- [ ] Docker on your Mac: Docker Desktop (or OrbStack / Colima if Desktop won't install on your macOS). Verify: `docker --version` (24+), `docker compose version` (v2.20+).
- [ ] Give Docker at least **6 GB RAM** in its settings — Chatwoot's four containers need ~3 GB.
- [ ] Accounts: GitHub (you have one), Docker Hub (free). You'll need a Linux server from Day 29 — see **Costs** above; the whole program costs about €10–13 total.
- [ ] Project layout — this folder becomes the `chatwoot-lab` git repo:
  ```
  devops_mentoring/
  ├── learning-plan.md      ← this file
  ├── README.md             ← how to run the lab from zero (Day 14)
  ├── runbook.md            ← symptom → cause → fix entries (Day 11 onward)
  ├── notes/                ← weekN.md, your own words
  ├── diagrams/             ← Mermaid files (render on GitHub)
  ├── meetings/             ← meeting-N.md prep
  ├── chatwoot/             ← docker-compose.yaml, .env.example (never .env)
  ├── stack/                ← Swarm stack files (Week 4+)
  ├── status-page/          ← tiny Sinatra app you build & ship via CI (Week 3+)
  └── scripts/              ← healthcheck.sh, backup.sh
  ```

---

# Month 1 — Docker fundamentals, Chatwoot on Compose, Registry, Swarm intro

Your mentor's Week 1 assumes you can already read a Compose file. So Week 1 here is Docker fundamentals on tiny images, and Chatwoot comes up on Day 6. Deliverables are unchanged.

## Week 1 — Docker fundamentals (Days 1–7)

- [ ] **Day 1 · Wed 14 Oct — Your first container**
  - Learn: [What is a container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)
  - Do: `docker run hello-world` → `docker run -d -p 8080:80 --name web nginx:1.27` → open http://localhost:8080 → `docker ps` → `docker stop web` → `docker start web` → `docker rm -f web`.
  - Done when: you can say what `-d`, `-p 8080:80`, and `--name` each do, and `docker ps -a` is empty.

- [ ] **Day 2 · Thu 15 Oct — Image vs container**
  - Learn: [What is an image?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/) and [image layers](https://docs.docker.com/get-started/docker-concepts/building-images/understanding-image-layers/).
  - Do: `docker pull nginx:1.27` → `docker images` → `docker history nginx:1.27` (those rows are layers). Run two containers from the one image: `web1` on 8081, `web2` on 8082. `docker exec -it web1 sh` → `echo hi > /usr/share/nginx/html/index.html` → check 8081 vs 8082. `docker rm -f web1 web2` → re-run web1 → change is gone.
  - Done when: one sentence in your own words: why two containers from one image don't share changes, and why changes vanish on `rm`.

- [ ] **Day 3 · Fri 16 Oct — Env vars, logs, exec (PostgreSQL)**
  - Learn: the "Environment Variables" section of the [postgres image page](https://hub.docker.com/_/postgres).
  - Do: `docker run -d --name pg postgres:16` → `docker logs pg` → read the error that tells you a password is required. Then `docker rm pg` → `docker run -d --name pg -e POSTGRES_PASSWORD=lab postgres:16` → `docker logs -f pg` until "ready to accept connections" → `docker exec -it pg psql -U postgres -c "create table t(id int); insert into t values (1);"` → `docker inspect pg --format '{{json .Config.Env}}'`.
  - Done when: you found the error in logs yourself and the row is inserted.

- [ ] **Day 4 · Sat 17 Oct — Volumes: where data lives**
  - Learn: [Volumes](https://docs.docker.com/engine/storage/volumes/) (first half) and [bind mounts](https://docs.docker.com/get-started/docker-concepts/running-containers/sharing-local-files/).
  - Do: `docker rm -f pg` → re-run Day 3's command → `select * from t;` → table gone. Now: `docker volume create pgdata` → `docker run -d --name pg -e POSTGRES_PASSWORD=lab -v pgdata:/var/lib/postgresql/data postgres:16` → insert a row → `docker rm -f pg` → run the same command again → row still there. `docker volume ls`, `docker volume inspect pgdata`. Bind mount: `mkdir html; echo hello > html/index.html; docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html:ro nginx:1.27` → edit the file → refresh.
  - Done when: data survived container deletion; one line each: named volume vs bind mount.

- [ ] **Day 5 · Sun 18 Oct — Networks: containers talk by name**
  - Learn: [Networking overview](https://docs.docker.com/engine/network/) — bridge networks and DNS by container name.
  - Do: `docker network create lab` → `docker run -d --name redis --network lab redis:7-alpine` → `docker run -it --rm --network lab redis:7-alpine redis-cli -h redis ping` → `PONG`. Now `-h localhost` instead → connection refused. `docker network inspect lab`. This is why Chatwoot's `.env` has `REDIS_URL=redis://redis:6379`, not `localhost`.
  - Done when: PONG via service name, and you can explain why `localhost` inside a container is the container itself.

- [ ] **Day 6 · Mon 19 Oct — Chatwoot up with Docker Compose**
  - Learn: [Chatwoot Docker production guide](https://developers.chatwoot.com/self-hosted/deployment/docker) (read it all, ~15 min). Start the image pull first — it's ~1 GB.
  - Do, in `chatwoot/`:
    ```bash
    curl -LO https://raw.githubusercontent.com/chatwoot/chatwoot/v4.18.0/.env.example
    curl -L -o docker-compose.yaml https://raw.githubusercontent.com/chatwoot/chatwoot/v4.18.0/docker-compose.production.yaml
    cp .env.example .env
    ```
    Edit `docker-compose.yaml`: replace `chatwoot/chatwoot:latest` with `chatwoot/chatwoot:v4.18.0` (both places); set `POSTGRES_PASSWORD` under the `postgres` service. Edit `.env`: same `POSTGRES_PASSWORD`, a `REDIS_PASSWORD`, `SECRET_KEY_BASE=$(openssl rand -hex 64)`, `FRONTEND_URL=http://localhost:3000`. Then:
    ```bash
    docker compose pull
    docker compose run --rm rails bundle exec rails db:chatwoot_prepare
    docker compose up -d
    curl -I localhost:3000/api   # expect HTTP 200
    ```
    Open http://localhost:3000 and create the admin account.
  - Done when: you're logged in to Chatwoot. (If not — buffer day tomorrow.)

- [ ] **Day 7 · Tue 20 Oct — Write-up & buffer**
  - Do: `notes/week1.md` — a table: image, container, volume, bind mount, network, port, env var — one line each in your own words. On paper, sketch the 4 Chatwoot services and arrows between them. `git init`, `.gitignore` with `.env`, `*.dump`, `backups/`; first commit. Finish anything unfinished.
  - Done when: committed; sketch exists; Chatwoot still runs after `docker compose restart`.

## Week 2 — Compose & troubleshooting on Chatwoot (Days 8–14)

- [ ] **Day 8 · Wed 21 Oct — Read the Compose file line by line**
  - Learn: [Compose application model](https://docs.docker.com/compose/intro/compose-application-model/) · [`depends_on`](https://docs.docker.com/reference/compose-file/services/#depends_on), [`restart`](https://docs.docker.com/reference/compose-file/services/#restart), [`env_file`](https://docs.docker.com/reference/compose-file/services/#env_file).
  - Do: add a `#` comment to every key in `docker-compose.yaml` explaining it. `docker compose config` shows the resolved file with `.env` substituted and the `base` anchor expanded. Notice: `rails` and `sidekiq` use the same image but different `command`; both mount `storage_data`; ports bind `127.0.0.1` only (why?); `restart: always`.
  - Done when: you can explain all four services and what `<<: *base` does.

- [ ] **Day 9 · Thu 22 Oct — The daily-driver commands**
  - Learn: [`docker compose` CLI reference](https://docs.docker.com/reference/cli/docker/compose/) — skim `ps`, `logs`, `exec`, `stats`, `restart`.
  - Do: `docker compose ps` · `docker compose logs -f --tail=50 rails` · `docker compose logs sidekiq` · `docker compose exec rails bundle exec rails console` → `Account.count` · `docker compose exec postgres psql -U postgres chatwoot -c '\dt'` · `docker stats --no-stream` · `docker compose restart sidekiq` · `docker compose top`.
  - Done when: `notes/cheatsheet.md` with 8 commands and when you reach for each.

- [ ] **Day 10 · Fri 23 Oct — How Rails, Sidekiq, Postgres, Redis fit together**
  - Learn: [Chatwoot architecture](https://developers.chatwoot.com/self-hosted/deployment/architecture) · Sidekiq's README "how it works" paragraph.
  - Do: in Chatwoot, invite an agent or send a message → watch `docker compose logs -f sidekiq` pick up the job. `docker compose exec redis redis-cli -a "$REDIS_PASSWORD" info keyspace`. Write `diagrams/chatwoot-services.md` in Mermaid: browser → rails:3000; rails → postgres:5432, redis:6379; sidekiq → postgres, redis; rails & sidekiq → `storage_data`.
  - Done when: diagram committed; you can name what Redis does for Chatwoot (Sidekiq queues, cache, ActionCable).

- [ ] **Day 11 · Sat 24 Oct — Break it #1: Redis**
  - Learn: the investigation order you'll use forever: `ps` (is it running?) → `logs` (what does it say?) → `exec`/connect (can it reach its dependency?) → config (is the value right?).
  - Do: (a) `docker compose stop redis` → use the UI, read `logs rails` → `docker compose start redis`. (b) In `.env` set `REDIS_URL=redis://localhost:6379` → `docker compose up -d` → `logs rails` → find the error → fix → `up -d`. Start `runbook.md`: entry #1 with symptom, the exact log line, cause, fix.
  - Done when: runbook entry #1 exists with the real error text.

- [ ] **Day 12 · Sun 25 Oct — Break it #2: port conflict and wrong DB password**
  - Do: (a) `docker run -d -p 127.0.0.1:3000:80 --name hog nginx:1.27` → `docker compose up -d rails` → "port is already allocated" → `lsof -i :3000` → `docker rm -f hog`. (b) Change `POSTGRES_PASSWORD` in `.env` only → `docker compose up -d` → `logs rails` shows `PG::ConnectionBad` → fix. Runbook entries #2, #3.
  - Done when: both fixed; runbook has 3 entries.

- [ ] **Day 13 · Mon 26 Oct — Persistence: prove data survives**
  - Learn: [`docker compose down`](https://docs.docker.com/reference/cli/docker/compose/down/) — read what `-v` does.
  - Do: create an inbox and a contact → `docker compose down` → `docker volume ls` (volumes still there) → `docker compose up -d` → data still there. Upload an avatar → `docker compose exec rails ls -R /app/storage | head`. First backup taste: `docker compose exec -T postgres pg_dump -U postgres chatwoot > backups/first.sql`.
  - Done when: `notes/week2.md` lists the 3 volumes and what lives in each, plus what `down -v` would destroy.

- [ ] **Day 14 · Tue 27 Oct — Write-up & buffer**
  - Do: write `README.md`: from zero to running Chatwoot (clone → `cp .env.example .env` → set 4 values → `pull` → `db:chatwoot_prepare` → `up -d`). Deliverable table in `notes/week2.md`: every port, volume, network, and env var you touched, with its role. Commit.
  - Done when: README + table committed.

## Week 3 — Registry & image lifecycle (Days 15–21)

- [ ] **Day 15 · Wed 28 Oct — Registry, repository, tag, digest**
  - Learn: [What is a registry?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-registry/)
  - Do: browse [chatwoot/chatwoot tags](https://hub.docker.com/r/chatwoot/chatwoot/tags) — see `latest`, `v4.18.0`, `develop`. `docker image inspect chatwoot/chatwoot:v4.18.0 --format '{{.Id}} {{.RepoDigests}}'`. `docker pull chatwoot/chatwoot:latest` and compare digests: same image or not?
  - Done when: registry / repository / tag / digest explained using the Chatwoot image as the example.

- [ ] **Day 16 · Thu 29 Oct — Push your first image (Docker Hub)**
  - Learn: [Build and push your first image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/) · [access tokens](https://docs.docker.com/security/access-tokens/) (use a token, never your password).
  - Do: create a Docker Hub access token → `docker login` with it. Small first: `docker tag nginx:1.27 YOURUSER/nginx-lab:1.27` → `docker push` → `docker rmi` → `docker pull`. Then the real one (start it and let it run): `docker tag chatwoot/chatwoot:v4.18.0 YOURUSER/chatwoot-lab:v4.18.0` → `docker push YOURUSER/chatwoot-lab:v4.18.0`.
  - Done when: both repos visible on Docker Hub; `nginx-lab` pulled back after deleting it locally.

- [ ] **Day 17 · Fri 30 Oct — Your first Dockerfile: the status page**
  - Learn: [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/) · [build cache](https://docs.docker.com/get-started/docker-concepts/building-images/using-the-build-cache/).
  - Do: `status-page/` — a tiny Sinatra app (`Gemfile`: sinatra, puma, rackup; `app.rb` with `GET /` → "status-page ok" and `GET /health` → JSON `{status: "ok"}`). Dockerfile:
    ```dockerfile
    FROM ruby:3.3-alpine
    WORKDIR /app
    RUN apk add --no-cache build-base
    COPY Gemfile Gemfile.lock ./
    RUN bundle install
    COPY . .
    EXPOSE 4567
    CMD ["bundle", "exec", "ruby", "app.rb", "-o", "0.0.0.0"]
    ```
    `docker build -t status-page:dev .` → run on 4567 → `curl localhost:4567/health`. Edit `app.rb` → rebuild → notice `bundle install` is cached. Edit `Gemfile` → rebuild → notice it isn't. That's why Gemfile is copied first.
  - Done when: app answers; you can explain the COPY order.

- [ ] **Day 18 · Sat 31 Oct — Tags by version and commit SHA; GHCR**
  - Learn: [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry).
  - Do: GitHub PAT (classic) with `write:packages` → `echo $PAT | docker login ghcr.io -u YOURUSER --password-stdin`. `docker build -t ghcr.io/YOURUSER/status-page:sha-$(git rev-parse --short HEAD) -t ghcr.io/YOURUSER/status-page:0.1.0 status-page/` → push both → pull by the sha tag. Write `notes/tagging.md`: `vX.Y.Z` for releases, `sha-<7>` for every build, `latest` never deployed.
  - Done when: two tags on GHCR; tagging rule written down.

- [ ] **Day 19 · Sun 1 Nov — Secrets hygiene and `.dockerignore`**
  - Learn: [Build best practices](https://docs.docker.com/build/building/best-practices/) — the parts on `.dockerignore` and secrets.
  - Do: proof that images leak: temporarily `COPY .env /app/.env` in a test Dockerfile → build → `docker run --rm test cat /app/.env` → your secrets are in the image, and `docker history` shows the layer. Delete that. Add `status-page/.dockerignore` (`.env`, `.git`, `log/`, `tmp/`). `git log -p | grep -i -E 'password|secret_key'` must return nothing. `docker scout quickview status-page:dev` (ships with Docker Desktop) — first look at vulnerability scanning.
  - Done when: `.dockerignore` committed; `notes/week3.md` has "things I will never put in an image".

- [ ] **Day 20 · Mon 2 Nov — Inside the Chatwoot image**
  - Learn: skim [Chatwoot's Dockerfile](https://github.com/chatwoot/chatwoot/blob/v4.18.0/docker/Dockerfile) — find the three `FROM` lines (node, pre-builder, final). It's multi-stage: build with Node + Ruby, ship only Ruby.
  - Do: `docker history chatwoot/chatwoot:v4.18.0 | head -40`. `docker compose exec rails sh` → `cat docker/entrypoints/rails.sh`, `id` (it runs as root — note it), `ls /app/storage`, `command -v wget curl` (busybox `wget` exists; `curl` doesn't — matters for healthchecks later). `docker image inspect chatwoot/chatwoot:v4.18.0 --format '{{.Config.Entrypoint}} {{.Config.Cmd}} {{.Config.ExposedPorts}}'`; compare with what the Compose file sets.
  - Done when: three bullets: what the entrypoint does before Rails boots.

- [ ] **Day 21 · Tue 3 Nov — Write-up & buffer**
  - Do: registry evidence for the deliverable — links/screenshots of your Docker Hub and GHCR repos plus the commands, in `notes/week3.md`. `docker logout` if the machine is shared. Commit.
  - Done when: evidence committed; week finished.

## Week 4 — Swarm intro + Meeting 1 prep (Days 22–28)

- [ ] **Day 22 · Wed 4 Nov — Swarm concepts, first service**
  - Learn: [Swarm key concepts](https://docs.docker.com/engine/swarm/key-concepts/) · [tutorial: create a service](https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/).
  - Do: `docker swarm init` → `docker node ls` → `docker service create --name web --replicas 2 -p 8080:80 nginx:1.27` → `docker service ls`, `docker service ps web` → `curl localhost:8080` a few times → `docker service logs web`.
  - Done when: manager, service, task, replica defined in your own words.

- [ ] **Day 23 · Thu 5 Nov — Scale, rolling update, rollback**
  - Learn: [rolling updates](https://docs.docker.com/engine/swarm/swarm-tutorial/rolling-update/) · [`docker service update`](https://docs.docker.com/reference/cli/docker/service/update/) (`--update-delay`, `--update-failure-action`, `--rollback`).
  - Do: `docker service scale web=4` → `docker service update --image nginx:1.28 --update-delay 10s web` while watching `watch docker service ps web` in another terminal → then `docker service update --image nginx:does-not-exist web` → watch it fail → `docker service rollback web` → `docker service ps web --no-trunc`.
  - Done when: you watched an update and a rollback happen task by task; note what `--update-failure-action rollback` would do.

- [ ] **Day 24 · Fri 6 Nov — Stack deploy: Compose file → Swarm**
  - Learn: [Deploy a stack](https://docs.docker.com/engine/swarm/stack-deploy/) · [the `deploy` key](https://docs.docker.com/reference/compose-file/deploy/).
  - Do: `stack/nginx-stack.yml` (nginx with `deploy: replicas: 2`) → `docker stack deploy -c stack/nginx-stack.yml demo` → `docker stack services demo`, `docker stack ps demo` → `docker stack rm demo`. Then, without deploying: `cd chatwoot && docker stack config -c docker-compose.yaml | head -60` and read the warnings — `restart` ignored, `depends_on` ignored, `127.0.0.1:` port binding ignored (Swarm's ingress listens on all nodes).
  - Done when: `notes/compose-vs-swarm.md` lists the keys Swarm ignores or changes.

- [ ] **Day 25 · Sat 7 Nov — Chatwoot on single-node Swarm (first attempt)**
  - Do: `docker compose down` (keep volumes!). Write `stack/chatwoot-stack.yml` from the Compose file: drop `restart`, add `deploy: {replicas: 1}` to rails/sidekiq, ports `3000:3000`, keep `env_file: ../chatwoot/.env`. Volumes: new ones for now. Deploy: `set -a; source chatwoot/.env; set +a; docker stack deploy -c stack/chatwoot-stack.yml chatwoot` (the `source` makes `${VAR}` substitution work). `docker stack ps chatwoot --no-trunc`. Rails boots but the DB is empty → migrations in Swarm are *your* job: `docker exec $(docker ps -q -f name=chatwoot_rails) bundle exec rails db:chatwoot_prepare` → log in at localhost:3000.
  - Done when: Chatwoot running as a stack; you hit the migration problem and solved it.

- [ ] **Day 26 · Sun 8 Nov — Secrets, and the Compose-vs-Swarm table**
  - Learn: [Docker secrets](https://docs.docker.com/engine/swarm/secrets/).
  - Do: `echo lab | docker secret create pg_password -` → in the stack, postgres gets `POSTGRES_PASSWORD_FILE: /run/secrets/pg_password` + `secrets: [pg_password]` (postgres image supports `_FILE`; Chatwoot's Rails doesn't — note that for Month 2). `docker service update --force chatwoot_rails` → watch the rolling restart. Finish `notes/compose-vs-swarm.md` as a table: scheduling, networking, restart, secrets, scaling, migrations.
  - Done when: postgres reads its password from a secret; table done.

- [ ] **Day 27 · Mon 9 Nov — Meeting 1 prep**
  - Do: `meetings/meeting-1.md`: goal status; a 12-minute demo order — (1) `docker compose ps` + login, (2) services diagram, (3) break `REDIS_URL` live and fix it, (4) registry tags on Hub/GHCR, (5) Swarm scale/update/rollback on nginx. Answer the 7 questions from the brief in writing, 2–4 lines each. 3 questions for your mentor. Your proposal for Month 2 (server + Linux/networking + Swarm deeper). Dry run with a timer.
  - Done when: `meeting-1.md` committed; dry run ≤ 15 min.

- [ ] **Day 28 · Tue 10 Nov — Buffer, second dry run, send agenda**
  - Do: fix whatever broke in the dry run. Decide which to demo (Compose or stack) and have it one command away. Send your mentor the agenda + README link.
  - **Month 1 deliverable check:** ☐ Chatwoot on Compose ☐ services diagram ☐ ports/volumes/networks/env table ☐ registry push/pull evidence ☐ Swarm service/scale/update/rollback ☐ README + runbook (≥3 entries).

---

# Month 2 — Your server: Linux, networking, HTTPS, Swarm deeper, backup/restore, incidents

You get a Linux server on Day 29 and keep it through Month 3. Everything you learn here happens on the box you'll deploy to. Multi-node Swarm days use [Play with Docker](https://labs.play-with-docker.com) (free, in the browser, nothing to install).

## Week 5 — Linux fundamentals on your server (Days 29–35)

- [ ] **Day 29 · Wed 11 Nov — Get your server and SSH in properly**
  - Learn: [DigitalOcean: Initial server setup with Ubuntu](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu) (the steps apply to any provider).
  - Do: pick a server from the **Costs** section — default Hetzner `CAX11` or `CX23`, Ubuntu 24.04, Falkenstein or Helsinki, **with an IPv4 address**. (Trying Oracle Free Tier? 30 minutes max, then fall back.) Nothing bought yet? `multipass launch --name lab --cpus 2 --memory 4G 24.04` on your Mac keeps this week moving. `ssh-keygen -t ed25519` → add the public key → `ssh root@IP` → create user `sang` with sudo → copy the key to that user → `~/.ssh/config` entry `Host lab` → disable password login in `/etc/ssh/sshd_config` → `ufw allow OpenSSH && ufw enable`.
  - Done when: `ssh lab` logs in with your key; password login refused.

- [ ] **Day 30 · Thu 12 Nov — Filesystem and permissions**
  - Learn: [Ubuntu: the command line for beginners](https://ubuntu.com/tutorials/command-line-for-beginners) (sections 1–5) · `chmod`/`chown` octal notation.
  - Do: on the server: `pwd ls -la cd cat less find grep`; look around `/etc`, `/var/log`, `/home`. Create a file, `chmod 600` it, try to read it as another user (`sudo -u nobody cat file`). Read `-rw-r--r--` out loud. Note: a container process has a uid too — Chatwoot's runs as root (Day 20), postgres as uid 999 — which is why bind mounts sometimes get "permission denied".
  - Done when: you can explain `-rwxr-x---` and `chmod 640` without looking.

- [ ] **Day 31 · Fri 13 Nov — Processes, CPU, RAM; Docker on the server**
  - Learn: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/) (apt repo method) · `ps`, `top`, `free`, signals.
  - Do: `ps aux | head`, `top` (press `M` for memory sort), `free -h`, `uptime` (load average = runnable processes). Add 2 GB swap so a 4 GB box never OOM-kills Chatwoot: `sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile`, then add `/swapfile none swap sw 0 0` to `/etc/fstab` (`free -h` shows it). Install Docker, `sudo usermod -aG docker $USER`, re-login. `docker run -d --name pg -e POSTGRES_PASSWORD=lab postgres:16` → `ps aux | grep postgres` → a container is just a process on the host. `docker stop pg` sends SIGTERM, waits 10 s, then SIGKILL — `docker stop -t 30` changes the grace period.
  - Done when: Docker runs on the server without sudo; you can point at a container's process in `ps`.

- [ ] **Day 32 · Sat 14 Nov — Disk and logs**
  - Learn: [Configure logging drivers](https://docs.docker.com/engine/logging/configure/) — the `json-file` `max-size`/`max-file` options.
  - Do: `df -h`, `sudo du -sh /var/lib/docker`, `docker system df`. `journalctl -u docker --since today`, `sudo ls /var/lib/docker/containers/*/` (the `*-json.log` is where `docker logs` reads from — it grows forever by default). Create `/etc/docker/daemon.json` with `{"log-driver":"json-file","log-opts":{"max-size":"10m","max-file":"3"}}` → `sudo systemctl restart docker`. Read what `docker system prune` removes before ever running it.
  - Done when: log rotation set; `notes/week5.md` has a "disk is full" checklist (df → docker system df → logs → prune).

- [ ] **Day 33 · Sun 15 Nov — Chatwoot on the server (Compose)**
  - Do: `git clone` your repo onto the server → create `chatwoot/.env` there by hand (`chmod 600`; it never travels through git) → `FRONTEND_URL=http://localhost:3000` for now → `docker compose pull` → `db:chatwoot_prepare` → `up -d`. Ports still bind `127.0.0.1`, so reach it through an SSH tunnel: on your Mac `ssh -L 3000:localhost:3000 lab` → open http://localhost:3000.
  - Done when: Chatwoot on the server, reachable through the tunnel; admin account created.

- [ ] **Day 34 · Mon 16 Nov — Reboot test, systemd, health script**
  - Do: `sudo reboot` → wait → `ssh lab` → `docker compose ps` — did it come back? (`restart: always` + `systemctl is-enabled docker`.) Write `scripts/healthcheck.sh`: `uptime`, `free -h`, `df -h /`, `docker ps --format 'table {{.Names}}\t{{.Status}}'`, `curl -s -o /dev/null -w '%{http_code}\n' localhost:3000/api`. Run it; commit it.
  - Done when: Chatwoot survived a reboot; one script shows CPU/RAM/disk/containers/HTTP in one screen.

- [ ] **Day 35 · Tue 17 Nov — Write-up & buffer**
  - Do: `notes/week5.md`: server facts (provider, specs, IP, user, how to connect), what you changed on it (ufw, sshd, daemon.json). Commit. Catch up.
  - Done when: a new laptop + this repo would be enough to get back onto the server.

## Week 6 — Networking, reverse proxy, HTTPS (Days 36–42)

- [ ] **Day 36 · Wed 18 Nov — Ports, IPs, DNS, HTTP by hand**
  - Learn: [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/) · `ss`, `dig`, `curl -v`.
  - Do: on the server: `ip addr` (which IP is public?), `sudo ss -tlnp` (who listens where — rails only on `127.0.0.1:3000`), `dig +short github.com`, `curl -v http://localhost:3000/api` and read every request/response header, `nc -zv localhost 5432`. From your Mac: `nc -zv SERVER_IP 3000` → refused/timeout — good.
  - Done when: you can read `ss -tlnp` and explain `127.0.0.1:3000` vs `0.0.0.0:3000`.

- [ ] **Day 37 · Thu 19 Nov — Docker networking in depth, and the ufw trap**
  - Learn: [bridge driver](https://docs.docker.com/engine/network/drivers/bridge/) · [Docker and packet filtering/firewalls](https://docs.docker.com/engine/network/packet-filtering-firewalls/) — the part about Docker bypassing ufw.
  - Do: `docker network inspect chatwoot_default` (IPs of each container). DNS inside: `docker compose exec rails getent hosts postgres redis`. `sudo iptables -t nat -L DOCKER -n` — these rules are what `-p` creates, and they sit *in front of* ufw. Experiment: `docker run -d -p 0.0.0.0:8081:80 nginx:1.27` → from your Mac `curl SERVER_IP:8081` → it answers even though ufw never allowed 8081. Remove it. Rule for life: publish with `127.0.0.1:` unless you mean the whole internet.
  - Done when: `runbook.md` has a "Docker ignores ufw" warning with the proof you just saw.

- [ ] **Day 38 · Fri 20 Nov — Reverse proxy with Caddy (on your Mac first)**
  - Learn: [Caddy reverse proxy quick-start](https://caddyserver.com/docs/quick-starts/reverse-proxy) · what a reverse proxy does (one door in, routes to apps, handles TLS).
  - Do: add a `caddy` service (`caddy:2`, ports `80:80`, a `Caddyfile` bind-mounted to `/etc/caddy/Caddyfile`, volume `caddy_data:/data`). `Caddyfile`: `:80 { reverse_proxy rails:3000 }`. Remove the rails `ports:` block → `docker compose up -d` → http://localhost works, http://localhost:3000 doesn't. Open a conversation — ActionCable WebSockets go through Caddy fine. `docker compose logs caddy`.
  - Done when: Chatwoot via Caddy on port 80 locally.

- [ ] **Day 39 · Sat 21 Nov — HTTPS on the server with a real hostname**
  - Learn: [Caddy automatic HTTPS](https://caddyserver.com/docs/automatic-https) · [Cloudflare: What is SSL/TLS?](https://www.cloudflare.com/learning/ssl/what-is-ssl/)
  - Do: a hostname pointing at the server — default: a free [DuckDNS](https://www.duckdns.org) name (`yourname.duckdns.org` → server IP, 2 minutes). A paid domain only if you want a nicer URL (see **Costs**). `sudo ufw allow 80,443/tcp`. `Caddyfile`: `chatwoot.yourdomain.tld { reverse_proxy rails:3000 }`. `.env`: `FRONTEND_URL=https://chatwoot.yourdomain.tld`, `FORCE_SSL=true`. `docker compose up -d` → `docker compose logs caddy` shows the certificate being obtained → `curl -I https://chatwoot.yourdomain.tld` → 200.
  - Done when: your mentor could open the URL and see the Chatwoot login with a valid padlock.

- [ ] **Day 40 · Sun 22 Nov — Request-flow diagram; SSH as a tool**
  - Do: `diagrams/request-flow.md` (Mermaid): browser → DNS → server:443 (Caddy, TLS ends here) → rails:3000 (plain HTTP on the Docker network) → postgres/redis; sidekiq beside it; the WebSocket path. SSH tricks you'll need in Month 3: run a remote command `ssh lab 'docker ps'`, copy a file `scp file lab:~/`, `ssh-agent`/`ssh-add` so you type the passphrase once.
  - Done when: diagram committed (deliverable); `ssh lab 'cd chatwoot-lab/chatwoot && docker compose ps'` works from your Mac.

- [ ] **Day 41 · Mon 23 Nov — Break it #3: networking failures**
  - Do: (a) stop Caddy → from outside: connection refused; inside: `curl localhost:3000/api` fine → proxy problem, not app. (b) `sudo ufw deny 443` → from outside: *timeout* (not refused) → firewall. Undo. (c) `FRONTEND_URL` set to the wrong host → app loads but links/websocket break → config. Runbook: a table "timeout vs refused vs 502 vs 500 — which layer, what to check first".
  - Done when: runbook has 3 networking entries and the symptom table.

- [ ] **Day 42 · Tue 24 Nov — Write-up & buffer**
  - Do: `notes/week6.md`: your hostname, how TLS works here in 3 lines, the Caddyfile explained. Commit. Catch up.
  - Done when: you can draw the request path from memory.

## Week 7 — Swarm deeper: multi-node, failures, placement, health (Days 43–49)

- [ ] **Day 43 · Wed 25 Nov — Multi-node Swarm (Play with Docker)**
  - Learn: [How nodes work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/) · [Play with Docker](https://labs.play-with-docker.com) (log in with Docker Hub; sessions last 4 h).
  - Do: 3 instances → `docker swarm init --advertise-addr <node1 ip>` → `docker swarm join …` on the other two → `docker node ls`. `docker service create --name web --replicas 6 -p 8080:80 nginx:1.27` → `docker service ps web` shows tasks spread across nodes → `curl node2:8080` works even if no task runs there (routing mesh). `docker node update --availability drain node3` → tasks move → `--availability active`.
  - Done when: you watched tasks reschedule off a drained node.

- [ ] **Day 44 · Thu 26 Nov — Node failure, quorum, and why databases are special**
  - Learn: [Swarm administration guide](https://docs.docker.com/engine/swarm/admin_guide/) — the manager quorum section.
  - Do (PWD): delete a worker instance → `docker service ps web` shows its tasks `Shutdown` and new ones elsewhere; time it. Then: a `postgres` service with a named volume lands on node2 → delete node2 → Swarm reschedules it on node3 with an *empty* volume → the data is gone. This is the whole reason the brief says "more replicas ≠ safe data". With 1 manager, delete the manager → nothing can be managed; 3 managers survive losing 1.
  - Done when: `notes/week7.md`: "pin stateful services to one node, back them up; 3 managers for quorum."

- [ ] **Day 45 · Fri 27 Nov — Placement constraints, labels, resource limits**
  - Learn: [`deploy.placement`](https://docs.docker.com/reference/compose-file/deploy/#placement) · [`deploy.resources`](https://docs.docker.com/reference/compose-file/deploy/#resources).
  - Do: `docker node update --label-add db=true <node>` → in `stack/chatwoot-stack.yml`: postgres `deploy.placement.constraints: ["node.labels.db == true"]`; rails `deploy.resources.limits.memory: 1500M`, `reservations.memory: 600M`; sidekiq similar. Then set rails to `limits.memory: 200M` on purpose → `docker service ps chatwoot_rails --no-trunc` → `task: non-zero exit (137)` = OOM-killed → runbook entry → restore the real limit.
  - Done when: stack file has placement + resources; the 137 symptom is in the runbook.

- [ ] **Day 46 · Sat 28 Nov — Chatwoot stack on the server (single-node Swarm), keeping the data**
  - Do: on the server `docker swarm init` → `cd chatwoot && docker compose down` (no `-v`) → `docker volume ls` and note the Compose volume names (e.g. `chatwoot_postgres_data`). In the stack file declare them `external: true` with those exact names so the stack reuses the same data. Add Caddy to the stack (ports 80/443). `deploy.update_config: {order: start-first, failure_action: rollback}` on rails. `set -a; source chatwoot/.env; set +a; docker stack deploy -c stack/chatwoot-stack.yml chatwoot` → `docker stack ps chatwoot` → open https://… → your inbox and contacts are still there.
  - Done when: Chatwoot runs as a Swarm stack on the server with the same data and HTTPS.

- [ ] **Day 47 · Sun 29 Nov — Healthchecks and service discovery**
  - Learn: [Dockerfile `HEALTHCHECK`](https://docs.docker.com/reference/dockerfile/#healthcheck) · [Compose `healthcheck`](https://docs.docker.com/reference/compose-file/services/#healthcheck). Swarm only sends traffic to healthy tasks and only continues a rolling update when the new task is healthy.
  - Do: rails: `healthcheck: {test: ["CMD-SHELL", "wget -qO- http://127.0.0.1:3000/api || exit 1"], interval: 30s, timeout: 10s, retries: 3, start_period: 90s}` (wget, because the image has no curl — Day 20). postgres: `pg_isready -U postgres`. redis: `redis-cli -a $$REDIS_PASSWORD ping`. Redeploy → `docker ps` shows `(healthy)` → `docker service update --force chatwoot_rails` → the old task keeps serving until the new one is healthy.
  - Done when: all services report healthy; a forced update caused no downtime (keep `curl` looping in another terminal to prove it).

- [ ] **Day 48 · Mon 30 Nov — Upgrading Chatwoot itself: procedure and rollback limits**
  - Learn: [Chatwoot upgrade guide](https://developers.chatwoot.com/self-hosted/deployment/upgrade).
  - Do: check [releases](https://github.com/chatwoot/chatwoot/releases) for a tag newer than v4.18.0. The procedure: `scripts/backup.sh` (Day 52 — for now a manual `pg_dump`) → change the tag in the stack file → `stack deploy` → `docker exec … rails db:chatwoot_prepare` → verify. Rollback *after* migrations is not just "old image": it's old image + restore the dump. Write `runbook.md` → "Upgrade Chatwoot" with these steps. If no newer tag exists, rehearse with the same tag.
  - Done when: upgrade runbook written; you can say in one sentence why DB migrations make rollback harder.

- [ ] **Day 49 · Tue 1 Dec — Write-up & buffer**
  - Do: `notes/week7.md` complete; commit the final stack file. Catch up.
  - Done when: the server runs from `stack/chatwoot-stack.yml` in git, nothing hand-edited on the box.

## Week 8 — Scanning, secrets, backup/restore, incident drills, Meeting 2 (Days 50–56)

- [ ] **Day 50 · Wed 2 Dec — Image scanning with Trivy**
  - Learn: [Trivy docs](https://trivy.dev/latest/) — "Getting started".
  - Do: `docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --severity HIGH,CRITICAL chatwoot/chatwoot:v4.18.0` and the same for your `status-page`. `trivy fs --scanners secret .` on your repo. Sort findings: base-image OS packages (you can't fix — pick a newer tag), app gems (upstream's job), your own Dockerfile (yours).
  - Done when: `notes/week8.md` lists the top findings and who owns each.

- [ ] **Day 51 · Thu 3 Dec — Secrets done properly**
  - Do: on the server, `.env` is `chmod 600` and owned by you — check. Postgres already uses a Docker secret (Day 26); do the same for Redis via a config shim if you like. For Rails: Chatwoot reads env vars, so the honest options are (a) `.env` with tight permissions, or (b) an entrypoint wrapper that exports `/run/secrets/*` before `exec`-ing the real entrypoint. Try (b): `entrypoint: ["sh", "-c", "export POSTGRES_PASSWORD=$(cat /run/secrets/pg_password); exec docker/entrypoints/rails.sh \"$@\"", "--"]`. Rotate your Docker Hub / GHCR tokens to the minimum scope.
  - Done when: `notes/week8.md` → "secrets risks and what I did about each" (deliverable).

- [ ] **Day 52 · Fri 4 Dec — Backups that leave the server**
  - Learn: [Back up a volume](https://docs.docker.com/engine/storage/volumes/#back-up-restore-or-migrate-data-volumes).
  - Do: `scripts/backup.sh`: `docker exec $(docker ps -q -f name=chatwoot_postgres) pg_dump -U postgres -Fc chatwoot > backups/db-$(date +%F).dump` and `docker run --rm -v chatwoot_storage_data:/data -v $PWD/backups:/backup alpine tar czf /backup/storage-$(date +%F).tgz -C /data .`. `crontab -e` → daily at 03:00. From your Mac: `scp lab:~/chatwoot-lab/backups/* ~/chatwoot-backups/`.
  - Done when: today's backup exists on your Mac, not only on the server.

- [ ] **Day 53 · Sat 5 Dec — Restore drill (the only proof a backup works)**
  - Do: on your Mac (local lab only!): `docker compose down -v` → `docker compose up -d postgres` → `docker compose exec -T postgres pg_restore -U postgres -d postgres --create --clean --if-exists < ~/chatwoot-backups/db-*.dump` → restore storage: `docker run --rm -v chatwoot_storage_data:/data -v ~/chatwoot-backups:/backup alpine sh -c 'cd /data && tar xzf /backup/storage-*.tgz'` → `docker compose up -d` → log in with the *server's* admin account → data present. Time it. `runbook.md` → "Restore from backup", exact commands.
  - Done when: server data lives on your Mac; runbook entry records time-to-restore.

- [ ] **Day 54 · Sun 6 Dec — Incident drills: crash loop, disk full, stuck worker**
  - Do: (a) blank `SECRET_KEY_BASE` → redeploy → rails crash-loops → `docker service ps --no-trunc` + `docker service logs` → fix. (b) `fallocate -l <most of free space> /tmp/fill` → watch postgres/redis logs complain → `df -h`, `docker system df` → `rm /tmp/fill`. (c) `docker service scale chatwoot_sidekiq=0` → send a message → nothing happens → `redis-cli llen queue:default` grows → scale back → queue drains. Each one → runbook (symptom → detection → fix → prevention).
  - Done when: runbook has ≥ 2 fully documented failure scenarios (deliverable).

- [ ] **Day 55 · Mon 7 Dec — Meeting 2 prep**
  - Do: `meetings/meeting-2.md`: answer the 5 questions from the brief in writing; demo order (12 min) — request-flow diagram → the live HTTPS site → `healthcheck.sh` → kill rails live (`docker kill`) and watch Swarm recover → restore-drill result → Trivy summary. 3 questions for your mentor. Propose Month 3: CI/CD from GitHub to this server, final demo is the live app. Dry run with a timer.
  - Done when: `meeting-2.md` committed; dry run ≤ 15 min.

- [ ] **Day 56 · Tue 8 Dec — Buffer, second dry run, send agenda**
  - Do: fix what broke; send agenda + repo link.
  - **Month 2 deliverable check:** ☐ request-flow diagram ☐ CPU/RAM/disk/process/container/logs checklist (`healthcheck.sh`) ☐ runbook: port conflict, DB/Redis unreachable, crash loop ☐ ≥ 2 failure scenarios recorded ☐ secrets/image/persistence/backup risk notes ☐ bonus: live HTTPS Chatwoot on Swarm.

---

# Month 3 — CI/CD to the server, verify, monitor, rollback → final demo

**The design:** GitHub Actions runs checks → builds `status-page` (fast, ~2 min) → tags it `sha-<7>` → pushes to GHCR → SSHes to your server and runs `docker stack deploy` → waits for health → fails loudly if not healthy. Chatwoot itself is deployed from the official image by tag through the same stack file, so a Chatwoot version bump is "change one line, push". Building Chatwoot from source in CI is an optional stretch (Day 82).

**Final demo:** change a line in `status-page` → push → pipeline → the change is live on `https://chatwoot.yourdomain.tld/status`; then push a broken build → pipeline goes red, Swarm rolls back, site stays up.

## Week 9 — CI: checks, tests, build, push (Days 57–63)

- [ ] **Day 57 · Wed 9 Dec — GitHub Actions: first workflow**
  - Learn: [GitHub Actions quickstart](https://docs.github.com/en/actions/quickstart) · [workflow syntax](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions) — `on`, `jobs`, `steps`, `runs-on`.
  - Do: push the repo to GitHub as `chatwoot-lab`, **public** (free unlimited Actions minutes, free GHCR, environment protection rules — see Costs; first: `git log --all -p | grep -ci 'password=' ` must be 0, and `.env` must not be tracked). `.github/workflows/ci.yml`: `on: [push, pull_request]`, one job on `ubuntu-latest`: `actions/checkout@v4`, `docker --version`, `echo "hello from CI"`. Watch it in the Actions tab.
  - Done when: a green check on your commit.

- [ ] **Day 58 · Thu 10 Dec — Quality checks: Compose config, Dockerfile lint, shell lint**
  - Do: steps: `docker compose -f chatwoot/docker-compose.yaml config -q` (give CI dummy values via `env:` or a committed `.env.ci`), `hadolint/hadolint-action@v3.1.0` on `status-page/Dockerfile`, `shellcheck scripts/*.sh`. Break the Compose YAML on a branch → red → fix → green.
  - Done when: CI catches a broken Compose file.

- [ ] **Day 59 · Fri 11 Dec — Tests for status-page**
  - Do: add `status-page/test/app_test.rb` (minitest + rack-test): `GET /` → 200, `GET /health` → JSON with `status`. `bundle exec rake test` locally. CI job `test`: `ruby/setup-ruby@v1` with `bundler-cache: true`, `working-directory: status-page`, `bundle exec rake test`. Make a test fail on a branch → red.
  - Done when: a failing test blocks the pipeline; passing tests are the gate for the next job.

- [ ] **Day 60 · Sat 12 Dec — Build the image in CI**
  - Learn: [Docker's GitHub Actions guide](https://docs.docker.com/build/ci/github-actions/) — `setup-buildx-action`, `metadata-action`, `build-push-action`.
  - Do: job `build` with `needs: test`: `docker/setup-buildx-action@v3` → `docker/metadata-action@v5` (`images: ghcr.io/${{ github.repository_owner }}/status-page`, `tags: type=sha,prefix=sha-` and `type=semver,pattern={{version}}`) → `docker/build-push-action@v6` with `context: status-page`, `push: false`, `load: true` → smoke test: `docker run -d -p 4567:4567 <image>` + `curl -fsS localhost:4567/health`.
  - Done when: the Actions log shows an image tagged `sha-<7>` built and smoke-tested.

- [ ] **Day 61 · Sun 13 Dec — Push to GHCR from CI, with build cache**
  - Do: `permissions: {contents: read, packages: write}` on the job → `docker/login-action@v3` (`registry: ghcr.io`, `username: ${{ github.actor }}`, `password: ${{ secrets.GITHUB_TOKEN }}`) → `push: ${{ github.ref == 'refs/heads/main' }}` → `cache-from: type=gha`, `cache-to: type=gha,mode=max`. Push twice; compare build times. Check the package page on GitHub; link it to the repo.
  - Done when: `ghcr.io/you/status-page:sha-<7>` was pushed by CI, not by you; second build is faster.

- [ ] **Day 62 · Mon 14 Dec — Trivy in CI; branch protection; PR flow**
  - Do: `aquasecurity/trivy-action` (pin its latest release tag) on the built image, `severity: CRITICAL,HIGH`, `ignore-unfixed: true`, `exit-code: 1`. Settings → Branches → protect `main`: require the CI checks before merge. Work on a branch → open a PR → see checks → merge → merge triggers the push to GHCR.
  - Done when: a PR shows test + build + scan checks; `main` can't be pushed to with red checks.

- [ ] **Day 63 · Tue 15 Dec — Write-up & buffer**
  - Do: `diagrams/pipeline.md` v1 (Mermaid): PR → checks → merge → build → push GHCR. `notes/week9.md`. Commit.
  - Done when: a stranger could read `ci.yml` with your notes beside it.

## Week 10 — CD: deploy to the server (Days 64–70)

- [ ] **Day 64 · Wed 16 Dec — Deploy design and a dedicated deploy key**
  - Learn: the three ways to deploy from CI: push over SSH (simple, explicit — **we use this**), pull-based agents (Watchtower/Shepherd), self-hosted runner. Note the trade-offs for your mentor.
  - Do: on the server: user `deploy` in the `docker` group, `~/chatwoot-lab` checked out. On your Mac: `ssh-keygen -t ed25519 -f deploy_key -C ci-deploy` → public key into `deploy`'s `authorized_keys` → test `ssh -i deploy_key deploy@host 'docker ps'`. Repo secrets: `DEPLOY_SSH_KEY` (private key), `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_KNOWN_HOSTS` (`ssh-keyscan -H host`). Delete `deploy_key` from your Mac.
  - Done when: secrets set; the key works and lives only in GitHub + the server.

- [ ] **Day 65 · Thu 17 Dec — status-page joins the stack; it reports on Chatwoot**
  - Do: `app.rb`: `/health` now also does `GET http://rails:3000/api` and returns `{status, chatwoot: "ok"|"down", image_tag: ENV["IMAGE_TAG"]}`; `/` shows the same in HTML. Stack: service `status` with `image: ghcr.io/you/status-page:${STATUS_TAG}`, `environment: [IMAGE_TAG=${STATUS_TAG}]`, healthcheck on `/health`. Caddyfile: `handle_path /status* { reverse_proxy status:4567 }`. Make the GHCR package public (lab) or `docker login ghcr.io` on the server with a read-only PAT and deploy with `--with-registry-auth`. Deploy by hand once from the server.
  - Done when: `https://…/status` shows the image tag and "chatwoot: ok".

- [ ] **Day 66 · Fri 18 Dec — The deploy job**
  - Do: job `deploy` (`needs: build`, `if: github.ref == 'refs/heads/main'`): write the key to `~/.ssh/id_ed25519` (chmod 600) and `known_hosts` from secrets → `ssh deploy@host "cd ~/chatwoot-lab && git pull --ff-only && set -a && . chatwoot/.env && set +a && STATUS_TAG=sha-${GITHUB_SHA::7} docker stack deploy --with-registry-auth -c stack/chatwoot-stack.yml chatwoot"`. Change the text on `/` → push to `main` → watch the job → refresh `/status`.
  - Done when: a commit to `main` reaches the server with zero manual steps.

- [ ] **Day 67 · Sat 19 Dec — Verify step and a health gate**
  - Do: after deploy, a `verify` step: loop up to 120 s on `ssh … "docker service inspect chatwoot_status --format '{{.UpdateStatus.State}}'"` until `completed` (or fail on `rollback_completed`/`paused`), then `curl -fsS https://…/status/health | grep '"chatwoot":"ok"'` and `curl -fsS -o /dev/null https://…/api`. Now break it on a branch: make `/health` return 500 → merge → Swarm's `failure_action: rollback` keeps the old task, `verify` fails, pipeline red, site still fine.
  - Done when: a bad deploy turns the pipeline red while the previous version keeps serving.

- [ ] **Day 68 · Sun 20 Dec — Chatwoot version bump through the pipeline (migrations)**
  - Do: `CHATWOOT_TAG` in the stack file (`image: chatwoot/chatwoot:${CHATWOOT_TAG}`), set in `.env` on the server. Deploy step gains a migration run after `stack deploy`: `docker exec $(docker ps -q -f name=chatwoot_rails | head -1) bundle exec rails db:chatwoot_prepare`. Pre-deploy backup: call `scripts/backup.sh` in the same SSH session. If a tag newer than v4.18.0 exists, bump it on a branch → PR → merge → watch → verify. Update `runbook.md` → "Upgrade Chatwoot" to the pipeline version.
  - Done when: the pipeline can change the Chatwoot version with backup + migration + verify, and the runbook says what to do if it fails.

- [ ] **Day 69 · Mon 21 Dec — Manual approval, release tags, a rollback button**
  - Learn: [GitHub environments](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment) (protection rules need a public repo or GitHub Pro).
  - Do: `environment: staging` on the deploy job with yourself as required reviewer → deploys wait for a click. `workflow_dispatch` with input `status_tag` → deploys any existing tag (that's your rollback button). Try it: dispatch with yesterday's `sha-…` → `/status` shows the old tag. `git tag v0.1.0 && git push --tags` → `metadata-action` also tags the image `0.1.0`.
  - Done when: you rolled back from the Actions UI without touching the server.

- [ ] **Day 70 · Tue 22 Dec — Write-up & buffer**
  - Do: `diagrams/pipeline.md` final: code change → CI/tests → build → push → deploy → verify → monitor/rollback. `notes/week10.md`: deploy design + trade-offs. Commit.
  - Done when: diagram matches `ci.yml` step for step.

## Week 11 — Logs, metrics, alerts, incident drills (Days 71–77)

Christmas week — Days 73 and 77 are swappable; keep the drills.

- [ ] **Day 71 · Wed 23 Dec — Logs you can actually use**
  - Do: `docker service logs -f --tail 100 chatwoot_rails`; confirm rotation from Day 32 is active (`docker inspect <cid> --format '{{.HostConfig.LogConfig}}'`). Add [Dozzle](https://dozzle.dev) (`amir20/dozzle`, mounts the Docker socket read-only) to a `monitoring` stack behind Caddy at `/logs` with `basic_auth`. Chatwoot `LOG_LEVEL=info` vs `debug` — try, compare, set back.
  - Done when: all container logs readable in the browser, behind a password.

- [ ] **Day 72 · Thu 24 Dec — Metrics: Prometheus, cAdvisor, node-exporter**
  - Learn: [Prometheus overview](https://prometheus.io/docs/introduction/overview/) · [cAdvisor](https://github.com/google/cadvisor) README (what it exposes).
  - Do: `stack/monitoring-stack.yml`: `prom/prometheus`, `gcr.io/cadvisor/cadvisor`, `prom/node-exporter` (host mode), a `prometheus.yml` scraping both. Deploy → Prometheus UI (through an SSH tunnel, don't publish it) → Status → Targets all `UP` → query `container_memory_usage_bytes{name=~"chatwoot.*"}`.
  - Done when: a graph of Chatwoot's RAM per container in Prometheus.

- [ ] **Day 73 · Fri 25 Dec — Grafana dashboards (light day)**
  - Do: add `grafana/grafana` behind Caddy at `/grafana` → Prometheus as data source → import dashboards **193** (Docker containers) and **1860** (Node Exporter Full). Look at CPU/RAM/disk/network for the server and per container.
  - Done when: one Grafana screen shows host + container health. (Or swap with Day 77 if you're off today.)

- [ ] **Day 74 · Sat 26 Dec — Alerts and uptime checks**
  - Do: Grafana alert rules: container restarts (`increase(container_last_seen…)` or `changes(container_start_time_seconds[10m]) > 0`), memory > 90%, disk < 15% → contact point: email or Telegram. External: [UptimeRobot](https://uptimerobot.com) (free) on `https://…/api` and `/status/health` every 5 min. [healthchecks.io](https://healthchecks.io) ping at the end of `backup.sh` so you hear about a backup that *didn't* run.
  - Done when: `docker kill` a container → an alert reaches your phone/email within 10 min.

- [ ] **Day 75 · Sun 27 Dec — Incident drill #1: bad deploy, end to end**
  - Do: on a branch make `status-page` crash on boot → merge → observe in order: pipeline red at `verify` → Swarm rollback in `docker service ps` → Dozzle shows the crash → Grafana shows restarts → alert fires. Write `postmortems/2026-12-27-bad-deploy.md`: timeline, detection, cause, fix, prevention. Fix forward with a PR.
  - Done when: a postmortem with real timestamps.

- [ ] **Day 76 · Mon 28 Dec — Incident drill #2: bad config in production**
  - Do: change `REDIS_URL` in the server `.env` to a wrong host → `stack deploy` → sidekiq crash-loops, rails degrades → detect from alerts/dashboards (not from logs first) → roll back with the `workflow_dispatch` button → confirm health → postmortem #2. Then the rollback guide: `runbook.md` → "How to roll back" (image tag via Actions; config via git revert + deploy; data via restore).
  - Done when: postmortem #2 and the rollback guide are committed.

- [ ] **Day 77 · Tue 29 Dec — Write-up & buffer**
  - Do: `notes/week11.md`: what each monitoring piece answers (logs = what happened; metrics = how much/when; alerts = wake me up). Commit.
  - Done when: you can say which tool you'd open first for each symptom in the runbook table.

## Week 12 — Polish, rehearse, final review (Days 78–84)

- [ ] **Day 78 · Wed 30 Dec — README: the whole system**
  - Do: `README.md` final: what runs where (diagram), how to run locally, how to deploy (push to `main`), how to roll back, how to restore, where logs/metrics are, the tag strategy. Link every diagram and the runbook.
  - Done when: README answers "how do I deploy / roll back / restore / see logs" in under a minute each.

- [ ] **Day 79 · Thu 31 Dec — Runbook consolidation**
  - Do: reread every runbook entry; one consistent format (symptom · how you notice · first check · cause · fix · prevention). Merge duplicates. `scripts/healthcheck.sh` final. Commit.
  - Done when: runbook has ≥ 10 clean entries.

- [ ] **Day 80 · Fri 1 Jan — Holiday / buffer**
  - Do: nothing, or whatever slipped. Happy new year.

- [ ] **Day 81 · Sat 2 Jan — Dress rehearsal #1**
  - Do: fresh clone on your Mac → follow README → local Chatwoot up. Then the 15-minute demo with a timer: (1) architecture + request flow, (2) change a line → push → pipeline → live on `/status`, (3) bad build → red → rollback → site fine, (4) Grafana/Dozzle/alert, (5) backup + restore summary, (6) trade-offs. Note everything that stumbled.
  - Done when: run-through ≤ 15 min; stumbles listed.

- [ ] **Day 82 · Sun 3 Jan — Stretch (optional): build Chatwoot from source in CI**
  - Do: a second workflow `chatwoot-image.yml`, `workflow_dispatch` only: checkout `chatwoot/chatwoot` at a tag → `build-push-action` with `file: docker/Dockerfile` → push to `ghcr.io/you/chatwoot:sha-…`. Expect 20–40 min and a ~1 GB image; that's the point — now you can explain why the main pipeline doesn't. Skip without guilt if Day 81 left fixes to do.
  - Done when: either an image on GHCR, or the stumbles from Day 81 fixed.

- [ ] **Day 83 · Mon 4 Jan — Meeting 3 prep**
  - Do: `meetings/meeting-3.md`: deliverables vs the brief (section 5, below); trade-offs you chose (SSH push deploy vs pull, single-node Swarm, env-file secrets, official image vs source build, no K8s); 3 things you'd do next (multi-node with managed DB, IaC with Ansible/Terraform, Kubernetes basics, proper secrets manager); questions for your mentor.
  - Done when: `meeting-3.md` committed.

- [ ] **Day 84 · Tue 5 Jan — Dress rehearsal #2, send agenda**
  - Do: full run-through once more; send the agenda, README link and the live URL to your mentor. After the meeting: snapshot or delete the server so billing stops (your backups are on your Mac — Day 52).
  - **Month 3 deliverable check:** ☐ CI runs checks/tests ☐ image built/tagged/pushed automatically ☐ deploy to the server from CI ☐ health verification step ☐ logs/investigation/rollback guide ☐ simulated bad deploy detected and recovered ☐ live demo URL.

---

## Definition of Done (from the brief, section 5)

- [ ] Start and describe Chatwoot's main services with Docker Compose.
- [ ] Explain image, container, volume, network, port, env vars, logs.
- [ ] Publish/pull images via a registry with explicit tags.
- [ ] Demo Swarm basics: services, replicas, update, rollback.
- [ ] Investigate Docker/networking problems with a systematic process.
- [ ] Demo CI/CD from code checks to a deployed, verified lab/staging environment.
- [ ] README, architecture diagrams, and runbook for the lab app.

## Resources

- Chatwoot: [repo](https://github.com/chatwoot/chatwoot) · [Docker guide](https://developers.chatwoot.com/self-hosted/deployment/docker) · [architecture](https://developers.chatwoot.com/self-hosted/deployment/architecture) · [upgrade](https://developers.chatwoot.com/self-hosted/deployment/upgrade) · [releases](https://github.com/chatwoot/chatwoot/releases)
- Docker: [Docker concepts (beginner series)](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/) · [Compose](https://docs.docker.com/compose/) · [Swarm](https://docs.docker.com/engine/swarm/) · [Build in GitHub Actions](https://docs.docker.com/build/ci/github-actions/)
- GitHub Actions: [docs](https://docs.github.com/en/actions) · GHCR: [docs](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- Caddy: [docs](https://caddyserver.com/docs/) · Trivy: [docs](https://trivy.dev/latest/) · Prometheus: [docs](https://prometheus.io/docs/) · Grafana: [docs](https://grafana.com/docs/)
- Play with Docker: [labs.play-with-docker.com](https://labs.play-with-docker.com)
