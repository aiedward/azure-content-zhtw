<properties
	pageTitle="開始使用 Azure AD Windows 市集 | Microsoft Azure"
	description="如何建置 Windows 市集應用程式來與 Azure AD 整合進行登入，並使用 OAuth 呼叫受 Azure AD 保護的 API。"
	services="active-directory"
	documentationCenter="windows"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="mobile-windows-store"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>


# 整合 Azure AD 與 Windows 市集應用程式

[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

如果您正在開發 Windows 市集應用程式，Azure AD 可讓您簡單又直截了當以 Active Directory 帳戶驗證您的使用者。它也可讓您的應用程式安全地使用任何受 Azure AD 保護的 Web API，例如 Office 365 API 或 Azure API。

對於需要存取受保護資源的 Windows 市集桌面應用程式，Azure AD 提供 Active Directory 驗證程式庫 (ADAL)。ADAL 存在的唯一目的是為了讓您的應用程式輕鬆取得存取權杖。為了示範究竟有多麼簡單，我們將建置一個可執行下列動作的「目錄搜尋程式」Windows 市集應用程式：

-	使用 [OAuth 2.0 驗證通訊協定](https://msdn.microsoft.com/library/azure/dn645545.aspx)取得呼叫 Azure AD Graph API 的存取權杖。
-	搜尋目錄以尋找具有指定 UPN 的使用者。
-	將使用者登出。

若要建立完整可用的應用程式，您必須：

2. 向 Azure AD 註冊您的應用程式。
3. 安裝及設定 ADAL。
5. 使用 ADAL 來取得 Azure AD 的權杖。

若要開始使用，請[下載基本架構專案](https://github.com/AzureADQuickStarts/NativeClient-WindowsStore/archive/skeleton.zip)或[下載完整的範例](https://github.com/AzureADQuickStarts/NativeClient-WindowsStore/archive/complete.zip)。每一個都是 Visual Studio 2015 解決方案。您還需要一個可以建立使用者並註冊應用程式的 Azure AD 租用戶。如果您還沒有租用戶，[了解如何取得租用戶](active-directory-howto-tenant.md)。

## *1.註冊目錄搜尋應用程式*
若要啟用您的應用程式以取得權杖，您必須先在 Azure AD 租用戶中進行註冊，並對其授與存取 Azure AD Graph API 的權限：

-	登入 [Azure 管理入口網站](https://manage.windowsazure.com)
-	在左側導覽中按一下 **Active Directory**
-	選取要在其中註冊應用程式的租用戶。
-	按一下 [**應用程式**] 索引標籤，然後按一下最下面抽屜的 [**新增**]。
-	遵照提示進行，並建立新的**原生用戶端應用程式**。
    -	應用程式的 [**名稱**] 將對使用者說明您的應用程式
    -	「**重新導向 Uri**」是配置和字串的組合，Azure AD 可用它來傳回權杖回應。現在請先輸入預留位置值，例如 `http://DirectorySearcher`。我們稍後將會取代此值。
-	完成註冊後，AAD 會為您的應用程式指派一個唯一用戶端識別碼。您在後續章節中將會用到這個值，所以請從 [**設定**] 索引標籤中複製此值。
- 此外，在 [**設定**] 索引標籤上找到 [其他應用程式的權限] 區段。對於 "Azure Active Directory" 應用程式，在 [委派權限] 下，加入 [以登入使用者的身分存取目錄] 權限。這樣做可讓您的應用程式查詢 Graph API 的使用者。

## *2.安裝及設定 ADAL*
既然您在 Azure AD 中已經擁有應用程式，您可以安裝 ADAL，並撰寫身分識別相關程式碼。為了讓 ADAL 能與 Azure AD 通訊，您需要提供一些應用程式註冊的相關資訊。
-	首先，使用 Package Manager Console 將 ADAL 新增到 DirectorySearcher 專案中。

```
PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
```

-	在 DirectorySearcher 專案中，開啟 `MainPage.xaml.cs`。取代 [`Config Values`] 區域中的值以反映您在 Azure 入口網站中所輸入的值。每當使用 ADAL 時，您的程式碼便會參考這些值。
    -	`tenant` 是指您的 Azure AD 租用戶網域，例如 contoso.onmicrosoft.com
    -	`clientId` 是指您從入口網站複製的應用程式 clientId。
-	您現在必須找出 Windows 市集應用程式的回呼 uri。在 `MainPage` 方法的這一行上設定中斷點：

```
redirectURI = Windows.Security.Authentication.Web.WebAuthenticationBroker.GetCurrentApplicationCallbackUri();
```
- 建置方案，並確認還原了封裝的所有參考。如果遺失封裝，請開啟 Nuget 封裝管理員，並還原套件。
- 執行應用程式，並在觸及中斷點時複製 `redirectUri` 的值並將其擱置在一旁。您應該會看到類似下面的畫面

```
ms-app://s-1-15-2-1352796503-54529114-405753024-3540103335-3203256200-511895534-1429095407/
```

- 回到 Azure 管理入口網站中應用程式的 [**設定**] 索引標籤上，使用這個值取代 **RedirectUri** 的值。

## *3.使用 ADAL 來取得 AAD 的權杖*
ADAL 的基本原則是每當您的應用程式需要存取權杖時，它只需呼叫 `authContext.AcquireToken(…)`，ADAL 就會進行其餘工作。

-	第一步是初始化應用程式的 `AuthenticationContext` - ADAL 的主要類別。您在這裡將 ADAL 與 Azure AD 通訊所需的座標傳給 ADAL，並告訴它如何快取權杖。

```C#
public MainPage()
{
    ...

    authContext = new AuthenticationContext(authority);
}
```

- 現在請找到 `Search(...)` 方法，這是在使用者按一下應用程式 UI 的 [搜尋] 按鈕時所叫用的方法。這個方法會對 Azure AD Graph API 提出 GET 要求，以查詢 UPN 開頭為指定搜尋詞彙的使用者。但為了能夠查詢 Graph API，要求的 `Authorization` 標頭必須包含 access\_token - ADAL 可以提供這方面的協助。

```C#
private async void Search(object sender, RoutedEventArgs e)
{
    ...
    AuthenticationResult result = null;
    try
    {
        result = await authContext.AcquireTokenAsync(graphResourceId, clientId, redirectURI, new PlatformParameters(PromptBehavior.Auto, false));
    }
    catch (AdalException ex)
    {
        if (ex.ErrorCode != "authentication_canceled")
        {
            ShowAuthError(string.Format("If the error continues, please contact your administrator.\n\nError: {0}\n\nError Description:\n\n{1}", ex.ErrorCode, ex.Message));
        }
        return;
    }
    ...
}
```
- 當您的應用程式透過呼叫 `AcquireTokenAsync(...)` 要求權杖時，ADAL 會嘗試在不要求使用者認證的情況下傳回權杖。如果 ADAL 決定使用者需要登入才能取得權杖，它會顯示登入對話方塊、收集使用者的認證，並在成功驗證後傳回權杖。如果基於任何原因 ADAL 無法傳回權杖，則 `AuthenticationResult` 狀態就會發生錯誤。

- 現在可以開始使用您剛才取得的 access\_token。另外，在 `Search(...)` 方法中，將您取得的權杖附加至 Graph API GET 要求的授權標頭中：

```C#
// Add the access token to the Authorization Header of the call to the Graph API, and call the Graph API.
httpClient.DefaultRequestHeaders.Authorization = new HttpCredentialsHeaderValue("Bearer", result.AccessToken);

```
- 您也可以使用 `AuthenticationResult` 物件，在應用程式中顯示使用者的相關資訊，例如使用者的識別碼：

```C#
// Update the Page UI to represent the signed in user
ActiveUser.Text = result.UserInfo.DisplayableId;
```
- 最後，您也可以使用 ADAL 來將使用者登出應用程式。當使用者按一下 [登出] 按鈕時，我們想要確保下次呼叫 `AcquireTokenAsync(...)` 時將會顯示登入檢視。有了 ADAL，這會和清除權杖快取一樣簡單：

```C#
private void SignOut()
{
    // Clear session state from the token cache.
    authContext.TokenCache.Clear();

    ...
}
```

恭喜！ 您現在擁有一個能夠驗證使用者、使用 OAuth 2.0 安全地呼叫 Web API，及取得使用者基本資訊的運作中 Windows 市集應用程式。如果您還沒有這麼做，現在正是將一些使用者植入租用戶的好時機。執行 DirectorySearcher 應用程式，並使用其中一個使用者登入。根據 UPN 搜尋其他使用者。關閉並重新執行應用程式。請注意，使用者工作階段會維持不變。登出 (按一下滑鼠右鍵以顯示底列)，再以其他使用者身分重新登入。

ADAL 可讓您輕鬆地將這些常見的身分識別功能全部納入您的應用程式。它會為您處理一切麻煩的事，包括快取管理、OAuth 通訊協定支援、向使用者顯示登入 UI、重新整理過期權杖等等。您唯一需要知道的就是單一 API 呼叫，`authContext.AcquireToken*(…)`。

[這裡](https://github.com/AzureADQuickStarts/NativeClient-WindowsStore/archive/complete.zip)提供完成的範例供您參考 (不含您的設定值)。您現在可以繼續其他身分識別案例。您可以嘗試：

[使用 Azure AD 保護 .NET Web API >>](active-directory-devquickstarts-webapi-dotnet.md)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=AcomDC_0921_2016-->