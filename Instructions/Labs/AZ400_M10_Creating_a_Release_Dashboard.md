---
lab:
    title: 'ラボ: リリース ダッシュボードの作成'
    module: 'モジュール 10: リリース戦略の設計'
---

# ラボ: リリース ダッシュボードの作成
# 学生用ラボ マニュアル

## ラボの概要

このラボでは、リリース ダッシュボードの作成と、REST API を使用して Azure DevOps リリース データを取得する手順を説明します。これにより、カスタム アプリケーションまたはダッシュボードでこの方法を利用できるようになります。 

ラボでは、Azure DevOps Starter リソースを活用します。このリソースは、アプリケーションをビルドして Azure にデプロイする Azure DevOps プロジェクトを自動的に作成します。 

## 目標

このラボを完了すると、次のことができるようになります。

- リリース ダッシュボードを作成する
- REST API を使用してリリース情報を取得する

## ラボの所要時間

-   推定時間: **45 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

次の資格情報を使用して、Windows 10 コンピューターにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### インストールされているアプリケーションを確認します

Windows 10 デスクトップでタスク バーを探します。タスク バーには、この課題で使用するアプリケーションのアイコンが含まれています:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)の手順に従って作成してください。

#### Azure サブスクリプションを準備する

-   既存の Azure サブスクリプションを特定するか、新しいサブスクリプションを作成します。
-   Azure サブスクリプションで所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。詳細については、[Azure ポータルを使用した Azure のロールの割り当ての一覧表示](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) を参照してください。

### 演習 1: リリース ダッシュボードを作成する

この演習では、Azure DevOps 組織にリリース ダッシュボードを作成します。

#### タスク 1: Azure DevOps スターター リソースを作成する

このタスクでは、Azure サブスクリプションに Azure DevOps スターター リソースを作成します。これにより、Azure DevOps 組織に対応するプロジェクトが自動的に作成されます。 

1.  ラボ コンピューターで、Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動して、このラボで使用する Azure サブスクリプションで所有者または共同作成者のロールを持つユーザー アカウントでサインインします。
1.  Azure portal で、**DevOps Starter** リソースの種類を検索して選択し、**DevOps スターター** ブレードで 「**追加**」 をクリックします。 
1. 「**Setting up DevOps starter with GitHub, change settings here**」をの「**here**」をクリックして、**Azure DevOps** を選択し、**Done** をクリックします。
1.  **DevOps Starter** ブレードの 「**新しいアプリケーションで新しく開始**」 ペインで、**NET** タイルを選択し、「**次へ**」 をクリックします。 
1.  **DevOps Starter** ブレードの 「**アプリケーション フレームワークの選択**」 ペインで、「**ASP.NET Core**」 タイルを選択し、「**データベースの追加**」 スライダーを 「**オン**」 の位置に移動して、「**次へ**」 をクリックします。 
1.  **DevOps Starter** ブレードの 「**Azure サービスを選択して アプリケーションペイン をデプロイする**」 で、「**Windows Web アプリ**」 タイルが選択されていることを確認し、「**次へ**」 をクリックします。 
1.  **DevOps Starter** ブレードで、「**Almost there!**」 ペイン上で、次の設定を指定します。

    | 設定 | 値 |
    | ------- | ----- |
    | プロジェクト名 | **Creating a Release Dashboard** |
    | Azure DevOps 組織 | このラボで使用する予定の Azure DevOps 組織の名前 |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | Web アプリの名前 | 文字、数字、ハイフンで構成され、文字または数字で開始および終了する 2 〜 60 文字のグローバルに一意の文字列  |
    | 場所 | Azure Web アプリと Azure SQL データベースをデプロイする Azure リージョンの名前 |

1.  **DevOps Starter** ブレードの 「**Location**」設定の下にある、「**Additional settings**」 をクリックします。
1.  「**追加設定**」 ペインで、次の設定を指定して 「**OK**」 をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | 新しい Azure DevOps 組織を作成する | **No** |
    | リソース グループ | **az400m10l02-rg** |
    | 価格レベル | **S1 Standard** |
    | Application Insights の場所 | Azure Web アプリの場所として選択したのと同じ Azure リージョンの名前 |
    | サーバー名 | 文字、数字、ハイフンで構成され、文字または数字で開始および終了する 3 〜 63 文字のグローバルに一意の文字列 |
    | ユーザー名を入力 | **dbadmin** |
    | 場所 | Azure Web アプリの場所として選択したのと同じ Azure リージョンの名前 |
    | データベース名 | **az400m10l02-db** |

