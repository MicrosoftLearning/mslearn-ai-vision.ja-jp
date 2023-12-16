---
lab:
  title: Azure AI Vision を使用して画像を分析する
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Azure AI Vision を使用して画像を分析する

Azure AI Vision は、ソフトウェア システムが画像を分析して視覚入力を解釈できるようにする人工知能機能です。 Microsoft Azure の **Vision** Azure AI サービスは、キャプションとタグを提案する画像の分析、一般的なオブジェクトや人物の検出など、一般的な Computer Vision タスク用の構築済みモデルを提供します。 Azure AI Vision サービスを使用して背景を削除したり、画像の前景のマット処理を作成したりすることもできます。

## このコースのリポジトリを複製する

このラボで作業している環境に **Azure AI Vision** コード リポジトリをまだクローンしていない場合は、次の手順に従ってクローンします。 それ以外の場合は、複製されたフォルダーを Visual Studio Code で開きます。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-ai-vision` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるダイアログが表示された場合は、 **[今はしない]** を選択します。 「*フォルダー内の Azure 関数プロジェクトが検出されました*」というメッセージが表示されたら、そのメッセージを閉じてかまいません。

## Azure AI サービス リソースをプロビジョニングする

サブスクリプションに **Azure AI サービス** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで、*Azure AI サービス*を検索し、**Azure AI サービス**を選択して、次の設定で Azure AI サービスのマルチサービス アカウント リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **リージョン**: *米国東部、フランス中部、韓国中部、北ヨーロッパ、東南アジア、西ヨーロッパ、米国西部、または東アジアから選択します\**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0

    \*Azure AI Vision 4.0 の機能は、現在、これらのリージョンでのみ使用できます。

3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## Azure AI Vision SDK を使用するための準備をする

この演習では、Azure AI Vision SDK を使用して画像を分析する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ウィンドウで、**Labfiles/01-analyze-images** フォルダーを参照し、言語の設定に応じて、**C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **image-analysis** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure AI Vision SDK パッケージをインストールします。

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis --prerelease
    ```

    > **注**: 開発キット拡張機能のインストールを求められた場合は、そのメッセージを閉じてかまいません。

    **Python**
    
    ```
    pip install azure-ai-vision
    ```
    
3. **image-analysis** フォルダーの内容を表示し、構成設定用のファイルが含まれていることに注意してください。
    - **C#** : appsettings.json
    - **Python**: .env

    構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。
4. **image-analysis** フォルダーには、クライアント アプリケーションのコード ファイルが含まれていることに注意してください。

    - **C#** : Program.cs
    - **Python**: image-analysis.py

    コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision SDK を使用するために必要な名前空間をインポートします。

**C#**

```C#
// Import namespaces
using Azure.AI.Vision.Common;
using Azure.AI.Vision.ImageAnalysis;
```

**Python**

```Python
# import namespaces
import azure.ai.vision as sdk
```
    
## ビューの画像は、あなたが分析します

この演習では、Azure AI Vision サービスを使用して複数の画像を分析します。

1. Visual Studio Code で、**image-analysis** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. 各画像ファイルを順番に選択して表示し、Visual Studio Code で表示します。

## 画像を分析してキャプションを提案する

これで、SDK を使用して Vision サービスを呼び出し、画像を分析する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **image-analysis.py**) の **Main** 関数で、構成設定をロードするためのコードが提供されていることに注意してください。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

**C#**

```C#
// Authenticate Azure AI Vision client
var cvClient = new VisionServiceOptions(
    aiSvcEndpoint,
    new AzureKeyCredential(aiSvcKey));
```

**Python**

```Python
# Authenticate Azure AI Vision client
cv_client = sdk.VisionServiceOptions(ai_endpoint, ai_key)
```

2. **Main** 関数で、先ほど追加したコードの下で、コードが画像ファイルへのパスを指定し、その画像パスを他の 2 つの関数 (**AnalyzeImage** と **BackgroundForeground**) に渡すことに注意してください。 これらの関数はまだ完全には実装されていません。

3. **AnalyzeImage** 関数のコメント "**Specify features to be retrieved**" の下に、次のコードを追加します。

**C#**

```C#
// Specify features to be retrieved
Features =
    ImageAnalysisFeature.Caption
    | ImageAnalysisFeature.DenseCaptions
    | ImageAnalysisFeature.Objects
    | ImageAnalysisFeature.People
    | ImageAnalysisFeature.Text
    | ImageAnalysisFeature.Tags
```

**Python**

```Python
# Specify features to be retrieved
analysis_options = sdk.ImageAnalysisOptions()

features = analysis_options.features = (
    sdk.ImageAnalysisFeature.CAPTION |
    sdk.ImageAnalysisFeature.DENSE_CAPTIONS |
    sdk.ImageAnalysisFeature.TAGS |
    sdk.ImageAnalysisFeature.OBJECTS |
    sdk.ImageAnalysisFeature.PEOPLE
)
```
    
