
<properties
	pageTitle="使用 Azure AD Connect Health 搭配 AD FS | Microsoft Azure"
	description="這是如何監視內部部署 AD FS 基礎結構的 Azure AD Connect Health 頁面。"
	services="active-directory"
	documentationCenter=""
	authors="karavar"
	manager="femila"
	editor="karavar"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="07/14/2016"
	ms.author="vakarand"/>

# 使用 Azure AD Connect Health 搭配 AD FS
下列文件適用於使用 Azure AD Connect Health 來監視 AD FS 基礎結構。如需使用 Azure AD Connect Health 來監視 Azure AD Connect (同步處理) 的詳細資訊，請參閱 [使用適用於同步處理的 Azure AD Connect Health](active-directory-aadconnect-health-sync.md)。此外，如需使用 Azure AD Connect Health 來監視 Active Directory 網域服務的詳細資訊，請參閱 [在 AD DS 使用 Azure AD Connect Health](active-directory-aadconnect-health-adds.md)。

## AD FS 的警示
[Azure AD Connect Health 警示] 區段將為您提供作用中警示的清單。每個警示都包含相關資訊、解決步驟，以及相關文件的連結。選取作用中或已解決的警示，您將會看到一個包含額外資訊的新刀鋒視窗，以及解決警示可以採取的步驟，和其他文件的連結。您也可以檢視過去已解決的警示的歷史資料。

選取警示，將會為您提供其他資訊，以及解決警示可以採取的步驟，和其他文件的連結。

![Azure AD Connect Health 入口網站](./media/active-directory-aadconnect-health/alert2.png)



## AD FS 的使用情況分析
Azure AD Connect Health 使用情況分析會分析您的同盟伺服器的驗證流量。選取使用情況分析方塊將會開啟使用情況分析刀鋒視窗，以顯示度量和群組。

