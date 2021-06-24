---
lab:
    title: '얼굴 감지, 분석 및 인식'
    module: '모듈 10 - 얼굴 감지, 분석 및 인식'
---

# 얼굴 감지, 분석 및 인식

사람의 얼굴을 감지, 분석 및 인식하는 기능은 AI의 핵심 기능입니다. 이 연습에서는 이미지에 포함된 얼굴로 작업을 하는 데 사용할 수 있는 두 가지 Azure Cognitive Services인 **Computer Vision** 서비스와 **Face** 서비스에 대해 살펴봅니다.

## 이 과정용 리포지토리 복제

이 과정용 코드 리포지토리를 아직 복제하지 않았으면 복제해야 합니다.

1. Visual Studio Code를 시작합니다.
2. 팔레트를 열고(Shift+Ctrl+P 누르기) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/AI-102KO-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버그에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## Cognitive Services 리소스 프로비전

구독에 **Cognitive Services** 리소스가 아직 없으면 리소스를 프로비전해야 합니다.

1. Azure Portal `https://portal.azure.com`을 열고 Azure 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
2. **&#65291;리소스 만들기** 단추를 선택하고 *cognitive services*를 검색한 후에 다음 설정을 사용하여 **Cognitive Services** 리소스를 만듭니다.
    - **구독**: *사용자의 Azure 구독*
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기(제한된 구독을 사용 중이라면 새 리소스 그룹을 만들 권한이 없을 수도 있으므로 제공된 리소스 그룹 사용)*
    - **지역**: *사용 가능한 아무 지역이나 선택*
    - **이름**: *고유한 이름 입력*
    - **가격 책정 계층**: 표준 S0
3. 필요한 체크박스를 선택하여 리소스를 만듭니다.
4. 배포가 완료될 때까지 기다렸다가 배포 세부 정보를 확인합니다.
5. 리소스가 배포되면 해당 리소스로 이동하여 **키 및 엔드포인트** 페이지를 확인합니다. 다음 절차에서 이 페이지에 표시되는 키 중 하나와 엔드포인트가 필요합니다.

## Computer Vision SDK 사용 준비

이 연습에서는 Computer Vision SDK를 사용해 이미지의 얼굴을 분석하는 부분 구현 클라이언트 애플리케이션을 완성합니다.

> **참고**: **C#** 또는 **Python**용 SDK 사용을 선택할 수 있습니다. 아래 단계에서 선호하는 언어에 적합한 작업을 수행하세요.

1. Visual Studio Code의 **탐색기** 창에서 **19-face** 폴더를 찾은 다음 언어 기본 설정에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. **computer-vision** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음 언어 기본 설정에 적합한 명령을 실행하여 Computer Vision SDK 패키지를 설치합니다.

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. **computer-vision** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.
    - **C#**: appsettings.json
    - **Python**: .env

4. 구성 파일을 열고 Cognitive Service 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.

5. **computer-vision** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: detect-faces.py

6. 코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Computer Vision SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**

    ```C#
    // 네임스페이스 가져오기
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
    ```

    **Python**

    ```Python
    # 네임스페이스 가져오기
    from azure.cognitiveservices.vision.computervision import ComputerVisionClient
    from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
    from msrest.authentication import CognitiveServicesCredentials
    ```

## 분석할 이미지 확인

이 연습에서는 Computer Vision 서비스를 사용하여 사람 이미지를 분석합니다.

1. Visual Studio Code에서 **computer-vision** 폴더와 이 폴더에 포함된 **images** 폴더를 차례로 확장합니다.
2. **people.jpg** 이미지를 선택하여 표시합니다.

## 이미지에서 얼굴 감지

이제 SDK를 사용해 Computer Vision 서비스를 호출하고 이미지의 얼굴을 감지할 준비가 되었습니다.

1. 클라이언트 애플리케이션용 코드 파일(**Program.cs** 또는 **detect-faces.py**)의 **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Computer Vision 클라이언트 인증** 주석을 찾습니다. 그 후에 이 주석 아래에 다음 언어별 코드를 추가하여 Computer Vision 클라이언트 개체를 만들고 인증합니다.

    **C#**

    ```C#
    // Computer Vision 클라이언트 인증
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    cvClient = new ComputerVisionClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Computer Vision 클라이언트 인증
    credential = CognitiveServicesCredentials(cog_key) 
    cv_client = ComputerVisionClient(cog_endpoint, credential)
    ```

2. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 이미지 파일 경로를 지정한 다음 **AnalyzeFaces** 함수로 해당 이미지 경로를 전달함을 확인합니다. 이 함수는 아직 완전히 구현되지 않았습니다.

3. **AnalyzeFaces** 함수의 **검색할 기능(얼굴) 지정** 주석 아래에 다음 코드를 추가합니다.

    **C#**

    ```C#
    // 검색할 기능(얼굴) 지정
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # 검색할 기능(얼굴) 지정
    features = [VisualFeatureTypes.faces]
    ```

4. **AnalyzeFaces** 함수의 **이미지 분석 가져오기** 주석 아래에 다음 코드를 추가합니다.

**C#**

```C
// 이미지 분석 가져오기
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // 얼굴 가져오기
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // 드로잉용 이미지 준비
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // 각 얼굴을 그리고 주석 추가
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person aged approximately {face.Age}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // 주석을 추가한 이미지 저장
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# 이미지 분석 가져오기
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # 얼굴 가져오기
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # 드로잉용 이미지 준비
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # 각 얼굴을 그리고 주석 추가
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person aged approximately {}'.format(face.age)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # 주석을 추가한 이미지 저장
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. 변경 내용을 저장하고 **computer-vision** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. 출력을 살펴봅니다. 감지된 얼굴 수가 표시됩니다.
7. 코드 파일과 같은 폴더에 생성된 **detected_faces.jpg** 파일을 표시하여 주석이 추가된 얼굴을 확인합니다. 여기서 코드는 얼굴 특성을 사용해 이미지에 나와 있는 각 사람의 연령을 추정했습니다. 그리고 경계 상자 좌표를 사용해 각 얼굴 주위에 사각형을 그렸습니다.

## Face SDK 사용 준비

**Computer Vision** 서비스는 기본적인 얼굴 감지 기능을 제공(기타 여러 이미지 분석 기능도 제공함)하는 반면 **Face** 서비스에서는 얼굴 분석 및 인식을 위한 더욱 포괄적인 기능을 제공합니다.

1. Visual Studio Code의 **탐색기** 창에서 **19-face** 폴더를 찾은 다음 언어 기본 설정에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. **face-api** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음 언어 기본 설정에 적합한 명령을 실행하여 Face SDK 패키지를 설치합니다.

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. **face-api** 폴더의 내용을 표시하여 구성 설정용 파일이 포함되어 있음을 확인합니다.
    - **C#**: appsettings.json
    - **Python**: .env

4. 구성 파일을 열고 Cognitive Service 리소스용 **엔드포인트** 및 인증 **키**를 반영하여 해당 파일에 포함된 구성 값을 업데이트합니다. 변경 내용을 저장합니다.

5. **face-api** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

6. 코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Computer Vision SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**

    ```C#
    // 네임스페이스 가져오기
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # 네임스페이스 가져오기
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. **Main** 함수에서 구성 설정 로드를 위한 코드가 제공되어 있음을 확인합니다. 그런 다음 **Face 클라이언트 인증** 주석을 찾습니다. 그 후에 이 주석 아래에 다음 언어별 코드를 추가하여 **FaceClient** 개체를 만들고 인증합니다.

    **C#**

    ```C#
    // Face 클라이언트 인증
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Face 클라이언트 인증
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. **Main** 함수의 방금 추가한 코드 아래에 있는 코드가 메뉴를 표시함을 확인합니다. 이 메뉴를 사용하면 코드의 함수를 호출하여 Face 서비스 기능을 살펴볼 수 있습니다. 이 연습의 나머지 부분에서 이러한 함수를 구현합니다.

## 얼굴 감지 및 분석

Face 서비스의 가장 기본적인 기능 중 하나는 이미지의 얼굴을 감지하고 해당 특성을 확인하는 것입니다. 이러한 특성으로는 연령, 감정 표현, 머리 색, 안경 유무 등이 있습니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **1**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **DetectFaces** 함수를 호출하여 이미지 파일의 경로를 전달합니다.
2. 코드 파일에서 **DetectFaces** 함수를 찾은 다음 **검색할 얼굴 기능 지정** 주석 아래에 다음 코드를 추가합니다.

    **C#**

    ```C#
    // 검색할 얼굴 기능 지정
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Age,
        FaceAttributeType.Emotion,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # 검색할 얼굴 기능 지정
    features = [FaceAttributeType.age,
                FaceAttributeType.emotion,
                FaceAttributeType.glasses]
    ```

3. **DetectFaces** 함수의 방금 추가한 코드 아래에서 **얼굴 가져오기** 주석을 찾은 후 다음 코드를 추가합니다.

**C#**

```C
// 얼굴 가져오기
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // 드로잉용 이미지 준비
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // 각 얼굴을 그리고 주석 추가
        foreach (var face in detected_faces)
        {
            // 얼굴 속성 가져오기
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // 얼굴을 그리고 주석 추가
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // 주석을 추가한 이미지 저장
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# 얼굴 가져오기
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # 드로잉용 이미지 준비
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # 각 얼굴을 그리고 주석 추가
        for face in detected_faces:

            # 얼굴 속성 가져오기
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # 얼굴을 그리고 주석 추가
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # 주석을 추가한 이미지 저장
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. **DetectFaces** 함수에 추가한 코드를 살펴봅니다. 이 코드는 이미지 파일을 분석하여 해당 파일에 포함된 얼굴을 감지합니다. 이때 연령, 감정, 안경 유무 특성도 함께 감지합니다. 각 얼굴에 할당되는 고유 얼굴 식별자를 비롯한 각 얼굴의 세부 정보가 표시됩니다. 그리고 경계 상자를 사용하여 이미지상의 얼굴 위치가 표시됩니다.
5. 변경 내용을 저장하고 **face-api** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    *이제 C# 출력에 **await** 연산자를 사용하는 비동기 함수 관련 경고가 표시될 수 있습니다. 이러한 경고는 무시할 수 있습니다.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 메시지가 표시되면 **1**을 입력하고 출력을 살펴봅니다. 출력에는 감지된 각 얼굴의 ID와 특성이 포함되어 있습니다.
7. 코드 파일과 같은 폴더에 생성된 **detected_faces.jpg** 파일을 표시하여 주석이 추가된 얼굴을 확인합니다.

## 얼굴 비교

얼굴을 분석할 때는 여러 얼굴을 비교하여 비슷한 얼굴을 찾는 작업을 흔히 수행합니다. 이 시나리오에서는 이미지에 나와 있는 사람을 *식별*할 필요는 없습니다. 여러 이미지에 *같은* 사람(또는 최소한 비슷하게 생긴 사람)이 표시되어 있는지만 확인하면 됩니다. 

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **2**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **CompareFaces** 함수를 호출하여 이미지 파일 2개(**person1.jpg** 및 **people.jpg**)의 경로를 전달합니다.
2. 코드 파일에서 **CompareFaces** 함수를 찾은 다음 콘솔에 메시지를 인쇄하는 기존 코드 아래에 다음 코드를 추가합니다.

**C#**

```C
// 이미지 1의 얼굴이 이미지 2에도 있는지 확인
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // 이미지 1의 첫 번째 얼굴 가져오기
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        // 이미지 2의 모든 얼굴 가져오기
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // 얼굴 가져오기
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // 드로잉용 이미지 준비
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // 각 얼굴을 그리고 주석 추가
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // 얼굴을 그리고 주석 추가
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // 주석을 추가한 이미지 저장
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# 이미지 1의 얼굴이 이미지 2에도 있는지 확인
with open(image_1, mode="rb") as image_data:
    # 이미지 1의 첫 번째 얼굴 가져오기
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # 이미지의 얼굴 강조 표시
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# 이미지 2의 모든 얼굴 가져오기
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # 이미지 2에서 이미지 1의 얼굴과 비슷한 얼굴 찾기
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # 드로잉용 이미지 준비
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # 일치하는 얼굴을 그리고 주석 추가
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # 주석을 추가한 이미지 저장
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. **CompareFaces** 함수에 추가한 코드를 살펴봅니다. 이 코드는 이미지 1에서 얼굴을 찾은 다음 새 이미지 파일 **face_to_match.jpg**에서 얼굴에 주석을 추가합니다. 그런 후에 이미지 2에서 모든 얼굴을 찾은 다음 해당 얼굴 ID를 사용하여 이미지 1의 얼굴과 비슷한 얼굴을 찾습니다. 비슷한 얼굴은 주석이 추가되어 새 이미지 파일 **matched_faces.jpg**에 저장됩니다.
4. 변경 내용을 저장하고 **face-api** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    *이제 C# 출력에 **await** 연산자를 사용하는 비동기 함수 관련 경고가 표시될 수 있습니다. 이러한 경고는 무시할 수 있습니다.*

    **Python**

    ```
    python analyze-faces.py
    ```
    
5. 메시지가 표시되면 **2**을 입력하고 출력을 살펴봅니다. 그런 다음 코드 파일과 같은 폴더에 생성된 **face_to_match.jpg** 및 **matched_faces.jpg** 파일을 표시하여 일치하는 얼굴을 확인합니다.
6. 메뉴 옵션 **2**를 선택하면 **person2.jpg**를 **people.jpg**에 비교하도록 **Main** 함수의 코드를 편집한 다음 프로그램을 다시 실행하여 결과를 확인합니다.

## 얼굴 인식 모델 학습 진행

AI 애플리케이션이 얼굴을 인식할 수 있는 특정인의 모델을 유지하려는 경우가 있을 수 있습니다. 얼굴 인식 기능을 사용하여 보안 건물에서 직원의 신원을 확인하는 생체 인식 보안 시스템을 원활하게 사용하려는 경우를 예로 들 수 있습니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **3**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **TrainModel** 함수를 호출하여 Cognitive Services 리소스에 등록할 새 **PersonGroup**용 이름과 ID, 그리고 직원 이름 목록을 전달합니다.
2. **face-api/images** 폴더에 이름이 직원 이름과 같은 폴더가 있음을 확인합니다. 각 폴더에는 이름이 지정된 직원의 이미지가 여러 개 포함되어 있습니다.
3. 코드 파일에서 **TrainModel** 함수를 찾은 다음 콘솔에 메시지를 인쇄하는 기존 코드 아래에 다음 코드를 추가합니다.

**C#**

```C
// 그룹이 이미 있으면 삭제
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// 그룹 만들기
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// 그룹에 각 직원 추가
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // 해당 직원 추가
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // 해당 직원의 여러 사진 추가
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // 모델 학습
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// 그룹의 직원 목록 가져오기
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# 그룹이 이미 있으면 삭제
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# 그룹 만들기
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# 그룹에 각 직원 추가
print('Adding people to the group...')
for person_name in image_folders:
    해당 직원 추가
    person = face_client.person_group_person.create(group_id, person_name)

    해당 직원의 여러 사진 추가
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# 모델 학습
print('Training model...')
face_client.person_group.train(group_id)

# 그룹의 직원 목록 가져오기
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. **TrainModel** 함수에 추가한 코드를 살펴봅니다. 이 코드는 다음 작업을 수행합니다.
    - 서비스에 등록된 **PersonGroup** 목록을 가져오고 그룹이 이미 있으면 지정된 그룹을 삭제합니다.
    - 지정된 ID와 이름으로 그룹을 만듭니다.
    - 각기 이름이 지정된 그룹에 직원을 추가하고 각 직원의 이미지를 여러 개 추가합니다.
    - 그룹에 추가한 이름이 지정된 직원과 해당 얼굴 이미지를 기반으로 하여 얼굴 인식 모델 학습을 진행합니다.
    - 그룹에 추가한 이름이 지정된 직원 목록을 검색하여 표시합니다.
5. 변경 내용을 저장하고 **face-api** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    *이제 C# 출력에 **await** 연산자를 사용하는 비동기 함수 관련 경고가 표시될 수 있습니다. 이러한 경고는 무시할 수 있습니다.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 메시지가 표시되면 **3**을 입력하고 출력을 살펴봅니다. **PersonGroup**은 직원 2명을 만들었습니다.

## 얼굴 인식

이제 **PeopleGroup**을 정의하고 얼굴 인식 모델을 학습시켰으므로 해당 모델을 사용하여 이미지에서 이름이 지정된 사람을 인식할 수 있습니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **4**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **RecognizeFaces** 함수를 호출하여 이미지 파일(**people.jpg**)의 경로와 얼굴 식별에 사용할 **PeopleGroup**의 ID를 전달합니다.
2. 코드 파일에서 **RecognizeFaces** 함수를 찾은 다음 콘솔에 메시지를 인쇄하는 기존 코드 아래에 다음 코드를 추가합니다.

**C#**

```C
// 이미지에서 얼굴 감지
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // 얼굴 가져오기
    if (detectedFaces.Count > 0)
    {
        
        // 얼굴 ID 목록 가져오기
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // 직원 그룹의 얼굴 식별
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // 인식된 얼굴의 이름 가져오기
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // 이미지의 얼굴에 주석 추가
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // 얼굴이 인식되면 이름으로 녹색 주석 추가
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // 얼굴이 인식되지 않으면 인식되지 않은 얼굴에 자홍색 주석만 추가
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // 주석을 추가한 이미지 저장
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# 이미지에서 얼굴 감지
with open(image_file, mode="rb") as image_data:

    # 얼굴 가져오기
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    얼굴 ID 목록 가져오기
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    직원 그룹의 얼굴 식별
    recognized_faces = face_client.face.identify(face_ids, group_id)

    인식된 얼굴의 이름 가져오기
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    이미지의 얼굴에 주석 추가
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            얼굴이 인식되면 이름으로 녹색 주석 추가
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            얼굴이 인식되지 않으면 인식되지 않은 얼굴에 자홍색 주석만 추가
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # 주석을 추가한 이미지 저장
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. **RecognizeFaces** 함수에 추가한 코드를 살펴봅니다. 이 코드는 이미지에서 얼굴을 찾은 다음 얼굴 ID 목록을 만듭니다. 그런 다음 직원 그룹을 사용해 얼굴 ID 목록의 얼굴 식별을 시도합니다. 인식된 얼굴에는 식별된 사람의 이름으로 주석이 추가되며 결과는 **recognized_faces.jpg**에 저장됩니다.
4. 변경 내용을 저장하고 **face-api** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    *이제 C# 출력에 **await** 연산자를 사용하는 비동기 함수 관련 경고가 표시될 수 있습니다. 이러한 경고는 무시할 수 있습니다.*

    **Python**

    ```
    python analyze-faces.py
    ```

5. 메시지가 표시되면 **4**을 입력하고 출력을 살펴봅니다. 그런 다음 코드 파일과 같은 폴더에 생성된 **recognized_faces.jpg** 파일을 표시하여 식별된 직원을 확인합니다.
6. 메뉴 옵션 **4**를 선택하면 **person2.jpg**의 얼굴을 인식하도록 **Main** 함수의 코드를 편집한 다음 프로그램을 다시 실행하여 결과를 확인합니다. 같은 직원이 인식되어야 합니다.

## 얼굴 확인

얼굴 인식은 신원 확인에 사용되는 경우가 많습니다. Face 서비스를 사용하면 이미지의 얼굴을 다른 얼굴 또는 **PersonGroup**에 등록된 사람의 얼굴과 비교하는 방식으로 얼굴을 확인할 수 있습니다.

1. 애플리케이션용 코드 파일의 **Main** 함수에서 사용자가 메뉴 옵션 **5**을 선택하면 실행되는 코드를 살펴봅니다. 이 코드는 **VerifyFace** 함수를 호출하여 이미지 파일(**person1.jpg**)의 경로, 이름, 그리고 얼굴 식별에 사용할 **PeopleGroup**의 ID를 전달합니다.
2. 코드 파일에서 **VerifyFaces** 함수를 찾은 다음 **직원 그룹의 직원 ID 가져오기** 주석 아래(결과를 인쇄하는 코드 위)에 다음 코드를 추가합니다.

**C#**

```C
// 직원 그룹의 직원 ID 가져오기
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // 이미지의 첫 번째 얼굴 가져오기
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                // 얼굴과 ID가 확인되면 일치 여부 확인
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# 직원 그룹의 직원 ID 가져오기
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        이미지의 첫 번째 얼굴 가져오기
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                # 얼굴과 ID가 확인되면 일치 여부 확인
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. **VerifyFaces** 함수에 추가한 코드를 살펴봅니다. 이 코드는 지정된 이름을 사용하여 그룹의 직원 ID를 조회합니다. 해당 직원이 있으면 코드는 이미지에 있는 첫 번째 얼굴의 얼굴 ID를 가져옵니다. 마지막으로 이미지에 얼굴이 있으면 코드는 지정된 직원의 ID와 해당 얼굴을 대조하여 확인합니다.
4. 변경 내용을 저장하고 **face-api** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python analyze-faces.py
    ```

5. 메시지가 표시되면 **5**를 입력하고 결과를 살펴봅니다.
6. **Main** 함수에서 메뉴 옵션 **5**를 선택하면 실행되는 코드를 편집하여 **person1.jpg** 및 **person2.jpg** 이미지와 이름의 다양한 조합으로 코드를 실행해 봅니다.

## 자세한 정보

얼굴 감지에 **Computer Vision** 서비스를 사용하는 방법에 대한 자세한 내용은 [Computer Vision 설명서](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)를 참조하세요.

**Face** 서비스에 대해 자세히 알아보려면 [Face 설명서](https://docs.microsoft.com/azure/cognitive-services/face/)를 참조하세요.
