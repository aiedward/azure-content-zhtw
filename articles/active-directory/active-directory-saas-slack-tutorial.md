<properties 
    pageTitle="教學課程：Azure Active Directory 與 Slack 整合 | Microsoft Azure" 
    description="了解如何使用 Slack 搭配 Azure Active Directory 來啟用單一登入、自動佈建和更多功能！" 
    services="active-directory" 
    authors="jeevansd"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="09/19/2016" 
    ms.author="jeedes" />

#教學課程：Azure Active Directory 與 Slack 整合
  
本教學課程的目的是要示範 Azure 與 Slack 的整合。本教學課程中說明的案例假設您已經具有下列項目：

-   有效的 Azure 訂用帳戶
-   啟用 Slack 單一登入的訂用帳戶
  
完成本教學課程之後，您指派給 Slack 的 Azure AD 使用者就能夠從您的 Slack 公司網站 (服務提供者起始登入)，或使用[存取面板](active-directory-saas-access-panel-introduction.md)來單一登入應用程式。
  
本教學課程中說明的案例由下列建置組塊組成：

1.  啟用 Slack 的應用程式整合
2.  設定單一登入
3.  設定使用者佈建
4.  指派使用者

![案例](./media/active-directory-saas-slack-tutorial/IC794980.png "案例")

##啟用 Slack 的應用程式整合
  
本節的目的是概述如何啟用 Slack 的應用程式整合。

###若要啟用 Slack 的應用程式整合，請執行下列步驟：

1.  在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory]。

    ![Active Directory](./media/active-directory-saas-slack-tutorial/IC700993.png "Active Directory")

2.  從 [目錄] 清單中，選取要啟用目錄整合的目錄。

3.  若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]。

    ![應用程式](./media/active-directory-saas-slack-tutorial/IC700994.png "應用程式")

4.  按一下頁面底部的 [新增]。

    ![新增應用程式](./media/active-directory-saas-slack-tutorial/IC749321.png "新增應用程式")

5.  在 [欲執行動作] 對話方塊中，按一下 [從資源庫加入應用程式]。

    ![從資源庫新增應用程式](./media/active-directory-saas-slack-tutorial/IC749322.png "從資源庫新增應用程式")

6.  在 [搜尋方塊]中，輸入 **Slack**。

    ![應用程式庫](./media/active-directory-saas-slack-tutorial/IC794981.png "應用程式庫")

7.  在結果窗格中，選取 [Slack]，然後按一下 [完成] 加入應用程式。

    ![案例](./media/active-directory-saas-slack-tutorial/IC796925.png "案例")

##設定單一登入
  
本節的目的是概述如何依據 SAML 通訊協定來使用同盟，讓使用者能夠以自己的 Azure AD 帳戶在 Slack 中進行驗證。在此程序中，您必須建立 Base-64 編碼的憑證檔案。如果您不熟悉這個程序，請參閱[如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)。

###若要設定單一登入，請執行下列步驟：

1.  在 Azure 傳統入口網站的 [Slack] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。

    ![設定單一登入](./media/active-directory-saas-slack-tutorial/IC794982.png "設定單一登入")

2.  在 [要如何讓使用者登入 Slack] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按 [下一步]。

    ![設定單一登入](./media/active-directory-saas-slack-tutorial/IC794983.png "設定單一登入")

3.  在 [設定應用程式 URL] 頁面的 [Slack 登入 URL] 文字方塊中，輸入您的 Slack 租用戶 URL (例如："*https://azuread.slack.com*"*)，然後按 [下一步]*。

    ![設定應用程式 URL](./media/active-directory-saas-slack-tutorial/IC794984.png "設定應用程式 URL")

4.  在 [設定在 Slack 單一登入] 頁面上，按一下 [下載憑證] 以下載您的憑證，然後在本機電腦上儲存憑證檔案。

    ![設定單一登入](./media/active-directory-saas-slack-tutorial/IC794985.png "設定單一登入")

5.  在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Slack 公司網站。

6.  移至 [Microsoft Azure AD] > [小組設定]。

    ![小組設定](./media/active-directory-saas-slack-tutorial/IC794986.png "小組設定")

7.  在 [小組設定] 區段中，按一下 [驗證] 索引標籤，然後按一下 [變更設定]。

    ![小組設定](./media/active-directory-saas-slack-tutorial/IC794987.png "小組設定")

8.  在 [SAML 驗證設定] 對話方塊上，執行下列步驟：

    ![SAML 設定](./media/active-directory-saas-slack-tutorial/IC794988.png "SAML 設定")

    1.  在 Azure 傳統入口網站的 [設定在 Slack 單一登入] 對話頁面上，複製 [SAML SSO URL] 值，然後將它貼至 [SAML 2.0 端點 (HTTP)] 文字方塊中。
    2.  在 Azure 傳統入口網站中的 [設定在 Slack 單一登入] 對話頁面上，複製 [簽發者 URL] 值，然後將它貼至 [識別提供者簽發者] 文字方塊中。
    3.  從您下載的憑證建立「Base-64 編碼」檔案。
    
        >[AZURE.TIP] 如需詳細資訊，請參閱[如何將二進位憑證轉換成文字檔](http://youtu.be/PlgrzUZ-Y1o)

    4.  在記事本中開啟您的 base-64 編碼的憑證，將其內容複製到剪貼簿，然後貼到 [公開憑證] 文字方塊中。
    5.  取消選取 [允許使用者變更其電子郵件地址]。
    6.  選取 [允許使用者選擇自己的使用者名稱]。
    7.  針對 [必須使用您的小組的驗證]，選取 [此為選擇性]。
    8.  按一下 [儲存組態]。

9.  在 Azure 傳統入口網站上，選取單一登入設定確認，然後按一下 [完成] 來關閉 [設定單一登入] 對話方塊。

    ![設定單一登入](./media/active-directory-saas-slack-tutorial/IC794989.png "設定單一登入")

##設定使用者佈建
  
若要讓 Azure AD 使用者能夠登入 Slack，必須將他們佈建到 Slack。
  
沒有動作項目可讓您設定 Slack 使用者佈建。當受指派使用者嘗試登入 Slack 時，則會自動建立一個 Slack 帳戶 (必要的話)。

##指派使用者
  
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

###若要將使用者指派到 Slack，請執行下列步驟：

1.  在 Azure 傳統入口網站中建立測試帳戶。

2.  在 [Slack] 應用程式整合頁面上，按一下 [指派使用者]。

    ![指派使用者](./media/active-directory-saas-slack-tutorial/IC794990.png "指派使用者")

3.  選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。

    ![是](./media/active-directory-saas-slack-tutorial/IC767830.png "是")
  
如果要測試您的單一登入設定，請開啟存取面板。如需存取面板的詳細資訊，請參閱[存取面板簡介](active-directory-saas-access-panel-introduction.md)。

<!---HONumber=AcomDC_0921_2016-->