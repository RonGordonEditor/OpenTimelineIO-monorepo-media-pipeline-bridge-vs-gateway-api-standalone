# OpenTimelineIO monorepo: media-pipeline-bridge (gateway-api service)

A small, production-minded “bridge” between editorial timelines (via **OpenTimelineIO**) and downstream media pipeline tasks (initially: **thumbnail rendering**), using:

- **FastAPI** for an HTTP “gateway” service
- **PostgreSQL** for durable timeline + render metadata
- **Redis + RQ** for background jobs
- **ffmpeg** (discovered via `imageio_ffmpeg.get_ffmpeg_exe()` to avoid local install pain)
- **Artifacts-on-disk** (thumbnails, stored OTIO, etc.) with stable paths/URLs

This repo is designed as a **monorepo** so the “service” and the “artifacts it generates” can live together in one coherent example project.

---

## Why this exists (portfolio context)

Post-production pipelines often need to move between NLEs and finishing systems while also generating “sidecar” media (thumbnails, proxies, QC frames, etc.) and tracking everything in a database. This project demonstrates:

- OTIO-based ingest (editorial timeline → database)
- A normalized DB schema (cuts → shots → renders/jobs)
- Async worker execution (RQ) and status polling
- A repeatable local dev workflow (server + worker)
- A path toward multi-machine scaling (multiple workstations/workers sharing DB + Redis + artifacts)

---

## High-level workflow

1. **Publish** an OTIO file for a cut/version.
2. Server stores the OTIO under `ARTIFACT_ROOT/` and creates DB records.
3. Client lists shots for a cut version.
4. Client requests a **thumbnail render** for a specific shot.
5. Server creates a `renders` row + `jobs` row and enqueues an RQ job.
6. Worker runs ffmpeg to generate `thumbnail.jpg` and updates DB with output path/URL.
7. Client polls `/jobs/{job_id}` until `succeeded`.

---

## Repo layout

- `services/gateway-api/`
  - `app/main.py` — FastAPI app
  - `app/models.py` — SQLAlchemy models
  - `app/config.py` — Settings / env vars
  - `app/tasks.py` — OTIO ingest worker task(s)
  - `app/render_tasks.py` — render worker task(s)
  - `migrations/` — Alembic migrations
- `artifacts/`
  - stored OTIO and generated renders (thumbnails)

---

## Requirements

- Python 3.13+
- Postgres (local). Recommended on macOS: **Postgres.app**
- Redis (local) for RQ
- ffmpeg: **not required** as a system install (we use `imageio-ffmpeg`)

---

## Local setup

### 1) Create a virtual environment + install deps

From `services/gateway-api/`:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
