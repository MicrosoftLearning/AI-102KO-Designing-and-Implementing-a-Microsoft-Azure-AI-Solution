---
lab:
    title: 'Azure Cognitive Search용 사용자 지정 기술 만들기'
    module: '모듈 12 - 지식 마이닝 솔루션 만들기'
---

# Azure Cognitive Search용 사용자 지정 기술 만들기

Azure Cognitive Search에서는 인식 기술 보강 파이프라인을 사용하여 문서에서 AI 생성 필드를 추출한 다음 검색 인덱스에 포함합니다. 포괄적인 기본 제공 기술 세트를 사용할 수 있습니다. 하지만 기본 제공 기술로는 특정 요구 사항을 충족할 수 없는 경우에는 사용자 지정 기술을 만들 수 있습니다.

이 연습에서는 사용자 지정 기술을 만듭니다. 이 기술은 문서의 개별 단어 빈도를 표로 만들어 가장 많이 사용되는 상위 5개 단어 목록을 생성한 다음 가상의 여행사인 Margie's Travel용 검색 솔루션에 추가합니다.

## 이 과정용 리포지토리 복제

이 랩에서 작업을 수행 중인 환경에 **AI-102-AIEngineer** 코드 리포지토리를 이미 복제했다면 Visual Studio Code에서 해당 리포지토리를 열고, 그렇지 않으면 다음 단계에 따라 리포지토리를 지금 복제합니다.

1. Visual Studio Code를 시작합니다.
2. 팔레트를 열고(Shift+Ctrl+P 누르기) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/AI-102KO-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버그에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## Azure 리소스 만들기

> **참고**: **Azure Cognitive Search 솔루션 만들기(22-azure-search.md)** 연습을 이전에 완료했으며 구독에 해당 연습의 Azure 리소스가 아직 남아 있으면 이 섹션을 건너뛰고 **검색 솔루션 만들기** 섹션부터 시작하면 됩니다. 그렇지 않은 경우에는 아래 단계에 따라 필요한 Azure 리소스를 프로비전합니다.

1. 웹 브라우저에서 Azure Portal `https://portal.azure.com`을 열고 Azure 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
2. 구독의 **리소스 그룹**을 확인합니다.
3. 제한된 구독을 사용 중이어서 리소스 그룹이 제공되어 있다면 해당 리소스 그룹을 선택하여 속성을 확인합니다. 그렇지 않은 경우에는 원하는 이름으로 새 리소스 그룹을 만들고 그룹이 만들어지면 해당 그룹으로 이동합니다.
4. 리소스 그룹의 **개요** 페이지에서 **구독 ID** 및 **위치**를 확인합니다. 후속 단계에서 이러한 값과 리소스 그룹의 이름이 필요합니다.
5. Visual Studio Code에서 **23-custom-search-skill** 폴더를 확장하고 **setup.cmd**를 선택합니다. 이 배치 스크립트를 사용하여 필요한 Azure 리소스를 만드는 데 필요한 Azure CLI(명령줄 인터페이스)를 실행합니다.
6. **23-custom-search-skill** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.
7. 터미널 창에 다음 명령을 입력하여 Azure 구독에 대한 인증된 연결을 설정합니다.

    ```
    az login --output none
    ```

8. 메시지가 표시되면 Azure 구독에 로그인합니다. 그런 다음 Visual Studio Code로 돌아와서 로그인 프로세스가 완료될 때까지 기다립니다.
9. 다음 명령을 실행하여 Azure 위치를 나열합니다.

    ```
    az account list-locations -o table
    ```

10. 출력에서 리소스 그룹 위치에 해당하는 **Name** 값을 찾습니다(예를 들어 *미국 동부*에 해당하는 이름은 *eastus*임).
11. **setup.cmd** 스크립트에서 구독 ID, 리소스 그룹 이름 및 위치 이름에 해당하는 값을 사용하여 **subscription_id**, **resource_group** 및 **location** 변수 선언을 수정합니다. 그런 다음 변경 내용을 저장합니다.
12. **23-custom-search-skill** 폴더의 터미널에서 다음 명령을 입력하여 스크립트를 실행합니다.

    ```
    setup
    ```

    > **참고**: Search CLI 모듈은 미리 보기 상태이므로 *- Running ..* 프로세스에서 실행이 중단될 수 있습니다. 실행이 2분 넘게 중단되면 Ctrl+C를 눌러 장기 실행 작업을 취소한 다음 스크립트를 종료할 것인지를 묻는 메시지가 표시되면 **N**을 선택합니다. 그러면 실행이 정상적으로 완료됩니다.
    >
    > 스크립트 실행이 실패하면 올바른 변수 이름을 적용하여 스크립트를 저장했는지 확인하고 다시 시도합니다.

