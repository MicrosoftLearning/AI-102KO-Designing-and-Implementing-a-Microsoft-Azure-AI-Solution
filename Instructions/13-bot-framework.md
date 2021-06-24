---
lab:
    title: 'Bot Framework SDK를 사용하여 봇 만들기'
    module: '모듈 7 - 대화형 AI 및 Azure Bot Service'
---

# Bot Framework SDK를 사용하여 봇 만들기

*봇*은 인간 사용자와의 대화에 참여할 수 있는 소프트웨어 에이전트입니다. Microsoft Bot Framework는 Azure Bot Service를 통해 클라우드 서비스로 제공할 수 있는 봇 빌드용으로 포괄적인 플랫폼을 제공합니다.

이 연습에서는 Microsoft Bot Framework SDK를 사용하여 봇을 만들고 배포합니다.

## 시작하기 전 확인 사항

먼저 봇 개발을 위한 환경을 준비하겠습니다.

### Bot Framework Emulator 업데이트

Bot Framework SDK를 사용하여 봇을 만들고, Bot Framework Emulator를 사용하여 봇을 테스트합니다. Bot Framework Emulator는 정기적으로 업데이트되므로 최신 버전이 설치되어 있는지 확인해보겠습니다.

> **참고**: 업데이트에는 이 연습의 지침에 영향을 주는 사용자 인터페이스 변경 내용이 포함되어 있을 수 있습니다.

1. **Bot Framework Emulator**를 시작하고, 업데이트를 설치하라는 메시지가 표시되면 현재 로그인되어 있는 사용자용으로 업데이트를 설치합니다. 자동으로 메시지가 표시되지 않으면 **도움말** 메뉴의 **업데이트 확인** 옵션을 사용하여 메뉴를 확인합니다.
2. 사용 가능한 업데이트를 설치한 후에는 나중에 다시 필요하게 될 때까지 Bot Framework Emulator를 닫습니다.

### 이 과정용 리포지토리 복제

이 랩에서 작업을 수행 중인 환경에 **AI-102-AIEngineer** 코드 리포지토리를 아직 복제하지 않았다면 다음 단계에 따라 리포지토리를 지금 복제합니다. 리포지토리를 복제한 경우에는 Visual Studio Code에서 복제한 폴더를 엽니다.

1. Visual Studio Code를 시작합니다.
2. 팔레트를 열고(Shift+Ctrl+P 누르기) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버그에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 봇 만들기

Bot Framework SDK를 사용하면 템플릿을 기반으로 봇을 만든 다음 구체적인 요구 사항을 충족하도록 코드를 사용자 지정할 수 있습니다.

> **참고**: 이 연습에서는 **C#** 또는 **Python**을 선택할 수 있습니다. 아래 단계에서 선호하는 언어에 적합한 작업을 수행하세요.

1. Visual Studio Code의 **탐색기** 창에서 **13-bot-framework** 폴더를 찾은 다음 언어 기본 설정에 따라 **C-Sharp** 또는 **Python** 폴더를 확장합니다.
2. 선택한 언어의 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다.
3. 터미널에서 다음 명령을 실행하여 필요한 봇 템플릿과 패키지를 설치합니다.

**C#**

```
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. 템플릿과 패키지를 설치한 후 다음 명령을 실행하여 *EchoBot* 템플릿을 기반으로 봇을 만듭니다.

**C#**

```
dotnet new echobot -n TimeBot
```

**Python**

```
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Python을 사용 중인 경우 cookiecutter에서 메시지가 표시되면 다음 세부 정보를 입력합니다.
- **bot_name**: TimeBot
- **bot_description**: A bot for our times
    
5. 터미널 창에서 다음 명령을 입력하여 현재 디렉터리를 **TimeBot** 폴더로 변경한 다음 봇용으로 생성된 코드 파일 목록을 표시합니다.

    ```
    cd TimeBot
    dir
    ```

## Bot Framework Emulator에서 봇 테스트

*EchoBot* 템플릿을 기반으로 봇을 만들었습니다. 이제 Bot Framework Emulator(시스템에 설치되어 있음)를 사용하여 봇을 로컬에서 실행해 테스트할 수 있습니다.

1. 터미널 창에서 현재 디렉터리가 봇 코드 파일이 포함된 **TimeBot** 폴더인지 확인한 후에 다음 명령을 입력하여 로컬에서 봇 실행을 시작합니다.

**C#**

```
dotnet run
```

**Python**

```
python app.py
```
    
봇이 시작되면 봇이 실행 중인 엔드포인트가 표시되는지 확인합니다. **http://localhost:3978**과 같은 엔드포인트가 표시되어야 합니다.

