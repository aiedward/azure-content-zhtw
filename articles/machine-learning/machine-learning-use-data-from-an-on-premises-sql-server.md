<properties
pageTitle="在 Machine Learning 中使用來自內部部署 SQL Server 資料庫的資料 | Azure"
description="使用來自內部部署 SQL Server 資料庫的資料，利用 Azure Machine Learning 執行進階分析。"
services="machine-learning"
documentationCenter=""
authors="garyericson"
manager="jhubbard"
editor="cgronlun"/>

<tags
ms.service="machine-learning"
ms.workload="data-services"
ms.tgt_pltfrm="na"
ms.devlang="na"
ms.topic="article"
ms.date="09/16/2016"
ms.author="garye;krishnan"/>

# 使用來自內部部署 SQL Server 資料庫的資料，利用 Azure Machine Learning 執行進階分析


使用內部部署資料的企業通常會想要針對他們的機器學習服務工作負載來利用雲端的範圍和靈活度。但是他們又不想因為將內部部署資料移至雲端而中斷目前的商務程序和工作流程。Azure Machine Learning 現在支援從內部部署 SQL Server 資料庫讀取資料，然後利用此資料，針對某個模型進行訓練與評分。您不再需要手動複製和同步處理雲端與內部部署伺服器之間的資料。相反地，Azure Machine Learning Studio 中的**匯入資料**模組現在可以直接從您的內部部署 SQL Server 資料庫讀取，以進行您的訓練和評分作業。

本文提供如何將內部部署 SQL Server 資料輸入至 Azure Machine Learning 的概觀。它假設您已熟悉像是工作區、模組、資料集、實驗等 Azure Machine Learning 概念。

> [AZURE.NOTE] 此功能不適用於免費的工作區。如需機器學習服務定價和層級的詳細資訊，請參閱 [Azure 機器學習服務定價](https://azure.microsoft.com/pricing/details/machine-learning/)。

<!-- --> 

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]


## 安裝 Microsoft 資料管理閘道器

若要存取 Azure Machine Learning 中的內部部署 SQL Server 資料庫，您需要下載並安裝 Microsoft 資料管理閘道器。當您在 Machine Learning Studio 中設定閘道器連接時，您將有機會使用 [下載並註冊資料閘道器] 對話方塊來下載並安裝閘道器，如下所述。

您也可以從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=39717)事先下載並執行 MSI 安裝程式套件來安裝資料管理閘道器。選擇最新版本，選取電腦適用的 32 位元或 64 位元。MSI 也可用來將現有的資料管理閘道器升級到最新版本，並保留所有設定。

閘道器的必要條件如下：

- 支援的 Windows 作業系統版本包括 Windows 7、Windows 8/8.1、Windows 10、Windows Server 2008 R2、Windows Server 2012 及 Windows Server 2012 R2。
- 閘道器電腦的建議組態至少為 2 GHz、4 核心、8 GB RAM 和 80 GB 磁碟。
- 如果主機電腦休眠，閘道器即無法回應資料要求。因此，安裝閘道之前，請先在電腦上設定適當的電源計劃。如果電腦已設定為休眠，安裝閘道器時會顯示訊息。
- 由於複製活動會以特定的頻率發生，因此，電腦上的資源使用量 (CPU、記憶體) 也會遵循與尖峰和閒置時間相同的模式。資源使用率也仰賴要移動的資料量。如果有多個複製作業正在進行，您將可觀察到資源使用量會在尖峰時段增加。雖然上述的基本組態在技術上來說即已足夠，但是根據資料移動的特定負載而定，您的組態最好具備比基本組態還要多的資源。

您應該在設定和使用資料管理閘道器時考慮下列內容：

- 單一電腦上只能安裝一個資料管理閘道器的執行個體。
- 您可以針對多個內部部署資料來源使用單一閘道器。
- 您可以將不同電腦上的多個閘道器連接到相同的內部部署資料來源。
- 您一次只能針對一個工作區設定一個閘道器。目前無法在工作區之間共用閘道器。
- 您可以針對單一工作區設定多個閘道器。例如，您可能想要使用一個閘道器，在開發期間連接到您的測試資料來源，並在準備運作之際使用生產閘道器。
- 雖然閘道器不需要和資料來源位在相同的電腦上，但越接近資料來源可縮短閘道器連線到資料來源的時間。建議您安裝閘道器的電腦不同於裝載內部部署資料來源的電腦，如此閘道器和資料來源才不會爭奪資源。
- 如果您已在電腦上安裝用於 Power BI 或 Azure Data Factory 案例的閘道器，請於另一台電腦上安裝另一個用於 Azure Machine Learning 的閘道器。

    > [AZURE.NOTE] 您無法在同一台電腦上同時執行資料管理閘道器和 Power BI Gateway。

