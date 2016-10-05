<properties
	pageTitle="如何在 Azure API 管理中設定 VPN 連線"
	description="瞭解如何在 API 管理中設定 VPN 連線，並透過它存取 Web 服務。"
	services="api-management"
	documentationCenter=""
	authors="antonba"
	manager="erikre"
	editor=""/>

<tags
	ms.service="api-management"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/09/2016"
	ms.author="antonba"/>

# 如何在 Azure API 管理中設定 VPN 連線

「API 管理」的 VPN 支援可讓您將「API 管理」閘道連接到「Azure 虛擬網路」(傳統)。這可讓 API 服務客戶安全地連線到其內部部署或公用網際網路無法存取的後端 Web 服務。

>[AZURE.NOTE] 「Azure API 管理」可以與傳統 VNET 搭配使用。如需建立傳統 VNET 的詳細資訊，請參閱[使用 Azure 入口網站建立虛擬網路 (傳統)](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。如需連接傳統 VNET 和 ARM VNET 的相關資訊，請參閱[連接傳統 VNet 和新的 VNet](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)。

## <a name="enable-vpn"> </a>啟用 VPN 連線

>VPN 連線僅供**進階**和**開發人員**層使用。若要切換到它，請在 [Azure 傳統入口網站][]中開啟 API 管理服務，然後開啟 [調整] 索引標籤。在 [**一般**] 區段下選取進階層，然後按一下 [儲存]。

若要啟用 VPN 連線，請在 [Azure 傳統入口網站][]中開啟 API 管理服務，然後切換到 [設定] 索引標籤。

在 VPN 區段中，將 [**VPN 連線**] 切換為 [**開啟**]。

![API 管理執行個體的設定索引標籤][api-management-setup-vpn-configure]

您現在會看見佈建 API 管理服務所在的所有區域的清單。

為每個區域選取 VPN 和子網路。VPN 的清單是依據您要設定的區域中所設定 Azure 訂用帳戶可用的虛擬網路而填入。

![選取 VPN][api-management-setup-vpn-select]

按一下畫面底部的 [**儲存**]。更新時，您將無法透過 Azure 傳統入口網站對 API 管理服務執行其他作業。服務閘道將保持可用，而執行時期呼叫應該不會受到影響。

請注意，閘道的 VIP 位址將在每次啟用或停用 VPN 時變更。

## <a name="connect-vpn"> </a>連接至 VPN 的 Web 服務

在您的 API 管理服務連接至 VPN 之後，於虛擬網路內存取 Web 服務與存取公用服務沒有差別。只需在建立新 API 或編輯現有 API 時，將 Web 服務的本機位址或主機名稱 (如果已設定 Azure 虛擬網路的 DNS 伺服器) 輸入到 [**Web 服務 URL**] 欄位。

![透過 VPN 加入 API][api-management-setup-vpn-add-api]

## API 管理 VPN 支援所需的連接埠

當 API 管理服務執行個體裝載於 VNET 時，會使用下表中的連接埠。如果封鎖這些連接埠，服務可能無法正常運作。搭配 VNET 使用 API 管理時，封鎖這其中一或多個連接埠是最常見的錯誤組態問題。

| 連接埠 | 方向 | 傳輸通訊協定 | 目的 | 來源 / 目的地 |
|------------------------------|------------------|--------------------|------------------------------------------------------------------|-----------------------------------|
| 80、443 | 輸入 | TCP | 與 API 管理的用戶端通訊 | INTERNET / VIRTUAL\_NETWORK |
| 80,443 | 輸出 | TCP | Azure 儲存體和 Azure 服務匯流排上的 API 管理相依性 | VIRTUAL\_NETWORK / INTERNET |
| 1433 | 輸出 | TCP | SQL 上的 API 管理相依性 | VIRTUAL\_NETWORK / INTERNET |
| 9350, 9351, 9352, 9353, 9354 | 輸出 | TCP | 服務匯流排上的 API 管理相依性 | VIRTUAL\_NETWORK / INTERNET |
| 5671 | 輸出 | AMQP | 記錄到事件中樞原則的 API 管理相依性 | VIRTUAL\_NETWORK / INTERNET |
| 6381, 6382, 6383 | 輸入/輸出 | UDP | Redis 快取上的 API 管理相依性 | VIRTUAL\_NETWORK / VIRTUAL\_NETWORK |
| 445 | 輸出 | TCP | 適用於 GIT 的 Azure 檔案共用上的 API 管理相依性 | VIRTUAL\_NETWORK / INTERNET |

## <a name="custom-dns"> </a>自訂 DNS 伺服器設定

API 管理取決於多項 Azure 服務。當 API 管理服務執行個體裝載於使用自訂 DNS 伺服器的 VNET 時，它必須能夠解析這些 Azure 服務的主機名稱。請遵循[這份](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-using-your-own-dns-server)有關自訂 DNS 設定的指引。

## <a name="related-content"> </a>相關內容


* [使用 Azure 傳統入口網站建立具有站對站 VPN 連線的虛擬網路][]
* [如何在 Azure API 管理中使用 API 偵測器來追蹤呼叫][]

[api-management-setup-vpn-configure]: ./media/api-management-howto-setup-vpn/api-management-setup-vpn-configure.png
[api-management-setup-vpn-select]: ./media/api-management-howto-setup-vpn/api-management-setup-vpn-select.png
[api-management-setup-vpn-add-api]: ./media/api-management-howto-setup-vpn/api-management-setup-vpn-add-api.png

[Enable VPN connections]: #enable-vpn
[Connect to a web service behind VPN]: #connect-vpn
[Related content]: #related-content

[Azure 傳統入口網站]: https://manage.windowsazure.com/

[使用 Azure 傳統入口網站建立具有站對站 VPN 連線的虛擬網路]: ../vpn-gateway/vpn-gateway-site-to-site-create.md
[如何在 Azure API 管理中使用 API 偵測器來追蹤呼叫]: api-management-howto-api-inspector.md

<!---HONumber=AcomDC_0921_2016-->