<properties 
	pageTitle="使用 REST 將檔案上傳至媒體服務帳戶 | Microsoft Azure" 
	description="了解如何建立並上傳資產，以將媒體內容移至媒體服務中。" 
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


# 使用 REST 將檔案上傳至媒體服務帳戶

 > [AZURE.SELECTOR]
 - [.NET](media-services-dotnet-upload-files.md)
 - [REST](media-services-rest-upload-files.md)
 - [入口網站](media-services-portal-upload-files.md)

在媒體服務中，您會將數位檔案上傳到到資產。[資產](https://msdn.microsoft.com/library/azure/hh974277.aspx)實體可以包含視訊、音訊、影像、縮圖集合、文字播放軌及隱藏式字幕檔案 (以及這些檔案的相關中繼資料)。 一旦檔案會上傳到資產，您的內容會安全地儲存在雲端，以便進行進一步的處理和串流。

>[AZURE.NOTE]當您選擇資產檔案名稱時適用下列考量︰
>
>- 媒體服務在建置串流內容的 URL 時使用 IAssetFile.Name 屬性的值 (例如，http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters.) 基於這個理由，不允許 percent-encoding。**Name** 屬性的值不能有下列任何 [percent-encoding-reserved 字元](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!*'();:@&=+$,/?%#"。而且，副檔名只能有一個 '.'。
>
>- 名稱長度不應超過 260 個字元。

上傳資產的基本工作流程分成下列各節：

- 建立資產
- 加密資產 (選擇性)
- 將檔案上傳至 blob 儲存體

AMS 也可讓您上傳大量資產。如需詳細資訊，請參閱[本節](media-services-rest-upload-files.md#upload_in_bulk)。

##上傳資產

###建立資產

>[AZURE.NOTE] 使用媒體服務 REST API 時，適用下列考量事項：
>
>在媒體服務中存取實體時，您必須在 HTTP 要求中設定特定的標頭欄位和值。如需詳細資訊，請參閱[媒體服務 REST API 開發設定](media-services-rest-how-to-use.md)。

>成功連線至 https://media.windows.net 後，您會收到指定另一個媒體服務 URI 的 301 重新導向。您必須依照[使用 REST API 連線至媒體服務](media-services-rest-connect-programmatically.md)所述，對新的 URI 進行後續呼叫。
 
資產是媒體服務中多種類型或物件集的容器，包括視訊、音訊、影像、縮圖集合、文字播放軌和隱藏式字幕檔案。在 REST API 中，建立資產必須傳送 POST 要求給媒體服務，並將關於您資產的任何屬性資訊放在要求主體中。

您可以在建立資產時指定的其中一個屬性是 **Options**。**Options** 是列舉值，描述可用來建立資產的加密選項。有效的值是以下清單的其中一個值，而不是值的組合。

- **None** = **0**：將不使用加密。這是預設值。請注意，使用這個選項時您的內容在傳輸中或在儲存體中不受保護。
	如果您計劃使用漸進式下載傳遞 MP4，請使用此選項。 

- **StorageEncrypted** = **1**：如果要用 AES-256 位元加密來加密您的檔案，以便進行上傳和儲存，請指定此值。

	如果您的資產是儲存體加密，必須設定資產傳遞原則。如需詳細資訊，請參閱[設定資產傳遞原則](media-services-rest-configure-asset-delivery-policy.md)。

- **CommonEncryptionProtected** = **2**：如果上傳使用一般加密方法 (例如 PlayReady) 保護的檔案，請指定此值。

- **EnvelopeEncryptionProtected** = **4**：如果上傳使用 AES 檔案加密的 HLS，請指定此值。請注意，檔案必須已由 Transform Manager 編碼和加密。

>[AZURE.NOTE]如果您的資產會使用加密，您必須建立 **ContentKey** 並將它連結到您的資產，如下列主題中所述：[如何建立 ContentKey](media-services-rest-create-contentkey.md)。請注意，檔案上傳到資產之後，您必須將 **AssetFile** 實體上的加密屬性更新為在**資產**加密期間得到的值。請使用 **MERGE** HTTP 要求執行此作業。


下列範例示範如何建立資產。

**HTTP 要求**

	POST https://media.windows.net/api/Assets HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.windows.net
	
	{"Name":"BigBuckBunny.mp4"}
	

**HTTP 回應**

如果成功，則會傳回下列內容：
	
	HTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 452
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
	request-id: e98be122-ae09-473a-8072-0ccd234a0657
	x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:06:40 GMT
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
	   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "State":0,
	   "Created":"2015-01-18T22:06:40.6010903Z",
	   "LastModified":"2015-01-18T22:06:40.6010903Z",
	   "AlternateId":null,
	   "Name":"BigBuckBunny.mp4",
	   "Options":0,
	   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StorageAccountName":"storagetestaccount001"
	}
	
###建立 AssetFile

[AssetFile](http://msdn.microsoft.com/library/azure/hh974275.aspx) 實體代表儲存在 blob 容器中的視訊或音訊檔案。資產檔案一律會與資產相關聯，而資產可包含一或多個資產檔案。如果資產檔案物件並未與 blob 容器中的數位檔案相關聯，媒體服務編碼器工作將會失敗。

請注意，**AssetFile** 執行個體和實際的媒體檔案是兩個不同的物件。AssetFile 執行個體包含媒體檔案的相關中繼資料，而媒體檔案包含實際的媒體內容。

當您將數位媒體檔案上傳至 blob 容器之後，您將使用 **MERGE** HTTP 要求，以媒體檔案的相關資訊來更新 AssetFile (如本主題稍後所示)。

**HTTP 要求**

	POST https://media.windows.net/api/Files HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.windows.net
	Content-Length: 164
	
	{  
	   "IsEncrypted":"false",
	   "IsPrimary":"false",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}


**HTTP 回應**

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 535
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
	Server: Microsoft-IIS/8.5
	request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
	x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 00:34:07 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
	   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
	   "Name":"BigBuckBunny.mp4",
	   "ContentFileSize":"0",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "EncryptionVersion":null,
	   "EncryptionScheme":null,
	   "IsEncrypted":false,
	   "EncryptionKeyId":null,
	   "InitializationVector":null,
	   "IsPrimary":false,
	   "LastModified":"2015-01-19T00:34:08.1934137Z",
	   "Created":"2015-01-19T00:34:08.1934137Z",
	   "MimeType":"video/mp4",
	   "ContentChecksum":null
	}


### 建立具有寫入權限的 AccessPolicy。 

將任何檔案上傳到 blob 儲存體之前，請設定寫入資產的存取原則權限。若要這樣做，請 POST HTTP 要求到 AccessPolicies 實體集。請在建立時定義 DurationInMinutes 值，否則您會在回應中收到 500 內部伺服器錯誤訊息。如需 AccessPolicies 的詳細資訊，請參閱 [AccessPolicy](http://msdn.microsoft.com/library/azure/hh974297.aspx)。

下列範例示範如何建立 AccessPolicy：
		
**HTTP 要求**

	POST https://media.windows.net/api/AccessPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.windows.net
	
	{"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"} 

**HTTP 要求**

	If successful, the following response is returned:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 312
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
	Server: Microsoft-IIS/8.5
	request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
	x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:18:06 GMT

	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AccessPolicies/@Element",
	   "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "Created":"2015-01-18T22:18:06.6370575Z",
	   "LastModified":"2015-01-18T22:18:06.6370575Z",
	   "Name":"NewUploadPolicy",
	   "DurationInMinutes":440.0,
	   "Permissions":2
	}

###取得上傳 URL

若要接收實際的上傳 URL，請建立 SAS 定位器。定位器為想要存取資產中之檔案的用戶端定義連線端點的開始時間和類型。您可以為指定的 AccessPolicy 與 Asset 配對建立多個 Locator 實體，以處理不同的用戶端要求與需求。這些 Locator 每個都會使用 StartTime 值加上 AccessPolicy 的 DurationInMinutes 值，以判斷可以使用 URL 的時間長度。如需詳細資訊，請參閱＜定位器＞[](http://msdn.microsoft.com/library/azure/hh974308.aspx)。


SAS URL 具有下列格式：

	{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}

適用一些考量事項：

- 您一次不能有超過五個唯一定位器與指定的資產相關聯。如需詳細資訊，請參閱＜定位器＞。
- 如果您需要立即上傳檔案，您應該將 StartTime 值設為目前時間的五分鐘前。這是因為用戶端電腦與媒體服務之間可能有時間差。此外，您的 StartTime 值必須是以下日期時間格式：YYYY-MM-DDTHH:mm:ssZ (例如，"2014-05-23T17:53:50Z")。
- 建立 Locator 之後到它可供使用時，中間可能會有 30 到 40 秒的延遲。此問題同時適用於 SAS URL 與原始定位器。

下列範例會示範如何建立 SAS URL 定位器，如要求主體中的 Type 屬性所定義 ("1" 代表 SAS 定位器，"2" 代表隨選原始定位器)。傳回的 **Path** 屬性包含上傳檔案必須使用的 URL。
	
**HTTP 要求**
	
	POST https://media.windows.net/api/Locators HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.windows.net
	{  
	   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StartTime":"2015-02-18T16:45:53",
	   "Type":1
	}


**HTTP 回應**

如果成功，則會傳回下列回應：
		
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 949
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
	Server: Microsoft-IIS/8.5
	request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
	x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 03:01:29 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Locators/@Element",
	   "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
	   "ExpirationDateTime":"2015-02-19T00:05:53",
	   "Type":1,
	   "Path":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
	   "BaseUri":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247",
	   "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
	   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StartTime":"2015-02-18T16:45:53",
	   "Name":null
	}

### 將檔案上傳至 blob 儲存體容器
	
一旦設定 AccessPolicy 與 Locator，實際檔案會使用 Azure 儲存體 REST API 上傳至 Azure Blob 儲存容器。您可以使用頁面或區塊 blob 上傳。

>[AZURE.NOTE] 您必須將要上傳的檔案名稱新增到上一節中所收到的 Locator **Path** 值。例如，https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4? . . .

如需使用 Azure 儲存體 blob 的詳細資訊，請參閱 [Blob 服務 REST API](http://msdn.microsoft.com/library/azure/dd135733.aspx)。


### 更新 AssetFile 

現在，您已上傳您的檔案，請更新 FileAsset 大小 (及其他) 資訊。例如：
	
	MERGE https://media.windows.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.11
	Host: media.windows.net
	
	{  
	   "ContentFileSize":"1186540",
	   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}


**HTTP 回應**

如果成功，會傳回下列訊息：
	HTTP/1.1 204 沒有內容

### 刪除 Locator 和 AccessPolicy 

**HTTP 要求**


	DELETE https://media.windows.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.11
	Host: media.windows.net

	
**HTTP 回應**

如果成功，則會傳回下列內容：

	HTTP/1.1 204 No Content 
	...

**HTTP 要求**

	DELETE https://media.windows.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.11
	Host: media.windows.net

**HTTP 回應**

如果成功，則會傳回下列內容：

	HTTP/1.1 204 No Content 
	...

##<a id="upload_in_bulk"></a>上傳大量資產

###建立 IngestManifest

IngestManifest 是適用於一組資產、資產檔案及統計資訊的容器，可用來判斷針對該組合進行大量內嵌的進度。


**HTTP 要求**

	POST https:// media.windows.net/API/IngestManifests HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net
	Content-Length: 36
	Expect: 100-continue
	
	{ "Name" : "ExampleManifestREST" }

###建立資產

建立 IngestManifestAsset 之前，您必須建立將使用大量內嵌完成的資產。資產是媒體服務中多種類型或物件集的容器，包括視訊、音訊、影像、縮圖集合、文字播放軌和隱藏式字幕檔案。在 REST API 中，建立資產需要將 HTTP POST 要求傳送至 Microsoft Azure 媒體服務，並在要求本文中放置關於資產的任何屬性資訊。在此範例中，資產是使用要求本文包含的 StorageEncrption(1) 選項所建立。

**HTTP 回應**

	POST https://media.windows.net/API/Assets HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net
	Content-Length: 55
	Expect: 100-continue
	
	{ "Name" : "ExampleManifestREST_Asset", "Options" : 1 }

###建立 IngestManifestAssets

IngestManifestAssets 代表 IngestManifest 中配合大量內嵌使用的資產。基本上是將資產連結到資訊清單。Azure 媒體服務會根據與 IngestManifestAsset 相關聯的 Ingestmanifestfile 集合，觀察內部的檔案上傳。將這些檔案上傳之後，資產便已完成。您可以使用 HTTP POST 要求來建立新的 IngestManifestAsset。在要求本文中，包含 IngestManifest 識別碼和資產識別碼，IngestManifestAsset 應該會將它們連結在一起以進行大量內嵌。

**HTTP 回應**

	POST https://media.windows.net/API/IngestManifestAssets HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net
	Content-Length: 152
	Expect: 100-continue
	{ "ParentIngestManifestId" : "nb:mid:UUID:5c77f186-414f-8b48-8231-17f9264e2048", "Asset" : { "Id" : "nb:cid:UUID:b757929a-5a57-430b-b33e-c05c6cbef02e"}}


###為每個資產建立 IngestManifestFile

IngestManifestFile 代表實際的視訊或音訊 Blob 物件，將針對資產上傳此物件以做為大量內嵌的一部分。除非資產正在使用加密選項，否則不需與加密相關的屬性。本節使用的範例示範如何建立 IngestManifestFile，針對先前建立的資產使用 StorageEncryption。


**HTTP 回應**

	POST https://media.windows.net/API/IngestManifestFiles HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net
	Content-Length: 367
	Expect: 100-continue
	
	{ "Name" : "REST_Example_File.wmv", "ParentIngestManifestId" : "nb:mid:UUID:5c77f186-414f-8b48-8231-17f9264e2048", "ParentIngestManifestAssetId" : "nb:maid:UUID:beed8531-9a03-9043-b1d8-6a6d1044cdda", "IsEncrypted" : "true", "EncryptionScheme" : "StorageEncryption", "EncryptionVersion" : "1.0", "EncryptionKeyId" : "nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510" }
	
###將檔案上傳至 Blob 儲存體

您可以使用任何高速用戶端應用程式，此應用程式能夠將資產檔案上傳至 IngestManifest 之 BlobStorageUriForUpload 屬性所提供的 Blob 儲存體容器 URI。一個著名的高速上傳服務是 [Aspera On Demand for Azure Application](http://go.microsoft.com/fwlink/?LinkId=272001)。

###監視大量內嵌進度

您可以藉由輪詢 IngestManifest 的 Statistics 屬性，來監視 IngestManifest 的大量內嵌作業進度。該屬性是複雜類型 [IngestManifestStatistics](https://msdn.microsoft.com/library/azure/jj853027.aspx)。若要輪詢 Statistics 屬性，請送出 HTTP GET 要求以傳遞 IngestManifest 識別碼。
 

##建立要用於加密的 ContentKey

如果您的資產會使用加密功能，就必須在建立資產檔案前，建立要用於加密的 ContentKey。對於儲存體加密，要求本文中應該包含下列屬性。
 
要求本文屬性 | 說明
---|---
識別碼 | 我們使用下列格式自行產生的 ContentKey 識別碼：“nb:kid:UUID:<NEW GUID>”。
ContentKeyType | 這是針對此內容金鑰以整數表示的內容金鑰類型。我們會傳遞值 1 來進行儲存體加密。
EncryptedContentKey | 我們會建立新的內容金鑰值，其為 256 位元 (32 位元組) 的值。此金鑰是藉由針對 GetProtectionKeyId 與 GetProtectionKey 方法執行 HTTP GET 要求，使用我們從 Microsoft Azure 媒體服務擷取的儲存體加密 X.509 憑證來加密的。
ProtectionKeyId | 這是適用於儲存體加密 X.509 憑證的保護金鑰識別碼，可用來加密我們的內容金鑰。
ProtectionKeyType | 這是適用於保護金鑰的加密類型，可用來將內容金鑰加密。針對本文範例，此值為 StorageEncryption(1)。
總和檢查碼 |MD5 會針對內容金鑰計算出總和檢查碼。它是使用內容金鑰來將內容識別碼加密計算而得的。範例程式碼示範如何計算總和檢查碼。


**HTTP 回應**
	
	POST https://media.windows.net/api/ContentKeys HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net
	Content-Length: 572
	Expect: 100-continue
	
	{"Id" : "nb:kid:UUID:316d14d4-b603-4d90-b8db-0fede8aa48f8", "ContentKeyType" : 1, "EncryptedContentKey" : "Y4NPej7heOFa2vsd8ZEOcjjpu/qOq3RJ6GRfxa8CCwtAM83d6J2mKOeQFUmMyVXUSsBCCOdufmieTKi+hOUtNAbyNM4lY4AXI537b9GaY8oSeje0NGU8+QCOuf7jGdRac5B9uIk7WwD76RAJnqyep6U/OdvQV4RLvvZ9w7nO4bY8RHaUaLxC2u4aIRRaZtLu5rm8GKBPy87OzQVXNgnLM01I8s3Z4wJ3i7jXqkknDy4VkIyLBSQvIvUzxYHeNdMVWDmS+jPN9ScVmolUwGzH1A23td8UWFHOjTjXHLjNm5Yq+7MIOoaxeMlKPYXRFKofRY8Qh5o5tqvycSAJ9KUqfg==", "ProtectionKeyId" : "7D9BB04D9D0A4A24800CADBFEF232689E048F69C", "ProtectionKeyType" : 1, "Checksum" : "TfXtjCIlq1Y=" }

### 將 ContentKey 連結到資產

您可以藉由傳送 HTTP POST 要求，來將 ContentKey 關聯至一或多個資產。下列要求是依識別碼將範例 ContentKey 連結至範例資產的範例。

**HTTP 回應**
	
	POST https://media.windows.net/API/Assets('nb:cid:UUID:b3023475-09b4-4647-9d6d-6fc242822e68')/$links/ContentKeys HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net
	Content-Length: 113
	Expect: 100-continue
	
	{ "uri": "https://media.windows.net/api/ContentKeys('nb%3Akid%3AUUID%3A32e6efaf-5fba-4538-b115-9d1cefe43510')"}

**HTTP 回應**

	GET https://media.windows.net/API/IngestManifests('nb:mid:UUID:5c77f186-414f-8b48-8231-17f9264e2048') HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.windows.net



##後續步驟

檢閱媒體服務學習路徑。

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##提供意見反應

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]



 
[How to Get a Media Processor]: media-services-get-media-processor.md
 

<!---HONumber=AcomDC_0921_2016-->