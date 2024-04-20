---
lab:
  title: 顔の検出と分析
  module: Module 4 - Detecting and Analyze Faces
---

# 顔の検出と分析

人間の顔を検出して分析する機能は、AI のコア機能です。 この演習では、画像内の顔を操作するために使用できる 2 つの Azure AI サービスである **Azure AI Vision** サービスと **Face** サービスについて説明します。

> **重要**: このラボは、制限された機能への追加アクセスを何も要求せずに完了できます。

> **注**: 2022 年 6 月 21 日から、個人を特定できる情報を返す Azure AI サービスの機能は、[制限付きアクセス](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)が許可されているお客様に限定されます。 さらに、感情的な状態を推測する機能は使用できなくなりました。 Microsoft が行った変更と理由について詳しくは、「[顔認識に対する責任ある AI 投資と保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)」をご覧ください。

## このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-ai-vision` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

## Azure AI サービス リソースをプロビジョニングする

サブスクリプションに **Azure AI サービス** リソースがまだない場合は、プロビジョニングする必要があります。

1. Azure portal (`https://portal.azure.com`) を開き、ご利用の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで 「*Azure AI サービス*」を検索し、**[Azure AI サービス]** を選択し、次の設定で Azure AI サービス マルチサービス アカウント リソースを作成します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを選択または作成します (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がないことがあります。提供されているものを使ってください)*
    - **[リージョン]**: 使用できるリージョンを選択します**
    - **[名前]**: *一意の名前を入力します*
    - **価格レベル**: Standard S0
3. 必要なチェック ボックスをオンにして、リソースを作成します。
4. デプロイが完了するまで待ち、デプロイの詳細を表示します。
5. リソースがデプロイされたら、そこに移動して、その **[キーとエンドポイント]** ページを表示します。 次の手順では、このページのエンドポイントとキーの 1 つが必要になります。

## Azure AI Vision SDK を使用するための準備をする

この演習では、Azure AI Vision SDK を使用して画像内の顔を分析する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**04-face** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **computer-vision** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Azure AI Vision SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 0.15.1-beta.1
    ```

    **Python**

    ```
    pip install azure-ai-vision==0.15.1b1
    ```
    
3. **computer-vision** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

4. 構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。

5. **computer-vision** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: detect-people.py

6. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision SDK を使用するために必要な名前空間をインポートします。

    **C#**

    ```C#
    // import namespaces
    using Azure.AI.Vision.Common;
    using Azure.AI.Vision.ImageAnalysis;
    ```

    **Python**

    ```Python
    # import namespaces
    import azure.ai.vision as sdk
    ```

## 分析する画像の確認

この演習では、Azure AI Vision サービスを使用して、人物の画像を分析します。

1. Visual Studio Code で、**computer-vision** フォルダーとそれに含まれる **images** フォルダーを展開します。
2. **people.jpg** 画像を選択して表示します。

## 画像内の顔を検出する

これで、SDK を使用して Computer Vision サービスを呼び出し、画像内の顔を検出する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **detect-people.py**) の **Main** 関数で、構成設定を読み込むためのコードが提供されていることを確認してください。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

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

2. **Main** 関数の追加したコードの下で、コードが画像ファイルへのパスを指定し、その画像パスを **AnalyzeImage** という名前の関数に渡していることを確認してください。 この関数はまだ完全には実装されていません。

3. **AnalyzeImage** 関数で、「**Specify features to be retrieved (PEOPLE)**」というコメントの下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify features to be retrieved (PEOPLE)
    Features =
        ImageAnalysisFeature.People
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (PEOPLE)
    analysis_options = sdk.ImageAnalysisOptions()
    
    features = analysis_options.features = (
        sdk.ImageAnalysisFeature.PEOPLE
    )    
    ```

