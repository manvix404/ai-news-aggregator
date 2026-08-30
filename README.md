**AI News Aggregator**

A small pipeline that scrapes AI-related content (YouTube, OpenAI, Anthropic), enriches it (transcripts, markdown), generates concise digests with LLMs, ranks them for a user profile, and sends a personalized daily email.

**Key Features**

- **Multi-source scraping:** YouTube RSS + transcripts, OpenAI RSS, Anthropic RSS.
- **Enrichment:** URL -> Markdown (Anthropic), transcripts for videos.
- **Digest generation:** LLM-driven title + 2–3 sentence summary per article.
- **Personalized curation:** LLM ranks digests against a `USER_PROFILE`.
- **Email delivery:** Renders markdown to HTML and sends via SMTP.

**High-level Architecture**

- **Scrapers:** `app/scrapers/` — fetch raw content and metadata.
- **Database (Postgres):** `app/database/` — SQLAlchemy models and `Repository` for data access.
- **Processing services:** `app/services/` — enrichment, digest creation, ranking orchestration, and email sending.
- **Agents:** `app/agent/` — small LLM wrappers (digest, curator, email) that encapsulate prompts and parsing.
- **Orchestration:** `app/daily_runner.py` — sequential pipeline executed by `main.py` for the daily job.

**Data Flow (simplified)**

1. Scrapers collect new items and insert them into the DB.
2. Background processing populates missing content (markdown/transcripts).
3. Digest generation converts content into short summaries and stores them as `Digest` records.
4. Curator agent ranks recent digests for a `USER_PROFILE`.
5. Email agent composes an introduction and top-N list; `app/services/email.py` renders and sends the email.

**Quickstart (local)**

1. Create a `.env` with required vars: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_HOST`, `POSTGRES_PORT`, `OPENAI_API_KEY`, `MY_EMAIL`, `APP_PASSWORD`.
2. Start Postgres (via Docker compose):

```bash
docker compose -f docker/docker-compose.yml up -d
```

3. (Optional) Create DB tables:

```bash
python -m app.database.create_tables
```

4. Run the daily pipeline (one-off):

```bash
python main.py
```

**Env vars used**

- `POSTGRES_*` — DB connection
- `OPENAI_API_KEY` — LLM access
- `MY_EMAIL`, `APP_PASSWORD` — SMTP credentials for sending email

**Deployment notes**

- `render.yaml` includes a scheduled cron-like service target; a `Dockerfile` is referenced but not included in the repo. Add a `Dockerfile` to containerize `python main.py` for scheduled runs.
