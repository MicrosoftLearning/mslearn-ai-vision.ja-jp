---
lab:
  title: 画像内のテキストの読み取り
  module: Module 11 - Reading Text in Images and Documents
---

# 画像内のテキストの読み取り

光学式文字認識 (OCR) は、画像およびドキュメント内のテキストの読み取りを処理する Computer Vision のサブセットです。 **Azure AI Vision** サービスにより、テキストを読み取るための API が提供されます。これについては、この演習で説明します。

## Computer Vision リソースをプロビジョニングする

サブスクリプションに **Computer Vision** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. **[リソースの作成]** を選択します。
1. 検索バーで *Computer Vision* を検索し、**[Computer Vision]** を選択して、次の設定でリソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **リージョン**: *米国東部、米国西部、フランス中部、韓国中部、北ヨーロッパ、東南アジア、西ヨーロッパ、東アジアから選択します\**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Free F0

    \*Azure AI Vision 4.0 の機能は、現在、これらのリージョンでのみ使用できます。

1. 必要なチェック ボックスをオンにして、リソースを作成します。
1. デプロイが完了するまで待ち、デプロイの詳細を表示します。
1. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## このコースのリポジトリを複製する

Azure portal から Cloud Shell を使用してコード開発を行います。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

> **ヒント**: 最近 **mslearn-ai-vision** リポジトリを既に複製した場合は、このタスクをスキップできます。 それ以外の場合は、次の手順に従って開発環境に複製します。

1. Azure portal で、ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、***PowerShell*** 環境を選択します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. PowerShell ペインで、次のコマンドを入力して、この演習用の GitHub リポジトリを複製します。

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. リポジトリが複製されたら、アプリケーション コード ファイルを含んだフォルダーに移動します。  

    ```
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Azure AI Vision SDK を使用するための準備をする

この演習では、Azure AI Vision SDK を使用してテキストを読み取る、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. 目的の言語のアプリケーション コード ファイルが含まれているフォルダーに移動します。  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
    ```

1. 言語の設定に応じて適切なコマンドを実行して、Azure AI Vision SDK パッケージと必要な依存関係をインストールします。

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. `ls` コマンドを使用すると、**computer-vision** フォルダーの内容を表示できます。 これには、構成設定用のファイルが含まれていることに注意してください。

    - **C#**: appsettings.json
    - **Python**: .env

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、含まれている構成値を更新して、Computer Vision リソースの**エンドポイント**と認証**キー**を反映します。
1. プレースホルダーを置き換えたら、コード エディター内で、**Ctrl + S** コマンドを使用するか、**右クリックして保存**で変更を保存してから、**Ctrl + Q** コマンドを使用するか、**右クリックして終了**で、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

## Azure AI Vision SDK を使用して画像からテキストを読み取る

**Azure AI Vision SDK** の機能の 1 つとして、画像からテキストを読み取ることができます。 この演習では、Azure AI Vision SDK を使用して画像からテキストを読み取る、部分的に実装されたクライアント アプリケーションを完成させます。

1. **read-text** フォルダーには、クライアント アプリケーションのコード ファイルが含まれています。

    - **C#** : Program.cs
    - **Python**: read-text.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision SDK を使用するために必要な名前空間をインポートします。

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

1. クライアント アプリケーションのコードファイルで **Main** 関数に、構成設定を読み込むためのコードが提供されています。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

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

1. **Main** 関数の、追加したコードの下で、コードが画像ファイルへのパスを指定し、**GetTextRead** 関数に画像パスを渡していることを確認してください。 この関数はまだ完全には実装されていません。

1. **GetTextRead** 関数の本文にいくつかコードを追加してみましょう。 「**Use Analyze image function to read text in image**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加し、`Analyze` 関数を呼び出すときにビジュアル機能が指定されていることを確認してください。

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
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
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

