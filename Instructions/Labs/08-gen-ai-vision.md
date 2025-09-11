---
lab:
  title: ビジョン対応チャット アプリを開発する
  description: Azure AI Foundry を使用して、画像入力をサポートする生成 AI アプリをビルドする。
---

# ビジョン対応チャット アプリを開発する

この演習では、*Phi-4-multimodal-instruct* 生成 AI モデルを使用して、画像を含むプロンプトに対する応答を生成します。 Azure AI Foundry と Azure AI モデル推論サービスを使用して、食料品店の新鮮な食材に AI 支援を提供するアプリを開発します。

> **注**:この演習は、変更される可能性があるプレリリース SDK ソフトウェアに基づいています。 必要に応じて、特定のバージョンのパッケージを使用しました。利用可能な最新バージョンが反映されていない可能性があります。 予期しない動作、警告、またはエラーが発生する場合があります。

この演習は、Azure AI Foundry Python SDK に基づいていますが、次のような複数の言語固有の SDK を使用して AI チャット アプリケーションを開発することができます。

- [Python 向け Azure AI プロジェクト](https://pypi.org/project/azure-ai-projects)
- [Microsoft .NET 向け Azure AI プロジェクト](https://www.nuget.org/packages/Azure.AI.Projects)
- [JavaScript 向け Azure AI プロジェクト](https://www.npmjs.com/package/@azure/ai-projects)

この演習は約 **30** 分かかります。

## Azure AI Foundry ポータルを開く

まず、Azure AI Foundry ポータルにサインインしましょう。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ホーム ページの情報を確認します。

## プロジェクトを開始するモデルを選択する

Azure AI *プロジェクト*には、AI 開発のための共同ワークスペースが用意されています。 まず、使用するモデルを選択し、それを使用するプロジェクトを作成しましょう。

> **注**: AI Foundry プロジェクトは *Azure AI Foundry* リソースをベースにすることができます。このリソースが、AI モデル (Azure OpenAI を含む)、Azure AI サービス、AI エージェントやチャット ソリューションを開発するためのその他のリソースへのアクセスを提供します。 または、*AI ハブ* リソースに基づいてプロジェクトを作成することもできます。このリソースには、セキュリティで保護されたストレージ、コンピューティング、専用のツールに使用する Azure リソースへの接続が含まれます。 Azure AI Foundry ベースのプロジェクトは、AI エージェントまたはチャット アプリ開発のリソースの管理を希望する開発者に最適です。 複雑な AI ソリューションに取り組むエンタープライズ開発チームには、AI ハブ ベースのプロジェクトの方が適しています。

1. ホーム ページの **[モデルと機能を調査する]** セクションで、プロジェクトで使用する `Phi-4-multimodal-instruct` モデルを検索します。

1. 検索結果で **Phi-4-multimodal-instruct** モデルを選んで詳細を確認してから、モデルのページの上部にある **[このモデルを使用する]** を選択します。

1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。

1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **Azure AI Foundry リソース**: *Azure AI Foundry リソースの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: **AI Foundry が推奨する**もの*の中から選択します\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択し、選んだ Phi-4-multimodal-instruct モデル デプロイを含むプロジェクトが作成されるまで待ちます。

    > 注:モデルの選択によっては、プロジェクトの作成プロセス中に追加のプロンプトが表示される場合があります。 使用条件に同意し、デプロイを完了します。

1. プロジェクトが作成されると、モデルが **[モデル + エンドポイント]** ページに表示されます。

    ![[モデル デプロイ] ページのスクリーンショット。](./media/ai-foundry-model-deployment.png)

## プレイグラウンドでモデルをテストする

これで、チャット プレイグラウンドでイメージベースのプロンプトを使用して、マルチモーダル モデル デプロイをテストできます。

1. [モデル デプロイ] ページで **[プレイグラウンドで開く]** を選択します。

1. 新しいブラウザー タブで、`https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg`から [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg) をダウンロードし、それをローカル ファイル システム上のフォルダーに保存します。

1. チャット プレイグラウンド ページの **[セットアップ]** ウィンドウで、**Phi-4-multimodal-instruct** モデル デプロイが選択されていることを確認します。

1. チャット セッションのメイン パネルで、チャット入力ボックスの下にある添付ボタン (**&128206;**) を使用して *mango.jpeg* 画像ファイルをアップロードしてから、テキスト `What desserts could I make with this fruit?` を追加してプロンプトを送信します。

    ![[チャット プレイグラウンド] ページのスクリーンショット。](../media/chat-playground-image.png)

1. 応答を確認します。これは、マンゴーを使用して作ることができるデザートに関するガイダンスを提供しているはずです。

## クライアント アプリケーションを作成する

モデルをデプロイしたので、クライアント アプリケーションでデプロイを使用できます。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。

1. **[エンドポイントとキー]** の領域で、**[Azure AI Foundry]** ライブラリが選択されていることを確認し、**[Azure AI Foundry プロジェクト エンドポイント]** に注目します。 この接続文字列を使用して、クライアント アプリケーションでプロジェクトに接続します。

1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

    ウェルカム通知を閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

    > **注**:ファイルを保持するストレージを選択するようにポータルから求められた場合は、**[ストレージ アカウントは必要ありません]** を選択し、お使いのサブスクリプションを選択して、**[適用]** キーを選択します。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリをクローンします (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、アプリケーション コード ファイルを含むフォルダーに移動します。  

    ```
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーを Foundry プロジェクト エンドポイント (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを Phi-4-multimodal-instruct モデル デプロイに割り当てた名前に置き換えます。

1. プレースホルダーを置き換えたら、コード エディター内で、**Ctrl + S** コマンドまたは**右クリック メニューの [保存]** を使用して変更を保存し、**Ctrl + Q** コマンドまたは**右クリック メニューの [終了]** を使用して Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### プロジェクトに接続してモデルのためにチャット クライアントを取得するためのコードを記述する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    ```
   code chat-app.py
    ```

1. コード ファイルで、ファイルの先頭に追加された既存のステートメントを書き留めて、必要な SDK 名前空間をインポートします。 次に、コメント**参照の追加**を探して、次のコードを追加し、前にインストールしたライブラリの名前空間を参照します。

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. **main** 関数のコメント**構成設定の取得**で、構成ファイルで定義したプロジェクト接続文字列とモデル デプロイ名の値がコードで読み込まれることに注意してください。
1. **main** 関数のコメント**構成設定の取得**で、構成ファイルで定義したプロジェクト接続文字列とモデル デプロイ名の値がコードで読み込まれることに注意してください。
1. コメント **Initialize the project client** を見つけて、次のコードを追加し、Azure AI Foundry プロジェクトに接続します。

    > **ヒント**: コードのインデント レベルを正しく維持するように注意してください。

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. コメント**チャット クライアントの取得**を探して、次のコードを追加し、モデルとチャットするためのクライアント オブジェクトを作成します。

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### URL ベースの画像プロンプトを送信するためのコードを記述する

1. コードには、ユーザーが「quit」と入力するまでプロンプトを入力できるようにするループが含まれていることに注意してください。 ループ セクションで、コメント **"Get a response to image input"** を見つけ、次の画像を含むプロンプトを送信する次のコードを追加します。

    ![マンゴーの写真。](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。ただし、まだ閉じないでください。

## Azure にサインインしてアプリを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
   az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Azure AI Foundry ハブを含むサブスクリプションを選択します。

1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python chat-app.py
    ```

1. メッセージが表示されたら、次のプロンプトを入力します。

    ```
   Suggest some recipes that include this fruit
    ```

1. 応答を確認します。 次に、「`quit`」と入力してプログラムを終了します。

### ローカル イメージ ファイルをアップロードするようにコードを変更する

1. アプリ コードのコード エディターのループ セクションで、前にコメント **"Get a response to image input"** の下に追加したコードを見つけます。 次に、次のようにコードを変更して、このローカル イメージ ファイルをアップロードします。

    ![ドラゴン フルーツの写真。](../media/mystery-fruit.jpeg)

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。 必要に応じて、コード エディターを閉じる (**CTRL + Q**) こともできます。

1. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    ```
   python chat-app.py
    ```

1. メッセージが表示されたら、次のプロンプトを入力します。

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. 応答を確認します。 次に、「`quit`」と入力してプログラムを終了します。

    > **注**: このシンプルなアプリには、会話履歴を保持するためのロジックが含まれていないので、モデルは、各プロンプトを前のプロンプトのコンテキストを持たない新しいリクエストとして処理します。

## クリーンアップ

Azure AI Foundry ポータルを確認し終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. [Azure ポータル](https://portal.azure.com)を開き、この演習で使用したリソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
