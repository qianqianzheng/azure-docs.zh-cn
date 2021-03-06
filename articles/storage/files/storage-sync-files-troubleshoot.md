---
title: "对 Azure 文件同步（预览版）进行故障排除 | Microsoft Docs"
description: "对 Azure 文件同步的常见问题进行故障排除"
services: storage
documentationcenter: 
author: wmgries
manager: klaasl
editor: jgerend
ms.assetid: 297f3a14-6b3a-48b0-9da4-db5907827fb5
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/08/2017
ms.author: wgries
ms.openlocfilehash: 265c5f660c4bee53a2faf4a073384587eb3f65fc
ms.sourcegitcommit: e38120a5575ed35ebe7dccd4daf8d5673534626c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/13/2017
---
# <a name="troubleshoot-azure-file-sync-preview"></a>对 Azure 文件同步（预览版）进行故障排除
使用 Azure 文件同步（预览版），既可将组织的文件共享集中在 Azure 文件中，又不失本地文件服务器的灵活性、性能和兼容性。 Azure 文件同步可将 Windows Server 转换为 Azure 文件共享的快速缓存。 可以使用 Windows Server 上可用的任意协议本地访问数据，包括 SMB、NFS 和 FTPS。 并且可以根据需要在世界各地具有多个缓存。

本文旨在帮助排查和解决在 Azure 文件同步部署中可能遇到的问题。 此外，还介绍了在需要对问题进行更深入调查的情况下，如何从系统收集重要日志。 如果本文未能涵盖你的问题，欢迎通过以下渠道联系我们（以升序排列）：

