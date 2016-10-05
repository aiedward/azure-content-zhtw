<properties
	pageTitle="Azure AD v2.0 .NET Web 應用程式 | Microsoft Azure"
	description="如何建置可使用個人 Microsoft 帳戶及工作或學校帳戶登入使用者的 .NET MVC Web 應用程式。"
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# 將登入新增至 .NET MVC Web 應用程式

v2.0 端點可讓您快速地將驗證新增至您的 Web 應用程式，同時支援個人 Microsoft 帳戶以及工作或學校帳戶。在 ASP.NET Web 應用程式中，您可以使用隨附於 .NET Framework 4.5 的 Microsoft OWIN 中介軟體來完成此項作業。

> [AZURE.NOTE]
	v2.0 端點並非支援每個 Azure Active Directory 案例和功能。如果要判斷是否應該使用 v2.0 端點，請閱讀 [v2.0 限制](active-directory-v2-limitations.md)。

 現在，我們將組建使用 OWIN 將使用者登入、顯示使用者的部分相關資訊，以及將使用者登出應用程式的 Web 應用程式。
 
 ## 本教學課程的「下載程式碼」記錄於 [GitHub](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet)。若要遵循執行，您可以[用 .zip 格式下載應用程式的基本架構](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet/archive/skeleton.zip)，或複製基本架構：

```git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet.git```

本教學課程最後也會提供完整的應用程式。

## 註冊應用程式
在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 建立新的應用程式，或遵循下列[詳細步驟](active-directory-v2-app-registration.md)。請確定：

- 將指派給您應用程式的**應用程式識別碼**複製起來，您很快會需要用到這些識別碼。
- 為您的應用程式新增 **Web** 平台。
- 輸入正確的**重新導向 URI**。重新導向 URI 會向 Azure AD 指出驗證回應應導向的位置，本教學課程的預設為 `https://localhost:44326/`。

## 安裝及設定 OWIN 驗證
在這裡，我們將設定 OWIN 中介軟體使用 OpenID Connect 驗證通訊協定。OWIN 將用來發出登入和登出要求、管理使用者的工作階段，以及取得使用者相關資訊等其他作業。

-	若要開始，請開啟專案根目錄中的 `web.config` 檔案，並在 `<appSettings>` 區段中輸入應用程式的組態值。
    -	`ida:ClientId` 是在註冊入口網站中指派給應用程式的**應用程式識別碼**。
    -	`ida:RedirectUri` 是您在入口網站中輸入的**重新導向 URI**。

-	接下來，使用 Package Manager Console 將Next, add the OWIN 中介軟體 NuGet 套件新增到專案中。

```
PM> Install-Package Microsoft.Owin.Security.OpenIdConnect
PM> Install-Package Microsoft.Owin.Security.Cookies
PM> Install-Package Microsoft.Owin.Host.SystemWeb
```

-	將「OWIN 啟動類別」加入名為 `Startup.cs` 的專案。以滑鼠右鍵按一下專案 --> [**新增**] --> [ **新增項目**] --> 搜尋「OWIN」。OWIN 中介軟體將會在應用程式啟動時叫用 `Configuration(...)` 方法。
-	將類別宣告變更為 `public partial class Startup`，我們已為您在另一個檔案中實作了此類別的一部分。在 `Configuration(...)` 方法中，請呼叫 ConfigureAuth(...)，為您的 Web 應用程式設定驗證。  

```C#
[assembly: OwinStartup(typeof(Startup))]

namespace TodoList_WebApp
{
	public partial class Startup
	{
		public void Configuration(IAppBuilder app)
		{
			ConfigureAuth(app);
		}
	}
}
```

-	開啟檔案 `App_Start\Startup.Auth.cs` 並實作 `ConfigureAuth(...)` 方法。您在 `OpenIdConnectAuthenticationOptions` 中所提供的參數將會做為您的應用程式與 Azure AD 進行通訊的座標使用。您還必須設定 Cookie 驗證，OpenID Connect 中介軟體會在表面下使用 Cookie。

```C#
public void ConfigureAuth(IAppBuilder app)
			 {
					 app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

					 app.UseCookieAuthentication(new CookieAuthenticationOptions());

					 app.UseOpenIdConnectAuthentication(
							 new OpenIdConnectAuthenticationOptions
							 {
									 // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0 
									 // The `Scope` describes the permissions that your app will need.  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/
									 // In a real application you could use issuer validation for additional checks, like making sure the user's organization has signed up for your app, for instance.

									 ClientId = clientId,
									 Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0 "),
									 RedirectUri = redirectUri,
									 Scope = "openid email profile",
									 ResponseType = "id_token",
									 PostLogoutRedirectUri = redirectUri,
									 TokenValidationParameters = new TokenValidationParameters
									 {
											 ValidateIssuer = false,
									 },
									 Notifications = new OpenIdConnectAuthenticationNotifications
									 {
											 AuthenticationFailed = OnAuthenticationFailed,
									 }
							 });
			 }
```

