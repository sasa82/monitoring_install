## monitoring_install

Production-ready monitoring stack based on Prometheus, Grafana and Alertmanager with exporters for ClickHouse, Kafka, PostgreSQL and Redis.

### Stack

| Component | Image | Port |
|-----------|-------|------|
| Prometheus | prom/prometheus | 9090 |
| Grafana | grafana/grafana | 3000 |
| Alertmanager | prom/alertmanager | 9094 |
| Node Exporter | prom/node-exporter | 9100 |
| ClickHouse Exporter node1 | f1yegor/clickhouse-exporter | 9116 |
| ClickHouse Exporter node2 | f1yegor/clickhouse-exporter | 9117 |
| Kafka Exporter | danielqsj/kafka-exporter | 9308 |
| PostgreSQL Exporter | prometheuscommunity/postgres-exporter | 9187 |
| Redis Exporter | oliver006/redis_exporter | 9121 |

### Prerequisites

- Docker + Docker Compose
- Running infrastructure stack from [infrastructure_install](https://github.com/sasa82/infrastructure_install)
- Telegram bot token and chat ID for alerts

### Quick Start

#### 1. Clone the repo

```bash
git clone https://github.com/sasa82/monitoring_install.git
cd monitoring_install/docker
```

#### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env` with your values - set `DOCKER_BIND_IP` to your private network IP and update all host and password values accordingly.

#### 3. Configure Alertmanager

```bash
cp docker/alertmanager/alertmanager.yml.example docker/alertmanager/alertmanager.yml
```

Edit `alertmanager.yml` and set your Telegram bot token and chat ID.

## Creating a Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` and follow instructions to get your bot token
3. Add the bot to a group or start a private chat
4. Send `/start` to the bot
5. Get your chat ID:

```bash
curl -s "https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates" | python3 -m json.tool | grep -A 2 '"id"'
```

#### 4. Start the stack

```bash
docker compose up -d
docker compose ps
```

#### 5. Access Grafana

Open `http://<your-server-ip>:3000` in browser. Default credentials: `admin` / `changeme`

### Security

All services bind to the private network IP defined in `DOCKER_BIND_IP` except Grafana which is publicly accessible on port `3000`.

### Dashboards

| Dashboard | Description |
|-----------|-------------|
| Node Exporter | Host CPU, Memory, Disk, Network |
| ClickHouse | Queries, Memory, Merges, Parts, Replication |
| Kafka | Brokers, Topics, Partitions, Consumer Groups |
| PostgreSQL | Connections, Queries, Cache, Replication |
| Redis | Memory, Clients, Commands, Keys, Network |

### Alert Rules

| File | Alerts |
|------|--------|
| node.yml | CPU, Memory, Disk, Load, Node Down |
| clickhouse.yml | Down, Memory, Delayed Inserts, Parts, Replication |
| kafka.yml | Down, Broker Down, Under Replicated, Consumer Lag |
| postgresql.yml | Down, Connections, Deadlocks, Slow Queries, Cache |
| redis.yml | Down, Memory, Connections, Rejections, Evictions |

Alerts are routed to Telegram with two severity levels - `warning` and `critical`.

### Directory Structure

```
monitoring_install/
└── docker/
    ├── docker-compose.yml
    ├── .env.example
    ├── prometheus/
    │   ├── prometheus.yml
    │   └── alerts/
    │       ├── node.yml
    │       ├── clickhouse.yml
    │       ├── kafka.yml
    │       ├── postgresql.yml
    │       └── redis.yml
    ├── grafana/
    │   ├── datasources/
    │   └── dashboards/
    │       ├── dashboard.yml
    │       └── json/
    │           ├── node-exporter.json
    │           ├── clickhouse.json
    │           ├── kafka.json
    │           ├── postgresql.json
    │           └── redis.json
    └── alertmanager/
        ├── alertmanager.yml.example
        └── alertmanager.yml
```

### Related Repositories

- [infrastructure_install](https://github.com/sasa82/infrastructure_install) - Ansible roles and Docker Compose for ClickHouse, Kafka, PostgreSQL, Redis
- [terraform](https://github.com/sasa82/terraform) - Cloud infrastructure provisioning for Hetzner and AWS
