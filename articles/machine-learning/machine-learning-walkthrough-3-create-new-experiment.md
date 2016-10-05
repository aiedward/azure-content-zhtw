<properties
	pageTitle="步驟 3：建立新的 Machine Learning 實驗 | Microsoft Azure"
	description="開發預測解決方案逐步解說的步驟 3：在 Azure Machine Learning Studio 中建立新的定型實驗。"
	services="machine-learning"
	documentationCenter=""
	authors="garyericson"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016" 
	ms.author="garye"/>


# 逐步解說步驟 3：建立新的 Azure Machine Learning 實驗

這是[在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](machine-learning-walkthrough-develop-predictive-solution.md)逐步解說的第三個步驟


1.	[建立機器學習服務工作區](machine-learning-walkthrough-1-create-ml-workspace.md)
2.	[上傳現有資料](machine-learning-walkthrough-2-upload-data.md)
3.	**建立新實驗**
4.	[訓練及評估模型](machine-learning-walkthrough-4-train-and-evaluate-models.md)
5.	[部署 Web 服務](machine-learning-walkthrough-5-publish-web-service.md)
6.	[存取 Web 服務](machine-learning-walkthrough-6-access-web-service.md)

----------

本逐步解說的下一步驟是在 Machine Learning Studio 中建立一個新實驗，並在實驗中使用我們上傳的資料集。

1.	在 Studio 中，按一下視窗底部的 [+新增]。
2.	選取 [實驗]，然後選取 [空白實驗]。選取畫布頂端的預設實驗名稱，然後將它重新命名為有意義的名稱。

	> [AZURE.TIP] 在 [**屬性**] 窗格中為實驗填入**摘要**和**描述**是不錯的作法。這些屬性會提供記錄實驗的機會，以便稍後查看它的人員能了解您的目標和方法。

3.	在實驗畫布左側的模組調色盤中，展開 [Saved Datasets]。
4.	尋找您在**我的資料集**下建立的資料集並將它拖曳到畫布上。您也可以在調色盤上方的 [搜尋] 方塊中輸入名稱，以尋找資料集。

## 準備資料

您可以按一下資料集的輸出連接埠 (底部的小圓圈)，然後選取 [視覺化]，來檢視整個資料集的前 100 列資料和部分統計資訊。

因為資料檔案並未隨附資料行標題，所以 Studio 已提供一般標題 (Col1、Col2 等等)。建立模型並不需要良好的標題，但良好的標題可讓您更容易使用實驗中的資料。此外，當我們最終在 Web 服務中發佈此模型時，標題將有助於服務的使用者識別資料行。

我們可以使用[編輯中繼資料][edit-metadata]模組來新增資料行標題。您需使用[編輯中繼資料][edit-metadata]模組來變更與資料集關聯的中繼資料。透過這種方式，可提供資料行標題更易記的名稱。

若要使用[編輯中繼資料][edit-metadata]，您必須先指定要修改的資料行 (在此案例中為全部)，接著指定要在這些資料行執行的動作 (在此案例中為變更資料行標題)。

1.	在模組調色盤的 [搜尋] 方塊中，輸入 "metadata"。您會看見[編輯中繼資料][edit-metadata]出現在模組清單中。
2.	按一下 [[編輯中繼資料][edit-metadata]] 模組並拖曳到畫布上，將它放在我們稍早新增的資料集下。
3.	將資料集連接到 [[編輯中繼資料][edit-metadata]]：按一下資料集的輸出連接埠 (資料集底部的小圓圈)、拖曳到[編輯中繼資料][edit-metadata]的輸入連接埠 (模組頂端的小圓圈)，然後放開滑鼠按鈕。即使您在畫布上移動資料集或模組，這兩者仍會保持連線。

    實驗目前看起來如下：

    ![新增編輯中繼資料][2]
    
    紅色驚嘆號標示表示我們尚未設定此模組的屬性。我們接下來會這樣做。
    
    > [AZURE.TIP] 您可以按兩下模組並輸入文字，為模組新增註解。這有助於您快速檢視模組在您實驗中的執行情況。在此案例中，請按兩下 [[編輯中繼資料][edit-metadata]] 模組，然後輸入註解「新增資料行標題」。按一下畫布上其他的任何地方來關閉文字方塊。按一下模組的向下箭號以顯示註解。

4.	選取 [[編輯中繼資料][edit-metadata]]，然後在畫布右邊的 [屬性] 窗格中，按一下 [啟動資料行選取器]。
5.	在 [選取資料行] 對話方塊中，選取 [可用資料行] 中的所有資料列，然後按一下 > 將它們移到 [選取的資料行]。對話方塊應該會看起來如下：![已選取所有資料行的資料行選取器][4]
7.	按一下 [**確定**] 核取記號。
8.	回到 [**屬性**] 窗格，找到 [**新增資料行名稱**] 參數。在這個欄位中，輸入資料集中 21 個資料行的名稱清單，並以逗號加以分隔，且依資料行順序排序。您可以從 UCI 網站上的資料集文件中取得資料行名稱，或為求簡便，您可以複製並貼上下列清單：

		  Status of checking account, Duration in months, Credit history, Purpose, Credit amount, Savings account/bond, Present employment since, Installment rate in percentage of disposable income, Personal status and sex, Other debtors, Present residence since, Property, Age in years, Other installment plans, Housing, Number of existing credits, Job, Number of people providing maintenance for, Telephone, Foreign worker, Credit risk  

    [屬性] 窗格將如下所示：

    ![編輯中繼資料的屬性][1]

