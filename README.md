
# OpenTimelineIO media-pipeline-bridge
A Python-based prototype that bridges editorial timelines (Avid/AAF → OTIO) to finishing systems (Baselight) via a database-driven “shots + renders” workflow.

This repo is designed to become a practical interoperability layer:
- **Ingest** editorial sequence metadata into a normalized DB model (projects → cuts → shots)
- **Query** shot lists from a REST API
- **Render** artifacts (starting with thumbnails) via an async worker queue (Redis/RQ + FFmpeg)
- **Persist** render outputs (path + URL) back to the database

> Status: working API + worker prototype; OTIO/AAF ingest → shots in DB; thumbnail renders via FFmpeg; Baselight integration planned.

---

## Why this exists
Post pipelines frequently break at the boundary between:
- **Editorial**: “timeline semantics” (cuts, track items, source ranges)
- **Finishing**: “media + conforms + renders”

The goal is a small, composable bridge where the database becomes the source of truth for:
- shot-level metadata
- render state (queued/running/succeeded/failed)
- artifact locations

---

## Architecture (current)
**FastAPI gateway** + **Postgres** + **Redis/RQ worker**:

1. Editorial ingest populates `projects`, `cuts`, `cut_versions`, `shots`
2. API endpoint requests an artifact render for a `shot_id`
3. API creates a `renders` row + a `jobs` row, then enqueues an RQ task
4. Worker runs FFmpeg, writes an artifact under `/artifacts`, updates DB with output path/URL

---

## Quickstart (local dev)

### Prereqs
- Python 3.11+ (tested with 3.13)
- PostgreSQL 14+ (tested with Postgres.app / v17)
- Redis
- FFmpeg available to the worker (either system install or `imageio-ffmpeg` fallback)

### Setup
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
