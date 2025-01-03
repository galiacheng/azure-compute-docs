---
title: Explore Azure Hybrid Benefit for Windows VMs
description: Learn how to maximize your Windows Software Assurance benefits to bring on-premises licenses to Azure.
ms.service: azure-virtual-machines
ms.subservice: billing
ms.collection: windows
ms.topic: how-to
ms.date: 12/02/2024
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.devlang: azurecli
---
# Explore Azure Hybrid Benefit for Windows VMs

 Azure Hybrid Benefit for Windows Server allows you to use your on-premises core licenses for Windows Server licenses to get Windows virtual machines on Azure at a reduced cost. You can use Azure Hybrid Benefit for Windows Server to deploy new virtual machines with Windows OS. This article goes over the steps on how to deploy new VMs with Azure Hybrid Benefit for Windows Server and how you can update existing running VMs. 

To qualify for Azure Hybrid Benefit for Windows Server, you need on-premises core licenses for Windows Server from an applicable program with active Software Assurance or qualifying subscription licenses. Software Assurance and qualifying subscription licenses are only available as part of certain commercial licensing agreements. To learn more about commercial licensing, see [Microsoft Licensing Resources](https://www.microsoft.com/licensing/default). To learn more about Windows Server core licenses, see [Windows Server product licensing](https://www.microsoft.com/licensing/product-licensing/windows-server).

You need a minimum of 8 core licenses (Datacenter or Standard edition) per VM. For example, 8 core licenses are still required if you run a 4-core instance. You may also run instances larger than 8 cores by allocating licenses equal to the core size of the instance. For example, 12 core licenses are required for a 12-core instance. For customers with processor licenses, each processor license is equivalent to 16 core licenses.

> [!IMPORTANT]
> - Workloads using Azure Hybrid Benefit can run only during the Software Assurance or subscription license term. When the Software Assurance or subscription license term approaches expiration, you must either renew your agreement with either Software Assurance or a subscription license, disable the hybrid benefit functionality, or deprovision those workloads that are using Azure Hybrid Benefit.
>
> - The Microsoft Product Terms for your program take precedent over this article. For more information, see [Microsoft Azure Product Terms](https://www.microsoft.com/licensing/terms/productoffering/MicrosoftAzure) and select your program to show the terms.

Using Azure Hybrid Benefit for Windows Server with any VMs running Windows Server OS are now supported in all regions, including VMs with additional software such as SQL Server or third-party marketplace software. 


## Classic VMs

For classic VMs, only deploying new VM from on premises custom images is supported. To take advantage of the capabilities supported in this article, you must first migrate classic VMs to Resource Manager model.

[!INCLUDE [classic-vm-deprecation](../includes/classic-vm-deprecation.md)]
 

## Ways to use Azure Hybrid Benefit for Windows Server
There are a few ways to use Windows virtual machines with the Azure Hybrid Benefit:

* You can deploy VMs from one of the provided  Windows Server images on the Azure Marketplace
* You can  upload a custom VM and deploy using a Resource Manager template or Azure PowerShell
* You can toggle and convert existing VM between running with Azure Hybrid Benefit or pay on-demand cost for Windows Server
* You can also apply Azure Hybrid Benefit for Windows Server on virtual machine scale set as well


## Create a VM with Azure Hybrid Benefit for Windows Server
All Windows Server OS based images are supported for Azure Hybrid Benefit for Windows Server. You can use Azure platform support images or upload your own custom Windows Server images. 

### Portal
To create a VM with Azure Hybrid Benefit for Windows Server, scroll to the bottom of the **Basics** tab during the creation process and under **Licensing** check the box to use an existing Windows Server license. 

### PowerShell

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -ImageName "Win2016Datacenter" `
    -LicenseType "Windows_Server"
```

### CLI
```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --license-type Windows_Server
```

### Template
Within your Resource Manager templates, an additional parameter `licenseType` must be specified. You can read more about [authoring Azure Resource Manager templates](/azure/azure-resource-manager/templates/syntax).

```json
"properties": {
    "licenseType": "Windows_Server",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
}    
```

## Convert an existing VM using Azure Hybrid Benefit for Windows Server
If you have an existing VM that you would like to convert to take advantage of Azure Hybrid Benefit for Windows Server, you can update your VM's license type by following the instructions below.

> [!NOTE]
> Changing the license type on the VM does not cause the system to reboot or cause a service interuption.  It is simply an update to a metadata flag.
> 

### Portal
From portal VM blade, you can update the VM to use Azure Hybrid Benefit by selecting "Configuration" option and toggle the "Azure hybrid benefit" option

### PowerShell
- Convert existing Windows Server VMs to Azure Hybrid Benefit for Windows Server

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "Windows_Server"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
- Convert Windows Server VMs with benefit back to pay-as-you-go

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "None"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
### CLI
- Convert existing Windows Server VMs to Azure Hybrid Benefit for Windows Server

    ```azurecli
    az vm update --resource-group myResourceGroup --name myVM --set licenseType=Windows_Server
    ```

### How to verify your VM is utilizing the licensing benefit
Once you've deployed your VM through either PowerShell, Resource Manager template or portal, you can verify the setting in the following methods.

### Portal
From portal VM blade, you can view the toggle for Azure Hybrid Benefit for Windows Server by selecting "Operating system" tab.

### PowerShell
The following example shows the license type for a single VM
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

Output:
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Server
```

This output contrasts with the following VM deployed without Azure Hybrid Benefit for Windows Server licensing:
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

### CLI
```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVM --query "[?licenseType=='Windows_Server']" -o table
```

> [!NOTE]
> Changing the license type on the VM does not cause the system to reboot or cause a service interuption. It is a metadata licensing flag only.

## List all VMs and virtual machine scale sets with Azure Hybrid Benefit for Windows Server in a subscription
To see and count all virtual machines and virtual machine scale sets deployed with Azure Hybrid Benefit for Windows Server, you can run the following command from your subscription:

### Portal
From the Virtual Machine or Virtual machine scale sets resource blade, you can view a list of all your VM(s) and licensing type by configuring the table column to include "OS licensing benefit". The VM can be under **Azure Hybrid Benefit for Windows**, **Not enabled**, or **Windows client with multi-tenant hosting** states.


### PowerShell
For virtual machines:
```powershell
Get-AzVM | ?{$_.LicenseType -like "Windows_Server"} | select ResourceGroupName, Name, LicenseType
```

For virtual machine scale sets:
```powershell
Get-AzVmss | Select * -ExpandProperty VirtualMachineProfile | ? LicenseType -eq 'Windows_Server' | select ResourceGroupName, Name, LicenseType
```

### CLI
For virtual machines:
```azurecli
az vm list --query "[?licenseType=='Windows_Server']" -o table
```

For virtual machine scale sets:
```azurecli
az vmss list --query "[?virtualMachineProfile.licenseType=='Windows_Server']" -o table
```

## Deploy a Virtual Machine Scale Set with Azure Hybrid Benefit for Windows Server
Within your virtual machine scale set Resource Manager templates, an additional parameter `licenseType` must be specified within your VirtualMachineProfile property. You can do this during create or update for your scale set through ARM template, PowerShell, Azure CLI or REST.

The following example uses ARM template with a Windows Server 2016 Datacenter image:

```json
"virtualMachineProfile": {
    "storageProfile": {
        "osDisk": {
            "createOption": "FromImage"
        },
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2016-Datacenter",
            "version": "latest"
        }
    },
    "licenseType": "Windows_Server",
    "osProfile": {
            "computerNamePrefix": "[parameters('vmssName')]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
    }
}    
```
You can also learn more about how to [Modify a virtual machine scale set](../../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md) for more ways to update your scale set.

## Next steps
- Read more about [How to save money with the Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/)
- Read more about [Frequently asked questions for Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/faq/)
- Learn more about [Azure Hybrid Benefit for Windows Server licensing detailed guidance](/windows-server/get-started/azure-hybrid-benefit)
- Learn more about [Azure Hybrid Benefit for Windows Server and Azure Site Recovery make migrating applications to Azure even more cost-effective](https://azure.microsoft.com/blog/hybrid-use-benefit-migration-with-asr/)
- Learn more about [Windows 10 on Azure with Multitenant Hosting Right](./windows-desktop-multitenant-hosting-deployment.md)
- Learn more about [Using Resource Manager templates](/azure/azure-resource-manager/management/overview)
