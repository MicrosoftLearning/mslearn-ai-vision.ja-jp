---
lab:
  title: 顔の検出と分析
  module: Module 4 - Detecting and Analyze Faces
---

# 顔の検出と分析

人間の顔を検出して分析する機能は、AI のコア機能です。 この演習では、画像内の顔を操作するために使用できる 2 つの Azure AI サービスである **Azure AI Vision** サービスと **Face** サービスについて説明します。

> **重要**: このラボは、制限された機能への追加アクセスを何も要求せずに完了できます。

> **注**: 2022 年 6 月 21 日から、個人を特定できる情報を返す Azure AI サービスの機能は、[制限付きアクセス](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)が許可されているお客様に限定されます。 さらに、感情的な状態を推測する機能は使用できなくなりました。 Microsoft が行った変更と理由について詳しくは、「[顔認識に対する責任ある AI 投資と保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)」をご覧ください。

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

> **ヒント**: 既に **mslearn-ai-vision** リポジトリを最近クローンした場合は、このタスクをスキップできます。 それ以外の場合は、次の手順に従って開発環境に複製します。

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
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## Azure AI Vision SDK を使用するための準備をする

この演習では、Azure AI Vision SDK を使用して画像を分析する、部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** 用の SDK のいずれかに使用することを選択できます。 以下の手順で、希望する言語に適したアクションを実行します。

1. 使用する言語のアプリケーション コード ファイルが格納されているフォルダーに移動します。  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
    ```

1. 使用する言語に応じて適切なコマンドを実行して、Azure AI Vision SDK パッケージと必須の依存関係をインストールします。

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

1. `ls` コマンドを使用して、**computer-vision** フォルダーの内容を表示できます。 構成設定用のファイルが含まれていることに注意してください。

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

1. コード ファイル内で、含まれている構成値を更新して、Computer Vision サービス リソースの**エンドポイント**と認証**キー**を反映します。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

## 分析する画像の確認

この演習では、Azure AI Vision サービスを使用して、人物の画像を分析します。

1. Cloud Shell のツール バーで、**[ファイルのアップロード/ダウンロード]**、次に **[ダウンロード]** を選択します。 新しいダイアログ ボックスで、次のファイル パスを入力し、**[ダウンロード]** を選択します。

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. **people.jpg** 画像を開いて表示します。

## 画像内の顔を検出する

これで、SDK を使用して Computer Vision サービスを呼び出し、画像内の顔を検出する準備が整いました。

1. **computer-vision** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: detect-people.py

1. クライアント アプリケーションのコード ファイルを開く次のコマンドを入力します。

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. **Import namespaces** というコメントの下に次の言語固有のコードを追加して、Azure AI Vision SDK を使用するために必要な名前空間をインポートします。

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
    
1. **Main** 関数で、構成設定をロードするためのコードが提供されていることを確認してください。 次に、「**Authenticate Azure AI Vision client**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、Azure AI Vision クライアント オブジェクトを作成および認証します。

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
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

1. **Main** 関数の追加したコードの下で、コードが画像ファイルへのパスを指定し、その画像パスを **AnalyzeImage** という名前の関数に渡していることを確認してください。 この関数はまだ完全には実装されていません。

1. **AnalyzeImage** 関数で、「**Get result with specify features to be retrieved (PEOPLE)**」というコメントの下に、次のコードを追加します。

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

1. **AnalyzeImage** 関数で、「**Draw bounding box around detected people**」というコメントの下に、次のコードを追加します。

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

1. 変更を保存し、コード エディターを閉じて、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 検出された顔の数を示す出力を確認します。
7. コード ファイルと同じフォルダーに生成された **people.jpg** ファイルをダウンロードして、注釈付きの顔を確認します。 この場合、コードは顔の属性を使用してボックスの左上の場所にラベルを付け、境界ボックスの座標を使用して各顔の周りに長方形を描画しています。

サービスによって検出されたすべてのユーザーの信頼度スコアを表示する場合は、コメント `Return the confidence of the person detected` の下のコード行のコメントを解除し、コードを再実行できます。

## Face SDK を使用する準備をする

**Azure AI Vision** サービスは (他の多くの画像分析機能とともに) 基本的な顔検出を提供しますが、**Face** サービスは顔の分析と認識のためのより包括的な機能を提供します。

1. コマンド `cd ../face-api` を実行して、**face-api** フォルダーに移動します。 次に、言語設定に適したコマンドを実行して、Face SDK パッケージをインストールします。

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. **face-api** フォルダーの内容を表示し、構成設定用のファイルが含まれていることを確認してください。
    - **C#** : appsettings.json
    - **Python**: .env

1. 構成ファイルを開き、構成値を更新して、Azure AI サービス リソースの**エンドポイント**と認証**キー**を反映します。 変更を保存します。

1. **face-api** フォルダーに、クライアント アプリケーションの次のコード ファイルが含まれていることを確認してください。

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

1. コード ファイルを開き、上部の既存の名前空間参照の下で、「**Import namespaces**」というコメントを見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して Vision SDK を使用するために必要な名前空間をインポートします。

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. **Main** 関数で、構成設定をロードするためのコードが提供されていることを確認してください。 次に、コメント "**Authenticate Face client**" を見つけます。 次に、このコメントの下に、次の言語固有のコードを追加して、**FaceClient** オブジェクトを作成および認証します。

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

1. **Main** 関数で、追加したコードの下に、コード内の関数を呼び出して Face サービスの機能を調べることができるメニューがコードに表示されることを確認してください。 この演習の残りの部分では、これらの関数を実装します。

## 顔を検出して分析する

顔認証サービスの最も基本的な機能の 1 つは、画像内の顔を検出し、頭部姿勢、ぼやけ、マスクの存在などの属性を決定することです。

1. アプリケーションのコード ファイルの **Main** 関数で、ユーザーがメニュー オプション **1** を選択した場合に実行されるコードを調べます。 このコードは **DetectFaces** 関数を呼び出し、パスを画像ファイルに渡します。
1. コード ファイルで **DetectFaces** 関数を見つけて、コメント "**Specify facial features to be retrieved**" の下に、次のコードを追加します。

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

1. **DetectFaces** 関数に追加したコードの下で、コメント "**Get faces**" を見つけて、次のコードを追加します。

    **C#**

    ```C#
    // Get faces
    using (var imageData = File.OpenRead(imageFile))
    {    
        var response = await faceClient.DetectAsync(
            BinaryData.FromStream(imageData),
            FaceDetectionModel.Detection03,
            FaceRecognitionModel.Recognition04,
            returnFaceId: false,
            returnFaceAttributes: features);
        IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

        if (detected_faces.Count() > 0)
        {
            Console.WriteLine($"{detected_faces.Count()} faces detected.");

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

            int faceCount=0;

            // Draw and annotate each face
            foreach (var face in detected_faces)
            {
                faceCount++;
                Console.WriteLine($"\nFace number {faceCount}");
            
                // Get face properties
                Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
                Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
                Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
                Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
                Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

                // Draw and annotate face
                var r = face.FaceRectangle;

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
        }
    }
    ```

    **Python**

    ```Python
    # Get faces
    with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=False,
            return_face_attributes=features,
        )

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

                print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
                print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
                print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
                print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
                print(' - Mask: {}'.format(face.face_attributes.mask.type))

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