> [AZURE.TIP] 如果您想要確認資料行標題，請執行實驗 (按一下實驗畫布下方的 [執行])。當執行結束 ([編輯中繼資料][edit-metadata]上會出現綠色的勾選記號) 時，按一下[編輯中繼資料][edit-metadata]模組的輸出連接埠，然後選取 [視覺化]。您可以用相同方式檢視任何模組的輸出，以檢視資料在實驗中的執行進度。

## 建立訓練和測試資料集

實驗的下一個步驟，是產生將用來訓練及測試模型的個別資料集。

為此，我們將使用[資料分割][split]模組。

1.	找出[資料分割][split]模組，將它拖曳到畫布上，然後連接到最後一個[編輯中繼資料][edit-metadata]模組。
2.	根據預設，分割率會是 0.5，並且會設定 [Randomized split] 參數。這表示有一半資料會隨機透過[資料分割][split]模組的一個連接埠輸出，一半透過另一個連接埠輸出。您可以調整這些參數和 [隨機種子] 參數，以變更訓練與測試資料之間的分割。在此範例中，我們會將其保留為原狀。
	> [AZURE.TIP] [第一個輸出資料集內的資料列比例] 屬性決定有多少資料透過左側輸出連接埠輸出。例如，如果您將分割率設為 0.7，則會有 70% 的資料透過左側連接埠輸出，30% 透過右側連接埠輸出。
3. 按兩下 [[資料分割][split]] 模組，並輸入註解「訓練/測試資料分割 50%」。

我們可以依自身需求使用[資料分割][split]模組的輸出，但在範例中，我們要選擇以左側輸出作為訓練資料，右側輸出作為測試資料。

如 UCI 網站上所說明，將高信用風險誤判為低風險的成本，會比將低信用風險誤判為高風險的成本多出 5 倍。為了說明這一點，我們將產生新資料集以反映此成本函數。在此新資料集中，每個高風險範例都會複寫 5 次，而每個低風險範例則不複寫。

我們可以使用 R 程式碼來執行此複寫：

1.	尋找[執行 R 指令碼][execute-r-script]模組並拖曳到實驗畫布上，並且將[資料分割][split]模組的左輸出連接埠連接至[執行 R 指令碼][execute-r-script]模組的第一個輸入連接埠 (「資料集1」)。
2. 按兩下 [[執行 R 指令碼][execute-r-script]] 模組並輸入註解「設定成本調整」。
2.	在 [**屬性**] 窗格中，刪除 [**R 指令碼**] 參數中的預設文字，然後輸入此指令碼：

		  dataset1 <- maml.mapInputPort(1)
		  data.set<-dataset1[dataset1[,21]==1,]
		  pos<-dataset1[dataset1[,21]==2,]
		  for (i in 1:5) data.set<-rbind(data.set,pos)
		  maml.mapOutputPort("data.set")


我們必須為[資料分割][split]模組的每個輸出執行此一相同複寫作業，使訓練和測試資料具有相同的成本調整。

1.	以滑鼠右鍵按一下[執行 R 指令碼][execute-r-script]模組，然後選取 [複製]。
2.	以滑鼠右鍵按一下實驗畫布，然後選取 [貼上]。
3.	此[執行 R 指令碼][execute-r-script]模組的第一個輸入連接埠連接到[資料分割][split]模組的右側輸出連接埠。

> [AZURE.TIP] 「執行 R 指令碼」模組的複本會包含與原始模組相同的指令碼。當您在畫布上複製並貼上模組時，複本將會保留原本的所有屬性。

實驗此時看起來會如下所示：

![Adding Split module and R scripts][3]

如需如何在您的實驗中使用 R 指令碼的詳細資訊，請參閱[透過 R 擴充您的實驗](machine-learning-extend-your-experiment-with-r.md)。

**下一步：[定型和評估模型](machine-learning-walkthrough-4-train-and-evaluate-models.md)**


[1]: ./media/machine-learning-walkthrough-3-create-new-experiment/create1.png
[2]: ./media/machine-learning-walkthrough-3-create-new-experiment/create2.png
[3]: ./media/machine-learning-walkthrough-3-create-new-experiment/create3.png
[4]: ./media/machine-learning-walkthrough-3-create-new-experiment/columnselector.png


<!-- Module References -->
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[edit-metadata]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/

<!---HONumber=AcomDC_0921_2016-->