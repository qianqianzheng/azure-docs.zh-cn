---
title: "为 Azure MFA 分配许可证 | Microsoft 文档"
description: "了解如何为 Microsoft Azure 多重身份验证分配许可证。"
services: multi-factor-authentication
documentationcenter: 
author: MicrosoftGuyJFlo
manager: femila
ms.assetid: 514ef423-8ee6-4987-8a4e-80d5dc394cf9
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/13/2017
ms.author: joflore
ms.reviewer: richagi
ms.custom: it-pro
ROBOTS: NOINDEX
ms.openlocfilehash: 48f6ca268630524dab6d239621a7824faaa67849
ms.sourcegitcommit: 9a61faf3463003375a53279e3adce241b5700879
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2017
---
# <a name="assigning-an-azure-mfa-azure-ad-premium-or-enterprise-mobility-license-to-users"></a>向用户分配 Azure MFA、Azure AD Premium 或企业移动性许可证
如果已购买 Azure MFA、Azure AD Premium 或企业移动性套件许可证，则不需要创建 Multi-Factor Auth 提供程序。 将许可证分配给用户后，即可开始为用户启用 MFA。

## <a name="to-assign-a-license"></a>分配许可证
1. 以管理员身份登录到 [Azure 经典门户](https://manage.windowsazure.com)。
2. 在左侧选择“Active Directory”。
3. 在“Active Directory”页上，双击要分配许可证的用户所在的目录。
4. 在“目录”页的顶部，选择“许可证”。
   ![分配许可证](./media/multi-factor-authentication-get-started-assign-licenses/assign1.png)
5. 在“许可证”页上，选择“多重身份验证”、“Active Directory Premium”或“企业移动性套件”。  如果只有一项，则会自动选择该项。
6. 在页面的底部，单击“分配”。
   ![分配许可证](./media/multi-factor-authentication-get-started-assign-licenses/assign3.png)
7. 在出现的框中，在要分配许可证的用户或组的旁边单击。  应会看到绿色勾号。
8. 单击该勾号图标以保存更改。
   ![分配许可证](./media/multi-factor-authentication-get-started-assign-licenses/assign4.png)
9. 应会看到一条消息指出分配了多少个许可证，以及有多少个许可证分配失败。  单击“确定” 。
   ![分配许可证](./media/multi-factor-authentication-get-started-assign-licenses/assign5.png)

## <a name="next-steps"></a>后续步骤

- 有关详细信息，请参阅[什么是 Microsoft Azure Active Directory 许可？](../active-directory/active-directory-licensing-whatis-azure-portal.md)
