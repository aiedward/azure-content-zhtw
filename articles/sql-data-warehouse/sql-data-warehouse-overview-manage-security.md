<properties
   pageTitle="保護 SQL 資料倉儲中的資料庫 | Microsoft Azure"
   description="保護 Azure SQL 資料倉儲中的資料庫以便開發解決方案的秘訣。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="ronortloff"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="08/30/2016"
   ms.author="rortloff;barbkess;sonyama"/>

# 保護 SQL 資料倉儲中的資料庫

> [AZURE.SELECTOR]
- [安全性概觀](sql-data-warehouse-overview-manage-security.md)
- [威脅偵測](sql-data-warehouse-security-threat-detection.md)
- [稽核概觀](sql-data-warehouse-auditing-overview.md)
- [稽核下層用戶端](sql-data-warehouse-auditing-downlevel-clients.md)
- [透明資料加密 (入口網站)](sql-data-warehouse-encryption-tde.md)
- [透明資料加密 (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)

本文逐步解說保護 Azure SQL 資料倉儲資料庫的基本概念。本文尤其著重於協助您開始利用資源，在資料庫上限制存取、保護資料，以及監視活動。

## 連接安全性

「連線安全性」是指如何使用防火牆規則和連線加密，限制和保護資料庫的連線。

伺服器和資料庫兩者都可使用防火牆規則，拒絕來自未明確設為允許清單的 IP 位址的連線嘗試。若要允許來自應用程式或用戶端機器的公用 IP 位址的連接，您必須先使用 Azure 入口網站、REST API 或 PowerShell 建立伺服器層級的防火牆規則。最好的作法是，您應該盡可能限制允許穿透您伺服器防火牆的 IP 位址範圍。若要從您的本機電腦存取 Azure SQL 資料倉儲，請確定您的網路和本機電腦上的防火牆允許 TCP 連接埠 1433 上的傳出通訊。如需詳細資訊，請參閱 [Azure SQL Database 防火牆][]、[sp\_set\_firewall\_rule][] 和 [sp\_set\_database\_firewall\_rule][]。

預設會加密與 SQL 資料倉儲的連線。停用加密的修改連線設定會被忽略。

## 驗證

「驗證」是指連線到資料庫時如何證明身分識別。SQL 資料倉儲目前支援使用使用者名稱和密碼的 SQL Server 驗證，以及 Azure Active Directory。

當您為資料庫建立邏輯伺服器時，採取使用者名稱和密碼指定了「伺服器管理員」登入。使用這些認證，您就可以透過 SQL Server 驗證，使用資料庫擁有者或 "dbo" 的身分驗證該伺服器上的任何資料庫。

不過，最佳作法是，貴組織的使用者應該使用不同的帳戶來驗證。因為萬一應用程式的程式碼容易受到 SQL 插入式攻擊，您就可以限制授與應用程式的權限，並降低惡意活動的風險。

若要建立 SQL Server 驗證使用者，請使用伺服器管理員登入連接到您伺服器上的 **master** 資料庫，並建立新的伺服器登入。此外，在主要資料庫中建立一個使用者做為 Azure SQL 資料倉儲使用者是不錯的主意。在主要資料庫中建立使用者，使用者就能使用類似 SSMS 的工具登入而不用指定資料庫名稱。它也可讓使用者使用物件總管來檢視 SQL Server 上的所有資料庫。

```sql
-- Connect to master database and create a login
CREATE LOGIN ApplicationLogin WITH PASSWORD = 'Str0ng_password';
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;
```

然後使用您的伺服器管理員登入，連接到 **SQL 資料倉儲資料庫**，並根據您剛建立的伺服器登入建立資料庫使用者。

```sql
-- Connect to SQL DW database and create a database user
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;
```

如果使用者將會進行其他作業，例如建立登入或建立新資料庫，則也需要為他們指派主要資料庫中的 `Loginmanager` 和 `dbmanager` 角色。如需有關這些額外角色及向 SQL Database 驗證的詳細資訊，請參閱[管理 Azure SQL Database 中的資料庫和登入][]。如需更多將 Azure AD 用於 SQL 資料倉儲的詳細資料，請參閱[使用 Azure Active Directory 驗證連線到 SQL 資料倉儲][]。


## Authorization

「授權」是指在 Azure SQL 資料倉儲內可以執行的動作，這是由使用者帳戶的角色成員資格和權限所控制。最好的作法是，您應該授與使用者所需的最低權限。Azure SQL 資料倉儲可讓您輕鬆地透過 T-SQL 中的角色進行上述管理：

```sql
EXEC sp_addrolemember 'db_datareader', 'ApplicationUser'; -- allows ApplicationUser to read data
EXEC sp_addrolemember 'db_datawriter', 'ApplicationUser'; -- allows ApplicationUser to write data
```

您所連線的伺服器管理員帳戶是 db\_owner 的成員，有權限在資料庫中執行任何動作。請儲存此帳戶，以便部署結構描述升級及其他管理作業。請使用具更多有限權限的 "ApplicationUser" 帳戶，從應用程式連線到具應用程式所需之最低權限的資料庫。

有許多方式可以進一步限制使用者透過 Azure SQL Database 可以執行的動作：

- 細微的[權限][]可讓您控制您可以對資料庫中個別資料行、資料表、檢視、程序和其他物件執行哪些作業。使用細微權限，以擁有最大控制權，並授與所需的最小權限。細微權限系統有點複雜，而且需要進行一些研究才能有效地使用。
- 除了 db\_datareader 和 db\_datawriter 以外，[資料庫角色][]均可以用來建立權力較大的應用程式使用者帳戶，或權力較小的管理帳戶。內建固定資料庫角色提供簡單的方式來授與權限，但可能會導致授與的權限多於所需的權限。
- [預存程序][]可用來限制對資料庫可採取的動作。

要從 Azure 傳統入口網站或使用 Azure 資源管理員 API 管理資料庫和邏輯伺服器，是由入口網站使用者帳戶的角色指派所控制。如需有關此主題的詳細資訊，請參閱 [Azure 入口網站中的角色型存取控制][]。

## 加密

Azure SQL 資料倉儲可以使用[透明資料加密][]，透過加密「靜止」時的資料，或儲存在資料庫檔案和備份中的資料，來協助保護您的資料。您必須是系統管理員或 master 資料庫中 dbmanager 角色的成員，才能啟用 TDE。若要加密您的資料庫，請連接到您伺服器上的主要資料庫並執行：


```sql
ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;
```

您也可以從 [Azure 入口網站][]中的資料庫設定，啟用透明資料加密。如需詳細資訊，請參閱[開始使用透明資料加密 (TDE)][]。

## 稽核

稽核和追蹤資料庫事件可協助您遵循法規，並找出可疑的活動。SQL 資料倉儲稽核可讓您將資料庫中的事件記錄到 Azure 儲存體帳戶中的稽核記錄。SQL 資料倉儲稽核也整合了 Microsoft Power BI，具備向下鑽研報表和分析的功能。如需詳細資訊，請參閱[開始使用 Azure Database 稽核][]。

## 後續步驟

如需使用不同通訊協定連接到 SQL 資料倉儲的詳細資料和範例，請參閱[連接到 SQL 資料倉儲][]。

<!--Image references-->

<!--Article references-->
[連接到 SQL 資料倉儲]: ./sql-data-warehouse-connect-overview.md
[開始使用 Azure Database 稽核]: ./sql-data-warehouse-auditing-overview.md
[開始使用透明資料加密 (TDE)]: ./sql-data-warehouse-encryption-tde.md
[使用 Azure Active Directory 驗證連線到 SQL 資料倉儲]: ./sql-data-warehouse-authentication.md

<!--MSDN references-->
[Azure SQL Database 防火牆]: https://msdn.microsoft.com/library/ee621782.aspx
[sp\_set\_firewall\_rule]: https://msdn.microsoft.com/library/dn270017.aspx
[sp\_set\_database\_firewall\_rule]: https://msdn.microsoft.com/library/dn270010.aspx
[資料庫角色]: https://msdn.microsoft.com/library/ms189121.aspx
[管理 Azure SQL Database 中的資料庫和登入]: https://msdn.microsoft.com/library/ee336235.aspx
[權限]: https://msdn.microsoft.com/library/ms191291.aspx
[預存程序]: https://msdn.microsoft.com/library/ms190782.aspx
[透明資料加密]: https://go.microsoft.com/fwlink/?LinkId=526242
[Azure 入口網站]: https://portal.azure.com/

<!--Other Web references-->
[Azure 入口網站中的角色型存取控制]: https://azure.microsoft.com/documentation/articles/role-based-access-control-configure

<!----HONumber=AcomDC_0907_2016-->