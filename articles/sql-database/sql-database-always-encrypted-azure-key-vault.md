<properties
	pageTitle="一律加密：透過資料庫加密保護 Azure SQL Database 中的機密資料 | Microsoft Azure"
	description="在數分鐘內開始保護您 SQL 資料庫中的機密資料。"
	keywords="資料加密, 加密金鑰, 雲端加密"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor="cgronlun"/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/18/2016"
	ms.author="sstein"/>

# 一律加密：保護 SQL Database 中的機密資料，並將加密金鑰儲存在「Azure 金鑰保存庫」中

> [AZURE.SELECTOR]
- [Azure 金鑰保存庫](sql-database-always-encrypted-azure-key-vault.md)
- [Windows 憑證存放區](sql-database-always-encrypted.md)


本文說明如何使用 [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/hh213248.aspx) 中的[一律加密精靈](https://msdn.microsoft.com/library/mt459280.aspx)，藉由資料加密來保護 SQL Database 中的機密資料。它也包含示範如何將每個加密金鑰儲存在「Azure 金鑰保存庫」中的指示。

「一律加密」是 Azure SQL Database 和 SQL Server 中新的資料加密技術，不論是當機密資料在伺服器上待用時、在用戶端與伺服器之間移動時，還是使用中時，都可協助保護資料。「一律加密」可確保機密資料在資料庫系統內一律不會以純文字顯示。設定資料加密之後，只有具備金鑰存取權的用戶端應用程式或應用程式伺服器才可以存取純文字資料。如需詳細資訊，請參閱[一律加密 (資料庫引擎)](https://msdn.microsoft.com/library/mt163865.aspx)。


將資料庫設定為使用「一律加密」之後，您將使用 Visual Studio 以 C# 建立用戶端應用程式來使用加密資料。

請依照本文章中的步驟操作，並了解如何為 Azure SQL Database 設定「一律加密」。在本文章中，您將學習到如何執行下列工作：

- 使用 SSMS 中的「一律加密」精靈來建立[一律加密的金鑰](https://msdn.microsoft.com/library/mt163865.aspx#Anchor_3)。
    - 建立[資料行主要金鑰 (CMK)](https://msdn.microsoft.com/library/mt146393.aspx)。
    - 建立[資料行加密金鑰 (CEK)](https://msdn.microsoft.com/library/mt146372.aspx)。
- 建立資料庫資料表並將資料行加密。
- 建立可插入、選取及顯示加密資料行資料的應用程式。


## 必要條件

針對本教學課程，您將需要：

- Azure 帳戶和訂用帳戶。如果您沒有帳戶，請註冊[免費試用](https://azure.microsoft.com/pricing/free-trial/)。
- [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 13.0.700.242 版或更新版本。
- [.NET Framework 4.6](https://msdn.microsoft.com/library/w0x726c2.aspx) 或更新版本 (於用戶端電腦上)。
- [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)。
- [Azure PowerShell](../powershell-install-configure.md) 1.0 或更新版本。請輸入 **(Get-Module azure -ListAvailable).Version** 來查看您正在執行的 PowerShell 版本。



## 讓用戶端應用程式能夠存取 SQL Database 服務

您必須讓用戶端應用程式能夠存取 SQL Database 服務，方法是設定必要的驗證，並取得在接下來的程式碼中驗證應用程式所需的「用戶端識別碼」和「密碼」。

1. 開啟 [Azure 傳統入口網站](http://manage.windowsazure.com)。
2. 選取 [Active Directory]，然後按一下您應用程式將會使用的 [Active Directory]。
3. 按一下 [應用程式]，然後按一下 [加入]。
4. 輸入您應用程式的名稱 (例如：*myClientApp*)，選取 [WEB 應用程式]，然後按一下箭號以繼續。
5. 針對 [登入 URL] 和 [應用程式識別碼 URI]，您可以輸入一個有效的 URL (例如：*http://myClientApp*)，然後繼續。
6. 按一下 [設定]。
7. 複製您的 [用戶端識別碼]。 \(您之後在程式碼中將需要此值)。
8. 在 [金鑰] 區段中，從 [選取持續時間] 下拉式清單中選取 [1 年]。(您將在儲存金鑰後，於步驟 14 複製金鑰)。
11. 向下捲動並按一下 [新增應用程式]。
12. 將 [顯示] 保留設定為 [Microsoft 應用程式]，然後選取 [Microsoft Azure 服務管理]。按一下核取記號以繼續。
13. 從 [委派權限] 下拉式清單中選取 [存取 Azure 服務管理]。
14. 按一下 [儲存]。
15. 完成儲存之後，複製 [金鑰] 區段中的金鑰值。(您之後在程式碼中將需要此值)。



## 建立金鑰保存庫來儲存您的金鑰

既然您的用戶端應用程式已完成設定，您也已取得用戶端識別碼，現在即可建立金鑰保存庫並設定其存取原則，以便讓您和您的應用程式能夠存取保存庫的密碼 (「一律加密」金鑰)。若要使用 SQL Server Management Studio 來建立新的資料行主要金鑰及設定加密，必須要有 *create*、*get*、*list*、*sign*、*verify*、*wrapKey* 及 *unwrapKey* 權限。

您可以執行下列指令碼來快速建立金鑰保存庫。如需這些 Cmdlet 的詳細說明，以及有關建立及設定金鑰保存庫的詳細資訊，請參閱[開始使用 Azure 金鑰保存庫](../Key-Vault/key-vault-get-started.md)。



    $subscriptionName = '<your Azure subscription name>'
    $userPrincipalName = '<username@domain.com>'
    $clientId = '<client ID that you copied in step 7 above>'
    $resourceGroupName = '<resource group name>'
    $location = '<datacenter location>'
    $vaultName = 'AeKeyVault'


    Login-AzureRmAccount
    $subscriptionId = (Get-AzureRmSubscription -SubscriptionName $subscriptionName).SubscriptionId
    Set-AzureRmContext -SubscriptionId $subscriptionId

    New-AzureRmResourceGroup –Name $resourceGroupName –Location $location
    New-AzureRmKeyVault -VaultName $vaultName -ResourceGroupName $resourceGroupName -Location $location

    Set-AzureRmKeyVaultAccessPolicy -VaultName $vaultName -ResourceGroupName $resourceGroupName -PermissionsToKeys create,get,wrapKey,unwrapKey,sign,verify,list -UserPrincipalName $userPrincipalName
    Set-AzureRmKeyVaultAccessPolicy  -VaultName $vaultName  -ResourceGroupName $resourceGroupName -ServicePrincipalName $clientId -PermissionsToKeys get,wrapKey,unwrapKey,sign,verify,list




## 建立空白 SQL Database
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 移至 [新增] > [資料 + 儲存體] > [SQL Database]。
3. 在新的或現有伺服器上建立名為 **Clinic** 的「空白」資料庫。如需有關如何在 Azure 入口網站中建立資料庫的詳細指示，請參閱[在幾分鐘內建立 SQL Database](sql-database-get-started.md)。

	![建立空白資料庫](./media/sql-database-always-encrypted-azure-key-vault/create-database.png)

在本教學課程稍後，您將會需要連接字串，因此在建立資料庫之後，請瀏覽至新的 Clinic 資料庫並複製連接字串。您可以隨時取得連接字串，但在 Azure 入口網站中很容易就可以複製它。

1. 按一下 [SQL Database] > [Clinic] > [顯示資料庫連接字串]。
2. 複製 **ADO.NET** 的連接字串。

	![複製連接字串](./media/sql-database-always-encrypted-azure-key-vault/connection-strings.png)


## 使用 SSMS 連接到資料庫

開啟 SSMS 並連接到包含實務課程資料庫的伺服器。


1. 開啟 SSMS。(移至 [連接] > [資料庫引擎] 以開啟 [連接到伺服器] 視窗 (如果此視窗未開啟))。
2. 輸入您的伺服器名稱和認證。可以在 SQL 資料庫刀鋒視窗上找到此伺服器名稱和稍早複製的連接字串。輸入完整的伺服器名稱，包括 *database.windows.net*。

	![複製連接字串](./media/sql-database-always-encrypted-azure-key-vault/ssms-connect.png)

如果 [新增防火牆規則] 視窗開啟，請登入 Azure，讓 SSMS 為您建立新的防火牆規則。


## 建立資料表

在本節中，您將建立資料表來保存病患的資料。它一開始並未加密 -- 您將在下一節中設定加密。

1. 展開 [資料庫]。
1. 在 [Clinic] 資料庫上按一下滑鼠右鍵，然後按一下 [新增查詢]。
2. 將下列 Transact-SQL (T-SQL) 貼到新的查詢視窗中並「執行」它。


        CREATE TABLE [dbo].[Patients](
         [PatientId] [int] IDENTITY(1,1),
         [SSN] [char](11) NOT NULL,
         [FirstName] [nvarchar](50) NULL,
         [LastName] [nvarchar](50) NULL,
         [MiddleName] [nvarchar](50) NULL,
         [StreetAddress] [nvarchar](50) NULL,
         [City] [nvarchar](50) NULL,
         [ZipCode] [char](5) NULL,
         [State] [char](2) NULL,
         [BirthDate] [date] NOT NULL
         PRIMARY KEY CLUSTERED ([PatientId] ASC) ON [PRIMARY] );
         GO


## 將資料行加密 (設定「一律加密」)

SSMS 提供一個精靈，可為您設定資料行主要金鑰、資料行加密金鑰及加密的資料行，來協助您輕鬆設定「一律加密」。

1. 展開 [資料庫] > [Clinic] > [資料表]。
2. 在 [Patients] 資料表上按一下滑鼠右鍵，然後選取 [加密資料行] 以開啟「一律加密精靈」：

    ![加密資料行](./media/sql-database-always-encrypted-azure-key-vault/encrypt-columns.png)

「一律加密」精靈包含下列區段︰[資料行選取]、[主要金鑰組態]、[驗證] 及 [摘要]。

### 資料行選取##

在 [簡介] 頁面上按 [下一步]，即可開啟 [資料行選取] 頁面。在此頁面上，您將選取要加密的資料行、[加密類型及要使用的資料行加密金鑰 (CEK)](https://msdn.microsoft.com/library/mt459280.aspx#Anchor_2)。

請加密每個病患的 **SSN** 和 **BirthDate** 資訊。SSN 資料行將使用決定性加密，這可支援等式查閱、聯結及群組依據。BirthDate 資料行將使用不支援操作的隨機加密。

將 SSN 資料行的 [加密類型] 設定為 [決定性]，並將 BirthDate 資料行設定為 [隨機化]。按 [下一步]。

![加密資料行](./media/sql-database-always-encrypted-azure-key-vault/column-selection.png)

### 主要金鑰組態###

您可以在 [主要金鑰組態] 頁面設定 CMK，以及選取要用來儲存 CMK 的金鑰存放區提供者。目前，您可以將 CMK 儲存在 Windows 憑證存放區、「Azure 金鑰保存庫」或硬體安全性模組 (HSM) 中。

本教學課程將說明如何將金鑰儲存在「Azure 金鑰保存庫」中。

1.     選取 [Azure 金鑰保存庫]。
1.     從下拉式清單中選取想要的金鑰保存庫。
1.     按 [下一步]。

![主要金鑰組態](./media/sql-database-always-encrypted-azure-key-vault/master-key-configuration.png)


### 驗證###

您現在可以加密資料行，或儲存為 PowerShell 指令碼以供日後執行。針對這個教學課程，請選取 [繼續以立即完成]，然後按 [下一步]。

### 摘要 ###

確認設定全都正確，然後按一下 [完成] 以完成 [一律加密] 的設定。


![摘要](./media/sql-database-always-encrypted-azure-key-vault/summary.png)


### 確認精靈的動作

完成精靈步驟之後，您的資料庫便已完成「一律加密」設定。精靈已執行下列動作：

- 建立資料行主要金鑰 (CMK) 並將它儲存在「Azure 金鑰保存庫」中。
- 建立資料行加密金鑰 (CMK) 並將它儲存在「Azure 金鑰保存庫」中。
- 設定選取的資料行以進行加密。Patients 資料表目前沒有任何資料，但在所選資料行中的所有現有資料現在都已加密。

您可以確認 SSMS 中是否已建立金鑰，方法是展開 [Clinic] > [安全性] > [一律加密的金鑰]。


## 建立搭配加密資料使用的用戶端應用程式

既然已設定好「一律加密」，您現在即可建置會在加密資料行上執行「插入」和「選取」動作的應用程式。

> [AZURE.IMPORTANT] 傳送純文字資料給具有「一律加密」資料行的伺服器時，您的應用程式必須使用 [SqlParameter](https://msdn.microsoft.com/library/system.data.sqlclient.sqlparameter.aspx) 物件。在不使用 SqlParameter 物件的情況下傳遞常值會導致例外狀況。

1. 開啟 Visual Studio，建立新的 C# 主控台應用程式。請確定您的專案是設定為 [.NET Framework 4.6] 或更新版本。
2. 將專案命名為 **AlwaysEncryptedConsoleAKVApp**，然後按一下 [確定]。![新的主控台應用程式](./media/sql-database-always-encrypted-azure-key-vault/console-app.png)
3. 移至 [工具] > [NuGet 封裝管理員] > [封裝管理員主控台]，以安裝下列 NuGet 封裝。

在 [封裝管理員主控台] 中執行下列兩行程式碼。

    Install-Package Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory



## 修改連接字串，以啟用 [一律加密]

本節說明如何在您的資料庫連接字串中啟用「一律加密」。


若要啟用「一律加密」，您必須將 **Column Encryption Setting** 關鍵字新增到您的連接字串中，並將它設定為 **Enabled**。

您可以直接在連接字串中進行此設定，或是使用 [SqlConnectionStringBuilder](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnectionstringbuilder.aspx) 來設定它。下一節中的範例應用程式會示範如何使用 **SqlConnectionStringBuilder**。



### 在連接字串中啟用 [一律加密]

在連接字串中加入下列關鍵字。

    Column Encryption Setting=Enabled


### 使用 SqlConnectionStringBuilder 來啟用一律加密

下列程式碼示範如何將 [SqlConnectionStringBuilder.ColumnEncryptionSetting](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnectionstringbuilder.columnencryptionsetting.aspx) 設定為 [Enabled](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnectioncolumnencryptionsetting.aspx) 來啟用「一律加密」。

    // Instantiate a SqlConnectionStringBuilder.
    SqlConnectionStringBuilder connStringBuilder =
       new SqlConnectionStringBuilder("replace with your connection string");

    // Enable Always Encrypted.
    connStringBuilder.ColumnEncryptionSetting =
       SqlConnectionColumnEncryptionSetting.Enabled;

## 註冊 Azure 金鑰保存庫提供者

下列程式碼示範如何使用 ADO.NET 驅動程式來註冊「Azure 金鑰保存庫」提供者。

    private static ClientCredential _clientCredential;

    static void InitializeAzureKeyVaultProvider()
    {
       _clientCredential = new ClientCredential(clientId, clientSecret);

       SqlColumnEncryptionAzureKeyVaultProvider azureKeyVaultProvider =
          new SqlColumnEncryptionAzureKeyVaultProvider(GetToken);

       Dictionary<string, SqlColumnEncryptionKeyStoreProvider> providers =
          new Dictionary<string, SqlColumnEncryptionKeyStoreProvider>();

       providers.Add(SqlColumnEncryptionAzureKeyVaultProvider.ProviderName, azureKeyVaultProvider);
       SqlConnection.RegisterColumnEncryptionKeyStoreProviders(providers);
    }



## 一律加密範例主控台應用程式

這個範例會示範如何：

- 修改連接字串，以啟用 [一律加密]。
- 將「Azure 金鑰保存庫」註冊為應用程式的金鑰存放區提供者。
- 將資料插入加密資料行。
- 在加密資料行中篩選特定值來選取記錄。

以下列程式碼取代 **Program.cs** 的內容。從 Azure 入口網站，針對 Main 方法前一行中的全域 connectionString 變數，使用有效的連接字串來取代其連接字串。這是此程式碼唯一需要進行的變更。

執行應用程式以查看「一律加密」的運作情況。

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using System.Data;
    using System.Data.SqlClient;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider;

    namespace AlwaysEncryptedConsoleAKVApp
    {
    class Program
    {
        // Update this line with your Clinic database connection string from the Azure portal.
        static string connectionString = @"<connection string from the portal>";
        static string clientId = @"<client id from step 7 above>";
        static string clientSecret = "<key from step 13 above>";


        static void Main(string[] args)
        {
            InitializeAzureKeyVaultProvider();

            Console.WriteLine("Signed in as: " + _clientCredential.ClientId);

            Console.WriteLine("Original connection string copied from the Azure portal:");
            Console.WriteLine(connectionString);

            // Create a SqlConnectionStringBuilder.
            SqlConnectionStringBuilder connStringBuilder =
                new SqlConnectionStringBuilder(connectionString);

            // Enable Always Encrypted for the connection.
            // This is the only change specific to Always Encrypted
            connStringBuilder.ColumnEncryptionSetting =
                SqlConnectionColumnEncryptionSetting.Enabled;

            Console.WriteLine(Environment.NewLine + "Updated connection string with Always Encrypted enabled:");
            Console.WriteLine(connStringBuilder.ConnectionString);

            // Update the connection string with a password supplied at runtime.
            Console.WriteLine(Environment.NewLine + "Enter server password:");
            connStringBuilder.Password = Console.ReadLine();


            // Assign the updated connection string to our global variable.
            connectionString = connStringBuilder.ConnectionString;


            // Delete all records to restart this demo app.
            ResetPatientsTable();

            // Add sample data to the Patients table.
            Console.Write(Environment.NewLine + "Adding sample patient data to the database...");

            InsertPatient(new Patient()
            {
                SSN = "999-99-0001",
                FirstName = "Orlando",
                LastName = "Gee",
                BirthDate = DateTime.Parse("01/04/1964")
            });
            InsertPatient(new Patient()
            {
                SSN = "999-99-0002",
                FirstName = "Keith",
                LastName = "Harris",
                BirthDate = DateTime.Parse("06/20/1977")
            });
            InsertPatient(new Patient()
            {
                SSN = "999-99-0003",
                FirstName = "Donna",
                LastName = "Carreras",
                BirthDate = DateTime.Parse("02/09/1973")
            });
            InsertPatient(new Patient()
            {
                SSN = "999-99-0004",
                FirstName = "Janet",
                LastName = "Gates",
                BirthDate = DateTime.Parse("08/31/1985")
            });
            InsertPatient(new Patient()
            {
                SSN = "999-99-0005",
                FirstName = "Lucy",
                LastName = "Harrington",
                BirthDate = DateTime.Parse("05/06/1993")
            });


            // Fetch and display all patients.
            Console.WriteLine(Environment.NewLine + "All the records currently in the Patients table:");

            foreach (Patient patient in SelectAllPatients())
            {
                Console.WriteLine(patient.FirstName + " " + patient.LastName + "\tSSN: " + patient.SSN + "\tBirthdate: " + patient.BirthDate);
            }

            // Get patients by SSN.
            Console.WriteLine(Environment.NewLine + "Now lets locate records by searching the encrypted SSN column.");

            string ssn;

            // This very simple validation only checks that the user entered 11 characters.
            // In production be sure to check all user input and use the best validation for your specific application.
            do
            {
                Console.WriteLine("Please enter a valid SSN (ex. 999-99-0003):");
                ssn = Console.ReadLine();
            } while (ssn.Length != 11);

            // The example allows duplicate SSN entries so we will return all records
            // that match the provided value and store the results in selectedPatients.
            Patient selectedPatient = SelectPatientBySSN(ssn);

            // Check if any records were returned and display our query results.
            if (selectedPatient != null)
            {
                Console.WriteLine("Patient found with SSN = " + ssn);
                Console.WriteLine(selectedPatient.FirstName + " " + selectedPatient.LastName + "\tSSN: "
                    + selectedPatient.SSN + "\tBirthdate: " + selectedPatient.BirthDate);
            }
            else
            {
                Console.WriteLine("No patients found with SSN = " + ssn);
            }

            Console.WriteLine("Press Enter to exit...");
            Console.ReadLine();
        }


        private static ClientCredential _clientCredential;

        static void InitializeAzureKeyVaultProvider()
        {

            _clientCredential = new ClientCredential(clientId, clientSecret);

            SqlColumnEncryptionAzureKeyVaultProvider azureKeyVaultProvider =
              new SqlColumnEncryptionAzureKeyVaultProvider(GetToken);

            Dictionary<string, SqlColumnEncryptionKeyStoreProvider> providers =
              new Dictionary<string, SqlColumnEncryptionKeyStoreProvider>();

            providers.Add(SqlColumnEncryptionAzureKeyVaultProvider.ProviderName, azureKeyVaultProvider);
            SqlConnection.RegisterColumnEncryptionKeyStoreProviders(providers);
        }

        public async static Task<string> GetToken(string authority, string resource, string scope)
        {
            var authContext = new AuthenticationContext(authority);
            AuthenticationResult result = await authContext.AcquireTokenAsync(resource, _clientCredential);

            if (result == null)
                throw new InvalidOperationException("Failed to obtain the access token");
            return result.AccessToken;
        }

        static int InsertPatient(Patient newPatient)
        {
            int returnValue = 0;

            string sqlCmdText = @"INSERT INTO [dbo].[Patients] \([SSN], [FirstName], [LastName], [BirthDate])
     VALUES (@SSN, @FirstName, @LastName, @BirthDate);";

            SqlCommand sqlCmd = new SqlCommand(sqlCmdText);


            SqlParameter paramSSN = new SqlParameter(@"@SSN", newPatient.SSN);
            paramSSN.DbType = DbType.AnsiStringFixedLength;
            paramSSN.Direction = ParameterDirection.Input;
            paramSSN.Size = 11;

            SqlParameter paramFirstName = new SqlParameter(@"@FirstName", newPatient.FirstName);
            paramFirstName.DbType = DbType.String;
            paramFirstName.Direction = ParameterDirection.Input;

            SqlParameter paramLastName = new SqlParameter(@"@LastName", newPatient.LastName);
            paramLastName.DbType = DbType.String;
            paramLastName.Direction = ParameterDirection.Input;

            SqlParameter paramBirthDate = new SqlParameter(@"@BirthDate", newPatient.BirthDate);
            paramBirthDate.SqlDbType = SqlDbType.Date;
            paramBirthDate.Direction = ParameterDirection.Input;

            sqlCmd.Parameters.Add(paramSSN);
            sqlCmd.Parameters.Add(paramFirstName);
            sqlCmd.Parameters.Add(paramLastName);
            sqlCmd.Parameters.Add(paramBirthDate);

            using (sqlCmd.Connection = new SqlConnection(connectionString))
            {
                try
                {
                    sqlCmd.Connection.Open();
                    sqlCmd.ExecuteNonQuery();
                }
                catch (Exception ex)
                {
                    returnValue = 1;
                    Console.WriteLine("The following error was encountered: ");
                    Console.WriteLine(ex.Message);
                    Console.WriteLine(Environment.NewLine + "Press Enter key to exit");
                    Console.ReadLine();
                    Environment.Exit(0);
                }
            }
            return returnValue;
        }


        static List<Patient> SelectAllPatients()
        {
            List<Patient> patients = new List<Patient>();


            SqlCommand sqlCmd = new SqlCommand(
              "SELECT [SSN], [FirstName], [LastName], [BirthDate] FROM [dbo].[Patients]",
                new SqlConnection(connectionString));


            using (sqlCmd.Connection = new SqlConnection(connectionString))

            using (sqlCmd.Connection = new SqlConnection(connectionString))
            {
                try
                {
                    sqlCmd.Connection.Open();
                    SqlDataReader reader = sqlCmd.ExecuteReader();

                    if (reader.HasRows)
                    {
                        while (reader.Read())
                        {
                            patients.Add(new Patient()
                            {
                                SSN = reader[0].ToString(),
                                FirstName = reader[1].ToString(),
                                LastName = reader["LastName"].ToString(),
                                BirthDate = (DateTime)reader["BirthDate"]
                            });
                        }
                    }
                }
                catch (Exception ex)
                {
                    throw;
                }
            }

            return patients;
        }


        static Patient SelectPatientBySSN(string ssn)
        {
            Patient patient = new Patient();

            SqlCommand sqlCmd = new SqlCommand(
                "SELECT [SSN], [FirstName], [LastName], [BirthDate] FROM [dbo].[Patients] WHERE [SSN]=@SSN",
                new SqlConnection(connectionString));

            SqlParameter paramSSN = new SqlParameter(@"@SSN", ssn);
            paramSSN.DbType = DbType.AnsiStringFixedLength;
            paramSSN.Direction = ParameterDirection.Input;
            paramSSN.Size = 11;

            sqlCmd.Parameters.Add(paramSSN);


            using (sqlCmd.Connection = new SqlConnection(connectionString))
            {
                try
                {
                    sqlCmd.Connection.Open();
                    SqlDataReader reader = sqlCmd.ExecuteReader();

                    if (reader.HasRows)
                    {
                        while (reader.Read())
                        {
                            patient = new Patient()
                            {
                                SSN = reader[0].ToString(),
                                FirstName = reader[1].ToString(),
                                LastName = reader["LastName"].ToString(),
                                BirthDate = (DateTime)reader["BirthDate"]
                            };
                        }
                    }
                    else
                    {
                        patient = null;
                    }
                }
                catch (Exception ex)
                {
                    throw;
                }
            }
            return patient;
        }


        // This method simply deletes all records in the Patients table to reset our demo.
        static int ResetPatientsTable()
        {
            int returnValue = 0;

            SqlCommand sqlCmd = new SqlCommand("DELETE FROM Patients");
            using (sqlCmd.Connection = new SqlConnection(connectionString))
            {
                try
                {
                    sqlCmd.Connection.Open();
                    sqlCmd.ExecuteNonQuery();

                }
                catch (Exception ex)
                {
                    returnValue = 1;
                }
            }
            return returnValue;
        }
    }

    class Patient
    {
        public string SSN { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime BirthDate { get; set; }
    }
    }



## 確認資料已加密

您可以透過使用 SSMS 來查詢 Patients 資料 (使用尚未啟用「資料行加密設定」的目前連線)，快速檢查伺服器上的實際資料是否已加密。

在 Clinic 資料庫上執行下列查詢。

    SELECT FirstName, LastName, SSN, BirthDate FROM Patients;

您可以看到加密的資料行不包含任何純文字資料。

   ![新的主控台應用程式](./media/sql-database-always-encrypted-azure-key-vault/ssms-encrypted.png)


若要使用 SSMS 來存取純文字資料，您可以將 *Column Encryption Setting=enabled* 參數新增到連線中。

1. 在 SSMS 中，於 [物件總管] 中您的伺服器上按一下滑鼠右鍵，然後選擇 [中斷連線]。
2. 按一下 [連接] > [資料庫引擎] 以開啟 [連接到伺服器] 視窗，然後按一下 [選項]。
3. 按一下 [其他連接參數]，然後輸入 **Column Encryption Setting=enabled**。

	![新的主控台應用程式](./media/sql-database-always-encrypted-azure-key-vault/ssms-connection-parameter.png)

4. 在 Clinic 資料庫上執行下列查詢。

        SELECT FirstName, LastName, SSN, BirthDate FROM Patients;

     您現在可以在加密的資料行中看到純文字資料。


	![新的主控台應用程式](./media/sql-database-always-encrypted-azure-key-vault/ssms-plaintext.png)


## 後續步驟
建立使用「一律加密」的資料庫之後，您可以執行下列操作：

- [輪替和清除金鑰](https://msdn.microsoft.com/library/mt607048.aspx)。
- [移轉已經透過一律加密來加密的資料](https://msdn.microsoft.com/library/mt621539.aspx)。


## 相關資訊

- [一律加密 (用戶端開發)](https://msdn.microsoft.com/library/mt147923.aspx)
- [透明資料加密](https://msdn.microsoft.com/library/bb934049.aspx)
- [SQL Server 加密](https://msdn.microsoft.com/library/bb510663.aspx)
- [一律加密精靈](https://msdn.microsoft.com/library/mt459280.aspx)
- [一律加密部落格](http://blogs.msdn.com/b/sqlsecurity/archive/tags/always-encrypted/)

<!---HONumber=AcomDC_0824_2016-->