4. **AnalyzeImage** 関数のコメント "**Get image analysis**" の下に、次のコードを追加します (後でコードを追加する場所を示すコメントを含む)。

**C#**

```C#
// Get image analysis
using var imageSource = VisionSource.FromFile(imageFile);

using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

var result = analyzer.Analyze();

if (result.Reason == ImageAnalysisResultReason.Analyzed)
{
    // get image captions
    if (result.Caption != null)
    {
        Console.WriteLine(" Caption:");
        Console.WriteLine($"   \"{result.Caption.Content}\", Confidence {result.Caption.Confidence:0.0000}");
    }

    //get image dense captions
    if (result.DenseCaptions != null)
    {
        Console.WriteLine(" Dense Captions:");
        foreach (var caption in result.DenseCaptions)
        {
            Console.WriteLine($"   \"{caption.Content}\", Confidence {caption.Confidence:0.0000}");
        }
        Console.WriteLine($"\n");
    }

    // Get image tags


    // Get objects in the image


    // Get people in the image

}
else
{
    var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
    Console.WriteLine(" Analysis failed.");
    Console.WriteLine($"   Error reason : {errorDetails.Reason}");
    Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
    Console.WriteLine($"   Error message: {errorDetails.Message}\n");
}
```

**Python**

```Python
# Get image analysis
image = sdk.VisionSource(image_file)

image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:
    # Get image captions
    if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.content, result.caption.confidence * 100))

    # Get image dense captions
    if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.content, caption.confidence * 100))

    # Get image tags


    # Get objects in the image


    # Get people in the image


else:
    error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
    print(" Analysis failed.")
    print("   Error reason: {}".format(error_details.reason))
    print("   Error code: {}".format(error_details.error_code))
    print("   Error message: {}".format(error_details.message))
```
    
5. 変更を保存して、**image-analysis** フォルダーの統合ターミナルに戻り、引数 **images/street.jpg** でプログラムを実行するには、次のコマンドを入力します。

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 出力を観察します。これには、**street.jpg** 画像の推奨キャプションが含まれている必要があります。
7. 今度は引数 **images/building.jpg** を使用してプログラムを再度実行し、**building.jpg** 画像に対して生成されるキャプションを確認します。
8. 前の手順を繰り返して、**images/person.jpg** ファイルのキャプションを生成します。

## 画像の推奨タグを取得

画像の内容に関する手がかりを提供する関連*タグ*を特定すると役立つ場合があります。

1. **AnalyzeImage** 関数のコメント "**Get image tags**" の下に、次のコードを追加します。

**C#**

```C#
// Get image tags
if (result.Tags != null)
{
    Console.WriteLine($" Tags:");
    foreach (var tag in result.Tags)
    {
        Console.WriteLine($"   \"{tag.Name}\", Confidence {tag.Confidence:0.0000}");
    }
    Console.WriteLine($"\n");
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、画像のキャプションに加えて、提案されたタグのリストが表示されることを確認します。

## 画像内のオブジェクトを検出して特定する

*物体検出*は、画像内の個々のオブジェクトが識別され、その場所が境界ボックスで示される特定の形式の Computer Vision です。

1. **AnalyzeImage** 関数のコメント "**Get objects in the image**" の下に、次のコードを追加します。

**C#**

```C#
// Get objects in the image
if (result.Objects != null)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (var detectedObject in result.Objects)
    {
        Console.WriteLine($"   \"{detectedObject.Name}\", Confidence {detectedObject.Confidence:0.0000}");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Name,font,brush,r.X, r.Y);
    }
                    
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");   
}

```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_file)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.name, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、検出されるオブジェクトを監視します。 実行するたびに、コードファイルと同じフォルダーに生成された **objects.jpg** ファイルを表示して、注釈付きのオブジェクトを確認します。

## 画像内の人物を検出して特定する

*人物検出*は、画像内の個々の人物が識別され、その場所が境界ボックスで示される Computer Vision 特定の形式です。

1. **AnalyzeImage** 関数で、「**Get people in the image**」というコメントの下に、次のコードを追加します。

**C#**

```C#
// Get people in the image
if (result.People != null)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (var person in result.People)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_file)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (省略可能) **検出された人物の信頼度を返す**セクションで、**Console.Writeline** コマンドのコメントを解除し、画像の特定の位置で検出された人物について返された信頼度レベルを確認します。

3. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、検出されるオブジェクトを監視します。 実行するたびに、コードファイルと同じフォルダーに生成された **objects.jpg** ファイルを表示して、注釈付きのオブジェクトを確認します。

> **注**: 前のタスクでは、単一の方法を使用して画像を分析し、コードを段階的に追加して結果を解析および表示しました。 SDK には、キャプションの提案、タグの識別、オブジェクトの検出などの個別のメソッドも用意されています。つまり、最も適切なメソッドを使用して必要な情報のみを返し、返す必要のあるデータ ペイロードのサイズを減らすことができます。 詳細については、[.NET SDK のドキュメント](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet)または [Python SDK のドキュメント](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision)を参照してください。

