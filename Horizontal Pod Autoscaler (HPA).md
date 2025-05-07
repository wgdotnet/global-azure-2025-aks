```d2
hpa: Horizontal Pod Autoscaler {

}

deployment: RC/Deployment {
  scale: Scale
}

pod1: Pod1
pod2: Pod2
pod3: Pod3

hpa -> deployment.scale
deployment.scale -> pod1
deployment.scale -> pod2
deployment.scale -> pod3
```

**Source**: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale

> ⚠️ HPA works with simple metrics like CPU or memory utilization only.

## How does HPA work?

$$
desiredReplicas = ceil[currentReplicas * \frac{currentMetricValue}{ desiredMetricValue}]
$$

**Example**

$$currentValue = 200m$$
$$desiredValue=100m$$
$$currentReplicas = 2$$
$$desiredReplicas = ceil[2 * \frac{200m}{100m}] = 2 * 2 = 4$$
