---
title: "为 SQL AlwaysOn 配置负载均衡器 | Microsoft Docs"
description: "介绍如何将负载均衡器配置为与 SQL AlwaysOn 配合工作，以及如何利用 powershell 为 SQL 实现创建负载均衡器"
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
ms.assetid: d7bc3790-47d3-4e95-887c-c533011e4afd
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: 3ebbf1c4009d89b1f18b2ff8ff5dd243c456dff8
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="configure-load-balancer-for-sql-always-on"></a>为 SQL AlwaysOn 配置负载均衡器

[!INCLUDE [load-balancer-basic-sku-include.md](../../includes/load-balancer-basic-sku-include.md)]

SQL Server AlwaysOn 可用性组现在可与 ILB 配合运行。 可用性组是 SQL Server 用于实现高可用性和灾难恢复的旗舰解决方案。 无论配置中的副本数目是多少，可用性组侦听器都可让客户端应用程序无缝连接到主副本。

侦听器 (DNS) 名称会映射到经过负载均衡的 IP 地址，Azure 的负载均衡器只会将传入流量定向到副本集中的主服务器。

可以使用 SQL Server AlwaysOn（侦听器）终结点的 ILB 支持。 现在，可以控制侦听器的可访问性，并可以从虚拟网络 (VNet) 中的特定子网内选择负载均衡的 IP 地址。

如果在侦听器上使用 ILB，只有以下项可以访问 SQL Server 终结点（例如 Server=tcp:ListenerName,1433;Database=DatabaseName）：

* 同一虚拟网络中的服务和 VM
* 来自连接的本地网络的服务和 VM
* 来自互连的 VNet 的服务和 VM

![ILB_SQLAO_NewPic](./media/load-balancer-configure-sqlao/sqlao1.png)

图 1 - 使用面向 Internet 的负载均衡器配置的 SQL AlwaysOn

## <a name="add-internal-load-balancer-to-the-service"></a>将内部负载均衡器添加到服务

1. 在以下示例中，我们会配置包含名为“Subnet-1”的子网的虚拟网络：

    ```powershell
    Add-AzureInternalLoadBalancer -InternalLoadBalancerName ILB_SQL_AO -SubnetName Subnet-1 -ServiceName SqlSvc
    ```
2. 为每个 VM 上的 ILB 添加负载均衡终结点

    ```powershell
    Get-AzureVM -ServiceName SqlSvc -Name sqlsvc1 | Add-AzureEndpoint -Name "LisEUep" -LBSetName "ILBSet1" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -
    DirectServerReturn $true -InternalLoadBalancerName ILB_SQL_AO | Update-AzureVM

    Get-AzureVM -ServiceName SqlSvc -Name sqlsvc2 | Add-AzureEndpoint -Name "LisEUep" -LBSetName "ILBSet1" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -DirectServerReturn $true -InternalLoadBalancerName ILB_SQL_AO | Update-AzureVM
    ```

    在上述示例中，有 2 个分别名为“sqlsvc1”和“sqlsvc2”的 VM 在云服务“Sqlsvc”中运行。 使用 `DirectServerReturn` 开关创建 ILB 后，可以将经过负载均衡的终结点添加到 ILB，以便 SQL 为可用性组配置侦听器。

有关 SQL AlwaysOn 的详细信息，请参阅[在 Azure 中为 AlwaysOn 可用性组配置内部负载均衡器](../virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener.md)。

## <a name="see-also"></a>另请参阅
[开始配置面向 Internet 的负载均衡器](load-balancer-get-started-internet-arm-ps.md)

[开始配置内部负载均衡器](load-balancer-get-started-ilb-arm-ps.md)

[配置负载均衡器分发模式](load-balancer-distribution-mode.md)

[配置负载均衡器的空闲 TCP 超时设置](load-balancer-tcp-idle-timeout.md)
