# 1. Introduction to DataStax Enterprise Metrics Collector (DSE MC) 

[DSE MC](https://docs.datastax.com/en/monitoring/doc/monitoring/opsUseMetricsCollector.html) is a relatively new monitoring solution from DataStax that is used to monitor DSE cluster metrics (including OS metrics). It is part of DSE software package and was introduced in DSE verion 6.7 (enabled by default) and backported on DSE 6.0.5+ and DSE 5.1.14+ (disabled by default).

DSE Metrics Collector (DSE MC) is based on [collectd](https://collectd.org/) and works as a sub-process spawned by DSE JVM and its life cycle is managed by DSE.

Please **NOTE** that DSE MC is not a complete monitoring solution by itself. It simply simplifies the collection of DSE metrics in a standard, high performing way, and without the need to install an agent on DSE servers (aka, agent-less metrics collection). The collected metrics need to be exported to other dedicated  monitoring systems such as [Prometheus](https://prometheus.io/), [Graphite](https://graphiteapp.org/), and etc. for metrics viewing, dashboard-ing, and so on. 

## 1.1. Integrate DSE MC with Prometheus and Grafana

DataStax has provided a [repository](https://github.com/datastax/dse-metric-reporter-dashboards) to demonstrate how to integrate and view DSE cluster metrics in [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) through DSE MC. In particular, from this repo, you can get:
* Ready-to-use Prometheus and Grafana docker containers that are deployable through docker-compose.
* Prometheus configuration template for collecting DSE cluster metrics via DSE MC.
* Grafana configuration template and pre-built dashboard definitions to view the collected DSE server and OS metrics.

## 1.2. Automate the Integration

Although DataStax has already simplified the integration of DSE MC with Prometheus and Grafna, the [procedure](https://docs.datastax.com/en/monitoring/doc/monitoring/metricsCollector/mcExportMetricsDocker.html) still has some manual work involved. For example,
* We still need to install docker and docker compose on the host machine where we intend to run Prometheus and Grafana (assuming we don't have existing Prometheus and Grafana servers)
* We still need to configure "collectd" and DSE MC on each of DSE servers.
* We still need to adjust Prometheus configuration per DSE cluster so it can monitor all DSE nodes within the cluster.

In the first part (***mc_prom_grafana*** sub-folder) of this repository, I'm demonstrating how to use Ansible to automate the integration of DSE metrics with Prometheus and Grafana via DSE MC. All the above mentioned manual steps are taken care of and the Ansible playbook is also able to automatically detect and configure almost all DSE cluster specific information (e.g. DSE cluster name; DSE node IP list) as much as possible.

- ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) The  automation procedure is described [here](https://github.com/yabinmeng/dse_mc_nb/mc_prom_grafana/README.md).


# 2. Incorporate NoSQLBench Metrics with DSE Metrics in Prometheus and Grafana

