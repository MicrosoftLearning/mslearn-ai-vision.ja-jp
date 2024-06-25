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
5. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## Azure AI Vision SDK を使用するための準備をする

この演習では、Azure AI Vision SDK を使用して画像を分析する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ウィンドウで、**Labfiles/01-analyze-images** フォルダーを参照し、言語の設定に応じて、**C-Sharp** フォルダーまたは **Python** フォルダーを展開します。
2. **image-analysis** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure AI Vision SDK パッケージをインストールします。

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **注**: 開発キット拡張機能のインストールを求められた場合は、そのメッセージを閉じてかまいません。

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
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
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
## ビューの画像は、あなたが分析します

この演習では、Azure AI Vision サービスを使用して複数の画像を分析します。

1. Visual Studio Code で、**image-analysis** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. 各画像ファイルを順番に選択して、Visual Studio Code で表示します。

## 画像を分析してキャプションを提案する

これで、SDK を使用して Vision サービスを呼び出し、画像を分析する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **image-analysis.py**) の **Main** 関数で、構成設定をロードするためのコードが提供されていることに注意してください。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

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

2. **Main** 関数で、先ほど追加したコードの下で、コードが画像ファイルへのパスを指定し、その画像パスを他の 2 つの関数 (**AnalyzeImage** と **BackgroundForeground**) に渡すことに注意してください。 これらの関数はまだ完全には実装されていません。

3. **AnalyzeImage** 関数のコメント "**Get result with specify features to be retrieved**" の下に、次のコードを追加します。

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
result = cv_client.analyze(
    image_data=image_data,
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.DENSE_CAPTIONS,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE],
)
```
    
4. **AnalyzeImage** 関数のコメント "**Display analysis results**" の下に、次のコードを追加します (後でコードを追加する場所を示すコメントを含む)。

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

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
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、画像のキャプションに加えて、提案されたタグのリストが表示されることを確認します。

## 画像内のオブジェクトを検出して特定する

*物体検出*は、画像内の個々のオブジェクトが識別され、その場所が境界ボックスで示される特定の形式の Computer Vision です。

1. **AnalyzeImage** 関数のコメント "**Get objects in the image**" の下に、次のコードを追加します。

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
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
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

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
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
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
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
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
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("backgroundForeground.png", "wb") as file:
    file.write(image)
print('  Results saved in backgroundForeground.png \n')
```
    
2. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像のコード ファイルと同じフォルダーに生成された **background.png** ファイルを開きます。  各画像から背景がどのように削除されているかに注目してください。

次に、画像の前景マットを生成してみましょう。

3. コード ファイルで、**BackgroundForeground** 関数を見つけ、コメント "**Define the API version and mode**" の下のモード変数を `foregroundMatting` に変更します。

4. 変更を保存し、**images** フォルダー内の画像ファイルごとにプログラムを 1 回実行し、各画像のコード ファイルと同じフォルダーに生成された **background.png** ファイルを開きます。  画像に対して前景マットがどのように生成されているかに注目してください。

## リソースをクリーンアップする

他のトレーニング モジュールに対してこのラボで作成された Azure リソースを使用していない場合は、それらを削除して、追加の料金が発生しないようにすることができます。 方法は以下のとおりです。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。

2. 上部の検索バーで、*Azure AI サービス マルチサービス アカウント*を検索し、このラボで作成した Azure AI サービス マルチサービス アカウント リソースを選択します。

3. [リソース] ページで **[削除]** を選択し、指示に従ってリソースを削除します。

## 詳細情報

この演習では、Azure AI Vision サービスの画像分析および操作機能のいくつかについて説明しました。 このサービスには、オブジェクトや人物を検出する機能や、その他の Computer Vision タスク機能も含まれています。

**Azure AI Vision** サービスの詳細については、「[Azure AI Vision のドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/)」を参照してください。