2. Bot Framework Emulator를 시작하고 다음과 같이 **/api/messages** 경로가 추가된 엔드포인트를 지정하여 봇을 엽니다.

    `http://localhost:3978/api/messages`

3. **실시간 채팅** 창에서 대화가 열리면 *Hello and welcome!* 메시지가 표시될 때까지 기다립니다.
4. *Hello*와 같은 메시지를 입력하고 봇의 응답을 확인합니다. 입력한 메시지가 그대로 다시 표시됩니다.
5. Bot Framework Emulator를 닫고 Visual Studio Code로 돌아온 후 터미널 창에서 **Ctrl+C**를 입력하여 봇을 중지합니다.

## 봇 코드 수정

사용자 입력을 그대로 다시 사용자에게 표시하는 봇을 만들었습니다. 이 봇은 별로 유용하지는 않지만 대화의 기본 흐름을 확인할 수는 있습니다. 봇과의 대화는 *활동*,시퀀스로 구성됩니다. 활동에서는 텍스트, 그래픽 또는 사용자 인터페이스 *카드*를 사용하여 정보를 교환합니다. 봇은 인사말로 대화를 시작합니다. 사용자가 봇과의 채팅 세션을 시작하면 트리거되는 *대화 업데이트* 활동에서 인사말이 생성됩니다. 그 이후의 대화에서는 일련의 추가 활동이 진행되며, 이러한 활동에서 사용자와 봇은 *메시지*를 주고받습니다.

1. Visual Studio Code에서 봇의 다음 코드 파일을 엽니다.
    - **C#**: TimeBot/Bots/EchoBot.cs
    - **Python**: TimeBot/bot.py

    이 파일의 코드는 *활동 처리기* 함수로 구성되어 있습니다. 함수 중 하나는 *구성원이 추가되었습니다.* 대화 업데이트 활동(채팅 세션에 다른 사람이 참가하는 경우)용이고 다른 하나는 *메시지* 활동(메시지가 수신되는 경우)용입니다. 대화는 *턴*이라는 개념을 기반으로 진행됩니다. 각 턴은 봇이 활동을 수신하여 처리하고 응답을 하는 상호 작용을 나타냅니다. *턴 컨텍스트*를 사용하여 현재 턴에서 처리 중인 활동 관련 정보를 추적합니다.

2. 코드 파일 맨 위에 다음 네임스페이스 가져오기 문을 추가합니다.

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. *메시지* 활동용 활동 처리기 함수를 다음 코드와 일치하도록 수정합니다.

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. 변경 내용을 저장하고 터미널 창에서 현재 디렉터리가 봇 코드 파일이 포함된 **TimeBot** 폴더인지 확인한 후에 다음 명령을 입력하여 로컬에서 봇 실행을 시작합니다.

**C#**

```
dotnet run
```

**Python**

```
python app.py
```

앞에서와 마찬가지로 봇이 시작되면 봇이 실행 중인 엔드포인트가 표시되는지 확인합니다.

5. Bot Framework Emulator를 시작하고 다음과 같이 **/api/messages** 경로가 추가된 엔드포인트를 지정하여 봇을 엽니다.

    `http://localhost:3978/api/messages`

6. **실시간 채팅** 창에서 대화가 열리면 *Hello and welcome!* 메시지가 표시될 때까지 기다립니다.
7. *Hello*와 같은 메시지를 입력하고 봇의 응답을 확인합니다. *Ask me what the time is*가 응답으로 표시됩니다.
8. *What is the time?*을 입력하고 응답을 확인합니다.

    이제 봇은 실행 중인 지역의 현지 시간을 표시하여 "What is the time?" 쿼리에 응답합니다. 다른 쿼리를 입력하는 경우에는 시간을 물어보라는 메시지가 사용자에게 표시됩니다. 현재 이 봇은 매우 제한적인 기능만 제공하며, Language Understanding 서비스와 추가 사용자 지정 코드를 통합하면 봇의 기능을 개선할 수 있습니다. 하지만 템플릿에서 만든 봇을 확장하는 방식을 통해 Bot Framework SDK를 사용하여 솔루션을 빌드하는 방법의 실제 예제를 이 봇에서 확인할 수 있습니다.

9. Bot Framework Emulator를 닫고 Visual Studio Code로 돌아온 후 터미널 창에서 **Ctrl+C**를 입력하여 봇을 중지합니다.

## 시간이 되면 다음 작업을 수행해 봅니다. Azure에 봇 배포

이제 Azure에 봇을 배포할 준비가 되었습니다. 봇을 배포할 때는 여러 단계를 수행하여 배포용 코드를 준비하고 필요한 Azure 리소스를 만듭니다.

### 리소스 그룹 생성 또는 선택