1. **DetectFaces** 関数に追加したコードを調べます。 画像ファイルを分析し、頭部姿勢、ぼやけ、マスクの存在などの属性も含めて、その画像ファイルに含まれるすべての顔を検出します。 各顔に割り当てられた一意の顔識別子を含む、各顔の詳細が表示されます。顔の位置は、境界ボックスを使用して画像に示されます。
1. 変更を保存し、コード エディターを閉じてから、次のコマンドを入力してプログラムを実行します。

    **C#**

    ```
    dotnet run
    ```

    *C# 出力に、**await** 演算子を使用している非同期関数に関する警告が表示される場合があります。これは無視してもかまいません。*

    **Python**

    ```
    python analyze-faces.py
    ```

1. プロンプトが表示されたら、「**1**」と入力し、出力を観察します。ここには、検出された各顔の ID と属性が含まれているはずです。
1. コード ファイルと同じフォルダーに生成された **detected_faces.jpg** ファイルをダウンロードして、注釈付きの顔を確認します。

## リソースをクリーンアップする

このラボで作成した Azure リソースを他のトレーニング モジュールに使用していない場合は、それらを削除して、追加料金が発生しないようにすることができます。

1. `https://portal.azure.com` で Azure portal を開き、上部の検索バーで、このラボで作成したリソースを検索します。

1. [リソース] ページで **[削除]** を選択し、指示に従ってリソースを削除します。 または、リソース グループ全体を削除して、すべてのリソースを同時にクリーンアップすることもできます。

## 詳細

**Face** サービスにはいくつかの追加機能がありますが、[責任ある AI 標準](https://aka.ms/aah91ff)に従うと、制限付きアクセス ポリシーで制限されます。 これらの機能には、顔認識モデルの識別、検証、作成が含まれます。 詳細とアクセスの申請については、「[Azure AI サービスの制限付きアクセス](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)」を参照してください。

顔検出に **Azure AI Vision** サービスを使用する方法の詳細については、「[Azure AI Vision のドキュメント](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)」を参照してください。

**Face** サービスの詳細については、[Face のドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity)を参照してください。
