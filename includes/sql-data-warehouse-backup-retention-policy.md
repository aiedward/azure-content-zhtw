
<!--
includes/sql-data-warehouse-backup-retention-policies.md

Latest Freshness check:  2016-05-05 , barbkess.

As of circa 2016-04-22, the following topics might include this include:
articles/sql-data-warehouse/sql-data-warehouse-overview-expectations.md
articles/sql-data-warehouse/sql-data-warehouse-overview-backup-and-restore.md
-->
SQL Data Warehouse 會使用 Azure 儲存體快照集，至少每 8 小時快照一次所有即時資料。這些快照會保留 7 天。這可讓您在擷取最新快照集的過去 7 天內，就有一個至少 21 個時間點可用來還原資料。

SQL 資料倉儲會在卸除資料庫前擷取資料庫快照集，並將其保留 7 天。發生這種情況時，就不會再保留即時資料庫中的快照集。這可讓您將已刪除的資料庫還原到其刪除時的時間點。

<!---HONumber=AcomDC_0608_2016-->