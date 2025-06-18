# Monitoring Stack Ansible Deployment

This repo contain ansible code for deploying Prometheus and Grafana in prepared VMs.

## Manual Setups

### Install Grafana on Debian or Ubuntu

There are multiple ways to install Grafana: using the Grafana Labs APT repository, by downloading a .deb package, or by downloading a binary .tar.gz file. Choose only one of the methods below that best suits your needs.

> Note: If you install via the .deb package or .tar.gz file, then you must manually update Grafana for each new version.

Complete the following steps to install Grafana from the APT repository:

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget

sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Updates the list of available packages
sudo apt-get update

# Installs the latest OSS release:
sudo apt-get install grafana
```

### Uninstall Grafana on Debian or Ubuntu

To uninstall Grafana, run the following commands in a terminal window:

```bash
sudo systemctl stop grafana-server

sudo service grafana-server stop

sudo apt-get remove grafana

# Optional: To remove the Grafana repository:
sudo rm -i /etc/apt/sources.list.d/grafana.list
```

### Start the Grafana server

The following instructions start the grafana-server process as the grafana user, which was created during the package installation.

If you installed with the APT repository or .deb package, then you can start the server using systemd or init.d. If you installed a binary .tar.gz file, then you execute the binary.

#### Start the Grafana server with systemd

Complete the following steps to start the Grafana server using systemd and verify that it is running.

```bash
# To start the service, run the following commands:
sudo systemctl daemon-reload
sudo systemctl start grafana-server

# To verify that the service is running, run the following command:
sudo systemctl status grafana-server

# To configure the Grafana server to start at boot, run the following command:
sudo systemctl enable grafana-server.service
```

> **Serve Grafana on a port < 1024:**
If you are using systemd and want to start Grafana on a port that is lower than 1024, you must add a systemd unit override.

Run the following command to create an override file in your configured editor.

```bash
# Alternatively, create a file in /etc/systemd/system/grafana-server.service.d/override.conf
sudo systemctl edit grafana-server.service
```

Add the following additional settings to grant the CAP_NET_BIND_SERVICE capability.

```ini
[Service]
# Give the CAP_NET_BIND_SERVICE capability
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# A private user cannot have process capabilities on the host's user
# namespace and thus CAP_NET_BIND_SERVICE has no effect.
PrivateUsers=false
```

#### Restart the Grafana server using systemd

To restart the Grafana server, run the following command:

```bash
sudo systemctl restart grafana-server
```

---

### Install Prometheus on Debian or Ubuntu

Prometheus is a monitoring platform that collects metrics from monitored targets by scraping metrics HTTP endpoints on these targets. This guide will show you how to install, configure and monitor our first resource with Prometheus. You'll download, install and run Prometheus. You'll also download and install an exporter, tools that expose time series data on hosts and services. Our first exporter will be Prometheus itself, which provides a wide variety of host-level metrics about memory usage, garbage collection, and more.

Download the latest release of Prometheus for your platform, then extract it:

```bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

The Prometheus server is a single binary called prometheus (or prometheus.exe on Microsoft Windows). We can run the binary and see help on its options by passing the --help flag.

```bash
./prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server
...
```

#### Configuring Prometheus

Prometheus configuration is YAML. The Prometheus download comes with a sample configuration in a file called prometheus.yml that is a good place to get started.

We've stripped out most of the comments in the example file to make it more succinct (comments are the lines prefixed with a #).

```yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```

There are three blocks of configuration in the example configuration file: **global, rule_files, and scrape_configs**.

The global block controls the Prometheus server's global configuration. We have two options present. The first, scrape_interval, controls how often Prometheus will scrape targets. You can override this for individual targets. In this case the global setting is to scrape every 15 seconds. The evaluation_interval option controls how often Prometheus will evaluate rules. Prometheus uses rules to create new time series and to generate alerts.

The rule_files block specifies the location of any rules we want the Prometheus server to load. For now we've got no rules.

The last block, scrape_configs, controls what resources Prometheus monitors. Since Prometheus also exposes data about itself as an HTTP endpoint it can scrape and monitor its own health. In the default configuration there is a single job, called prometheus, which scrapes the time series data exposed by the Prometheus server. The job contains a single, statically configured, target, the localhost on port 9090. Prometheus expects metrics to be available on targets on a path of /metrics. So this default job is scraping via the URL: [metrics](http://localhost:9090/metrics).

The time series data returned will detail the state and performance of the Prometheus server.

#### Starting Prometheus

To start Prometheus with our newly created configuration file, change to the directory containing the Prometheus binary and run:

```bash
./prometheus --config.file=prometheus.yml
```

Prometheus should start up. You should also be able to browse to a status page about itself at [localhost](http://localhost:9090). Give it about 30 seconds to collect data about itself from its own HTTP metrics endpoint.

You can also verify that Prometheus is serving metrics about itself by navigating to its own metrics endpoint: [metrics](http://localhost:9090/metrics).

## References

1. [setup-grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)
2. [download-prometheus](https://prometheus.io/download/)

---

## Ansible install

```bash
ansible-galaxy collection install prometheus.prometheus
ansible-playbook -e @secrets.enc deploy_monitors.yml
```
