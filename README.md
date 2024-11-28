# Prometheus, Node Exporter, and Grafana Setup

This guide will help you install and configure Prometheus, Node Exporter, and Grafana to monitor your system metrics.

## Prerequisites

- SLES15 system with root privileges.

## Installation

### Step 1: Install Prometheus, Node Exporter, and Grafana

Install the required packages using `zypper`:

```sh
sudo zypper install golang-github-prometheus-prometheus golang-github-prometheus-node_exporter grafana
```

### Step 2: Configure Prometheus

Create or edit the `prometheus.yml` file with the following content:

```yaml
global:
    # Scrape faster than default, eventually
    # scrape_interval: 1s

scrape_configs:
    - job_name: 'prometheus'
        static_configs:
            - targets: ['localhost:9090']

    - job_name: 'node-exporter'
        static_configs:
            - targets: ['localhost:9100']
```

Move the configuration file to the appropriate directory:

```sh
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```
#### (Optional) ####

By default, a data rentention period of 15 days is set: it can be tuned modifying the content of /etc/sysconfig/prometheus as follows:

```sh
ARGS="--storage.tsdb.retention.time=30d"
```

### Step 3: Start the Services

Enable and start Prometheus, Node Exporter, and Grafana services:

```sh
sudo systemctl enable prometheus
sudo systemctl start prometheus

sudo systemctl enable prometheus-node_exporter
sudo systemctl start prometheus-node_exporter

sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### Step 4: Adjust Firewall Settings

Ensure that the necessary ports are open:

```sh
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --permanent --add-port=9100/tcp
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

### Step 5: Access the Services

- Prometheus: [http://localhost:9090](http://localhost:9090)
- Node Exporter: [http://localhost:9100/metrics](http://localhost:9100/metrics)
- Grafana: [http://localhost:3000](http://localhost:3000) (default login: `admin`, password: `admin`)

### Step 6: Configure Grafana

1. Access Grafana from your browser

2. Add Prometheus as a data source:
    - Go to "Connections" -> "Your connections"
    - Click the *Add new data source* button
    - Select "Prometheus" and add the URL: `http://localhost:9090`
    - Click "Save & Test" to verify the connection.

2. Import the Node Exporter dashboard:
     - Go to "Dashboards" -> Select "Import" from the "New" button dropdown
     - Use the dashboard ID `1860` (Node Exporter Full)

## Conclusion

You have successfully set up Prometheus, Node Exporter, and Grafana to monitor your system metrics. You can now explore the Grafana dashboard to view various system metrics.

## Bonus track: import / export timeseries

I've only been able to achieve this as follows:

1. Stop prometheus on the source machine:
```sh
systemctl stop prometheus
```
2. Create a compress archive of its /var/lib/prometheus/metrics
```sh
tar czvf metrics.tar.gz -C /var/lib/prometheus/ metrics
```

3. Restart prometheus on the source machine
```sh
systemctl start prometheus
```

4. Transfer the archive to the target machine, in its /var/lib/prometheus directory

5. Stop prometheus on the target machine and wipe the /var/lib/prometheus directory
(!_all the data collected by prometheus will be lost: think of making a backup_!)
```sh
rm -rf /var/lib/prometheus/*
```

6. Extract the archive
```sh
tar xzvf metrics.tar.gz
```

7. Restart prometheus on the target machine