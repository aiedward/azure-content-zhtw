<properties 
	pageTitle="Azure 媒體服務概念 | Microsoft Azure" 
	description="本主題提供 Azure 媒體服務概念的概觀" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/19/2016"
	ms.author="juliako"/>

#Azure 媒體服務概念 

本主題提供媒體服務概念的最重要概觀。

##<a id="assets"></a>資產和儲存體

###Assets

[資產](https://msdn.microsoft.com/library/azure/hh974277.aspx)包含數位檔案 (包括視訊、音訊、影像、縮圖集合、文字播放軌和隱藏式輔助字幕檔案)，以及這些檔案的相關中繼資料。將數位檔案上傳到資產之後，可以用於媒體服務編碼和串流工作流程。

資產會對應到 Azure 儲存體帳戶中的 blob 容器，且資產中的檔案會儲存為該容器中的 blob。

決定要在資產中上傳及儲存何種媒體內容時，適用下列考量：

- 資產應該只包含一個唯一的媒體內容執行個體。例如，一份電視影集、電影或廣告的剪輯。
- 資產不應包含多份視聽檔案的轉譯或剪輯。不當使用資產的例子是嘗試在資產內儲存一份以上電視影集、廣告或同一作品的多個拍攝角度。在資產內儲存多份視聽檔案的轉譯或剪輯，可能會在工作流程稍後的提交編碼工作、串流處理和保護資產傳遞時造成問題。

###資產檔案 
[AssetFile](https://msdn.microsoft.com/library/azure/hh974275.aspx) 代表儲存在 blob 容器中的實際視訊或音訊檔案。資產檔案一律會與資產相關聯，一個資產可包含一或多個檔案。如果資產檔案物件並未與 blob 容器中的數位檔案相關聯，媒體服務編碼器工作將會失敗。

**AssetFile** 執行個體和實際媒體檔是兩個不同的物件。AssetFile 執行個體包含媒體檔案的相關中繼資料，而媒體檔案包含實際的媒體內容。

請勿在不使用媒體服務 API 的情況下，嘗試變更由媒體服務所產生的 blob 容器內容。

###資產加密選項

根據您想要上傳、儲存和傳遞的內容類型，媒體服務會提供各種加密選項供您選擇。

**無** - 不使用加密。這是預設值。請注意，使用這個選項時您的內容在傳輸中或在儲存體中不受保護。

如果您計劃使用漸進式下載傳遞 MP4，請使用此選項來上傳內容。

**StorageEncrypted** - 使用此選項，使用 AES 256 位元加密對您的純文字內容進行本機加密，然後將它上傳到已靜止加密儲存的 Azure 儲存體。使用儲存體加密保護的資產會在編碼之前自動解除加密並放在加密的檔案系統中，並且選擇性地在上傳為新的輸出資產之前重新加密。儲存體加密的主要使用案例是當您想要使用強式加密保護靜止在磁碟上的高品質輸入媒體檔案時。

若要傳遞儲存體加密資產，您必須設定資產的傳遞原則，讓媒體服務知道您的內容傳遞方式。串流處理資產之前，串流伺服器會移除儲存體加密，並使用指定的傳遞原則來串流處理您的內容 (例如，AES、PlayReady 或不加密)。

**CommonEncryptionProtected** - 如果您想要使用一般加密或 PlayReady DRM (例如，受到 PlayReady DRM 保護的 Smooth Streaming) 來加密 (或上傳已加密) 內容，請使用此選項。

**EnvelopeEncryptionProtected** - 如果您想要保護利用進階加密標準 (AES) 所加密的 HTTP 即時串流 (HLS) 或上傳利用相同標準所加密的已保護 HTTP 即時串流 (HLS)，請使用此選項。請注意，如果您上傳已透過 AES 加密的 HLS，它必須是由 Transform Manager 加密。

###存取原則 

[AccessPolicy](https://msdn.microsoft.com/library/azure/hh974297.aspx) 定義資產存取的權限 (例如讀取、寫入和清單) 和持續時間。您通常會傳遞 AccessPolicy 物件到定位器，接著再用來存取資產中包含的檔案。


###Blob 容器

Blob 容器提供一組 blob。Blob 容器在媒體服務中是做為存取控制的分界點，以及資產上的共用存取簽章 (SAS) 定位器。Azure 儲存體帳戶可以包含無限多個 blob 容器。容器可以儲存無限制的 Blob。

>[AZURE.NOTE]請勿在不使用媒體服務 API 的情況下，嘗試變更由媒體服務所產生的 blob 容器內容。

###<a id="locators"></a>定位器

[定位器](https://msdn.microsoft.com/library/azure/hh974308.aspx)提供一個進入點，可供存取資產中包含的檔案。存取原則用來定義用戶端可存取特定資產的權限與持續時間。定位器對存取原則可以有多對一關聯性，以便在所有用戶端都使用相同的權限與持續時間設定時，可以讓不同定位器對不同的用戶端提供不同的開始時間和連線類型；不過，由於 Azure 儲存體服務所設定的共用存取原則限制，特定資產一次不能有超過五個唯一定位器與之相關聯。

媒體服務支援兩種類型的定位器：一個是 OnDemandOrigin 定位器，用於串流媒體 (例如 MPEG DASH、HLS 或 Smooth Streaming)，或漸進式下載媒體；另一個則是 SAS URL 定位器，用於上傳媒體檔案至 Azure 儲存體或從 Azure 儲存體下載媒體檔案。

請注意，建立 OrDemandOrigin 定位器時，不應使用清單權限 (AccessPermissions.List)。

###儲存體帳戶

所有對 Azure 儲存體的存取都是透過儲存體帳戶進行。媒體服務帳戶可以與一個或多個儲存體帳戶產生關聯。帳戶可以包含不限數目的容器，只要它們的大小總計低於每個儲存體帳戶 500TB 即可。媒體服務提供 SDK 層級工具，可讓您管理多個儲存體帳戶，以及在上傳至這些帳戶期間根據計量或隨機分佈，負載平衡資產的分佈。如需詳細資訊，請參閱「使用 [Azure 儲存體](https://msdn.microsoft.com/library/azure/dn767951.aspx)」。

##工作 (Job) 和工作 (Task)

[工作](https://msdn.microsoft.com/library/azure/hh974289.aspx)通常用來處理 (例如索引或編碼) 一個音訊/視訊簡報。如果您要處理多個視訊，請為每個要編碼的視訊各建立一個工作。

工作包含要進行之處理的相關中繼資料。每個工作 (Job) 包含一或多個[工作 (Task)](https://msdn.microsoft.com/library/azure/hh974286.aspx)，該工作 (Task) 指定不可部分完成的處理工作、該處理工作的輸入資產、輸出資產、媒體處理器和其相關設定。工作 (Job) 中的工作 (Task) 可以鏈結在一起，其中一項工作 (Task) 的輸出資產指定為下一個工作 (Task) 的輸入資產。透過這種方式，一項工作 (Job) 可以包含一個媒體簡報所需的所有處理。

##<a id="encoding"></a>編碼

Azure 媒體服務提供多個用於將雲端中之媒體編碼的選項。

開始使用媒體服務時，請務必了解轉碼器和檔案格式之間的差異。轉碼器是實作壓縮/解壓縮演算法的軟體，而檔案格式是保存已壓縮視訊的容器。

媒體服務提供動態封裝，這讓您以媒體服務支援的串流格式 (MPEG DASH、HLS、Smooth Streaming、HDS) 提供調適性位元速率 MP4 或 Smooth Streaming 編碼內容，而不必重新封裝成這些串流格式。

若要利用[動態封裝](media-services-dynamic-packaging-overview.md)，您需要執行下列動作：

- 將您的夾層 (來源) 檔編碼為一組調適性位元速率 MP4 檔案或調適性位元速率 Smooth Streaming 檔案 (編碼步驟稍後示範於本教學課程中)。
- 為您計畫從該處傳遞內容的串流端點取得至少一個隨選串流單元。如需詳細資訊，請參閱[如何調整隨選串流保留單元](media-services-portal-manage-streaming-endpoints.md)。

媒體服務支援本文中所描述的下列隨選編碼器：

- [Media Encoder Standard](media-services-encode-asset.md#media-encoder-standard)
- [Media Encoder Premium Workflow](media-services-encode-asset.md#media-encoder-premium-workflow)

如需支援編碼器的相關資訊，請參閱「[編碼器](media-services-encode-asset.md)」。


##即時資料流

在 Azure 媒體服務中，通道代表處理即時串流內容的管線。通道會以兩種方式之一收到即時輸入串流：

- 內部部署即時編碼器會傳送多位元速率 RTMP 或 Smooth Streaming (分散的 MP4) 到通道。您可以使用下列輸出多位元速率 Smooth Streaming 的即時編碼器：Elemental、Envivio、Cisco。下列即時編碼器會輸出 RTMP：Adobe Flash Live、Telestream Wirecast 和 Tricaster 轉錄器。內嵌的資料流可以通過通道，無需進一步的處理。接到要求時，媒體服務會傳遞串流給客戶。

- 單一位元速率串流 (下列格式之一：RTP (MPEG-TS))、RTMP 或 Smooth Streaming (分散的 MP4)) 會傳送至通道來執行啟用的通道來以媒體服務執行即時編碼。通道接著會執行即時編碼，將連入的單一位元速率串流編碼成多位元速率 (自動調整) 視訊串流。接到要求時，媒體服務會傳遞串流給客戶。

###通道

在媒體服務中，[通道](https://msdn.microsoft.com/library/azure/dn783458.aspx)負責處理即時資料流內容。通道會提供輸入端點 (內嵌 URL)，接著您再提供給即時轉碼器。通道從即時轉碼器接收即時輸入資料流，再透過一或多個 StreamingEndpoint 進行串流處理。通道也會提供預覽端點 (預覽 URL)，您可在進一步處理和傳遞之前先用來預覽及驗證您的資料流。

您可在建立通道時取得內嵌 URL 和預覽 URL。通道不需處於啟動狀態也可以取得這些 URL。當您準備好要開始從即時轉碼器推播資料給通道時，就必須先啟動通道。即時轉碼器開始內嵌資料後，您就可以預覽您的資料流。

每個媒體服務帳戶可以包含多個通道、多個程式和多個 StreamingEndpoints。根據頻寬和安全性需求，StreamingEndpoint 服務可以專屬於一或多個通道。任何 StreamingEndpoint 可以從任何通道中提取。


###程式

[程式](https://msdn.microsoft.com/library/azure/dn783463.aspx)可讓您控制即時資料流區段的發佈和儲存體。通道會管理程式。通道和程式的關聯性非常類似於傳統媒體，此處的通道有常數內容資料流，而程式的範圍是該通道上的某些計時事件。您可以透過設定 **ArchiveWindowLength** 屬性，指定您想要保留已記錄程式內容的時數。此值最小可以設定為 5 分鐘，最大可以設定為 25 個小時。

ArchiveWindowLength 也指定用戶端可從目前即時位置往回搜尋的最大時間量。程式可以執行超過指定的時間量，但是超過此時間長度的內容會被持續捨棄。此屬性的這個值也會決定用戶端資訊清單可以成長多長的時間。

每個程式都是與「資產」相關聯。若要發佈程式，您必須建立相關資產的定位器。擁有此定位器，可讓您建置可提供給用戶端的串流 URL。

通道支援最多三個同時執行的程式，因此您可以建立相同內送串流的多個封存。這可讓您視需要發行和封存事件的不同部分。例如，您的商務需求是封存 6 小時的程式，但只廣播最後 10 分鐘。為了達成此目的，您必須建立兩個同時執行的程式。其中一個程式設定為封存 6 小時的事件，但是未發行該程式。另一個程式則設定為封存 10 分鐘，並發行程式。


如需詳細資訊，請參閱：

- [使用啟用的通道來以 Azure 媒體服務執行即時編碼](media-services-manage-live-encoder-enabled-channels.md)
- [使用通道，從內部部署編碼器接收多位元速率即時串流](media-services-live-streaming-with-onprem-encoders.md)
- [配額和限制](media-services-quotas-and-limitations.md)。

##保護內容

###動態加密

Azure 媒體服務可讓您保護媒體從離開電腦到進行儲存、處理和傳遞時的安全。媒體服務可讓您傳遞利用進階加密標準 (AES) (使用 128 位元加密金鑰) 動態加密和使用 PlayReady 和/或 Widevine DRM 的一般加密 (CENC) 的內容。媒體服務也提供服務，傳遞 AES 金鑰和 PlayReady 授權給授權用戶端。

目前，您可以加密下列串流格式：HLS、MPEG DASH 和 Smooth Streaming。無法加密 HDS 串流格式，或漸進式下載。

如果您想要媒體服務加密資產，則需要建立加密金鑰 (CommonEncryption 或 EnvelopeEncryption) 與資產的關聯，同時設定金鑰的授權原則。

如果您要串流儲存體加密資產，您必須設定資產的傳遞原則，以指定資產的傳遞方式。

播放程式要求串流時，媒體服務便會使用信封加密 (使用 AES) 或一般加密 (使用 PlayReady 或 Widevine)，並使用指定的金鑰動態加密您的內容。為了將串流解密，播放程式將從金鑰傳遞服務要求金鑰。為了決定使用者是否有權取得金鑰，服務會評估為金鑰指定的授權原則。


###權杖限制

內容金鑰授權原則可能會有一個或多個授權限制：Open、權杖限制或 IP 限制。token 限制原則必須伴隨著安全權杖服務 (STS) 所發出的權杖。媒體服務支援簡單 Web 權杖 (SWT) 格式和 JSON Web 權杖 (JWT) 格式的權杖。媒體服務不提供安全權杖服務。您可以建立自訂 STS，或利用 Microsoft Azure ACS 來發行權杖。STS 必須設定為建立使用指定的索引鍵和問題宣告您在權杖限制組態中指定簽署的權杖。如果權杖有效，且權杖中的宣告符合針對金鑰 (或授權) 所設定的宣告，媒體服務金鑰傳遞服務會將所要求的金鑰 (或授權) 傳回給用戶端。

設定 token 限制原則時，您必須指定主要驗證金鑰、簽發者和對象參數。主要驗證金鑰包含簽署權杖使用的金鑰，簽發者是發行權杖的安全權杖服務。對象 (有時稱為範圍) 描述權杖或權杖獲授權存取之資源的用途。媒體服務金鑰傳遞服務會驗證權杖中的這些值符合在範本中的值。

如需詳細資訊，請參閱下列文章。

[保護內容概觀](media-services-content-protection-overview.md) [利用 AES-128 保護](media-services-protect-with-aes128.md) [利用 DRM 保護](media-services-protect-with-drm.md)

##傳遞

###<a id="dynamic_packaging"></a>動態封裝

使用媒體服務時，建議您將夾層檔案的編碼設為調適性位元速率 MP4 集，然後使用[動態封裝](media-services-dynamic-packaging-overview.md)將該 MP4 集轉換為需要的格式。


###串流端點

StreamingEndpoint 代表可以直接將內容傳遞給用戶端播放程式應用程式，或傳遞給內容傳遞網路 (CDN) 進行進一步發佈的串流服務 (Azure 媒體服務現在提供 Azure CDN 整合)。 來自 StreamingEndpoint 服務的輸出資料流可以是即時資料流，也可以是媒體服務帳戶中的隨選視訊資產。此外，您可以藉由調整縮放單元 (也稱為串流單元)，控制 StreamingEndpoint 服務處理成長頻寬需求的產能。建議您為生產環境中的應用程式配置一個或多個縮放單位。縮放單位提供您可以以 200 Mbps 為增量購買的專用流出容量和目前包含動態封裝的其他功能。

建議您使用動態封裝和/或動態加密。若要使用這些功能，負責傳送資料流的端點至少要有一個資料流單位。如需詳細資訊，請參閱「[調整資料流單位](media-services-portal-manage-streaming-endpoints.md)」。

依預設，您最多可以在媒體服務帳戶中加入 2 個串流端點。如欲要求放寬限制，請參閱「[配額和限制](media-services-quotas-and-limitations.md)」。

只有 StreamingEndpoint 處於執行中狀態時，才會向您收取費用。

###資產傳遞原則

媒體服務內容傳遞工作流程的其中一個步驟，是設定您要串流的[資產的傳遞原則](https://msdn.microsoft.com/library/azure/dn799055.aspx)。資產傳遞原則會告訴媒體服務您想要如何傳遞資產：您的資產應該動態封裝成哪個串流通訊協定 (如 MPEG DASH、HLS、Smooth Streaming 或所有)，您是否想要動態加密您的資產及其方式 (信封或一般加密)。

如果您有儲存體加密的資產，在可以串流處理資產之前，串流伺服器會先移除儲存體加密，並使用指定的傳遞原則來串流您的內容。例如，若要傳遞使用進階加密標準 (AES) 加密金鑰加密的資產，請將原則類型設定為 DynamicEnvelopeEncryption。如果您要移除儲存體加密，並且不受阻礙地串流資產，請將原則類型設定為 NoDynamicEncryption。

###漸進式下載

漸進式下載可以在下載完整檔案前就開始播放媒體。只能漸進式下載 MP4 檔案。

請注意，如果希望加密的資產可供漸進式下載，您必須先將其解密。

若要提供漸進式下載 URL 給使用者，您必須先建立 OnDemandOrigin 定位器。建立定位器會提供資產的基底路徑。接著您必須附加上 MP4 檔案的名稱。例如：

http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

###串流 URL

將內容串流到用戶端。若要提供漸進式下載 URL 給使用者，您必須先建立 OnDemandOrigin 定位器。建立定位器可針對其中包含您要串流之內容的資產，提供其基底路徑。不過，若要能夠串流此內容，您還需要進一步修改此路徑。若要建構串流資訊清單檔案的完整 URL，您必須串連定位器的 Path 值和資訊清單 (filename.ism) 的檔案名稱。接著，在定位器路徑後面附加 /Manifest 和適當的格式 (如果需要)。

您也可以透過 SSL 連線串流您的內容。若要這樣做，請確定您的串流 URL 以 HTTPS 開頭。

請注意，只有在您從中傳遞內容的串流端點在 2014 年 9 月 10 日之後建立時，才能透過 SSL 串流。如果您的串流 URL 是根據 9 月 10 日之後建立的串流端點，則 URL 會包含 "streaming.mediaservices.windows.net" (新格式)。包含 "origin.mediaservices.windows.net" (舊格式) 的串流 URL 不支援 SSL。如果您的 URL 是舊格式，而且您希望能夠透過 SSL 串流，請建立新的串流端點。使用根據新的串流端點建立的 URL，透過 SSL 串流處理內容。

下列清單說明不同的串流格式，並提供範例：

- Smooth Streaming

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest


- MPEG DASH

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=mpd-time-csf)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)



- Apple HTTP 即時資料流 (HLS) V4

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=m3u8-aapl)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)



- Apple HTTP 即時資料流 (HLS) V3

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=m3u8-aapl-v3)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

- HDS (僅適用於 Adobe PrimeTime/Access 使用人)

{串流端點名稱-媒體服務帳戶名稱}.streaming.mediaservices.windows.net/{定位器識別碼}/{檔案名稱}.ism/Manifest(format=f4m-f4f)

http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=f4m-f4f)


##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0921_2016-->