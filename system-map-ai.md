# System Map AI Prompts

## 1. Describe the System


⏺                       ┌────────────────────────────────────┐
                        │              BROWSER                │
                        └──────────────────┬─────────────────┘
                                           │ HTTP :3000
                                           ▼
                                ┌──────────────────────┐
                                │   React Frontend     │
                                │    [React 16]        │
                                │       :3000          │
                                └──────────┬───────────┘
                                           │ HTTP/JSON :8000
                                           ▼
         ┌─────────────┐  HTTP   ┌─────────────────────────────┐   HTTP   ┌─────────────┐
         │  GitHub API │◀───────│          Django API          │─────────▶│  Slack API  │
         │  (external) │───────▶│  [Django · DRF · Py 3.11]   │          │  (external) │
         └─────────────┘  OAuth │             :8000            │          └─────────────┘
                                └──────┬──────────────┬─────────┘
                                       │              │
                            DB :5432   │   PUB :6379  │
                                       ▼              ▼
                            ┌──────────────┐    ┌────────────────┐
                            │  PostgreSQL  │    │     Valkey     │
                            │   [pg 16]    │    │     :6379      │
                            │    :5432     │    └───────┬────────┘
                            └──────┬───────┘            │ SUB :6379
                                   │                    ▼
                                   │         ┌──────────────────────┐   HTTP   ┌─────────────┐
                                   │         │       Monarch        │─────────▶│  GitHub API │
                                   │         │  [Python · asyncio]  │   HTTP   │  (external) │
                                   │         │   :8080  ·  :8081    │─────────▶│  Slack API  │
                                   │         └──────────────────────┘          │  (external) │
                                   │                                            └─────────────┘
                                   │ DB :5432
                                   ▼
                            ┌──────────────────┐
                            │Postgres Exporter │
                            │      :9187       │
                            └──────────┬───────┘
                                       │ HTTP scrape :9187
                                       ▼
                            ┌──────────────────┐
                            │   Prometheus     │◀─── HTTP scrape :8000/metrics ── Django API
                            │     :9090        │
                            └──────────┬───────┘
                                       │ HTTP query :9090
                                       ▼
                            ┌──────────────────┐
                            │     Grafana      │
                            │      :3001       │
                            └──────────────────┘

## 2. Convert to a Mermaid Diagram

```mermaid

graph TD
      BROWSER(["🌐 BROWSER"])

      subgraph frontend["Client"]
          React["React Frontend\nReact 16 · :3000"]
      end

      subgraph api["Backend"]
          Django["Django API\nDjango · DRF · Python 3.11 · :8000"]
      end

      subgraph data["Data Layer"]
          PG[("PostgreSQL\npg 16 · :5432")]
          Valkey[("Valkey\nMessage Broker · :6379")]
      end

      subgraph monarch_svc["Microservice"]
          Monarch["Monarch\nPython · asyncio · :8080 · :8081"]
      end

      subgraph observability["Observability"]
          PG_Exp["Postgres Exporter · :9187"]
          Prometheus["Prometheus · :9090"]
          Grafana["Grafana · :3001"]
      end

      subgraph external["External Services"]
          GitHub(["GitHub API"])
          Slack(["Slack API"])
      end

      BROWSER -->|"HTTP :3000"| React
      React -->|"HTTP/JSON :8000"| Django

      Django <-->|"HTTP OAuth"| GitHub
      Django -->|"HTTP REST"| Slack

      Django -->|"DB · psycopg2 · :5432"| PG
      Django -->|"PUB · :6379"| Valkey

      Valkey -->|"SUB · :6379"| Monarch
      Monarch -->|"HTTP REST"| GitHub
      Monarch -->|"HTTP REST"| Slack

      PG -->|"DB · :5432"| PG_Exp
      Prometheus -->|"HTTP scrape · :9187"| PG_Exp
      Prometheus -->|"HTTP scrape · :8000/metrics"| Django
      Grafana -->|"HTTP query · :9090"| Prometheus

```