# 1. Introduction to NB Metrics Output

When executing an NB scenario, there are different ways to generate the execution metrics:

* In the log output
* In CSV files
* In HDR histogram logs
* In Histogram Stats logs (CSV)
* To a monitoring system via graphite
* via the --docker-metrics option

In the first part of this repository, we launched 2 pre-built docker containers (one for Prometheus server and one for Grafana server) as the DSE MC monitoring platform. The goal in this part of the repository is on how to integrate NB metrics into the same Prometheus and Grafana monitoring platform on which DSE cluster metrics are monitored and displayed. 

**NOTE**: The NB option of "--docker-metrics" launches its own Prometheus and Grafana containers and can't be easily integrated with existing DSE MC Prometheus and Grafana solution (either dockerized or not). This option is therefore not considered in this repository.

# 2. NB Metrics Output Format - Graphite

NB actually has an option "**--report-graphite-to <graphite_srv_ip>[:<graphite_listening_port>]**" that can exporting NB metrics to a Graphite server directly. Since Graphite can be used as a direct data source to Grafana, it is possible for us to integrate NB metrics into existing Prometheus and Grafana monitoring platform for DSE cluster metrics.

There are 2 ways to do this:
* **Option 1**: Use a standalone, full-fledged Graphite server and bypass Promeheus. By full-fledged Graphite server, I mean it has all 3 components:
  * Graphite Carbon: a daemon process that listens for time series data
  * Graphite Whisper: a simple database library for storing time series data
  * Graphite Webapp:  a [Django](https://www.djangoproject.com/) webapp that renders graphs on-demand

* **Option 2**: Use the lightweight [Prometheus Graphite Exporter](https://github.com/prometheus/graphite_exporter) that allows a Prometheus server to scrape metrics based on [Graphite Plaintext Protocol](https://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-plaintext-protocol).

The final code/script presented in this repository is based on **Option 2**, but I will also cover Option 1 in the document.

## 2.1. Option 1 - Exporting NB Metrics to Grafana via a Standalone Graphite Server

Please refer [Graphite doucmentation](https://graphite.readthedocs.io/en/latest/install.html#id2) for detailed methods of how to install Graphite. A good reference can also be found from [here](https://www.vultr.com/docs/how-to-install-and-configure-graphite-on-ubuntu-16-04#Step_5__Configure_Carbon).

After installed, Graphite Carbon will listen on **port 2003** by default. From NB perspective, this is the port NB is going to send metrics to. An example is as below:

```
$ ./nb run driver=cql workload=cql-iot.yaml host="<dse_server_ip> tags=phase:rampup threads=5 cycles=5M --report-graphite-to <graphite_server_machine_ip>:2003
```

While NB secnario is running, let's check Graphite UI and we can see that a list of execution metrics related with this particular named scenario ("cql-iot.yaml") are available:

<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/docs/nb_integration/screenshots/nb_graphite_carbon.png" width=600>

Now that we have NB metrics data fed into Graphite, we can add a Graphite data source in Grafana and add some Grafana dashboards that correspond to NB metrics.

<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/docs/nb_integration/screenshots/grafana_graphite_ds.png" width=400 height=500> <img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/docs/nb_integration/screenshots/grafana_nb_dashboad.png" width=600 height=500>

## 2.2. Option 2 - Exporting NB Metrics to Grafana via Prometheus and Prometheus Graphite Exporter

Option 1 requires installing a complete Graphite server and all its dependency components (including a database and a web server). For many cases, this may not be a feasible option (e.g. being either too complicated or restricted by security policies for production deployment). 

Now, since NB is able to expose metrices through [Graphite Plaintext Protocol](https://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-plaintext-protocol), the easiest approach to brdige the gap between NB and our existing Prometheus and Grafana based monitoring platform is through [Prometheus Graphite Exporter](https://github.com/prometheus/graphite_exporter), as demonstrated in the following diagram.

<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/docs/nb_integration/screenshots/graphite_exporter.png" width=600>

So when a Prometheus Graphite Exporter process is running, it listens on (by default) port 9109 for incoming metrics stream based on Graphite Plaintext Protocol and meanwhile it also has port  9108 open for Prometheus server to scrape data. From NB perspective, port 9109 is where NB is going to send metrics to. 

Our existing Prometheus and Grafana servers are launched using pre-configured docker containers through a [docker-compose](https://github.com/datastax/dse-metric-reporter-dashboards/blob/master/docker-compose.yml) file (see [here](https://github.com/datastax/dse-metric-reporter-dashboards) for more details).

We can modifiy that docker-compose file a bit so that a Prometheus Graphite Exporter is also started as a dependency container on which Prometheus and Grafana containers rely on. The key parts of the updated docker compose file looks like below:

```
ersion: "3"
services:
  graphite-exporter:
    container_name: nb-metric-dashboards_graphite_exporter_1
    image: "prom/graphite-exporter"
    ports:
      - "9109:9109"
  prometheus:
    ... ...
    links:
      - "graphite-exporter"
  grafana:
    ... ...
    links:
      - "graphite-exporter"
  ... ...
```

The complete upadated docker-compose file can be found [here](https://github.com/yabinmeng/dse_mc_nb/tree/master/scripts/docker/docker-compose.yml).

Once the containers are up and running, you should see 3 docker containers, as below:

```
$ docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                        NAMES
f399e98a20b7        grafana/grafana:5.3.2     "/run.sh"                24 hours ago        Up 24 hours         0.0.0.0:3000->3000/tcp                       dse-metric-dashboards_grafana_1
41ef1c890694        prom/prometheus:v2.17.1   "/bin/prometheus --c…"   24 hours ago        Up 24 hours         0.0.0.0:9090->9090/tcp                       dse-metric-dashboards_prometheus_1
74e20799ebfb        prom/graphite-exporter    "/bin/graphite_expor…"   25 hours ago        Up 24 hours         9108/tcp, 9109/udp, 0.0.0.0:9109->9109/tcp   nb-metric-dashboards_graphite_exporter_1
```

Let's run the NB scenario (cql-iot.yaml) again (**NOTE** the graphite port difference).

```
$ ./nb run driver=cql workload=cql-iot.yaml host="<dse_server_ip> tags=phase:rampup threads=5 cycles=5M --report-graphite-to <graphite_exporter_machine_ip>:1909
```

Now since the NB metrics can be scraped in Prometheus (through Graphite Exporter), we're able to create Grafana dashboards for NB metrics, along with other pre-configured DSE dashboards. **NOTE** that there is NO need to add a different data source because both NB metrics and DSE metrics data are managed in the same Prometheus server.

The Grafana dashboard below shows the throughput metrics (mean/m-1/m-5/m-15) for the executed NB scenario.

<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/docs/nb_integration/screenshots/nb_grafana_dasboard_rate.png" width=1000 height=600>