<properties
   pageTitle="在 Azure 自動化中開始 Runbook | Microsoft Azure"
   description="摘要說明可以用來在 Azure 自動化中啟動 Runbook 的不同的方法，並提供使用 Azure 入口網站和 Windows PowerShell 的詳細資訊。"
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="jwhit"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/15/2016"
   ms.author="magoedte;bwren"/>

# 在 Azure 自動化中啟動 Runbook

下表將協助您判斷在 Azure 自動化中啟動 Runbook 的方法，最適合您的特定案例。這篇文章包含有關使用 Azure 入口網站和 Windows PowerShell 啟動 Runbook 的詳細資料。其他方法的詳細資訊在其他文件中提供，您可以從下列連結來存取。

| **方法** | **特性** |
|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Azure 入口網站](#starting-a-runbook-with-the-azure-portal) | <li>互動式使用者介面的最簡單方法。<br> <li>提供簡單參數值的表單。<br> <li>輕鬆追蹤工作狀態。<br> <li>使用 Azure 登入資訊驗證存取。 |
| [Windows PowerShell](https://msdn.microsoft.com/library/dn690259.aspx) | <li>使用 Windows PowerShell Cmdlet 從命令列呼叫。<br> <li>可以包含在具有多個步驟的自動化解決方案中。<br> <li>要求會透過憑證或 OAuth 使用者主體/服務主體進行驗證。<br> <li>提供簡單和複雜的參數值。<br> <li>追蹤工作狀態。<br> <li>支援 PowerShell Cmdlet 所需的用戶端。 |
| [Azure 自動化 API](https://msdn.microsoft.com/library/azure/mt662285.aspx) | <li>提供最有彈性但也最複雜的方法。<br> <li>可從任何能發出 HTTP 要求的自訂程式碼呼叫。<br> <li>透過憑證或 OAuth 使用者主體/服務主體驗證要求。<br> <li>提供簡單和複雜的參數值。<br> <li>追蹤工作狀態。 |
| [Webhook](automation-webhooks.md) | <li>從單一 HTTP 要求啟動 Runbook。<br> <li>透過 URL 中的安全性權杖進行驗證。<br> <li>用戶端無法在建立 Webhook 時覆寫指定的參數值。Runbook 可以定義已填入 HTTP 要求詳細資料的單一參數。<br> <li>無法透過 Webhook URL 追蹤工作狀態。 |
| [回應 Azure 警示](../log-analytics/log-analytics-alerts.md) | <li>啟動 Runbook 以回應 Azure 警示。<br> <li>針對 Runbook 和警示的連結設定 Webhook。<br> <li>透過 URL 中的安全性權杖進行驗證。<br> <li>目前僅支援計量的警示。 |
| [排程](automation-scheduling-a-runbook.md) | <li>以每小時、每日或每週排程，自動啟動 Runbook。<br> <li>可透過 Azure 入口網站、PowerShell Cmdlet 或 Azure API 操控排程。<br> <li>提供可用於排程的參數值。 |
| [另一個 Runbook](automation-child-runbooks.md) | <li>在另一個 Runbook 中使用 Runbook 作為活動。<br> <li>對多個 Runbook 使用的功能來說很有用。<br> <li>提供參數值給子 Runbook，並在父 Runbook 中使用輸出。 |

下圖說明 Runbook 的生命週期的詳細逐步程序。它包含了 Runbook 在 Azure 自動化中啟動的不同方式、Hybrid Runbook Worker 執行 Azure 自動化 Runbook 所需的元件，以及不同元件之間的互動。若要了解如何在您的資料中心執行自動化 Runbook，請參閱[混合式 Runbook 背景工作](automation-hybrid-runbook-worker.md)

![Runbook 架構](media/automation-starting-runbook/runbooks-architecture.png)

## 使用 Azure 入口網站啟動 Runbook

1.	從您的自動化帳戶，按一下 [**Runbook**] 部分，以開啟 [**Runbook**] 分頁。
2.	按一下 Runbook 以開啟其 [**Runbook**] 分頁。
3.	按一下 [啟動]。
4.	如果 Runbook 沒有參數，系統會提示您確認您是否想要啟動它。如果 Runbook 有參數，[**啟動 Runbook**] 分頁會開啟，因此您可以提供參數值。請參閱以下的 [Runbook 參數](#Runbook-parameters)，以取得參數的進一步詳細資訊。
5.	[**工作**] 分頁隨即開啟，因此您可以追蹤工作的狀態。

## 使用 Windows PowerShell 啟動 Runbook

您可以使用 [Start-AzureRmAutomationRunbook](https://msdn.microsoft.com/library/mt603661.aspx)，利用 Windows PowerShell 來啟動 Runbook。下列範例程式碼會啟動名稱為 Test-Runbook 的 Runbook。

```
Start-AzureRmAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" -ResourceGroupName "ResourceGroup01"
```

Start-AzureRmAutomationRunbook 會傳回工作物件，一旦啟動 Runbook，您即可用來追蹤其狀態。然後您可以使用此工作物件搭配 [Get-AzureRmAutomationJob](https://msdn.microsoft.com/library/mt619440.aspx) 來判斷工作的狀態，以及搭配 [Get-AzureRmAutomationJobOutput](https://msdn.microsoft.com/library/mt603476.aspx) 來取得其輸出。下列範例程式碼會啟動名稱為 Test-Runbook 的 Runbook，等到它完成，然後顯示其輸出。

```
$runbookName = "Test-Runbook"
$ResourceGroup = "ResourceGroup01"
$AutomationAcct = "MyAutomationAccount"

$job = Start-AzureRmAutomationRunbook –AutomationAccountName $AutomationAcct -Name $runbookName -ResourceGroupName $ResourceGroup

$doLoop = $true
While ($doLoop) {
   $job = Get-AzureRmAutomationJob –AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup
   $status = $job.Status
   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped"))
}

Get-AzureRmAutomationJobOutput –AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup –Stream Output
```

如果 Runbook 需要參數，則您必須提供其作為 [hashtable](http://technet.microsoft.com/library/hh847780.aspx)，其中雜湊表的索引鍵符合參數名稱，而值是參數值。下列範例顯示如何啟動 Runbook 具有名為 FirstName 和 LastName 的兩個參數、名為 RepeatCount 的整數，和名為 Show 的布林值參數。如需有關參數的詳細資訊，請參閱以下的 [Runbook 參數](#Runbook-parameters)。

```
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Start-AzureRmAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" -ResourceGroupName "ResourceGroup01" –Parameters $params
```

## Runbook 參數

從 Azure 入口網站或 Windows PowerShell 啟動 Runbook 時，指示會透過 Azure 自動化 Web 服務傳送。此服務不支援具有複雜資料型別的參數。如 [Azure 自動化中的子 Runbook](automation-child-runbooks.md) 中所述，如果您需要提供複雜參數的值，您必須從另一個 Runbook 呼叫它內嵌。

Azure 自動化 Web 服務會為特定資料型別的參數提供特殊功能，如下列小節所述。

### 具名值

如果參數是資料類型 [object]，則您可以使用下列 JSON 格式，對它傳送具名值清單：{Name1:'Value1', Name2:'Value2', Name3:'Value3'}。這些值必須是簡單型別。Runbook 將接收參數做為 [PSCustomObject](https://msdn.microsoft.com/library/system.management.automation.pscustomobject%28v=vs.85%29.aspx)，並具有對應到每個具名值的屬性。

請考慮可接受稱為 user 的參數的下列測試 Runbook。

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][object]$user
   )
    $userObject = $user | ConvertFrom-JSON
	if ($userObject.Show) {
        foreach ($i in 1..$userObject.RepeatCount) {
            $userObject.FirstName
            $userObject.LastName
        }
    }
}
```

下列文字可用於 user 參數。

```
{FirstName:'Joe',LastName:'Smith',RepeatCount:'2',Show:'True'}
```

這會導致下列輸出。

```
Joe
Smith
Joe
Smith
```

### 陣列

如果參數是陣列，例如 [array] 或 [string]，則您可以使用下列 JSON 格式對其傳送值清單：*[Value1,Value2,Value3]*。這些值必須是簡單型別。

請考慮可接受稱為 *user* 的參數的下列測試 Runbook。

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][array]$user
   )
    if ($user[3]) {
        foreach ($i in 1..$user[2]) {
            $ user[0]
            $ user[1]
        }
    }
}
```

下列文字可用於 user 參數。

```
["Joe","Smith",2,true]
```

這會導致下列輸出。

```
Joe
Smith
Joe
Smith
```

### 認證

如果參數是資料型別 **PSCredential**，則可以提供 Azure 自動化[認證資產](automation-credentials.md)的名稱。Runbook 將會使用您所指定的名稱擷取認證。

請考慮可接受稱為 credential 的參數的下列測試 Runbook。

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][PSCredential]$credential
   )
   $credential.UserName
}
```

假設有稱為 *My Credential* 的認證資產，則下列文字可用於 user 參數。

```
My Credential
```

假設在認證中的使用者名稱是 *jsmith*，這會導致下列輸出。

```
jsmith
```

## 後續步驟

-	目前文件中的 Runbook 架構提供在 Azure 中管理資源的 Runbook 整體概觀，並利用 Hybrid Runbook Worker 進行內部部署。若要了解如何在您的資料中心執行自動化 Runbook，請參閱[混合式 Runbook 背景工作](automation-hybrid-runbook-worker.md)。
-	若要深入了解如何建立模組化 Runbook，以供其他 Runbook 用於特定或一般函式，請參閱[子 Runbook](automation-child-runbooks.md)。

<!---HONumber=AcomDC_0921_2016-->