13. 스크립트 실행이 완료되면 표시되는 출력을 검토하여 Azure 구독과 관련된 다음 정보를 확인합니다(뒷부분에서 이러한 값이 필요함).
    - 스토리지 계정 이름
    - 스토리지 연결 문자열
    - Cognitive Services 계정
    - Cognitive Services 키
    - Search 서비스 엔드포인트
    - Search 서비스 관리 키
    - Search 서비스 쿼리 키

14. Azure Portal에서 리소스 그룹을 새로 고쳐 Azure Storage 계정, Azure Cognitive Services 리소스 및 Azure Cognitive Search 리소스가 포함되어 있는지 확인합니다.

## 검색 솔루션 만들기

이제 필요한 Azure 리소스가 준비되었으므로 다음 구성 요소로 이루어진 검색 솔루션을 만들 수 있습니다.

- Azure Storage 컨테이너의 문서를 참조하는 **데이터 원본**
- 문서에서 AI 생성 필드를 추출하는 기술 보강 파이프라인을 정의하는 **기술 세트**
- 문서 레코드의 검색 가능 세트를 정의하는 **인덱스**
- 데이터 원본에서 문서를 추출하여 기술 세트를 적용한 다음 인덱스를 채우는 **인덱서**

이 연습에서는 Azure Cognitive Search REST 인터페이스를 사용하여 JSON 요청을 제출해 이러한 구성 요소를 만듭니다.

1. Visual Studio Code의 **23-custom-search-skill** 폴더에서 **create-search** 폴더를 확장하고 **data_source.json**을 선택합니다. 이 파일에는 **margies-custom-data** 데이터 원본의 JSON 정의가 포함되어 있습니다.
2. **YOUR_CONNECTION_STRING** 자리 표시자는 Azure Storage 계정의 연결 문자열로 바꿉니다. 이 연결 문자열은 다음과 같습니다.

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Azure Portal의 스토리지 계정 **액세스 키** 페이지에서 연결 문자열을 확인할 수 있습니다.*

3. 업데이트된 JSON 파일을 저장하고 닫습니다.
4. **create-search** 폴더에서 **skillset.json**을 엽니다. 이 파일에는 **margies-custom-skillset** 기술 세트의 JSON 정의가 포함되어 있습니다.
5. 기술 세트 정의 맨 위에 있는 **cognitiveServices** 요소에서 **YOUR_COGNITIVE_SERVICES_KEY** 자리 표시자를 Cognitive Services 리소스의 키 중 하나로 바꿉니다.

    *Azure Portal의 Cognitive Services 리소스 **키 및 엔드포인트** 페이지에서 키를 확인할 수 있습니다.*

6. 업데이트된 JSON 파일을 저장하고 닫습니다.
7. **create-search** 폴더에서 **index.json**을 엽니다. 이 파일에는 **margies-custom-index** 인덱스의 JSON 정의가 포함되어 있습니다.
8. 인덱스의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
9. **create-search** 폴더에서 **indexer.json**을 엽니다. 이 파일에는 **margies-custom-indexer** 인덱서의 JSON 정의가 포함되어 있습니다.
10. 인덱서의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
11. **create-search** 폴더에서 **create-search.cmd**를 엽니다. 이 배치 스크립트는 cURL 유틸리티를 사용해 Azure Cognitive Search 리소스의 JSON 정의를 REST 인터페이스에 제출합니다.
12. **YOUR_SEARCH_URL** 및 **YOUR_ADMIN_KEY** 변수 자리 표시자를 각각 **Url**, 그리고 Azure Cognitive Search 리소스의 **관리 키** 중 하나로 바꿉니다.

    *Azure Portal의 Azure Cognitive Search 리소스 **개요** 및 **키** 페이지에서 이러한 값을 확인할 수 있습니다.*

