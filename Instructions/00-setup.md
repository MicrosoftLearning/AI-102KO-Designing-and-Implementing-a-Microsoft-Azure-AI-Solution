---
lab:
    title: '랩 환경 설정'
    module: '설정'
---

# 랩 환경 설정

이 과정의 연습은 호스트된 랩 환경에서 완료해야 합니다. 개인 컴퓨터에서 연습을 완료하려는 경우 다음 소프트웨어를 설치하면 됩니다. 개인 환경 사용 시에는 예기치 않은 대화 상자가 표시되거나 동작이 수행될 수 있습니다. 다양한 로컬 구성이 가능하기 때문에 과정 팀은 사용자 환경에서 발생할 수 있는 문제를 지원할 수 없습니다.

> **참고**: 아래 지침은 Windows 10용입니다. Linux 또는 MacOS를 사용할 수도 있습니다. 선택한 OS에 맞게 랩 지침을 조정해야 할 수도 있습니다.

### 기본 운영 체제(Windows 10)

#### Windows 10

Windows 10을 설치하고 모든 업데이트를 적용합니다.

#### Edge

[Edge(Chromium)](https://microsoft.com/edge)을 설치합니다.

### .NET Core SDK

1. https://dotnet.microsoft.com/download에서 다운로드하여 설치합니다(런타임만 다운로드하지 말고 .NET Core SDK를 다운로드해야 함).

### C++ 재배포 가능 패키지

1. https://aka.ms/vs/16/release/vc_redist.x64.exe에서 Visual C++ 재배포 가능 패키지(x64)를 다운로드하여 설치합니다.

### Node.JS

1. https://nodejs.org/en/download/에서 최신 LTS 버전을 다운로드합니다. 
2. 기본 옵션을 사용하여 설치합니다.

### Python(및 필수 패키지)

1. https://docs.conda.io/en/latest/miniconda.html에서 버전 3.8을 다운로드합니다. 
2. 설치 프로그램을 실행하여 설치합니다. **중요**: PATH 변수에 Miniconda를 추가하고 Miniconda를 기본 Python 환경으로 등록하는 옵션을 선택합니다.
3. 설치 후 Anaconda 프롬프트를 열고 다음 명령을 입력하여 패키지를 설치합니다. 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### Azure CLI

1. https://docs.microsoft.com/ko-kr/cli/azure/install-azure-cli?view=azure-cli-latest에서 다운로드합니다. 
2. 기본 옵션을 사용하여 설치합니다.

### Git

1. https://git-scm.com/download.html에서 다운로드한 다음 기본 옵션을 사용하여 설치합니다.


### Visual Studio Code(및 확장)

1. https://code.visualstudio.com/Download에서 다운로드합니다. 
2. 기본 옵션을 사용하여 설치합니다. 
3. 설치 후 Visual Studio Code를 시작하고 **확장** 탭(CTRL+SHIFT+X) 를 누르면 표시됨)에서 다음 Microsoft 확장을 검색하여 설치합니다.
    - Python
    - C#
    - Azure Functions
    - PowerShell


### Bot Framework Emulator

https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md의 지침에 따라 사용 중인 운영 체제용 Bot Framework Emulator의 안정적인 최신 버전을 다운로드하여 설치합니다.

### Bot Framework Composer

https://docs.microsoft.com/ko-kr/composer/install-composer에서 설치합니다.