1.  **DevOps Starter** ブレードに戻り、「**Review + Create**」 をクリックします。

    > **注**: デプロイが完了するのを待ちます。**DevOps Starter** リソースのプロビジョニングには約 2 分かかります。

1.  DevOps Starter リソースがプロビジョニングされたことの確認を受け取ったら、「**リソースに移動**」 ボタンをクリックします。これにより、ブラウザーが DevOps Starter ブレードにリダイレクトされます。
1.  DevOps Starter ブレードで、CI/CD パイプラインが正常に完了するまで進行状況を追跡します。 

    > **注**: 対応する Azure Web アプリと Azure SQL データベースの作成には約 5 分かかる場合があります。このプロセスにより、すぐにデプロイできるリポジトリとビルドおよびリリース パイプラインを含む Azure DevOps プロジェクトが自動的に作成されます。Azure リソースは、自動的にトリガーされるデプロイ パイプラインの一部として作成されます。 

#### タスク 2: Azure DevOps リリースを作成する

このタスクでは、デプロイが失敗するリリースを含む、いくつかの Azure DevOps リリースを作成します。

1.  Azure portal を表示している Web ブラウザーの DevOps スターター ページのツールバーで、「**Project homepage(プロジェクトのホームページ)**」 をクリックします。これにより、Azure Devops ポータルで**リリー スダッシュボードの作成**プロジェクトを表示する別のブラウザー タブが自動的に開きます。サインインするように求められたら、Azure DevOps 組織の資格情報で認証します。

    > **注**: 最初に、正常にデプロイされる新しいリリースを作成しておきます。

1.  Azure DevOps ポータルの左側の垂直メニューで 「**リポジトリ**」 をクリックし、リポジトリ内のフォルダーのリストで、**Applications\\aspnet-core-dotnet-core\\Pages** フォルダーに移動し、**Index.html** エントリをクリックします。 
1.  「**Index.chtml**」 ペインで、「**Edit**」 をクリックし、**20** 行目の `<div class="description line-2"> Your ASP.NET Core app is up and running on Azure</div>` を `<div class="description line-2"> Your ASP.NET Core app v1.1 is up and running on Azure</div>` に置き替え、「**Commit**」 クリックして、「**Commit**」 ペインで、もう一度 「**Commit**」 をクリックします。これにより、ビルド パイプラインが自動的にトリガーされます。 
1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインで、「**Pipelines**」 をクリックします。
1.  「**Pipelibnes**」 ペインの 「**Recent**」 タブで、一覧に表示されているパイプライン（-CI で終わる名前）をクリックし、「**Runs**」 タブで、最新の実行結果を選択します。「**Summary**」 タブで、「**Jobs**」 セクションの 「**Build**」 をクリックし、正常に完了するまでジョブを監視します。 
1.  ジョブが完了したら、Azure DevOps ポータルの左側の垂直ナビゲーションペインの 「**Pipelines**」 セクションで、「**Release**」 をクリックします。
1.  「**Release-2** 」 エントリをクリックし、「**Pipelines**」 タブで**dev**ステージをクリックし、「**View Logs**」 をクリックして、正常に完了するまで、デプロイの進行状況を監視します。 

    > **注**: ここで、デプロイが失敗する新しいリリースを作成します。失敗の原因は、新しいリリースに関連する変更が無効であると見なされる組み込みアセンブリ テストです。