- 即使您針對其他資料使用 Azure ExpressRoute，還是需要針對 Azure Machine Learning 使用資料管理閘道器。即使您使用 ExpressRoute，也應該將資料來源視為內部部署資料來源 (其位於防火牆後面)，並使用資料管理閘道來建立 Machine Learning 與資料來源之間的連接。

您可以在[利用資料管理閘道在內部部署來源和雲端之間移動資料](../data-factory/data-factory-move-data-between-onprem-and-cloud.md#considerations-for-using-data-management-gateway)一文中，從[使用資料管理閘道的考量](../data-factory/data-factory-move-data-between-onprem-and-cloud.md#considerations-for-using-data-management-gateway)一節開始找到有關安裝必要條件、安裝步驟及疑難排解秘訣的詳細資訊。

## <span id="using-the-data-gateway-step-by-step-walk" class="anchor"><span id="_Toc450838866" class="anchor"></span></span>將內部部署 SQL Server 資料庫的資料輸入至 Azure Machine Learning


在本逐步解說中，您將會在 Azure Machine Learning 工作區中設定資料管理閘道器、設定該閘道器，然後讀取來自內部部署 SQL Server 資料庫的資料。

> [AZURE.TIP] 開始之前，請先針對 `studio.azureml.net` 停用瀏覽器的快顯封鎖程式。如果您使用 Google Chrome 瀏覽器，請下載並安裝數個可在 Google Chrome 線上應用程式商店 [Click Once 應用程式擴充功能](https://chrome.google.com/webstore/search/clickonce?_category=extensions)上取得的外掛程式之一。

### 步驟 1：建立閘道器

第一個步驟是建立並設定閘道器來存取您的內部部署 SQL 資料庫。

1. 登入 [Azure Machine Learning Studio](https://studio.azureml.net/Home/)，然後選取您想要使用的工作區。

2. 按一下左側的 [設定] 刀鋒視窗，然後按一下頂端的 [資料閘道] 索引標籤。

3. 按一下螢幕底部的 [新增資料閘道]。

    ![](media/machine-learning-use-data-from-an-on-premises-sql-server/new-data-gateway-button.png)

4. 在 [新增資料閘道] 對話方塊中，輸入**閘道名稱**並選擇性地新增**描述**。按一下右下角的箭號，以移至組態的下一個步驟。

    ![](media/machine-learning-use-data-from-an-on-premises-sql-server/new-data-gateway-dialog-enter-name.png)

5. 在 [下載並註冊資料閘道器] 對話方塊中，將閘道器註冊金鑰複製到剪貼簿。

    ![](media/machine-learning-use-data-from-an-on-premises-sql-server/download-and-register-data-gateway.png)

6. <span id="note-1" class="anchor"></span>如果您還沒有下載並安裝 Microsoft 資料管理閘道，則可按一下 [下載資料管理閘道]。這會帶您前往 Microsoft 下載中心，您可以在其中選取所需的閘道器版本，然後下載並安裝它。您可以在[利用資料管理閘道在內部部署來源和雲端之間移動資料](../data-factory/data-factory-move-data-between-onprem-and-cloud.md)一文的前幾節中，找到有關安裝必要條件、安裝步驟及疑難排解秘訣的詳細資訊。

7. 安裝閘道之後，「資料管理閘道組態管理員」便會隨即開啟，並顯示 [註冊閘道] 對話方塊。貼上您複製到剪貼簿的**閘道註冊金鑰**，然後按一下 [註冊]。

8. 如果您已經安裝閘道，請執行「資料管理閘道組態管理員」，按一下 [變更金鑰]，貼上您複製到剪貼簿的**閘道註冊金鑰**，然後按一下 [確定]。

9. 安裝完成時，便會隨即顯示「Microsoft 資料管理閘道組態管理員」的 [註冊閘道] 對話方塊。貼上您複製到剪貼簿的閘道註冊金鑰，然後按一下 [註冊]。

    ![](media/machine-learning-use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-register-gateway.png)

10. 當「Microsoft 資料管理閘道組態管理員」中的 [常用] 索引標籤中設定了下列值時，閘道組態即已完成︰

    - [閘道名稱] 和 [執行個體名稱] 設定為閘道的名稱。

    - [註冊] 設定為 [已註冊]。

    - [狀態] 設定為 [已啟動]。

    - 底部的狀態列會顯示 [已連接到資料管理閘道雲端服務] 和一個綠色的核取記號。

     ![](media/machine-learning-use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-registered.png)

     註冊成功時，也會更新 Azure Machine Learning Studio。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\gateway-registered.png)

11. 在 [下載並註冊資料閘道] 對話方塊中，按一下核取記號以完成安裝。[設定] 頁面會將閘道狀態顯示為「線上」。在右側窗格中，您將會發現狀態和其他有用的資訊。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\gateway-status.png)

12. 在 [Microsoft 資料管理閘道組態管理員] 中，切換到 [憑證] 索引標籤。此索引標籤上指定的憑證可用來加密/解密您在入口網站指定之內部部署資料存放區的認證。這是產生的預設憑證。Microsoft 建議將此憑證變更為您自己在憑證管理系統中備份的憑證。按一下 [變更] 改為使用您自己的憑證。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\data-gateway-configuration-manager-certificate.png)

