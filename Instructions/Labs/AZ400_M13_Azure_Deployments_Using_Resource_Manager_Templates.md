---
lab:
    title: 'ラボ: Resource Manager テンプレートを使用した Azure デプロイ'
    module: 'モジュール 13: Azure ツールを使用したインフラストラクチャと構成の管理'
---

# ラボ: Resource Manager テンプレートを使用した Azure デプロイ
# 学生用ラボ マニュアル

## ラボの概要

このラボでは、Azure Resource Manager (ARM) テンプレートを作成し、リンクされたテンプレートのコンセプトを使ってこれをモジュラー化します。その後、主要なデプロイ テンプレートを修正し、リンクされたテンプレートと更新された依存関係を呼び出し、最終的にテンプレートを Azure にデプロイします。

## 目標

このラボを完了すると、次のことができるようになります。

- Resource Manager テンプレートの作成
- ストレージ リソース向けのリンク済みテンプレートを作成する
- リンク済みテンプレートを Azure Blob Storage にアップロードして SAS トークンを生成する
- メイン テンプレートを変更して、リンク済みテンプレートを呼び出す
- メイン テンプレートを変更して依存関係を更新する
- リンク済みテンプレートを使用してリソースを Azure にデプロイする

## ラボの所要時間

-   推定時間: **60 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/).このラボでは前提条件の一部としてインストールされます。 

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。この中には、Visual Studio Code が含まれます。

#### タスク 1: Git と Visual Studio Code をインストールして構成する

このタスクでは、Visual Studio Code をインストールします。この前提条件をすでに実装している場合は、直接、次のタスクに進むことができます。