1.  Azure DevOps ポータルの左側の垂直メニューで 「**Repos**」 をクリックし、リポジトリ内のフォルダーのリストで、**Applications\\aspnet-core-dotnet-core\\Pages** フォルダーに移動し、**Index.cshtml** エントリをクリックします。 
1.  「**Index.chtml**」 ペインで、「**Edit**」 をクリックし、**4** 行目の `    ViewData["Title"] = "Home Page - ASP.NET Core";` を `    ViewData["Title"] = "Home Page v1.2 - ASP.NET Core";` に置き替え、「**Commit**」 をクリックし、「**Commit**」 ペインでもう一度 「**Commit**」 をクリックします。これにより、ビルド パイプラインが自動的にトリガーされます。 
1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインで、「**パイプライン**」 をクリックします。
1.  「**Pipelines**」 ペインの 「**Recent**」 タブで、実行中の CI エントリをクリックし、「**Runs**」 タブで、最新の実行を選択します。「**Summary**」 タブで、「**Jobs**」 セクションの 「**Build**」 をクリックし、正常に完了するまでジョブをと監視します。 
1.  ジョブが完了したら、Azure DevOps ポータルの左側の垂直ナビゲーションペインの 「**Pipelines**」 セクションで、「**Release**」 をクリックします。
1.  「**Release**」 タブで 「**Release-3**」 エントリをクリックし、「**Pipelines**」 タブで**Dev**ステージをクリックし、「**View Logs**」 をクリックして、**Test Assemblies**段階で失敗するまで、デプロイの進行状況を監視します。 

#### タスク 3: Azure DevOps リリース ダッシュボードを作成する

このタスクでは、ダッシュボードを作成し、それにリリース関連のウィジェットを追加します。

1.  Azure DevOps ポータルの左側の垂直メニューで 「**Overview**」 をクリックし、 「**Dashboards**」 をクリックして、「**Add a widget**」 をクリックします。
1.  ウィジェットのリストを下にスクロールし、「**Deploy Status**」 エントリを選択して、「**Add**」 をクリックします。
1.  前の手順で説明した手順を使用して、**Release Pipeline Overview**ウィジェットを追加します。
1.  マウスを使用して**Release Pipeline Overview**を**Deploy Status** ウィジェットの右側にドラッグし、「**Done Editting**」 をクリックします。
1.  ダッシュボード ペインに戻り、**Deploy Status** ウィジェットを表す長方形をクリックします。 
1.  「**Configuration**」 ペインで、次の設定を指定し (他のすべての設定はデフォルト値のままにします)、「**Save**」 をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | ビルド パイプライン | **<固有の文字列> - CI** |
    | リンクされたリリース パイプライン | **<固有の文字列> - CD; **<固有の文字列>- CD\dev** |

1.  ダッシュボードペインに戻り、「**Release Pipeline Overview**」 ウィジェットをクリックします。  
1.  「**Configuration**」 ペインで、次の設定を指定し (他のすべての設定はデフォルト値のままにします)、「**Save**」 をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | リリース パイプライン | **az400m10l02 - CD** |

1.  ダッシュボードペインにバック、ウィジェットで表示されるコンテンツを更新するには、「**更新**」 クリックします。

    > **注**: ウィジェット上のリンクを使用すると、Azure DevOps ポータルの対応するペインに直接移動できます。

### 演習 2: REST API を介してリリース情報をクエリする

この演習では、Postman を使用して REST API を介してリリース情報をクエリします。

#### タスク 1: Azure DevOps パーソナル アクセス トークンを生成する

このタスクでは、この演習の次のタスクでインストールする Postman アプリからの認証に使用される Azure DevOps パーソナル アクセス トークンを生成します。