봇은 여러 Azure 리소스를 사용합니다. 리소스 그룹 하나에서 이러한 리소스를 만들 수 있습니다.

1. Azure Portal `https://portal.azure.com`을 열고 Azure 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
2. **리소스 그룹** 페이지를 표시하여 구독에 포함된 리소스 그룹을 확인합니다.
3. 사용 가능한 지역에서 고유한 이름으로 새 리소스 그룹을 만듭니다. ("샌드박스" 구독을 사용 중이어서 기존 리소스 그룹을 사용해야 한다면 해당 리소스 그룹 이름을 적어 둡니다.)

### Azure 애플리케이션 등록 만들기

봇이 사용자 및 웹 서비스와 통신하려면 애플리케이션 등록이 필요합니다.

1. **TimeBot** 폴더의 터미널 창에서 다음 명령을 입력하여 Azure CLI(명령줄 인터페이스)를 사용해 Azure에 로그인합니다. 브라우저가 열리면 Azure 구독에 로그인합니다.

```
az login
```

2. Azure 구독이 여러 개이면 다음 명령을 입력하여 봇을 배포할 구독을 선택합니다.

```
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
```

3. 다음 명령을 입력하여 암호를 **Super$ecretPassw0rd**로 지정해 **TimeBot**용 애플리케이션 등록을 만듭니다(원하는 경우 다른 표시 이름과 암호를 사용해도 되지만, 뒷부분에서 해당 정보가 필요하므로 적어 두어야 함).

```
az ad app create --display-name "TimeBot" --password "Super$ecretPassw0rd" --available-to-other-tenants
```

4. 명령 실행이 완료되면 대형 JSON 응답이 표시됩니다. 이 응답에서 **appId** 값을 찾아서 적어 둡니다. 다음 절차에서 필요합니다.

### Azure 리소스 만들기

Bot Framework SDK를 사용하여 템플릿에서 봇을 만들 때는 필수 Azure 리소스를 만드는 데 필요한 Azure Resource Manager 템플릿이 제공됩니다.

1. **TimeBot** 폴더의 터미널 창에서 다음 명령을 입력합니다(한 줄에 입력해야 함). PLACEHOLDER 값은 다음과 같이 바꿉니다.
    - **YOUR_RESOURCE_GROUP**: 기존 리소스 그룹의 이름
    - **YOUR_APP_ID**: 이전 절차에서 적어 둔 **appId** 값
    - **지역**: Azure 지역 코드(예: *eastus*)
    - **기타 모든 자리 표시자**: 새 리소스의 이름을 지정하는 데 사용할 고유한 값. 4~42자 사이의 전역적으로 고유한 문자열을 리소스 ID로 지정해야 합니다. **BotId** 및 **newWebAppName** 매개 변수에 사용하는 값은 뒷부분에서 필요하므로 적어 둡니다.

```
az deployment group create --resource-group "YOUR_RESOURCE_GROUP" --template-file "deploymenttemplates/template-with-preexisting-rg.json" --parameters appId="YOUR_APP_ID" appSecret="Super$ecretPassw0rd" botId="A_UNIQUE_BOT_ID" newWebAppName="A_UNIQUE_WEB_APP_NAME" newAppServicePlanName="A_UNIQUE_PLAN_NAME" appServicePlanLocation="REGION" --name "A_UNIQUE_SERVICE_NAME"
```

2. 명령 실행이 완료될 때까지 기다립니다. 명령이 정상적으로 실행되면 JSON 응답이 표시됩니다.

    오류가 발생하는 경우 명령에 오타가 있거나 고유하게 지정한 이름이 기존 리소스와 충돌하기 때문일 수 있습니다. 해당 문제를 수정한 후 다시 시도합니다. Azure Portal을 사용하여 오류가 발생하기 전에 만든 리소스를 삭제해야 할 수 있습니다.

3. 명령 실행이 완료되면 Azure Portal에서 리소스 그룹을 표시하여 생성된 리소스를 확인합니다.

### 배포용 봇 코드 준비

이제 필요한 Azure 리소스를 만들었으므로 리소스에 배포할 코드를 준비할 수 있습니다.

1. Visual Studio Code의 **TimeBot** 폴더 터미널 창에서 다음 명령을 입력하여 배포용 코드 종속성을 준비합니다.

**C#**

```
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "TimeBot.csproj"
```

**Python**

```
rmdir /S /Q  __pycache__
notepad requirements.txt
```

- 두 번째 명령을 실행하면 Python 환경용 requirements.txt 파일이 메모장에서 열립니다. 다음과 일치하도록 이 파일을 수정한 다음 변경 내용을 저장하고 메모장을 닫습니다.

```
botbuilder-core==4.11.0
aiohttp
```

### 배포용 zip 보관 파일 만들기

