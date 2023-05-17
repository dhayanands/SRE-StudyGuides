# Node Exporter Installation & Configuration

- [Node Exporter Installation & Configuration](#node-exporter-installation--configuration)
  - [Installation](#installation)
  - [Configuration](#configuration)

## Installation

- Download the tar file from [link](https://prometheus.io/download/#node_exporter)
- Extract the tarball under `/prom`
- create softlink `/prom/node_exporter` to the extracted file
- create softlink to the  `/prom/node_exporter`  under `/usr/local/bin`

## Configuration

- Create the environment file `/etc/sysconfig/node_exporter` with the below content

```bash
OPTIONS="--collector.textfile.directory /var/lib/node_exporter/textfile_collector --collector.systemd"
```

- create service file `/etc/systemd/system/node_exporter.service`

```bash
[Unit]
Description=Node Exporter

[Service]
User=node_exporter
EnvironmentFile=/etc/sysconfig/node_exporter
ExecStart=/usr/local/bin/node_exporter/node_exporter $OPTIONS

[Install]
WantedBy=multi-user.target
```

- start the service