13. 업데이트된 배치 파일을 저장합니다.
14. **create-search** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.
15. **create-search** 폴더의 터미널 창에서 다음 명령을 입력하여 배치 스크립트를 실행합니다.

    ```
    create-search
    ```

16. 스크립트 실행이 완료되면 Azure Portal의 Azure Cognitive Search 리소스 페이지에서 **인덱서** 페이지를 선택하고 인덱싱 프로세스가 완료될 때까지 기다립니다.

    ***새로 고침**을 선택하여 인덱싱 작업 진행률을 추적할 수 있습니다. 인덱싱이 완료되려면 1분 정도 걸릴 수 있습니다.*

## 인덱스 검색

이제 인덱스가 생성되었으므로 해당 인덱스를 검색할 수 있습니다.

1. Azure Cognitive Search 리소스 블레이드 위쪽에서 **검색 탐색기**를 선택합니다.
2. 검색 탐색기의 **쿼리 문자열** 상자에 다음 쿼리 문자열을 입력한 다음 **검색**을 선택합니다.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment gt 0.5
    ```

    이 쿼리는 *London*이 언급되어 있고, 작성자가 *Reviewer*이며, **sentiment** 점수가 *0.5*점보다 높은 모든 문서(즉, 런던을 언급한 긍정적인 내용의 리뷰)의 **url**, **sentiment**, **keyphrases**를 검색합니다.

## 사용자 지정 기술용 Azure 함수 만들기

검색 솔루션에는 여러 기본 제공 인식 기술이 포함되어 있습니다. 이러한 기술은 감정 점수, 이전 작업에서 확인했던 핵심 구 목록과 같은 문서의 정보를 사용하여 인덱스를 보강합니다.

사용자 지정 기술을 만들면 인덱스를 추가로 보강할 수 있습니다. 예를 들어 각 문서에서 사용 빈도가 가장 높은 단어를 식별하려고 하는데 이 기능을 제공하는 기본 제공 기술은 없는 경우 사용자 지정 기술을 만들면 유용할 수 있습니다.

여기서는 단어 개수 기능을 사용자 지정 기술로 구현하기 위해 원하는 언어로 Azure 함수를 만듭니다.

1. Visual Studio Online에서 Azure 확장 탭(**&boxplus;**)을 표시하여 **Azure Functions** 확장이 설치되어 있는지 확인합니다. 이 확장을 사용하면 Visual Studio Code에서 Azure Functions를 만들어 배포할 수 있습니다.
2. Azure 탭(**&Delta;**)의 **Azure Functions** 창에서 원하는 언어에 따라 다음 설정을 사용하여 새 프로젝트(&#128194;)를 만듭니다.

    ### **C#**

    - **폴더**: **23-custom-search-skill/C-Sharp/wordcount**로 이동
    - **언어**: C#
    - **템플릿**: HTTP 트리거
    - **함수 이름**: wordcount
    - **네임스페이스**: margies.search
    - **권한 부여 수준**: 기능

    ### **Python**

    - **폴더**: **23-custom-search-skill/Python/wordcount**로 이동
    - **언어**: Python
    - **가상 환경**: 가상 환경 건너뛰기
    - **템플릿**: HTTP 트리거
    - **함수 이름**: wordcount
    - **권한 부여 수준**: 기능

    ***launch.json**을 덮어쓴다는 메시지가 표시되면 해당 파일을 덮어씁니다.*

3. **탐색기**(**&#128461;**) 탭으로 다시 전환하여 **wordcount** 폴더에 이제 Azure 함수용 코드 파일이 포함되어 있음을 확인합니다.

    *Python을 선택한 경우 코드 파일은 역시 이름이 **wordcount***인 하위 폴더에 있을 수도 있습니다.

4. 함수의 기본 코드 파일은 자동으로 열렸을 것입니다. 파일이 자동으로 열리지 않았으면 선택한 언어에 해당하는 파일을 엽니다.
    - **C#**: wordcount.cs
    - **Python**: \_\_init\_\_&#46;py

5. 파일의 전체 내용을 선택한 언어에 해당하는 다음 코드로 바꿉니다.

### **C#**

```C#
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System.Collections.Generic;
Microsoft.Extensions.Logging 사용;
using System.Text.RegularExpressions;
using System.Linq;

