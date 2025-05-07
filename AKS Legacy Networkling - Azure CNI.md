```d2
vars: {
	d2-config: {
		layout-engine: elk
	}
}


azure-vnet-spoke: vnet {
	azure-vnet-subnet: subnet {
		azure-vm: node
		k8s-pod-1: pod
		k8s-pod-2: pod
		k8s-pod-3: pod
		k8s-svc-1: service
	}
}

# all
**.*.style.stroke: transparent
**.*.style.fill: transparent
**.*.label.near: bottom-center

# vnet
azure-vnet*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/network/VirtualNetwork.svg
azure-vnet*.style.stroke: green
azure-vnet*.style.font-color: green
azure-vnet*.label.near: bottom-right

# subnet
**.azure-vnet-subnet*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/network/Subnet.svg
**.azure-vnet-subnet*.style.stroke: green
**.azure-vnet-subnet*.style.font-color: green
**.azure-vnet*.label.near: bottom-right

# vm
**.azure-vm*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/compute/VirtualMachine.svg

# pod
**.k8s-pod*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/resources/labeled/pod.svg

# service
**.k8s-svc*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/resources/labeled/svc.svg
```
>⚠️ **Important**
> On **31st March 2028**, azure-cni networking for Azure Kubernetes Service (AKS) will be retired.

**Source:** https://learn.microsoft.com/en-us/azure/aks/concepts-network-legacy-cni

- Every pod gets an IP address from the subnet and can be accessed directly. Systems in the same virtual network as the AKS cluster see the pod IP as the source address for any traffic from the pod. 
- Systems outside the AKS cluster virtual network see the node IP as the source address for any traffic from the pod. These IP addresses must be unique across your network space and must be planned in advance. 
- Each node has a configuration parameter (POD LIMIT) for the maximum number of pods that it supports. **==The equivalent number of IP addresses per node are then reserved up front for that node==**.
	- ==This is very important during cluster upgrades, as new nodes reserve (POD LIMIT + 1) addresses as soon as they are materialized==

> ==This approach requires more planning, and often leads to IP address exhaustion or the need to rebuild clusters in a larger subnet as your application demands grow.==

With Azure CNI Node Subnet, each pod receives an IP address in the IP subnet and can communicate directly with other pods and services. Your clusters can be as large as the IP address range you specify. However, you must plan the IP address range in advance, and all the IP addresses are consumed by the AKS nodes based on the maximum number of pods they can support. Advanced network features and scenarios such as [virtual nodes](https://learn.microsoft.com/en-us/azure/aks/virtual-nodes-cli) or Network Policies (either Azure or Calico) are supported with Azure CNI.