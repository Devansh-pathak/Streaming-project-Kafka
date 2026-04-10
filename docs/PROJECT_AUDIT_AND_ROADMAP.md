# Project Audit & Enhancement Roadmap

## 1) What you have built so far

You already have the core building blocks of a real-time streaming analytics pipeline:

1. **Kafka event producer notebook** (`02_Kafka_Generator.ipynb`)
   - Installs and uses `confluent_kafka`.
   - Connects to Confluent Cloud.
   - Generates synthetic clickstream/e-commerce events (`view_item`, `add_to_cart`, `purchase`, etc.) and publishes to Kafka topic `user_events`.

2. **Analytics/consumer notebook** (`03_Analytics.ipynb`)
   - Defines a schema for incoming JSON events.
   - Configures secure Kafka options (SASL/SSL).
   - Uses Delta storage paths under Databricks Volume (`/Volumes/workspace/default/my_volume/...`).
   - Reads saved data and shows live stream from Delta.
   - Implements a first fraud heuristic: `>= 4 clicks in 10 seconds` per `ip_address`.

3. **Project framing** (`README.md`)
   - Declares project objective: stream live data via Kafka and detect anomalies in real time.

---

## 2) Current gaps to improve before scaling

### A. Security (highest priority)
- API key and secret are hardcoded directly in notebooks.
- This is risky for source control and team collaboration.

### B. Code organization and maintainability
- Logic currently sits in notebooks only.
- Hard to test, reuse, and deploy in CI/CD.

### C. Data quality and schema governance
- No explicit handling for malformed records, nulls, duplicates, or schema changes.

### D. Observability
- No clear metrics/monitoring/alerts for pipeline lag, throughput, bad events, or query failures.

### E. Product functionality
- Current anomaly rule is a good start but still simple.
- No user/session features, no scoring service, no alert routing.

---

## 3) Enhancement plan (prioritized)

## Phase 1 — Production hygiene (quick wins: 1–2 days)
1. **Remove hardcoded secrets**
   - Use Databricks secret scopes or environment variables injected at runtime.
   - Rotate currently exposed credentials.

2. **Centralize configuration**
   - Keep topic names, Kafka bootstrap server, paths, and thresholds in a `config.yaml` or Python config module.

3. **Create medallion layers explicitly**
   - Bronze: raw Kafka payloads + ingest metadata.
   - Silver: parsed, deduped, validated records.
   - Gold: KPI and fraud/alert aggregates.

4. **Add idempotency and deduplication**
   - Introduce deterministic event id (`event_id`) in producer.
   - Deduplicate in silver layer using watermark + dropDuplicates.

## Phase 2 — Engineering quality (2–5 days)
1. **Refactor notebooks into Python package**
   - `src/producer/`, `src/streaming/`, `src/features/`, `src/anomaly/`.
   - Keep notebooks for exploration only; production logic in `.py` files.

2. **Add testing**
   - Unit tests for parsing, validation, rule logic.
   - Contract tests for schema evolution.

3. **Add CI**
   - Lint (`ruff`), format (`black`), tests (`pytest`) on each PR.

4. **Structured logging**
   - Log event counts, bad records, and query progress in JSON logs.

## Phase 3 — New functionality (1–2 weeks)
1. **Richer anomaly detection**
   - Add velocity checks by user, per item abuse rates, impossible geo movement, and purchase spikes.
   - Build a weighted risk score instead of one hard threshold.

2. **Real-time alerting**
   - Write suspicious events to an alerts topic/table.
   - Send notifications (Slack/webhook/email).

3. **Analytics dashboards**
   - Conversion funnel by traffic source.
   - Top suspicious IPs/items over time.
   - Stream health dashboard (lag, throughput, failure rate).

4. **Replay/backfill support**
   - Parameterized start offsets/time window for historical replay and model/rule re-runs.

---

## 4) Suggested target architecture

- **Producer service** (Kafka producer with event_id + schema version)
- **Kafka topic(s)** (`user_events_raw`, optional `alerts`)
- **Bronze stream** (raw append-only Delta)
- **Silver stream** (validated, deduped, typed, enriched)
- **Gold stream** (metrics + anomaly scores)
- **Alert sink** (Slack/webhook/topic)
- **Monitoring** (query status, lag, bad-record rate)

---

## 5) Concrete code-level upgrades to implement first

1. Add `event_id`, `schema_version`, and `ingest_ts` fields in producer payload.
2. Parse JSON with strict schema and route malformed events to quarantine table.
3. Convert timestamps early and standardize to UTC.
4. Add `dropDuplicates(["event_id"])` with watermark in silver stream.
5. Move rule thresholds to configuration (no hardcoded constants).
6. Persist anomaly results into a Delta table and expose in dashboard.

---

## 6) Nice-to-have features after stabilization

- Feature store for reusable streaming features.
- Online model scoring for fraud probability.
- A/B testing framework for rule changes.
- Data contracts and schema registry integration.
- Load testing harness for peak traffic simulation.

---

## 7) Immediate next actions for this repository

1. Rotate exposed Confluent credentials immediately.
2. Replace hardcoded secrets in both notebooks with secret references.
3. Add `docs/architecture.md` and `docs/runbook.md`.
4. Add repo scaffolding:
   - `src/`
   - `tests/`
   - `pyproject.toml`
   - `.github/workflows/ci.yml`
5. Keep notebooks as demo surfaces, but run production logic from Python modules.

