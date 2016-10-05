<properties
	pageTitle="Azure Active Directory Identity Protection - 如何解鎖使用者 | Microsoft Azure"
	description="了解如何解鎖由 Azure Active Directory Identity Protection 原則所封鎖的使用者。"
	services="active-directory"
	keywords="Azure Active Directory Identity Protection，解鎖使用者"
	documentationCenter=""
	authors="markusvi"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="markvi"/>

#Azure Active Directory Identity Protection - 如何解鎖使用者

使用 Azure Active Directory Identity Protection 時，如果符合設定的條件，您就可以設定原則來封鎖使用者。一般而言，遭封鎖的使用者會聯絡服務台以解除封鎖。本主題說明解鎖遭封鎖使用者的執行步驟。


## 判斷封鎖的原因

解鎖使用者的第一個步驟，您需要判斷封鎖使用者的原則類型為何，以決定後續步驟。使用 Azure Active Directory Identity Protection 時，使用者可能是因登入風險原則或使用者風險原則而遭封鎖。

您可以從使用者嘗試登入期間所出現的對話方塊標題取得封鎖使用者的原則類型︰

|原則 | 使用者對話方塊|
|--- | --- |
|登入風險 | ![封鎖的登入](./media/active-directory-identityprotection-unblock-howto/02.png) |
|使用者風險 | ![封鎖的帳戶](./media/active-directory-identityprotection-unblock-howto/104.png) |


封鎖使用者的類型為︰

- 登入風險原則，也就是可疑的登入
- 使用者風險原則，也就是有風險的帳戶

 
## 解鎖可疑的登入

若要解鎖可疑的登入，您有下列選擇︰

1. **從熟悉的位置或裝置登入** - 可疑的登入會遭封鎖通常是因為使用者嘗試從不熟悉的位置或裝置登入。使用者可以嘗試從熟悉的位置或裝置登入，以迅速判斷這是否是遭封鎖的原因。


3. **從原則中排除** - 如果您認為目前的登入原則設定對特定使用者造成問題，您可以排除這些使用者。如需詳細資訊，請參閱[登入風險原則](active-directory-identityprotection.md#sign-in-risk-policy)。
 
4. **停用原則** - 如果您認為您的原則設定對所有使用者造成問題，您可以停用原則。如需詳細資訊，請參閱[登入風險原則](active-directory-identityprotection.md#sign-in-risk-policy)。


## 解鎖有風險的帳戶

若要解鎖有風險的帳戶，您有下列選擇︰

1. **重設密碼** - 您可以重設使用者的密碼。如需詳細資訊，請參閱[手動安全密碼重設](active-directory-identityprotection.md#manual-secure-password-reset)。

2. **關閉所有風險事件** - 如果已達到設定的封鎖存取權限之使用者風險層級，使用者風險原則就會封鎖使用者。您可以手動關閉已報告的風險事件來降低使用者的風險層級。如需詳細資訊，請參閱[手動關閉風險事件](active-directory-identityprotection.md#closing-risk-events-manually)。

3. **從原則中排除** - 如果您認為目前的登入原則設定對特定使用者造成問題，您可以排除這些使用者。如需詳細資訊，請參閱[使用者風險原則](active-directory-identityprotection.md#user-risk-policy)。
 
4. **停用原則** - 如果您認為您的原則設定對所有使用者造成問題，您可以停用原則。如需詳細資訊，請參閱[使用者風險原則](active-directory-identityprotection.md#user-risk-policy)。




## 後續步驟

 您想要深入了解 Azure AD Identity Protection？ 查看 [Azure Active Directory Identity Protection](active-directory-identityprotection.md)。
 

<!---HONumber=AcomDC_0921_2016-->