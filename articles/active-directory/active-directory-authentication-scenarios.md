
<properties
   pageTitle="Azure AD 的驗證案例 | Microsoft Azure"
   description="Azure Active Directory (AAD) 五個最常見驗證案例的概觀"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/16/2016"
   ms.author="mbaldwin"/>

# Azure AD 的驗證案例

Azure Active Directory (Azure AD) 提供身分識別做為服務來簡化開發人員的驗證工作，支援業界標準通訊協定，例如 OAuth 2.0 和 OpenID Connect，以及適用於不同平台的開放原始碼程式庫，協助您開始快速撰寫程式碼。本文件將協助您了解 Azure AD 支援的各種案例，並示範如何開始著手。分成下列各節：

- [Azure AD 中的驗證基本概念](#basics-of-authentication-in-azure-ad)

- [Azure AD 安全性權杖中的宣告](#claims-in-azure-ad-security-tokens)

- [在 Azure AD 中登錄應用程式的基本概念](#basics-of-registering-an-application-in-azure-ad)

- [應用程式類型和案例](#application-types-and-scenarios)

  - [Web 瀏覽器到 Web 應用程式](#web-browser-to-web-application)

  - [單一頁面應用程式 (SPA)](#single-page-application-spa)

  - [原生應用程式到 Web API](#native-application-to-web-api)

  - [Web 應用程式到 Web API](#web-application-to-web-api)

  - [精靈或伺服器應用程式到 Web API](#daemon-or-server-application-to-web-api)



## Azure AD 中的驗證基本概念

如果您不熟悉 Azure AD 中的驗證基本概念，請閱讀本節。否則，您可以往下跳到[應用程式類型和案例](#application-types-and-scenarios)。

我們來看一下需要身分識別的最基本案例：Web 瀏覽器使用者需要向 Web 應用程式驗證。此案例在 [Web 瀏覽器到 Web 應用程式](#web-browser-to-web-application)一節中有更詳細的描述，但提供很好的起點來說明 Azure AD 的功能，並從概念上說明案例的運作方式。請針對此案例細想下圖：

![登入 Web 應用程式概觀](./media/active-directory-authentication-scenarios/basics_of_auth_in_aad.png)

根據上圖，您對其中各種元件必須有所了解：

- Azure AD 是身分識別提供者，負責驗證組織目錄中的使用者和應用程式的身分識別，最後在成功驗證那些使用者和應用程式時發行安全性權杖。


- 想要將驗證外包給 Azure AD 的應用程式必須在 Azure AD 中註冊，Azure AD 會在目錄中註冊並唯一識別此應用程式。


- 開發人員可以使用開放原始碼 Azure AD 驗證程式庫，為您處理通訊協定的細節，以輕鬆完成驗證。如需詳細資訊，請參閱 [Azure Active Directory 驗證程式庫](active-directory-authentication-libraries.md)。


• 使用者通過驗證之後，應用程式必須驗證使用者的安全性權杖，以確定相關各方的驗證成功。開發人員可以使用提供的驗證程式庫來驗證來自 Azure AD 的任何權杖，包括 JSON Web Token (JWT) 或 SAML 2.0。如果您想要手動執行驗證，請參閱 [JWT 權杖處理常式](https://msdn.microsoft.com/library/dn205065.aspx)文件。


> [AZURE.IMPORTANT] Azure AD 使用公開金鑰密碼編譯來簽署權杖並驗證它們有效。如需有關應用程式中必要的邏輯以確保永遠以最新金鑰更新的相關資訊，請參閱 [Azure AD 中簽署金鑰變換的相關重要資訊](active-directory-signing-key-rollover.md)。


• 驗證程序的要求和回應流程由使用的驗證通訊協定決定，例如 OAuth 2.0、OpenID Connect，WS-同盟或 SAML 2.0。[Azure Active Directory 驗證通訊協定](active-directory-authentication-protocols.md)主題和下列各節中更詳細地討論這些通訊協定。

> [AZURE.NOTE] Azure AD 支援 OAuth 2.0 和 OpenID Connect 標準，這些標準廣泛運用持有者權杖，包括以 JWT 表示的持有者權杖。持有人權杖是輕巧型的安全性權杖，授權「持有者」存取受保護的資源。從這個意義上說，「持有者」是可出示權杖的任何一方。雖然某一方必須先向 Azure AD 驗證以收到持有人權杖，但如果傳輸和儲存時未採取必要的步驟來保護權杖，它可能會被非預期的一方攔截和使用。雖然某些安全性權杖都有內建的機制，可防止未經授權的人士使用權杖，但持有者權杖沒有這項機制，而必須以安全通道來傳輸，例如傳輸層安全性 (HTTPS)。如果持有人權杖以純文字傳輸，惡意人士可能使用攔截式攻擊來取得權杖，然後未經授權存取受保護的資源。儲存或快取持有者權杖供以後使用時，也適用相同的安全性原則。務必確定您的應用程式以安全的方式傳輸和儲存持有者權杖。關於持有者權杖的其他安全性考量，請參閱 [RFC 6750 第 5 節](http://tools.ietf.org/html/rfc6750)。


既然您已有基本概念的概觀，請閱讀下列各節，了解 Azure AD 中的佈建運作方式，以及 Azure AD 支援的常見案例。


## Azure AD 安全性權杖中的宣告

Azure AD 所簽發的安全性權杖包含宣告，或已驗證之主體的相關資訊的判斷提示。這些宣告可以供應用程式用於執行各種工作。比方說，它們可以用來驗證權杖、識別主體的目錄租用戶、顯示使用者資訊、判斷主體的授權等。任何給定的安全性權杖中的宣告取決於權杖類型、用來驗證使用者的認證類型，以及應用程式組態。下表提供由 Azure AD 發出的每一種宣告的簡短描述。如需詳細資訊，請參閱[支援的權杖和宣告類型](active-directory-token-and-claims.md)。


| 宣告 | 說明 |
|-------|-------------|
| 應用程式識別碼 | 識別使用權杖的應用程式。
| 觀眾 | 識別權杖針對的收件者資源。 |
| 應用程式驗證內容類別參考 | 指出如何驗證用戶端 (公用用戶端與機密用戶端的比較)。 |
| 驗證時刻 | 記錄驗證發生的日期和時間。 |
| 驗證方法 | 指出如何驗證權杖的主體 (密碼、憑證等)。 |
| 名字 | 提供 Azure AD 中設定的使用者名字。 |
| 群組 | 包含使用者所屬的 Azure AD 群組的物件識別碼。 |
| 身分識別提供者 | 記錄驗證權杖主體的身分識別提供者。 |
| 發出時間 | 記錄核發權杖的時間，通常用於權杖有效期限。 |
| 簽發者 | 識別發出權杖的 STS，以及 Azure AD 租用戶。 |
| 姓氏 | 提供 Azure AD 中設定的使用者姓氏。 |
| 名稱 | 提供人類看得懂的值，用以識別權杖的主體。 |
| 物件識別碼 | 包含主體在 Azure AD 中不可變的唯一識別碼。 |
| 角色 | 包含已授與使用者的 Azure AD 應用程式角色的易記名稱。 |
| Scope | 指出授與用戶端應用程式的權限。 |
| 主旨 | 指出權杖判斷提示相關資訊的主體。 |
| 租用戶識別碼 | 包含發出權杖的目錄租用戶的不變唯一識別碼。 |
| 權杖存留期 | 定義權杖有效的時間間隔。 |
| 使用者主體名稱 | 包含主體的使用者主體名稱。 |
| 版本 | 包含權杖的版本號碼。 |


## 在 Azure AD 中登錄應用程式的基本概念

將驗證外包給 Azure AD 的任何應用程式必須在目錄中註冊。此步驟涉及向 Azure AD 說明您的應用程式，包括它所在的 URL、驗證之後用來傳送回覆的 URL、用來識別應用程式的 URI 等。需要這項資訊有幾個主要理由：

- Azure AD 處理登入或交換權杖時，需要座標才能與應用程式通訊。其中包括：

  - 應用程式識別碼 URI：應用程式的識別碼。這個值會在驗證期間傳送至 Azure AD，以指出呼叫端想要哪一個應用程式的權杖。此外，這個值包含在權杖中，應用程式由此可知它就是預定的目標。


  - 回覆 URL 和重新導向 URI：在 Web API 或 Web 應用程式中，回覆 URL 是 Azure AD 傳送驗證回應的目標位置，包括驗證成功時的權杖。在原生應用程式中，重新導向 URI 是 Azure AD 在 OAuth 2.0 要求中將使用者代理程式重新導向的唯一識別碼。


  - 用戶端識別碼：註冊應用程式時由 Azure AD 產生的應用程式識別碼。當要求授權碼或權杖時，用戶端識別碼和金鑰會在驗證期間傳送至 Azure AD。


  - 金鑰：向 Azure AD 驗證來呼叫 Web API 時，隨用戶端識別碼一起傳送的金鑰。


- Azure AD 需要確定應用程式具有存取目錄資料、您組織中其他應用程式等等的必要權限。

當您了解可以開發及與 Azure AD 整合的應用程式有兩種類別時，佈建就變得更清楚：

- 單一租用戶應用程式：單一租用戶應用程式適合在一個組織中使用。這些通常是由企業開發人員撰寫的企業營運 (LoB) 應用程式。單一租用戶應用程式只需要由一個目錄中的使用者存取，因此，它只需要佈建在一個目錄中。這些應用程式通常是由組織中的開發人員註冊。


- 多租用戶應用程式：多租用戶應用程式適合在許多組織中使用，而不只一個組織。這些通常是由獨立軟體廠商 (ISV) 撰寫的軟體即服務 (SaaS) 應用程式。多租用戶應用程式需要佈建在將會用到它們的每個目錄中，而這需要使用者或系統管理員同意才能註冊。當應用程式目錄中已在目錄中註冊並獲權存取 Graph API 或其他 Web API 時，此同意程序會啟動。當不同組織的使用者或系統管理員註冊來使用應用程式時，他們會看到一個對話方塊顯示應用程式需要的權限。使用者或系統管理員可以同意應用程式，讓應用程式存取所述的資料，最後將應用程式註冊在他們的目錄中。如需詳細資訊，請參閱 [同意架構的概觀](active-directory-integrating-applications.md#overview-of-the-consent-framework)。

開發多租用戶應用程式，而非單一租用戶應用程式時，有一些其他考量需要注意。例如，如果要讓您的應用程式供多個目錄中的使用者使用，您需要有機制來判斷他們所在的租用戶。單一租用戶應用程式只需要在它自己的目錄中查看使用者，但多租用戶應用程式需要從 Azure AD 的所有目錄中識別特定的使用者。為了完成這項工作，Azure AD 提供一個共同驗證端點，供任何多租用戶應用程式引導登入要求，而非提供租用戶特定的端點。對於 Azure AD 中的所有目錄，此端點為 https://login.microsoftonline.com/common而租用戶特定的端點可能是 https://login.microsoftonline.com/contoso.onmicrosoft.com開發您的應用程式時尤其必須考量共同端點，因為在登入、登出和權杖驗證期間，您需要必要的邏輯來處理多個租用戶。

如果您目前正在開發單一租用戶應用程式，但想要提供給許多組織使用，您可以在 Azure AD 中輕鬆地變更應用程式及其組態，將它變成具備多租用戶功能。此外，不論您是在單一租用戶或多租用戶應用程式中提供驗證，Azure AD 對所有目錄中的所有權杖都使用相同的簽署金鑰。

本文件列出的每個案例都有一個小節來說明其佈建需求。如需關於在 Azure AD 中佈建應用程式的深入資訊，以及了解單一和多租用戶應用程式之間的差異，請參閱[整合應用程式與 Azure Active Directory](active-directory-integrating-applications.md)，以取得詳細資訊。繼續閱讀來了解 Azure AD 中常見的應用程式案例。

## 應用程式類型和案例

本文件中所述的每個案例也可以使用各種語言和平台進行開發。它們皆有完整的程式碼範例支援，這些程式碼範例是在我們的[程式碼範例指南](active-directory-code-samples.md)中，或者直接從相對應的 [Github 範例存放庫](https://github.com/Azure-Samples?utf8=%E2%9C%93&query=active-directory)提供使用。此外，如果您的應用程式需要端對端案例的特定片段或區段，在大部分情況下可以獨立加入這項功能。例如，如果您有一個呼叫 Web API 的原生應用程式，您可以輕鬆加入也會呼叫該 Web API 的 Web 應用程式。下圖說明這些案例和應用程式類型，以及如何加入不同的元件：

![應用程式類型和案例](./media/active-directory-authentication-scenarios/application_types_and_scenarios.png)

以下是 Azure AD 支援的五個主要應用程式案例：

- [Web 瀏覽器到 Web 應用程式](#web-browser-to-web-application)：使用者需要登入 Azure AD 所保護的 Web 應用程式。

- [單一頁面應用程式 (SPA)](#single-page-application-spa)：使用者需要登入 Azure AD 所保護的單一頁面應用程式。

- [原生應用程式到 Web API](#native-application-to-web-api)：在電話、平板電腦或電腦執行的原生應用程式需要驗證使用者，以便從 Azure AD 所保護的 Web API 取得資源。

- [Web 應用程式到 Web API](#web-application-to-web-api)：Web 應用程式需要從 Azure AD 所保護的 Web API 取得資源。

- [精靈或伺服器應用程式到 Web API](#daemon-or-server-application-to-web-api)：無 Web 使用者介面的精靈應用程式或伺服器應用程式，需要從 Azure AD 所保護的 Web API 取得資源。

### Web 瀏覽器到 Web 應用程式

本節描述一個應用程式向 Web 應用程式驗證 Web 瀏覽器中的使用者。在此案例中，Web 應用程式會引導使用者的瀏覽器，將他們登入 Azure AD。Azure AD 會透過使用者的瀏覽器傳回登入回應，其中包含關於安全性權杖中的使用者宣告。此案例支援使用 WS-同盟、SAML 2.0 和 OpenID Connect 通訊協定來登入。


#### 圖表
![瀏覽器到 Web 應用程式的驗證流程](./media/active-directory-authentication-scenarios/web_browser_to_web_api.png)


#### 通訊協定流程的描述


1. 當使用者造訪應用程式且需要登入時，他們透過登入要求而重新導向 Azure AD 中的驗證端點。


2. 使用者在登入頁面上登入。


3. 如果驗證成功，Azure AD 會建立驗證權杖，並將登入回應傳回至 Azure 管理入口網站中設定的應用程式回覆 URL。對於實際執行應用程式，此回覆 URL 應該為 HTTPS。傳回的權杖包含應用程式驗證權杖所需的使用者與 Azure AD 宣告。


4. 應用程式會使用 Azure AD 的同盟中繼資料文件可用的公開簽署金鑰和簽發者資訊來驗證權杖。應用程式驗證權杖之後，Azure AD 會對使用者啟動新的工作階段。此工作階段可讓使用者存取應用程式，直到過期為止。


#### 程式碼範例


請參閱「Web 瀏覽器到 Web 應用程式」案例的程式碼範例。請經常回來查看，我們會隨時加入新的範例。[Web 瀏覽器到 Web 應用程式](active-directory-code-samples.md#web-browser-to-web-application)。


#### 註冊


- 單一租用戶：如果您只是要為您的組織建置應用程式，則必須使用 Azure 管理入口網站，將它註冊在公司的目錄中。


- 多租用戶：如果您要建置的應用程式可供組織外的使用者使用，它必須註冊在公司的目錄中，但也必須在每個將會使用該應用程式的組織的目錄中註冊。若要讓您的應用程式出現在他們的目錄中，您可以為客戶加上註冊程序，讓他們同意應用程式。當他們註冊您的應用程式時，他們會看到對話方塊顯示應用程式所需的權限，以及是否同意的選項。根據所需的權限，其他組織的系統管理員可能必須同意。當使用者或系統管理員同意時，應用程式就會註冊在他們的目錄中。如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](active-directory-integrating-applications.md)。


#### 權杖到期

當 Azure AD 所簽發的權杖存留期到期時，使用者工作階段就過期。如有需要，您的應用程式可以縮短這段時間，例如因為一段時間沒有活動而將使用者登出。當工作階段到期時，會提示使用者重新登入。





### 單一頁面應用程式 (SPA)

本節描述單一頁面應用程式的驗證，此應用程式會使用 Azure AD 和 OAuth 2.0 隱含授權授與來保護其 Web API 後端。單一頁面應用程式通常建構為在瀏覽器中執行的 JavaScript 展示層 (前端)，以及在伺服器上執行並實作應用程式商務邏輯的 Web API 後端。若要深入了解隱含授權授與，並協助您決定其是否適合您的應用程式案例，請參閱 [了解 Azure Active Directory 中的 OAuth2 隱含授與流程](active-directory-dev-understanding-oauth2-implicit-grant.md)。

在此案例中，當使用者登入時，JavaScript 前端會使用 [Active Directory Authentication Library for JavaScript (ADAL.JS)](https://github.com/AzureAD/azure-activedirectory-library-for-js/tree/dev) 和隱含授權授與，從 Azure AD 取得的識別碼權杖 (id\_token)。權杖會留在快取中，而用戶端呼叫 Web API 後端時會將權杖附加至要求做為持有人權杖，並使用 OWIN 中介軟體來保護。
#### 圖表

![單一頁面應用程式圖表](./media/active-directory-authentication-scenarios/single_page_app.png)

#### 通訊協定流程的描述

1. 使用者導覽至 Web 應用程式。


2. 應用程式將 JavaScript 前端 (展示層) 傳回至瀏覽器。


3. 使用者起始登入，例如按一下登入連結。瀏覽器傳送 GET 給 Azure AD 授權端點來要求識別碼權杖。此要求在查詢參數中包含用戶端識別碼和回覆 URL。


4. Azure AD 根據 Azure 管理入口網站中設定的註冊回覆 URL，驗證回覆 URL。


5. 使用者在登入頁面上登入。


6. 如果驗證成功，Azure AD 就會建立識別碼權杖，並當做 URL 片段 (#) 傳回至應用程式的回覆 URL。對於實際執行應用程式，此回覆 URL 應該為 HTTPS。傳回的權杖包含應用程式驗證權杖所需的使用者與 Azure AD 宣告。


7. 瀏覽器中執行的 JavaScript 用戶端程式碼從回應中擷取權杖，用以保護對應用程式 Web API 後端的呼叫。


8. 瀏覽器使用授權標頭中的存取權杖，呼叫應用程式的 Web API 後端。

#### 程式碼範例


請參閱「單一頁面應用程式 (SPA)」案例的程式碼範例。請務必經常回來查看，我們會隨時加入新的範例。[單一頁面應用程式 (SPA)](active-directory-code-samples.md#single-page-application-spa)。


#### 註冊


- 單一租用戶：如果您只是要為您的組織建置應用程式，則必須使用 Azure 管理入口網站，將它註冊在公司的目錄中。


- 多租用戶：如果您要建置的應用程式可供組織外的使用者使用，它必須註冊在公司的目錄中，但也必須在每個將會使用該應用程式的組織的目錄中註冊。若要讓您的應用程式出現在他們的目錄中，您可以為客戶加上註冊程序，讓他們同意應用程式。當他們註冊您的應用程式時，他們會看到對話方塊顯示應用程式所需的權限，以及是否同意的選項。根據所需的權限，其他組織的系統管理員可能必須同意。當使用者或系統管理員同意時，應用程式就會註冊在他們的目錄中。如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](active-directory-integrating-applications.md)。

註冊應用程式之後，它必須設定為使用 OAuth 2.0 隱含授權通訊協定。根據預設，應用程式都停用此通訊協定。若要對您的應用程式啟用 OAuth2 隱含授權通訊協定，請從 Azure 管理入口網站下載其應用程式資訊清單，將 "oauth2AllowImplicitFlow" 值設為 true，再將資訊清單上傳回到入口網站。如需詳細指示，請參閱[啟用單一頁面應用程式的 OAuth 2.0 隱含授權](active-directory-integrating-applications.md)。


#### 權杖到期

當您使用 ADAL.js 來管理向 Azure AD 驗證時，有數個功能可協助更新過期的權杖，並針對應用程式可能呼叫的其他 Web API 資源取得權杖。當使用者成功向 Azure AD 驗證時，瀏覽器與 Azure AD 之間會為使用者建立一個以 Cookie 保護的工作階段。請務必注意，工作階段存在於使用者與 Azure AD 之間，而不是在使用者與伺服器上執行的 Web 應用程式之間。當權杖過期時，ADAL.js 會使用此工作階段來自動取得另一個權杖。在作法上是使用隱藏的 iFrame 來傳送和接收要求，而且是透過 OAuth 隱含授權通訊協定。ADAL.js 也可以使用這個相同的機制，針對應用程式會呼叫的其他 Web API 資源，自動從 Azure AD 取得存取權杖，但這些資源必須支援跨原始資源共用 (CORS)、已註冊在使用者的目錄中，而且使用者在登入期間已做出任何必要的同意。


### 原生應用程式到 Web API


本節描述代表使用者呼叫 Web API 的原生應用程式。此案例是根據 OAuth 2.0 授權碼授與類型和公用用戶端，如 [OAuth 2.0 規格](http://tools.ietf.org/html/rfc6749)第 4.1 節所述。此原生應用程式使用 OAuth 2.0 通訊協定，為使用者取得存取權杖。接著，此存取權杖隨著要求傳送至 Web API，Web API 再授權使用者並傳回所需的資源。

#### 圖表

![原生應用程式到 Web API 圖表](./media/active-directory-authentication-scenarios/native_app_to_web_api.png)

#### 原生應用程式到 API 的驗證流程

#### 通訊協定流程的描述


如果您使用 AD 驗證程式庫，則會為您處理如下所述的大部分通訊協定細節，例如瀏覽器快顯視窗、權杖快取和重新整理權杖的處理。

1. 原生應用程式使用瀏覽器快顯視窗，對 Azure AD 中的授權端點提出要求。此要求包含用戶端識別碼和原生應用程式的重新導向 URI (如管理入口網站所示)，以及 Web API 的應用程式識別碼 URI。如果使用者尚未登入，系統會提示他們再次登入。


2. Azure AD 驗證使用者。如果是多租用戶應用程式，且需要同意才能使用應用程式，則會要求使用者同意 (如果他們還沒有同意)。表示同意且成功驗證之後，Azure AD 會發出授權碼回應傳回至用戶端應用程式的重新導向 URI。


3. 當 Azure AD 發出授權碼回應傳回至重新導向 URI 時，用戶端應用程式會停止瀏覽器互動，並從回應中擷取授權碼。用戶端應用程式會使用此授權碼，傳送要求至 Azure AD 的權杖端點，此要求包含授權碼、用戶端應用程式的詳細資料 (用戶端識別碼和重新導向 URI)，以及所需的資源 (Web API 的應用程式識別碼 URI)。


4. Azure AD 驗證授權碼及用戶端應用程式和 Web API 的相關資訊。成功驗證後，Azure AD 會傳回兩個權杖：JWT 存取權杖和 JWT 重新整理權杖。此外，Azure AD 會傳回使用者的基本資訊，例如其顯示名稱和租用戶識別碼。


5. 用戶端應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。


6. 當存取權杖到期時，用戶端應用程式會收到錯誤，指出使用者必須再次驗證。如果應用程式具有有效的重新整理權杖，它可用來取得新的存取權杖，不會提示使用者重新登入。如果重新整理權杖到期，應用程式必須再一次以互動方式驗證使用者。


> [AZURE.NOTE] Azure AD 所簽發的重新整理權杖可用來存取多個資源。例如，如果您的用戶端應用程式有權限呼叫兩個 Web API，重新整理權杖也可用來取得其他 Web API 的存取權杖。


#### 程式碼範例


請參閱「原生應用程式到 Web API」案例的程式碼範例。請經常回來查看，我們會隨時加入新的範例。[原生應用程式到 Web API](active-directory-code-samples.md#native-application-to-web-api)。


#### 註冊


- 單一租用戶：原生應用程式和 Web API 必須註冊在 Azure AD 的相同目錄中。Web API 可以設定為公開一組權限，用以限制原生應用程式對其資源的存取權。然後，用戶端應用程式從 Azure 管理入口網站的 [其他應用程式的權限] 下拉式功能表中，選取所需的權限。


- 多租用戶：首先，原生應用程式僅註冊在開發人員或發行者的目錄中。第二，設定原生應用程式來指出它運作所需的權限。當目的地目錄中的使用者或系統管理員同意應用程式時 (使得應用程式可供組織使用)，這份必要權限清單會顯示在對話方塊中。有些應用程式只需要使用者層級權限，亦即組織中的任何使用者都可同意應用程式。其他應用程式需要系統管理員層級權限，亦即組織中的使用者無法同意應用程式。只有目錄管理員才能對需要此權限層級的應用程式表示同意。當使用者或系統管理員同意時，只有 Web API 會註冊在他們的目錄中。如需詳細資訊，請參閱[整合應用程式與 Azure Active Directory](active-directory-integrating-applications.md)。


#### 權杖到期


當原生應用程式使用其授權碼來取得 JWT 存取權杖時，它也會收到 JWT 重新整理權杖。當存取權杖到期時，重新整理權杖可用來重新驗證使用者，而不需要他們再次登入。然後，此重新整理權杖用來驗證使用者，結果會產生新的存取權杖和重新整理權杖。





### Web 應用程式到 Web API


本節描述需要從 Web API 取得資源的 Web 應用程式。在此案例中，有兩個識別類型可供 Web 應用程式用來驗證和呼叫 Web API：應用程式識別或委派的使用者識別。

*應用程式識別*：此案例使用 OAuth 2.0 用戶端認證授與來驗證應用程式和存取 Web API。使用應用程式識別時，Web API 只能偵測到 Web 應用程式正在呼叫它，因為 Web API 沒有收到關於使用者的任何資訊。如果應用程式收到使用者的相關資訊，此資訊會透過應用程式通訊協定傳送，但未經過 Azure AD 簽署。Web API 信任 Web 應用程式已驗證使用者。基於這個理由，這種模式稱為受信任子系統。

*委派的使用者識別*：有兩種方式可實現此案例：OpenID Connect 和搭配機密用戶端的 OAuth 2.0 授權碼授與。Web 應用程式會為使用者取得存取權杖，向 Web API 證明使用者已順利通過 Web 應用程式的驗證，而且 Web 應用程式能夠取得委派的使用者識別來呼叫 Web API。此存取權杖隨著要求傳送至 Web API，Web API 再授權使用者並傳回所需的資源。

#### 圖表

![Web 應用程式到 Web API 圖表](./media/active-directory-authentication-scenarios/web_app_to_web_api.png)



#### 通訊協定流程的描述

下列流程中討論應用程式識別和委派的使用者識別類型。它們之間的主要差異是委派的使用者識別必須先取得授權碼，使用者才能登入並存取 Web API。

##### 採用 OAuth 2.0 用戶端認證授與的應用程式識別

1. 使用者在 Web 應用程式中登入 Azure AD (請參閱上方的 [Web 瀏覽器到 Web 應用程式](#web-browser-to-web-application))。


2. Web 應用程式需要取得存取權杖，才能向 Web API 驗證和擷取所需的資源。它向 Azure AD 的權杖端點提出要求，並提供認證、用戶端識別碼和 Web API 的應用程式識別碼 URI。


3. Azure AD 驗證應用程式，並傳回用來呼叫 Web API 的 JWT 存取權杖。


4. Web 應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。

##### 採用 OpenID Connect 的委派的使用者識別

1. 使用者使用 Azure AD 登入 Web 應用程式 (請參閱上方的 [Web 瀏覽器到 Web 應用程式](#web-browser-to-web-application)一節)。如果 Web 應用程式的使用者尚未同意允許 Web 應用程式代表他來呼叫 Web API，使用者必須同意。應用程式會顯示它需要的權限，如果其中任何一項是系統管理員層級權限，則目錄中的一般使用者將無法同意。此同意程序僅適用於多租用戶應用程式，而非單一租用戶應用程式，因為應用程式已具有必要的權限。使用者登入後，Web 應用程式會收到識別碼權杖和使用者相關資訊，以及授權碼。


2. Web 應用程式會使用 Azure AD 簽發的授權碼，傳送要求至 Azure AD 的權杖端點，此要求包含授權碼、用戶端應用程式的詳細資料 (用戶端識別碼和重新導向 URI)，以及所需的資源 (Web API 的應用程式識別碼 URI)。


3. Azure AD 驗證授權碼及 Web 應用程式和 Web API 的相關資訊。成功驗證後，Azure AD 會傳回兩個權杖：JWT 存取權杖和 JWT 重新整理權杖。


4. Web 應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。

##### 採用 OAuth 2.0 授權碼授與的委派的使用者識別

1. 使用者已經登入 Web 應用程式，其驗證機制與 Azure AD 無關。


2. Web 應用程式需要授權碼才能取得存取權杖，因此它透過瀏覽器向 Azure AD 授權端點發出要求，並提供用戶端識別碼和成功驗證之後的 Web 應用程式重新導向 URI。使用者登入 Azure AD。


3. 如果 Web 應用程式的使用者尚未同意允許 Web 應用程式代表他來呼叫 Web API，使用者必須同意。應用程式會顯示它需要的權限，如果其中任何一項是系統管理員層級權限，則目錄中的一般使用者將無法同意。此同意程序僅適用於多租用戶應用程式，而非單一租用戶應用程式，因為應用程式已具有必要的權限。


4. 使用者同意之後，Web 應用程式會收到它取得存取權杖所需的授權碼。


5. Web 應用程式會使用 Azure AD 簽發的授權碼，傳送要求至 Azure AD 的權杖端點，此要求包含授權碼、用戶端應用程式的詳細資料 (用戶端識別碼和重新導向 URI)，以及所需的資源 (Web API 的應用程式識別碼 URI)。


6. Azure AD 驗證授權碼及 Web 應用程式和 Web API 的相關資訊。成功驗證後，Azure AD 會傳回兩個權杖：JWT 存取權杖和 JWT 重新整理權杖。


7. Web 應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。

#### 程式碼範例

請參閱「Web 應用程式到 Web API」案例的程式碼範例。請經常回來查看，我們會隨時加入新的範例。Web [應用程式到 Web API](active-directory-code-samples.md#web-application-to-web-api)。


#### 註冊

- 單一租用戶：無論是應用程式識別或委派的使用者識別的情況，Web 應用程式和 Web API 都必須註冊在 Azure AD 的相同目錄中。Web API 可以設定為公開一組權限，用以限制 Web 應用程式對其資源的存取權。如果使用委派的識別類型，Web 應用程式需要從 Azure 管理入口網站的 [其他應用程式的權限] 下拉式功能表中，選取所需的權限。如果使用應用程式識別類型，則不需要此步驟。


- 多租用戶：首先，設定 Web 應用程式來指出它運作所需的權限。當目的地目錄中的使用者或系統管理員同意應用程式時 (使得應用程式可供組織使用)，這份必要權限清單會顯示在對話方塊中。有些應用程式只需要使用者層級權限，亦即組織中的任何使用者都可同意應用程式。其他應用程式需要系統管理員層級權限，亦即組織中的使用者無法同意應用程式。只有目錄管理員才能對需要此權限層級的應用程式表示同意。當使用者或系統管理員同意時，Web 應用程式和 Web API 都會註冊在他們的目錄中。

#### 權杖到期

當 Web 應用程式使用其授權碼來取得 JWT 存取權杖時，它也會收到 JWT 重新整理權杖。當存取權杖到期時，重新整理權杖可用來重新驗證使用者，而不需要他們再次登入。然後，此重新整理權杖用來驗證使用者，結果會產生新的存取權杖和重新整理權杖。


### 精靈或伺服器應用程式到 Web API


本節描述需要從 Web API 取得資源的精靈或伺服器應用程式。有兩個子案例適用於本節：需要呼叫 Web API 且基於 OAuth 2.0 用戶端認證授與類型的精靈；需要呼叫 Web API 且基於 OAuth 2.0 On-Behalf-Of 草稿規格的伺服器應用程式 (例如 Web API)。

在精靈應用程式需要呼叫 Web API 的案例中，務必了解一些事情。首先，使用者無法與精靈應用程式互動，應用程式必須有自己的身分識別。像是批次工作，或在背景執行的作業系統服務，就是精靈應用程式的例子。這類應用程式會使用其應用程式識別，並向 Azure AD 出示其用戶端識別碼、認證 (密碼或憑證) 和應用程式識別碼 URI，以要求存取權杖。成功驗證之後，精靈會從 Azure AD 收到存取權杖，然後用來呼叫 Web API。

在伺服器應用程式需要呼叫 Web API 的案例中，最好使用範例。假設使用者已在原生應用程式上驗證，而此原生應用程式需要呼叫 Web API。Azure AD 會發出 JWT 存取權杖來呼叫 Web API。如果 Web API 需要呼叫另一個下游 Web API，它可以使用代表流程來委派使用者的身分識別，並向第二層 Web API 驗證。

#### 圖表

![精靈或伺服器應用程式到 Web API 圖表](./media/active-directory-authentication-scenarios/daemon_server_app_to_web_api.png)

#### 通訊協定流程的描述

##### 採用 OAuth 2.0 用戶端認證授與的應用程式識別

1. 首先，伺服器應用程式本身必須向 Azure AD 驗證，沒有任何人為互動，例如互動式登入對話方塊。它向 Azure AD 的權杖端點提出要求，並提供認證、用戶端識別碼和應用程式識別碼 URI。


2. Azure AD 驗證應用程式，並傳回用來呼叫 Web API 的 JWT 存取權杖。


3. Web 應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。


##### 採用 OAuth 2.0 On-Behalf-Of 草稿規格的委派的使用者識別

下面討論的流程假設使用者已在另一個應用程式上驗證 (例如，原生應用程式)，且其使用者識別已用來取得第一層 Web API 的存取權杖。

1. 原生應用程式將存取權杖傳送到第一層 Web API。


2. 第一層 Web API 傳送要求至 Azure AD 的權杖端點，並提供其用戶端識別碼和認證，以及使用者的存取權杖。此外，傳送的要求包含 on\_behalf\_of 參數，指出 Web API 正在代表原始使用者要求新權杖來呼叫下游 Web API。


3. Azure AD 確認第一層 Web API 有權存取第二層 Web API，並驗證要求，然後傳回 JWT 存取權杖和 JWT 重新整理權杖給第一層 Web API。


4. 接著，第一層 Web API 透過 HTTPS，在要求的 Authorization 標頭中附加權杖字串，以呼叫第二層 Web API。只要存取權杖和重新整理權杖有效，第一層 Web API 就可以繼續呼叫第二層 Web API。

#### 程式碼範例

請參閱「精靈或伺服器應用程式到 Web API」案例的程式碼範例。請經常回來查看，我們會隨時加入新的範例。[伺服器或精靈應用程式到 Web API](active-directory-code-samples.md#server-or-daemon-application-to-web-api)

#### 註冊

- 單一租用戶：無論是應用程式識別或委派的使用者識別的情況，精靈或伺服器應用程式都必須註冊在 Azure AD 的相同目錄中。Web API 可以設定為公開一組權限，用以限制精靈或伺服器應用程式對其資源的存取權。如果使用委派的識別類型，伺服器應用程式需要從 Azure 管理入口網站的 [其他應用程式的權限] 下拉式功能表中，選取所需的權限。如果使用應用程式識別類型，則不需要此步驟。


- 多租用戶：首先，設定精靈或伺服器應用程式來指出它運作所需的權限。當目的地目錄中的使用者或系統管理員同意應用程式時 (使得應用程式可供組織使用)，這份必要權限清單會顯示在對話方塊中。有些應用程式只需要使用者層級權限，亦即組織中的任何使用者都可同意應用程式。其他應用程式需要系統管理員層級權限，亦即組織中的使用者無法同意應用程式。只有目錄管理員才能對需要此權限層級的應用程式表示同意。當使用者或系統管理員同意時，這兩個 Web API 都會註冊在他們的目錄中。

#### 權杖到期

當第一個應用程式使用其授權碼來取得 JWT 存取權杖時，它也會收到 JWT 重新整理權杖。當存取權杖到期時，重新整理權杖可用來重新驗證使用者，而不會提示輸入認證。然後，此重新整理權杖用來驗證使用者，結果會產生新的存取權杖和重新整理權杖。

## 另請參閱

[Azure Active Directory 開發人員指南](active-directory-developers-guide.md)

[Azure Active Directory 程式碼範例](active-directory-code-samples.md)

[Azure AD 中簽署金鑰變換的相關重要資訊](active-directory-signing-key-rollover.md)

[Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/library/azure/dn645545.aspx)

<!---HONumber=AcomDC_0921_2016-->