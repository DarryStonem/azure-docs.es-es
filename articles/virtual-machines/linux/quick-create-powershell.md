---
title: "Inicio rápido de Azure: creación de máquinas virtuales con PowerShell | Microsoft Docs"
description: "Aprenda rápidamente a crear una máquina virtual Linux con PowerShell."
services: virtual-machines-linux
documentationcenter: virtual-machines
author: neilpeterson
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/03/2017
ms.author: nepeters
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: cfbd863ca7e65ddad585d4305d5e24b8f6bb744a
ms.contentlocale: es-es
ms.lasthandoff: 05/03/2017


---

# <a name="create-a-linux-virtual-machine-with-powershell"></a>Creación de una máquina virtual Linux con PowerShell

El módulo de Azure PowerShell se usa para crear y administrar recursos de Azure desde la línea de comandos de PowerShell o en scripts. Esta guía se detalla cómo usar PowerShell para crear y ejecutar máquinas virtuales de Azure con Ubuntu 14.04 LTS.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/en-us/free/?WT.mc_id=A261C142F) antes de empezar.

Asegúrese de tener instalada la versión más reciente del módulo de Azure PowerShell. Para más información, vea [Instalación y configuración de Azure PowerShell](/powershell/azure/overview).

Por último, se tiene que almacenar una clave SSH pública con el nombre *id_rsa.pub* en el directorio *.ssh* de su perfil de usuario de Windows. Si quiere obtener información detallada sobre cómo crear claves SSH para Azure, consulte [este artículo](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="log-in-to-azure"></a>Inicie sesión en Azure.

Inicie sesión en la suscripción de Azure con el comando `Login-AzureRmAccount` y siga las instrucciones de la pantalla.

```powershell
Login-AzureRmAccount
```

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

```powershell
New-AzureRmResourceGroup -Name myResourceGroup -Location eastus
```

## <a name="create-networking-resources"></a>Creación de los recursos de red principales

Cree una red virtual, una subred, una dirección IP pública. Estos recursos se utilizan para proporcionar conectividad de red a la máquina virtual y conectarse a Internet.

```powershell
# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork -ResourceGroupName myResourceGroup -Location eastus `
-Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup -Location eastus `
-AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)"
```

Cree un grupo de seguridad de red y una regla de grupo de seguridad de red. El grupo de seguridad de red protege la máquina virtual con reglas entrantes y salientes. En este caso, se crea una regla de entrada para el puerto 22, que permite las conexiones entrantes SSH. También queremos crear una regla de entrada para el puerto 80, que permita el tráfico web entrante.

```powershell
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleSSH  -Protocol Tcp `
-Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 22 -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
-Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location eastus `
-Name myNetworkSecurityGroup -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

Cree una tarjeta de red con [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) para la máquina virtual. Esta conecta la máquina virtual a una subred, un grupo de seguridad de red y una dirección IP pública.

```powershell
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location eastus `
-SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-virtual-machine"></a>Create virtual machine

Cree una configuración de máquina virtual. Esta configuración incluye los ajustes que se usan al implementar la máquina virtual como una imagen de máquina virtual, el tamaño y la configuración de autenticación.

```powershell
# Define a credential object
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration
$vmConfig = New-AzureRmVMConfig -VMName myVM -VMSize Standard_D1 | `
Set-AzureRmVMOperatingSystem -Linux -ComputerName myVM -Credential $cred -DisablePasswordAuthentication | `
Set-AzureRmVMSourceImage -PublisherName Canonical -Offer UbuntuServer -Skus 14.04.2-LTS -Version latest | `
Add-AzureRmVMNetworkInterface -Id $nic.Id

# Configure SSH Keys
$sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"
Add-AzureRmVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"
```

Cree la máquina virtual con [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm).

```powershell
New-AzureRmVM -ResourceGroupName myResourceGroup -Location eastus -VM $vmConfig
```

## <a name="connect-to-virtual-machine"></a>Conexión a la máquina virtual

Una vez finalizada la implementación, cree una conexión SSH con la máquina virtual.

Use el comando [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) para devolver la dirección IP pública de la máquina virtual.

```powershell
Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress
```

En un sistema con SSH instalado, usa el comando siguiente para conectarse a la máquina virtual. Si trabaja en Windows, puede usar [Putty](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-ssh-from-windows?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-private-key-for-putty) para crear la conexión. 

```bash 
ssh <Public IP Address>
```

Cuando se le solicite, el nombre de usuario de inicio de sesión es *azureuser*. Si se ha escrito una frase de contraseña al crear las claves SSH, debe escribirla también.


## <a name="install-nginx"></a>Instalación de NGINX

Use el siguiente script de bash para actualizar los orígenes de paquetes e instalar el paquete NGINX más reciente. 

```bash 
#!/bin/bash

# update package source
apt-get -y update

# install NGINX
apt-get -y install nginx
```

## <a name="view-the-ngix-welcome-page"></a>Página principal de NGIX

Con NGINX instalado y el puerto 80 abierto en la máquina virtual desde Internet, puede usar el explorador web que elija para ver la página principal de NGINX. Asegúrese de utilizar la dirección IP pública que ha anotado antes para visitar la página predeterminada. 

![Sitio predeterminado de NGINX](./media/quick-create-cli/nginx.png) 
## <a name="delete-virtual-machine"></a>Eliminación de máquinas virtuales

Cuando ya no se necesiten, puede usar el comando [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

[Tutorial de creación de máquinas virtuales de alta disponibilidad](create-cli-complete.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

[Ejemplos de PowerShell de implementación de máquinas virtuales](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