1.  まだ Visual Studio Code がインストールされていない場合は、ラボのコンピューターから Web ブラウザーを起動し、[Visual Studio Code ダウンロード ページ](https://code.visualstudio.com/) に移動し、これをダウンロードしてインストールします。 

### 演習 1: Azure Resource Manager テンプレートを作成してデプロイする

このラボでは、Azure Resource Manager テンプレートを作成し、リンクされたテンプレートを使ってこれをモジュラー化します。その後、主要なデプロイ テンプレートを修正し、リンクされたテンプレートと更新された依存関係を呼び出し、最終的にテンプレートを Azure にデプロイします。

#### タスク 1: Resource Manager テンプレートの作成

このタスクでは、Visual Studio Code を使用して Resource Manager テンプレートを作成します

1.  ラボのコンピューターから Visual Studio Code を起動し、Visual Studio Code で 「**ファイル**」 トップ レベル メニューをクリックします。ドロップダウン メニューで 「**基本設定**」 を選択します。カスケード メニューで 「**拡張機能**」 を選択し、「**拡張機能の選択**」 テキストボックスに「**Azure Resource Manager (ARM) ツール**」と入力します。該当する検索結果を選択し、「**インストール**」 をクリックして Azure Resource Manager ツールをインストールします。
1.  Visual Studio Code で 「**ファイル**」 トップ レベル メニューをクリックし、ドロップダウン メニューで 「**ファイルを開く**」 を選択します。「ファイルを開く」 ダイアログ ボックスに URL (**https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json**) を入力し、「**開く**」 をクリックします。

    > **注**: テンプレートを最初から作成するよりも、[Azure クイックスタート テンプレート](https://azure.microsoft.com/ja-jp/resources/templates/) のひとつ (**シンプルな Windows テンプレート VM のデプロイ**) を使用します。テンプレートは GitHub からダウンロードできます ([101-vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows))。

1.  ラボのコンピューターでエクスプローラーを開き、テンプレートの格納で使用する以下のローカル フォルダーを作成します。

    - **C:\\templates** 
    - **C:\\templates\\storage** 

1.  azuredeploy.json テンプレートのある Visual Studio Code ウィンドウに戻り、「**ファイル**」 トップ レベル メニューをクリックします。ドロップダウン メニューで 「**名前を付けて保存**」 をクリックし、新しく作成されたローカル フォルダー **C:\\templates** でテンプレートを **azuredeploy.json** として保存します。
1.  テンプレートをレビューし、その構造をよりよく把握します。テンプレートには 5 種類のリソースが含まれています。

    - Microsoft.Storage/storageAccounts
    - Microsoft.Network/publicIPAddresses
    - Microsoft.Network/virtualNetworks
    - Microsoft.Network/networkInterfaces
    - Microsoft.Compute/virtualMachines

1.  Visual Studio Code でファイルを再び保存しますが、今回は保存先として **C:\\templates\\storage**、ファイル名として **storage.json** を選択します。

    > **注**: これで 2 つの同一の JSON ファイルができました。**C:\\templates\\azuredeploy.json** と **C:\\templates\\storage\\storage.json** です。

#### タスク 2: ストレージ リソース向けのリンク済みテンプレートを作成する

このタスクでは、前のタスクで保存したテンプレートを変更し、リンク済みのストレージ テンプレート「**storage.json**」がストレージ アカウントのみを作成し、最初のテンプレートで実行が起動されるようにします。リンク済みのストレージ テンプレートは、メイン テンプレート「**azuredeploy.json**」に値を戻す必要があります。この値は、リンク済みストレージ テンプレートの出力要素で定義されます。

1.  Visual Studio Code ウィンドウに表示されている **storage.json** ファイルの 「**リソース セクション**」 で、**storageAccounts** リソース以外のすべてのリソース要素を削除します。これによりリソース セクションは以下のようになるはずです。

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[variables('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2018-07-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage",
        "properties": {}
      }
    ],
    ```

1.  storageAccount の名前要素を変数からパラメーターに変更します

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[parameters('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2018-07-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage",
        "properties": {}
      }
    ],
    ```

1.  次に、変数セクション全体とあらゆる変数の定義を削除します:

    ```json
    "variables": {
      "storageAccountName": "[concat('bootdiags', uniquestring(resourceGroup().id))]",
      "nicName": "myVMNic",
      "addressPrefix": "10.0.0.0/16",
      "subnetName": "Subnet",
      "subnetPrefix": "10.0.0.0/24",
      "virtualNetworkName": "MyVNET",
      "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
      "networkSecurityGroupName": "default-NSG"
    },
    ```

1.  次に、場所以外のあらゆるパラメーター値を削除し、以下のパラメーター コードを追加すると、次のような結果になります:

    ```json
    "parameters": {
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for all resources."
        }
      },
        "storageAccountName":{
        "type": "string",
        "metadata": {
          "description": "Azure Storage account name."
        }
      }
    },
    ```

1.  次に、出力セクションを更新して、storageURI 出力値を定義します。メイン テンプレートの仮想マシン リソース定義では、storageUri 値が必要です。この値を、出力値としてメイン テンプレートに値を渡し返します。以下のようになるように出力を変更します。

    ```json
    "outputs": {
      "storageUri": {
        "type": "string",
        "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
      }
    }
    ```

1. 最後に、テンプレート スキーマのバージョンを 2015-01-01 から 2019-04-01 に更新します。テンプレート定義ファイルの最初の数行を以下のように更新してください。

    ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "storageAccountName":{
              "type": "string",
              "metadata": {
                "description": "Azure Storage account name."
              }
    ```

1. Storage.json テンプレートを保存します。リンク済みストレージ テンプレートは次のようになります:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccountName":{
          "type": "string",
          "metadata": {
            "description": "Azure Storage account name."
          }
        },
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        }
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[parameters('storageAccountName')]",
          "apiVersion": "2016-01-01",
          "location": "[parameters('location')]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": {}
        }
      ],
      "outputs": {
      "storageUri": {
        "type": "string",
        "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
        }
      }
    }
    ```

#### タスク 3: リンク済みテンプレートを Azure Blob Storage にアップロードして SAS トークンを生成する

このタスクでは、前のタスクで作成したリンク済みテンプレートを Azure Blob Storage にアップロードし、SAS トークンを生成して、その後のデプロイ中のアクセスを提供します。

> **注**: テンプレートにリンクする際、Azure Resource Manager サービスは http または https のいずれかを介してこれにアクセスできなくてはなりません。これを実行するため、リンク済みストレージ テンプレート「**storage.json**」を Azure の Blob ストレージにアップロードします。その後、デジタル署名された URL を生成します。これは、該当する BLOB への限定的なアクセスを提供します。これらのステップは、Azure Cloud Shell で Azure CLI を使用して実行します。また、Azure Portal を介して BLOB コンテナーを手動で作成し、ファイルをアップロードして URL を生成するか、ラボのコンピューターにインストールされている Azure CLI または Azure PowerShell のいずれかのモジュールを使用します。

1.  ラボのコンピューターで、Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。 

1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 

    > **注**: また、[Azure Cloud Shell](http://shell.azure.com) に直接移動することもできます。

1.  **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 

    > **注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。 

1.  Cloud Shell ペインの **PowerShell** セッションから、以下を実行して BLOB ストレージ コンテナーを作成し、前のタスクで作成したテンプレート ファイルをアップロードします。その後、メイン テンプレートで参照してリンク済みテンプレートにアクセスできるように SAS トークンを生成します。
1.  まず、以下のコードのラインをコピーして貼り付け、デプロイ先の Azure リージョンの値を設定します。プロンプトに示されているように、コマンドは入力を待ちます。

    ```powershell
    # Azure VM のプロビジョニングが可能で最も近い Azure の名前を提供
    $location = Read-Host -Prompt 'Enter the name of Azure region (i.e. centralus)'
    ```
1. 次に、以下のコードをコピーして同じ Cloud Shell セッションに貼り付け、BLOB ストレージ コンテナーを作成し、前のタスクで作成したテンプレート ファイルをアップロードします。その後、メイン テンプレートで参照してリンク済みテンプレートにアクセスできるように SAS トークンを生成します。

    ```powershell
    # これは Azure ストレージ アカウントに名前を割り当てるために使われるランダムな文字列
    $suffix = Get-Random
    $resourceGroupName = 'az400m13l01-RG'
    $storageAccountName = 'az400m13blob' + $suffix

    # 作成する BLOB コンテナーの名前
    $containerName = 'linktempblobcntr' 
    
    # このラボで使用される、完了したリンク済みテンプレート
    $linkedTemplateURL = "https://raw.githubusercontent.com/Microsoft/PartsUnlimited/master/Labfiles/AZ-400T05_Implementing_Application_Infrastructure/M01/storage.json" 

    # リンク済みテンプレートのダウンロードとアップロードに使用されるファイル名
    $fileName = 'storage.json' 
    
    # ラボのリンク済みテンプレートを Azure Cloud Shell ホーム ディレクトリにダウンロード
    Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$fileName" 
    
    # リソース グループの作成
    New-AzResourceGroup -Name $resourceGroupName -Location $location 
    
    # ストレージ アカウントの作成
    $storageAccount = New-AzStorageAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $storageAccountName `
      -Location $location `
      -SkuName 'Standard_LRS'
    
    $context = $storageAccount.Context
    
    # コンテナーの作成
    New-AzureStorageContainer -Name $containerName -Context $context
        
    # リンク済みテンプレートのアップロード
    Set-AzureStorageBlobContent `
      -Container $containerName `
      -File "$home/$fileName" `
      -Blob $fileName `
      -Context $context
    
    # SAS トークンの生成。24 時間の有効期限を設定しますが、セキュリティを強化するためにより短い値を使うこともできます。
    $templateURI = New-AzureStorageBlobSASToken `
      -Context $context `
      -Container $containerName `
      -Blob $fileName `
      -Permission r `
      -ExpiryTime (Get-Date).AddHours(24.0) `
      -FullUri
    
    "Resource Group Name: $resourceGroupName"
    "Linked template URI with SAS token: $templateURI"
    ```
    > **注**: スクリプトで生成された最終的な出力を必ず記録してください。これは、ラボの後半で必要になります。
    
    > **注**: 出力値は以下のようになるはずです:

    ```
    Resource Group Name: az400m13l01-RG
    Linked template URI with SAS token: https://az400m13blob1677205310.blob.core.windows.net/linktempblobcntr/storage.json?sv=2018-03-28&sr=b&sig=B4hDLt9rFaWHZXToJlMwMjejAQGT7x0INdDR9bHBQnI%3D&se=2020-11-23T21%3A54%3A53Z&sp=r
    ```

    > **注**: セキュリティのレベルを強化する必要がある場合は、メイン テンプレートのデプロイ中に SAS トークンをダイナミックに生成し、より短い有効期間を SAS トークンに割り当てることができます。

1.  「Cloud Shell」 ペインを閉じます。

#### タスク 4: メイン テンプレートを変更して、リンク済みテンプレートを呼び出す

このタスクでは、メイン テンプレートを変更し、前のタスクで Azure Blob Storage にアップロードされたリンク済みテンプレートを参照します。

> **注**: あらゆるストレージ要素をモジュラー化してテンプレート構造に加えた変更を説明するため、メイン テンプレートを変更して新しいストレージ リソース定義を呼び出す必要があります。

1.  Visual Studio Code で 「**ファイル**」 トップ レベル メニューをクリックし、ドロップダウン メニューで 「**ファイルを開く**」 を選択します。「ファイルを開く」 ダイアログ ボックスで **C:\\templates\\azuredeploy.json** に移動して選択し、「**開く**」 をクリックします。
1.  **Azuredeploy.json** ファイルのリソース セクションで、ストレージ リソース要素を削除します。

    ```json
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2018-07-01",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {}
    },
    ```

1.  次に、新しく削除されたストレージ リソース要素があった場所に直接、以下のコードを追加します:

    > **注**: `<linked_template_URI_with_SAS_token>` プレースホルダーは、前のタスクの最後で記録した実際の値に必ず置き換えてください。

    ```json
    {
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "properties": {
          "mode": "Incremental",
          "templateLink": {
              "uri":"<linked_template_URI_with_SAS_token>"
          },
          "parameters": {
              "storageAccountName":{"value": "[variables('storageAccountName')]"},
              "location":{"value": "[parameters('location')]"}
          }
       }
    },
    ```    

1.  メイン テンプレートで以下の詳細をレビューします:

    - メイン テンプレートの Microsoft.Resources/deployments リソースを使用して、別のテンプレートにリンクします。
    - デプロイ リソースには、linkedTemplate という名前が付いています。この名前は、依存関係を構成する場合に使用されます。
    - リンク済みテンプレートを呼び出す際は、増分デプロイ モードのみを使用できます。
    - templateLink/uri には、リンク済みテンプレートの URI が含まれています。
    - パラメーターを使用して、メイン テンプレートからリンク済みテンプレートに値を渡します。

1.  テンプレートを保存します。


#### タスク 5: メイン テンプレートを変更して依存関係を更新する

このタスクでは、メイン テンプレートを変更して、更新する必要のある残りの依存関係を説明します。

> **注**: ストレージ アカウントはリンク済みストレージ テンプレートで定義されているため、**Microsoft.Compute/virtualMachines** リソース定義を更新する必要があります。 

1.  仮想マシン要素のリソース セクションで、以下を置き換えることによって **dependsOn** 要素を更新します:

    ```json
    "dependsOn": [
      "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
      "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ```

    with

    ```json
    "dependsOn": [
     "linkedTemplate",
     "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ```

1.  **Microsoft.Compute/virtualMachines** 要素のリソース セクションで、**properties/diagnosticsProfile/bootDiagnostics/storageUri** 要素を再構成し、以下を置き換えることによってリンク済みストレージ テンプレートで定義された出力値を反映させます:

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
      }
    ```

    with

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference('linkedtemplate').outputs.storageUri.value]"
      }
    ```

1.  更新されたメイン デプロイ テンプレートを保存します。

#### タスク 6: リンク済みテンプレートを使用してリソースを Azure にデプロイする

> **注**: テンプレートは複数の方法でデプロイできます。Azure Portal から直接デプロイしたり、ローカルで、または Azure Cloud Shell からインストールされた Azure CLI や PowerShell を使用したりできます。このラボでは、Azure Cloud Shell から Azure CLI を使用します。  

> **注**: Azure Cloud Shell を使用するには、メイン デプロイ テンプレート「azuredeploy.json」を Cloud Shell のホーム ディレクトリにアップロードします。また、リンク済みテンプレートをアップロードした場合と同様に、Azure Blob Storage にアップロードし、ローカル ファイルのシステム パスではなく URI を使用して参照することもできます。

1.  ラボのコンピューターで、Azure Portal が表示されている Web ブラウザーで 「**Cloud Shell**」 アイコンをクリックして Cloud Shell を開きます。 
    > **注**: この演習で以前に使用した PowerShell セッションがまだアクティブな場合は、Bash に切り替えなくてもこれを使用できます (次のステップ)。Cloud Shell の PowerShell と Bash セッションの両方で以下のステップを実行できます。新しい Cloud Shell セッションを開く場合は手順に従ってください。 
1.  Cloud Shell ペインで 「**PowerShell**」 をクリックします。ドロップダウン メニューで 「**バッシュ**」 をクリックし、指示されたら 「**確認**」 をクリックします。 
1.  Cloud Shell ペインで、「**ファイルのアップロード / ダウンロード**」 アイコンをクリックし、ドロップダウン メニューで 「**アップロード**」 をクリックします。 
1.  「**開く**」 ダイアログ ボックスで、**C:\\templates\\azuredeploy.json** に移動してこれを選択し、「**開く**」 をクリックします。
1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、新しくアップロードされたテンプレートを使用してデプロイを実行します。

    ```bash
    az deployment group create --name az400m13l01deployment --resource-group az400m13l01-RG --template-file azuredeploy.json
    ```

1.  'adminUsername’ の値を提供するよう指示されたら、「**Student**」と入力して **Enter** キーを押します。
1.  'adminPassword' の値を提供するよう指示されたら、「**Pa55w.rd1234**」と入力して **Enter** キーを押します。

1.  テンプレートをデプロイする上記のコマンドの実行時にエラーが出た場合は以下を試してください。

    - 複数の Azure サブスクリプションがある場合は、リソース グループがデプロイされている適切な場所にサブスクリプションのコンテキストが設定されていることを確認してください。
    - 指定した URI を介してリンク済みテンプレートにアクセスできることを確認します。

> **注**: 次のステップでは、メイン デプロイ テンプレートで残りのリソース定義 (ネットワークや仮想マシンのリソース定義など) をモジュラー化できます。 

> **注**: デプロイされたリソースを使用する予定がない場合は、関連した料金が発生しないようにリソースを削除してください。リソース グループ「**az400m13l01-RG**」を削除するだけです。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```bash
    az group list --query "[?starts_with(name,'az400m13l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```bash
    az group list --query "[?starts_with(name,'az400m13l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Azure Resource Manager テンプレートを作成し、リンク済みテンプレートを使用してこれをモジュラー化し、メイン デプロイ テンプレートを変更してリンク済みテンプレートと更新済みの依存関係を呼び出し、最後にテンプレートを Azure にデプロイする方法を学びました。