4. **AnalyzeImage** 関数で、「**Get image analysis**」というコメントの下に、次のコードを追加します。

    **C#**

    ```C
    // Get image analysis
    using var imageSource = VisionSource.FromFile(imageFile);
    
    using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);
    
    var result = analyzer.Analyze();
    
    if (result.Reason == ImageAnalysisResultReason.Analyzed)
    {
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
                // Draw object bounding box if confidence > 50%
                if (person.Confidence > 0.5)
                {
                    // Draw object bounding box
                    var r = person.BoundingBox;
                    Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
                    graphics.DrawRectangle(pen, rect);
        
                    // Return the confidence of the person detected
                    Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
                }
            }
        
            // Save annotated image
            String output_file = "detected_people.jpg";
            image.Save(output_file);
            Console.WriteLine("  Results saved in " + output_file + "\n");
        }
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
                # Draw object bounding box if confidence > 50%
                if detected_people.confidence > 0.5:
                    # Draw object bounding box
                    r = detected_people.bounding_box
                    bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
                    draw.rectangle(bounding_box, outline=color, width=3)
            
                    # Return the confidence of the person detected
                    print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
                    
            # Save annotated image
            plt.imshow(image)
            plt.tight_layout(pad=0)
            outputfile = 'detected_people.jpg'
            fig.savefig(outputfile)
            print('  Results saved in', outputfile)
    
    else:
        error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
        print(" Analysis failed.")
        print("   Error reason: {}".format(error_details.reason))
        print("   Error code: {}".format(error_details.error_code))
        print("   Error message: {}".format(error_details.message))
    ```

5. 変更を保存し、**computer-vision** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 検出された顔の数を示す出力を確認します。
7. コード ファイルと同じフォルダーに生成された **detected_people.jpg** ファイルを表示して、注釈付きの顔を確認します。 この場合、コードは顔の属性を使用してボックスの左上の場所にラベルを付け、境界ボックスの座標を使用して各顔の周りに長方形を描画しています。

## Face SDK を使用する準備をする

**Azure AI Vision** サービスは (他の多くの画像分析機能とともに) 基本的な顔検出を提供しますが、**Face** サービスは顔の分析と認識のためのより包括的な機能を提供します。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**04-face** フォルダーを参照し、言語の設定に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **face-api** フォルダーを右クリックして、統合ターミナルを開きます。 次に、言語設定に適したコマンドを実行して、Face SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.8.0-preview.3
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.6.0
    ```
    
3. **face-api** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

4. 構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。

5. **face-api** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して Vision SDK を使用するために必要な名前空間をインポートします。

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. **Main** 関数で、構成設定をロードするためのコードが提供されていることを確認してください。 次に、コメント "**Authenticate Face client**" を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、**FaceClient** オブジェクトを作成および認証します。

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. **Main** 関数で、追加したコードの下に、コード内の関数を呼び出して Face サービスの機能を調べることができるメニューがコードに表示されることを確認してください。 この演習の残りの部分では、これらの関数を実装します。

## 顔を検出して分析する

Face サービスの最も基本的な機能の 1 つは、画像内の顔を検出し、頭部姿勢、ぼやけ、眼鏡の存在などの属性を決定することです。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードは **DetectFaces** 関数を呼び出し、パスを画像ファイルに渡します。
2. コード ファイルで **DetectFaces** 関数を見つけて、コメント "**Specify facial features to be retrieved**" の下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify facial features to be retrieved
    IList<FaceAttributeType> features = new FaceAttributeType[]
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. **DetectFaces** 関数に追加したコードの下で、コメント "**Get faces**" を見つけて、次のコードを追加します。

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count() > 0)
    {
        Console.WriteLine($"{detected_faces.Count()} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.White);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face number {faceCount}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face number {}'.format(face_count)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. **DetectFaces** 関数に追加したコードを調べます。 画像ファイルを分析し、オクルージョン、ぼかし、眼鏡の有無などの属性を含め、画像ファイルに含まれるすべての顔を検出します。 各顔に割り当てられた一意の顔識別子を含む、各顔の詳細が表示されます。顔の位置は、境界ボックスを使用して画像に示されます。
5. 変更を保存して **face-api** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. プロンプトが表示されたら、「**1**」と入力し、出力を観察します。ここには、検出された各顔の ID と属性が含まれているはずです。
7. コード ファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルを表示して、注釈付きの顔を確認します。

## 詳細情報

**Face** サービスにはいくつかの追加機能がありますが、[責任ある AI 標準](https://aka.ms/aah91ff)に従うと、制限付きアクセス ポリシーで制限されます。 これらの機能には、顔認識モデルの識別、検証、作成が含まれます。 詳細とアクセスの申請については、「[Azure AI サービスの制限付きアクセス](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)」を参照してください。

顔検出に **Azure AI Vision** サービスを使用する方法の詳細については、「[Azure AI Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)」を参照してください。

**Face** サービスの詳細については、[Face のドキュメント](https://docs.microsoft.com/azure/cognitive-services/face/)を参照してください。