13. (選擇性) 如果您想啟用詳細資訊記錄功能來疑難排解閘道的問題，請在 [Microsoft 資料管理閘道組態管理員] 中切換到 [診斷] 索引標籤，然後選取 [啟用詳細資訊記錄以進行疑難排解] 選項。您可以在 [Windows 事件檢視器] 中，於 [應用程式及服務記錄檔] -&gt; [資料管理閘道] 節點下方找到記錄資訊。您也可以使用 [診斷] 索引標籤，使用閘道來測試與內部部署資料來源的連線。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\data-gateway-configuration-manager-verbose-logging.png)

如此即可完成 Azure Machine Learning 中的閘道器設定程序。您現在可以開始使用內部部署資料。

您可以在 Studio 中針對每個工作區建立和設定多個閘道器。例如，您可能想要有一個可在開發期間連接到測試資料來源的閘道器，以及另一個適用於生產資料來源的閘道器。根據貴公司的環境而定，Azure Machine Learning 可讓您彈性地設定多個閘道器。目前您無法在工作區之間共用一個閘道器，而且單一電腦上只能安裝一個閘道器。如需安裝閘道時的其他考量，請參閱[利用資料管理閘道在內部部署來源和雲端之間移動資料](../data-factory/data-factory-move-data-between-onprem-and-cloud.md)一文中的[使用資料管理閘道的考量](../data-factory/data-factory-move-data-between-onprem-and-cloud.md#considerations-for-using-data-management-gateway)。

### 步驟 2︰使用閘道器讀取來自內部部署資料來源的資料

設定閘道之後，您可以將**匯入資料**模組新增到會輸入來自內部部署 SQL Server 資料庫之資料的實驗。

1.  在 Machine Learning Studio 中，選取 [實驗] 索引標籤，按一下左下角的 [+新增]，然後選取 [空白實驗] \(或從數個可用的範例實驗中選取其中一個)。

2.  找出**匯入模型**模組，並將它拖曳到實驗畫布。

3.  按一下畫布下方的 [另存新檔]。輸入「Azure Machine Learning 內部部署 SQL Server 教學課程」做為實驗名稱，選取工作區，然後按一下 [確定] 核取記號。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\experiment-save-as.png)

4.  按一下 [匯入資料] 模組來選取它，然後在畫布右邊的 [屬性] 窗格中，選取 [資料來源] 下拉式清單中的 [內部部署 SQL Database]。

5.  選取您安裝並註冊的**資料閘道**。您可以藉由選取 [(新增資料閘道...)] 來設定其他閘道器。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\import-data-select-on-premises-data-source.png)

6.  輸入 SQL **Database 伺服器名稱**和**資料庫名稱**，以及您想要執行的 SQL **Database 查詢**。

7.  按一下 [使用者名稱和密碼] 下方的 [輸入值]，然後輸入資料庫認證。根據您內部部署 SQL Server 的設定方式而定，您可以使用 Windows 整合式驗證或 SQL Server 驗證。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\database-credentials.png)
    
    「需要值」的訊息將變更為「已設定值」並具有綠色核取記號。除非資料庫資訊或密碼變更，否則您只需輸入認證一次。Azure Machine Learning 會使用您在安裝閘道器來加密雲端中的認證時所提供的憑證。Azure 永遠都不會儲存未加密的內部部署認證。

    ![](media\machine-learning-use-data-from-an-on-premises-sql-server\import-data-properties-entered.png)

8.  按一下 [執行] 來執行實驗。

實驗完成執行之後，就可以將您從資料庫匯入的資料視覺化，方法是按一下**匯入資料**模組的輸出連接埠，然後選取 [視覺化]。

當實驗完成開發之後，您就能部署和操作您的模型。使用批次執行服務時，將會讀取來自**匯入資料**模組中所設定之內部部署 SQL Server 資料庫的資料並用於計分。雖然您可以使用要求回應服務來對內部部署資料進行評分，但是 Microsoft 建議改用 [Excel 增益集](machine-learning-excel-add-in-for-web-services.md)。目前不論是在您的實驗或是已發行的 Web 服務中，都不支援透過**匯出資料**寫入內部部署 SQL Server 資料庫。

<!---HONumber=AcomDC_0921_2016-->