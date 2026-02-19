# Elastic-SIEM-Local-Brute-Force-Detection-Lab

# Elastic SIEM – Local Brute Force Detection Lab (Windows)

Local SIEM lab built with Elastic to ingest Windows Security logs and detect potential brute-force login attempts.
The lab uses a **standalone Elastic Agent** on a Windows host and runs Elasticsearch + Kibana locally via **Docker/WSL2**.

## Architecture
Windows (Elastic Agent, standalone) → Elasticsearch → Kibana  
Runtime: Docker Desktop + WSL2 (Ubuntu)

## Data Source
- Windows Security Event Logs
- Focus: failed logon events (**Event ID 4625**)

Example query used for hunting and detection:
```kql
event.code: 4625
