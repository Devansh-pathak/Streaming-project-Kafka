# Streaming-project-Kafka
Project where we stream live data via kafka and detect anomalies in realtime

## Project status and roadmap
See `docs/PROJECT_AUDIT_AND_ROADMAP.md` for a detailed assessment and enhancement plan.

## Security-first update
Hardcoded Kafka credentials have been removed from notebooks.
Set these environment variables before running notebooks:
- `KAFKA_BOOTSTRAP_SERVERS`
- `KAFKA_API_KEY`
- `KAFKA_API_SECRET`
- `KAFKA_TOPIC` (optional)

## Additional documentation
- `docs/architecture.md`
- `docs/runbook.md`
- `config/config.example.yaml`