## 背景を削除する、または画像の前景マット処理を生成する

場合によっては、画像の背景を削除したり、その画像の前景のマットを作成したりすることが必要な場合があります。 まず背景の削除から始めましょう。

1. コード ファイルで **BackgroundForeground** 関数を見つけ、「**Remove the background from the image or generate a foreground matte**」というコメントの下に次のコードを追加します。

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($"\nRemove the background from the image or generate a foreground matte");

using var imageSource = VisionSource.FromFile(imageFile);

var analysisOptions = new ImageAnalysisOptions()
{
    // Set the image analysis segmentation mode to background or foreground
    SegmentationMode = ImageSegmentationMode.BackgroundRemoval
};

using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

var result = analyzer.Analyze();

// Remove the background or generate a foreground matte
if (result.Reason == ImageAnalysisResultReason.Analyzed)
{
    using var segmentationResult = result.SegmentationResult;

    var imageBuffer = segmentationResult.ImageBuffer;
    Console.WriteLine($"\n Segmentation result:");
    Console.WriteLine($"   Output image buffer size (bytes) = {imageBuffer.Length}");
    Console.WriteLine($"   Output image height = {segmentationResult.ImageHeight}");
    Console.WriteLine($"   Output image width = {segmentationResult.ImageWidth}");

    string outputImageFile = "newimage.jpg";
    using (var fs = new FileStream(outputImageFile, FileMode.Create))
    {
        fs.Write(imageBuffer.Span);
    }
    Console.WriteLine($"   File {outputImageFile} written to disk\n");
}
else
{
    var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
    Console.WriteLine(" Analysis failed.");
    Console.WriteLine($"   Error reason : {errorDetails.Reason}");
    Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
    Console.WriteLine($"   Error message: {errorDetails.Message}");
    Console.WriteLine(" Did you set the computer vision endpoint and key?\n");
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemove the background from the image or generate a foreground matte')

image = sdk.VisionSource(image_file)

analysis_options = sdk.ImageAnalysisOptions()

# Set the image analysis segmentation mode to background or foreground
analysis_options.segmentation_mode = sdk.ImageSegmentationMode.BACKGROUND_REMOVAL
    
image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:

    image_buffer = result.segmentation_result.image_buffer
    print(" Segmentation result:")
    print("   Output image buffer size (bytes) = {}".format(len(image_buffer)))
    print("   Output image height = {}".format(result.segmentation_result.image_height))
    print("   Output image width = {}".format(result.segmentation_result.image_width))

    output_image_file = "newimage.jpg"
    with open(output_image_file, 'wb') as binary_file:
        binary_file.write(image_buffer)
    print("   File {} written to disk".format(output_image_file))

else:

    error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
    print(" Analysis failed.")
    print("   Error reason: {}".format(error_details.reason))
    print("   Error code: {}".format(error_details.error_code))
    print("   Error message: {}".format(error_details.message))
    print(" Did you set the computer vision endpoint and key?")
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像のコード ファイルと同じフォルダーに生成された **newimage.jpg** ファイルを開きます。  各画像から背景がどのように削除されているかに注目してください。

次に、画像の前景マットを生成してみましょう。

3. コード ファイルで **BackgroundForeground** 関数を見つけ、「**Set the image analysis segmentation mode to background or foreground**」というコメントの下に前に追加したコードを次のコードに置き換えます。

**C#**

```C#
// Set the image analysis segmentation mode to background or foreground
SegmentationMode = ImageSegmentationMode.ForegroundMatting
```

**Python**

```Python
# Set the image analysis segmentation mode to background or foreground
analysis_options.segmentation_mode = sdk.ImageSegmentationMode.FOREGROUND_MATTING
```

4. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像のコード ファイルと同じフォルダーに生成された **newimage.jpg** ファイルを開きます。  画像に対して前景マットがどのように生成されているかに注目してください。

## リソースをクリーンアップする

他のトレーニング モジュールに対してこのラボで作成された Azure リソースを使用していない場合は、それらを削除して、追加の料金が発生しないようにすることができます。 方法は以下のとおりです。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。

2. 上部の検索バーで、*Azure AI サービス マルチサービス アカウント*を検索し、このラボで作成した Azure AI サービス マルチサービス アカウント リソースを選択します。

3. [リソース] ページで **[削除]** を選択し、指示に従ってリソースを削除します。

## 詳細情報

この演習では、Azure AI Vision サービスの画像分析および操作機能のいくつかについて説明しました。 このサービスには、オブジェクトや人物を検出する機能や、その他の Computer Vision タスク機能も含まれています。

**Azure AI Vision** サービスの詳細については、「[Azure AI Vision のドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/)」を参照してください。
