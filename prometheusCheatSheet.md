## Prometheus Cheat Sheet

#### Instant vector selectors

Instant vector selectors allow the selection of a set of time series and a single sample value for each at a given timestamp (instant): in the simplest form, only a metric name is specified. This results in an instant vector containing elements for all time series that have this metric name.
This example selects all time series that have the http_requests_total metric name:
`http_requests_total`

It is possible to filter these time series further by appending a comma separated list of label matchers in curly braces ({}).
This example selects only those time series with the http_requests_total metric name that also have the job label set to prometheus and their group label set to canary:
`http_requests_total{job="prometheus",group="canary"}`
It is also possible to negatively match a label value, or to match label values against regular expressions. The following label matching operators exist:

    =: Select labels that are exactly equal to the provided string.
    !=: Select labels that are not equal to the provided string.
    =~: Select labels that regex-match the provided string.
    !~: Select labels that do not regex-match the provided string.

For example, this selects all http_requests_total time series for staging, testing, and development environments and HTTP methods other than GET.
`http_requests_total{environment=~"staging|testing|development",method!="GET"}`

----
#### Range Vector Selectors
Range vector literals work like instant vector literals, except that they select a range of samples back from the current instant. Syntactically, a range duration is appended in square brackets ([]) at the end of a vector selector to specify how far back in time values should be fetched for each resulting range vector element.
Time durations are specified as a number, followed immediately by one of the following units:

    s - seconds
    m - minutes
    h - hours
    d - days
    w - weeks
    y - years

* In this example, we select all the values we have recorded within the last 5 minutes for all time series that have the metric name http_requests_total and a job label set to prometheus:
`http_requests_total{job="prometheus"}[5m]`

**Offset modifier:** The offset modifier allows changing the time offset for individual instant and range vectors in a query.

* For example, the following expression returns the value of http_requests_total 5 minutes in the past relative to the current query evaluation time:
    `http_requests_total offset 5m`

Note that the offset modifier always needs to follow the selector immediately, i.e. the following would be correct:
`sum(http_requests_total{method="GET"} offset 5m)`

The same works for range vectors. This returns the 5-minute rate that http_requests_total had a week ago:
`rate(http_requests_total[5m] offset 1w)`

----


* Return all time series with the metric http_requests_total:
    `http_requests_total`

* Return all time series with the metric http_requests_total and the given job and handler labels:

    `http_requests_total{job="apiserver", handler="/api/comments"}`

* Return a whole range of time (in this case 5 minutes) for the same vector, making it a range vector:

    `http_requests_total{job="apiserver", handler="/api/comments"}[5m]`

**NOTE:** An expression resulting in a range vector cannot be graphed directly, but viewed in the tabular ("Console") view of the expression browser.

* Using regular expressions, you could select time series only for jobs whose name match a certain pattern, in this case, all jobs that end with server:

    `http_requests_total{job=~".*server"}`

All regular expressions in Prometheus use RE2 syntax.

* To select all HTTP status codes except 4xx ones, you could run:

    `http_requests_total{status!~"4.."}`

## Subquery:
* Return the 5-minute rate of the http_requests_total metric for the past 30 minutes, with a resolution of 1 minute.

    `rate(http_requests_total[5m])[30m:1m]`

* This is an example of a nested subquery. The subquery for the deriv function uses the default resolution. Note that using subqueries unnecessarily is unwise.

    `max_over_time(deriv(rate(distance_covered_total[5s])[30s:5s])[10m:])`

#### Using functions, operators:
* Return the per-second rate for all time series with the http_requests_total metric name, as measured over the last 5 minutes:

    `rate(http_requests_total[5m])`

* Assuming that the http_requests_total time series all have the labels job (fanout by job name) and instance (fanout by instance of the job), we might want to sum over the rate of all instances, so we get fewer output time series, but still preserve the job dimension:

    `sum by (job) (rate(http_requests_total[5m]))`

* If we have two different metrics with the same dimensional labels, we can apply binary operators to them and elements on both sides with the same label set will get matched and propagated to the output. For example, this expression returns the unused memory in MiB for every instance (on a fictional cluster scheduler exposing these metrics about the instances it runs):

    `(instance_memory_limit_bytes - instance_memory_usage_bytes) / 1024 / 1024`

* The same expression, but summed by application, could be written like this:

    `sum by (app, proc) (instance_memory_limit_bytes - instance_memory_usage_bytes) / 1024 / 1024`

If the same fictional cluster scheduler exposed CPU usage metrics like the following for every instance:
```
instance_cpu_time_ns{app="lion", proc="web", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="elephant", proc="worker", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="turtle", proc="api", rev="4d3a513", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="fox", proc="widget", rev="4d3a513", env="prod", job="cluster-manager"}
```


## Aggregation operators
Like PromQL, LogQL supports a subset of built-in aggregation operators that can be used to aggregate the element of a single vector, resulting in a new vector of fewer elements but with aggregated values:

    sum: Calculate sum over labels
    min: Select minimum over labels
    max: Select maximum over labels
    avg: Calculate the average over labels
    stddev: Calculate the population standard deviation over labels
    stdvar: Calculate the population standard variance over labels
    count: Count number of elements in the vector
    bottomk: Select smallest k elements by sample value
    topk: Select largest k elements by sample value

### Examples
* Get the top 10 applications by the highest log throughput:

    `topk(10,sum(rate({region="us-east1"}[5m])) by (name))`

* We could get the top 3 CPU users grouped by application (app) and process type (proc) like this:

    `topk(3, sum by (app, proc) (rate(instance_cpu_time_ns[5m])))`

* Get the count of logs during the last five minutes, grouping by level:

    `sum(count_over_time({job="mysql"}[5m])) by (level)`

* Assuming this metric contains one time series per running instance, you could count the number of running instances per application like this:

    `count by (app) (instance_cpu_time_ns)`

* Get the rate of HTTP GET requests from NGINX logs:

    `avg(rate(({job="nginx"} |= "GET")[10s])) by (region)`


----
----

### How to form queries:
* All CPU idle seconds: `node_cpu_seconds_total{mode='idle'}` # Instant Vector
* CPU idle seconds for 5min: `node_cpu_seconds_total{mode='idle'}[5m]` # Range Vector
* CPU idle seconds rate: `rate(node_cpu_seconds_total{mode='idle'}[5m])` # Input range vector output instant vector
* CPU idle seconds sum by cpu:
```
sum by (cpu) (rate(node_cpu_seconds_total{mode='idle'}[5m]))
OR
sum (rate(node_cpu_seconds_total{mode='idle'}[5m])) by (cpu)
```
* CPU idle seconds sum by CPU / total CPU seconds by CPU:
```
sum by (cpu) (rate(node_cpu_seconds_total{mode='idle'}[5m])) / sum by (cpu) (rate(node_cpu_seconds_total[5m]))
OR
sum(rate(node_cpu_seconds_total{mode='idle'}[5m])) by (cpu) / sum(rate(node_cpu_seconds_total[5m])) by (cpu)
```