namespace margies.search
{
    public static class wordcount
    {

        //응답용 클래스 정의
        private class WebApiResponseError
        {
            public string message { get; set; }
        }

        private class WebApiResponseWarning
        {
            public string message { get; set; }
        }

        private class WebApiResponseRecord
        {
            public string recordId { get; set; }
            public Dictionary<string, object> data { get; set; }
            public List<WebApiResponseError> errors { get; set; }
            public List<WebApiResponseWarning> warnings { get; set; }
        }

        private class WebApiEnricherResponse
        {
            public List<WebApiResponseRecord> values { get; set; }
        }

        //사용자 지정 기술용 함수
        [FunctionName("wordcount")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)]HttpRequest req, ILogger log)
        {
            log.LogInformation("Function initiated.");

            string recordId = null;
            string originalText = null;

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            // 유효성 검사
            if (data?.values == null)
            {
                return new BadRequestObjectResult(" Could not find values array");
            }
            if (data?.values.HasValues == false || data?.values.First.HasValues == false)
            {
                return new BadRequestObjectResult("Could not find valid records in values array");
            }

            WebApiEnricherResponse response = new WebApiEnricherResponse();
            response.values = new List<WebApiResponseRecord>();
            foreach (var record in data?.values)
            {
                recordId = record.recordId?.Value as string;
                originalText = record.data?.text?.Value as string;

                if (recordId == null)
                {
                    return new BadRequestObjectResult("recordId cannot be null");
                }

                // 응답 수집
                WebApiResponseRecord responseRecord = new WebApiResponseRecord();
                responseRecord.data = new Dictionary<string, object>();
                responseRecord.recordId = recordId;
                responseRecord.data.Add("text", Count(originalText));

                response.values.Add(responseRecord);
            }

            return (ActionResult)new OkObjectResult(response); 
        }


            public static string RemoveHtmlTags(string html)
        {
            string htmlRemoved = Regex.Replace(html, @"<script[^>]*>[\s\S]*?</script>|<[^>]+>| ", " ").Trim();
            string normalised = Regex.Replace(htmlRemoved, @"\s{2,}", " ");
            return normalised;
        }

        public static List<string> Count(string text)
        {
            
            //html 요소 제거
            text=text.ToLowerInvariant();
            string html = RemoveHtmlTags(text);
            
            //단어 목록으로 분할
            List<string> list = html.Split(" ").ToList();
            
            //영문자가 아닌 문자 제거
            var onlyAlphabetRegEx = new Regex(@"^[A-z]+$");
            list = list.Where(f => onlyAlphabetRegEx.IsMatch(f)).ToList();

            //중지 단어 제거
            string[] stopwords = { "", "i", "me", "my", "myself", "we", "our", "ours", "ourselves", "you", 
                    "you're", "you've", "you'll", "you'd", "your", "yours", "yourself", 
                    "yourselves", "he", "him", "his", "himself", "she", "she's", "her", 
                    "hers", "herself", "it", "it's", "its", "itself", "they", "them", 
                    "their", "theirs", "themselves", "what", "which", "who", "whom", 
                    "this", "that", "that'll", "these", "those", "am", "is", "are", "was",
                    "were", "be", "been", "being", "have", "has", "had", "having", "do", 
                    "does", "did", "doing", "a", "an", "the", "and", "but", "if", "or", 
                    "because", "as", "until", "while", "of", "at", "by", "for", "with", 
                    "about", "against", "between", "into", "through", "during", "before", 
                    "after", "above", "below", "to", "from", "up", "down", "in", "out", 
                    "on", "off", "over", "under", "again", "further", "then", "once", "here", 
                    "there", "when", "where", "why", "how", "all", "any", "both", "each", 
                    "few", "more", "most", "other", "some", "such", "no", "nor", "not", 
                    "only", "own", "same", "so", "than", "too", "very", "s", "t", "can", 
                    "will", "just", "don", "don't", "should", "should've", "now", "d", "ll",
                    "m", "o", "re", "ve", "y", "ain", "aren", "aren't", "couldn", "couldn't", 
                    "didn", "didn't", "doesn", "doesn't", "hadn", "hadn't", "hasn", "hasn't", 
                    "haven", "haven't", "isn", "isn't", "ma", "mightn", "mightn't", "mustn", 
                    "mustn't", "needn", "needn't", "shan", "shan't", "shouldn", "shouldn't", "wasn", 
                    "wasn't", "weren", "weren't", "won", "won't", "wouldn", "wouldn't"}; 
            list = list.Where(x => x.Length > 2).Where(x => !stopwords.Contains(x)).ToList();
            
            //키와 개수별 고유 단어를 가져와 개수를 기준으로 정렬
            var keywords = list.GroupBy(x => x).OrderByDescending(x => x.Count());
            var klist = keywords.ToList();

            // 상위 10개 단어 반환
            var numofWords = 10;
            if(klist.Count<10)
                numofWords=klist.Count;
            List<string> resList = new List<string>();
            for (int i = 0; i < numofWords; i++)
            {
                resList.Add(klist[i].Key);
            }
            return resList;
        }
    }
}
```

## **Python**

```Python
import logging
import os
import sys
import json
from string import punctuation
from collections import Counter
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Wordcount function initiated.')

    # "values" 모음이 결과로 반환됨
    result = {
        "values": []
    }
    statuscode = 200

    # 단어 개수에서 다음 목록의 단어는 제외됨
    stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
                "you're", "you've", "you'll", "you'd", 'your', 'yours', 'yourself', 
                'yourselves', 'he', 'him', 'his', 'himself', 'she', "she's", 'her', 
                'hers', 'herself', 'it', "it's", 'its', 'itself', 'they', 'them', 
                'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
                'this', 'that', "that'll", 'these', 'those', 'am', 'is', 'are', 'was',
                'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
                'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
                'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
                'about', 'against', 'between', 'into', 'through', 'during', 'before', 
                'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
                'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
                'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
                'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
                'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 
                'will', 'just', 'don', "don't", 'should', "should've", 'now', 'd', 'll',
                'm', 'o', 're', 've', 'y', 'ain', 'aren', "aren't", 'couldn', "couldn't", 
                'didn', "didn't", 'doesn', "doesn't", 'hadn', "hadn't", 'hasn', "hasn't", 
                'haven', "haven't", 'isn', "isn't", 'ma', 'mightn', "mightn't", 'mustn', 
                "mustn't", 'needn', "needn't", 'shan', "shan't", 'shouldn', "shouldn't", 'wasn', 
                "wasn't", 'weren', "weren't", 'won', "won't", 'wouldn', "wouldn't"]

    try:
        values = req.get_json().get('values')
        logging.info(values)

        for rec in values:
            # 이 레코드의 기본 JSON 응답 생성
            val = {
                    "recordId": rec['recordId'],
                    "data": {
                        "text":None
                    },
                    "errors": None,
                    "warnings": 없음
                }
            try:
                # 입력 레코드에서 처리할 텍스트 가져오기
                txt = rec['data']['text']
                // 숫자 제거
                txt = ''.join(c for c in txt if not c.isdigit())
                # 문장 부호를 제거하고 문자를 소문자로 변환
                txt = ''.join(c for c in txt if c not in punctuation).lower()
                # 중지 단어 제거
                txt = ' '.join(w for w in txt.split() if w not in stopwords)
                # 단어 개수를 계산하여 가장 많이 나오는 10개 단어 가져오기
                wordcount = Counter(txt.split()).most_common(10)
                words = [w[0] for w in wordcount]
                # 이 텍스트 레코드의 출력에 상위 10개 단어 추가
                val["data"]["text"] = words
            except:
                # 이 텍스트 레코드에 오류가 발생한 경우. 이 경우에는 오류 및 경고 목록 추가
                val["errors"] =[{"message": "An error occurred processing the text."}]
                val["warnings"] = [{"message": "One or more inputs failed to process."}]
            finally:
                # 응답에 이 레코드의 값 추가
                result["values"].append(val)
    except Exception as ex:
        statuscode = 500
        # 전역 오류가 발생한 경우. 이 경우에는 오류 응답 반환
        val = {
                "recordId": None,
                "data": {
                    "text":None
                },
                "errors": [{"message": ex.args}],
                "warnings": [{"message": "The request failed to process."}]
            }
        result["values"].append(val)
    finally:
        # 응답 반환
        return func.HttpResponse(body=json.dumps(result), mimetype="application/json", status_code=statuscode)
