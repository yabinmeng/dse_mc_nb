
# 

#### 1.2.1.1. **hosts** file 

In order to monitor a specific DSE cluster, we need to first update the **hosts** file by adding the proper host machine IP(s) under the two categories: 
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
#### 1.2.1.2. Get DSE Server IP List

* For a DSE cluster (either single-DC or multi-DC), the following bash command can be used to get the IP list of all DSE servers within the cluster. 
```
$  nodetool status | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}"
```

#### 1.2.1.3. Executing the Script

Running the Ansible playbook is simple, as below. Please make sure the required SSH access (with sudo privilege) for Ansible execution is pre-configured properly.

```bash
ansible-playbook -i ./hosts dse_metrics_collector.yaml --private-key <ssh_private_key> -u <ssh_user>
```

Once the Ansible playbook is successfully executed, you can view a plethera of DSE cluster metrics from pre-built Grafana dashboards by accessing the following address:

```bash
http://<Prometheus_and_Grafana_Host_IP>:3000
```
