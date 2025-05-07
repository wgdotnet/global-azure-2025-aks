

```d2
vars: {
	d2-config: {
		layout-engine: elk
	}
}

azure-vnet-hub: hub {
	azure-dns: privatelink.*.azmk8s.io
	azure-vnet-gateway: gateway
	azure-firewall: firewall
}

azure-vnet-spoke: spoke {
	azure-vnet-nsg: nsg {
		azure-aks-cluster: cluster
	}
}

azure-vnet-hub -- azure-vnet-spoke

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

# nsg
**.azure-vnet-nsg*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/network/NetworkSecurityGroup.svg
**.azure-vnet-nsg*.style.fill: "#e1d5e7"
**.azure-vnet-nsg*.style.font-color: purple
**.azure-vnet-nsg*.style.stroke: transparent

# dns
**.azure-dns*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/network/DnsZone.svg

# aks
**.azure-aks*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/compute/AKS.svg

# gateway
**.azure-vnet-gateway*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/network/Gateway.svg

# firewall
**.azure-firewall*.icon: https://github.com/cloud-tek/icons/raw/refs/heads/main/azure/network/Firewall.svg
```
The most critical choices when designing the cluster are associated with selecting the appropriate network model. The choice may not be immediately clear. Before proceeding with any further considerations, the following questions need to be asked:

- What is the available network architecture?
	- Is HUB & Spoke architecture available?
- Are we designing a public or a private cluster?
	- This directly impacts the accessibility of the cluster's API endpoint
- How are we going to control egress traffic from the cluster?
	- Are we going to use a UDR?
	- Are we gogin to use Network policies?
- What [[AKS Networking Model]] are we going to choose?