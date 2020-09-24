# Introduction to NB Metrics Output

When executing an NB scenario, there are different ways to generate the execution metrics:

* In the log output
* In CSV files
* In HDR histogram logs
* In Histogram Stats logs (CSV)
* To a monitoring system via graphite
* via the --docker-metrics option

In the first part of this repository, we launched 2 pre-built docker containers (one for Prometheus server and one for Grafana server) as the DSE MC monitoring platform. The goal in this part of the repository is on how to integrate NB metrics (esp. in CSV format and in Graphite mode) into the same Prometheus and Grafana monitoring platform on which DSE cluster metrics are monitored and displayed. 

**NOTE**: The NB option of "--docker-metrics" launches its own Prometheus and Grafana containers and can't be easily integrated with existing DSE MC Prometheus and Grafana solution (either dockerized or not). This option is therefore not considered in this repository.

# Metrics Output Format - Graphite

NB actually has an option "--report-graphite-to <graphite_srv_ip>[:<graphite_listening_port>]" that can exporting NB metrics to a Graphite server directly. Since Graphite can be used as a direct data source to Grafana, it is possible for us to integrate NB metrics into existing Prometheus and Grafana monitoring platform for DSE cluster metrics.

There are 2 ways to do this:
* **Option 1**: Use a standalone, full-fledged Graphite server and bypass Promeheus. By full-fledged Graphite server, I mean it has all 3 components:
  * Graphite Carbon: a daemon process that listens for time series data
  * Graphite Whisper: a simple database library for storing time series data
  * Graphite Webapp:  a [Django](https://www.djangoproject.com/) webapp that renders graphs on-demand

* **Option 2**: Use the lightweight [Prometheus Graphite Exporter](https://github.com/prometheus/graphite_exporter) that allows a Prometheus server to scrape metrics based on [Graphite Plaintext Protocol](https://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-plaintext-protocol).

The final code/script presented in this repository is based on **Option 2**, but I will also cover Option 1 in the document.

## Option 1: Exporting NB Metrics to Grafana through a Complete Graphite Server

Please refer [Graphite doucmentation](https://graphite.readthedocs.io/en/latest/install.html#id2) for detailed methods of how to install Graphite. A good reference can also be found from [here](https://www.vultr.com/docs/how-to-install-and-configure-graphite-on-ubuntu-16-04#Step_5__Configure_Carbon).

After installed, Graphite Carbon will listen on **port 2003** by default. From NB perspective, this is the port NB is going to send metrics to. An example is as below:

```
$ ./nb run driver=cql workload=cql-iot.yaml host="<dse_server_ip> tags=phase:rampup threads=5 cycles=5M --report-graphite-to <graphite_server_ip>:2003
```

While NB secnario is running, let's check Graphite UI and we can see that a list of execution metrics related with this particular named scenario ("cql-iot.yaml") are available:

<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/nb_integration/screenshots/nb_graphite_carbon.png" width=600>

Now that we have NB metrics data fed into Graphite, we can add a [Graphite data source](https://github.com/yabinmeng/dse_mc_nb/blob/master/nb_integration/screenshots/grafana_graphite_ds.png) in Grafana and add some [Grafana dashboards](<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/nb_integration/screenshots/grafana_nb_dashboad.png" width=600>).

<img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/nb_integration/screenshots/grafana_graphite_ds.png" width=400> <img src="https://github.com/yabinmeng/dse_mc_nb/blob/master/nb_integration/screenshots/grafana_nb_dashboad.png" width=600>


# Metrics Output Format - CSV Format

One common metrics output format is CSV file. The following NB "run" command with option "**--report-csv-to <output_subfolder_name>**" option will generate a series of CSV files in the specified subfolder.

```
$ ./nb run driver=cql workload=cql-iot.yaml host="<dse_srv_ip>" tags=phase:rampup threads=10 cycles=5M --progress console:10s --report-csv-to csv_result
```

In the above example, the NB scenario definition file is "cql-iot.yaml" file and when the execution finishes, we'll see a bunch of CSV files created in a subfolder named "csv_result", with the following naming convention:
* <scenario_definition_file_name>.<metric_category>.csv, e.g.
  * cql-iot.yaml.execute.csv, or 
  * cql-iot.yaml.result.csv,
  * ... ...

Among these metrics output CSV files, the most important one is the "xxx.result.csv" file which represents the scenario's overall execution results that are measured through a variety of different performance metrics. 

The metrics in these result files are organized in a time-series manner. A snippet of the "xxx.result.csv" file is shown as below:

```
t,count,max,mean,min,stddev,p50,p75,p95,p98,p99,p999,mean_rate,m1_rate,m5_rate,m15_rate,rate_unit,duration_unit
1600914791,34117,60940287.000000,1803827.492156,17806.000000,1905487.655309,1410687.000000,1927103.000000,4036351.000000,5983743.000000,7678719.000000,22299647.000000,5154.923941,4766.000000,4766.000000,4766.000000,calls/second,nanoseconds
1600914801,107287,27215871.000000,1303639.684363,1749.000000,1045685.701754,1109119.000000,1414655.000000,2206463.000000,4069759.000000,5847807.000000,13336575.000000,6559.366070,5156.514798,4848.676947,4793.825412,calls/second,nanoseconds
... ... 
```

## Feed Historical NB CSV Metrics into Prometheus and Grafana


