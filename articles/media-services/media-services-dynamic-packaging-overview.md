<properties
	pageTitle="動態封裝概觀 | Microsoft Azure"
	description="本主題提供動態封裝的概觀。"
	authors="Juliako"
	manager="erikre"
	editor=""
	services="media-services"
	documentationCenter=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016" 
	ms.author="juliako"/>


#動態封裝

##Overview

Microsoft Azure Media Services 可用來針對數種用戶端技術 (例如 iOS、XBOX、Silverlight、Windows 8) 提供許多媒體來源檔案格式、媒體串流格式和內容保護格式。這些用戶端各自使用不同的通訊協定，例如 iOS 需要 HTTP 即時串流 (HLS) V4 格式，而 Silverlight 與 Xbox 需要 Smooth Streaming。如果您有一組自動調整位元速率 (多位元速率) MP4 (ISO Base Media 14496-12) 檔案或一組自動調整位元速率 Smooth Streaming 檔案，想要傳遞給了解 MPEG DASH、HLS 或 Smooth Streaming 的用戶端，應該利用媒體服務動態封裝。

使用動態封裝，您只需要建立包含一組自動調整位元速率 MP4 檔案或自動調整位元速率 Smooth Streaming 檔案的資產。然後隨選串流伺服器會根據資訊清單或片段要求中的指定格式，確保您以自己選擇的通訊協定接收串流。因此，您只需要儲存及支付一種儲存格式之檔案的費用，媒體服務會根據用戶端的要求建置及提供適當的回應。

下圖顯示傳統編碼和靜態封裝工作流程。

![靜態編碼](./media/media-services-dynamic-packaging-overview/media-services-static-packaging.png)

下圖顯示動態封裝工作流程。

![動態編碼](./media/media-services-dynamic-packaging-overview/media-services-dynamic-packaging.png)


>[AZURE.NOTE]若要利用動態封裝，您必須先取得至少一個串流端點的隨選串流單位，您打算從中傳遞您的內容。如需詳細資訊，請參閱[如何調整媒體服務](media-services-portal-manage-streaming-endpoints.md)。

##常見的案例

1. 上傳輸入檔案 (稱為夾層檔)。例如，H.264、MP4 或 WMV (如需支援格式清單，請參閱[媒體編碼器標準所支援的格式](media-services-media-encoder-standard-formats.md))。

1. 將夾層檔編碼為 H.264 MP4 自動調整位元速率集。

1. 藉由建立隨選定位器，發行包含自動調整位元速率 MP4 集的資產。

1. 建置串流 URL 來存取和串流您的內容。


##準備動態串流的資產

若要準備動態串流的資產，您有兩個選項：

1. [上傳主檔案](media-services-dotnet-upload-files.md)。
2. [使用媒體編碼器標準編碼器產生 H.264 MP4 自動調整位元速率集](media-services-dotnet-encode-with-media-encoder-standard.md)。
3. [串流處理內容](media-services-deliver-content-overview.md)。

-或-

1. 上傳預先編碼的 MP4 檔案。

>[AZURE.NOTE] 但不建議您這樣做。

2. [驗證預先編碼的檔案](media-services-static-packaging.md#validating-adaptive-bitrate-mp4s-encoded-with-external-encoders)。
3. [串流處理內容](media-services-deliver-content-overview.md)。


##<a id="unsupported_formats"></a>動態封裝不支援的格式

動態封裝不支援下列來源檔案格式。

- Dolby digital mp4 檔案。
- Dolby digital smooth 檔案。

##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0921_2016-->