
# 1. Overview

The Ansible playbook (*dse_metrics_collector.yaml*) presented here is used to automate the [procedure](https://docs.datastax.com/en/monitoring/doc/monitoring/metricsCollector/mcExportMetricsDocker.html) of exporting DSE metrics using DSE MC to preconfigured Docker images. In particular, the Ansible playbook does the following tasks:

* [Metrics Server Host] Install docker engine and docker-compose
* [Metrics Server Host] Download and install DSE MC
* [DSE Server Hosts] Configure Prometheus collectd exporter on each DSE server
* [Metrics Server Host] Configure Prometheus server to scrape data from every DSE server
* [Metrics Server Host] Start docker containers for Prometheus and Grafana 

# 2. Execute the Script

## 2.1. **hosts** file 

We first need to update the **hosts** file by adding the proper host machine IP(s) under the two categories: 
* *[dse_server]*: the list of IPs of the DSE servers in the cluster 
* *[metrics_server]*: the host machine IP where the Prometheus and Grafana servers are running

```bash
[dse_server]
<DSE_Node_IP_1>
<DSE_Node_IP_2>
<DSE_Node_IP_3>
...

[metrics_server]
<Prometheus_and_Grafana_Host_IP>
```

### 2.1.1. Get DSE Server IP List

* For a DSE cluster (either single-DC or multi-DC), the following bash command can be used to get the IP list of all DSE servers within the cluster. 
```
$  nodetool status | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}"
```

## 2.2. Ansible playbook

```bash
ansible-playbook -i ./hosts dse_metrics_collector.yaml --private-key <ssh_private_key> -u <ssh_user>
```

Once the Ansible playbook is successfully executed, you can view a plethera of DSE cluster metrics from pre-built Grafana dashboards by accessing the following address:

```bash
http://<Prometheus_and_Grafana_Host_IP>:3000
```