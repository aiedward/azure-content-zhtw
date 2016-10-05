<properties
	pageTitle="Azure AD v2.0 隱含流程 |Microsoft Azure"
	description="使用針對單一頁面應用程式的隱含流程的 Azure AD v2.0 實作，建置 Web 應用程式。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="msmbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# 2\.0 通訊協定 - 使用隱含流程的 SPA
使用 v2.0 端點，您可以讓具有 Microsoft 的個人和工作/學校帳戶的使用者登入您的單一頁面 app。主要在瀏覽器上執行的單一頁面和其他 JavaScript 應用程式，在驗證時會面臨一些有趣的挑戰：

- 這些應用程式的安全性特性與傳統的伺服器架構 Web 應用程式大不相同。
- 許多授權伺服器與身分識別提供者不支援 CORS 要求。
- 重新導向離開應用程式的完整網頁瀏覽器變得對使用者經驗特別有侵入性。

對於這些應用程式 (AngularJS、Ember.js、React.js 等)，Azure AD 支援 OAuth 2.0 隱含授權流程。隱含流程相關說明，請參閱 [OAuth 2.0 規格](http://tools.ietf.org/html/rfc6749#section-4.2)。其主要優點是它可讓應用程式從 Azure AD 取得權杖，不需要執行後端伺服器認證交換。這可讓應用程式登入使用者、維護工作階段，並且取得用戶端 JavaScript 程式碼中所有其他 Web API 的權杖。使用隱含流程時有幾個重要的安全性考量 - 特別是關於[用戶端](http://tools.ietf.org/html/rfc6749#section-10.3)和[使用者模擬](http://tools.ietf.org/html/rfc6749#section-10.3)。

如果您想要使用隱含流程與 Azure AD 將驗證新增至 JavaScript 應用程式，建議您使用我們的開放原始碼 JavaScript 程式庫，[adal.js](https://github.com/AzureAD/azure-activedirectory-library-for-js)。[這裡](active-directory-appmodel-v2-overview.md#getting-started)有幾個 AngularJS 教學課程可以幫助您入門。

不過，如果您不想使用單一頁面應用程式中的程式庫，並且自行傳送通訊協定訊息，請遵循下列一般步驟。

> [AZURE.NOTE]
	v2.0 端點並非支援每個 Azure Active Directory 案例和功能。若要判斷是否應該使用 v2.0 端點，請閱讀相關的 [v2.0 限制](active-directory-v2-limitations.md)。
    
## 通訊協定圖表
整個隱含登入流程看起來類似下列圖表 - 以下將詳細說明每個步驟。

![OpenId Connect 區隔線](../media/active-directory-v2-flows/convergence_scenarios_implicit.png)

## 傳送登入要求

若要一開始將使用者登入您的應用程式，您可以傳送 [OpenID Connect](active-directory-v2-protocols-oidc.md) 授權要求，以及從 v2.0 端點取得 `id_token`：

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token+token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&scope=openid%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&response_mode=fragment
&state=12345
&nonce=678910
```

> [AZURE.TIP] 按一下下面的連結以執行此要求！ 登入之後，您的瀏覽器應重新導向至在位址列中有 `id_token` 的 `https://localhost/myapp/`。<a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token+token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=openid%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&response_mode=fragment&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| 參數 | | 說明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必要 | 要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。允許的值為 `common`、`organizations`、`consumers` 及租用戶識別碼。如需更多詳細資訊，請參閱[通訊協定基本概念](active-directory-v2-protocols.md#endpoints)。 |
| client\_id | 必要 | 註冊入口網站 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com)) 指派給應用程式的應用程式識別碼。 |
| response\_type | 必要 | 必須包含 OpenID Connect 登入的 `id_token`。它也可能包含 response\_type `token`。這裡使用 `token`，讓您的應用程式能夠立即從授權端點接收存取權杖，而不需要向授權端點進行第二次要求。如果您使用 `token` response\_type，`scope` 參數必須包含範圍，以指出要對哪個資源發出權杖。 |
| redirect\_uri | 建議使用 | 應用程式的 redirect\_uri，您的應用程式可在此傳送及接收驗證回應。其必須完全符合您在入口網站中註冊的其中一個 redirect\_uris，不然就必須得是編碼的 url。 |
| scope | 必要 | 範圍的空格分隔清單。針對 OpenID Connect，即必須包含範圍 `openid`，其會在同意 UI 中轉譯成「讓您登入」權限。(選擇性) 您可能也想要納入 `email` 或 `profile` [範圍](active-directory-v2-scopes.md)，以授與其他使用者資料的存取權。您也可以在此要求中包含其他範圍，以要求同意各種資源。 |
| response\_mode | 建議使用 | 指定將產生的權杖送回到應用程式所應該使用的方法。對於隱含流程應該是 `fragment`。 |
| state | 建議使用 | 同樣會隨權杖回應傳回之要求中所包含的值。其可以是您想要之任何內容的字串。隨機產生的唯一值通常用於[防止跨站台要求偽造攻擊](http://tools.ietf.org/html/rfc6749#section-10.12)。此狀態也用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |
| nonce | 必要 | 由應用程式產生且包含在要求中的值，會以宣告方式包含在產生的 id\_token 中。應用程式接著便可確認此值，以減少權杖重新執行攻擊。此值通常是隨機的唯一字串，可用以識別要求的來源。 |
| prompt | 選用 | 表示需要的使用者互動類型。此時的有效值為「登入」、「無」和「同意」。`prompt=login` 會強制使用者在該要求上輸入認證，否定單一登入。`prompt=none` 則相反 - 它會確保不會對使用者顯示任何互動式提示。如果要求無法透過單一登入以無訊息方式完成，v2.0 端點會傳回錯誤。`prompt=consent` 會在使用者登入之後觸發 OAuth 同意對話方塊，詢問使用者是否要授與權限給應用程式。 |
| login\_hint | 選用 | 如果您事先知道其使用者名稱，可用來預先填入使用者登入頁面的使用者名稱/電子郵件地址欄位。通常應用程式會在重新驗證期間使用此參數，已經使用 `preferred_username` 宣告從上一個登入擷取使用者名稱。 |
| domain\_hint | 選用 | 可以是 `consumers` 或 `organizations` 其中一個。如果包含，它會略過使用者在 v2.0 登入頁面上經歷的以電子郵件為基礎的探索程序，導致稍微更佳流暢的使用者經驗。通常應用程式會在重新驗證期間使用此參數，方法是從 id\_token 擷取 `tid` 宣告。如果 `tid` 宣告值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，您應該使用 `domain_hint=consumers`。否則，使用 `domain_hint=organizations`。 |

此時，會要求使用者輸入其認證並完成驗證。v2.0 端點也會確保使用者已經同意 `scope` 查詢參數所示的權限。如果使用者未曾同意這些權限的任何一項，就會要求使用者同意要求的權限。[這裡提供權限、同意與多租用戶應用程式](active-directory-v2-scopes.md)的詳細資料。

一旦使用者驗證並同意，v2.0 端點就會使用 `response_mode` 參數中指定的方法，將回應傳回至位於指定所在 `redirect_uri` 的應用程式。

#### 成功回應

使用 `response_mode=fragment` 和 `response_type=id_token+token` 的成功回應如下所示 (內含換行符號以利閱讀)：

```
GET https://localhost/myapp/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&token_type=Bearer
&expires_in=3599
&scope=https%3a%2f%2fgraph.microsoft.com%2fmail.read 
&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=12345
```

| 參數 | 說明 |
| ----------------------- | ------------------------------- |
| access\_token | 如果 `response_type` 包含 `token` 則納入。應用程式要求的存取權杖，在此案例中為 Microsoft Graph 的存取權杖。存取權杖不得進行解碼或檢查，可被視為不透明的字串。 |
| token\_type | 如果 `response_type` 包含 `token` 則納入。一律為 `Bearer`。 |
| expires\_in | 如果 `response_type` 包含 `token` 則納入。表示權杖有效的秒數 (針對快取目的)。 |
| scope | 如果 `response_type` 包含 `token` 則納入。表示在 access\_token 的有效範圍。 |
| id\_token | 應用程式要求的 id\_token。您可以使用 id\_token 確認使用者的身分識別，並以使用者開始工作階段。如需 Id\_token 及其內容的詳細資訊，請參閱 [v2.0 端點權杖參考](active-directory-v2-tokens.md)。 |
| state | 如果要求中包含狀態參數，回應中就應該出現相同的值。應用程式應該確認要求和回應中的狀態值完全相同。 |


#### 錯誤回應
錯誤回應可能也會傳送至 `redirect_uri`，讓應用程式可以適當地處理：

```
GET https://localhost/myapp/#
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| 參數 | 說明 |
| ----------------------- | ------------------------------- |
| 錯誤 | 用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error\_description | 協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |

## 驗證 id\_token
僅接收 id\_token 不足以驗證使用者，您必須驗證 id\_token 簽章，並依照應用程式的需求確認權杖中的宣告。v2.0 端點使用 [JSON Web Tokens (JWT)](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 和公開金鑰加密簽署權杖及驗證其是否有效。

您可以選擇驗證用戶端程式碼中的 `id_token`，但是常見的作法是將 `id_token` 傳送至後端伺服器，並且在那裡執行驗證。一旦驗證了 id\_token 的簽章，就會有數項宣告需要驗證。如需詳細資訊，請參閱 [v2.0 權杖參考](active-directory-v2-tokens.md)，包括[驗證權杖](active-directory-v2-tokens.md#validating-tokens)和[有關簽署金鑰變換的重要資訊](active-directory-v2-tokens.md#validating-tokens)。我們建議利用程式庫來剖析和驗證權杖 - 對於大部分語言和平台至少有一個可用。
<!--TODO: Improve the information on this-->

您可能也希望根據自己的案例驗證其他宣告。一些常見的驗證包括：

- 確保使用者/組織已註冊應用程式。
- 確保使用者擁有正確的授權/權限
- 確保驗證具有特定強度，例如多重要素驗證。

如需 id\_token 中宣告的詳細資訊，請參閱 [v2.0 端點權杖參考](active-directory-v2-tokens.md)。

一旦驗證完畢 id\_token，即可利用使用者開始工作階段，並使用 id\_token 中的宣告來取得應用程式中的使用者相關資訊。這項資訊可以用於顯示、記錄、授權等等。

## 取得存取權杖

您已經將使用者註冊到單一頁面應用程式，您可以取得存取權杖以呼叫受到 Azure AD 保護的 Web API，例如 [Microsoft Graph](https://graph.microsoft.io)。即使您已經收到使用 `token` response\_type 的權杖，仍可使用此方法來取得其他資源的權杖，而不需重新導向使用者進行再次登入。

在正常的 OpenID Connect/OAuth 流程中，您可以藉由對 v2.0 `/token` 端點進行要求來完成這個操作。不過，v2.0 端點不支援 CORS 要求，因此進行 AJAX 呼叫以取得和重新整理權杖是不可能的。相反地，您可以在隱藏的 iframe 中使用隱含流程，為其他 Web API 取得新權杖：

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&response_mode=fragment
&state=12345&nonce=678910
&prompt=none
&domain_hint=organizations
&login_hint=myuser@mycompany.com
```

> [AZURE.TIP] 請嘗試將下列要求貼至瀏覽器！ (別忘了以您的使用者的正確值取代 `domain_hint` 和 `login_hint` 值)

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&response_mode=fragment&state=12345&nonce=678910&prompt=none&domain_hint={{consumers-or-organizations}}&login_hint={{your-username}}
```

| 參數 | | 說明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必要 | 要求路徑中的 `{tenant}` 值可用來控制可登入應用程式的人員。允許的值為 `common`、`organizations`、`consumers` 及租用戶識別碼。如需更多詳細資訊，請參閱[通訊協定基本概念](active-directory-v2-protocols.md#endpoints)。 |
| client\_id | 必要 | 註冊入口網站 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com)) 指派給應用程式的應用程式識別碼。 |
| response\_type | 必要 | 必須包含 OpenID Connect 登入的 `id_token`。它也可能包含其他 response\_types，例如 `code`。 |
| redirect\_uri | 建議使用 | 應用程式的 redirect\_uri，您的應用程式可在此傳送及接收驗證回應。其必須完全符合您在入口網站中註冊的其中一個 redirect\_uris，不然就必須得是編碼的 url。 |
| scope | 必要 | 範圍的空格分隔清單。若要取得權杖，請包含您感興趣資源要求的所有[範圍](active-directory-v2-scopes.md)。 |
| response\_mode | 建議使用 | 指定將產生的權杖送回到應用程式所應該使用的方法。可以是 `query`、`form_post` 或 `fragment` 其中一個。 |
| state | 建議使用 | 同樣會隨權杖回應傳回之要求中所包含的值。其可以是您想要之任何內容的字串。隨機產生的唯一值通常用於防止跨站台要求偽造攻擊。此狀態也用於在驗證要求出現之前，於應用程式中編碼使用者的狀態資訊，例如之前所在的網頁或檢視。 |
| nonce | 必要 | 由應用程式產生且包含在要求中的值，會以宣告方式包含在產生的 id\_token 中。應用程式接著便可確認此值，以減少權杖重新執行攻擊。此值通常是隨機的唯一字串，可用以識別要求的來源。 |
| prompt | 必要 | 若要重新整理並取得隱藏 iframe 中的權杖，您應該使用 `prompt=none` 以確保 iframe 不會懸置在 v2.0 登入頁面上，並立即返回。 |
| login\_hint | 必要 | 如需重新整理並取得隱藏 iframe 中的權杖，您必須在此提示中包含使用者的使用者名稱，以便區分使用者在特定時間點可能具有的多個工作階段。您可以使用 `preferred_username` 宣告擷取先前登入的使用者名稱。 |
| domain\_hint | 必要 | 可以是 `consumers` 或 `organizations` 其中一個。若要重新整理並取得隱藏 iframe 中的權杖，您必須在要求中包含 domain\_hint。您應該從先前登入的 id\_token 擷取 `tid` 宣告，以判斷要使用哪一個值。如果 `tid` 宣告值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，您應該使用 `domain_hint=consumers`。否則，使用 `domain_hint=organizations`。 |

由於 `prompt=none` 參數，此要求會立即成功或失敗，並且傳回給您的應用程式。會使用 `response_mode` 參數中指定的方法，將成功的回應傳送至指定的 `redirect_uri` 給您的應用程式。

#### 成功回應
使用 `response_mode=fragment` 的成功回應如下所示：

```
GET https://localhost/myapp/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=12345
&token_type=Bearer
&expires_in=3599
&scope=https%3A%2F%2Fgraph.windows.net%2Fdirectory.read
```

| 參數 | 說明 |
| ----------------------- | ------------------------------- |
| access\_token | 應用程式要求的權杖。 |
| token\_type | 一律為 `Bearer`。 |
| state | 如果要求中包含狀態參數，回應中就應該出現相同的值。應用程式應該確認要求和回應中的狀態值完全相同。 |
| expires\_in | 存取權杖的有效期 (以秒為單位)。 |
| scope | 存取權杖有效的範圍。 |

#### 錯誤回應
錯誤回應可能也會傳送至 `redirect_uri`，讓應用程式可以適當地處理。如果是 `prompt=none`，預期的錯誤為：

```
GET https://localhost/myapp/#
error=user_authentication_required
&error_description=the+request+could+not+be+completed+silently
```

| 參數 | 說明 |
| ----------------------- | ------------------------------- |
| 錯誤 | 用以分類發生的錯誤類型與回應錯誤的錯誤碼字串。 |
| error\_description | 協助開發人員識別驗證錯誤根本原因的特定錯誤訊息。 |

如果您在 iframe 要求中收到此錯誤，使用者必須再次以互動方式登入以擷取新的權杖。您可以選擇對於您的應用程式合理的任何方式處理這種情況。

## 重新整理權杖

`id_token` 和 `access_token` 馬上就會到期，因此您的應用程式必須準備好定期重新整理這些權杖。若要重新整理其中任何一個類型的權杖，您可以使用 `prompt=none` 參數來控制 Azure AD 的行為，執行上述的相同隱藏的 iframe 要求。如果您想要收到新的 `id_token`，務必使用 `response_type=id_token` 和 `scope=openid`，以及 `nonce` 參數。


## 傳送登出要求

v2.0 端點目前不支援 OpenIdConnect `end_session_endpoint`。這表示應用程式無法向 v2.0 端點傳送要求，而無法結束使用者工作階段及清除 v2.0 端點設定的 Cookie。若要將使用者登出，應用程式只需結束自身的使用者工作階段，並完整地將使用者工作階段留給 v2.0 端點即可。下次使用者嘗試登入時，就會看到列出其主動登入帳戶的 [選擇帳戶] 頁面。在該頁面上，使用者可以選擇登出任一帳戶，結束 v2.0 端點的工作階段。

<!--

When you wish to sign the user out of the  app, it is not sufficient to clear your app's cookies or otherwise end the session with the user.  You must also redirect the user to the v2.0 endpoint for sign out.  If you fail to do so, the user will be able to re-authenticate to your app without entering their credentials again, because they will have a valid single sign-on session with the v2.0 endpoint.

You can simply redirect the user to the `end_session_endpoint` listed in the OpenID Connect metadata document:

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/logout?
post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
```

| Parameter | | Description |
| ----------------------- | ------------------------------- | ------------ |
| post_logout_redirect_uri | recommended | The URL which the user should be redirected to after successful logout.  If not included, the user will be shown a generic message by the v2.0 endpoint.  |

-->

<!---HONumber=AcomDC_0921_2016-->