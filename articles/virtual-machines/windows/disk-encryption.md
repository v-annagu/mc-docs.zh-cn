---
title: Azure 托管磁盘的服务器端加密 - PowerShell
description: Azure 存储在将数据保存到存储群集之前会对其进行静态加密，以此保护数据。 可以依赖于 Azure 托管的密钥来加密托管磁盘，也可以使用客户管理的密钥通过自己的密钥来管理加密。
author: rockboyfor
origin.date: 04/21/2020
ms.date: 07/13/2020
ms.testscope: no
ms.testdate: 07/13/2020
ms.topic: conceptual
ms.author: v-yeche
ms.service: virtual-machines-windows
ms.subservice: disks
ms.openlocfilehash: c08b0fde65cc7432f29cc35425c6b4d7be55f449
ms.sourcegitcommit: 2bd0be625b21c1422c65f20658fe9f9277f4fd7c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2020
ms.locfileid: "86440980"
---
<!--Verified successfully-->
<!--PG notified the Customer-managed keys GA on global-->
# <a name="server-side-encryption-of-azure-managed-disks"></a>Azure 托管磁盘的服务器端加密

默认情况下，在将数据保存到云时，Azure 托管磁盘会自动加密数据。 服务器端加密 (SSE) 可保护数据，并帮助实现组织安全性和符合性承诺。

