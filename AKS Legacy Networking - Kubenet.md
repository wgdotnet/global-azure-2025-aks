---
tags:
  - network
---

```d2
vars: {
	d2-config: {
		layout-engine: elk
	}
}

azure-vnet: vnet {
	azure-vnet-subnet: subnet {
		azure-vm-node1: node
		azure-vm-node2: node
		azure-vm-node3: node
	}
}

k8s-local-ip-address-pool: local ip address pool {
	k8s-pod-1: pod
	k8s-pod-2: pod
	k8s-pod-3: pod
	k8s-pod-4: pod
	k8s-pod-5: pod
	k8s-pod-6: pod
}

azure-vnet.azure-vnet-subnet.azure-vm-node1 -- k8s-local-ip-address-pool.k8s-pod-1
azure-vnet.azure-vnet-subnet.azure-vm-node1 -- k8s-local-ip-address-pool.k8s-pod-2
azure-vnet.azure-vnet-subnet.azure-vm-node2 -- k8s-local-ip-address-pool.k8s-pod-3
azure-vnet.azure-vnet-subnet.azure-vm-node2 -- k8s-local-ip-address-pool.k8s-pod-4
azure-vnet.azure-vnet-subnet.azure-vm-node3 -- k8s-local-ip-address-pool.k8s-pod-5
azure-vnet.azure-vnet-subnet.azure-vm-node3 -- k8s-local-ip-address-pool.k8s-pod-6

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

# k8s-local-ip-address-pool
k8s-local-ip-address-pool.style.stroke: gray
k8s-local-ip-address-pool.style.stroke-dash: 3
k8s-local-ip-address-pool.style.fill: transparent
k8s-local-ip-address-pool.label.near: bottom-right

# pod
**.k8s-pod*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/resources/labeled/pod.svg

# service
**.k8s-svc*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/resources/labeled/svc.svg
```
> ⚠️ **Important**
> On **31st March 2028**, kubenet networking for Azure Kubernetes Service (AKS) will be retired.

**Source:** https://learn.microsoft.com/en-us/azure/aks/configure-kubenet

AKS clusters use kubenet and create an Azure virtual network and subnet for you by default. With kubenet, nodes get an IP address from the Azure virtual network subnet. Pods receive an IP address from a logically different address space to the Azure virtual network subnet of the nodes. Network address translation (NAT) is then configured so the pods can reach resources on the Azure virtual network. The source IP address of the traffic is NAT'd to the node's primary IP address. This approach greatly reduces the number of IP addresses you need to reserve in your network space for pods to use.

(...)

With _kubenet_, only the nodes receive an IP address in the virtual network subnet. Pods can't communicate directly with each other. Instead, User Defined Routing (UDR) and IP forwarding handle connectivity between pods across nodes. ==UDRs and IP forwarding configuration is created and maintained by the AKS service by default==, but you can [bring your own route table for custom route management](https://learn.microsoft.com/en-us/azure/aks/configure-kubenet#bring-your-own-subnet-and-route-table-with-kubenet) if you want.

(...)

==Azure supports a maximum of _400_ routes in a UDR, so you can't have an AKS cluster larger than 400 nodes==

**When to Use Kubenet:**

- When IP address space is constrained.
- When advanced Azure networking features are not required.
- For smaller clusters or development environments.
    

**When Not to Use Kubenet:**

- When you need direct pod IP integration with Azure VNet.
- When you require Azure network policies, virtual nodes, or support for Windows nodes.