```
    
6. 업데이트된 파일을 저장합니다.
7. 코드 파일이 포함된 **wordcount** 폴더를 마우스 오른쪽 단추로 클릭하고 **함수 앱에 배포**를 선택합니다. 그런 후 다음 언어별 설정을 사용하여 함수를 배포합니다(메시지가 표시되면 Azure에 로그인).

    ### **C#**

    - **구독**(메시지가 표시되는 경우): Azure 구독을 선택합니다.
    - **함수**: Azure에서 새 함수 앱 만들기(고급)
    - **함수 앱 이름**: 전역적으로 고유한 이름을 입력합니다.
    - **런타임**: .NET Core 3.1
    - **OS**: Linux
    - **호스팅 계획**: Consumption
    - **리소스 그룹**: Azure Cognitive Search 리소스가 포함된 리소스 그룹
        - 참고: 이 리소스 그룹에 Windows 기반 웹앱이 이미 포함되어 있으면 해당 그룹에 Linux 기반 함수를 배포할 수 없습니다. 기존 웹앱을 삭제하거나 다른 리소스 그룹에 함수를 배포하세요.
    - **스토리지 계정**: Margie's Travel 문서가 저장된 스토리지 계정
    - **Application Insights**: 지금은 건너뛰기

    *Visual Studio Code에서는 함수 프로젝트를 만들 때 저장한 **vscode** 폴더의 구성을 기준으로 하여 함수의 컴파일된 버전을 배포(**bin** 하위 폴더에)합니다.*

    ### **Python**

    - **구독**(메시지가 표시되는 경우): Azure 구독을 선택합니다.
    - **함수**: Azure에서 새 함수 앱 만들기(고급)
    - **함수 앱 이름**: 전역적으로 고유한 이름을 입력합니다.
    - **런타임**: Python 3.8
    - **호스팅 계획**: Consumption
    - **리소스 그룹**: Azure Cognitive Search 리소스가 포함된 리소스 그룹
        - 참고: 이 리소스 그룹에 Windows 기반 웹앱이 이미 포함되어 있으면 해당 그룹에 Linux 기반 함수를 배포할 수 없습니다. 기존 웹앱을 삭제하거나 다른 리소스 그룹에 함수를 배포하세요.
    - **스토리지 계정**: Margie's Travel 문서가 저장된 스토리지 계정
    - **Application Insights**: 지금은 건너뛰기

8. Visual Studio Code에서 함수를 배포하는 동안 기다립니다. 배포가 완료되면 알림이 표시됩니다.

## 함수 테스트

이제 Azure에 함수를 배포했으므로 Azure Portal에서 함수를 테스트할 수 있습니다.

1. [Azure Portal](https://portal.azure.com)을 열고 함수 앱을 만든 리소스 그룹으로 이동합니다. 그런 다음 함수 앱용 앱 서비스를 엽니다.
2. 앱 서비스 블레이드의 **함수** 페이지에서 **wordcount** 함수를 엽니다.
3. **wordcount** 함수 블레이드에서 **코드 + 테스트** 페이지를 표시한 다음 **테스트/실행** 창을 엽니다.
4. **테스트/실행** 창에서 기존 **본문**을 다음 JSON으로 바꿉니다. 이 JSON에는 Azure Cognitive Search 기술에 필요한 스키마가 반영되어 있습니다. 이 기술은 문서 하나 이상의 데이터가 포함된 레코드를 처리용으로 제출합니다.

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
5. **실행**을 클릭하고 함수가 반환되는 HTTP 응답 콘텐츠를 확인합니다. 이 응답에는 기술 사용 시 Azure Cognitive Search에 적용되어야 하는 스키마가 반영되어 있습니다. 즉, 각 문서에 대한 응답이 반환됩니다. 여기서 응답은 각 문서에 포함된 용어 최대 10개로 구성되어 있습니다. 이러한 용어는 문서에 나오는 빈도의 내림차순으로 정렬됩니다.

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

6. **테스트/실행** 창을 닫고 **wordcount** 함수 블레이드에서 **함수 URL 가져오기**를 클릭합니다. 그런 다음 기본 키의 URL을 클립보드에 복사합니다. 다음 절차에서 이 URL이 필요합니다.

## 검색 솔루션에 사용자 지정 기술 추가

이제검색 솔루션 기술 세트에 함수를 사용자 지정 기술로 포함하고 이 함수가 생성하는 결과를 인덱스의 필드에 매핑해야 합니다. 

1. Visual Studio Code의 **23-custom-search-skill/update-search** 폴더에서 **update-skillset.json** 파일을 엽니다. 이 파일에는 기술 세트의 JSON 정의가 포함되어 있습니다.
2. 기술 세트 정의를 검토합니다. 이 정의에는 이전과 같은 기술, 그리고 새 **WebApiSkill** 기술인 **get-top-words**가 포함되어 있습니다.
3. **get-top-words** 기술 정의를 편집하여 **uri** 값을 Azure 함수의 URL(이전 절차에서 클립보드에 복사한 URL)로 설정합니다. **YOUR-FUNCTION-APP-URL**을 해당 URL로 바꾸면 됩니다.
4. 기술 세트 정의 맨 위에 있는 **cognitiveServices** 요소에서 **YOUR_COGNITIVE_SERVICES_KEY** 자리 표시자를 Cognitive Services 리소스의 키 중 하나로 바꿉니다.

    *Azure Portal의 Cognitive Services 리소스 **키 및 엔드포인트** 페이지에서 키를 확인할 수 있습니다.*

5. 업데이트된 JSON 파일을 저장하고 닫습니다.
6. **update-search** 폴더에서 **update-index.json**을 엽니다. 이 파일에는 **margies-custom-index** 인덱스의 JSON 정의가 포함되어 있습니다. 인덱스 정의 맨 아래에는 추가 필드 **top_words**가 있습니다.
7. 인덱스의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
8. **update-search** 폴더에서 **update-indexer.json**을 엽니다. 이 파일에는 **margies-custom-indexer**의 JSON 정의 및 **top_words** 필드에 대한 추가 매핑이 포함되어 있습니다.
9. 인덱서의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
10. **update-search** 폴더에서 **update-search.cmd**를 엽니다. 이 배치 스크립트는 cURL 유틸리티를 사용해 Azure Cognitive Search 리소스의 업데이트된 JSON 정의를 REST 인터페이스에 제출합니다.
11. **YOUR_SEARCH_URL** 및 **YOUR_ADMIN_KEY** 변수 자리 표시자를 각각 **Url**, 그리고 Azure Cognitive Search 리소스의 **관리 키** 중 하나로 바꿉니다.

    *Azure Portal의 Azure Cognitive Search 리소스 **개요** 및 **키** 페이지에서 이러한 값을 확인할 수 있습니다.*

12. 업데이트된 배치 파일을 저장합니다.
13. **update-search** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.
14. **update-search** 폴더의 터미널 창에서 다음 명령을 입력하여 배치 스크립트를 실행합니다.

    ```
    update-search
    ```

15. 스크립트 실행이 완료되면 Azure Portal의 Azure Cognitive Search 리소스 페이지에서 **인덱서** 페이지를 선택하고 인덱싱 프로세스가 완료될 때까지 기다립니다.

    ***새로 고침**을 선택하여 인덱싱 작업 진행률을 추적할 수 있습니다. 인덱싱이 완료되려면 1분 정도 걸릴 수 있습니다.*

## 인덱스 검색

이제 인덱스가 생성되었으므로 해당 인덱스를 검색할 수 있습니다.

1. Azure Cognitive Search 리소스 블레이드 위쪽에서 **검색 탐색기**를 선택합니다.
2. 검색 탐색기의 **쿼리 문자열** 상자에 다음 쿼리 문자열을 입력한 다음 **검색**을 선택합니다.

    ```
    search=Las Vegas&$select=url,top_words
    ```

    이 쿼리는 *Las Vegas*를 언급하는 모든 문서의 **url** 및 **top_words** 필드를 검색합니다.

## 자세한 정보

Azure Cognitive Search용 사용자 지정 기술을 만드는 방법에 대한 자세한 내용은 [Azure Cognitive Search 설명서](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)를 참조하세요.
