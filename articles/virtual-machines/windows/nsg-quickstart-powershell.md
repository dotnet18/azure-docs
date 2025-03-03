---
title: Open ports to a VM using Azure PowerShell 
description: Learn how to open a port / create an endpoint to your VM using Azure PowerShell
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn 
ms.custom: devx-track-azurepowershell

---
# How to open ports and endpoints to a VM using PowerShell

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

[!INCLUDE [virtual-machines-common-nsg-quickstart](../../../includes/virtual-machines-common-nsg-quickstart.md)]

## Quick commands
To create a Network Security Group and ACL rules you need [the latest version of Azure PowerShell installed](/powershell/azure/). You can also [perform these steps using the Azure portal](nsg-quickstart-portal.md).

Log in to your Azure account:

```powershell
Connect-AzAccount
```

In the following examples, replace parameter names with your own values. Example parameter names included *myResourceGroup*, *myNetworkSecurityGroup*, and *myVnet*.

Create a rule with [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig). The following example creates a rule named *myNetworkSecurityGroupRule* to allow *tcp* traffic on port *80*:

```powershell
$httprule = New-AzNetworkSecurityRuleConfig `
    -Name "myNetworkSecurityGroupRule" `
    -Description "Allow HTTP" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority "100" `
    -SourceAddressPrefix "Internet" `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 80
```

Next, create your Network Security group with [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) and assign the HTTP rule you just created as follows. The following example creates a Network Security Group named *myNetworkSecurityGroup*:

```powershell
$nsg = New-AzNetworkSecurityGroup `
    -ResourceGroupName "myResourceGroup" `
    -Location "EastUS" `
    -Name "myNetworkSecurityGroup" `
    -SecurityRules $httprule
```

Now let's assign your Network Security Group to a subnet. The following example assigns an existing virtual network named *myVnet* to the variable *$vnet* with [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork):

```powershell
$vnet = Get-AzVirtualNetwork `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVnet"
```

Associate your Network Security Group with your subnet with [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig). The following example associates the subnet named *mySubnet* with your Network Security Group:

```powershell
$subnetPrefix = $vnet.Subnets|?{$_.Name -eq 'mySubnet'}

Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name "mySubnet" `
    -AddressPrefix $subnetPrefix.AddressPrefix `
    -NetworkSecurityGroup $nsg
```

Finally, update your virtual network with [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) in order for your changes to take effect:

```powershell
Set-AzVirtualNetwork -VirtualNetwork $vnet
```


## More information on Network Security Groups
The quick commands here allow you to get up and running with traffic flowing to your VM. Network Security Groups provide many great features and granularity for controlling access to your resources. You can read more about [creating a Network Security Group and ACL rules here](tutorial-virtual-network.md#secure-network-traffic).

For highly available web applications, you should place your VMs behind an Azure Load Balancer. The load balancer distributes traffic to VMs, with a Network Security Group that provides traffic filtering. For more information, see [How to load balance Linux virtual machines in Azure to create a highly available application](tutorial-load-balancer.md).

## Next steps
In this example, you created a simple rule to allow HTTP traffic. You can find information on creating more detailed environments in the following articles:

* [Azure Resource Manager overview](../../azure-resource-manager/management/overview.md)
* [What is a network security group?](../../virtual-network/network-security-groups-overview.md)
* [Azure Load Balancer Overview](../../load-balancer/load-balancer-overview.md)
