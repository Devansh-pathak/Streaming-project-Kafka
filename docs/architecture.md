# Architecture (Target)

## Overview
1. Producer publishes clickstream events to Kafka topic `user_events`.
2. Streaming job ingests Kafka into Bronze Delta table (raw payload + metadata).
3. Silver stream parses schema, validates records, and deduplicates by `event_id`.
4. Gold stream computes KPIs and anomaly/fraud signals.
5. Alerts sink publishes suspicious events to dashboard + notification channels.

## Data Model Additions
- `event_id` (string): unique event identity for deduplication.
- `schema_version` (string): schema evolution compatibility.
- `ingest_ts` (timestamp/string): producer-side ingest marker.

## Reliability Controls
- Watermarking and deduplication in silver layer.
- Separate checkpoint locations per stream/query.
- Quarantine table for malformed JSON payloads.