1. **GetTextRead** 関数に先ほど追加したコードの "**Return the text detected in the image (画像で検出されたテキストを返します)**" というコメントの下に、次のコードを追加します (このコードは、画像のテキストをコンソールに出力し、画像のテキストを強調表示する **text.jpg** 画像を生成します)。

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
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

1. C# プログラム ファイルの場合のみ、多角形を描画するために依然としてヘルパー関数が必要です。 **Helper method to draw a polygon given an array of SKPoints** というコメントの下に、次のコードを追加します。

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードは **GetTextRead** 関数を呼び出し、パスを *Lincoln.jpg* 画像ファイルに渡します。

1. 変更を保存し、コード エディターを閉じます。

1. Cloud Shell のツール バーで、**[ファイルのアップロード/ダウンロード]**、**[ダウンロード]** の順に選択します。 新しいダイアログ ボックスで、次のファイル パスを入力し、**[ダウンロード]** を選択します。

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. 画像 **Lincoln.jpg** を開いて表示します。

1. コードで処理される画像を表示したら、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. プロンプトが表示されたら、**1** を入力して、画像から抽出されたテキストである出力を確認します。

1. **read-text** フォルダーで、画像 **text.jpg** が作成されました。 ファイル パス `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` を使用してダウンロードすると、テキストの各 "行" が多角形でどのように囲まれるかがわかります。**

1. コード ファイルに戻り、**Return the position bounding box around each line** というコメントを見つけます。 このコメントの下に、次のコードを追加します。

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

1. 変更を保存し、コード エディターを閉じてから、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. メッセージが表示されたら、「**1**」と入力し、画像内の各行のテキストが画像内のそれぞれの位置に出力されるのを確認します。

1. コード ファイルをもう一度開き、**Return each word detected in the image and the position bounding box around each word with the confidence level of each word** というコメントを見つけます。 このコメントの下に、次のコードを追加します。

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
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

1. 変更を保存し、コード エディターを閉じてから、次のコマンドを入力してプログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. メッセージが表示されたら、「**1**」と入力し、画像内のテキストの各単語が画像内のそれぞれの位置に出力されるのを確認します。 各単語の信頼レベルがどのように返されるかという点にも注目してください。

1. 画像 **text.jpg** をダウンロードすると、各 "単語" が多角形でどのように囲まれるかがわかります。**

## Azure AI Vision SDK を使用して画像から手書きのテキストを読み取る

前の演習では、画像から明確に定義されたテキストを読み取りましたが、手書きのノートや論文からテキストを読み取る必要が生じる場合もあります。 幸いなことに、**Azure AI Vision SDK** では、明確に定義されたテキストを読み取るために使用したのと同じコードを使用して、手書きのテキストを読み取ることもできます。 前の演習と同じコードを使用しますが、今回は別の画像を使用します。

1. ファイル パス `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` を使用して **Note.jpg** をダウンロードし、コードで次に処理する画像を表示します。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **2** を選択した場合に実行されるコードを調べます。 このコードでは、**GetTextRead** 関数を呼び出し、*Note.jpg* 画像ファイルへのパスを渡します。

1. ターミナルから次のコマンドを入力して、プログラムを実行します。

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. プロンプトが表示されたら、「**2**」を入力して、画像から抽出されたテキストである出力を確認します。

1. 画像 **text.jpg** をもう一度ダウンロードすると、メモの各 "単語" が多角形でどのように囲まれるかがわかります。**

## リソースをクリーンアップする

他のトレーニング モジュールに対してこのラボで作成された Azure リソースを使用していない場合は、それらを削除して、追加の料金が発生しないようにすることができます。 方法は以下のとおりです。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。

1. 検索バーで *Computer Vision* を検索し、このラボで作成した Computer Vision リソースを選択します。

1. [リソース] ページで **[削除]** を選択し、指示に従ってリソースを削除します。

## 詳細情報

**Azure AI Vision** サービスを使用してテキストを読み取る方法の詳細については、「[Azure AI Vision のドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)」を参照してください。
