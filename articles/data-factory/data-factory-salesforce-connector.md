<properties
	pageTitle="使用 Data Factory 從 Salesforce 移動資料 | Microsoft Azure"
	description="了解如何使用 Azure Data Factory 從 Salesforce 移動資料。"
	services="data-factory"
	documentationCenter=""
	authors="spelluru"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="data-factory"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/07/2016"
	ms.author="spelluru"/>

# 使用 Azure Data Factory 從 Salesforce 移動資料
本文概述如何在 Azure Data Factory 使用複製活動，將資料從 Salesforce 複製到[支援的來源與接收](data-factory-data-movement-activities.md#supported-data-stores)資料表的 [接收] 欄底下列出的任何資料存放區。本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援的資料存放區組合。

Azure Data Factory 目前只支援將資料從 Salesforce 移動到 [支援的接收資料存放區]((data-factory-data-movement-activities.md#supported-data-stores)，但不支援將資料從其他資料存放區移動到 Salesforce。

## 必要條件
- 您必須使用下列其中一個 Salesforce 版本︰Developer Edition、Professional Edition、Enterprise Edition 或 Unlimited Edition。
- 必須啟用 API 權限。請參閱[如何在 Salesforce 中透過權限集啟用 API 存取權？](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/)
- 若要將資料從 Salesforce 複製到內部部署資料存放區，您必須在內部部署環境中至少安裝資料管理閘道 2.0。

## 複製資料精靈
若要建立管線以將資料從 Salesforce 複製到任何支援的接收資料存放區，最簡單的方式是使用複製資料精靈。如需使用複製資料精靈建立管線的快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。這些範例示範如何將資料從 Salesforce 複製到 Azure Blob 儲存體。不過，您可以在 Azure Data Factory 中使用複製活動，將資料複製到[這裡](data-factory-data-movement-activities.md#supported-data-stores)所說的任何接收器。

## 範例：從 Salesforce 複製資料到 Azure Blob
此範例會每隔一小時就把 Salesforce 的資料複製到 Azure Blob。範例後面的各節會說明這些範例中使用的 JSON 屬性。您可以使用 Azure Data Factory 中的複製活動，把資料直接複製到[資料移動活動](data-factory-data-movement-activities.md#supported-data-stores)一文中所述的任何接收器。

以下是為了實作案例所必須建立的 Data Factory 構件。清單後面的各節會提供有關這些步驟的詳細資料。

- [Salesforce](#salesforce-linked-service-properties) 類型的連結服務
- [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties) 類型的連結服務
- [RelationalTable](#salesforce-dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)
- [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)
- 具有使用 [RelationalSource](#relationalsource-type-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)

**Salesforce 連結服務**

此範例使用 **Salesforce** 連結服務。如需此連結服務所支援的屬性，請參閱 [Salesforce 連結服務](#salesforce-linked-service-properties)一節。如需如何重設/取得安全性權杖的指示，請參閱[取得安全性權杖](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm)。

	{
		"name": "SalesforceLinkedService",
		"properties":
		{
			"type": "Salesforce",
			"typeProperties":
			{
				"username": "<user name>",
				"password": "<password>",
				"securityToken": "<security token>"
			}
		}
	}

**Azure 儲存體連結服務**

	{
	  "name": "AzureStorageLinkedService",
	  "properties": {
	    "type": "AzureStorage",
	    "typeProperties": {
	      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
	    }
	  }
	}

**Salesforce 輸入資料集**

	{
		"name": "SalesforceInput",
		"properties": {
			"linkedServiceName": "SalesforceLinkedService",
			"type": "RelationalTable",
			"typeProperties": {
				"tableName": "AllDataType__c"  
			},
			"availability": {
				"frequency": "Hour",
				"interval": 1
			},
			"external": true,
			"policy": {
				"externalData": {
					"retryInterval": "00:01:00",
					"retryTimeout": "00:10:00",
					"maximumRetry": 3
				}
			}
		}
	}

將 **external** 設定為 **true** 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

> [AZURE.IMPORTANT] 任何自訂物件都需要 API 名稱的「\_\_c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

**Azure Blob 輸出資料集**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。

	{
	    "name": "AzureBlobOutput",
	    "properties":
	    {
	        "type": "AzureBlob",
	        "linkedServiceName": "AzureStorageLinkedService",
	        "typeProperties":
	        {
	            "folderPath": "adfgetstarted/alltypes_c"
	        },
	        "availability":
	        {
	            "frequency": "Hour",
	            "interval": 1
	        }
	    }
	}


**具有複製活動的管線**

此管線包含複製活動，該活動已設定為使用上述輸入和輸出資料集並排定為每小時執行。在管線 JSON 定義中，已將 **source** 類型設為 **RelationalSource**，並將 **sink** 類型設為 **BlobSink**。

如需 RelationalSource 支援的屬性清單，請參閱 [RelationalSource 類型屬性](#relationalsource-type-properties)。

	{  
		"name":"SamplePipeline",
		"properties":{  
			"start":"2016-06-01T18:00:00",
			"end":"2016-06-01T19:00:00",
			"description":"pipeline with copy activity",
			"activities":[  
			{
				"name": "SalesforceToAzureBlob",
				"description": "Copy from Salesforce to an Azure blob",
				"type": "Copy",
				"inputs": [
				{
					"name": "SalesforceInput"
				}
				],
				"outputs": [
				{
					"name": "AzureBlobOutput"
				}
				],
				"typeProperties": {
					"source": {
						"type": "RelationalSource",
						"query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"				
					},
					"sink": {
						"type": "BlobSink"
					}
				},
				"scheduler": {
					"frequency": "Hour",
					"interval": 1
				},
				"policy": {
					"concurrency": 1,
					"executionPriorityOrder": "OldestFirst",
					"retry": 0,
					"timeout": "01:00:00"
				}
			}
			]
		}
	}

> [AZURE.IMPORTANT] 任何自訂物件都需要 API 名稱的「\_\_c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## Salesforce 連結服務屬性

下表提供 Salesforce 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| 類型 | 類型屬性必須設為：**Salesforce**。 | 是 |
| username |指定使用者帳戶的使用者名稱。 | 是 |
| password | 指定使用者帳戶的密碼。 | 是 |
| securityToken | 指定使用者帳戶的安全性權杖。如需如何重設/取得安全性權杖的指示，請參閱[取得安全性權杖](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm)。若要整體了解安全性權杖，請參閱[安全性和 API](https://developer.salesforce.com/docs/atlas.zh-TW.api.meta/api/sforce_api_concepts_security.htm)。 | 是 |

## Salesforce 資料集屬性

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。**RelationalTable** 類型資料集的 typeProperties 區段有下列屬性：

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| tableName | Salesforce 中資料表的名稱。 | 否 (如果已指定 **RelationalSource** 的 **query**) |

> [AZURE.IMPORTANT]  任何自訂物件都需要 API 名稱的「\_\_c」部分。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

## RelationalSource 類型屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。名稱、描述、輸入和輸出資料表以及各種原則等屬性都適用於所有活動類型。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同，而在複製活動的案例中，可用的屬性會根據來源與接收的類型而有所不同。

在複製活動的案例中，如果來源類型為 **RelationalSource** (包含 Salesforce)，則 typeProperties 區段可使用下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| query | 使用自訂查詢來讀取資料。 | SQL-92 查詢或 [Salesforce 物件查詢語言 (SOQL) ](https://developer.salesforce.com/docs/atlas.zh-TW.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm) 查詢。例如：select * from MyTable\_\_c。 | 否 (如果已指定 **dataset** 的 **tableName**) |

> [AZURE.IMPORTANT] 任何自訂物件都需要 API 名稱的 "\_\_c" 部分。<br> 當您指定的查詢在 DateTime 資料行上包含 **where** 子句，請使用 SOQL。例如：$$Text.Format('SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= {0:yyyy-MM-ddTHH:mm:ssZ} AND LastModifiedDate < {1:yyyy-MM-ddTHH:mm:ssZ}', WindowStart, WindowEnd) 或 SQL 查詢，例如 $$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts'{0:yyyy-MM-dd HH:mm:ss}'}} AND LastModifiedDate < {{ts'{1:yyyy-MM-dd HH:mm:ss}'}}', WindowStart, WindowEnd)。

![Data Factory - Salesforce 連線 - API 名稱](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## 從 Salesforce 報表擷取資料
您可以藉由指定 {call "<report name>"} 格式的查詢 (例如 "query": "{call "TestReport"}")，以從 Salesforce 報表擷取資料。

## Salesforce 要求限制
Salesforce 對於 API 要求總數和並行 API 要求均有限制。如需詳細資訊，請參閱 [Salesforce 開發人員限制](http://resources.docs.salesforce.com/200/20/zh-TW/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf)文章的＜API 要求限制＞一節。

如果並行要求數目超過限制，便會進行節流，而且您會看到隨機失敗。如果要求總數超過限制，Salesforce 帳戶將會封鎖 24 小時。在上述兩種情況中，您也可能會收到「REQUEST\_LIMIT\_EXCEEDED」錯誤。


[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

### Salesforce 的類型對應
Salesforce 類型 | 以 .NET 為基礎的類型
--------------- | ---------------
自動編號 | String
核取方塊 | Boolean
貨幣 | 兩倍
日期 | DateTime
日期/時間 | DateTime
電子郵件 | String
識別碼 | String
查閱關聯性 | String
複選挑選清單 | String
Number | 兩倍
百分比 | 兩倍
電話 | String
挑選清單 | String
文字 | String
文字區域 | String
文字區域 (完整) | String
文字區域 (豐富) | String
文字 (加密) | String
URL | String

[AZURE.INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]
[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## 效能和微調  
若要了解 Azure Data Factory 中影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)。

<!---HONumber=AcomDC_0817_2016-->