1.  ラボ コンピューターの Azure DevOps ポータルを表示する Web ブラウザー ウィンドウで、Azure DevOps ページの右上隅にある 「**ユーザー設定**」 アイコンをクリックし、ドロップダウン メニューで 「**Personal Access Tokens**」 をクリックします。「**New Token**」 をクリックします。
1.  「**Create a new personal access token**」 ペインで、一番下にある「**Show all scopes**」 リンクをクリックし、次の設定を指定して 「**Create**」 をクリックします (他のすべてのスコープはデフォルト値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **リリース ダッシュボード ラボの作成** |
    | 適用範囲 | **Release** |
    | アクセス許可 | **Read** |
    | 適用範囲 | **Build** |
    | アクセス許可 | **Read** |

1.  「**Success!**」 ペインで、パーソナル アクセス トークンの値をクリップボードにコピーします。

    > **注**: トークンの値を必ず記録してください。このペインを閉じると、取得できなくなります。 

1.  「**Close**」 をクリックします。

#### タスク 2: Postman を使用して REST API を介してリリース情報をクエリする

このタスクでは、Postman を使用して REST API を介してリリース情報をクエリします。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[Postman ダウンロードページ](https://www.postman.com/downloads/)に移動し、「**アプリのダウンロード**」 ボタンをクリックし、ドロップダウン メニューで、「**Windows 64 ビット**」 をクリックし、ダウンロードしたファイルをクリックして、インストールを実行します。インストールが完了すると、Postman デスクトップ アプリが自動的に起動します。 
1.  「**Postman アカウントの作成**」 ペインで、メール アドレス、ユーザー名、パスワードを入力し、「**無料アカウントの作成**」 をクリックします。 

    > **注**: アカウント プロビジョニングのプロセスを完了し、Postman アカウントをアクティブ化するために、Postman からメールが届きます。

1.  サインインしたら、Postman デスクトップ アプリ ウィンドウの左上隅にある 「**New(新規)**」 をクリックし、「**Create New**」 ペインで 「**Request**」 をクリックし、「**Request name**」 テキストボックスに「**Get-Releases**」と入力して、「**+ Create Collection**」 をクリックします。「**Name your collection**」 テキストボックスに「**Azure DevOps az400m10l02 クエリ**」と入力し、右側のチェックマークをクリックしてから、「**Save to Azure DevOps az400m10l02 クエリに保存**」 ボタンをクリックします。
1.  別の Web ブラウザー ウィンドウを開き、[**Releases - List** Microsoft Docs ページ](https://docs.microsoft.com/ja-jp/rest/api/azure/devops/release/releases/list?view=azure-devops-rest-6.0)に移動し、その内容を確認します。 
1.  Postman デスクトップ アプリに戻り、アプリ ウィンドウの「**Authorization**」 タブをクリックし、「**Type**」 ドロップダウン リストで 「**Basic Auth**」 エントリを選択し、「**Password**」 テキスト ボックスに前のタスクでコピーしたトークンの値を貼り付けます (「**Username**」 テキスト ボックスの値を設定しないでください)。
1.  アプリウィンドウに **GET** が表示されていることを確認し、すべてのリリースを一覧表示するには、「**Enter request URL**」 テキストボックスに次のように入力して 「**Send**」 をクリックします (`<organization_name>` の値を Azure DevOps 組織の名前に置き換えます)。

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/releases?api-version=6.0
    ```

1.  アプリ ウィンドウの右下のセクションにある 「**Body**」 タブにリストされている出力を確認し、このラボの前の演習で作成したリリースのリストが含まれていることを確認します。
1.  Microsoft Docs のコンテンツを表示している Web ブラウザー ウィンドウに切り替え、[**Deployments - List** Microsoft Docs ページ](https://docs.microsoft.com/ja-jp/rest/api/azure/devops/release/deployments/list?view=azure-devops-rest-6.0)に移動して、そのコンテンツを確認します。 
1.  すべてのデプロイを一覧表示するには、URL に次を入力して 「**送信**」 をクリックします (`<organization_name>` の値を Azure DevOps 組織の名前に置き換えます)。

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?api-version=6.0
    ```

1.  アプリ ウィンドウの右下のセクションにある 「**Body**」 タブにリストされている出力を確認し、このラボの前の演習で作成したデプロイのリストが含まれていることを確認します。
1.  すべての失敗したデプロイを一覧表示するには、URL を次のように入力して 「**Send**」 をクリックします (`<organization_name>` の値を Azure DevOps 組織の名前に置き換えます)。

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?DeploymentStatus=failed&api-version=6.0
    ```

1.  アプリウィンドウの右下のセクションにある 「**Body**」 タブにリストされている出力を確認し、このラボの前の演習で開始した**失敗した**展開のみが含まれていることを確認します。

### 演習 3: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングされた Azure リソースを削除して、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボでは、リリース ダッシュボードを作成および構成する方法と、REST API を使用して Azure DevOps リリースデータを取得する方法を学習しました。
