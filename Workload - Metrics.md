---
tags:
  - workload
---
From the perspective of workload's scalability the following metrics are important:

## CPU utilization
This is self explanatory. CPU utilization is one of the most popular metrics used for autoscaling a workload, especially in simple HPA scenarios
## Memory utilization
This is self explanatory. CPU utilization is one of the most popular metrics used for autoscaling a workload, especially in simple HPA scenarios
## Requests per second
RPS is a good indicator of the workload's performance, however if there are multiple endpoints presence, there's a risk that each endpoint performs differently, therefore this metrics should be used with caution for autoscaling
## Messages per second
MPS is a good indicator of the workload's performance, there's a similar risk to the one defined in RPS, if the workload is handling multiple message types, there may be performance differences, therefore this metrics should be used with caution for autoscaling
## Time to become ready
Once a new replica of the workload starts, it is important to understand how long does it take to start-up. Situations like a single Azure KeyVault instance used by hundreds of starting pods, may lead to pods getting stuck while starting up, when their processing power is needed. Being aware of the workload's time to become ready is a key metric that needs to be observed to provide reliability.