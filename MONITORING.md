# Monitoring & Logging Stack

## Overview
Complete observability stack for Quiz-O-Meter application.

## Components

### 1. Prometheus
- **Purpose:** Metrics collection and alerting
- **Access:** http://your-server-ip:32002
- **What it monitors:**
  - Pod CPU/Memory usage
  - Container restarts
  - Network traffic
  - Custom application metrics

### 2. Grafana
- **Purpose:** Visualization and dashboards
- **Access:** http://your-server-ip:32001
- **Credentials:**
  - Username: `admin`
  - Password: `admin123` (change this!)
- **Pre-configured dashboards:**
  - Kubernetes cluster overview
  - Node metrics
  - Pod metrics
  - Quiz-O-Meter application dashboard

### 3. Loki
- **Purpose:** Log aggregation
- **Access:** Via Grafana (datasource)
- **What it collects:**
  - All pod logs
  - Container stdout/stderr
  - Application logs

### 4. Promtail
- **Purpose:** Log collection agent
- **Runs on:** Every node (DaemonSet)
- **Function:** Scrapes logs and sends to Loki

### 5. AlertManager
- **Purpose:** Alert routing and notifications
- **Access:** Part of Prometheus stack
- **Configured alerts:**
  - Backend/Frontend down
  - High CPU/Memory usage
  - Frequent pod restarts
  - Redis down

## Access URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| Grafana | http://your-server-ip:32001 | admin / admin123 |
| Prometheus | http://your-server-ip:32002 | No auth |
| AlertManager | http://your-server-ip:32003 | No auth |

**Or via Cloudflare Tunnel:**
- https://grafana.your-domain.com
- https://prometheus.your-domain.com

## Using Grafana

### Access Dashboards
1. Login to Grafana
2. Navigate: Dashboards → Browse
3. Pre-installed dashboards:
   - **Kubernetes / Compute Resources / Cluster** - Overall cluster health
   - **Kubernetes / Compute Resources / Namespace (Pods)** - Per-namespace view
   - **Quiz-O-Meter Application** - Custom dashboard

### View Logs
1. Navigate: Explore (compass icon)
2. Select datasource: **Loki**
3. Query examples:

All logs from quiz-dev namespace
{namespace="quiz-dev"}

Backend logs only
{namespace="quiz-dev", pod=~".backend."}

Error logs
{namespace="quiz-dev"} |= "error"

Last 5 minutes of frontend logs
{namespace="quiz-dev", pod=~".frontend."} [5m]


### Create Alerts
1. Navigate to dashboard
2. Edit panel → Alert tab
3. Define conditions
4. Save

## Common Queries

### Prometheus Queries (PromQL)

#### CPU Usage

#### CPU usage by pod
sum(rate(container_cpu_usage_seconds_total{namespace="quiz-dev"}[5m])) by (pod)

#### CPU percentage
sum(rate(container_cpu_usage_seconds_total{namespace="quiz-dev"}[5m])) by (pod) /
sum(container_spec_cpu_quota{namespace="quiz-dev"}) by (pod) * 100


#### Memory Usage
#### Memory usage by pod
sum(container_memory_working_set_bytes{namespace="quiz-dev"}) by (pod)

#### Memory percentage
sum(container_memory_working_set_bytes{namespace="quiz-dev"}) by (pod) /
sum(container_spec_memory_limit_bytes{namespace="quiz-dev"}) by (pod) * 100



#### Network
#### Network receive rate
sum(rate(container_network_receive_bytes_total{namespace="quiz-dev"}[5m])) by (pod)

#### Network transmit rate
sum(rate(container_network_transmit_bytes_total{namespace="quiz-dev"}[5m])) by (pod)


#### Pod Status

Running pods
sum(kube_pod_status_phase{namespace="quiz-dev", phase="Running"})

Pod restarts
sum(kube_pod_container_status_restarts_total{namespace="quiz-dev"}) by (pod)

text

### Loki Queries (LogQL)

#### All logs from dev environment
{namespace="quiz-dev"}

#### Backend errors
{namespace="quiz-dev", pod=~".backend."} |= "error"

#### Frontend 404 errors
{namespace="quiz-dev", pod=~".frontend."} |= "404"

#### Logs containing specific text
{namespace="quiz-dev"} |~ "database connection"

#### Count errors per minute
sum(rate({namespace="quiz-dev"} |= "error" [1m])) by (pod)



## Alerts Configuration

### Current Alerts
1. **QuizBackendDown** - Backend unavailable for 2+ minutes
2. **QuizFrontendDown** - Frontend unavailable for 2+ minutes
3. **HighCPUUsage** - CPU > 80% for 5+ minutes
4. **HighMemoryUsage** - Memory > 85% for 5+ minutes
5. **PodRestartingFrequently** - Restarts in last 15 minutes
6. **RedisDown** - Redis unavailable for 1+ minute

### View Active Alerts
1. Go to Prometheus UI: http://your-server-ip:32002
2. Navigate: Alerts tab
3. See firing/pending alerts

## Troubleshooting

### Grafana Not Showing Data
Check if Prometheus is running
kubectl get pods -n monitoring | grep prometheus

Check datasource connection in Grafana
Settings → Data Sources → Prometheus → Test


### Loki Logs Not Appearing

#### Check Promtail is running
kubectl get pods -n monitoring | grep promtail

#### Check Loki datasource in Grafana
Settings → Data Sources → Loki → Test


### High Resource Usage
#### Check Prometheus storage
kubectl get pvc -n monitoring

#### Reduce retention period (default 30d)
Edit Prometheus config to 15d or 7d


## Maintenance

### Backup Grafana Dashboards
#### Export all dashboards
kubectl exec -n monitoring monitoring-grafana-xxxxx --
sqlite3 /var/lib/grafana/grafana.db .dump > grafana-backup.sql



### Clear Old Metrics
Prometheus automatically deletes old data based on retention period (30 days).

### Update Monitoring Stack
helm repo update
helm upgrade monitoring prometheus-community/kube-prometheus-stack -n monitoring