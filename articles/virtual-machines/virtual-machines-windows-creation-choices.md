<properties
	pageTitle="建立 Windows VM 的不同方式 | Microsoft Azure"
	description="列出使用資源管理員建立 Windows 虛擬機器的不同方式。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="infrastructure-services"
	ms.date="07/06/2016"
	ms.author="cynthn"/>

# 使用資源管理員建立 Windows 虛擬機器的不同方式

Azure 提供建立虛擬機器的不同方式，因為虛擬機器適用於不同的使用者和用途。這表示您需要針對虛擬機器以及建立方式進行一些選擇。本文提供這些選項及指示連結的摘要說明。

## Azure 入口網站

使用 Azure 入口網站是嘗試設定虛擬機器的簡單方法，特別是在您剛開始使用 Azure 時。

[使用入口網站建立執行 Windows 的虛擬機器](virtual-machines-windows-hero-tutorial.md)

## 範本

虛擬機器需要各種資源 (例如可用性設定組和儲存體帳戶)。您不是分開部署與管理每個資源，而是建立一個 Azure Resource Manager 範本，藉此經由協調的單一作業來部署與佈建所有資源。

- [利用 Resource Manager 範本建立 Windows 虛擬機器](virtual-machines-windows-ps-template.md)


## Azure PowerShell

如果您偏好使用命令殼層，可以使用 Azure PowerShell。

- [使用 PowerShell 建立 Windows VM](virtual-machines-windows-ps-create.md)


## Visual Studio

使用 Visual Studio 搭配 Azure Tools for Visual Studio 和 Azure SDK 來建置、管理與部署 VM。

[Azure Tools for Visual Studio](https://www.visualstudio.com/features/azure-tools-vs)

<!---HONumber=AcomDC_0706_2016-->