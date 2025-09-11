---
lab:
  title: 画像を分類する
  description: Azure AI Custom Vision サービスを使用して、分類モデルをトレーニングします。
---

# 画像を分類する

**Azure AI Custom Vision** サービスを使用すると、独自の画像でトレーニングされた Computer Vision モデルを作成できます。 これを使用して、*画像分類* および *物体検出* モデルをトレーニングできます。その後、公開してアプリケーションから利用できます。

この演習では、Computer Vision サービスを使用して、果物の 3 つのクラス (リンゴ、バナナ、オレンジ) を識別できる画像分類モデルをトレーニングします。

この演習は、Azure Custom Vision Python SDK に基づいていますが、次のような複数の言語固有の SDK を使用して Vision アプリケーションを開発することができます。

* [Azure Custom Vision for JavaScript (トレーニング)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training)
* [Azure Custom Vision for JavaScript (予測)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction)
* [Azure Custom Vision for Microsoft .NET (トレーニング)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
* [Azure Custom Vision for Microsoft .NET (予測)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)
* [Azure Custom Vision for Java (トレーニング)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-training/1.1.0-preview.2/jar)
* [Azure Custom Vision for Java (予測)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-prediction/1.1.0-preview.2/jar)

この演習は約 **45** 分かかります。

## Custom Vision リソースを作成する

モデルをトレーニングする前に、*トレーニング* と *予測* のために Azure リソースが必要になります。 これらのタスクごとに **Custom Vision** リソースを作成することも、単一のリソースを作成して両方のタスクに使用することもできます。 この演習では、トレーニング用と予測用の **Custom Vision** リソースを作成します。

1. [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、Azure 資格情報を使用してサインインします。 表示されているすべてのウェルカム メッセージまたはヒントを閉じます。
1. **[リソースの作成]** を選択します。
1. 検索バーで `Custom Vision` を検索し、**[Custom Vision]** を選択して、次の設定でリソースを作成します。
    - **作成オプション**: 両方
    - **[サブスクリプション]**:*ご自身の Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **[リージョン]**: *使用できるリージョンを選択します*
    - **[名前]**: "Custom Vision リソースの有効な名前"**
    - **トレーニング価格レベル**: F0
    - **[予測価格レベル]**: F0

1. リソースを作成し、デプロイが完了するまで待ってから、デプロイの詳細を表示します。 2 つの Custom Vision リソースがプロビジョニングされていることに注意してください。1 つはトレーニング用で、もう 1 つは予測用です。

    > **注**:各リソースには独自の*エンドポイント*と*キー*があり、コードからのアクセスを管理するために使用されます。 画像分類モデルをトレーニングするには、コードで *トレーニング* リソース (エンドポイントとキーを含む) を使用する必要があります。トレーニング済みモデルを使用して画像クラスを予測するには、コードで *予測* リソース (エンドポイントとキーを含む) を使用する必要があります。

1. リソースがデプロイされたら、リソース グループに移動して表示します。 2 つのカスタム ビジョン リソースが表示されます。1 つはサフィックスが ***-Prediction*** です。

## Custom Vision ポータルで Custom Vision プロジェクトを作成する

画像分類モデルをトレーニングするには、トレーニング リソースに基づいて Custom Vision プロジェクトを作成する必要があります。 これを行うには、Custom Vision ポータルを使用します。

1. 新しいブラウザー タブを開きます (後で戻るため、Azure portal のタブは開いたままにしておきます)。
1. 新しいブラウザー タブで、[Custom Vision ポータル](https://customvision.ai) (`https://customvision.ai`) を開きます。 メッセージが表示されたら、Azure 資格情報を使用してサインインし、サービス使用条件に同意します。
1. Custom Vision ポータルで、次の設定を使って新しいプロジェクトを作成します。
    - **名前**: `Classify Fruit`
    - **説明**: `Image classification for fruit`
    - **Resource**:"Custom Vision リソース"**
    - **プロジェクトの種類**: Classification
    - **分類の種類**: Multiclass (Single tag per image)
    - **ドメイン**: 食品

### 画像をアップロードし、タグ付けする

1. ブラウザーの新しいタブで、`https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/image-classification/training-images.zip` から [トレーニング画像](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/image-classification/training-images.zip)をダウンロードし、zip フォルダーを展開してその内容を表示します。 このフォルダーには、リンゴ、バナナ、オレンジの画像のサブフォルダーが含まれています。
1. Custom Vision ポータルの画像分類プロジェクトで、**[画像の追加]** をクリックし、先ほどダウンロードして展開した **training-images/apple** フォルダー内のすべてのファイルを選択します。 次に、次のようにタグ `apple` を指定して、画像ファイルをアップロードします。

    ![タグ apple を指定して apple をアップロードする場合を示すスクリーンショット。](../media/upload_apples.jpg)

1. **[画像の追加]** (**[+]**) ツール バー アイコンを使用して前の手順を繰り返します。タグ `banana` を指定して **banana** フォルダー内の画像をアップロードし、タグ `orange` を指定して **orange** フォルダー内の画像をアップロードします。
1. Custom Vision プロジェクトでアップロードした画像を探します。次のように各クラスの画像が 15 個あるはずです。

    ![タグ付けされた果物の画像 - りんご 15 個、バナナ 15 個、みかん 15 個](../media/fruit.jpg)

### モデルをトレーニングする

1. Custom Vision プロジェクトで、画像の上にある **[トレーニング]** (&#9881;<sub>&#9881;</sub>) をクリックし、タグ付けされた画像を使用して分類モデルをトレーニングします。 **クイック トレーニング** オプションを選び、トレーニングの反復が完了するまで待ちます (1 分ほどかかる場合があります)。
1. モデルの反復をトレーニングしたら、*Precision*、*Recall*、*AP* のパフォーマンス メトリックを確認します。これらは分類モデルの予測精度を測るものであり、すべて高くするようにします。

    ![モデルのメトリックスのスクリーンショット。](../media/custom-vision-metrics.png)

> **注**: パフォーマンス メトリックは、各予測の 50% の確率しきい値に基づいています (つまり、モデルが特定のクラスの画像である確率が 50% 以上であると計算した場合、そのクラスが予測されます)。 これはページの左上で調整できます。

### モデルのテスト

1. パフォーマンス メトリックの上にある **[クイック テスト]** をクリックします。
1. **[画像の URL]** ボックスに「`https://aka.ms/test-apple`」と入力し、 *[画像のクイック テスト] (&#10132;)* ボタンをクリックします。
1. モデルによって返される予測を確認します。次のように *apple* の確率スコアが最も高くなるはずです。

    ![apple のクラス予測を含む画像のスクリーンショット。](../media/test-apple.jpg)

1. 次の画像をテストしてみてください。
    - `https://aka.ms/test-banana`
    - `https://aka.ms/test-orange`

1. その後、**Quick Test** ウィンドウを閉じます。

### プロジェクト設定を表示する

作成したプロジェクトには一意の識別子が割り当てられており、それを操作するコードで指定する必要があります。

1. **Performance** ページの右上にある *設定* (&#9881;) アイコンをクリックして、プロジェクトの設定を表示します。
1. **General** (左側) の下で、このプロジェクトを一意に識別する **[プロジェクト ID]** に注意してください。
1. 右側の **[リソース]** の下に、キーとエンドポイントが表示されていることに注意してください。 これらは、*トレーニング* リソースの詳細です (この情報は、Azure portal でリソースを表示することでも取得できます)。

## *トレーニング* API を使用する

Custom Vision ポータルは、画像のアップロードとタグ付け、およびモデルのトレーニングに使用できる便利なユーザー インターフェイスを提供します。 ただし、シナリオによっては、Custom Vision トレーニング API を使用してモデル トレーニングを自動化することができます。

### アプリケーション構成を準備する

1. Azure portal が表示されているブラウザー タブに戻ります (後で戻るため、Custom Vision ポータルのタブは開いたままにしておきます)。
1. Azure portal で、ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

    > **注**:ファイルを保持するストレージを選択するようにポータルから求められた場合は、**[ストレージ アカウントは不要です]** を選択し、お使いのサブスクリプションを選択して、**[適用]** を選択します。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell ペインをサイズ変更して、さらに多くの内容を表示できるようにします。

    > **ヒント**" 上部の境界線をドラッグすると、ペインのサイズを変更できます。 最小化ボタンと最大化ボタンを使用して、Cloud Shell とメイン ポータル インターフェイスを切り替えることもできます。

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリをクローンします (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、次のコマンドを使用してアプリケーション コード ファイルに移動します。

    ```
   cd mslearn-ai-vision/Labfiles/image-classification/python/train-classifier
   ls -a -l
    ```

    このフォルダーには、アプリのアプリケーション構成ファイルとコード ファイルが含まれています。 また、**/more-training-images** サブフォルダーも含まれており、このサブフォルダーには、モデルの追加トレーニングを実行するために使用するいくつかの画像ファイルが含まれています。

1. 次のコマンドを実行して、トレーニング用の Azure AI Custom Vision SDK パッケージとその他の必要なパッケージをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. 次のコマンドを入力して、アプリの構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイル内で、含まれている構成値を更新して、Custom Vision "トレーニング" リソースの**エンドポイント**と認証**キー**、および以前に作成した Custom Vision プロジェクトの**プロジェクト ID** を反映します。**
1. プレースホルダーを置き換えたら、コード エディター内で **Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### モデルのトレーニングを実行するコードを記述する

1. Cloud Shell コマンド ラインで、次のコマンドを入力して、クライアント アプリケーションのコード ファイルを開きます。

    ```
   code train-classifier.py
    ```

1. コード ファイル内の次の詳細に注意してください。
    - Azure AI Custom Vision SDK の名前空間がインポートされます。
    - **Main** 関数は構成設定を取得し、キーとエンドポイントを使用して認証済みの 
    - **CustomVisionTrainingClient** を作成します。これは、プロジェクトへの **Project** 参照を作成するためにプロジェクト ID と共に使用されます。
    - **Upload_Images** 関数は、Custom Vision プロジェクトで定義されているタグを取得し、対応する名前のフォルダーからプロジェクトに画像ファイルをアップロードして、適切なタグ ID を割り当てます。
    - **Train_Model** 関数は、プロジェクトの新しいトレーニング反復を作成し、トレーニングが完了するのを待ちます。

1. コード エディターを閉じ (*Ctrl + Q* キー)、次のコマンドを入力してプログラムを実行します。

    ```
   python train-classifier.py
    ```

1. プログラムが終了するのを待ちます。 次に、Custom Vision ポータルが表示されているブラウザー タブに戻り、プロジェクトの **[トレーニング画像]** ページを表示します (必要に応じてブラウザーを更新します)。
1. いくつかの新しいタグ付き画像がプロジェクトに追加されていることを確認します。 次に、 **Performance** ページを表示し、新しいイテレーションが作成されたことを確認します。

## クライアント アプリケーションで画像分類子を使用する

以上で、トレーニング済みモデルを公開してクライアント アプリケーションで使用する準備が整いました。

### 画像分類モデルを発行する

1. Custom Vision ポータルの **Performance** ページで、 **[&#128504; Publish]** をクリックして、トレーニング済みモデルを次の設定で公開します。
    - **モデル名**: `fruit-classifier`
    - **Resources**: *前に作成した "-Prediction" で終わる**予測**リソース (トレーニング リソースでは<u>ありません</u>)。*
1. **[プロジェクト設定]** ページの左上にある *[プロジェクトギャラリー]* (👁) アイコンをクリックして、プロジェクトが一覧表示されている Custom Vision ポータルの [ホーム] ページに戻ります。
1. Custom Vision ポータルの [ホーム] ページの右上にある *設定* (&#9881;) アイコンをクリックして、Custom Vision サービスの設定を表示します。 次に、 **[リソース]** の下で、"-Prediction" で終わる *予測* リソース (トレーニング リソースでは<u>ありません</u>) を見つけて、その**キー**と**エンドポイント**の値を確認します (この情報は、Azure portal のリソースで表示して取得することもできます)。

### クライアント アプリケーションからの画像分類子を使用する

1. Azure portal と Cloud Shell ペインが表示されているブラウザー タブに戻ります。
1. Cloud Shell で次のコマンドを実行して、クライアント アプリケーションのフォルダーに切り替え、含まれているファイルを表示します。

    ```
   cd ../test-classifier
   ls -a -l
    ```

    このフォルダーには、アプリのアプリケーション構成ファイルとコード ファイルが含まれています。 また、**/test-images** サブフォルダーも含まれており、このサブフォルダーには、モデルのテストに使用するいくつかの画像ファイルが含まれています。

1. 次のコマンドを実行して、予測用の Azure AI Custom Vision SDK パッケージとその他の必要なパッケージをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. 次のコマンドを入力して、アプリの構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. 構成値を更新して、Custom Vision "<u>予測</u>" リソースの**エンドポイント**と認証**キー**、分類プロジェクトの**プロジェクト ID**、公開済みのモデルの名前 (*fruit-classifier*) を反映します。** 変更を保存し (*Ctrl + S* キー)、コード エディターを閉じます (*Ctrl + Q* キー)。
1. Cloud Shell コマンド ラインで、次のコマンドを入力して、クライアント アプリケーションのコード ファイルを開きます。

    ```
   code test-classifier.py
    ```

1. コードを確認し、次の詳細に注意します。
    - Azure AI Custom Vision SDK の名前空間がインポートされます。
    - **Main** 関数は構成設定を取得し、キーとエンドポイントを使用して認証済みの **CustomVisionPredictionClient** を作成します。
    - 予測クライアント オブジェクトは、**test-images** フォルダー内の各画像のクラスを予測するために使用され、各リクエストのプロジェクト ID とモデル名を指定します。 各予測には、可能なクラスごとの確率が含まれ、確率が 50% を超える予測タグのみが表示されます。

1. コード エディターを閉じ、次のコマンドを入力してプログラムを実行します。

    ```
   python test-classifier.py
    ```

    プログラムは、分類のために次の各画像をモデルに送信します。

    ![りんごの画像](../media/IMG_TEST_1.jpg)

    **IMG_TEST_1.jpg**

    <br/><br/>

    ![バナナの画像](../media/IMG_TEST_2.jpg)

    **IMG_TEST_2.jpg**

    <br/><br/>

    ![オレンジの画像](../media/IMG_TEST_3.jpg)

    **IMG_TEST_3.jpg**

1. 各予測のラベル (タグ) ）と確率スコアを表示します。

## リソースをクリーンアップする

Azure AI Custom Vision を調べ終えたら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. `https://portal.azure.com` で Azure portal を開き、上部の検索バーで、このラボで作成したリソースを検索します。

1. [リソース] ページで **[削除]** を選択し、指示に従ってリソースを削除します。 または、リソース グループ全体を削除して、すべてのリソースを同時にクリーンアップすることもできます。
