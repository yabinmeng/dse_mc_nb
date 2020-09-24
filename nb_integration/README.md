# Introduction to NB Metrics

When executing an NB scenario, there are different ways to generate the execution metrics. For example, a common metrics output format is CSV file. The following NB "run" command with option "**--report-csv-to <output_subfolder_name>**" option will generate a series of CSV files in the specified subfolder.

```
$ ./nb run driver=cql workload=cql-iot.yaml host="<dse_srv_ip>" tags=phase:rampup threads=10 cycles=5M --progress console:10s --report-csv-to csv_result
```

In the above example, the NB scenario definition file is "cql-iot.yaml" file and when the execution finishes, we'll see a bunch of CSV files created in a subfolder named "csv_result", with the following naming convention:
* <scenario_definition_file_name>.<metric_category>.csv, e.g.
  * cql-iot.yaml.execute.csv, or 
  * cql-iot.yaml.result.csv,
  * ... ...

