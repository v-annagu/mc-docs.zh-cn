---
title: Azure Stack Hub 的 Close-PrivilegedEndpoint 特权终结点
description: PowerShell Azure Stack 特权终结点 - Close-PrivilegedEndpoint 参考
author: WenJason
ms.topic: reference
origin.date: 04/27/2020
ms.date: 06/22/2020
ms.author: v-jay
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: a84d51f29ec80b4381e7909babdb3484f69ea14f
ms.sourcegitcommit: d86e169edf5affd28a1c1a4476d72b01a7fb421d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85096834"
---
# <a name="start-secretrotation"></a>Start-SecretRotation

## <a name="synopsis"></a>摘要
触发戳记上的机密轮换。

## <a name="syntax"></a>语法

```
Start-SecretRotation [-PathAccessCredential <Object>] [-ReRun] [-CertificatePassword <Object>] [-Internal]
 [-PfxFilesPath <Object>] [-AsJob]
```

## <a name="description"></a>说明
调用机密轮换进程。

## <a name="parameters"></a>parameters

### <a name="-internal"></a>-Internal
 

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-pfxfilespath"></a>-PfxFilesPath
共享用于外部证书轮换的新 pfx 文件的路径。
如果需要外部证书轮换，则这是必需的。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-pathaccesscredential"></a>-PathAccessCredential
用于访问 PfxFilesPath 的凭据。
如果需要外部证书轮换，则这是必需的。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-certificatepassword"></a>-CertificatePassword
提供的新 pfx 文件的密码。
如果需要外部证书轮换，则这是必需的。
它可能与初始部署时提供的原始 pfx 密码不同。
我们将使用正确的 CA 密码重新生成 pfx 文件。

用法：
    # Rotates external certificates only
    Start-SecretRotation -PfxFilesPath \<String\> -PathAccessCredential \<PSCredential\> -CertificatePassword \<SecureString\>

    # Rotates internal secrets only
    Start-SecretRotation -Internal  

    # Reruns internal secrets only
    Start-SecretRotation -Internal -ReRun 

    # Reruns external certificates only
    Start-SecretRotation -ReRun

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-rerun"></a>-ReRun
 

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-asjob"></a>-AsJob


```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="next-steps"></a>后续步骤

有关如何访问和使用特权终结点的信息，请参阅[使用 Azure Stack Hub 中的特权终结点](/azure-stack/operator/azure-stack-privileged-endpoint)。