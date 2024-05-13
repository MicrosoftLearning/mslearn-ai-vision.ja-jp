---
lab:
  title: 画像内のテキストの読み取り
  module: Module 11 - Reading Text in Images and Documents
---

# 画像内のテキストの読み取り

光学式文字認識 (OCR) は、画像およびドキュメント内のテキストの読み取りを処理する Computer Vision のサブセットです。 **Azure AI Vision** サービスにより、テキストを読み取るための API が提供されます。これについては、この演習で説明します。

## このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリをクローンする必要があります。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-ai-vision` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。 「*フォルダー内の Azure 関数プロジェクトが検出されました*」というメッセージが表示されたら、そのメッセージを閉じてかまいません。

## Azure AI サービス リソースをプロビジョニングする

サブスクリプションに **Azure AI サービス** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで 「*Azure AI サービス*」を検索し、**[Azure AI サービス]** を選択し、次の設定で Azure AI サービス マルチサービス アカウント リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **リージョン**: *米国東部、フランス中部、韓国中部、北ヨーロッパ、東南アジア、西ヨーロッパ、米国西部、または東アジアから選択します\**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0

    \*Azure AI Vision 4.0 の機能は、現在、これらのリージョンでのみ使用できます。

3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その**キーとエンドポイント** ページを表示します。 次の手順で、このページのエンドポイントとキーの 1 つが必要になります。

## Azure AI Vision SDK を使用するための準備をする

この演習では、Azure AI Vision SDK を使用してテキストを読み取る、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の**エクスプローラー** ウィンドウで、**Labfiles\05-ocr** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **read-text** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure AI Vision SDK パッケージをインストールします。

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **注**: 開発キット拡張機能のインストールを求められた場合は、そのメッセージを閉じてかまいません。

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. **read-text** フォルダーの内容を表示し、構成設定用のファイルが含まれていることにご注意ください。

    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。


## Azure AI Vision SDK を使用して画像からテキストを読み取る

**Azure AI Vision SDK** の機能の 1 つは、画像からテキストを読み取る機能です。 この演習では、Azure AI Vision SDK を使用して画像からテキストを読み取る、部分的に実装されたクライアント アプリケーションを完成させます。

1. **read-text** フォルダーには、クライアント アプリケーションのコード ファイルが含まれています。

    - **C#** : Program.cs
    - **Python**: read-text.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision SDK を使用するために必要な名前空間をインポートします。

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. クライアント アプリケーションのコードファイルで **Main** 関数に、構成設定を読み込むためのコードが提供されています。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```
    
    **Python**
    
    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

3. **Main** 関数の、追加したコードの下で、コードが画像ファイルへのパスを指定し、**GetTextRead** 関数に画像パスを渡していることを確認してください。 この関数はまだ完全には実装されていません。

4. **GetTextRead** 関数の本文にいくつかコードを追加してみましょう。 「**Use Analyze image function to read text in image**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加し、`Analyze` 関数を呼び出すときにビジュアル機能が指定されていることを確認してください。

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

5. **GetTextRead** 関数に先ほど追加したコードの "**Return the text detected in the image (画像で検出されたテキストを返します)**" というコメントの下に、次のコードを追加します (このコードは、画像のテキストをコンソールに出力し、画像のテキストを強調表示する **text.jpg** 画像を生成します)。

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

6. **read-text/images** フォルダーで、**Lincoln.jpg** を選択し、コードによって処理されるファイルを表示します。

7. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを *Lincoln.jpg* 画像ファイルに渡します。

8. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. プロンプトが表示されたら、**1** を入力して、画像から抽出されたテキストである出力を確認します。

10. **read-text** フォルダーで **text.jpg** 画像を選択し、テキストの各*行*の周囲に多角形があることを確認します。

11. Visual Studio Code のコード ファイルに戻り、"**Return the position bounding box around each line (各行を囲む位置境界ボックスを返します)**" というコメントを見つけます。 このコメントの下に、次のコードを追加します。

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");  
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

12. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. メッセージが表示されたら、「**1**」と入力し、画像内の各行のテキストが画像内のそれぞれの位置に出力されるのを確認します。


14. Visual Studio Code のコード ファイルに戻り、「**Return each word detected in the image and the position bounding box around each word with the confidence level of each word**」というコメントを見つけます。 このコメントの下に、次のコードを追加します。

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

15. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. メッセージが表示されたら、「**1**」と入力し、画像内のテキストの各単語が画像内のそれぞれの位置に出力されるのを確認します。 各単語の信頼レベルがどのように返されるかという点にも注目してください。

17. **read-text** フォルダーで **text.jpg** 画像を選択し、各*単語*の周囲に多角形があることを確認します。

## Azure AI Vision SDK を使用して画像から手書きのテキストを読み取る

前の演習では、画像から明確に定義されたテキストを読み取りましたが、手書きのノートや論文からテキストを読み取る必要が生じる場合もあります。 幸いなことに、**Azure AI Vision SDK** では、明確に定義されたテキストを読み取るために使用したのと同じコードを使用して、手書きのテキストを読み取ることもできます。 前の演習と同じコードを使用しますが、今回は別の画像を使用します。

1. **read-text/images** フォルダーで、**Note.jpg** を開いて、コードが処理するファイルを表示します。

2. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **2** を選択した場合に実行されるコードを調べます。 このコードでは、**GetTextRead** 関数を呼び出し、*Note.jpg* 画像ファイルへのパスを渡します。

3. **read-text** フォルダーの統合ターミナルで、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. プロンプトが表示されたら、「**2**」を入力して、画像から抽出されたテキストである出力を確認します。

5. **read-text** フォルダーで **text.jpg** 画像を選択し、各*単語*の周囲に多角形があることを確認します。

## リソースをクリーンアップする

他のトレーニング モジュールに対してこのラボで作成された Azure リソースを使用していない場合は、それらを削除して、追加の料金が発生しないようにすることができます。 方法は以下のとおりです。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。

2. 上部の検索バーで、*Azure AI サービス マルチサービス アカウント*を検索し、このラボで作成した Azure AI サービス マルチサービス アカウント リソースを選択します

3. [リソース] ページで **[削除]** を選択し、指示に従ってリソースを削除します。

## 詳細情報

**Azure AI Vision** サービスを使用してテキストを読み取る方法の詳細については、「[Azure AI Vision のドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)」を参照してください。
