<properties
	pageTitle="如何使用 Python 的佇列儲存體 | Microsoft Azure"
	description="了解如何使用 Python 的 Azure 佇列服務來建立和刪除佇列，以及插入、取得和刪除訊息。"
	services="storage"
	documentationCenter="python"
	authors="robinsh"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="python"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="cbrooks;robinsh"/>

# 如何使用 Python 的佇列儲存體

[AZURE.INCLUDE [storage-selector-queue-include](../../includes/storage-selector-queue-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-queues.md)]

## Overview

本指南說明如何使用 Azure 佇列儲存體服務執行一般案例。這些範例是以 Python 所撰寫，並使用 [Microsoft Azure Storage SDK for Python (適用於 Python 的 Microsoft Azure 儲存體 SDK)]。所涵蓋的案例包括「插入」、「查看」、「取得」和「刪除」佇列訊息，以及「建立和刪除佇列」。如需佇列的詳細資訊，請參閱 [後續步驟] 一節。

[AZURE.INCLUDE [storage-queue-concepts-include](../../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 作法：建立佇列

**QueueService** 物件可讓您操作佇列。下列程式碼將建立 **QueueService** 物件。將下列內容新增至您想要在其中以程式設計方式存取 Azure 儲存體之任何 Python 檔案內的頂端附近：

	from azure.storage.queue import QueueService

下列程式碼會使用儲存體帳戶名稱和帳戶金鑰來建立 **QueueService** 物件：將 'myaccount' 和 'mykey' 取代為您的帳戶名稱和金鑰。

	queue_service = QueueService(account_name='myaccount', account_key='mykey')

	queue_service.create_queue('taskqueue')


## 作法：將訊息插入佇列中

若要將訊息插入佇列，請使用 **put\_message** 方法建立新訊息，並將它新增至佇列。

	queue_service.put_message('taskqueue', u'Hello World')


## 作法：預覽下一個訊息

透過呼叫 **peek\_messages** 方法，您可以在佇列前面查看訊息，而無需將它從佇列中移除。依預設，**peek\_messages** 查看單一訊息。

	messages = queue_service.peek_messages('taskqueue')
	for message in messages:
		print(message.content)


## 做法：清除佇列訊息

您的程式碼可以使用兩個步驟來將訊息從佇列中移除。呼叫 **get\_messages** 時，依預設，您會取得佇列中的下一個訊息。對於從此佇列讀取訊息的任何其他程式碼而言，將無法看到從 **get\_messages** 傳回的訊息。依預設，此訊息會維持 30 秒的不可見狀態。若要完成從佇列中移除訊息，您還必須呼叫 **delete\_message**。這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。您的程式碼會在處理完訊息之後立即呼叫 **delete\_message**。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		print(message.content)
		queue_service.delete_message('taskqueue', message.id, message.pop_receipt)

自訂從佇列中擷取訊息的方法有兩種。首先，您可以取得一批訊息 (最多 32 個)。其次，您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。下列程式碼範例將使用 **get\_messages** 方法，在一次呼叫中取得 16 個訊息。接著它會使用 for 迴圈處理每個訊息。它也會將可見度逾時設定為每個訊息五分鐘。

	messages = queue_service.get_messages('taskqueue', num_messages=16, visibility_timeout=5*60)
	for message in messages:
		print(message.content)
		queue_service.delete_message('taskqueue', message.id, message.pop_receipt)		


## 作法：變更佇列訊息的內容

您可以在佇列中就地變更訊息內容。如果訊息代表工作作業，則您可以使用此功能來更新工作作業的狀態。以下程式碼使用 **update\_message** 方法來更新訊息。可見度逾時設定為 0，表示會立即顯示訊息，並且會更新內容。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		queue_service.update_message('taskqueue', message.id, message.pop_receipt, 0, u'Hello World Again')

## 作法：取得佇列長度

您可以取得佇列中的估計訊息數目。**get\_queue\_metadata** 方法會要求佇列服務傳回佇列的相關中繼資料和 **approximate\_message\_count**。由於佇列服務在回應您的要求之後可以新增或移除訊息，此結果僅為近似值。

	metadata = queue_service.get_queue_metadata('taskqueue')
	count = metadata.approximate_message_count

## 作法：刪除佇列

若要刪除佇列及其內含的所有訊息，請呼叫 **delete\_queue** 方法。

	queue_service.delete_queue('taskqueue')

## 後續步驟

了解佇列儲存體的基本概念之後，請使用下列連結深入了解。

- [Python 開發人員中心](/develop/python/)
- [Azure 儲存體服務 REST API](http://msdn.microsoft.com/library/azure/dd179355)
- [Azure 儲存體團隊部落格]
- [Microsoft Azure Storage SDK for Python]

[Azure 儲存體團隊部落格]: http://blogs.msdn.com/b/windowsazurestorage/
[Microsoft Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python
[Microsoft Azure Storage SDK for Python (適用於 Python 的 Microsoft Azure 儲存體 SDK)]: https://github.com/Azure/azure-storage-python

<!---HONumber=AcomDC_0921_2016-->