# eth-validator-aws
## Overview
This project demonstrates how to deploy an Ethereum validator node (Goerli testnet) on AWS EC2 using Docker.

### Architecture
- AWS EC2 (Ubuntu)
- Geth running in Docker
- Ports: 30303 (P2P), 8545 (RPC)

## How to Run

1. Provision EC2 (Ubuntu 22.04)
2. Install Docker: `scripts/install-docker.sh`
3. Start Geth node: `docker-compose up -d`
4. Verify sync with: `docker logs -f geth`

axjkj
C:\Users\aditya.singh1\OneDrive - Comviva Technologies LTD\Desktop\eth-validator-aws\docker\docker-compose.yml



✅ Step-by-Step Monitoring Setup
🔹 Step 1: Expose Geth Prometheus Metrics
Edit your docker-compose.yml to include this flag under Geth:

yaml
Copy code
command:
  - --goerli
  - --http
  - --http.addr=0.0.0.0
  - --http.api=eth,net,web3
  - --syncmode=snap
  - --metrics
  - --metrics.expensive
  - --pprof
  - --pprof.addr=0.0.0.0
⚠️ These will expose metrics at:
http://<your-ec2-ip>:6060/debug/metrics/prometheus

Also, add port 6060 to your compose:

yaml
Copy code
ports:
  - "6060:6060"
Then apply changes:

bash
Copy code
docker compose down
docker compose up -d
Confirm:

bash
Copy code
curl http://localhost:6060/debug/metrics/prometheus
🔹 Step 2: Add Prometheus + Node Exporter via Docker Compose
Inside your docker/ directory, create monitoring.yml:

yaml
Copy code
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  node_exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"
Create the file prometheus.yml next to it:

yaml
Copy code
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'geth'
    static_configs:
      - targets: ['host.docker.internal:6060']  # or EC2 IP if on different host

  - job_name: 'node'
    static_configs:
      - targets: ['host.docker.internal:9100']  # or EC2 IP
🔹 Step 3: Add Grafana
Append Grafana to the same monitoring.yml:

yaml
Copy code
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana

volumes:
  grafana-storage:
Then start everything:

bash
Copy code
docker compose -f monitoring.yml up -d
Access:

Prometheus: http://<ec2-ip>:9090

Grafana: http://<ec2-ip>:3000 (admin / admin)

🔹 Step 4: Import Ethereum Dashboard into Grafana
Once inside Grafana:

Add Prometheus as a data source (URL: http://prometheus:9090)

Import public dashboard ID: 13502 (Ethereum Geth Node Overview)

Start watching CPU, peers, sync status, blocks

📁 Directory Structure to Push to GitHub
bash
Copy code
eth-validator-aws/
├── docker/
│   ├── docker-compose.yml         # Geth setup
│   ├── monitoring.yml             # Prometheus, Grafana, node_exporter
│   ├── prometheus.yml             # Prometheus scrape config
│   └── README.md                  # Monitoring section