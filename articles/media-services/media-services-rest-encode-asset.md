<properties 
	pageTitle="如何使用 Media Encoder Standard 為資產編碼 | Microsoft Azure" 
	description="了解如何使用媒體編碼器標準為媒體服務上的媒體內容編碼。程式碼範例會使用 REST API。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/19/2016"
	ms.author="juliako"/>


#如何使用 Media Encoder Standard 為資產編碼


> [AZURE.SELECTOR]
- [.NET](media-services-dotnet-encode-asset.md)
- [REST](media-services-rest-encode-asset.md)
- [入口網站](media-services-portal-encode.md)

##Overview
若要透過網際網路傳遞數位視訊，您必須壓縮媒體。數位視訊檔案十分龐大，而且可能太大而無法透過網際網路傳遞，或是太大而使您客戶的裝置無法正確顯示。編碼是壓縮視訊和音訊，好讓客戶能檢視您的媒體的程序。

編碼工作是媒體服務中最常見的處理作業。您建立編碼工作以將媒體檔案從一種編碼轉換成另一種編碼。編碼時，您可以使用媒體服務內建的編碼器 (媒體編碼器標準)。您也可以使用媒體服務合作夥伴提供的編碼器；第三方編碼器可透過 Azure Marketplace 取得。您可以使用針對您的編碼器定義的預設字串，或使用預設組態檔，指定編碼工作的詳細資料。若要查看可用的預設類型，請參閱[媒體編碼器標準的工作預設](http://msdn.microsoft.com/library/mt269960)。

每個工作可以有一或多個工作，視您想要完成的處理類型而定。透過 REST API，您可以用兩種方式之一建立工作和其相關的工作：

- 工作可以透過 Job 實體上的 Tasks 導覽屬性，或
- 透過 OData 批次處理進行內嵌定義。


建議一律將夾層檔編碼為調適性位元速率 MP4 集，然後使用[動態封裝](media-services-dynamic-packaging-overview.md)將該集合轉換為所要的格式。若要利用動態封裝，您必須先取得至少一個串流端點的隨選串流單位，您打算從中傳遞您的內容。如需詳細資訊，請參閱[如何調整媒體服務](media-services-portal-manage-streaming-endpoints.md)。

如果您的輸出資產是儲存體加密，必須設定資產傳遞原則。如需詳細資訊，請參閱[設定資產傳遞原則](media-services-rest-configure-asset-delivery-policy.md)。


>[AZURE.NOTE]開始參考媒體處理器之前，請確認您擁有正確的媒體處理器識別碼。如需詳細資訊，請參閱[取得媒體處理器](media-services-rest-get-media-processor.md)。

##建立具有單一編碼工作的工作

>[AZURE.NOTE] 使用媒體服務 REST API 時，適用下列考量事項：
>
>在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。如需詳細資訊，請參閱[媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。

>成功連線至 https://media.windows.net 後，您會收到指定另一個媒體服務 URI 的 301 重新導向。您必須依照[使用 REST API 連線至媒體服務](media-services-rest-connect-programmatically.md)所述，對新的 URI 進行後續呼叫。
>
>使用 JSON 並指定在要求中使用 **\_\_metadata** 關鍵字時 (例如，為了參考連結的物件)，您「必須」將 **Accept** 標頭設為 [JSON Verbose 格式](http://www.odata.org/documentation/odata-version-3-0/json-verbose-format/)：Accept: application/json;odata=verbose。

下列範例會示範如何建立和張貼工作，並將一個工作設為在特定的解析度與品質將視訊編碼。透過媒體編碼器標準進行編碼時，您可以使用[這裡](http://msdn.microsoft.com/library/mt269960)指定的工作組態預設。

要求：

POST https://media.windows.net/API/Jobs HTTP/1.1 Content-Type: application/json;odata=verbose Accept: application/json;odata=verbose DataServiceVersion: 3.0 MaxDataServiceVersion: 3.0 x-ms-version: 2.11 Authorization: Bearer <token value> x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 Host: media.windows.net

	
	{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aaab7f15b-3136-4ddf-9962-e9ecb28fb9d2')"}}],  "Tasks" : [{"Configuration" : "H264 Multiple Bitrate 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",  "TaskBody" : "<?xml version="1.0" encoding="utf-8"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"}]}

回應：
	
	HTTP/1.1 201 Created

	. . . 

###設定輸出資產的名稱

下列範例示範如何設定 assetName 屬性：

	{ "TaskBody" : "<?xml version="1.0" encoding="utf-8"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName="CustomOutputAssetName">JobOutputAsset(0)</outputAsset></taskBody>"}

##考量

- TaskBody 屬性必須使用 XML 常值來定義工作所使用的輸入或輸出資產數目。工作主題包含 XML 的 XML 結構描述定義。
- 在 TaskBody 定義中，<inputAsset> 和 <outputAsset> 的每一個內部值必須設定為 JobInputAsset(value) 或 JobOutputAsset(value)。
- 每個工作可以有多個輸出資產。一個 JobOutputAsset(x) 只能使用一次做為工作中的工作輸出。
- 您可以指定 JobInputAsset 或 JobOutputAsset 做為工作的輸入資產。
- 工作不能形成循環。
- 您傳遞至 JobInputAsset 或 JobOutputAsset 的 value 參數代表資產的索引值。實際資產定義在作業實體定義上的 InputMediaAsset 與 OutputMediaAsset 導覽屬性。
- 由於媒體服務建置在 OData v3 之上，因此 InputMediaAsset 與 OutputMediaAsset 導覽屬性集合中的個別資產會透過 "\_\_metadata : uri" 名稱 / 值組參考。.
- InputMediaAsset 對應至您在媒體服務中建立的一個或多個資產。OutputMediaAsset 由系統建立。它們不會參考現有的資產。
- OutputMediaAsset 可以使用 assetName 屬性命名。如果這個屬性不存在，則 OutputMediaAsset 的名稱將是 <outputAsset> 元素的任何內部文字值，而且尾碼為工作名稱值或工作識別碼值 (在未定義 Name 屬性的情況下)。例如，如果您將 assetName 的值設為 "Sample"，則 OutputMediaAsset Name 屬性會設為 "Sample"。不過，如果您未設定 assetName 的值，但已將工作名稱設為 "NewJob"，則 OutputMediaAsset Name 會是 "JobOutputAsset(value)\_NewJob"。


##建立具有鏈結工作的工作

在許多應用程式案例中，開發人員想要建立一連串的處理工作。在媒體服務中，您可以建立一連串的鏈結工作。每一項工作會執行不同的處理步驟，而且可以使用不同的媒體處理器。鏈結工作可以將資產從一個工作遞交到另一個工作，對資產執行一連串的工作。不過，在工作中執行的工作不必按照順序。當您建立鏈結工作時，鏈結的 **ITask** 物件會建立在單一 **IJob** 物件中。

>[AZURE.NOTE] 目前有每個工作裡 30 個工作的限制。如果您需要鏈結超過 30 個工作，請建立多個工作來包含這些工作。


	POST https://media.windows.net/api/Jobs HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

	{  
	   "Name":"NewTestJob",
	   "InputMediaAssets":[  
	      {  
	         "__metadata":{  
	            "uri":"https://testrest.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A910ffdc1-2e25-4b17-8a42-61ffd4b8914c')"
	         }
	      }
	   ],
	   "Tasks":[  
	      {  
	         "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
	         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	         "TaskBody":"<?xml version="1.0" encoding="utf-8"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
	      },
	      {  
	         "Configuration":"H264 Smooth Streaming 720p",
	         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	         "TaskBody":"<?xml version="1.0" encoding="utf-16"?><taskBody><inputAsset>JobOutputAsset(0)</inputAsset><outputAsset>JobOutputAsset(1)</outputAsset></taskBody>"
	      }
	   ]
	}


###考量

啟用工作鏈結：

- 工作必須有至少 2 個工作
- 必須有至少一個工作的輸入是工作中另一項工作的輸出。

## 使用 OData 批次處理 

以下範例示範如何使用 OData 批次處理來建立工作和作業。如需批次處理的資訊，請參閱[開放式資料通訊協定 (OData) 批次處理](http://www.odata.org/documentation/odata-version-3-0/batch-processing/)。
 
	POST https://media.windows.net/api/$batch HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Content-Type: multipart/mixed; boundary=batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
	Accept: multipart/mixed
	Accept-Charset: UTF-8
	Authorization: Bearer <token>
	x-ms-version: 2.11
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	Host: media.windows.net
	
	
	--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
	Content-Type: multipart/mixed; boundary=changeset_122fb0a4-cd80-4958-820f-346309967e4d
	
	--changeset_122fb0a4-cd80-4958-820f-346309967e4d
	Content-Type: application/http
	Content-Transfer-Encoding: binary
	
	POST https://media.windows.net/api/Jobs HTTP/1.1
	Content-ID: 1
	Content-Type: application/json
	Accept: application/json
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	Accept-Charset: UTF-8
	Authorization: Bearer <token>
	x-ms-version: 2.11
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	
	{"Name" : "NewTestJob", "InputMediaAssets@odata.bind":["https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A2a22445d-1500-80c6-4b34-f1e5190d33c6')"]}
	
	--changeset_122fb0a4-cd80-4958-820f-346309967e4d
	Content-Type: application/http
	Content-Transfer-Encoding: binary
	
	POST https://media.windows.net/api/$1/Tasks HTTP/1.1
	Content-ID: 2
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	Accept-Charset: UTF-8
	Authorization: Bearer <token>
	x-ms-version: 2.11
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	
	{  
	   "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
	   "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	   "TaskBody":"<?xml version="1.0" encoding="utf-8"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName="Custom output name">JobOutputAsset(0)</outputAsset></taskBody>"
	}

	--changeset_122fb0a4-cd80-4958-820f-346309967e4d--
	--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e--
 


## 使用 JobTemplate 建立工作


使用一組常用工作處理多個資產時，JobTemplates 非常實用，因為它可以指定預設的工作預設項目、工作順序等等。

下列範例說明如何使用內嵌定義的 TaskTemplate 建立 JobTemplate。TaskTemplate 使用媒體編碼器標準作為 MediaProcessor 來編碼資產檔案；但是，也可以使用其他 MediaProcessor。


	POST https://media.windows.net/API/JobTemplates HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	Host: media.windows.net

	
	{"Name" : "NewJobTemplate25", "JobTemplateBody" : "<?xml version="1.0" encoding="utf-8"?><jobTemplate><taskBody taskTemplateId="nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789"><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody></jobTemplate>", "TaskTemplates" : [{"Id" : "nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789", "Configuration" : "H264 Smooth Streaming 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56", "Name" : "SampleTaskTemplate2", "NumberofInputAssets" : 1, "NumberofOutputAssets" : 1}] }
 

>[AZURE.NOTE]與其他媒體服務實體不同的是，您必須為每個 TaskTemplate 定義新的 GUID 識別碼，並將它置入 taskTemplateId，以及在您的要求本文中置入識別碼屬性。內容識別配置必須遵循「識別 Azure 媒體服務實體」中所述的配置。而且，不能更新 JobTemplate。相反地，您必須使用更新後的變更建立一個新的 JobTemplate。
 

如果成功，則會傳回下列回應：
	
	HTTP/1.1 201 Created
	
	. . .


下面的範例說明如何建立參照 JobTemplate 識別碼的工作：

	POST https://media.windows.net/API/Jobs HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	Host: media.windows.net

	
	{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A3f1fe4a2-68f5-4190-9557-cd45beccef92')"}}], "TemplateId" : "nb:jtid:UUID:15e6e5e6-ac85-084e-9dc2-db3645fbf0aa"}
	 

如果成功，則會傳回下列回應：
	
	HTTP/1.1 201 Created
	
	. . . 



##媒體服務學習路徑

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##後續步驟
您已了解如何建立為資產編碼的作業，現在請移至[如何使用媒體服務檢查作業進度](media-services-rest-check-job-progress.md) (英文) 主題。


##另請參閱

[取得媒體處理器](media-services-rest-get-media-processor.md)

<!---HONumber=AcomDC_0921_2016-->