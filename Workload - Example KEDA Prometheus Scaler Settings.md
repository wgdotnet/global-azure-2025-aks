---
tags:
  - workload
---
## Prometheus Scaler Example

> The trigger will fire, when sum of percentages of cpu requests for pods with matching name, will exceed 80%

```yaml
triggers:
- metadata:
	query: |-
		sum((quantile_over_time(0.99,
			max by (pod)(
				rate(
					container_cpu_usage_seconds_total{container="my-container", namespace="my-namespace"}[5m]
				)
			)[5m:1m]
		)) / max by (pod)(kube_pod_container_resource_requests{resource="cpu", namespace="my-namespace", container="my-container"}) * 100)
	serverAddress: 'http://prometheus-server-service.monitoring.svc.cluster.local:9090'
	threshold: '80'
	type: prometheus
```

### Breakdown

#### `container_cpu_usage_seconds_total{container="my-container", namespace="my-namespace"}`

This metric tracks the cumulative CPU seconds consumed by the container "my-container" in the namespace "my-namespace".
#### `rate(...[5m])`
Calculates the per-second average CPU usage rate over the last 5 minutes for each time series (each pod's container).
#### `max by (pod)(rate(...))`
For each pod, selects the maximum CPU usage rate among all containers matching the filter
#### `[5m:1m]`
This is a range vector selector with a step of 1 minute over the last 5 minutes, producing a series of CPU usage rates sampled every minute for the last 5 minutes.
#### `quantile_over_time(0.99, ...)`
Computes the 99th percentile (0.99 quantile) of the CPU usage rates over the last 5 minutes sampled at 1-minute intervals, per pod. This captures near-peak CPU usage, smoothing out spikes and noise
#### `max by (pod)(kube_pod_container_resource_requests{resource="cpu", namespace="my-namespace", container="my-container"})`
Retrieves the CPU resource requests (in cores) for each pod's container. This represents the guaranteed CPU allocation requested for that container
#### Division and multiplication by 100
Divides the 99th percentile CPU usage rate by the CPU request to get usage as a fraction of the requested CPU, then multiplies by 100 to convert it to a percentage.
#### `sum(...)`
Sums the resulting CPU usage percentages across all pods in the namespace, giving an aggregate percentage of CPU request utilization by all pods running "my-container" in "my-namespace".

## Tuning

The trigger can be made more agressive by modifying the width of the sliding window. This can prove to be useful in case of front-line services, which are handling direct customer traffic and need to scale up more aggressively.
