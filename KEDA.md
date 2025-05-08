---
tags:
  - k8s/autoscaling
---

**Source:** https://keda.sh

**KEDA** is Kubernetes Event Driven Autoscaler.

## How does KEDA work

- **KEDA Operator** keeps track of event sources and changes the number of app instances up or down, depending on the demand.
- **Metrics Server** provides external metrics to Kubernetes’ HPA so it can make scaling decisions.
- **Scalers** connect to event sources like message queues or databases, pulling data on current usage or load.
- **Custom Resource Definitions (CRDs)** define how your apps should scale based on triggers like queue length or API request rates.

![[keda-architecture.png]]

External events, like an increase in queue messages, trigger the **ScaledObject**, which sets the scaling rules. The **Controller** handles the scaling, while the **Metrics Adapter** sends data to the HPA for real-time scaling decisions. **Admission Webhooks** ensure your configuration is correct and won’t cause problems.

This setup lets Kubernetes adjust resources automatically based on what’s happening outside, keeping things efficient and responsive.

## KEDA Custom Resource Definitions (CRDs)

### ScaledObject

**Source:** https://keda.sh/docs/2.14/concepts/scaling-deployments/#scaledobject-spec

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: {scaled-object-name}
  annotations:
    scaledobject.keda.sh/transfer-hpa-ownership: "true"     # Optional. Use to transfer an existing HPA ownership to this ScaledObject
    validations.keda.sh/hpa-ownership: "true"               # Optional. Use to disable HPA ownership validation on this ScaledObject
    autoscaling.keda.sh/paused: "true"                      # Optional. Use to pause autoscaling of objects explicitly
spec:
  scaleTargetRef:
    apiVersion:    {api-version-of-target-resource}         # Optional. Default: apps/v1
    kind:          {kind-of-target-resource}                # Optional. Default: Deployment
    name:          {name-of-target-resource}                # Mandatory. Must be in the same namespace as the ScaledObject
    envSourceContainerName: {container-name}                # Optional. Default: .spec.template.spec.containers[0]
  pollingInterval:        30                                # Optional. Default: 30 seconds
  initialCooldownPeriod:  0                                 # Optional. Default: 0 seconds
  cooldownPeriod:         300                               # Optional. Default: 300 seconds
  idleReplicaCount:       0                                 # Optional. Default: ignored, must be less than minReplicaCount
  minReplicaCount:        1                                 # Optional. Default: 0
  maxReplicaCount:        100                               # Optional. Default: 100
  fallback:                                                 # Optional. Section to specify fallback options
    failureThreshold: 3                                     # Mandatory if fallback section is included
    replicas: 6                                             # Mandatory if fallback section is included
  advanced:                                                 # Optional. Section to specify advanced options
    restoreToOriginalReplicaCount: true/false               # Optional. Default: false
    horizontalPodAutoscalerConfig:                          # Optional. Section to specify HPA related options
      name: {name-of-hpa-resource}                          # Optional. Default: keda-hpa-{scaled-object-name}
      behavior:                                             # Optional. Use to modify HPA's scaling behavior
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
          - type: Percent
            value: 100
            periodSeconds: 15
  triggers:
  # {list of triggers to activate scaling of the target resource}
```

### ScaledJob

**Source:** https://keda.sh/docs/2.14/concepts/scaling-jobs/

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: {scaled-job-name}
  labels:
    my-label: {my-label-value}                # Optional. ScaledJob labels are applied to child Jobs
  annotations:
    autoscaling.keda.sh/paused: true          # Optional. Use to pause autoscaling of Jobs
    my-annotation: {my-annotation-value}      # Optional. ScaledJob annotations are applied to child Jobs
spec:
  jobTargetRef:
    parallelism: 1                            # [max number of desired pods](https://kubernetes.io/docs/concepts/workloads/controllers/job/#controlling-parallelism)
    completions: 1                            # [desired number of successfully finished pods](https://kubernetes.io/docs/concepts/workloads/controllers/job/#controlling-parallelism)
    activeDeadlineSeconds: 600                #  Specifies the duration in seconds relative to the startTime that the job may be active before the system tries to terminate it; value must be positive integer
    backoffLimit: 6                           # Specifies the number of retries before marking this job failed. Defaults to 6
    template:
      # describes the [job template](https://kubernetes.io/docs/concepts/workloads/controllers/job)
  pollingInterval: 30                         # Optional. Default: 30 seconds
  successfulJobsHistoryLimit: 5               # Optional. Default: 100. How many completed jobs should be kept.
  failedJobsHistoryLimit: 5                   # Optional. Default: 100. How many failed jobs should be kept.
  envSourceContainerName: {container-name}    # Optional. Default: .spec.JobTargetRef.template.spec.containers[0]
  minReplicaCount: 10                         # Optional. Default: 0
  maxReplicaCount: 100                        # Optional. Default: 100
  rolloutStrategy: gradual                    # Deprecated: Use rollout.strategy instead (see below).
  rollout:
    strategy: gradual                         # Optional. Default: default. Which Rollout Strategy KEDA will use.
    propagationPolicy: foreground             # Optional. Default: background. Kubernetes propagation policy for cleaning up existing jobs during rollout.
  scalingStrategy:
    strategy: "custom"                        # Optional. Default: default. Which Scaling Strategy to use. 
    customScalingQueueLengthDeduction: 1      # Optional. A parameter to optimize custom ScalingStrategy.
    customScalingRunningJobPercentage: "0.5"  # Optional. A parameter to optimize custom ScalingStrategy.
    pendingPodConditions:                     # Optional. A parameter to calculate pending job count per the specified pod conditions
      - "Ready"
      - "PodScheduled"
      - "AnyOtherCustomPodCondition"
    multipleScalersCalculation : "max" # Optional. Default: max. Specifies how to calculate the target metrics when multiple scalers are defined.
  triggers:
  # {list of triggers to create jobs}
```
