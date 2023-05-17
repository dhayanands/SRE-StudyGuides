
# Alertmanager Installation & Configuration

- [Alertmanager Installation & Configuration](#alertmanager-installation--configuration)
  - [Installation](#installation)
  - [Configuration](#configuration)

## Installation

- Download the tar file from [link](https://prometheus.io/download/#alertmanager)
- Extract the tarball under `/prom`
- create softlink `/prom/alertmanager` to the extracted file
- create softlink to the  `/prom/alertmanager`  under `/usr/local/bin`
- create service file `/etc/systemd/system/alertmanager.service`

```bash
[Unit]
Description=Prometheus Alert Manager Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager/alertmanager \
        --config.file=/prom/config/alertmanager.yml \
        --log.level=debug \
        --web.route-prefix=/alertmanager \
        --web.external-url=http://localhost:9093/alertmanager
[Install]
WantedBy=multi-user.target
```

## Configuration

- Update the Alertmanager configuration `/prom/config/alertmanager.yml`

```bash
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 24h
  receiver: 'email'
receivers:
#  - name: 'web.hook'
#    webhook_configs:
#      - url: 'http://127.0.0.1:5001/'
  - name: 'email'
    email_configs:
    - send_resolved: true
      to: 'dhayanand@email.com
      from: 'alert@server.com'
      smarthost: 'smtp.server.com'
      auth_username: 'XXXXXXXXXX'
      auth_password: 'XXXXXXXXXXXXX'
      auth_identity: 'XXXXXXXXXX'
      auth_secret: 'XXXXXXXX'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
