<properties
	pageTitle="Windows 通用 SDK 整合"
	description="適用於 Azure Mobile Engagement 的 Windows 通用 SDK 整合" 									
	services="mobile-engagement"
	documentationCenter="mobile"
	authors="piyushjo"
	manager="dwrede"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-windows-store"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="08/12/2016"
	ms.author="piyushjo;ricksal" />

#適用於 Azure Mobile Engagement 的 Windows 通用 SDK 整合

此文件說明適用於 Azure Mobile Engagement Windows 通用 SDK 的所有整合及組態選項。

## 必要條件

開始本教學課程之前，您必須先完成 [15 分鐘教學課程](mobile-engagement-windows-store-dotnet-get-started.md)。

## 進階功能

### 報告功能
您可以新增這些功能：

1. [進階報告選項](mobile-engagement-windows-store-advanced-reporting.md)

2. [進階組態選項](mobile-engagement-windows-store-advanced-configuration.md)

### 通知

[如何在您的 Windows 通用 App 整合 Reach (通知)](mobile-engagement-windows-store-integrate-engagement-reach.md)

### 標記計畫實作：

[如何在您的 Windows 通用 App 使用進階的 Mobile Engagement 標記 API](mobile-engagement-windows-store-use-engagement-api.md)

##版本資訊

###3\.4.0 (04/19/2016)

-   Reach 重疊增強功能。
-   已加入 "TestLogLevel" API 來啟用/停用/篩選 SDK 所發出的主控台記錄檔。
-   已修正活動中通知目標設定為應用程式啟動時未顯示的第一個活動。

如需較早版本，請參閱[完整版本資訊](mobile-engagement-windows-store-release-notes.md)

##升級程序

如果您已經整合舊版 Engagement 到您的應用程式，在升級 SDK 時您必須考慮以下幾點。

如果您錯過了幾個版本的 SDK，您必須遵循幾個程序。請參閱完整的[升級程序](mobile-engagement-windows-store-upgrade-procedure.md)。例如，如果您要從 0.10.1 移轉到 0.11.0，必須先遵循「從 0.9.0 到 0.10.1」的程序，然後「從 0.10.1 到 0.11.0」的程序。

###從 3.3.0 到 3.4.0

####測試記錄檔

SDK 所產生的主控台記錄檔現在可以啟用/停用/篩選。若要自訂，請將屬性 `EngagementAgent.Instance.TestLogEnabled` 更新為 `EngagementTestLogLevel` 列舉的其中一個可用值，例如︰

			EngagementAgent.Instance.TestLogLevel = EngagementTestLogLevel.Verbose;
			EngagementAgent.Instance.Init();

####資源

已改善 Reach 重疊。它是 SDK NuGet 封裝資源的一部分。

當您升級到新版的 SDK，可以選擇是否要保留資源之重疊資料夾中的現有檔案︰

* 如果先前的重疊對您而言可以運作，或是您要手動整合 `WebView` 元素，則您可以決定保留現有檔案，這樣仍然可以運作。
* 若要更新為新的重疊，請將資源的整個 `overlay` 資料夾取代為來自 SDK 封裝的新資料夾 (UWP 應用程式︰升級後，您可以從 %USERPROFILE%\\.nuget\\packages\\MicrosoftAzure.MobileEngagement\\3.4.0\\content\\win81\\Resources 取得新的重疊資料夾)。

> [AZURE.WARNING] 使用新的重疊會覆寫先前版本上所做的任何自訂。

### 從舊版升級

請參閱[升級程序](mobile-engagement-windows-store-upgrade-procedure.md)

<!---HONumber=AcomDC_0817_2016-->