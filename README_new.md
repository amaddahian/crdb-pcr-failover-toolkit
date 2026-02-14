# CockroachDB Physical Cluster Replication (PCR) Toolkit

A comprehensive bash toolkit for managing CockroachDB Physical Cluster Replication (PCR) across two clusters for disaster recovery, testing, and demonstrations.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Menu Structure](#menu-structure)
- [Configuration](#configuration)
- [Common Operations](#common-operations)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Advanced Usage](#advanced-usage)

---

## 🎯 Overview

This toolkit automates the deployment and management of two CockroachDB clusters configured for physical cluster replication (PCR). It provides:

- **Cluster A (CA)** - Primary cluster with tenant `va`
- **Cluster B (CB)** - Standby cluster with tenant `vb`
- **Bidirectional Replication** - A→B and B→A failover scenarios
- **Monitoring Stack** - Prometheus + Grafana with multi-tenant metrics
- **Integration Support** - Kafka CDC, Ceph/S3 backup storage
- **Version Management** - Rolling upgrades, downgrades, and rollbacks

### Architecture Diagram

![CRDB Replication & CDC Architecture](Gemini_Generated_Image_k5wmg7k5wmg7k5wm.png)

This toolkit implements the architecture shown above:
- **PCR Replication Stream** - Physical replication between Cluster A and Cluster B
- **Bidirectional Data Flow** - Support for both A→B and B→A replication
- **CDC Changefeed** - Stream data changes to Kafka topics
- **External Storage** - Backup integration with Ceph/S3-compatible storage

---

## ✨ Features

### Core Capabilities
- ✅ **Automated Cluster Creation** - Create 3-node clusters with customizable versions
- ✅ **Physical Cluster Replication** - Set up bidirectional replication between clusters
- ✅ **Failover & Failback** - Test disaster recovery scenarios (A→B, B→A)
- ✅ **Load Generation** - MOVR and TPC-C workloads for testing
- ✅ **Rolling Upgrades** - Patch and major version upgrades with finalization
- ✅ **HAProxy Load Balancing** - Optional load balancers for each cluster

### Monitoring & Observability
- 📊 **Prometheus + Grafana** - Pre-configured monitoring stack
- 🎯 **Multi-Tenant Metrics** - Scrape metrics from system and application tenants
- 📈 **CockroachDB Dashboards** - Auto-imported official Grafana dashboards
- 📝 **SQL Audit Logging** - All SQL statements logged with timestamps

### Integrations
- 🔄 **Kafka CDC** - Change Data Capture integration
- 💾 **Ceph/S3 Storage** - S3-compatible backup storage
- 🐛 **Debug Collection** - Automated debug artifact collection (logs, tsdump)

---

## 📦 Requirements

### System Requirements
- **OS**: Linux (Ubuntu/RHEL/CentOS) or macOS
- **Memory**: 8GB+ RAM recommended
- **Disk**: 20GB+ free space
- **Network**: Open ports for cluster communication

### Software Dependencies

#### Required
- **Docker** (20.10+) or **Podman** (3.0+)
- **Bash** (4.0+)
- **curl**
- **jq** (for Grafana dashboard import)

#### Optional
- **docker-compose** or **podman-compose** (for monitoring stack)
- **promtool** (for Prometheus config validation)
- **timeout** command (for SQL query timeout protection)

### Installation

```bash
# Install Docker (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y docker.io docker-compose jq

# Install Docker (macOS)
brew install docker docker-compose jq

# Verify installations
docker --version
docker-compose --version
jq --version
```

---

## 🚀 Quick Start

### 1. Run the Script

```bash
# Make executable
chmod +x pcr_021426.sh

# Launch menu
./pcr_021426.sh menu
```

### 2. Create Both Clusters (Quick Start Option q2)

```
Select [q1-q4, 1-7, 0, q]: q2
Number of nodes per cluster (default 3): 3
Image version (e.g., 24.1.17) or empty for default: 24.3.0
Create both CA and CB with n=3, version='24.3.0'? [y/N]: y
```

### 3. Run Full Demo (Quick Start Option q1)

This runs the complete A→B→A replication cycle:
1. Load data on Cluster A
2. Start replication A→B
3. Failover A→B
4. Load data on Cluster B
5. Start replication B→A
6. Failover B→A
7. Restart replication A→B

```
Select [q1-q4, 1-7, 0, q]: q1
```

### 4. Access Monitoring (Quick Start Option q3)

```
Select [q1-q4, 1-7, 0, q]: q3
  1) Setup monitoring stack
  2) Start monitoring stack
  ...
```

Then access:
- **Prometheus**: http://localhost:8095
- **Grafana**: http://localhost:8096 (admin/admin)
- **CockroachDB Console**: http://localhost:8080 (Cluster A)

---

## 📊 Menu Structure

### Main Menu

```
================================================================
    CockroachDB Physical Cluster Replication (PCR) Menu
================================================================

QUICK START (Common Operations)
  q1) Full Demo (automated A→B→A cycle)
  q2) Create Both Clusters (CA + CB)
  q3) Start Monitoring (Prometheus + Grafana)
  q4) DB Console (Web UI)

MAIN MENU
  1) Cluster Management
  2) Data Operations
  3) Replication
  4) Version Management
  5) Monitoring & Debug
  6) Integrations
  7) Settings

  0) Cleanup
  q) Quit
```

### 1. Cluster Management
- Create Cluster A (CA)
- Create Cluster B (CB)
- Create Both Clusters
- Cluster Status & Health
- DB Console (Web UI)
- Cleanup

### 2. Data Operations
- Load Data (MOVR/TPCC workloads)
- Check Row Counts
- Run SQL (Interactive)

### 3. Replication
- Guided Demo (Full automated A→B→A cycle)
- Start Replication (A→B, B→A, Custom)
- Failover (A→B, B→A)
- Restart Replication
- Check Replication Health

### 4. Version Management
- Rolling PATCH upgrade/downgrade
- Rolling MAJOR upgrade
- Finalize major upgrade
- Rollback (pre-finalization)
- Verify post-finalization

### 5. Monitoring & Debug
- Prometheus + Grafana
- Collect Debug Artifacts
- Settings

### 6. Integrations
- Kafka CDC Integration
- Ceph/S3 Storage Setup
- Show Ceph Backup URI Helper

### 7. Settings
- Configure load balancers
- Set cluster parameters
- License configuration

---

## ⚙️ Configuration

### Environment Variables

```bash
# Container Runtime
export RUNTIME=docker          # or 'podman'

# CockroachDB Version
export CRDB_VERSION=24.3.0     # Default cluster version

# Monitoring
export GRAFANA_ADMIN_PASSWORD=mySecurePassword  # Grafana password

# SQL Execution
export SQL_TIMEOUT=300         # SQL query timeout (seconds)

# Logging
export PCR_LOG_DIR=$HOME       # SQL audit log directory

# Display
export PCR_HEADING_STYLE=rule  # Heading style: rule|ascii|box|thick
```

### Network Configuration

The script creates a single Docker network `roachnet1` with the following port mappings:

#### Cluster A (CA)
- **SQL Ports**: 26257 (roach1), 27257 (roach2), 28257 (roach3)
- **HTTP Ports**: 8080 (roach1), 8081 (roach2), 8082 (roach3)
- **Load Balancer**: 26357 (if enabled)

#### Cluster B (CB)
- **SQL Ports**: 26257 (roach4), 27257 (roach5), 28257 (roach6)
- **HTTP Ports**: 8090 (roach4), 8091 (roach5), 8092 (roach6)
- **Load Balancer**: 27357 (if enabled)

#### Monitoring
- **Prometheus**: 8095
- **Grafana**: 8096

---

## 🎮 Common Operations

### Create Clusters

```bash
# Via menu: Main Menu → 1 (Cluster Management) → 3 (Create Both Clusters)
# Or Quick Start: q2

# Specify version
Image version: 24.3.0

# Creates:
# - Cluster A: roach1, roach2, roach3 (tenant: va)
# - Cluster B: roach4, roach5, roach6 (tenant: vb)
```

### Start Replication A→B

```bash
# Via menu: Main Menu → 3 (Replication) → 2 (Start Replication) → 1 (A→B)

# This will:
# 1. Create tenant 'vb-readonly' on Cluster B
# 2. Start physical replication stream from va to vb-readonly
# 3. Monitor replication lag
```

### Failover A→B

```bash
# Via menu: Main Menu → 3 (Replication) → 3 (Failover) → 1 (A→B)

# This will:
# 1. Complete replication cutover
# 2. Promote vb-readonly to vb (read-write)
# 3. Verify data consistency
```

### Run SQL Queries

```bash
# Via menu: Main Menu → 2 (Data Operations) → 3 (Run SQL)

# Choose cluster: CA or CB
# Choose tenant: system, va, vb, etc.
# Enter SQL commands interactively
```

### Load Test Data

```bash
# Via menu: Main Menu → 2 (Data Operations) → 1 (Load Data)

# Options:
# 1) MOVR workload (initial load)
# 2) MOVR workload (additional 10s)
# 3) TPCC workload (fixtures import)
```

### Perform Rolling Upgrade

```bash
# Via menu: Main Menu → 4 (Version Management)

# Steps:
# 1) Rolling PATCH upgrade - Minor version change (e.g., 24.3.0 → 24.3.1)
# 2) Rolling MAJOR upgrade - Major version change (e.g., 24.1 → 24.3)
# 3) Finalize major upgrade - Commit to new version
# 4) Rollback (if needed before finalization)
```

---

## 📊 Monitoring

### Setup Prometheus + Grafana

```bash
# Via menu: Quick Start → q3 (Start Monitoring)

# Steps:
1. Setup monitoring stack
2. Start monitoring stack

# Access:
- Prometheus: http://localhost:8095
- Grafana: http://localhost:8096
  - Username: admin
  - Password: admin (or GRAFANA_ADMIN_PASSWORD)
```

### Pre-configured Dashboards

The following CockroachDB dashboards are automatically imported:

1. **Runtime Metrics** - Live nodes, memory, CPU, goroutines
2. **Storage Availability** - Disk usage, capacity, storage operations
3. **SQL Queries & Transactions** - Query latency, QPS, connections
4. **Replication & Replicas** - Replica counts, range operations

### Multi-Tenant Metrics

Prometheus scrapes metrics from all tenants using the `?cluster=system` parameter:

- **System tenant** - Node-level metrics, cluster health
- **Application tenants** - va, vb, vb-readonly specific metrics

Filter by tenant in Grafana:
```promql
# System tenant only
sql_conns{cluster="cluster-a", tenant="system"}

# Application tenant only
sql_conns{cluster="cluster-a", tenant="va"}

# All tenants
sql_conns{cluster="cluster-a"}
```

### SQL Audit Logging

All SQL statements are automatically logged to:
```
$HOME/sql_audit/c54_sql_<timestamp>.csv
```

Format:
```csv
timestamp,cluster,version,sql
"2026-02-14T10:30:45-0800","CA","24.3.0","SELECT version();"
```

---

## 🔧 Troubleshooting

### Containers Not Starting

```bash
# Check container status
docker ps -a

# Check logs
docker logs roach1
docker logs prometheus
docker logs grafana

# Check network
docker network inspect roachnet1
```

### Grafana Shows No Data

```bash
# Verify Prometheus has targets
curl http://localhost:8095/api/v1/targets

# Check tenant labels
curl http://localhost:8095/api/v1/label/tenant/values

# Verify metrics exist
curl 'http://localhost:8095/api/v1/query?query=sql_conns'

# Restart monitoring stack
# Menu → 5 (Monitoring & Debug) → 1 (Prometheus + Grafana) → 3 (Stop) → 2 (Start)
```

### Replication Not Working

```bash
# Check replication status
# Menu → 3 (Replication) → 5 (Check Replication Health)

# Verify tenant exists
docker exec -it roach4 ./cockroach sql --certs-dir=/certs \
  --execute "SHOW VIRTUAL CLUSTERS;"

# Check replication stream
docker exec -it roach4 ./cockroach sql --certs-dir=/certs \
  --execute "SELECT * FROM [SHOW VIRTUAL CLUSTER vb-readonly WITH REPLICATION STATUS];"
```

### Heading Display Issues (Ubuntu)

If you see broken box characters:

```bash
# Set heading style to simple ASCII
export PCR_HEADING_STYLE=rule
./pcr_021426.sh menu

# Or use ASCII boxes
export PCR_HEADING_STYLE=ascii
```

### Port Already in Use

```bash
# Find process using port
lsof -i :8080
# or
netstat -tulpn | grep 8080

# Change ports in script or stop conflicting service
```

### Permission Denied Errors

```bash
# Docker permission (Linux)
sudo usermod -aG docker $USER
newgrp docker

# Or run with sudo
sudo ./pcr_021426.sh menu
```

---

## 🚀 Advanced Usage

### Custom Cluster Sizes

```bash
# Create 5-node clusters
# Menu → 1 (Cluster Management) → 1 (Create Cluster A)
Number of nodes for CA (default 3): 5
```

### Custom Replication Scenarios

```bash
# Menu → 3 (Replication) → 6 (Custom Replication)

# Choose source/destination clusters and tenants
Source cluster: CA
Source tenant: va
Destination cluster: CB
Destination tenant: vb-custom
```

### HAProxy Load Balancing

```bash
# Menu → 7 (Settings) → Enable load balancers

# Access via load balancer
psql "postgresql://root@localhost:26357/defaultdb?sslmode=require&options=-ccluster=va"
```

### Backup to Ceph/S3

```bash
# Menu → 6 (Integrations) → 2 (Ceph/S3 Storage Setup)

# Then backup
BACKUP DATABASE movr INTO 's3://backup-bucket/cluster-a?AUTH=specified&AWS_ACCESS_KEY_ID=xxx&AWS_SECRET_ACCESS_KEY=xxx';
```

### Kafka CDC Integration

```bash
# Menu → 6 (Integrations) → 1 (Kafka CDC Integration)

# Create changefeed
CREATE CHANGEFEED FOR TABLE movr.users
  INTO 'kafka://kafka:9092'
  WITH updated, resolved;
```

### Custom Heading Styles

```bash
# Set before running
export PCR_HEADING_STYLE=thick   # Fancy box drawing (macOS)
export PCR_HEADING_STYLE=ascii   # ASCII boxes
export PCR_HEADING_STYLE=rule    # Simple rules (default, Ubuntu-friendly)

./pcr_021426.sh menu
```

### Dry-Run Mode

```bash
# Test commands without executing
export DRY_RUN=1
./pcr_021426.sh menu
```

---

## 📝 File Locations

```
$HOME/
├── sql_audit/              # SQL audit logs
│   └── c54_sql_*.csv
├── monitoring/             # Prometheus + Grafana
│   ├── docker-compose.yml
│   ├── prometheus/
│   │   └── prometheus.yml
│   └── grafana/
│       └── provisioning/
└── pcr_021426.sh          # Main script
```

---

## 🤝 Contributing

This is a demo/testing toolkit. For production deployments, please refer to official CockroachDB documentation:
- [CockroachDB Docs](https://www.cockroachlabs.com/docs/)
- [Physical Cluster Replication](https://www.cockroachlabs.com/docs/stable/physical-cluster-replication-overview.html)

---

## 📄 License

Refer to your organization's licensing terms for CockroachDB usage.

---

## 🆘 Support

For issues with this toolkit:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review logs: `docker logs <container_name>`
3. Check SQL audit logs: `$HOME/sql_audit/c54_sql_*.csv`

For CockroachDB issues:
- [CockroachDB Community](https://www.cockroachlabs.com/community/)
- [CockroachDB Support](https://support.cockroachlabs.com/)

---

## 🎯 Quick Reference

### Essential Commands

```bash
# Start the menu
./pcr_021426.sh menu

# Quick demo (full A→B→A cycle)
# Select: q1

# Create both clusters
# Select: q2

# Start monitoring
# Select: q3

# Access DB Console
# Select: q4

# Cleanup everything
# Select: 0 → 6 (FULL RESET)
```

### Important URLs

- **Cluster A Console**: http://localhost:8080
- **Cluster B Console**: http://localhost:8090
- **Prometheus**: http://localhost:8095
- **Grafana**: http://localhost:8096
- **SQL (Cluster A)**: postgresql://root@localhost:26257
- **SQL (Cluster B)**: postgresql://root@localhost:26257

---

**Version**: 2024-01-26
**Script**: pcr_021426.sh
**Last Updated**: February 2026
