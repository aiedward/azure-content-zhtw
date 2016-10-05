<properties
   pageTitle="如何執行存取權檢閱 | Microsoft Azure"
   description="了解如何使用 Azure Privileged Identity Management 應用程式來執行檢閱。"
   services="active-directory"
   documentationCenter=""
   authors="kgremban"
   manager="femila"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="kgremban"/>

# 如何在 Azure AD Privileged Identity Management 中執行存取權檢閱

Azure Active Directory (AD) Privileged Identity Management 簡化了企業管理以特殊權限身分存取 Azure AD 中的資源和其他 Microsoft 線上服務 (如 Office 365 或 Microsoft Intune) 的方式。

如果您已被指派系統管理角色，貴組織的特殊權限角色管理員可能會要求您定期確認您仍需要該角色來執行作業。您可能會收到包含連結的電子郵件，或請直接移至 [Azure 入口網站](https://portal.azure.com)。請遵循本文中的步驟，執行獲指派角色的自我檢閱。

如果您是對存取權檢閱感興趣的特殊權限角色管理員，請在[如何開始存取權檢閱](active-directory-privileged-identity-management-how-to-start-security-review.md)中取得更多詳細資訊。

## 加入 Privileged Identity Management 應用程式

您可以使用 [Azure 入口網站](https://portal.azure.com/)中的 Azure AD Privileged Identity Management (PIM) 應用程式來執行檢閱。如果入口網站中沒有 Azure AD Privileged Identity Management 應用程式，請遵循這些步驟操作。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 選取 Azure 入口網站右上角的使用者名稱，然後選取您要操作的目錄。
3. 選取 [更多服務] 並使用 [篩選器] 文字方塊來搜尋 [Azure AD Privileged Identity Management]。
4. 選取 [釘選到儀表板]，然後按一下 [建立]。Privileged Identity Management 應用程式隨即開啟。


## 核准或拒絕存取

當您核准或拒絕存取權時，只是在告訴檢閱者您是否仍然使用此角色。如果您想要繼續擔任此角色，請選擇 [核准]，如果您已不再需要此存取權，則請選擇 [拒絕]。您的狀態不會立即變更，必須等到檢閱者套用結果之後才會變更。請依照下列步驟來尋找並完成存取權檢閱︰

1. 在 PIM 應用程式中，選取 [檢閱特殊存取權限]。如果您有任何擱置中的存取權檢閱時，它們會顯示在 Azure AD 的 [存取權檢閱] 刀鋒視窗中。
2. 選取您想要完成的檢閱。
3. 除非該檢閱是您所建立，否則您會顯示為該檢閱中的唯一使用者。選取您名稱旁邊的核取記號。
4. 選擇 [核准] 或 [拒絕]。您可能需要在 [提供原因] 文字方塊中提供您的決定原因。
5. 關閉 [檢閱 Azure AD 角色] 刀鋒視窗。


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 後續步驟
[AZURE.INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!--Image references-->

[1]: ./media/active-directory-privileged-identity-management-configure/PIM_EnablePim.png

<!---HONumber=AcomDC_0921_2016-->