1. 本文评论部分。
2. [Azure 存储论坛](https://social.msdn.microsoft.com/Forums/home?forum=windowsazuredata)。
3. [Azure 文件 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files)。 
4. Microsoft 支持部门。 若要创建新的支持请求，请在 Azure 门户中的“帮助”选项卡上，选择“帮助和支持”按钮，然后选择“新建支持请求”。

## <a name="agent-installation-and-server-registration"></a>代理安装和服务器注册
<a id="agent-installation-failures"></a>**排查代理安装失败问题**  
如果 Azure 文件同步代理安装失败，请在安装代理的过程中，在权限提升的命令提示符下运行以下命令，以启用日志记录：

```
StorageSyncAgent.msi /l*v Installer.log
```

查看 installer.log 确定安装失败的原因。 

> [!Note]  
> 如果将计算机设置为使用 Microsoft 更新，代理安装将会失败，并且 Windows 更新服务不会运行。

<a id="server-registration-missing"></a>服务器未在 Azure 门户中的“已注册的服务器”下列出  
如果对于存储同步服务，服务器未在“已注册的服务器”下列出：
1. 登录到要注册的服务器。
2. 打开文件资源管理器，然后转到存储同步代理安装目录（默认位置为 C:\Program Files\Azure\StorageSyncAgent）。 
3. 运行 ServerRegistration.exe 并完成向导中的操作，将服务器注册到存储同步服务。

<a id="server-already-registered"></a>**服务器注册在安装 Azure 文件同步代理期间显示以下消息：“此服务器已注册”** 

![包含“服务器已注册”错误消息的服务器注册对话框屏幕截图](media/storage-sync-files-troubleshoot/server-registration-1.png)

如果以前向存储同步服务注册了服务器，则会显示此消息。 若要从当前存储同步服务中注销服务器并向新存储同步服务进行注册，请完成[从 Azure 文件同步中注销服务器](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service)中所述的步骤。

如果存储同步服务中的“已注册服务器”下未列出该服务器，请在想要注销的服务器上运行以下 PowerShell 命令：

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Reset-StorageSyncServer
```

> [!Note]  
> 如果该服务器是群集的一部分，可以使用可选的 *Reset-StorageSyncServer -CleanClusterRegistration* 参数来同时删除群集注册。

<a id="web-site-not-trusted"></a>**注册服务器时，出现许多“网站不受信任”的响应。为什么？**  
如果在服务器注册过程中启用了“Internet Explorer 增强的安全性”策略，则会出现此问题。 有关如何以正确的方式禁用“Internet Explorer 增强的安全性”策略的详细信息，请参阅[准备 Windows Server 以使用 Azure 文件同步](storage-sync-files-deployment-guide.md#prepare-windows-server-to-use-with-azure-file-sync)，以及[如何部署 Azure 文件同步（预览版）](storage-sync-files-deployment-guide.md)。

## <a name="sync-group-management"></a>同步组管理
<a id="cloud-endpoint-using-share"></a>**云终结点创建失败，并出现以下错误：“指定的 Azure 文件共享已被其他 CloudEndpoint 使用”**  
如果 Azure 文件共享已被其他云终结点使用，则会出现此问题。 

如果看到此消息，并且 Azure 文件共享当前未被云终结点使用，请完成以下步骤清除 Azure 文件共享上的 Azure 文件同步元数据：

> [!Warning]  
> 删除云终结点当前正在使用的 Azure 文件共享上的元数据会导致 Azure 文件同步操作失败。 

1. 在 Azure 门户中，转到 Azure 文件共享。  
2. 右键单击该 Azure 文件共享，并选择“编辑元数据”。
3. 右键单击“SyncService”，并选择“删除”。

<a id="cloud-endpoint-authfailed"></a>**云终结点创建失败，并出现以下错误：“AuthorizationFailed”**  
当用户帐户缺少足够权限来创建云终结点时，将出现此问题。 

要创建云终结点，用户帐户必须具有下列 Microsoft 授权权限：  
* 读取：获取角色定义
* 写入：创建或更新自定义角色定义
* 读取：获取角色分配
* 写入：创建角色分配

以下内置角色具有所需的 Microsoft 授权权限：  
* 所有者
* 用户访问管理员

若要确定自己的用户帐户角色色否具有适当的权限，请执行以下操作：  
1. 在 Azure 门户中，选择“资源组”。
2. 选择存储帐户所在的资源组，再选择“访问控制(IAM)”。
3. 选择用户帐户的**角色**（如“所有者”或“参与者”）。
4. 在“资源提供程序”列表中，选择“Microsoft 授权”。 
    * “角色分配”应具有“读取”和“写入”权限。
    * “角色定义”应具有“读取”和“写入”权限。

<a id="cloud-endpoint-deleteinternalerror"></a>**云终结点删除失败，并出现以下错误：“MgmtInternalError”**  
如果在删除云终结点之前删除了 Azure 文件共享或存储帐户，则可能会出现此问题。 将来的更新中会解决此问题。 到时，便可以在删除 Azure 文件共享或存储帐户之后删除云终结点。

同时，为了防止此问题发生，请先删除云终结点，再删除 Azure 文件共享或存储帐户。

## <a name="sync"></a>同步
<a id="afs-change-detection"></a>**我通过 SMB 或门户在 Azure 文件共享中直接创建了一个文件，该文件同步到同步组中的服务器需要多长时间？**  
[!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

<a id="broken-sync"></a>**在服务器上同步失败**  
如果在服务器上同步失败：
1. 请验证对于要同步到 Azure 文件共享的目录，Azure 门户中是否存在服务器终结点：
    
    ![在 Azure 门户中具有云终结点和服务器终结点的同步组屏幕截图](media/storage-sync-files-troubleshoot/sync-troubleshoot-1.png)

2. 在事件查看器中检查位于 Applications and Services\Microsoft\FileSync\Agent 下的操作日志和诊断事件日志。
    1. 验证服务器是否已建立 Internet 连接。
    2. 验证 Azure 文件同步服务是否正在服务器上运行。 为此，请打开服务 MMC 管理单元，然后验证存储同步代理服务 (FileSyncSvc) 是否正在运行。

<a id="replica-not-ready"></a>**同步失败，并出现以下错误：“0x80c8300f - 副本未准备就绪，无法执行所需的操作”**  
如果创建云终结点并使用包含数据的 Azure 文件共享，则可能会出现此问题。 在 Azure 文件共享上完成更改检测作业后（最长可能需要 24 小时），同步应会开始正常工作。

<a id="broken-sync-files"></a>**排查单个文件无法同步的问题**  
如果单个文件无法同步：
1. 在事件查看器中检查位于 Applications and Services\Microsoft\FileSync\Agent 下的操作日志和诊断事件日志。
2. 验证文件中是否不存在打开的句柄。

    > [!NOTE]
    > Azure 文件同步会定期创建 VSS 快照以同步具有打开的句柄的文件。

## <a name="cloud-tiering"></a>云分层 
<a id="files-fail-tiering"></a>**排查文件无法分层的问题**  
如果文件无法分层到 Azure 文件：

1. 验证 Azure 文件共享中是否存在这些文件。

    > [!NOTE]
    > 文件必须先同步到 Azure 文件共享，然后才能分层。
2. 在事件查看器中检查位于 Applications and Services\Microsoft\FileSync\Agent 下的操作日志和诊断事件日志。
    1. 验证服务器是否已建立 Internet 连接。 
    2. 验证 Azure 文件同步筛选器驱动程序（StorageSync.sys 和 StorageSyncGuard.sys）是否正在运行。
        - 在权限提升的命令提示符下，运行 `fltmc`。 验证 StorageSync.sys 和 StorageSyncGuard.sys 文件系统筛选器驱动程序是否已列出。

<a id="files-fail-recall"></a>排查无法重新调用文件的问题  
如果无法重新调用文件：
1. 在事件查看器中检查位于 Applications and Services\Microsoft\FileSync\Agent 下的操作日志和诊断事件日志。
    1. 验证 Azure 文件共享中是否存在这些文件。
    2. 验证服务器是否已建立 Internet 连接。 
    3. 验证 Azure 文件同步筛选器驱动程序（StorageSync.sys 和 StorageSyncGuard.sys）是否正在运行。
        - 在权限提升的命令提示符下，运行 `fltmc`。 验证 StorageSync.sys 和 StorageSyncGuard.sys 文件系统筛选器驱动程序是否已列出。

<a id="files-unexpectedly-recalled"></a>**排查在服务器上意外重新调用文件的问题**  
读取大量文件的防病毒、备份和其他应用程序会导致意外的重新调用，除非这些应用程序遵循脱机属性并跳过读取这些文件的内容。 跳过支持此选项的产品的脱机文件可帮助避免在执行防病毒软件扫描或备份作业等操作时出现意外的重新调用。

请咨询软件供应商，了解如何配置其解决方案以跳过读取脱机文件。

在其他一些情况下（例如，在文件资源管理器中浏览文件时），也可能出现意外的重新调用。 在服务器上的文件资源管理器中打开包含云分层文件的文件夹可能导致意外的重新调用。 在服务器上启用防病毒解决方案时更是如此。

## <a name="general-troubleshooting"></a>常规故障排除
如果在服务器上遇到 Azure 文件同步问题，请先完成以下步骤：
1. 在事件查看器中检查操作日志和诊断事件日志。
    - 同步、分层和重新调用问题记录在 Applications and Services\Microsoft\FileSync\Agent 下的诊断和操作事件日志中。
    - 与管理服务器相关的（例如配置设置）问题记录在 Applications and Services\Microsoft\FileSync\Management 下的诊断和操作事件日志中。
2. 验证 Azure 文件同步服务是否正在服务器上运行：
    - 打开服务 MMC 管理单元，然后验证存储同步代理服务 (FileSyncSvc) 是否正在运行。
3. 验证 Azure 文件同步筛选器驱动程序（StorageSync.sys 和 StorageSyncGuard.sys）是否正在运行。
    - 在权限提升的命令提示符下，运行 `fltmc`。 验证 StorageSync.sys 和 StorageSyncGuard.sys 文件系统筛选器驱动程序是否已列出。

如果问题未得到解决，请运行 AFSDiag 工具：
1. 创建用于保存 AFSDiag 输出的目录（例如，C:\Output）。
2. 打开权限提升的 PowerShell 窗口并运行以下命令（在每条命令后面按 Enter）：

    ```PowerShell
    cd "c:\Program Files\Azure\StorageSyncAgent"
    Import-Module .\afsdiag.ps1
    Debug-Afs c:\output # Note: Use the path created in step 1.
    ```

3. 对于 Azure 文件同步内核模式跟踪级别，请输入 **1**（除非指定为创建更详细的跟踪）并按 Enter。
4. 对于 Azure 文件同步用户模式跟踪级别，请输入 **1**（除非指定为创建更详细的跟踪）并按 Enter。
5. 再现问题。 完成后，输入 **D**。
6. 随即会将一个包含日志和跟踪文件的 .zip 文件保存到指定的输出目录。

## <a name="see-also"></a>另请参阅
- [Azure 文件常见问题解答](storage-files-faq.md)
- [在 Windows 中排查 Azure 文件问题](storage-troubleshoot-windows-file-connection-problems.md)
- [在 Linux 中排查 Azure 文件问题](storage-troubleshoot-linux-file-connection-problems.md)
