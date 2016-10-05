<properties
	pageTitle="使用 AzCopy 複製或移動資料到儲存體 | Microsoft Azure"
	description="使用 AzCopy 公用程式來從 Blob、資料表和檔案內容移動或來回複製資料。從本機檔案複製資料到 Azure 儲存體，或在儲存體帳戶內或之間複製資料。輕鬆地將資料移轉至 Azure 儲存體。"
	services="storage"
	documentationCenter=""
	authors="micurd"
	manager="jahogg"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/02/2016"
	ms.author="micurd;tamram"/>

# 使用 AzCopy 命令列公用程式傳輸資料

## Overview

AzCopy 是個 Windows 命令列公用程式，專為使用簡單命令高效率地將資料複製到和複製出 Microsoft Azure Blob、檔案和表格儲存體所設計。您可以從儲存體帳戶內或是在儲存體帳戶之間，從一個物件複製資料到另一個物件。

> [AZURE.NOTE] 本指南假設您已熟悉 [Azure 儲存體](https://azure.microsoft.com/services/storage/)。如果不熟悉，閱讀 [Azure 儲存體簡介](storage-introduction.md)說明文件會很有幫助。最重要的是，您需要[建立儲存體帳戶](storage-create-storage-account.md#create-a-storage-account)才能開始使用 AzCopy。

## 下載並安裝 AzCopy

### Windows

下載[最新版本的 AzCopy](http://aka.ms/downloadazcopy)。

### Mac/Linux

AzCopy 不適用於 Mac/Linux OS。不過，Azure CLI 是適合用來將資料複製到和複製出 Azure 儲存體的替代方案。若要深入了解，請閱讀[使用 Azure CLI 搭配 Azure 儲存體](storage-azure-cli.md)。

## 撰寫第一個 AzCopy 命令

AzCopy 命令的基本語法是：

	AzCopy /Source:<source> /Dest:<destination> [Options]

開啟命令視窗，並瀏覽至電腦上的 AzCopy 安裝目錄，也就是 `AzCopy.exe` 可執行檔的位置。若有需要，您可以在您的系統路徑中加入 AzCopy 安裝位置。根據預設，AzCopy 會安裝到 `%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy` 或 `%ProgramFiles%\Microsoft SDKs\Azure\AzCopy`。

下列範例會示範各種不同的 Microsoft Azure Blob、檔案和資料表資料複製案例。如需每個範例中所使用參數的詳細說明，請參閱 [AzCopy 參數](#azcopy-parameters)一節。

## Blob：下載

### 下載單一 Blob

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"

請注意，如果資料夾 `C:\myfolder` 不存在，AzCopy 會加以建立並將 `abc.txt ` 下載到新資料夾。

### 從次要地區下載單一 Blob

	AzCopy /Source:https://myaccount-secondary.blob.core.windows.net/mynewcontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

請注意，您必須啟用讀取權限異地備援儲存體。

### 下載所有 Blob

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /S

假設下列 Blob 位於指定容器中：

	abc.txt
	abc1.txt
	abc2.txt
	vd1\a.txt
	vd1\abcd.txt

下載作業之後，目錄 `C:\myfolder` 將包含下列檔案：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\vd1\a.txt
	C:\myfolder\vd1\abcd.txt

如果您未指定選項 `/S`，則不會下載任何 Blob。

### 下載具有指定首碼的 Blob

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:a /S

假設下列 Blob 位於指定容器中。將會下載所有以首碼 `a` 開頭的 Blob：

	abc.txt
	abc1.txt
	abc2.txt
	xyz.txt
	vd1\a.txt
	vd1\abcd.txt

下載作業之後，資料夾 `C:\myfolder` 將包含下列檔案：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt

首碼會套用到虛擬目錄，虛擬目錄會構成第一部分的 Blob 名稱。在上述範例中，虛擬目錄不符合指定的首碼，所以不會被下載。此外，如果未指定選項 `\S`，則 AzCopy 不會下載任何 Blob。

### 將匯出檔案的最後修改時間設定為與來源 Blob 相同的最後修改時間

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT

您也可以根據其最後修改時間，將 Blob 從下載作業中排除。例如，如果您想要排除最後修改時間比目的地檔案還要新或相同的 Blob，請新增 `/XN` 選項：

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XN

或者，如果您想要排除最後修改時間比目的地檔案還要舊或相同的 Blob，請新增 `/XO` 選項：

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XO

## Blob：上傳

### 上傳單一檔案

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Pattern:"abc.txt"

如果指定的目的地容器不存在，則 AzCopy 將建立此容器並將檔案上傳至該容器中。

### 上傳單一檔案到虛擬目錄

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer/vd /DestKey:key /Pattern:abc.txt

如果指定的虛擬目錄不存在，則 AzCopy 將上傳檔案並在其名稱中加上此虛擬目錄 (*例如*，上述範例中的 `vd/abc.txt`)。

### 上傳所有檔案

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /S

指定 `/S` 選項以遞迴方式將指定目錄的內容上傳到 Blob 儲存體，這表示也會上傳所有的子資料夾及其檔案。例如，假設下列檔案位於 `C:\myfolder` 資料夾內：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

上傳作業之後，容器將包含下列檔案：

  	abc.txt
	abc1.txt
	abc2.txt
	subfolder\a.txt
	subfolder\abcd.txt

如果您未指定選項 `/S`，AzCopy 將不會以遞迴方式上傳。上傳作業之後，容器將包含下列檔案：

	abc.txt
	abc1.txt
	abc2.txt

### 上傳符合指定模式的檔案

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Pattern:a* /S

假設下列檔案位於 `C:\myfolder` 資料夾內：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\xyz.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

上傳作業之後，容器將包含下列檔案：

	abc.txt
	abc1.txt
	abc2.txt
	subfolder\a.txt
	subfolder\abcd.txt

如果您未指定選項 `/S`，AzCopy 將只會上傳沒有位於虛擬目錄的 Blob：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt

### 指定目的地 blob 的 MIME 內容類型

根據預設，AzCopy 會將目的地 blob 的內容類型設定為 `application/octet-stream`。從 3.1.0 版開始，您可以透過 `/SetContentType:[content-type]` 選項明確指定內容類型。此語法會在上傳作業中設定所有 Blob 的內容類型。

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.windows.net/myContainer/ /DestKey:key /Pattern:ab /SetContentType:video/mp4

如果您指定 `/SetContentType` 但未指定任何值，則 AzCopy 會根據副檔名來設定每個 blob 或檔案的內容類型。

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.windows.net/myContainer/ /DestKey:key /Pattern:ab /SetContentType

## Blob：複製

### 複製儲存體帳戶內的單一 Blob

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1 /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /SourceKey:key /DestKey:key /Pattern:abc.txt

當您複製儲存體帳戶內的 Blob 時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### 跨儲存體帳戶複製單一 Blob

	AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

當您跨儲存體帳戶複製 Blob 時，系統會執行[伺服器端複製](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)作業。

### 將單一 Blob 從次要地區複製到主要區域

	AzCopy /Source:https://myaccount1-secondary.blob.core.windows.net/mynewcontainer1 /Dest:https://myaccount2.blob.core.windows.net/mynewcontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

請注意，您必須啟用讀取權限異地備援儲存體。

### 跨儲存體帳戶複製單一 Blob 及其快照

	AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt /Snapshot

複製作業之後，目標容器將包含 Blob 及其快照。假設上述範例中的 Blob 有兩份快照，則容器將包含下列 Blob 及快照：

	abc.txt
	abc (2013-02-25 080757).txt
	abc (2014-02-21 150331).txt

### 以同步方式跨儲存體帳戶複製 Blob

根據預設，AzCopy 會以非同步方式在兩個儲存體端點之間複製資料。因此，複製作業會在背景中執行，並使用在 blob 複製速度方面沒有 SLA 的備用頻寬容量，而 AzCopy 會定期檢查複製狀態，直到複製完成或失敗為止。

`/SyncCopy` 選項可確保複製作業達到一致的速度。AzCopy 執行同步複製的方式是先將要複製的 blob，從指定的來源下載到本機記憶體，再上傳至 Blob 儲存體目的地。

	AzCopy /Source:https://myaccount1.blob.core.windows.net/myContainer/ /Dest:https://myaccount2.blob.core.windows.net/myContainer/ /SourceKey:key1 /DestKey:key2 /Pattern:ab /SyncCopy

相較於非同步複製，`/SyncCopy` 可能會產生額外的輸出成本，建議的方法是在與來源儲存體帳戶位於同一區域的 Azure VM 中使用這個選項，以避免產生輸出成本。

## 檔案：下載

### 下載單一檔案

	AzCopy /Source:https://myaccount.file.core.windows.net/myfileshare/myfolder1/ /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

如果指定的來源是 Azure 檔案共用，則您必須指定確切檔案名稱 (*例如* `abc.txt`) 以下載單一檔案，或指定 `/S` 選項以遞迴方式下載共用中的所有檔案。嘗試同時指定檔案模式和 `/S` 選項將會造成錯誤。

### 下載所有檔案

	AzCopy /Source:https://myaccount.file.core.windows.net/myfileshare/ /Dest:C:\myfolder /SourceKey:key /S

請注意，將不會下載任何空白資料夾。

## 檔案：上傳

### 上傳單一檔案

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /Pattern:abc.txt

### 上傳所有檔案

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /S

請注意，將不會上傳任何空白資料夾。

### 上傳符合指定模式的檔案

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.windows.net/myfileshare/ /DestKey:key /Pattern:ab* /S

## 檔案：複製

### 跨檔案共用複製

	AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare1/ /Dest:https://myaccount2.file.core.windows.net/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S

### 從檔案共用複製到 Blob

	AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare/ /Dest:https://myaccount2.blob.core.windows.net/mycontainer/ /SourceKey:key1 /DestKey:key2 /S

請注意，不支援從檔案儲存體非同步複製到分頁 Blob。

### 從 Blob 複製到檔案共用

	AzCopy /Source:https://myaccount1.blob.core.windows.net/mycontainer/ /Dest:https://myaccount2.file.core.windows.net/myfileshare/ /SourceKey:key1 /DestKey:key2 /S

### 以同步方式複製檔案

您可以指定 `/SyncCopy` 選項，以同步方式從檔案儲存體複製資料到檔案儲存體、從檔案儲存體複製資料到 Blob 儲存體，以及從 Blob 儲存體複製資料到檔案儲存體，AzCopy 會將來源資料下載至本機記憶體，並重新上傳到目的地，來執行此項作業。

	AzCopy /Source:https://myaccount1.file.core.windows.net/myfileshare1/ /Dest:https://myaccount2.file.core.windows.net/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy

從檔案儲存體複製到 Blob 儲存體時，預設的 Blob 類型是區塊 Blob，使用者可以指定 `/BlobType:page` 選項來變更目的地 Blob 類型。

請注意，相較於非同步複製，`/SyncCopy` 可能會產生額外的輸出成本，建議的方法是在與來源儲存體帳戶位於同一區域的 Azure VM 中使用這個選項，以避免產生輸出成本。

## 資料表：匯出

### 匯出資料表

	AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key

AzCopy 會將資訊清單檔寫入至指定的目的地資料夾。匯入程序會使用資訊清單檔案來尋找必要的資料檔案，並執行資料驗證。資訊清單檔預設會使用下列命令慣例：

	<account name>_<table name>_<timestamp>.manifest

使用者也可以指定 `/Manifest:<manifest file name>` 選項，設定資訊清單檔案名稱。

	AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /Manifest:abc.manifest

### 將匯出分割成多個檔案

	AzCopy /Source:https://myaccount.table.core.windows.net/mytable/ /Dest:C:\myfolder /SourceKey:key /S /SplitSize:100

AzCopy 會在分割資料檔案名稱中使用*磁碟區索引*，以區分多個檔案。磁碟區索引由兩部分組成：*資料分割索引鍵範圍索引*和*分割檔案索引*。兩個索引皆以零為基礎。

如果使用者未指定 `/PKRS` 選項，資料分割索引鍵範圍索引將會是 0。

例如，在使用者指定 `/SplitSize` 選項之後，假設 AzCopy 產生兩個資料檔。產生的資料檔案名稱可能會是：

	myaccount_mytable_20140903T051850.8128447Z_0_0_C3040FE8.json
	myaccount_mytable_20140903T051850.8128447Z_0_1_0AB9AC20.json

請注意，`/SplitSize` 選項的最小可能值為 32MB。如果指定的目的地是 Blob 儲存體，則 AzCopy 會分割已達到 Blob 大小限制 (200GB) 的資料檔，而不論使用者是否已指定 `/SplitSize` 選項。

### 將資料表匯出為 JSON 或 CSV 資料檔案格式

AzCopy 依預設會將資料表匯出至 JSON 資料檔。您可以指定選項 `/PayloadFormat:JSON|CSV` 以將資料表匯出為 JSON 或 CSV。

	AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PayloadFormat:CSV

在指定 CSV 裝載格式時，AzCopy 也會產生結構描述檔案，且每個資料檔的副檔名為 `.schema.csv`。

### 並行匯出資料表實體

	AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PKRS:"aa#bb"

當使用者指定 `/PKRS` 選項時，AzCopy 將會啟動並行作業以匯出實體。每個作業分別會匯出一個資料分割索引鍵範圍。

請注意，並行作業的數目也由 `/NC` 選項控制。在複製資料表實體時，即使未指定 `/NC`，AzCopy 會以核心處理器的數目作為 `/NC` 的預設值。當使用者指定 `/PKRS` 選項時，AzCopy 會從兩個值 (資料分割索引鍵範圍和隱含或明確指定的並行作業) 中選擇較小者來使用，以決定要啟動的並行作業數目。如需詳細資訊，請在命令列上輸入 `AzCopy /?:NC`。

### 將資料表匯出至 Blob

	AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:https://myaccount.blob.core.windows.net/mycontainer/ /SourceKey:key1 /Destkey:key2

AzCopy 將會使用下列命令慣例，在 Blob 容器中產生 JSON 資料檔案：

	<account name>_<table name>_<timestamp>_<volume index>_<CRC>.json

產生的 JSON 資料檔案會遵循基本中繼資料的裝載格式。如需此裝載格式的詳細資訊，請參閱[資料表服務作業的裝載格式](http://msdn.microsoft.com/library/azure/dn535600.aspx)。

請注意，在將資料表匯出至 Blob 時，AzCopy 會先將資料表實體下載到本機暫存資料檔，再將這些實體上傳至 Blob。這些暫存資料檔會放入日誌檔案資料夾中，預設路徑為 “<code>%LocalAppData%\\Microsoft\\Azure\\AzCopy</code>”，您可以指定 /Z:[journal-file-folder] 選項來變更日誌檔案資料夾位置，從而變更暫存資料檔位置。暫存資料檔大小取決於資料表實體大小和您使用 /SplitSize 選項所指定的大小，雖然本機磁碟中的暫存資料檔在上傳至 Blob 之後就立即刪除，但請確定您有足夠的本機磁碟空間，可儲存這些尚未刪除的暫存資料檔。

## 資料表：匯入

### 匯入資料表

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.core.windows.net/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace

選項 `/EntityOperation` 指出如何將實體插入資料表。可能的值包括：

- `InsertOrSkip`：略過現有實體，或插入新實體 (若不存在於資料表中)。
- `InsertOrMerge`：合併現有實體，或插入新實體 (若不存在於資料表中)。
- `InsertOrReplace`：取代現有實體，或插入新實體 (若不存在於資料表中)。

請注意，您無法在匯入案例中指定 `/PKRS` 選項。不同於必須指定 `/PKRS` 選項以啟動並行作業的匯出案例，AzCopy 依預設會在您匯入資料表時啟動並行作業。依預設啟動的並行作業數目等於核心處理器的數目；但您可以使用 `/NC` 選項指定不同的並行數目。如需詳細資訊，請在命令列上輸入 `AzCopy /?:NC`。

請注意，AzCopy 只支援匯入 JSON 而不支援匯入 CSV。AzCopy 不支援從使用者建立的 JSON 和資訊清單檔案匯入資料表。這兩種檔案都必須來自 AzCopy 資料表匯出。若要避免錯誤，請勿修改匯出的 JSON 或資訊清單檔案。

### 使用 Blob 將實體匯入至資料表

假設有 Blob 容器包含下列檔案：代表 Azure 資料表和其隨附資訊清單檔案的 JSON 檔案。

	myaccount_mytable_20140103T112020.manifest
	myaccount_mytable_20140103T112020_0_0_0AF395F1DC42E952.json

您可以執行下列命令，以使用該 Blob 容器中的資訊清單檔案將實體匯入資料表中：

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:https://myaccount.table.core.windows.net/mytable /SourceKey:key1 /DestKey:key2 /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:"InsertOrReplace"

## 其他 AzCopy 功能

### 只複製目的地中沒有的資料

`/XO` 和 `/XN` 參數分別可讓您在複製作業中排除較舊或較新的來源資源。如果您只想複製目的地中沒有的來源資源，則可以在 AzCopy 命令中同時指定這兩個參數：

	/Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /XO /XN

	/Source:C:\myfolder /Dest:http://myaccount.file.core.windows.net/myfileshare /DestKey:<destkey> /S /XO /XN

	/Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:http://myaccount.blob.core.windows.net/mycontainer1 /SourceKey:<sourcekey> /DestKey:<destkey> /S /XO /XN

注意：當來源或目的地其中之一是資料表時，不支援此做法。

### 使用回應檔案指定命令列參數

	AzCopy /@:"C:\responsefiles\copyoperation.txt"

您可以在回應檔案中包含任何 AzCopy 命令列參數。AzCopy 處理檔案中的參數，就好像在命令列上指定這些參數一様，執行使用檔案內容的直接取代。

假設名為 `copyoperation.txt` 且包含下列資料行的回應檔案。每個 AzCopy 參數可以指定在同一行

	/Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

或不同行：

	/Source:http://myaccount.blob.core.windows.net/mycontainer
	/Dest:C:\myfolder
	/SourceKey:<sourcekey>
	/S
	/Y

如果您將參數分割成兩行，如以下的 `/sourcekey` 參數所示，則 AzCopy 將會失敗：

	http://myaccount.blob.core.windows.net/mycontainer
 	C:\myfolder
	/sourcekey:
	<sourcekey>
	/S
	/Y

### 使用多個回應檔案指定命令列參數

假設名為 `source.txt` 且指定來源容器的回應檔案：

	/Source:http://myaccount.blob.core.windows.net/mycontainer

及名為 `dest.txt` 且在檔案系統中指定目的地資料夾的回應檔案：

	/Dest:C:\myfolder

及名為 `options.txt` 且指定 AzCopy 選項的回應檔案：

	/S /Y

若要使用這些回應檔案 (所有這些檔案皆位於 `C:\responsefiles` 目錄內) 呼叫 AzCopy，請使用此命令：

	AzCopy /@:"C:\responsefiles\source.txt" /@:"C:\responsefiles\dest.txt" /SourceKey:<sourcekey> /@:"C:\responsefiles\options.txt"   

AzCopy 處理此命令，就好像您在命令列上包含所有個別參數一樣：

	AzCopy /Source:http://myaccount.blob.core.windows.net/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

### 指定共用存取簽章 (SAS)

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1 /Dest:https://myaccount.blob.core.windows.net/mycontainer2 /SourceSAS:SAS1 /DestSAS:SAS2 /Pattern:abc.txt

您也可以在容器 URI 上指定 SAS：

	AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer1/?SourceSASToken /Dest:C:\myfolder /S

### 日誌檔案資料夾

每次發佈命令至 AzCopy 時，它會檢查預設資料夾或透過此選項指定的資料夾中是否有日誌檔案存在。如果在這兩個地方都找不到日誌檔案，AzCopy 會將此作業視為新的作業，並產生新的日誌檔案。

如果找到日誌檔案，則 AzCopy 將檢查所輸入的命令列是否符合日誌檔案中的命令列。如果這兩個命令列相符，AzCopy 便會繼續未完成的作業。如果這兩個命令列不符，系統將提示您覆寫日誌檔案並開始新的作業，或取消目前作業。

如果您想要使用預設的日誌檔案位置：

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Z

如果省略 `/Z`，或指定 `/Z` 選項但沒有指定資料夾路徑 (如上所示)，則 AzCopy 會在預設位置上建立日誌檔案，預設位置是 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。如果日誌檔案已存在，則 AzCopy 會根據此日誌檔案繼續作業。

如果您想要指定自訂的日誌檔案位置：

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /Z:C:\journalfolder\

如果日誌檔案不存在，本範例將建立日誌檔案。如果日誌檔案已存在，則 AzCopy 會根據此日誌檔案繼續作業。

如果您想要繼續 AzCopy 作業：

	AzCopy /Z:C:\journalfolder\

本範例會繼續最後一個無法完成的作業。

### 產生記錄檔

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /V

如果指定 `/V` 選項，但沒有提供詳細資訊記錄的檔案路徑，則 AzCopy 會在預設位置上建立記錄檔，預設位置是 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。

否則，您可以在自訂位置建立記錄檔：

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.windows.net/mycontainer /DestKey:key /V:C:\myfolder\azcopy1.log

請注意，如果您在 `/V` 選項後面指定相對路徑 (例如 `/V:test/azcopy1.log`)，則系統會在目前工作目錄中的一個名為 `test` 的子資料夾內建立詳細資訊記錄。

### 指定要啟動的並行作業數目

`/NC` 選項可指定並行複製作業的數目。根據預設，AzCopy 依預設會啟動特定數量的並行作業，以提高資料傳輸的輸送量。若為資料表作業，並行作業數目等於您所擁有的處理器數目。若為 Blob 和檔案作業，並行作業數目等於您所擁有處理器數目的 8 倍。如果您在低頻寬的網路上執行 AzCopy，則您可以在 /NC 中指定較低的數字，以避免因為資源競爭所導致的失敗。

### 針對 Azure 儲存體模擬器執行 AzCopy

您可以針對 Blob，對 [Azure 儲存體模擬器](storage-use-emulator.md)執行 AzCopy：

	AzCopy /Source:https://127.0.0.1:10000/myaccount/mycontainer/ /Dest:C:\myfolder /SourceKey:key /SourceType:Blob /S

以及針對資料表來執行：

	AzCopy /Source:https://127.0.0.1:10002/myaccount/mytable/ /Dest:C:\myfolder /SourceKey:key /SourceType:Table

## AzCopy 參數

以下內容說明 AzCopy 的參數。您也可以從命令列中輸入下列其中一個命令，以取得 AzCopy 的使用說明：

- 如需 AzCopy 的詳細命令列說明：`AzCopy /?`
- 如需任何 AzCopy 參數的詳細說明：`AzCopy /?:SourceKey`
- 如需命令列範例：`AzCopy /?:Samples`

### /Source:"source"

指定要複製的來源資料。來源可以是檔案系統目錄、Blob 容器、Blob 虛擬目錄、儲存體檔案共用、儲存體檔案目錄或 Azure 資料表。

**適用於：**Blob、檔案、資料表

### /Dest:"destination"

指定複製的目的地。目的地可以是檔案系統目錄、Blob 容器、Blob 虛擬目錄、儲存體檔案共用、儲存體檔案目錄或 Azure 資料表。

**適用於：**Blob、檔案、資料表

### /Pattern:"file-pattern"

指定一個檔案模式以指出所要複製的檔案。/Pattern 參數的行為取決於來源資料的位置以及是否有遞迴模式選項。遞迴模式可透過選項 /S 來指定。

如果指定來源是檔案系統中的目錄，則標準萬用字元便會立即生效，且所提供的檔案模式會與目錄中的檔案做比對。如果已指定選項 /S，則 AzCopy 也會將指定模式與該目錄下任何子資料夾中的所有檔案做比對。

如果指定來源是 Blob 容器或虛擬目錄，則萬用字元並不適用。如果已指定選項 /S，則 AzCopy 會將指定的檔案模式解譯為 Blob 首碼。如果未指定選項 /S，則 AzCopy 會將檔案模型與確切 Blob 名稱做比對。

如果指定來源是 Azure 檔案共用，則您必須指定確切檔案名稱 (例如 abc.txt) 以複製單一檔案，或指定選項 /S 以遞迴方式複製共用中的所有檔案。嘗試同時指定檔案模式和選項 /S，將會造成錯誤。

當 /Source 是 Blob 容器或 Blob 的虛擬目錄時，AzCopy 會使用區分大小寫比對，並在所有其他情況下使用不區分大小寫比對。

未指定檔案模式時，使用的預設檔案模式如下：針對檔案系統位置，會使用 *.*，針對「Azure 儲存體」位置，則是使用空白首碼。不支援指定多個檔案模式。

**適用於：**Blob、檔案

### /DestKey:"storage-key"

指定目的地資源的儲存體帳戶金鑰。

**適用於：**Blob、檔案、資料表

### /DestSAS:"sas-token"

以讀取和寫入權限指定目的地的共用存取簽章 (SAS) (如果適用的話)。為 SAS 加上雙引號，因為它可能包含特殊命令列字元。

如果目的地資源是 Blob 容器、檔案共用或資料表，您可以指定此選項後面接著 SAS 權杖，或者您可以不使用此選項，直接將 SAS 指定為目的地 Blob 容器、檔案共用或資料表 URI 的一部分。

如果來源和目的地都是 Blob，則目的地 Blob 必須與來源 Blob 位於相同的儲存體帳戶中。

**適用於：**Blob、檔案、資料表

### /SourceKey:"storage-key"

指定來源資源的儲存體帳戶金鑰。

**適用於：**Blob、檔案、資料表

### /SourceSAS:"sas-token"

以讀取和清單權限指定來源的共用存取簽章 (如果適用的話)。為 SAS 加上雙引號，因為它可能包含特殊命令列字元。

如果來源資源是 Blob 容器，且未提供金鑰及 SAS，則可透過匿名存取讀取該 Blob 容器。

如果來源是檔案共用或資料表，則必須提供金鑰或 SAS。

**適用於：**Blob、檔案、資料表

### /S

指定複製作業的遞迴模式。在遞迴模式中，AzCopy 將複製所有符合指定檔案模式的 Blob 或檔案，包括子資料夾中的 Blob 或檔案。

**適用於：**Blob、檔案

### /BlobType:"block" | "page" | "append"

指定目的地 Blob 是區塊 Blob、分頁 Blob 或附加 Blob。此選項僅在上傳 Blob 時才適用。否則會產生錯誤。如果目的地是 Blob 且未指定此選項，則 AzCopy 預設會建立區塊 Blob。

**適用於：**Blob

### /CheckMD5

計算下載資料的 MD5 雜湊，並驗證 MD5 雜湊是否儲存在符合計算之雜湊的 Blob 或檔案的 Content-MD5 屬性中。MD5 檢查依預設為關閉，因此您必須指定此選項，才能在下載資料時執行 MD5 檢查。

請注意，Azure 儲存體並不保證儲存供 Blob 或檔案使用的 MD5 雜湊是最新的版本。每次修改 Blob 或檔案時將 MD5 更新是用戶端的責任。

在上傳至服務之後，AzCopy 一定會為 Azure Blob 或檔案設定 Content-MD5 屬性。

**適用於：**Blob、檔案

### /Snapshot

指出是否傳輸快照。此選項僅在來源是 Blob 才有效。

傳輸的 Blob 快照集會以下列格式重新命名：blob-name (snapshot-time).extension

依預設，不會複製快照。

**適用於：**Blob

### /V:[verbose-log-file]

將詳細資訊狀態訊息輸出至記錄檔。

根據預設，在 `%LocalAppData%\Microsoft\Azure\AzCopy` 中詳細資訊記錄檔會被命名為 AzCopyVerbose.log。如果您在此選項中指定現有檔案位置，則詳細資訊記錄將會被附加到該檔案。

**適用於：**Blob、檔案、資料表

### /Z:[journal-file-folder]

指定用於繼續作業的日誌檔案資料夾。

如果作業遭到中斷，AzCopy 絕對支援繼續作業。

如果未指定此選項，或指定此選項但沒有指定資料夾路徑，則 AzCopy 將在預設位置上建立日誌檔案，預設位置是 %LocalAppData%\\Microsoft\\Azure\\AzCopy。

每次發佈命令至 AzCopy 時，它會檢查預設資料夾或透過此選項指定的資料夾中是否有日誌檔案存在。如果在這兩個地方都找不到日誌檔案，AzCopy 會將此作業視為新的作業，並產生新的日誌檔案。

如果找到日誌檔案，則 AzCopy 將檢查所輸入的命令列是否符合日誌檔案中的命令列。如果這兩個命令列相符，AzCopy 便會繼續未完成的作業。如果這兩個命令列不符，系統將提示您覆寫日誌檔案並開始新的作業，或取消目前作業。

成功完成作業時，系統便會將日誌檔案刪除。

請注意，不支援根據舊版的 AzCopy 所建立的日誌檔案繼續作業。

**適用於：**Blob、檔案、資料表

### /@:"parameter-file"

指定包含參數的檔案。AzCopy 處理檔案中的參數，就好像在命令列上指定這些參數一様。

在回應檔案中，您可以在單行中指定多個參數，或每一行各自指定一個參數。請注意，各個參數無法橫跨多行。

回應檔案可包含開頭為 # 符號的命令列。

您可以指定多個回應檔案。不過，請記住 AzCopy 不支援巢狀回應檔案。

**適用於：**Blob、檔案、資料表

### /Y

隱藏所有 AzCopy 確認提示。

**適用於：**Blob、檔案、資料表

### /L

僅指定清單作業，不會複製資料。

AzCopy 會解譯使用這個選項為模擬不使用 /L 選項來執行此命令列，而且會計算有多少物件會被複製，您可以同時指定 /V 選項，檢查要複製哪些物件到詳細記錄檔中。

這個選項的行為還取決於來源資料的位置，以及是否有遞迴模式選項 /S 和檔案模式選項 /Pattern。

使用此選項時，AzCopy 會需要此來源位置的清單和讀取權限。

**適用於：**Blob、檔案

### /MT

將下載檔案的最後修改時間設定為與來源 Blob 或檔案相同的最後修改時間。

**適用於：**Blob、檔案

### /XN

排除較新的來源資源。如果最後修改時間比目的地還要新或同時間，則不會複製資源。

**適用於：**Blob、檔案

### /XO

排除較舊的來源資源。如果最後修改時間比目的地還要舊或同時間，則不會複製資源。

**適用於：**Blob、檔案

### /A

僅上傳已設定 [封存] 屬性的檔案。

**適用於：**Blob、檔案

### /IA:[RASHCNETOI]

僅上傳已設定任何指定屬性的檔案。

可用屬性包括：

- R = 唯讀檔案
- A = 準備封存的檔案
- S = 系統檔案
- H = 隱藏檔案
- C = 壓縮檔案
- N = 正常檔案
- E = 加密檔案
- T = 暫存檔案
- O = 離線檔案
- I = 未編製索引的檔案

**適用於：**Blob、檔案

### /XA:[RASHCNETOI]

排除已設定任何指定屬性的檔案。

可用屬性包括：

- R = 唯讀檔案
- A = 準備封存的檔案
- S = 系統檔案
- H = 隱藏檔案
- C = 壓縮檔案
- N = 正常檔案
- E = 加密檔案
- T = 暫存檔案
- O = 離線檔案
- I = 未編製索引的檔案

**適用於：**Blob、檔案

### /Delimiter:"delimiter"

指出在 Blob 名稱中，用來分隔虛擬目錄的分隔符號字元。

依預設，AzCopy 會使用 / 作為分隔符號字元。不過，AzCopy 支援使用任何常見字元 (例如 @、# 或 %) 作為分隔符號。如果您必須在命令列中包含其中一個特殊字元，請為檔案名稱加上雙引號。

此選項僅適用於下載 Blob。

**適用於：**Blob

### /NC:"number-of-concurrent-operations"

指定並行作業的數目。

AzCopy 依預設會啟動特定數量的並行作業，以提高資料傳輸的輸送量。請注意，在低頻寬環境中的大量並行作業有可能會拖垮網路連線，並使作業無法完成。根據實際可用網路頻寬來節流處理並行作業。

並行作業數的上限為 512。

**適用於：**Blob、檔案、資料表

### /SourceType:"Blob" | "Table"

指定 `source` 資源是本機開發環境中可用、於儲存體模擬器中執行的 Blob。

**適用於：**Blob、資料表

### /DestType:"Blob" | "Table"

指定 `destination` 資源是本機開發環境中可用、於儲存體模擬器中執行的 Blob。

**適用於：**Blob、資料表

### /PKRS:"key1#key2#key3#..."

分割資料分割索引鍵範圍以支援平行匯出資料表資料的作業，這樣可以加快匯出作業的速度。

若未指定此選項，AzCopy 會使用單一執行緒來匯出資料表實體。例如，如果使用者指定 /PKRS:"aa#bb"，則 AzCopy 會啟動三個並行作業。

每個作業分別會匯出三個資料分割索引鍵範圍的其中一個，如下所示：

  [first-partition-key, aa)

  [aa, bb)

  [bb, last-partition-key]

**適用於：**資料表

### /SplitSize:"file-size"

指定匯出的檔案分割大小 (以 MB 為單位)，允許的最小值為 32。

若未指定此選項，則 AzCopy 會將資料表資料匯出至單一檔案。

如果資料表資料匯出至 Blob，且匯出的檔案大小達到 Blob 的大小限制 200 GB，則 AzCopy 會分割匯出的檔案，即使未指定此選項亦然。

**適用於：**資料表

### /EntityOperation:"InsertOrSkip" | "InsertOrMerge" | "InsertOrReplace"

指定資料表資料匯入行為。

- InsertOrSkip - 略過現有實體，或插入新實體 (若不存在於資料表中)。

- InsertOrMerge - 合併現有實體，或插入新實體 (若不存在於資料表中)。

- InsertOrReplace - 取代現有實體，或插入新實體 (若不存在於資料表中)。

**適用於：**資料表

### /Manifest:"manifest-file"

指定資料表匯出和匯入作業的資訊清單檔。

在匯出作業期間，此選項是選擇性的，如果沒有指定這個選項，AzCopy 便會產生具有預先定義名稱的資訊清單檔案。

在匯入作業期間，為了尋找資料檔案，這是必要選項。

**適用於：**資料表

### /SyncCopy

指出在兩個 Azure 儲存體端點之間是否以同步方式複製 blob 或檔案。

AzCopy 依預設會使用伺服器端的非同步複本。指定此選項可執行同步複製，也就是將 blog 或檔案下載至本機記憶體，再上傳至 Azure 儲存體。

當您複製檔案是在 Blob 儲存體內、在檔案儲存體內，或從 Blob 儲存體到檔案儲存體 (或反過來) 時，您可以使用此選項。

**適用於：**Blob、檔案

### /SetContentType:"content-type"

指定目的地 blob 或檔案的 MIME 內容類型。

AzCopy 預設會將 blob 或檔案的內容類型設定為 application/octet-stream。您可以明確指定這個選項的值來設定所有 blob 或檔案的內容類型。

如果您指定這個選項但未指定任何值，則 AzCopy 會根據副檔名來設定每個 blob 或檔案的內容類型。

**適用於：**Blob、檔案

### /PayloadFormat:"JSON" | "CSV"

指定資料表匯出的資料檔格式。

如果未指定此選項，根據預設，AzCopy 會匯出 JSON 格式的資料表資料檔案。

**適用於：**資料表

## 已知問題和最佳作法

### 限制複製資料時的並行寫入

使用 AzCopy 複製 Blob 或檔案時，請留意當您在複製資料時，另一個應用程式可能正在修改該資料。請儘可能地確定在複製作業過程中，您正要複製的資料並不在修改中。例如，當複製與 Azure 虛擬機器相關聯的 VHD 時，請確定目前沒有其他應用程式正在寫入此 VHD。要這樣做的一個好方法是租用要複製的資源。此外，您可以首先建立 VHD 的快照，然後複製此快照。

如果您無法在複製時防止其他應用程式寫入 Blob 或檔案，請記住，工作完成時，複製的資源可能不再與來源資源完全相同。

### 在一部電腦上執行一個 AzCopy 執行個體。
AzCopy 設計為充分利用電腦資源來加速資料傳輸，建議您在一部電腦上只執行一個 AzCopy 執行個體，如果需要更多並行作業，您可以指定 `/NC` 選項。如需詳細資訊，請在命令列上輸入 `AzCopy /?:NC`。

### 當您「使用 FIPS 相容演算法於加密、雜湊及簽章」時，請為 AzCopy 啟用 FIPS 相容的 MD5 演算法。
複製物件時，AzCopy 預設會使用 .NET MD5 實作來計算此 MD5，但仍有一些需要 AzCopy 啟用 FIPS 相容 MD5 設定的安全性需求。

您可以建立具有 `AzureStorageUseV1MD5` 屬性的 app.config 檔案 `AzCopy.exe.config`，並將它放在 AzCopy.exe 旁邊。

	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	  <appSettings>
	    <add key="AzureStorageUseV1MD5" value="false"/>
	  </appSettings>
	</configuration>

針對屬性 “AzureStorageUseV1MD5” • True - 預設值，AzCopy 將使用 .NET MD5 實作。• False – AzCopy 將使用 FIPS 相容的 MD5 演算法。

請注意 Windows 電腦預設會停用 FIPS 相容演算法，可以在您執行的視窗中輸入 secpol.msc，並在 [安全性設定] -> [本機原則] -> [安全性選項] -> [系統密碼編譯: 使用 FIPS 相容演算法於加密、雜湊及簽章] 檢查此參數。

## 後續步驟

如需關於 Azure 儲存體和 AzCopy 的詳細資訊，請參閱下列資源。

### Azure 儲存體文件：

- [Azure 儲存體簡介](storage-introduction.md)
- [如何使用 .NET 的 Blob 儲存體](storage-dotnet-how-to-use-blobs.md)
- [如何使用 .NET 的檔案儲存體](storage-dotnet-how-to-use-files.md)
- [如何使用 .NET 的資料表儲存體](storage-dotnet-how-to-use-tables.md)
- [如何建立、管理或刪除儲存體帳戶](storage-create-storage-account.md)

### Azure 儲存體部落格文章：
- [Azure 儲存體資料移動文件庫預覽簡介](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)
- [AzCopy：簡介同步複製和自訂內容類型](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/01/13/azcopy-introducing-synchronous-copy-and-customized-content-type.aspx)
- [AzCopy: 宣布正式發行 AzCopy 3.0 和具有資料表和檔案支援的 AzCopy 4.0 預覽版本](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)
- [AzCopy: 針對大規模複製案例最佳化](http://go.microsoft.com/fwlink/?LinkId=507682)
- [AzCopy: 支援讀取存取異地備援儲存體](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)
- [AzCopy: 使用可重新啟動模式和 SAS 權杖傳輸資料](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx)
- [AzCopy: 使用跨帳戶複製 Blob](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)
- [AzCopy: 上傳/下載 Azure Blob 的檔案](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)

<!---HONumber=AcomDC_0921_2016-->