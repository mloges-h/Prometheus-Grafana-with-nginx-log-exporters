# 🚀 Centralized Prometheus + Grafana Monitoring for NGINX (with NGINX Log Exporter & Alertmanager)

This project implements a **centralized monitoring system** using:

* **Prometheus**
* **Grafana**
* **Alertmanager**
* **NGINX Prometheus Exporter (stub_status)**
* **NGINX Log Exporter (access logs)**

It provides real-time dashboards, alerting, and complete NGINX observability for production environments.

---

# 📌 Features

### ✅ **1. Centralized Prometheus Server**

* Scrapes metrics from:

  * NGINX stub_status exporter (port `9113`)
  * NGINX log exporter (port `9913`)
* Integrated with Alertmanager
* Custom alert rules

### ✅ **2. Grafana Dashboards**

* Ready Grafana datasource for Prometheus
* Dashboards for:

  * Request rate
  * Response code breakdown (2xx/4xx/5xx)
  * Live traffic
  * Error spikes
  * Upstream failures

### ✅ **3. Alertmanager Integration**

* Email-based alerting
* Custom alert rule groups
* Alerts for:

  * No 200 OK responses
  * High 400/500 errors
  * Exporter down
  * Upstream failures
  * NGINX down

### ✅ **4. NGINX Log Exporter (access.log monitoring)**

* Parses `/var/log/nginx/access.log`
* Provides accurate:

  * status code counts
  * method counts
  * latency
  * request paths

### ✅ **5. NGINX Prometheus Exporter**

* Uses `/status` endpoint
* Collects live NGINX traffic stats

---

# 📡 Architecture Diagram

```
NGINX Servers
 ├── Stub Status Exporter (9113)
 ├── Nginx Log Exporter (9913)
 └── Access Logs
        ↓
Prometheus (Central Server)
 │
 ├── Alertmanager (Email Alerts)
 └── Grafana Dashboards
```

---

# 📁 Project Structure (Recommended)

```
.
├── prometheus/
│   ├── prometheus.yml
│   ├── alert-rules.yml
│   └── ...
├── alertmanager/
│   └── alertmanager.yml
├── exporters/
│   ├── nginx-exporter-docker-command.txt
│   ├── nginxlog.yaml
│   └── setup-notes.md
├── grafana/
│   ├── grafana.ini
│   └── dashboard.json (optional)
└── README.md
```

---

# 🔧 Prometheus Configuration (Summary)

### `prometheus.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093

rule_files:
  - "/opt/prometheus-3.5.0.linux-amd64/alert-rules.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
        labels:
          app: "prometheus"

  - job_name: "nginx_monitoring"
    static_configs:
      - targets: ["13.204.43.210:9113"]
        labels:
          instance: nginx-primary-server
```

---

# 🚨 Alert Rules (Summary)

### `alert-rules.yml`

Alerts included:

* No 200 responses
* High 400 errors
* High 500 errors
* Upstream 500 errors
* Exporter down
* NGINX down

Example:

```yaml
- alert: NginxHigh500Errors
  expr: rate(nginx_http_requests_total{status="500"}[2m]) > 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "NGINX 500 errors detected"
    description: "Application is returning 500 (server errors)."
```

---

# 📬 Alertmanager Configuration

### `alertmanager.yml`

Email alerts:

```yaml
receivers:
  - name: 'email-alert'
    email_configs:
      - to: 'logesh@sixthstar.in'
        from: 'logesh@sixthstar.in'
        smarthost: 'star.sixthstar.in:587'
        auth_username: 'logesh@sixthstar.in'
        auth_password: '********'
```

---

# 📈 Grafana Setup

Enable SMTP in:

`/etc/grafana/grafana.ini`

```ini
[smtp]
enabled = true
host = star.sixthstar.in:587
user = logesh@sixthstar.in
password = "********"
from_address = logesh@sixthstar.in
from_name = Grafana Alerts
startTLS_policy = OpportunisticStartTLS
```

Restart:

```
systemctl restart grafana-server
```

Grafana URLs:

* Prometheus Query Explorer → `https://test-grafana.twixor.com/prometheus/query`
* Grafana Dashboard → `https://test-grafana.twixor.com`

---

# 🟢 NGINX Status Exporter (stub_status)

Enable status endpoint:

```
server {
    listen 8080;

    location /status {
        stub_status;
    }
}
```

Reload NGINX:

```
nginx -t
systemctl reload nginx
```

Run exporter in Docker:

```bash
docker run -d \
  --name nginx-exporter-local \
  --network=host \
  nginx/nginx-prometheus-exporter:1.5.1 \
  --nginx.scrape-uri=http://127.0.0.1:8080/status
```

---

# 📘 NGINX Log Exporter

Config:

```yaml
listen:
  port: 9913
  address: 0.0.0.0

namespaces:
  - name: nginx
    format: "$remote_addr - $remote_user [$time_local] \"$request\" $status $body_bytes_sent \"$http_referer\" \"$http_user_agent\""
    source_files:
      - /var/log/nginx/access.log
```

Run:

```
/opt/prometheus-nginxlog-exporter -config-file /opt/nginxlog.yaml &
```

---

# 🧪 Testing Error Alerts

## Generate repeated 500 errors:

```bash
end=$((SECONDS+120))
while [ $SECONDS -lt $end ]; do
  curl -s http://<server-ip>:8081/error500 > /dev/null
  sleep 0.5
done
```

Check logs exporter:

```
curl http://localhost:9913/metrics | grep status
```


# 🧩 Conclusion

This project provides a **complete production-ready monitoring setup** for NGINX, combining:

* Prometheus
* Alertmanager
* Grafana
* Exporters (status + access.log)
* Email alerts
* Dashboards