>[AZURE.NOTE] 若要搭配 AD FS 使用使用情況分析，您必須確定已啟用 AD FS 稽核。如需詳細資訊，請參閱[啟用 AD FS 的稽核](active-directory-aadconnect-health-agent-install.md#enable-auditing-for-ad-fs)。

![Azure AD Connect Health 入口網站](./media/active-directory-aadconnect-health/report1.png)

若要選取其他度量，請指定時間範圍；若要變更群組，只要在使用情況分析圖表上按一下滑鼠右鍵，並選取 [編輯圖表]。接著，您可以指定時間範圍、變更或選取度量，以及變更群組。您可以根據不同的「度量」檢視驗證流量的分佈，並使用相關的「分組依據」參數，為每個度量分組，如下所述。

| 度量 | 分組依據 | 分組是什麼意思，為什麼分組很有用？ |
| ------ | -------- | -------------------------------------------- |
| 要求總數：同盟服務所處理的要求的總數 | 全部 | 這將會顯示未分組的要求總數的計數。 |
| | 應用程式 | 此選項將會根據目標信賴憑證者，為要求總數分組。這個分組對於了解哪個應用程式收到多少百分比的總流量非常有幫助。 |
| | 伺服器 | 此選項將會根據處理要求的伺服器，為要求總數分組。這個分組對於了解總流量的負載分佈非常有幫助。 |
| | 加入工作場所 | 此選項將會根據要求是否來自已加入工作場所的裝置 (已知)，為要求總數分組。這個分組對於了解是否使用識別基礎結構未知的裝置存取您的資源非常有幫助。 |
| | 驗證方法 | 此選項將會根據用於驗證的驗證方法，為要求總數分組。這個分組對於了解用於驗證的常見驗證方法非常有幫助。以下是可能的驗證方法<ol> <li>Windows 整合式驗證 (Windows)</li> <li>表單型驗證 (表單)</li> <li>SSO (單一登入)</li> <li>X 509 憑證驗證 (憑證)</li> <br>請注意，如果同盟伺服器收到包含 SSO Cookie 的要求，則該要求會列入 SSO (單一登入)。在這種情況下，如果 Cookie 有效，則不會要求使用者提供認證，並不間斷地存取應用程式。如果您有多個同盟伺服器所保護的信賴憑證者，則此情況非常常見。 |
| | 網路位置 | 此選項將會根據使用者的網路位置，為要求總數分組。它可以是內部網路或外部網路。這個分組對於了解流量百分比是來自內部網路還是外部網路非常有幫助。 |
| 失敗要求總數：同盟服務所處理的失敗要求總數。<br> (此度量僅能在適用於 Windows Server 2012 R2 的 AD FS 上使用)| 錯誤類型 | 這將會根據預先定義的錯誤類型，顯示錯誤數目。這個分組對於了解常見的錯誤類型非常有幫助。<ul><li>不正確的使用者名稱或密碼：由於不正確的使用者名稱或密碼而導致的錯誤。</li> <li>「外部網路鎖定」：由於從外部網路鎖定的使用者收到要求而導致的失敗</li><li>「過期的密碼」：由於使用者使用已過期的密碼登入而導致的失敗。</li><li>「停用的帳戶」：由於使用者使用已停用的帳戶登入而導致的失敗。</li><li>「裝置驗證」：由於使用者無法使用裝置驗證進行驗證而導致的失敗。</li><li>「使用者憑證驗證」:由於使用者因為憑證無效而無法進行驗證所導致的失敗。</li><li>「MFA」：由於使用者無法使用 Multi Factor Authentication 來進行驗證而導致的失敗。</li><li>「其他認證」：「發佈授權」：由於授權失敗而導致的失敗。</li><li>「發佈委派」：由於發佈委派錯誤而導致的失敗。</li><li>「權杖接受」：由於 ADFS 拒絕來自第三方身分識別提供者的權杖而導致的失敗。</li><li>「通訊協定」：由於通訊協定錯誤而導致的失敗。</li><li>「未知」：全部攔截。不適合已定義的類別的其他任何失敗。</li> |
| | 伺服器 | 這將會根據伺服器，為錯誤分組。這對於了解跨伺服器的錯誤分佈非常有幫助。分佈不平均可能是伺服器處於錯誤狀態的指標。 |
| | 網路位置 | 這將會根據要求的網路位置 (內部網路與外部網路)，為錯誤分組。這對於了解失敗的要求類型非常有幫助。 |
| | 應用程式 | 這將會根據目標應用程式 (信賴憑證者)，為失敗分組。這對於了解哪個目標應用程式將看到最多錯誤數目非常有幫助。 |
| 使用者計數：系統中有效的唯一使用者的平均數目 | 全部 | 這會提供在所選時間配量內使用同盟服務的使用者平均數目的計數。系統不會為使用者分組。<br>平均值將取決於選取的時間配量。 |
| | 應用程式 | 這將會根據目標應用程式 (信賴憑證者)，為使用者的平均數目分組。這對於了解有多少使用者正在使用哪一個應用程式非常有幫助。 |


## AD FS 的效能監視
Azure AD Connect Health 效能監視會提供關於度量的監視資訊。選取 [監視] 方塊將會開啟一個刀鋒視窗，以提供關於度量的詳細的資訊。


![Azure AD Connect Health 入口網站](./media/active-directory-aadconnect-health/perf1.png)


選取刀鋒視窗頂端的 [篩選] 選項，您可以依伺服器篩選以查看個別伺服器的度量。若要變更度量，只要在監視刀鋒視窗底下的監視圖表上按一下滑鼠右鍵，並選取 [編輯圖表]。接著，在開啟的新刀鋒視窗中，您可以從下拉式清單中選取其他度量，並指定檢視效能資料的時間範圍。

## AD FS 報告
Azure AD Connect Health 提供有關 AD FS 活動與效能的報告。這些報告可協助系統管理員深入了解 AD FS 伺服器上的活動。

### 使用者名稱/密碼登入失敗的前 50 個使用者

AD FS 伺服器上驗證要求失敗的常見原因之一就是要求所提供的認證無效，也就是錯誤的使用者名稱或密碼。這通常是因為密碼太複雜、忘記密碼或打錯字。

但還是有其他原因會導致 AD FS 伺服器所處理的這類要求數量超出預期。其中包括可快取使用者認證的應用程式和到期的認證，或嘗試以一系列的常見密碼登入使用者帳戶的惡意使用者。

Azure AD Connect Health for ADFS 會提供一份報告，內容有關因為使用者名稱或密碼無效而登入嘗試失敗的前 50 個使用者。處理伺服器陣列中所有 AD FS 伺服器所產生的稽核事件，即可達成此目的

![Azure AD Connect Health 入口網站](./media/active-directory-aadconnect-health-adfs/report1a.png)

您可以在這份報告中輕鬆取得下列資訊︰

- 過去 30 天內使用者名稱/密碼錯誤的失敗要求總數
- 每天由於使用者名稱/密碼不正確而登入失敗的平均使用者人數。

按一下此組件即可前往可提供其他詳細資料的主要報告刀鋒視窗。其中包括一個提供趨勢資訊的圖形，以便建立有關使用者名稱或密碼錯誤之要求的基準，以及前 50 個使用者清單及其嘗試失敗次數。

此圖形可提供以下資訊：

- 每天由於使用者名稱/密碼不正確而登入失敗的總數。
- 每天登入失敗的唯一使用者總數。

![Azure AD Connect Health 入口網站](./media/active-directory-aadconnect-health-adfs/report2a.png)

此報告提供下列資訊：

| 報告項目 | 說明
| ------ | -------- |
|使用者識別碼| 顯示所使用的使用者識別碼。請注意，這個值是使用者在某些情況下輸入的內容，您也會看到使用的錯誤使用者識別碼。|
|嘗試失敗|顯示該特定使用者識別碼的嘗試失敗總數。此資料表是依據最多失敗嘗試次數以遞減順序排序。|
|上次失敗|此項目顯示上次發生失敗時的時間戳記。



>[AZURE.NOTE] 此報告會每隔 2 小時以該段時間內收集的新資訊自動進行更新。因此，報告中不包含過去 2 小時內的登入嘗試。



## 相關連結

* [Azure AD Connect Health](active-directory-aadconnect-health.md)
* [Azure AD Connect Health 代理程式安裝](active-directory-aadconnect-health-agent-install.md)
* [Azure AD Connect Health 操作](active-directory-aadconnect-health-operations.md)
* [使用 Azure AD Connect Health 進行同步處理](active-directory-aadconnect-health-sync.md)
* [在 AD DS 使用 Azure AD Connect Health](active-directory-aadconnect-health-adds.md)
* [Azure AD Connect Health 常見問題集](active-directory-aadconnect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](active-directory-aadconnect-health-version-history.md)

<!---HONumber=AcomDC_0928_2016-->