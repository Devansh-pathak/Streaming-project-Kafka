# Runbook

## Prerequisites
- Databricks workspace with Kafka connector availability.
- Confluent Cloud API key/secret.

## Required Environment Variables
- `KAFKA_BOOTSTRAP_SERVERS`
- `KAFKA_API_KEY`
- `KAFKA_API_SECRET`
- `KAFKA_TOPIC` (optional, defaults to `user_events`)

## Setup
1. Set environment variables in your notebook/job cluster.
2. Open `02_Kafka_Generator.ipynb` and start producer.
3. Open `03_Analytics.ipynb` and run analytics cells.

## Security Rotation (Immediate)
1. Revoke previously committed Confluent credentials.
2. Create new credentials in Confluent Cloud.
3. Store them in secret scope / runtime environment variables only.

## Common Checks
- Validate stream writes are visible under configured Delta paths.
- Verify fraud alerts stream is active and checkpoint is advancing.
