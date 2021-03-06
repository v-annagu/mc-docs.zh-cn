---
title: 排查 Azure Automation State Configuration 问题
description: 本文介绍如何排查 Azure Automation State Configuration 问题。
services: automation
ms.service: automation
ms.subservice: ''
author: WenJason
ms.author: v-jay
origin.date: 04/16/2019
ms.date: 05/11/2020
ms.topic: conceptual
manager: digimobile
ms.openlocfilehash: e3b19350938e54103cba466562d0d8a5a0357fd7
ms.sourcegitcommit: 981a75a78f8cf74ab5a76f9e6b0dc5978387be4b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/22/2020
ms.locfileid: "83801153"
---
# <a name="troubleshoot-azure-automation-state-configuration-issues"></a>排查 Azure Automation State Configuration 问题

本文介绍如何排查在 Azure Automation State Configuration 中编译或部署配置时出现的问题。

## <a name="diagnose-an-issue"></a>诊断问题

收到配置编译或部署错误时，可以借助下面这些步骤来诊断问题。

### <a name="1-ensure-that-your-configuration-compiles-successfully-on-the-local-machine"></a>1.确保配置在本地计算机上编译成功

Azure Automation State Configuration 构建在 PowerShell Desired State Configuration (DSC) 基础之上。 可以在 [PowerShell DSC 文档](https://docs.microsoft.com/powershell/scripting/overview)中找到 DSC 语言和语法的文档。

在本地计算机上编译 DSC 配置即可发现并解决常见错误，例如：

   - 缺少模块。
   - 语法错误。
   - 逻辑错误。

### <a name="2-view-dsc-logs-on-your-node"></a>2.在节点上查看 DSC 日志

如果配置编译成功，但在应用到节点时失败，则可在 DSC 日志中查找详细信息。 若要了解在何处查找这些日志，请参阅 [DSC 事件日志在哪里](https://docs.microsoft.com/powershell/scripting/dsc/troubleshooting/troubleshooting#where-are-dsc-event-logs)。

[xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics) 模块可以帮助你分析 DSC 日志中的详细信息。 如果你联系支持部门，他们会要求你提供这些日志，以便对你的问题进行诊断。

可以根据[安装稳定版本的模块](https://github.com/PowerShell/xDscDiagnostics#install-the-stable-version-module)中的说明在本地计算机上安装 `xDscDiagnostics` 模块。

若要在 Azure 计算机上安装 `xDscDiagnostics` 模块，请使用 [Invoke-AzVMRunCommand](https://docs.microsoft.com/powershell/module/az.compute/invoke-azvmruncommand?view=azps-3.7.0)。 也可以按照[使用“运行命令”在 Windows VM 中运行 PowerShell 脚本](../../virtual-machines/windows/run-command.md)中的步骤，在 Azure 门户中使用“运行命令”选项。

若要了解如何使用 **xDscDiagnostics**，请参阅[使用 xDscDiagnostics 分析 DSC 日志](https://docs.microsoft.com/powershell/scripting/dsc/troubleshooting/troubleshooting#using-xdscdiagnostics-to-analyze-dsc-logs)。 另请参阅 [xDscDiagnostics Cmdlet](https://github.com/PowerShell/xDscDiagnostics#cmdlets)。

## <a name="scenario-a-configuration-with-special-characters-cant-be-deleted-from-the-portal"></a><a name="unsupported-characters"></a>场景：无法从门户删除带有特殊字符的配置

### <a name="issue"></a>问题

尝试通过门户删除 DSC 配置时看到以下错误：

```error
An error occurred while deleting the DSC configuration '<name>'.  Error-details: The argument configurationName with the value <name> is not valid.  Valid configuration names can contain only letters,  numbers, and underscores.  The name must start with a letter.  The length of the name must be between 1 and 64 characters.
```

### <a name="cause"></a>原因

此错误是计划要解决的临时问题。

### <a name="resolution"></a>解决方法

使用 [Remove-AzAutomationDscConfiguration](https://docs.microsoft.com/powershell/module/Az.Automation/Remove-AzAutomationDscConfiguration?view=azps-3.7.0 cmdlet 来删除配置。

## <a name="scenario-failed-to-register-the-dsc-agent"></a><a name="failed-to-register-agent"></a>场景：无法注册 DSC 代理

### <a name="issue"></a>问题

运行 [Set-DscLocalConfigurationManager](https://docs.microsoft.com/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager?view=powershell-5.1) 或其他 DSC cmdlet 时遇到错误：

```error
Registration of the Dsc Agent with the server
https://<location>-agentservice-prod-1.azure-automation.cn/accounts/00000000-0000-0000-0000-000000000000 failed. The
underlying error is: Failed to register Dsc Agent with AgentId 00000000-0000-0000-0000-000000000000 with the server htt
ps://<location>-agentservice-prod-1.azure-automation.cn/accounts/00000000-0000-0000-0000-000000000000/Nodes(AgentId='00000000-0000-0000-0000-000000000000'). .
    + CategoryInfo          : InvalidResult: (root/Microsoft/...gurationManager:String) [], CimException
    + FullyQualifiedErrorId : RegisterDscAgentCommandFailed,Microsoft.PowerShell.DesiredStateConfiguration.Commands.Re
   gisterDscAgentCommand
    + PSComputerName        : <computerName>
```

### <a name="cause"></a>原因

此错误通常由防火墙、位于代理服务器后面的计算机或其他网络错误导致。

### <a name="resolution"></a>解决方法

验证计算机是否可以访问 DSC 的相应终结点，然后重试。 有关所需端口和地址的列表，请参阅[网络规划](../automation-dsc-overview.md#network-planning)。

## <a name="a-nameunauthorizedscenario-status-reports-return-the-response-code-unauthorized"></a><a name="unauthorized"><a/>场景：状态报表返回响应代码“未授权”

### <a name="issue"></a>问题

将某个节点注册到 Azure Automation State Configuration 时收到以下错误消息之一：

```error
The attempt to send status report to the server https://{your Automation account URL}/accounts/xxxxxxxxxxxxxxxxxxxxxx/Nodes(AgentId='xxxxxxxxxxxxxxxxxxxxxxxxx')/SendReport returned unexpected response code Unauthorized.
```

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC / Registration of the Dsc Agent with the server failed.
```

### <a name="cause"></a>原因

此问题由错误或过期的证书引起。 请参阅[重新注册节点](../automation-dsc-onboarding.md#re-register-a-node)。

此问题也可能是由于代理配置不允许访问 * **.azure-automation.cn** 而导致的。 有关详细信息，请参阅[配置专用网络](../automation-dsc-overview.md#network-planning)。 

### <a name="resolution"></a>解决方法

使用以下步骤重新注册失败的 DSC 节点。

#### <a name="step-1-unregister-the-node"></a>步骤 1：取消注册节点

1. 在 Azure 门户中，转到“主页” > “自动化帐户”-> (你的自动化帐户) >“State Configuration (DSC)”。  
1. 选择“节点”，然后选择有问题的节点。
1. 选择“取消注册”以取消注册该节点。

#### <a name="step-2-uninstall-the-dsc-extension-from-the-node"></a>步骤 2：从节点中卸载 DSC 扩展

1. 在 Azure 门户中，转到“主页” > “虚拟机”> (失败的节点) >“扩展”。  
1. 选择“Microsoft.Powershell.DSC”，即 PowerShell DSC 扩展。
1. 选择“卸载”以卸载该扩展。

#### <a name="step-3-remove-all-bad-or-expired-certificates-from-the-node"></a>步骤 3：从节点中删除所有错误的或过期的证书

在失败的节点上，从权限提升的 PowerShell 提示符处运行以下命令：

```powershell
$certs = @()
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC"}
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC-OaaS Client Authentication"}
$certs += dir cert:\localmachine\CA | ?{$_.subject -like "CN=AzureDSCExtension*"}
"";"== DSC Certificates found: " + $certs.Count
$certs | FL ThumbPrint,FriendlyName,Subject
If (($certs.Count) -gt 0)
{ 
    ForEach ($Cert in $certs) 
    {
        RD -LiteralPath ($Cert.Pspath) 
    }
}
```

#### <a name="step-4-reregister-the-failing-node"></a>步骤 4：重新注册失败的节点

1. 在 Azure 门户中，转到“主页” > “自动化帐户”-> (你的自动化帐户) >“State Configuration (DSC)”。  
1. 选择“节点”。
1. 选择“添加”  。
1. 选择故障节点。
1. 选择“连接”，然后选择所需的选项。

## <a name="scenario-node-is-in-failed-status-with-a-not-found-error"></a><a name="failed-not-found"></a>场景：节点处于失败状态，出现“未找到”错误

### <a name="issue"></a>问题

该节点有一个状态为“失败”的报表，其中包含错误：

```error
The attempt to get the action from server https://<url>//accounts/<account-id>/Nodes(AgentId=<agent-id>)/GetDscAction failed because a valid configuration <guid> cannot be found.
```

### <a name="cause"></a>原因

将节点分配到配置名称（例如 **ABC**）而不是节点配置（MOF 文件）名称（例如 **ABC.WebServer**）时，通常会发生此错误。

### <a name="resolution"></a>解决方法

* 确保为节点分配节点配置名称而不是配置名称。
* 可以使用 Azure 门户或 PowerShell cmdlet 将节点配置分配给节点。

  * 在 Azure 门户中，转到“主页” > “自动化帐户”-> (你的自动化帐户) >“State Configuration (DSC)”。   选择一个节点，然后选择“分配节点配置”。
  * 使用 [Set-AzAutomationDscNode](https://docs.microsoft.com/powershell/module/Az.Automation/Set-AzAutomationDscNode?view=azps-3.7.0) cmdlet。

## <a name="scenario-no-node-configurations-mof-files-were-produced-when-a-configuration-was-compiled"></a><a name="no-mof-files"></a>场景：编译配置时未生成节点配置（MOF 文件）

### <a name="issue"></a>问题

DSC 编译作业暂停，且出现错误：

```error
Compilation completed successfully, but no node configuration **.mof** files were generated.
```

### <a name="cause"></a>原因

如果 DSC 配置中 `Node` 关键字后面的表达式的计算结果为 `$null`，则不会生成节点配置。

### <a name="resolution"></a>解决方法

使用以下解决方法之一来解决问题：

* 确保配置定义中 `Node` 关键字旁边的表达式的计算结果不为 Null。
* 如果要在编译配置时传递 [ConfigurationData](../automation-dsc-compile.md)，请确保从配置数据传递配置所预期的值。

## <a name="scenario-the-dsc-node-report-becomes-stuck-in-the-in-progress-state"></a><a name="dsc-in-progress"></a>场景：DSC 节点报告停滞在“正在进行”状态

### <a name="issue"></a>问题

DSC 代理输出：

```error
No instance found with given property values
```

### <a name="cause"></a>原因

已升级 Windows Management Framework (WMF) 版本，但已损坏 Windows Management Instrumentation (WMI)。

### <a name="resolution"></a>解决方法

按照 [DSC 已知问题和限制](https://docs.microsoft.com/powershell/scripting/wmf/known-issues/known-issues-dsc)中的说明进行操作。

## <a name="scenario-unable-to-use-a-credential-in-a-dsc-configuration"></a><a name="issue-using-credential"></a>场景：无法在 DSC 配置中使用凭据

### <a name="issue"></a>问题

DSC 编译作业已暂停并出现错误：

```error
System.InvalidOperationException error processing property 'Credential' of type <some resource name>: Converting and storing an encrypted password as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
```

### <a name="cause"></a>原因

在配置中使用了凭据，但未提供正确的 `ConfigurationData`，从而无法将每个节点配置的 `PSDscAllowPlainTextPassword` 设置为 true。

### <a name="resolution"></a>解决方法

确保传入正确的 `ConfigurationData`，以便将配置中提到的每个节点配置的 `PSDscAllowPlainTextPassword` 设置为 true。 请参阅[在 Azure Automation State Configuration 中编译 DSC 配置](../automation-dsc-compile.md)。

## <a name="scenario-failure-processing-extension-error-when-enabling-a-machine-from-a-dsc-extension"></a><a name="failure-processing-extension"></a>场景：从 DSC 扩展启用计算机时出现“处理扩展失败”错误

### <a name="issue"></a>问题

使用 DSC 扩展启用计算机时失败并出现以下错误：

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'. Error message: \"DSC COnfiguration 'RegistrationMetaConfigV2' completed with error(s). Following are the first few: Registration of the Dsc Agent with the server <url> failed. The underlying error is: The attempt to register Dsc Agent with Agent Id <ID> with the server <url> return unexpected response code BadRequest. .\".
```

### <a name="cause"></a>原因

为节点分配了服务中不存在的节点配置名称时，通常会出现此错误。

### <a name="resolution"></a>解决方法

* 确保为节点分配的名称与服务中的名称完全匹配。
* 可以选择不包含节点配置名称，这会导致系统启用节点而不分配节点配置。

## <a name="scenario-failure-with-a-general-error-when-applying-a-configuration-in-linux"></a><a name="failure-linux-temp-noexec"></a>场景：在 Linux 中应用配置时失败并出现常规错误

### <a name="issue"></a>问题

在 Linux 中应用配置时失败并出现以下错误：

```error
This event indicates that failure happens when LCM is processing the configuration. ErrorId is 1. ErrorDetail is The SendConfigurationApply function did not succeed.. ResourceId is [resource]name and SourceInfo is ::nnn::n::resource. ErrorMessage is A general error occurred, not covered by a more specific error code..
```

### <a name="cause"></a>原因

如果 **/tmp** 位置设置为 `noexec`，则当前版本的 DSC 将无法应用配置。

### <a name="resolution"></a>解决方法

从 **/tmp** 位置中删除 `noexec` 选项。

## <a name="scenario-node-configuration-names-that-overlap-can-result-in-a-bad-release"></a><a name="compilation-node-name-overlap"></a>场景：节点配置名称重叠可能会生成错误的版本

### <a name="issue"></a>问题

如果使用单个配置脚本来生成多个节点配置，而某些节点配置名称是其他名称的子集，则编译服务最终可能分配错误的配置。 仅当使用单个脚本来生成配置且每个节点都有配置数据，以及仅当字符串开头出现名称重叠时，才会出现此问题。 例如，如果在使用单个配置脚本生成配置时，根据的是使用 cmdlet 作为哈希表传递的节点数据，且节点数据包含名为 **server** 和 **1server** 的服务器，则可能会出现上述问题。

### <a name="cause"></a>原因

这是编译服务的一个已知问题。

### <a name="resolution"></a>解决方法

最佳解决方法是在本地或 CI/CD 管道中进行编译，然后将节点配置 MOF 文件直接上传到服务。 如果必须在服务中进行编译，则次佳解决方法是拆分编译作业，这样就不会发生名称重叠现象。

## <a name="scenario-gateway-timeout-error-on-dsc-configuration-upload"></a><a name="gateway-timeout"></a>场景：上传 DSC 配置时出现网关超时错误

#### <a name="issue"></a>问题

上传 DSC 配置时遇到 `GatewayTimeout` 错误。 

### <a name="cause"></a>原因

需要很长时间编译的 DSC 配置可能会导致此错误。

### <a name="resolution"></a>解决方法

通过在所有 [Import-DSCResource](https://docs.microsoft.com/powershell/scripting/dsc/configurations/import-dscresource?view=powershell-5.1) 调用中显式包含 `ModuleName` 参数，可以更快地分析 DSC 配置。

## <a name="next-steps"></a>后续步骤

如果你的问题未在本文中列出，或者你无法解决自己的问题，请尝试通过以下渠道之一获取更多支持：

* 通过 [Azure 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home)获取 Azure 专家的解答
* 提出 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/zh-cn/support/contact/)以获取支持。
