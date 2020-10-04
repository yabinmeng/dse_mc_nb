# 1. Introduction to DataStax Enterprise Metrics Collector (DSE MC) 

[DSE MC](https://docs.datastax.com/en/monitoring/doc/monitoring/opsUseMetricsCollector.html) is a relatively new monitoring solution from DataStax that is used to monitor DSE cluster metrics (including OS metrics). It is part of DSE software package and was introduced in DSE verion 6.7 (enabled by default) and backported on DSE 6.0.5+ and DSE 5.1.14+ (disabled by default).

DSE Metrics Collector (DSE MC) is based on [collectd](https://collectd.org/) and works as a sub-process spawned by DSE JVM and its life cycle is managed by DSE.

Please **NOTE** that DSE MC is not a complete monitoring solution by itself. It simply simplifies the collection of DSE metrics in a standard, high performing way, and without the need to install an agent on DSE servers (aka, agent-less metrics collection). The collected metrics need to be exported to other dedicated  monitoring systems such as [Prometheus](https://prometheus.io/), [Graphite](https://graphiteapp.org/), and etc. for metrics viewing, dashboard-ing, and so on. 

## 1.1. Integrate DSE MC with Prometheus and Grafana

DataStax has provided a [repository](https://github.com/datastax/dse-metric-reporter-dashboards) to demonstrate how to integrate and view DSE cluster metrics in [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/) through DSE MC. In particular, from this repo, you can get:
* Ready-to-use Prometheus and Grafana docker containers that are deployable through docker-compose.
* Prometheus configuration template for collecting DSE cluster metrics via DSE MC.
* Grafana configuration template and pre-built dashboard definitions to view the collected DSE server and OS metrics.

## 1.2. Automate the Integration Procedure

Although DataStax has already simplified the integration of DSE MC with Prometheus and Grafna, the [procedure](https://docs.datastax.com/en/monitoring/doc/monitoring/metricsCollector/mcExportMetricsDocker.html) still has some manual work involved. For example,
* We still need to install docker and docker compose on the host machine where we intend to run Prometheus and Grafana (assuming we don't have existing Prometheus and Grafana servers)
* We still need to configure "collectd" and DSE MC on each of DSE servers.
* We still need to adjust Prometheus configuration per DSE cluster so it can monitor all DSE nodes within the cluster.

In the first part (***docs/mc_prom_grafana*** sub-folder) of this repository, we're going to demonstrate how to use Ansible to automate the integration of DSE metrics with Prometheus and Grafana via DSE MC. All the above mentioned manual steps are taken care of and the Ansible playbook is also able to automatically detect and configure almost all DSE cluster specific information (e.g. DSE cluster name; DSE node IP list) as much as possible.

---

&#x1F4D8;&#x1F4D8; The **automation procedure** is described [here](https://github.com/yabinmeng/dse_mc_nb/tree/master/docs/mc_prom_grafana).

---

# 2. Incorporate NoSQLBench Metrics with DSE Metrics in Prometheus and Grafana

## 2.1. Quick Introduction of NoSQLBench (NB)

[NoSQLBench (NB)](https://github.com/nosqlbench/nosqlbench) is "an open-source, pluggable, NoSQL benchmarking sutie". NB is both a performance testing execution framework and workload generation engine. It supports testing different systems (e.g. Cassandra, DSE, MongoDB, Kafka, etc.) through pluggable drivers. 

For more detailed information about how to use NB, please refer to its document site at [here](http://docs.nosqlbench.io/#/). 

## 2.2. Combine NB Metrics and DSE Metrics Together 

When using NB for performance testing, it is often valuable to look the the performance metrics in a holistic view, covering both application side metrics (NB execution metrics) and DSE server side metrics.

In the second part (***docs/nb_integration***) of this repository, I will show how to consolidate both NB metrics and DSE (MC) metris together in the Prometheus server and show the metrics dashboards via the same Grafana UI.

---

&#x1F4D8;&#x1F4D8; The **integration procedure** is described [here](https://github.com/yabinmeng/dse_mc_nb/tree/master/docs/nb_integration).

---

# Appendix. Some Color Unicode Symbols in Mark-down

```
RED APPLE (&#x1F34E;): 🍎
GREEN APPLE (&#x1F34F;): 🍏
BLUE HEART (&#x1F499;): 💙
GREEN HEART (&#x1F49A;): 💚
YELLOW HEART (&#x1F49B;): 💛
PURPLE HEART (&#x1F49C;): 💜
GREEN BOOK (&#x1F4D7;): 📗
BLUE BOOK (&#x1F4D8;): 📘
ORANGE BOOK (&#x1F4D9;): 📙
LARGE RED CIRCLE (&#x1F534;): 🔴
LARGE BLUE CIRCLE (&#x1F535;): 🔵
LARGE ORANGE DIAMOND (&#x1F536;): 🔶
LARGE BLUE DIAMOND (&#x1F537;): 🔷
SMALL ORANGE DIAMOND (&#x1F538;): 🔸
SMALL BLUE DIAMOND (&#x1F539;): 🔹
UP-POINTING RED TRIANGLE (&#x1F53A;): 🔺
DOWN-POINTING RED TRIANGLE (&#x1F53B;): 🔻
UP-POINTING SMALL RED TRIANGLE (&#x1F53C;): 🔼
DOWN-POINTING SMALL RED TRIANGLE (&#x1F53D;): 🔽
```

**NOTE**: for more color unicode symbols, please check [here](https://apps.timwhitlock.info/emoji/tables/unicode)