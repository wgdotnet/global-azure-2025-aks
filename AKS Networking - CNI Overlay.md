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

azure-vnet-overlay: Overlay Network {
	k8s-node-1: node1 {
		cni-overlay: CNI Overlay Plugin 
		pod-cidr: "Pod CIDR 10.10.1.0/24" {
			k8s-pod-1: Pod 10.10.1.2
			k8s-pod-2: Pod 10.10.1.3
			k8s-pod-3: Pod 10.10.1.4
		}
	}

	k8s-node-2: node2 {
		cni-overlay: CNI Overlay Plugin
		pod-cidr: "Pod CIDR 10.10.2.0/24" { 
			k8s-pod-1: Pod 10.10.2.2
			k8s-pod-2: Pod 10.10.2.3
			k8s-pod-3: Pod 10.10.2.4
		}
	}
}

azure-vnet: vnet {
	azure-vnet-subnet: subnet {
		azure-vm-node1: node1 192.168.1.4
		azure-vm-node2: node2 192.168.1.5
	}
}


azure-vnet-overlay.k8s-node-1.pod-cidr.k8s-pod-1 -- azure-vnet.azure-vnet-subnet.azure-vm-node1: nat
azure-vnet-overlay.k8s-node-1.pod-cidr.k8s-pod-2 -- azure-vnet.azure-vnet-subnet.azure-vm-node1: nat
azure-vnet-overlay.k8s-node-1.pod-cidr.k8s-pod-3 -- azure-vnet.azure-vnet-subnet.azure-vm-node1: nat
azure-vnet-overlay.k8s-node-2.pod-cidr.k8s-pod-1 -- azure-vnet.azure-vnet-subnet.azure-vm-node2: nat
azure-vnet-overlay.k8s-node-2.pod-cidr.k8s-pod-2 -- azure-vnet.azure-vnet-subnet.azure-vm-node2: nat
azure-vnet-overlay.k8s-node-2.pod-cidr.k8s-pod-3 -- azure-vnet.azure-vnet-subnet.azure-vm-node2: nat

# all
**.*.style.stroke: transparent
**.*.style.fill: transparent
**.*.label.near: bottom-center

# pod-cidr
**.pod-cidr*.label.near: bottom-right
**.pod-cidr*.style.stroke: gray
**.pod-cidr*.style.stroke-dash: 3

# cni-overlay
**.cni-overlay*.style.fill: "#90d5ff"
**.cni-overlay*.style.stroke: "#57b9ff"

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

# k8s-node
**.k8s-node*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/infrastructure_components/labeled/node.svg
**.k8s-node*.style.stroke: gray
**.k8s-node*.style.stroke-dash: 3
**.k8s-node*.style.fill: transparent
**.k8s-node*.label.near: bottom-right

# pod
**.k8s-pod*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/resources/labeled/pod.svg

# service
**.k8s-svc*.icon: https://github.com/kubernetes/community/raw/refs/heads/master/icons/svg/resources/labeled/svc.svg
```

**Source:** https://learn.microsoft.com/en-us/azure/aks/azure-cni-overlay

In Overlay networking, only the Kubernetes cluster nodes are assigned IPs from subnets. Pods receive IPs from a private CIDR provided at the time of cluster creation. Each node is assigned a `/24` address space carved out from the same CIDR. Extra nodes created when you scale out a cluster automatically receive `/24` address spaces from the same CIDR. Azure CNI assigns IPs to pods from this `/24` space.

A separate routing domain is created in the Azure Networking stack for the pod's private CIDR space, which creates an Overlay network for direct communication between pods. There's no need to provision custom routes on the cluster subnet or use an encapsulation method to tunnel traffic between pods, which provides connectivity performance between pods on par with VMs in a VNet. Workloads running within the pods are not even aware that network address manipulation is happening.

Communication with endpoints outside the cluster, such as on-premises and peered VNets, happens using the node IP through NAT. Azure CNI translates the source IP (Overlay IP of the pod) of the traffic to the primary IP address of the VM, which enables the Azure Networking stack to route the traffic to the destination. Endpoints outside the cluster can't connect to a pod directly. You have to publish the pod's application as a Kubernetes Load Balancer service to make it reachable on the VNet.

## Notes

### Max Pods per node

$$
/24 = 2^{(32-24)} = 2^8 = 256
$$
`/24` means 256 IP addresses per node. (250 max pods)

### Pod CIDRs

- The same pod CIDR space can be used on multiple independent AKS clusters in the same VNet.
- Pod CIDR space must not overlap with the cluster subnet range.
- Pod CIDR space must not overlap with directly connected networks (like VNet peering, ExpressRoute, or VPN). If external traffic has source IPs in the pod CIDR range, it needs translation to a non-overlapping IP via SNAT to communicate with the cluster.

> ⚠️ The private CIDR ranges available for the POD CIDR are defined in [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918)

| Min Host    | Max Host        | Prefix         |
| ----------- | --------------- | -------------- |
| 10.0.0.0    | 10.255.255.255  | 10.0.0.0/8     |
| 172.16.0.0  | 172.31.255.255  | 172.16.0.0/12  |
| 192.168.0.0 | 192.168.255.255 | 192.168.0.0/16 |
> ⚠️ If Pod CIDR is not specified, AKS will assign a default CIDR `10.244.0.0/16`, which can be broken down into 256 `/24` segments

$$
2^{(24-16)} = 2^8 = 256
$$
or ...

$$
2^{(32-16)} / 2^{(32-24)} - 66,536 / 256 = 256
$$

See: https://visualsubnetcalc.com/