Azure 托管磁盘中的数据使用 256 位 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)（可用的最强大分组加密之一）以透明方式加密，且符合 FIPS 140-2 规范。 有关加密模块基础 Azure 托管磁盘的详细信息，请参阅[加密 API：下一代](https://docs.microsoft.com/windows/desktop/seccng/cng-portal)

加密不会影响托管磁盘的性能，并且不会产生额外的费用。 

> [!NOTE]
> 临时磁盘不是托管磁盘，未由 SSE 加密；有关临时磁盘的详细信息，请参阅[托管磁盘概述：磁盘角色](managed-disks-overview.md#disk-roles)。

## <a name="about-encryption-key-management"></a>关于加密密钥管理

可以依赖于平台托管的密钥来加密托管磁盘，也可以使用自己的密钥来管理加密。 如果选择使用自己的密钥管理加密，可以指定一个客户托管密钥，用于加密和解密托管磁盘中的所有数据。 

以下部分更详细地介绍了密钥管理的每个选项。

## <a name="platform-managed-keys"></a>平台托管的密钥

默认情况下，托管磁盘使用平台托管的加密密钥。 自 2017 年 6 月 10 日起，所有新托管磁盘、快照、图像和写入现有托管磁盘中的新数据都会使用平台托管密钥自动进行静态加密。

## <a name="customer-managed-keys"></a>客户管理的密钥

可以选择使用自己的密钥在每个托管磁盘的级别管理加密。 使用客户托管密钥对托管磁盘进行服务器端加密提供了与 Azure Key Vault 的集成体验。

<!--Not Available on  You can either import your RSA keys to your Key Vault or generate new RSA keys in Azure Key Vault.-->
<!--Not Available on  [your RSA keys](../../key-vault/keys/hsm-protected-keys.md)-->

Azure 托管磁盘使用[信封加密](../../storage/common/storage-client-side-encryption.md#encryption-and-decryption-via-the-envelope-technique)以完全透明的方式处理加密和解密。 它使用基于 [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256 的数据加密密钥 (DEK) 对数据进行加密，DEK 反过来使用你的密钥进行保护。 存储服务生成数据加密密钥，并使用 RSA 加密通过客户托管密钥对其进行加密。 通过信封加密，可以根据合规性策略定期轮替（更改）密钥，而不会影响 VM。 轮替密钥时，存储服务会使用新的客户托管密钥对数据加密密钥进行重新加密。 

必须授予对 Key Vault 中的托管磁盘的访问权限，才能使用你的密钥来加密和解密 DEK。 这允许你完全控制数据和密钥。 可以随时禁用密钥或撤销对托管磁盘的访问权限。 还可以通过 Azure Key Vault 监视来审核加密密钥用法，以确保仅托管磁盘或其他受信任的 Azure 服务访问你的密钥。

对于高级 SSD、标准 SSD 和标准 HDD：禁用或删除密钥后，包含使用该密钥的磁盘的任何 VM 都会自动关闭。 之后，VM 将无法使用，除非再次启用密钥或分配新密钥。

<!--Not Available on For ultra disks, when you disable or delete a key, any VMs with ultra disks using the key won't automatically shut down. Once you deallocate and restart the VMs then the disks will stop using the key and then VMs won't come back online. To bring the VMs back online, you must assign a new key or enable the existing key.-->

下图显示了托管磁盘如何借助 Azure Active Directory 和 Azure Key Vault 使用客户托管密钥发出请求：

![托管磁盘和客户托管密钥工作流。 管理员创建 Azure Key Vault，然后创建并设置磁盘加密集。 该集与 VM 关联，这允许磁盘使用 Azure AD 进行身份验证](media/disk-storage-encryption/customer-managed-keys-sse-managed-disks-workflow.png)

下表更详细地介绍了该图：

1. Azure Key Vault 管理员创建密钥保管库资源。
1. 密钥保管库管理员可以将 RSA 密钥导入 Key Vault，也可以在 Key Vault 中生成新的 RSA 密钥。
1. 该管理员创建磁盘加密集资源的实例，指定 Azure Key Vault ID 和密钥 URL。 磁盘加密集是为了简化托管磁盘的密钥管理而引入的新资源。 
1. 创建磁盘加密集时，将在 Azure Active Directory (AD) 中创建[系统分配的托管标识](../../active-directory/managed-identities-azure-resources/overview.md)，并将其与磁盘加密集相关联。 
1. 然后，Azure Key Vault 管理员授予托管标识权限，以在密钥保管库中执行操作。
1. VM 用户可以通过将磁盘与磁盘加密集相关联来创建磁盘。 VM 用户还可以通过将现有资源的客户托管密钥与磁盘加密集相关联来启用客户托管密钥的服务器端加密。 
1. 托管磁盘使用托管标识将请求发送到 Azure Key Vault。
1. 若要读取或写入数据，托管磁盘会将请求发送到 Azure Key Vault 以加密（包装）和解密（解包）数据加密密钥，以便执行数据的加密和解密。 

若要撤销对客户托管密钥的访问权限，请参阅 [Azure Key Vault PowerShell](https://docs.microsoft.com/powershell/module/azurerm.keyvault/) 和 [Azure Key Vault CLI](https://docs.azure.cn/cli/keyvault?view=azure-cli-latest)。 撤销访问权限会实际阻止对存储帐户中所有数据的访问权限，因为 Azure 存储无法访问加密密钥。

### <a name="supported-regions"></a>支持的区域

[!INCLUDE [virtual-machines-disks-encryption-regions](../../../includes/virtual-machines-disks-encryption-regions.md)]

### <a name="restrictions"></a>限制

目前，客户托管密钥具有以下限制：

- 如果为磁盘启用了此功能，则无法禁用它。
    如果需要解决此问题，则必须[复制所有数据](disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk)到完全不同的托管磁盘（未使用客户托管密钥）。
- 仅支持大小为 2080 的[软件密钥](../../key-vault/keys/about-keys.md)，不支持其他密钥或其他大小。

    <!--Not Available on and HSM RSA keys-->

- 从使用服务器端加密和客户托管密钥加密的自定义映像创建的磁盘必须使用相同的客户托管密钥进行加密，且必须位于同一订阅中。
- 从使用服务器端加密和客户托管密钥加密的磁盘创建的快照必须使用相同的客户托管密钥进行加密。
- 与客户托管密钥相关的所有资源（Azure Key Vault、磁盘加密集、VM、磁盘和快照）都必须位于同一订阅和区域中。
- 使用客户托管密钥加密的磁盘、快照和映像不能移至另一个订阅。
- 使用服务器端加密和客户托管密钥加密的托管磁盘也不能使用 Azure 磁盘加密进行加密，反之亦然

<!--Not Available on [Preview: Use customer-managed keys for encrypting images](../image-version-encryption.md)-->

### <a name="powershell"></a>PowerShell

#### <a name="setting-up-your-azure-key-vault-and-diskencryptionset"></a>设置 Azure Key Vault 和 DiskEncryptionSet

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-powershell](../../../includes/virtual-machines-disks-encryption-create-key-vault-powershell.md)]

#### <a name="create-a-vm-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>使用市场映像创建 VM，并使用客户托管密钥加密 OS 和数据磁盘

```powershell
$VMLocalAdminUser = "yourVMLocalAdminUserName"
$VMLocalAdminSecurePassword = ConvertTo-SecureString <password> -AsPlainText -Force
$LocationName = "chinaeast"
$ResourceGroupName = "yourResourceGroupName"
$ComputerName = "yourComputerName"
$VMName = "yourVMName"
$VMSize = "Standard_DS3_v2"
$diskEncryptionSetName="yourdiskEncryptionSetName"

$NetworkName = "yourNetworkName"
$NICName = "yourNICName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"

$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
$NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName -Location $LocationName -SubnetId $Vnet.Subnets[0].Id

$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $($VMName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage

$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name $($VMName +"DataDisk1") -DiskSizeInGB 128 -StorageAccountType Premium_LRS -CreateOption Empty -Lun 0 -DiskEncryptionSetId $diskEncryptionSet.Id 

New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine -Verbose
```

#### <a name="create-an-empty-disk-encrypted-using-server-side-encryption-with-customer-managed-keys-and-attach-it-to-a-vm"></a>创建一个使用客户托管密钥的服务器端加密进行了加密的空磁盘，并将其附加到 VM

```PowerShell
$vmName = "yourVMName"
$LocationName = "chinaeast"
$ResourceGroupName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskSKU = "Premium_LRS"
$diskSizeinGiB = 30
$diskLUN = 1
$diskEncryptionSetName="yourDiskEncryptionSetName"

$vm = Get-AzVM -Name $vmName -ResourceGroupName $ResourceGroupName 

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Empty -DiskSizeInGB $diskSizeinGiB -StorageAccountType $diskSKU -Lun $diskLUN -DiskEncryptionSetId $diskEncryptionSet.Id 

Update-AzVM -ResourceGroupName $ResourceGroupName -VM $vm

```

#### <a name="encrypt-existing-managed-disks"></a>加密现有托管磁盘 

不得将现有磁盘附加到正在运行的 VM，以便可以使用以下脚本加密这些磁盘：

```PowerShell
$rgName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskEncryptionSetName = "yourDiskEncryptionSetName"

$diskEncryptionSet = Get-AzDiskEncryptionSet -ResourceGroupName $rgName -Name $diskEncryptionSetName

New-AzDiskUpdateConfig -EncryptionType "EncryptionAtRestWithCustomerKey" -DiskEncryptionSetId $diskEncryptionSet.Id | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
```

#### <a name="create-a-virtual-machine-scale-set-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>使用市场映像创建虚拟机规模集，并使用客户托管密钥加密 OS 和数据磁盘

```PowerShell
$VMLocalAdminUser = "yourLocalAdminUser"
$VMLocalAdminSecurePassword = ConvertTo-SecureString Password@123 -AsPlainText -Force
$LocationName = "chinaeast"
$ResourceGroupName = "yourResourceGroupName"
$ComputerNamePrefix = "yourComputerNamePrefix"
$VMScaleSetName = "yourVMSSName"
$VMSize = "Standard_DS3_v2"
$diskEncryptionSetName="yourDiskEncryptionSetName"

$NetworkName = "yourVNETName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"

$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix

$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet

$ipConfig = New-AzVmssIpConfig -Name "myIPConfig" -SubnetId $Vnet.Subnets[0].Id 

$VMSS = New-AzVmssConfig -Location $LocationName -SkuCapacity 2 -SkuName $VMSize -UpgradePolicyMode 'Automatic'

$VMSS = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" -VirtualMachineScaleSet $VMSS -Primary $true -IpConfiguration $ipConfig

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

# Enable encryption at rest with customer managed keys for OS disk by setting DiskEncryptionSetId property 

$VMSS = Set-AzVmssStorageProfile $VMSS -OsDiskCreateOption "FromImage" -DiskEncryptionSetId $diskEncryptionSet.Id -ImageReferenceOffer 'WindowsServer' -ImageReferenceSku '2012-R2-Datacenter' -ImageReferenceVersion latest -ImageReferencePublisher 'MicrosoftWindowsServer'

$VMSS = Set-AzVmssOsProfile $VMSS -ComputerNamePrefix $ComputerNamePrefix -AdminUsername $VMLocalAdminUser -AdminPassword $VMLocalAdminSecurePassword

# Add a data disk encrypted at rest with customer managed keys by setting DiskEncryptionSetId property 

$VMSS = Add-AzVmssDataDisk -VirtualMachineScaleSet $VMSS -CreateOption Empty -Lun 1 -DiskSizeGB 128 -StorageAccountType Premium_LRS -DiskEncryptionSetId $diskEncryptionSet.Id

$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

New-AzVmss -VirtualMachineScaleSet $VMSS -ResourceGroupName $ResourceGroupName -VMScaleSetName $VMScaleSetName
```

#### <a name="change-the-key-of-a-diskencryptionset-to-rotate-the-key-for-all-the-resources-referencing-the-diskencryptionset"></a>更改 DiskEncryptionSet 的密钥，以轮替所有引用 DiskEncryptionSet 的资源的密钥

```PowerShell
$ResourceGroupName="yourResourceGroupName"
$keyVaultName="yourKeyVaultName"
$keyName="yourKeyName"
$diskEncryptionSetName="yourDiskEncryptionSetName"

$keyVault = Get-AzKeyVault -VaultName $keyVaultName -ResourceGroupName $ResourceGroupName

$keyVaultKey = Get-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName

Update-AzDiskEncryptionSet -Name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName -SourceVaultId $keyVault.ResourceId -KeyUrl $keyVaultKey.Id
```

#### <a name="find-the-status-of-server-side-encryption-of-a-disk"></a>查找磁盘的服务器端加密状态

[!INCLUDE [virtual-machines-disks-encryption-status-powershell](../../../includes/virtual-machines-disks-encryption-status-powershell.md)]

> [!IMPORTANT]
> 客户托管密钥依赖于 Azure 资源的托管标识（Azure Active Directory (Azure AD) 的一项功能）。 配置客户托管密钥时，实际上会自动将托管标识分配给你的资源。 如果随后将订阅、资源组或托管磁盘从一个 Azure AD 目录移动到另一个目录，则与托管磁盘关联的托管标识不会转移到新租户，因此，客户托管密钥可能不再有效。 有关详细信息，请参阅[在 Azure AD 目录之间转移订阅](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories)。

<!--MOONCAKE: *Disk Encryption Sets* FEATURE IS INVALID ON AZURE CHINA PORTAL-->
<!--Not Avaiable on [!INCLUDE [virtual-machines-disks-encryption-portal](../../../includes/virtual-machines-disks-encryption-portal.md)]-->
<!--Not Avaiable on  Customer-managed keys rely on managed identities for Azure resources-->

## <a name="server-side-encryption-versus-azure-disk-encryption"></a>服务器端加密与 Azure 磁盘加密

[Azure 磁盘加密](../../security/fundamentals/azure-disk-encryption-vms-vmss.md)利用 Windows 的 [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview) 功能，通过来宾 VM 中的客户托管密钥来加密托管磁盘。  使用客户托管密钥的服务器端加密改进了 ADE，它通过加密存储服务中的数据使你可以为 VM 使用任何 OS 类型和映像。

## <a name="next-steps"></a>后续步骤

- [探索 Azure 资源管理器模板以使用客户管理密钥创建加密磁盘](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [什么是 Azure 密钥保管库？](../../key-vault/general/overview.md)
    
    <!--Not Available on - [Replicate machines with customer-managed keys enabled disks](../../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)-->
    
- [使用 PowerShell 设置 VMware VM 到 Azure 的灾难恢复](../../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [使用 PowerShell 和 Azure 资源管理器为 Hyper-V VM 设置到 Azure 的灾难恢复](../../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)

<!-- Update_Description: update meta properties, wording update, update link -->