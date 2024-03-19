# Prometheus

## Introduction

### Overview

#### What is Prometheus?

[Prometheus](https://github.com/prometheus) is an open-source systems monitoring and alerting toolkit originally built at SoundCloud.
Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community.
It is now a standalone open source project and maintained independently of any company.
To emphasize this, and to clarify the project's governance structure, Prometheus joined the Cloud Native Computing Foundation in 2016 as the second hosted project, after Kubernetes.

Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels.

For more elaborate overviews of Prometheus, see the resources linked from the media section.

##### Features

Prometheus's main features are:

* a multi-dimensional [data model](https://prometheus.io/docs/concepts/data_model/) with time series data identified by metric name and key/value pairs
* PromQL, a flexible query language to leverage this dimensionality
* no reliance on distributed storage; single server nodes are autonomous
* time series collection happens via a pull model over HTTP
* [pushing time series](https://prometheus.io/docs/instrumenting/pushing/) is supported via an intermediary gateway
* targets are discovered via service discovery or static configuration
* multiple modes of graphing and dashboarding support

##### What are metrics?

Metrics are numerical measurements in layperson terms.
The term time series refers to the recording of changes over time.
What users want to measure differs from application to application.
For a web server, it could be request times; for a database, it could be the number of active connections or active queries, and so on.

Metrics play an important role in understanding why your application is working in a certain way.
Let's assume you are running a web application and discover that it is slow.
To learn what is happening with your application, you will need some information.
For example, when the number of requests is high, the application may become slow.
If you have the request count metric, you can determine the cause and increase the number of servers to handle the load.

##### Components

The Prometheus ecosystem consists of multiple components, many of which are optional:

* the main [Prometheus server](https://github.com/prometheus/prometheus) which scrapes and stores time series data
* [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) for instrumenting application code
* a push gateway for supporting short-lived jobs
* special-purpose [exporters](https://prometheus.io/docs/instrumenting/exporters/) for services like HAProxy, StatsD, Graphite, etc.
* an [[alertmanager]](https://github.com/prometheus/alertmanager) to handle alerts
* various support tools

Most Prometheus components are written in Go, making them easy to build and deploy as static binaries.

##### Architecture

Prometheus scrapes metrics from instrumented jobs, either directly or via an intermediary push gateway for short-lived jobs.
It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts.
Grafana or other API consumers can be used to visualize the collected data.

#### When does it fit?

Prometheus works well for recording any purely numeric time series.
It fits both machine-centric monitoring as well as monitoring of highly dynamic service-oriented architectures.
In a world of microservices, its support for multi-dimensional data collection and querying is a particular strength.

Prometheus is designed for reliability, to be the system you go to during an outage to allow you to quickly diagnose problems.
Each Prometheus server is standalone, not depending on network storage or other remote services.
You can rely on it when other parts of your infrastructure are broken, and you do not need to setup extensive infrastructure to use it.

### Glossary

#### Alert

#### Alertmanager

The Alertmanager takes in alerts, aggregates them into groups, de-duplicates, applies silences, throttles, and then sends out notifications to email, Pagerduty, Slack etc.

#### Client Library

#### Endpoint

A source of metrics that can be scraped, usually corresponding to a single process.

#### Exporter

An exporter is a binary running alongside the application you want to obtain metrics from.
The exporter exposes Prometheus metrics, commonly by converting metrics that are exposed in a non-Prometheus format into a format that Prometheus supports.

#### Notification

A notification represents a group of one or more alerts, and is sent by the Alertmanager to email, Pagerduty, Slack etc.

#### Remote Read

Remote read is a Prometheus feature that allows transparent reading of time series from other systems (such as long term storage) as part of queries.

#### Remote Write

Remote write is a Prometheus feature that allows sending ingested samples on the fly to other systems, such as long term storage.

#### Target

A target is the definition of an object to scrape.
For example, what labels to apply, any authentication required to connect, or other information that defines how the scrape will occur.

#### Time Series

The Prometheus time series are streams of timestamped values belonging to the same metric and the same set of labeled dimensions.
Prometheus stores all data as time series.

## Concepts

### Data Model

Prometheus fundamentally stores all data as [time series](https://en.wikipedia.org/wiki/Time_series): streams of timestamped values belonging to the same metric and the same set of labeled dimensions.
Besides stored time series, Prometheus may generate temporary derived time series as the result of queries.

#### Metric names and labels

Every time series is uniquely identified by its metric name and optional key-value pairs called labels.

##### Metric names:

* Specify the general feature of a system that is measured (e.g. `http_requests_total` - the total number of HTTP requests received).
* Metric names may contain ASCII letters, digits, underscores, and colons. It must match the regex `[a-zA-Z_:][a-zA-Z0-9_:]*`.

Note: The colons are reserved for user defined recording rules.
They should not be used by exporters or direct instrumentation.

##### Metric labels:

* Enable Prometheus's dimensional data model to identify any given combination of labels for the same metric name. It identifies a particular dimensional instantiation of that metric (for example: all HTTP requests that used the method `POST` to the `/api/tracks` handler). The query language allows filtering and aggregation based on these dimensions.
* The change of any labels value, including adding or removing labels, will create a new time series.
* Labels may contain ASCII letters, numbers, as well as underscores. They must match the regex `[a-zA-Z_][a-zA-Z0-9_]*`.
* Label names beginning with `__` (two "_") are reserved for internal use.
* Label values may contain any Unicode characters.
* Labels with an empty label value are considered equivalent to labels that do not exist.

#### Samples

Samples form the actual time series data.

#### Notation

Given a metric name and a set of labels, time series are frequently identified using this notation:
```
<metric name>{<label name>=<label value>, ...}
```
For example, a time series with the metric name `api_http_requests_total` and the labels `method="POST"` and `handler="/messages"` could be written like this:
```
api_http_requests_total{method="POST", handler="/messages"}
```

### Metric Types

The Prometheus client libraries offer four core metric types.
These are currently only differentiated in the client libraries (to enable APIs tailored to the usage of the specific types) and in the wire protocol.
The Prometheus server does not yet make use of the type information and flattens all data into untyped time series.
This may change in the future.

#### Counter

A *counter* is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart.
For example, you can use a counter to represent the number of requests served, tasks completed, or errors.

Do not use a counter to expose a value that can decrease.
For example, do not use a counter for the number of currently running processes; instead use a gauge.

#### Gauge

A *gauge* is a metric that represents a single numerical value that can arbitrarily go up and down.

Gauges are typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of concurrent requests.

#### Histogram

A *histogram* samples observations (usually things like request durations or response sizes) and counts them in configurable buckets.
It also provides a sum of all observed values.

#### Summary

Similar to a histogram, a *summary* samples observations (usually things like request durations and response sizes).
While it also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window.

### Jobs and Instances

In Prometheus terms, an endpoint you can scrape is called an *instance*, usually corresponding to a single process.
A collection of instances with the same purpose, a process replicated for scalability or reliability for example, is called a *job*.

For example, an API server job with four replicated instances:
* job: api-server
  * instance 1: 1.2.3.4:5670
  * instance 2: 1.2.3.4:5671
  * instance 3: 5.6.7.8:5670
  * instance 4: 5.6.7.8:5671

#### Automatically generated labels and time series

When Prometheus scrapes a target, it attaches some labels automatically to the scraped time series which serve to identify the scraped target:
* `job`: The configured job name that the target belongs to.
* `instance`: The `<host>:<port>` part of the target's URL that was scraped.

If either of these labels are already present in the scraped data, the behavior depends on the `honor_labels` configuration option.
See the [scrape configuration documentation](#scrape_config) for more information.

### Prometheus Remote-write Specification

## Prometheus

### Getting Started

This guide is a "Hello World"-style tutorial which shows how to install, configure, and use a simple Prometheus instance.
You will download and run Prometheus locally, configure it to scrape itself and an example application, then work with queries, rules, and graphs to use collected time series data.

#### Downloading and running Prometheus

#### Configuring Prometheus to monitor itself

Prometheus collects metrics from *targets* by scraping metrics HTTP endpoints. Since Prometheus exposes data in the same manner about itself, it can also scrape and monitor its own health.

While a Prometheus server that collects only data about itself is not very useful, it is a good starting example.

#### Starting Prometheus

#### Using the expression browser

Let us explore data that Prometheus has collected about itself.
To use Prometheus's built-in expression browser, navigate to http://localhost:9090/graph and choose the "Table" view within the "Graph" tab.

As you can gather from localhost:9090/metrics, one metric that Prometheus exports about itself is named `prometheus_target_interval_length_seconds` (the actual amount of time between target scrapes).
Enter the below into the expression console and then click "Execute":
```
prometheus_target_interval_length_seconds
```
This should return a number of different time series (along with the latest value recorded for each), each with the metric name `prometheus_target_interval_length_seconds`, but with different labels.
These labels designate different latency percentiles and target group intervals.

If we are interested only in 99th percentile latencies, we could use this query:
```
prometheus_target_interval_length_seconds{quantile="0.99"}
```
To count the number of returned time series, you could write:
```
count(prometheus_target_interval_length_seconds)
```
For more about the expression language, see the [expression language documentation](https://prometheus.io/docs/prometheus/2.47/querying/basics/).

#### Using the graphing interface

To graph expressions, navigate to http://localhost:9090/graph and use the "Graph" tab.

For example, enter the following expression to graph the per-second rate of chunks being created in the self-scraped Prometheus:
```
rate(prometheus_tsdb_head_chunks_created_total[1m])
```
Experiment with the graph range parameters and other settings.

#### Starting up some sample targets

#### Configure Prometheus to monitor the sample targets

Now we will configure Prometheus to scrape these new targets.
Let's group all three endpoints into one job called `node`.
We will imagine that the first two endpoints are production targets, while the third one represents a canary instance.
To model this in Prometheus, we can add several groups of endpoints to a single job, adding extra labels to each group of targets.
In this example, we will add the `group="production"` label to the first group of targets, while adding `group="canary"` to the second.

To achieve this, add the following job definition to the `scrape_configs` section in your `prometheus.yml` and restart your Prometheus instance:
```yaml
scrape_configs:
  - job_name:       'node'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```
Go to the expression browser and verify that Prometheus now has information about time series that these example endpoints expose, such as `node_cpu_seconds_total`.

#### Configure rules for aggregating scraped data into new time series

Though not a problem in our example, queries that aggregate over thousands of time series can get slow when computed ad-hoc.
To make this more efficient, Prometheus can prerecord expressions into new persisted time series via configured *recording rules*.
Let's say we are interested in recording the per-second rate of cpu time (`node_cpu_seconds_total`) averaged over all cpus per instance (but preserving the `job`, `instance` and `mode` dimensions) as measured over a window of 5 minutes.
We could write this as:
```
avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```
Try graphing this expression.

To record the time series resulting from this expression into a new metric called `job_instance_mode:node_cpu_seconds:avg_rate5m`, create a file with the following recording rule and save it as `prometheus.rules.yml`:
```yaml
groups:
- name: cpu-node
  rules:
  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```
To make Prometheus pick up this new rule, add a `rule_files` statement in your `prometheus.yml`.
The config should now look like this:
```yaml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds.

  # Attach these extra labels to all timeseries collected by this Prometheus instance.
  external_labels:
    monitor: 'codelab-monitor'

rule_files:
  - 'prometheus.rules.yml'

scrape_configs:
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']

  - job_name:       'node'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```
Restart Prometheus with the new configuration and verify that a new time series with the metric name `job_instance_mode:node_cpu_seconds:avg_rate5m` is now available by querying it through the expression browser or graphing it.

#### Reloading configuration

As mentioned in the [configuration documentation](https://prometheus.io/docs/prometheus/2.47/configuration/configuration/) a Prometheus instance can have its configuration reloaded without restarting the process by using the `SIGHUP` signal.
If you're running on Linux this can be performed by using `kill -s SIGHUP <PID>`, replacing `<PID>` with your Prometheus process ID.

#### Shutting down your instance gracefully

While Prometheus does have recovery mechanisms in the case that there is an abrupt process failure it is recommend to use the `SIGTERM` signal to cleanly shutdown a Prometheus instance.
If you're running on Linux this can be performed by using `kill -s SIGTERM <PID>`, replacing `<PID>` with your Prometheus process ID.

### Installation

#### Using pre-compiled binaries

#### From source

#### Using Docker

#### Using configuration management systems

##### Ansible

##### Chef

##### Puppet

##### SaltStack

### Configuration

Prometheus is configured via command-line flags and a configuration file.
While the command-line flags configure immutable system parameters (such as storage locations, amount of data to keep on disk and in memory, etc.), the configuration file defines everything related to scraping [jobs and their instances](#jobs-and-instances), as well as which rule files to load.

To view all available command-line flags, run `./prometheus -h`.

Prometheus can reload its configuration at runtime.
If the new configuration is not well-formed, the changes will not be applied.
A configuration reload is triggered by sending a `SIGHUP` to the Prometheus process or sending a HTTP POST request to the `/-/reload` endpoint (when the `--web.enable-lifecycle` flag is enabled).
This will also reload any configured rule files.

#### Configuration file

To specify which configuration file to load, use the `--config.file` flag.

The file is written in [YAML format](https://en.wikipedia.org/wiki/YAML), defined by the scheme described below.
Brackets indicate that a parameter is optional.
For non-list parameters the value is set to the specified default.

The global configuration specifies parameters that are valid in all other configuration contexts.
They also serve as defaults for other configuration sections.

##### `<scrape_config>`

A `scrape_config` section specifies a set of targets and parameters describing how to scrape them.
In the general case, one scrape configuration specifies a single job.
In advanced configurations, this may change.

Targets may be statically configured via the `static_configs` parameter or dynamically discovered using one of the supported service-discovery mechanisms.

Additionally, `relabel_configs` allow advanced modifications to any target and its labels before scraping.

Where `<job_name>` must be unique across all scrape configurations.

##### `<ec2_sd_config>`

EC2 SD configurations allow retrieving scrape targets from AWS EC2 instances.
The private IP address is used by default, but may be changed to the public IP address with relabeling.

The IAM credentials used must have the ec2:DescribeInstances permission to discover scrape targets, and may optionally have the ec2:DescribeAvailabilityZones permission if you want the availability zone ID available as a label (see below).

The following meta labels are available on targets during [relabeling](#relabel_config):
* `__meta_ec2_ami`: the EC2 Amazon Machine Image
* `__meta_ec2_architecture`: the architecture of the instance
* `__meta_ec2_availability_zone`: the availability zone in which the instance is running
* `__meta_ec2_availability_zone_id`: the availability zone ID in which the instance is running (requires `ec2:DescribeAvailabilityZones`)
* `__meta_ec2_instance_id`: the EC2 instance ID
* `__meta_ec2_instance_lifecycle`: the lifecycle of the EC2 instance, set only for 'spot' or 'scheduled' instances, absent otherwise
* `__meta_ec2_instance_state`: the state of the EC2 instance
* `__meta_ec2_instance_type`: the type of the EC2 instance
* `__meta_ec2_ipv6_addresses`: comma separated list of IPv6 addresses assigned to the instance's network interfaces, if present
* `__meta_ec2_owner_id`: the ID of the AWS account that owns the EC2 instance
* `__meta_ec2_platform`: the Operating System platform, set to 'windows' on Windows servers, absent otherwise
* `__meta_ec2_primary_subnet_id`: the subnet ID of the primary network interface, if available
* `__meta_ec2_private_dns_name`: the private DNS name of the instance, if available
* `__meta_ec2_private_ip`: the private IP address of the instance, if present
* `__meta_ec2_public_dns_name`: the public DNS name of the instance, if available
* `__meta_ec2_public_ip`: the public IP address of the instance, if available
* `__meta_ec2_region`: the region of the instance
* `__meta_ec2_subnet_id`: comma separated list of subnets IDs in which the instance is running, if available
* `__meta_ec2_tag_<tagkey>`: each tag value of the instance
* `__meta_ec2_vpc_id`: the ID of the VPC in which the instance is running, if available

##### `<kubernetes_sd_config>`

Kubernetes SD configurations allow retrieving scrape targets from Kubernetes' REST API and always staying synchronized with the cluster state.

One of the following `role` types can be configured to discover targets:
* `node`
* `service`
* `pod`
* `endpoints`
* `endpointslice`
* `ingress`

See [this example Prometheus configuration file](https://github.com/prometheus/prometheus/blob/release-2.47/documentation/examples/prometheus-kubernetes.yml) for a detailed example of configuring Prometheus for Kubernetes.

You may wish to check out the 3rd party [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator), which automates the Prometheus setup on top of Kubernetes.

##### `<static_config>`

A `static_config` allows specifying a list of targets and a common label set for them.
It is the canonical way to specify static targets in a scrape configuration.
```yaml
# The targets specified by the static config.
targets:
  [ - '<host>' ]

# Labels assigned to all metrics scraped from the targets.
labels:
  [ <labelname>: <labelvalue> ... ]
```

##### `<relabel_config>`

Relabeling is a powerful tool to dynamically rewrite the label set of a target before it gets scraped.
Multiple relabeling steps can be configured per scrape configuration.
They are applied to the label set of each target in order of their appearance in the configuration file.

Initially, aside from the configured per-target labels, a target's `job` label is set to the `job_name` value of the respective scrape configuration.
The `__address__` label is set to the `<host>:<port>` address of the target.
After relabeling, the instance label is set to the value of `__address__` by default if it was not set during relabeling.
The `__scheme__` and `__metrics_path__` labels are set to the scheme and metrics path of the target respectively.
The `__param_<name>` label is set to the value of the first passed URL parameter called `<name>`.

The `__scrape_interval__` and `__scrape_timeout__` labels are set to the target's interval and timeout.
This is **experimental** and could change in the future.

Additional labels prefixed with `__meta_` may be available during the relabeling phase.
They are set by the service discovery mechanism that provided the target and vary between mechanisms.

Labels starting with `__` will be removed from the label set after target relabeling is completed.

If a relabeling step needs to store a label value only temporarily (as the input to a subsequent relabeling step), use the `__tmp` label name prefix.
This prefix is guaranteed to never be used by Prometheus itself.
```yaml
# The source labels select values from existing labels. Their content is concatenated
# using the configured separator and matched against the configured regular expression
# for the replace, keep, and drop actions.
[ source_labels: '[' <labelname> [, ...] ']' ]

# Separator placed between concatenated source label values.
[ separator: <string> | default = ; ]

# Label to which the resulting value is written in a replace action.
# It is mandatory for replace actions. Regex capture groups are available.
[ target_label: <labelname> ]

# Regular expression against which the extracted value is matched.
[ regex: <regex> | default = (.*) ]

# Modulus to take of the hash of the source label values.
[ modulus: <int> ]

# Replacement value against which a regex replace is performed if the
# regular expression matches. Regex capture groups are available.
[ replacement: <string> | default = $1 ]

# Action to perform based on regex matching.
[ action: <relabel_action> | default = replace ]
```
`<regex>` is any valid [RE2 regular expression](https://github.com/google/re2/wiki/Syntax).
It is required for the `replace`, `keep`, `drop`, `labelmap`, `labeldrop` and `labelkeep` actions.
The regex is anchored on both ends.
To un-anchor the regex, use `.*<regex>.*`.

`<relabel_action>` determines the relabeling action to take:
* `replace`: Match `regex` against the concatenated `source_labels`. Then, set `target_label` to `replacement`, with match group references (`${1}`, `${2}`, ...) in `replacement` substituted by their value. If `regex` does not match, no replacement takes place.
* `lowercase`: Maps the concatenated `source_labels` to their lower case.
* `uppercase`: Maps the concatenated `source_labels` to their upper case.
* `keep`: Drop targets for which `regex` does not match the concatenated `source_labels`.
* `drop`: Drop targets for which `regex` matches the concatenated `source_labels`.
* `keepequal`: Drop targets for which the concatenated `source_labels` do not match `target_label`.
* `dropequal`: Drop targets for which the concatenated `source_labels` do match `target_label`.
* `hashmod`: Set `target_label` to the modulus of a hash of the concatenated `source_labels`.
* `labelmap`: Match `regex` against all source label names, not just those specified in `source_labels`. Then copy the values of the matching labels to label names given by `replacement` with match group references (`${1}`, `${2}`, ...) in `replacement` substituted by their value.
* `labeldrop`: Match `regex` against all label names. Any label that matches will be removed from the set of labels.
* `labelkeep`: Match `regex` against all label names. Any label that does not match will be removed from the set of labels.

Care must be taken with `labeldrop` and `labelkeep` to ensure that metrics are still uniquely labeled once the labels are removed.

##### `<remote_write>`

`write_relabel_configs` is relabeling applied to samples before sending them to the remote endpoint.
Write relabeling is applied after external labels.
This could be used to limit which samples are sent.

### Defining Recording Rules

#### Configuring rules

Prometheus supports two types of rules which may be configured and then evaluated at regular intervals: recording rules and [alerting rules](#alerting-rules).
To include rules in Prometheus, create a file containing the necessary rule statements and have Prometheus load the file via the `rule_files` field in the Prometheus configuration.
Rule files use YAML.

#### Syntax-checking rules

To quickly check whether a rule file is syntactically correct without starting a Prometheus server, you can use Prometheus's `promtool` command-line utility tool:

#### Recording rules

Recording rules allow you to precompute frequently needed or computationally expensive expressions and save their result as a new set of time series.
Querying the precomputed result will then often be much faster than executing the original expression every time it is needed.
This is especially useful for dashboards, which need to query the same expression repeatedly every time they refresh.

Recording and alerting rules exist in a rule group.
Rules within a group are run sequentially at a regular interval, with the same evaluation time.
The names of recording rules must be valid metric names.
The names of alerting rules must be valid label values.

The syntax of a rule file is:
```
groups:
  [ - <rule_group> ]
```

##### `<rule_group>`

```
# The name of the group. Must be unique within a file.
name: <string>

# How often rules in the group are evaluated.
[ interval: <duration> | default = global.evaluation_interval ]

# Limit the number of alerts an alerting rule and series a recording
# rule can produce. 0 is no limit.
[ limit: <int> | default = 0 ]

rules:
  [ - <rule> ... ]
```

##### `<rule>`

The syntax for recording rules is:
```
# The name of the time series to output to. Must be a valid metric name.
record: <string>

# The PromQL expression to evaluate. Every evaluation cycle this is
# evaluated at the current time, and the result recorded as a new set of
# time series with the metric name as given by 'record'.
expr: <string>

# Labels to add or overwrite before storing the result.
labels:
  [ <labelname>: <labelvalue> ]
```

The syntax for alerting rules is:
```
# The name of the alert. Must be a valid label value.
alert: <string>

# The PromQL expression to evaluate. Every evaluation cycle this is
# evaluated at the current time, and all resultant time series become
# pending/firing alerts.
expr: <string>

# Alerts are considered firing once they have been returned for this long.
# Alerts which have not yet fired for long enough are considered pending.
[ for: <duration> | default = 0s ]

# How long an alert will continue firing after the condition that triggered it
# has cleared.
[ keep_firing_for: <duration> | default = 0s ]

# Labels to add or overwrite for each alert.
labels:
  [ <labelname>: <tmpl_string> ]

# Annotations to add to each alert.
annotations:
  [ <labelname>: <tmpl_string> ]
```

### Alerting Rules

Alerting rules allow you to define alert conditions based on Prometheus expression language expressions and to send notifications about firing alerts to an external service.
Whenever the alert expression results in one or more vector elements at a given point in time, the alert counts as active for these elements' label sets.

#### Defining alerting rules

Alerting rules are configured in Prometheus in the same way as [recording rules](https://prometheus.io/docs/prometheus/2.47/configuration/recording_rules/).

An example rules file with an alert would be:
```yaml
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```
The optional `for` clause causes Prometheus to wait for a certain duration between first encountering a new expression output vector element and counting an alert as firing for this element.
In this case, Prometheus will check that the alert continues to be active during each evaluation for 10 minutes before firing the alert.
Elements that are active, but not firing yet, are in the pending state.
Alerting rules without the `for` clause will become active on the first evaluation.

The `labels` clause allows specifying a set of additional labels to be attached to the alert.
Any existing conflicting labels will be overwritten. The label values can be templated.

The `annotations` clause specifies a set of informational labels that can be used to store longer additional information such as alert descriptions or runbook links.
The annotation values can be templated.

##### Templating

Label and annotation values can be templated using [console templates](https://prometheus.io/docs/visualization/consoles).
The `$labels` variable holds the label key/value pairs of an alert instance.
The configured external labels can be accessed via the `$externalLabels` variable.
The `$value` variable holds the evaluated value of an alert instance.

#### Inspecting alerts during runtime

To manually inspect which alerts are active (pending or firing), navigate to the "Alerts" tab of your Prometheus instance.
This will show you the exact label sets for which each defined alert is currently active.

For pending and firing alerts, Prometheus also stores synthetic time series of the form `ALERTS{alertname="<alert name>", alertstate="<pending or firing>", <additional alert labels>}`.
The sample value is set to `1` as long as the alert is in the indicated active (pending or firing) state, and the series is marked stale when this is no longer the case.

## Visualization

## Instrumenting

## Operating

## Alerting

### Configuration

Alertmanager is configured via command-line flags and a configuration file.
While the command-line flags configure immutable system parameters, the configuration file defines inhibition rules, notification routing and notification receivers.

The visual editor can assist in building routing trees.

To view all available command-line flags, run `alertmanager -h`.

Alertmanager can reload its configuration at runtime. If the new configuration is not well-formed, the changes will not be applied and an error is logged.
A configuration reload is triggered by sending a `SIGHUP` to the process or sending a HTTP POST request to the `/-/reload` endpoint.

#### Configuration file

To specify which configuration file to load, use the `--config.file` flag.
```
./alertmanager --config.file=alertmanager.yml
```

##### `<route>`

A route block defines a node in a routing tree and its children.
Its optional configuration parameters are inherited from its parent node if not set.

Every alert enters the routing tree at the configured top-level route, which must match all alerts (i.e. not have any configured matchers).
It then traverses the child nodes.
If `continue` is set to false, it stops after the first matching child.
If `continue` is true on a matching node, the alert will continue matching against subsequent siblings.
If an alert does not match any children of a node (no matching child nodes, or none exist), the alert is handled based on the configuration parameters of the current node.
```yaml
[ receiver: <string> ]
# The labels by which incoming alerts are grouped together. For example,
# multiple alerts coming in for cluster=A and alertname=LatencyHigh would
# be batched into a single group.
#
# To aggregate by all possible labels use the special value '...' as the sole label name, for example:
# group_by: ['...']
# This effectively disables aggregation entirely, passing through all
# alerts as-is. This is unlikely to be what you want, unless you have
# a very low alert volume or your upstream notification system performs
# its own grouping.
[ group_by: '[' <labelname>, ... ']' ]

# Whether an alert should continue matching subsequent sibling nodes.
[ continue: <boolean> | default = false ]

# DEPRECATED: Use matchers below.
# A set of equality matchers an alert has to fulfill to match the node.
match:
  [ <labelname>: <labelvalue>, ... ]

# DEPRECATED: Use matchers below.
# A set of regex-matchers an alert has to fulfill to match the node.
match_re:
  [ <labelname>: <regex>, ... ]

# A list of matchers that an alert has to fulfill to match the node.
matchers:
  [ - <matcher> ... ]

# How long to initially wait to send a notification for a group
# of alerts. Allows to wait for an inhibiting alert to arrive or collect
# more initial alerts for the same group. (Usually ~0s to few minutes.)
[ group_wait: <duration> | default = 30s ]

# How long to wait before sending a notification about new alerts that
# are added to a group of alerts for which an initial notification has
# already been sent. (Usually ~5m or more.)
[ group_interval: <duration> | default = 5m ]

# How long to wait before sending a notification again if it has already
# been sent successfully for an alert. (Usually ~3h or more).
[ repeat_interval: <duration> | default = 4h ]

# Times when the route should be muted. These must match the name of a
# mute time interval defined in the mute_time_intervals section.
# Additionally, the root node cannot have any mute times.
# When a route is muted it will not send any notifications, but
# otherwise acts normally (including ending the route-matching process
# if the `continue` option is not set.)
mute_time_intervals:
  [ - <string> ...]

# Times when the route should be active. These must match the name of a
# time interval defined in the time_intervals section. An empty value
# means that the route is always active.
# Additionally, the root node cannot have any active times.
# The route will send notifications only when active, but otherwise
# acts normally (including ending the route-matching process
# if the `continue` option is not set).
active_time_intervals:
  [ - <string> ...]

# Zero or more child routes.
routes:
  [ - <route> ... ]
```

##### `<matcher>`

A matcher is a string with a syntax inspired by PromQL and OpenMetrics.
The syntax of a matcher consists of three tokens:
* A valid Prometheus label name.
* One of `=`, `!=`, `=~`, or `!~`. `=` means equals, `!=` means that the strings are not equal, `=~` is used for equality of regex expressions and `!~` is used for un-equality of regex expressions. They have the same meaning as known from PromQL selectors.
* A UTF-8 string, which may be enclosed in double quotes. Before or after each token, there may be any amount of whitespace.

The 3rd token may be the empty string.
Within the 3rd token, OpenMetrics escaping rules apply: `\"` for a double-quote, `\n` for a line feed, `\\` for a literal backslash.
Unescaped `"` must not occur inside the 3rd token (only as the 1st or last character).
However, literal line feed characters are tolerated, as are single `\` characters not followed by `\`, `n`, or `"`.
They act as a literal backslash in that case.

Matchers are ANDed together, meaning that all matchers must evaluate to "true" when tested against the labels on a given alert.
For example, an alert with these labels:
```
{"alertname":"Watchdog","severity":"none"}
```
would NOT match this list of matchers:
```yaml
matchers:
  - alertname = Watchdog
  - severity =~ "warning|critical"
```
In the configuration, multiple matchers are combined in a YAML list.
However, it is also possible to combine multiple matchers within a single YAML string, again using syntax inspired by PromQL.
In such a string, a leading `{` and/or a trailing `}` is optional and will be trimmed before further parsing.
Individual matchers are separated by commas outside of quoted parts of the string.
Those commas may be surrounded by whitespace.
Parts of the string inside unescaped double quotes `"…"` are considered quoted (and commas don't act as separators there).
If double quotes are escaped with a single backslash `\`, they are ignored for the purpose of identifying quoted parts of the input string.
If the input string, after trimming the optional trailing `}`, ends with a comma, followed by optional whitespace, this comma and whitespace will be trimmed.

### Exporters and Integrations

There are a number of libraries and servers which help in exporting existing metrics from third-party systems as Prometheus metrics.
This is useful for cases where it is not feasible to instrument a given system with Prometheus metrics directly (for example, HAProxy or Linux system stats).

#### Third-party exporters

##### Other monitoring systems

* [StatsD exporter](https://github.com/prometheus/statsd_exporter) (official)
