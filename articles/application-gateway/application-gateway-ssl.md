<properties
   pageTitle="使用傳統部署設定適用於 SSL 卸載的應用程式閘道 | Microsoft Azure"
   description="本文提供使用 Azure 傳統部署模型建立具有 SSL 卸載之應用程式閘道的指示。"
   documentationCenter="na"
   services="application-gateway"
   authors="georgewallace"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/09/2016"
   ms.author="gwallace"/>

# 使用傳統部署模型設定適用於 SSL 卸載的應用程式閘道

> [AZURE.SELECTOR]
-[Azure portal](application-gateway-ssl-portal.md)
-[Azure Resource Manager PowerShell](application-gateway-ssl-arm.md)
-[Azure Classic PowerShell](application-gateway-ssl.md)

Azure 應用程式閘道可以設定為在閘道終止安全通訊端層 (SSL) 工作階段，以避免 Web 伺服陣列發生高成本的 SSL 解密工作。SSL 卸載也可以簡化 Web 應用程式的前端伺服器設定和管理。


## 開始之前

1. 使用 Web Platform Installer 安裝最新版的 Azure PowerShell Cmdlet。您可以從[下載頁面](https://azure.microsoft.com/downloads/)的 **Windows PowerShell** 區段下載並安裝最新版本。
2. 請確認您的運作中虛擬網路具有有效子網路。請確定沒有虛擬機器或是雲端部署正在使用子網路。應用程式閘道必須單獨位於虛擬網路子網路中。
3. 您要設定來使用應用程式閘道的伺服器必須存在，或是在虛擬網路中建立其端點，或是已指派公用 IP/VIP。

若要在應用程式閘道上設定 SSL 卸載，請依列出的順序執行下列步驟：

1. [建立應用程式閘道](#create-an-application-gateway)
2. [上傳 SSL 憑證](#upload-ssl-certificates)
3. [設定閘道](#configure-the-gateway)
4. [設定閘道組態](#set-the-gateway-configuration)
5. [啟動閘道](#start-the-gateway)
6. [確認閘道狀態](#verify-the-gateway-status)


## 建立應用程式閘道

若要建立閘道，請使用 **New-AzureApplicationGateway** Cmdlet，並將值取代為您自己的值。此時還不會開始對閘道計費。會在稍後的步驟中於成功啟動閘道之後開始計費。

	New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")

若要驗證是否已建立閘道，您可以使用 **Get-AzureApplicationGateway** Cmdlet。

在範例中，*Description*、*InstanceCount* 和 *GatewaySize* 是選用參數。*InstanceCount* 的預設值是 2，且最大值是 10。*GatewaySize* 的預設值是 Medium。Small 和 Large 也是可用的值。因為尚未啟動閘道，所以 *VirtualIPs* 和 *DnsName* 會顯示為空白。閘道處於執行中狀態之後，就會建立這些值。

	Get-AzureApplicationGateway AppGwTest

## 上傳 SSL 憑證

使用 **Add-AzureApplicationGatewaySslCertificate** 將伺服器憑證以 *pfx* 格式上傳至應用程式閘道。憑證名稱是使用者選擇的名稱，而且必須在應用程式閘道中是唯一的。應用程式閘道上的所有憑證管理作業會使用此名稱參考該憑證。

下列範例會顯示 Cmdlet，請將範例中的值取代為您自己的值。

	Add-AzureApplicationGatewaySslCertificate  -Name AppGwTest -CertificateName GWCert -Password <password> -CertificateFile <full path to pfx file>

接下來，驗證憑證上傳。使用 **Get-AzureApplicationGatewayCertificate** Cmdlet。

這個範例的第一行顯示 Cmdlet，後面接著輸出。

	Get-AzureApplicationGatewaySslCertificate AppGwTest

	VERBOSE: 5:07:54 PM - Begin Operation: Get-AzureApplicationGatewaySslCertificate
	VERBOSE: 5:07:55 PM - Completed Operation: Get-AzureApplicationGatewaySslCertificate
	Name           : SslCert
	SubjectName    : CN=gwcert.app.test.contoso.com
	Thumbprint     : AF5ADD77E160A01A6......EE48D1A
	ThumbprintAlgo : sha1RSA
	State..........: Provisioned

>[AZURE.NOTE] 憑證密碼必須由 4 到 12 個字元、字母或數字所組成。不接受使用特殊字元。

## 設定閘道

應用程式閘道組態是由多個值所組成。可以將值繫結在一起，以建構組態。

值如下：

- **後端伺服器集區：**後端伺服器的 IP 位址清單。列出的 IP 位址應屬於虛擬網路子網路或是公用 IP/VIP。
- **後端伺服器集區設定：**每個集區都包括一些設定，例如連接埠、通訊協定和以 Cookie 為基礎的同質性。這些設定會繫結至集區，並套用至集區內所有伺服器。
- **前端連接埠：**此連接埠是在應用程式閘道上開啟的公用連接埠。流量會到達此連接埠，然後重新導向至其中一個後端伺服器。
- **接聽程式：**接聽程式具有前端連接埠、通訊協定 (Http 或 Https，這些值都區分大小寫) 和 SSL 憑證名稱 (如果已設定 SSL 卸載)。
- **規則：**規則會繫結接聽程式和後端伺服器集區，並定義當流量到達特定接聽程式時，應該導向到哪個後端伺服器集區。目前只支援*基本*規則。*基本*規則是循環配置資源的負載分配。

**其他組態注意事項**

針對 SSL 憑證組態，**HttpListener** 中的通訊協定應該變更為 Https (區分大小寫)。**SslCert** 元素會新增到 **HttpListener** 中，其中值會設定為與上傳先前的 SSL 憑證一節中所使用的名稱相同。前端連接埠應該更新為 443。

**啟用以 Cookie 為基礎的同質性**：您可以設定應用程式閘道，以確保來自用戶端工作階段的要求一律會導向至 Web 伺服陣列中的相同 VM。此案例透過插入允許閘道適當導向流量的工作階段 Cookie 來完成。若要啟用以 Cookie 為基礎的同質性，請在 **BackendHttpSettings** 元素中將 **CookieBasedAffinity** 設定為 *Enabled*。



建立組態物件或使用組態 XML 檔案，即可建構組態。若要使用組態 XML 檔案以建構組態，請使用下列範例：

**組態 XML 範例**

	<?xml version="1.0" encoding="utf-8"?>
	<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
	    <FrontendIPConfigurations />
	    <FrontendPorts>
	        <FrontendPort>
	            <Name>FrontendPort1</Name>
	            <Port>443</Port>
	        </FrontendPort>
	    </FrontendPorts>
	    <BackendAddressPools>
	        <BackendAddressPool>
	            <Name>BackendPool1</Name>
	            <IPAddresses>
	                <IPAddress>10.0.0.1</IPAddress>
	                <IPAddress>10.0.0.2</IPAddress>
	            </IPAddresses>
	        </BackendAddressPool>
	    </BackendAddressPools>
	    <BackendHttpSettingsList>
	        <BackendHttpSettings>
	            <Name>BackendSetting1</Name>
	            <Port>80</Port>
	            <Protocol>Http</Protocol>
	            <CookieBasedAffinity>Enabled</CookieBasedAffinity>
	        </BackendHttpSettings>
	    </BackendHttpSettingsList>
	    <HttpListeners>
	        <HttpListener>
	            <Name>HTTPListener1</Name>
	            <FrontendPort>FrontendPort1</FrontendPort>
	            <Protocol>Https</Protocol>
	            <SslCert>GWCert</SslCert>
	        </HttpListener>
	    </HttpListeners>
	    <HttpLoadBalancingRules>
	        <HttpLoadBalancingRule>
	            <Name>HttpLBRule1</Name>
	            <Type>basic</Type>
	            <BackendHttpSettings>BackendSetting1</BackendHttpSettings>
	            <Listener>HTTPListener1</Listener>
	            <BackendAddressPool>BackendPool1</BackendAddressPool>
	        </HttpLoadBalancingRule>
	    </HttpLoadBalancingRules>
	</ApplicationGatewayConfiguration>


## 設定閘道組態

接下來，您需要設定應用程式閘道。您可以將 **Set-AzureApplicationGatewayConfig** Cmdlet 與組態物件或組態 XML 檔案搭配使用。

	Set-AzureApplicationGatewayConfig -Name AppGwTest -ConfigFile D:\config.xml

## 啟動閘道

設定閘道之後，請使用 **Start-AzureApplicationGateway** Cmdlet 來啟動閘道。成功啟動閘道之後，會開始應用程式閘道計費。


**注意：** **Start-AzureApplicationGateway** Cmdlet 最多可能需要 15-20 分鐘才能完成。

	Start-AzureApplicationGateway AppGwTest

## 確認閘道狀態

使用 **Get-AzureApplicationGateway** Cmdlet 來檢查閘道狀態。如果 **Start-AzureApplicationGateway** 在上一個步驟中成功，則 *State* 應該會是 Running，而 *VirtualIPs* 和 *DnsName* 應該會有有效的項目。

這個範例所示範的應用程式閘道已啟動、執行中並準備好要接受流量。

	Get-AzureApplicationGateway AppGwTest

	Name          : AppGwTest2
	Description   :
	VnetName      : testvnet1
	Subnets       : {Subnet-1}
	InstanceCount : 2
	GatewaySize   : Medium
	State         : Running
	VirtualIPs    : {23.96.22.241}
	DnsName       : appgw-4c960426-d1e6-4aae-8670-81fd7a519a43.cloudapp.net


## 後續步驟


如果您想進一步了解一般負載平衡選項，請參閱：

- [Azure 負載平衡器](https://azure.microsoft.com/documentation/services/load-balancer/)
- [Azure 流量管理員](https://azure.microsoft.com/documentation/services/traffic-manager/)

<!---HONumber=AcomDC_0921_2016-->