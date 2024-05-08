<h1 align="center" style="border-bottom: none">
   Monitoring
</h1>
<hr><br>
<div align="center">

[![CI](https://github.com/prometheus/prometheus/actions/workflows/ci.yml/badge.svg)](https://github.com/prometheus/prometheus/actions/workflows/ci.yml)
![Docker Repository on Quay](https://quay.io/repository/prometheus/prometheus/status)
![Docker Pulls](https://img.shields.io/docker/pulls/prom/prometheus.svg?maxAge=604800)

</div>


Service monitoring involves the continuous tracking and assessment of various aspects of a service to ensure its optimal performance, availability, and reliability. It typically includes the monitoring of key performance indicators (KPIs), such as response time, uptime, error rates, and resource utilization. The primary goal of service monitoring is to proactively identify issues or potential failures, allowing for timely intervention and resolution to minimize downtime and ensure a positive user experience.

## Prometheus
Prometheus, a [Cloud Native Computing Foundation](https://cncf.io/) project, is a systems and service monitoring system. It collects metrics
from configured targets at given intervals, evaluates rule expressions,
displays the results, and can trigger alerts when specified conditions are observed.

## Architecture overview

![Architecture overview](images/architecture.svg)

## Install

There are various ways of installing Prometheus.

### Docker images

Docker images are available on [Quay.io](https://quay.io/repository/prometheus/prometheus) or [Docker Hub](https://hub.docker.com/r/prom/prometheus/).

You can launch a Prometheus container for trying it out with

```bash
docker run --name prometheus -d -p 9090:9090 -v /prometheus/:/etc/prometheus/  prom/prometheus
```

Prometheus will now be reachable at <http://localhost:9090/>.


## Monitoring .Net Core Application With Prometheus
We need to install a few NuGet packages, please run the below commands in the package manager console.

```bash
dotnet add package prometheus-net

dotnet add package prometheus-net.AspNetCore

dotnet add package prometheus-net.AspNetCore.HealthChecks
```

We need this packages to create a new metrics or to expose default metrics available with the package.

Once package are installed, we need to integrate Prometheus Exporter to ASP. Net Core Middleware. We will modify the Startup.cs to configure our middleware to export metrics. 

we add the following line at the top of the file.

```C#
using Prometheus
```

Next, we are going to add a method which is made available through the .NET Core Prometheus package. This will expose the Prometheus metrics on the `/metrics` URL. Add `endpoints.MapMetrics()` to the endpoint configuration inside `app.UseEndpoints`.


```C#
public void Configure(IApplicationBuilder app, ...)
{
    
    app.UseHttpMetrics( options =>
    {
        options.AddCustomLabel("host", 
            context => context.Request.Host.Host);
    });

    app.UseEndpoints(endpoints =>
    {
        // ...
        endpoints.MapMetrics();
    })
}
```

Now go to `Prometheus` config file and create a job for scrape your `.Net Core Application`.

```bash
  - job_name: application_metrics
    scrape_interval: 15s
    tls_config:
      insecure_skip_verify: false
    static_configs:
      - targets: [<<APPLICATION-HOSTNAME>>]
```


## Redis Metrics Exporter
You can run it like this:

```bash
docker run -d --name redis_exporter -p 9121:9121 oliver006/redis_exporter
```

Now go to `Prometheus` config file and create a job for scrape your     `Redis`.

```bash
  - job_name: 'redis_exporter_metrics'
    static_configs:
      - targets:
        - redis://<<REDIS-HOSTNAME:6379>>
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <<REDIS-EXPORTER-HOSTNAME>>:9121
```

Now we are done, we are successfully able to export metrics to Prometheus and Visualize in the Grafana Dashboard.