---
tags:
  - aks
---
## The system namespace

`kubectl --namespace kube-system get pods` will list all pods running int he System namespace. 
This workload is provided by Microsoft and is beyond customers' control. Any attempts to modify the workload's settings will be overwritten by Microsoft's automation.

This is troublesome, because some of the workload's QoS classes are `Burstable` (see: [[K8S QoS Classes]]). Which may destabilize the customers' workload or make it impossible to schedule.

> **Recommendation**
> Run the contents of the `system` namespace,  in a logically separated nodepool. Additional workloads such as: prometheus, service meshes, keda, cert-manager, argocd, kyverno and any other core platform workloads should be executed there. 
## Example: Nodepool separation

> **Discussion**
> Examine and discuss the setup below. 

```json
"agentPoolProfiles": [
    {
        "name": "sysd4dsv5",
        "count": 3,
        "vmSize": "Standard_D4ds_v5",
        "osDiskSizeGB": 128,
        "osDiskType": "Ephemeral",
        "osType": "Linux",
        "mode": "System",
        "vnetSubnetID": "<SYSTEM_SUBNET_ID>",
        "maxPods": "40",
        "enableAutoscaling": "false",
        "orchestratorVersion": "1.33.0",
        "nodeLabels": {
			"my-company.com/nodepool": "system",
			"my-company.com/project": "myproject",
			"my-project.com/environment": "dev"
        },
        "nodeTaints": [
        ],
        "availabilityZones": [
            "1",
            "2",
            "3"
        ],
        "type": "VirtualMachineScaleSets"
    },
    {
        "name": "maind4dsv5",
        "count": 3,
        "vmSize": "Standard_D4ds_v5",
        "osDiskSizeGB": 128,
        "osDiskType": "Ephemeral",
        "osType": "Linux",
        "mode": "User",
        "vnetSubnetID": "<MAIN_SUBNET_ID>",
        "maxPods": "40",
        "enableAutoscaling": "true",
        "orchestratorVersion": "1.33.0",
        "nodeLabels": {
			"my-company.com/nodepool": "main",
			"my-company.com/project": "myproject",
			"my-project.com/environment": "dev"
        },
        "nodeTaints": [
        ],
        "maxCount": 7,
        "minCount": 3,
        "availabilityZones": [
                  "1",
                  "2",
                  "3"
        ],
        "type": "VirtualMachineScaleSets"
    }
]
```