봇 파일을 배포하려면 .zip 보관 파일로 패키지해야 합니다. 이 보관 파일은 봇의 루트 폴더에 있는 파일과 폴더로 만들어야 합니다(루트 폴더 자체를 압축하면 <u>안 되며</u> 루트 폴더의 내용을 압축해야 함).

1. Visual Studio Code의 **탐색기** 창에서 **TimeBot** 폴더의 아무 파일 또는 폴더나 마우스 오른쪽 단추로 클릭하고 **파일 탐색기에 표시**를 선택합니다.
2. 파일 탐색기 창에서 **TimeBot** 폴더의 <u>모든</u> 파일을 선택합니다. 그런 다음 선택한 파일을 마우스 오른쪽 단추로 클릭하고 **보내기** > **압축(ZIP) 폴더**를 선택합니다.
3. **TimeBot** 폴더에 생성된 압축 파일의 이름을 **TimeBot.zip**으로 바꿉니다.

### 봇 배포 및 테스트

이제 코드가 준비되었으므로 코드를 배포할 수 있습니다.

1. Visual Studio Code의 **TimeBot** 폴더 터미널 창에서 다음 명령을 입력하여(한 줄에 입력해야 함) 패키지된 코드 파일을 배포합니다. PLACEHOLDER 값은 다음과 같이 바꿉니다.
    - **YOUR_RESOURCE_GROUP**: 기존 리소스 그룹의 이름
    - **YOUR_WEB_APP_NAME**: Azure 리소스를 만들 때 **newWebAppName** 매개 변수에 대해 지정했던 고유한 이름

```
az webapp deployment source config-zip --resource-group "YOUR_RESOURCE_GROUP" --name "YOUR_WEB_APP_NAME" --src "TimeBot.zip"
```

2. Azure Portal에서 리소스가 포함된 리소스 그룹 내의 **봇 채널 등록** 리소스(Azure 리소스를 만들 때 **BotId** 매개 변수에 할당한 이름이 지정되어 있음)를 엽니다.
3. **봇 관리** 섹션에서 **웹 채팅에서 테스트**를 선택합니다. 그런 다음 봇이 초기화될 때까지 기다립니다.
4. *Hello*와 같은 메시지를 입력하고 봇의 응답을 확인합니다. *Ask me what the time is*가 응답으로 표시됩니다.
5. *What is the time?*을 입력하고 응답을 확인합니다.

## 웹 페이지에서 웹 채팅 채널 사용

Azure Bot Service의 주요 이점 중 하나는 여러 *채널*을 통해 봇을 제공할 수 있다는 것입니다.

1. Azure Portal에서, 봇의 블레이드에서 봇이 연결된 **채널**을 봅니다.
2. **웹 채팅** 채널이 자동으로 추가되었으며 흔히 사용되는 통신 플랫폼용 기타 채널도 사용 가능합니다.
3. **웹 채팅** 채널 옆의 **편집**을 클릭합니다. 그러면 웹 페이지에서 봇에 포함해야 하는 설정이 포함된 페이지가 열립니다. 봇을 포함하려면 제공된 HTML embed 태그와 봇용으로 생성된 비밀 키 중 하나가 필요합니다.
4. **Embed 태그**를 복사합니다.
5. Visual Studio Code에서 **13-bot-framework/web-client** 폴더를 확장하고 이 폴더에 포함된 **default.html** 파일을 선택합니다.
6. HTML 코드에서 복사한 embed 태그를 **add the iframe for the bot here** 주석 바로 아래에 붙여넣습니다.
7. Azure Portal로 돌아와 비밀 키 중 하나(어떤 키든 관계없음)에서 **표시**를 선택하고 키를 복사합니다. 그런 다음 Visual Studio Code로 돌아와서 앞에서 추가한 HTML embed 태그에 키를 붙여넣어 **YOUR_SECRET_HERE**를 키로 바꿉니다.
8. Visual Studio Code의 **탐색기** 창에서 **default.html**을 마우스 오른쪽 단추로 클릭하고 **파일 탐색기에 표시**를 선택합니다.
9. 파일 탐색기 창에서 **default.html**을 Microsoft Edge에서 엽니다.
10. 그러면 열리는 웹 페이지에서 *Hello*를 입력하여 봇을 테스트합니다. 메시지를 제출할 때까지 봇은 초기화되지 않습니다. 따라서 인사말 메시지가 표시된 직후에 시간을 물어보라는 메시지가 표시됩니다.
11. *What is the time?*을 입력하여 봇을 테스트합니다.

## 추가 정보

Bot Framework에 대해 자세히 알아보려면 [Bot Framework 설명서](https://docs.microsoft.com/azure/bot-service/index-bf-sdk)를 참조하세요.
