# Prometheus Installation and Configuration

- [Prometheus Installation and Configuration](#prometheus-installation-and-configuration)
  - [Installation](#installation)
  - [Configuration](#configuration)
  - [Validate Config](#validate-config)
  - [service discovery](#service-discovery)

## Installation

- Download the tar file from [link](https://prometheus.io/download/#prometheus)
- Extract the tarball under `/prom`
- create softlink `/prom/prometheus` to the extracted file
- create softlink to the  `/prom/prometheus`  under `/usr/local/bin`
- create service file `/etc/systemd/system/prometheus.service`

```bash
[Unit]
Description=Prometheus Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/prometheus/prometheus \
        --config.file=/prom/config/prometheus.yml
          # enables admin api for power user requests -- clear the DB ## curl -X POST http://localhost:9090/api/v1/admin/tsdb/clean_tombstones
        --web.enable-admin-api \
          # enables prometheus to reload the config with an http request without restart of service --  ## curl -X POST http://localhost:9090/-/reload
        --web.enable-lifecycle
[Install]
WantedBy=multi-user.target
```

- start the service

## Configuration

- edit `/prom/config/prometheus.yml`

```yml

# "global" section has settings which apply to all targets
global:
  scrape_interval: 30s

# "scrape_configs" section defines the actual targets which are grouped together into a named "jobs"
scrape_configs:
  - job_name: 'prometheus'
    # "static_configs" are fixed list of targets
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'dynamic-servers'
    # "file_sd_configs" for having dynamic list of servers using service discovery
    file_sd_configs:
      - files:
        - 'serverlist.json'

- job_name: 'web'
    metrics_path: /metrics
    scheme: http # can include TLS and auth for HTTPS 
    static_configs:
      - targets: ['webserver:8080']
```

```json
// serverlist.json file used for service discovery

[
    {
        "targets": [ "server1:9001" ]
    },
    {
        "targets": [ "server2:9001" ]
    },
]

```

## Validate Config

```bash
$prometheus_dir/promtool check config prometheus.yml
```

## service discovery

- DNS servive discovery
- Kubernetes containers based on labels
- Azure, GCP etc., have their own ways