## 傳送驗證要求
您的應用程式現在已正確設定，將使用 OpenID Connect 驗證通訊協定與 v2.0 端點通訊。OWIN 已經處理所有製作驗證訊息、驗證 Azure AD 的權杖和維護使用者工作階段的瑣碎詳細資料。剩餘的工作就是提供使用者一個登入和登出的方式。

- 您可以在控制器中使用授權標記，要求使用者在存取特定頁面時登入。開啟 `Controllers\HomeController.cs`，並在 [關於] 控制器中加入 `[Authorize]` 標記。

```C#
[Authorize]
public ActionResult About()
{
  ...
```

-	您也可以使用 OWIN 從程式碼中直接發出驗證要求。開啟 `Controllers\AccountController.cs`。在 SignIn() 和 SignOut() 動作中，將分別發出 OpenID Connect 挑戰和登出要求。

```C#
public void SignIn()
{
    // Send an OpenID Connect sign-in request.
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(new AuthenticationProperties { RedirectUri = "/" }, OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}

// BUGBUG: Ending a session with the v2.0 endpoint is not yet supported.  Here, we just end the session with the web app.  
public void SignOut()
{
    // Send an OpenID Connect sign-out request.
    HttpContext.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
    Response.Redirect("/");
}
```

-	現在，請開啟 `Views\Shared\_LoginPartial.cshtml`。這裡是您向使用者顯示應用程式的登入和登出連結，以及在檢視中列印出使用者名稱的位置。

```HTML
@if (Request.IsAuthenticated)
{
    <text>
        <ul class="nav navbar-nav navbar-right">
            <li class="navbar-text">

                @*The 'preferred_username' claim can be used for showing the user's primary way of identifying themselves.*@

                Hello, @(System.Security.Claims.ClaimsPrincipal.Current.FindFirst("preferred_username").Value)!
            </li>
            <li>
                @Html.ActionLink("Sign out", "SignOut", "Account")
            </li>
        </ul>
    </text>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
    </ul>
}
```

## 顯示使用者資訊
使用 OpenID Connect 驗證使用者時，v2.0 端點會將 id\_token 傳回至包含[宣告](active-directory-v2-tokens.md#id_tokens)的應用程式，或有關使用者的判斷提示。您可以使用這些宣告來個人化應用程式：

- 開啟 `Controllers\HomeController.cs` 檔案。您可以透過 `ClaimsPrincipal.Current` 安全性主體物件來存取控制器中的使用者宣告。

```C#
[Authorize]
public ActionResult About()
{
    ViewBag.Name = ClaimsPrincipal.Current.FindFirst("name").Value;

    // The object ID claim will only be emitted for work or school accounts at this time.
    Claim oid = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier");
    ViewBag.ObjectId = oid == null ? string.Empty : oid.Value;

    // The 'preferred_username' claim can be used for showing the user's primary way of identifying themselves
    ViewBag.Username = ClaimsPrincipal.Current.FindFirst("preferred_username").Value;

    // The subject or nameidentifier claim can be used to uniquely identify the user
    ViewBag.Subject = ClaimsPrincipal.Current.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;

    return View();
}
```

## 執行

最後，建置並執行您的應用程式！ 使用個人 Microsoft 帳戶或工作或學校帳戶登入，並注意上方導覽列中使用者身分識別的反映狀態。您的 Web 應用程式現在使用業界標準的通訊協定保護，可以使用個人與工作/學校帳戶來驗證使用者。

如需參考，[此處以 .zip 格式提供](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet/archive/complete.zip)完整範例 (不含您的組態值)，您也可以從 GitHub 將其複製：

```git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet.git```

## 後續步驟

您現在可以進入更進階的主題。您可以嘗試：

[使用 v2.0 端點保護 Web API >>](active-directory-devquickstarts-webapi-dotnet.md)

如需其他資源，請參閱：
- [v2.0 開發人員指南 >>](active-directory-appmodel-v2-overview.md)
- [StackOverflow "azure-active-directory" 標記 >>](http://stackoverflow.com/questions/tagged/azure-active-directory)

## 取得產品的安全性更新

我們鼓勵您造訪[此頁面](https://technet.microsoft.com/security/dd252948)並訂閱資訊安全摘要報告警示，以在安全性事件發生時收到通知。

<!---HONumber=AcomDC_0921_2016-->