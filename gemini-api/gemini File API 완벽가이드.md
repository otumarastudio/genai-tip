# Gemini File API 완벽 가이드 (한글)

> Gemini API에 이미지, 오디오, 비디오, 문서 등 **미디어 파일을 전달하는 4가지 방법**을 상세히 설명합니다.

---

## 목차

1. [한눈에 보기](#한눈에-보기)
2. [방법 1: Inline Data (직접 전달)](#방법-1-inline-data-직접-전달)
3. [방법 2: File API (업로드)](#방법-2-file-api-업로드)
4. [방법 3: GCS 등록](#방법-3-gcs-등록-google-cloud-storage)
5. [방법 4: External URL](#방법-4-external-url-외부-url)
6. [파일 관리 API](#파일-관리-api)
7. [지원 파일 형식](#지원-파일-형식)
8. [멀티모달 프롬프트 전략](#멀티모달-프롬프트-전략)
9. [실전 활용 예시](#실전-활용-예시)
10. [문제 해결 가이드](#문제-해결-가이드)
11. [제한사항 및 주의점](#제한사항-및-주의점)

---

## 한눈에 보기

### 어떤 방법을 선택해야 할까?

```
┌─────────────────────────────────────────────────────────────────┐
│                    파일을 Gemini에 전달하려면?                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  파일 크기는?     │
                    └─────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
    100MB 이하            100MB~2GB          이미 클라우드에 있음
          │                   │                   │
          ▼                   ▼                   ▼
  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
  │ Inline Data   │   │  File API     │   │ GCS 등록 또는  │
  │ 또는          │   │  Upload       │   │ External URL  │
  │ External URL  │   │               │   │               │
  └───────────────┘   └───────────────┘   └───────────────┘
```

### 입력 방식 비교표

| 방법 | 최대 크기 | 저장 기간 | 언제 사용? | 비용 |
|-----|---------|----------|----------|-----|
| **Inline Data** | 100MB (PDF: 50MB) | 없음 (매번 전송) | 작은 파일, 빠른 테스트, 실시간 처리 | 무료 |
| **File API Upload** | 2GB/파일, 20GB/프로젝트 | 48시간 | 대용량 파일, 여러 번 재사용 | 무료 |
| **GCS 등록** | 2GB/파일, 무제한 | 30일 (등록 유효) | GCS에 이미 있는 파일 | 무료 |
| **External URL** | 100MB | 없음 (매번 가져옴) | 공개 URL, S3/Azure 사전 서명 URL | 무료 |

> **TIP**: 모든 File API 기능은 **무료**입니다. Gemini API가 지원되는 모든 지역에서 사용 가능합니다.

### 핵심 API 요약

```python
from google import genai

client = genai.Client()

# 파일 관리 API
client.files.upload(file="경로")              # 업로드
client.files.list()                           # 목록 조회
client.files.get(name="files/xxx")            # 메타데이터 조회
client.files.delete(name="files/xxx")         # 삭제
client.files.register_files(uris=["gs://..."]) # GCS 등록
```

---

## 방법 1: Inline Data (직접 전달)

**가장 간단한 방법**입니다. 파일을 Base64로 인코딩하여 요청에 직접 포함합니다.

### 언제 사용하나요?

- 파일 크기가 **100MB 이하** (PDF는 50MB 이하)
- 빠른 테스트나 프로토타이핑
- 일회성 요청 (같은 파일을 여러 번 사용하지 않을 때)

### Python 예제

```python
from google import genai
from google.genai import types
import pathlib

client = genai.Client()

# 로컬 파일 읽기
filepath = pathlib.Path('my_document.pdf')

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        types.Part.from_bytes(
            data=filepath.read_bytes(),
            mime_type='application/pdf',
        ),
        "이 문서의 핵심 내용을 3줄로 요약해줘"
    ]
)
print(response.text)
```

### JavaScript 예제

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from 'node:fs';
import * as path from 'node:path';

const ai = new GoogleGenAI({});

async function main() {
    const filePath = path.join('content', 'my_document.pdf');

    const contents = [
        { text: "이 문서의 핵심 내용을 3줄로 요약해줘" },
        {
            inlineData: {
                mimeType: 'application/pdf',
                data: fs.readFileSync(filePath).toString("base64")
            }
        }
    ];

    const response = await ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: contents
    });
    console.log(response.text);
}

main();
```

### URL에서 파일 가져와서 전달하기

이미 웹에 있는 파일을 다운로드해서 Inline으로 전달할 수도 있습니다.

```python
from google import genai
from google.genai import types
import httpx

client = genai.Client()

# URL에서 PDF 다운로드
doc_url = "https://example.com/report.pdf"
doc_data = httpx.get(doc_url).content

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        types.Part.from_bytes(
            data=doc_data,
            mime_type='application/pdf',
        ),
        "이 보고서의 주요 결론은?"
    ]
)
print(response.text)
```

> **TIP**: Inline Data는 매 요청마다 파일을 전송하므로, 같은 파일을 여러 번 사용한다면 File API Upload가 더 효율적입니다.

---

## 방법 2: File API (업로드)

**대용량 파일**이나 **여러 번 재사용**할 파일에 적합합니다.

### 언제 사용하나요?

- 파일 크기가 **100MB~2GB**
- 같은 파일로 여러 번 질문할 때
- 총 요청 크기가 20MB를 초과할 때

### 핵심 특징

| 항목 | 내용 |
|-----|-----|
| 최대 파일 크기 | 2GB |
| 프로젝트당 총 용량 | 20GB |
| 저장 기간 | **48시간** (자동 삭제) |
| 비용 | **무료** |

### Python 예제

```python
from google import genai

client = genai.Client()

# 1. 파일 업로드
uploaded_file = client.files.upload(file="path/to/video.mp4")
print(f"업로드 완료: {uploaded_file.name}")
print(f"URI: {uploaded_file.uri}")

# 2. 업로드된 파일로 질문하기
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=["이 영상의 주요 장면을 설명해줘", uploaded_file]
)
print(response.text)

# 3. 같은 파일로 추가 질문 (48시간 내)
response2 = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=["이 영상에서 등장하는 사람들은?", uploaded_file]
)
print(response2.text)
```

### JavaScript 예제

```javascript
import {
  GoogleGenAI,
  createUserContent,
  createPartFromUri,
} from "@google/genai";

const ai = new GoogleGenAI({});

async function main() {
  // 1. 파일 업로드
  const myfile = await ai.files.upload({
    file: "path/to/audio.mp3",
    config: { mimeType: "audio/mpeg" },
  });
  console.log(`업로드 완료: ${myfile.name}`);

  // 2. 업로드된 파일로 질문하기
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: createUserContent([
      createPartFromUri(myfile.uri, myfile.mimeType),
      "이 오디오의 내용을 요약해줘",
    ]),
  });
  console.log(response.text);
}

await main();
```

### Go 예제

```go
file, err := client.Files.UploadFromPath(ctx, "path/to/sample.mp3", nil)
if err != nil {
    log.Fatal(err)
}
defer client.Files.Delete(ctx, file.Name) // 사용 후 삭제

resp, err := client.Models.GenerateContent(ctx, "gemini-2.5-flash", []*genai.Content{
  {
    Parts: []*genai.Part{
      genai.NewPartFromFile(*file),
      genai.NewPartFromText("이 오디오의 내용을 요약해줘"),
    },
  },
}, nil)

if err != nil {
    log.Fatal(err)
}
printResponse(resp)
```

> **TIP**: 48시간이 지나면 파일이 자동 삭제됩니다. 장기 보관이 필요하면 GCS를 사용하세요.

---

## 방법 3: GCS 등록 (Google Cloud Storage)

이미 **Google Cloud Storage에 있는 파일**을 다운로드/재업로드 없이 바로 사용합니다.

### 언제 사용하나요?

- 데이터가 이미 GCS에 저장되어 있을 때
- 대용량 파일을 장기간 보관하며 사용할 때
- 프로덕션 환경에서 안정적인 파일 관리가 필요할 때

### 설정 단계

#### 1단계: Service Agent 권한 부여

```bash
# 1. Gemini API 활성화 후 Service Agent 생성
gcloud beta services identity create \
  --service=generativelanguage.googleapis.com \
  --project=<your_project>

# 2. Service Agent에 Storage Object Viewer 역할 부여
# Google Cloud Console > IAM에서 설정
```

#### 2단계: 인증 설정

**로컬 개발 환경** (Google Cloud 외부):

```python
from google.oauth2.service_account import Credentials

GCS_READ_SCOPES = [
    'https://www.googleapis.com/auth/devstorage.read_only',
    'https://www.googleapis.com/auth/cloud-platform'
]

credentials = Credentials.from_service_account_file(
    'service-account.json',
    scopes=GCS_READ_SCOPES
)
```

**Google Cloud 환경** (Cloud Run, Compute Engine 등):

```python
import google.auth

GCS_READ_SCOPES = [
    'https://www.googleapis.com/auth/devstorage.read_only',
    'https://www.googleapis.com/auth/cloud-platform'
]

credentials, project = google.auth.default(scopes=GCS_READ_SCOPES)
```

#### 3단계: 파일 등록 및 사용

```python
from google import genai
from google.genai.types import Part

client = genai.Client()

# GCS 파일 등록
registered_files = client.files.register_files(
    uris=[
        "gs://my_bucket/documents/report.pdf",
        "gs://my_bucket/images/chart.png"
    ],
    auth=credentials
)

# 등록된 파일로 질문하기
for f in registered_files.files:
    print(f"등록 완료: {f.name}")

    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[
            Part.from_uri(file_uri=f.uri, mime_type=f.mime_type),
            "이 파일의 내용을 분석해줘"
        ],
    )
    print(response.text)
```

> **TIP**: GCS 등록은 한 번 하면 **최대 30일간** 유효합니다. File API Upload의 48시간보다 훨씬 깁니다.

---

## 방법 4: External URL (외부 URL)

**공개된 HTTPS URL**이나 **사전 서명된 URL**(S3 Presigned URL, Azure SAS 등)을 직접 전달합니다.

### 언제 사용하나요?

- 파일이 이미 웹에 공개되어 있을 때
- AWS S3, Azure Blob 등에 저장된 파일을 사용할 때
- 파일을 다운로드/업로드하고 싶지 않을 때

### 제한사항

- 최대 **100MB**
- **Gemini 2.0 모델에서는 지원되지 않음** (2.5 이상 사용)
- 로그인이 필요하거나 페이월 뒤에 있는 URL은 사용 불가

### Python 예제

```python
from google import genai
from google.genai.types import Part

client = genai.Client()

# 공개 URL 직접 사용
pdf_url = "https://example.com/public/whitepaper.pdf"

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        Part.from_uri(
            file_uri=pdf_url,
            mime_type="application/pdf",
        ),
        "이 백서의 핵심 주장은 무엇인가요?"
    ],
)
print(response.text)
```

### JavaScript 예제

```javascript
import { GoogleGenAI, createPartFromUri } from '@google/genai';

const client = new GoogleGenAI({});

const pdfUrl = "https://example.com/public/whitepaper.pdf";

async function main() {
  const response = await client.models.generateContent({
    model: 'gemini-2.5-flash',
    contents: [
      createPartFromUri(pdfUrl, "application/pdf"),
      "이 백서의 핵심 주장은 무엇인가요?",
    ],
  });
  console.log(response.text);
}

main();
```

### 지원되는 파일 형식 (External URL)

| 카테고리 | MIME Types |
|---------|------------|
| **텍스트** | `text/html`, `text/css`, `text/plain`, `text/xml`, `text/csv`, `text/rtf`, `text/javascript` |
| **애플리케이션** | `application/json`, `application/pdf` |
| **이미지** | `image/bmp`, `image/jpeg`, `image/png`, `image/webp` |

### 안전성 검사

Gemini는 URL에 대해 **콘텐츠 검수**를 수행합니다:
- 로그인 필요 페이지 → 실패
- 페이월 콘텐츠 → 실패
- 안전하지 않은 콘텐츠 → `URL_RETRIEVAL_STATUS_UNSAFE` 오류

> **TIP**: 비공개 파일에는 **사전 서명된 URL**을 사용하세요. 적절한 만료 시간과 최소 권한을 설정하는 것이 중요합니다.

---

## 파일 관리 API

File API로 업로드한 파일을 관리하는 방법입니다.

### 파일 목록 조회

```python
from google import genai

client = genai.Client()

print('내 파일 목록:')
for f in client.files.list():
    print(f"  - {f.name} ({f.mime_type}, {f.size_bytes} bytes)")
```

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

async function main() {
  const listResponse = await ai.files.list({ config: { pageSize: 10 } });
  for await (const file of listResponse) {
    console.log(`${file.name} - ${file.mimeType}`);
  }
}

await main();
```

### 파일 메타데이터 조회

```python
from google import genai

client = genai.Client()

# 파일 업로드
myfile = client.files.upload(file='path/to/sample.mp3')

# 메타데이터 조회
file_info = client.files.get(name=myfile.name)
print(f"이름: {file_info.name}")
print(f"URI: {file_info.uri}")
print(f"MIME: {file_info.mime_type}")
print(f"크기: {file_info.size_bytes} bytes")
print(f"상태: {file_info.state}")  # PROCESSING, ACTIVE, FAILED
```

### 파일 삭제

파일은 **48시간 후 자동 삭제**되지만, 수동으로 즉시 삭제할 수도 있습니다.

```python
from google import genai

client = genai.Client()

# 파일 삭제
client.files.delete(name="files/abc123xyz")
print("파일이 삭제되었습니다.")
```

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

async function main() {
  await ai.files.delete({ name: "files/abc123xyz" });
  console.log("파일이 삭제되었습니다.");
}

await main();
```

### 파일 상태 (State)

| 상태 | 설명 |
|-----|-----|
| `PROCESSING` | 업로드 중 / 처리 중 |
| `ACTIVE` | 사용 가능 |
| `FAILED` | 업로드/처리 실패 |

> **TIP**: 대용량 비디오는 `PROCESSING` 상태가 오래 걸릴 수 있습니다. `ACTIVE` 상태가 될 때까지 기다린 후 사용하세요.

---

## 지원 파일 형식

### 이미지

| MIME Type | 확장자 | 비고 |
|-----------|-------|-----|
| `image/jpeg` | .jpg, .jpeg | 가장 일반적 |
| `image/png` | .png | 투명 배경 지원 |
| `image/webp` | .webp | 웹 최적화 |
| `image/bmp` | .bmp | 비압축 |
| `image/heic` | .heic | iPhone 기본 (PNG/JPEG 변환 권장) |
| `image/heif` | .heif | iPhone 기본 (PNG/JPEG 변환 권장) |

> **이미지 TIP**: 최대 이미지 크기는 **7MB**입니다. 페이지당 **3072x3072** 픽셀로 자동 스케일링됩니다.

### 오디오

| MIME Type | 확장자 | 토큰 계산 |
|-----------|-------|---------|
| `audio/mpeg` | .mp3 | 1초 = 32 토큰 |
| `audio/wav` | .wav | 1분 = 1,920 토큰 |
| `audio/ogg` | .ogg | |
| `audio/flac` | .flac | |
| `audio/aac` | .aac | |
| `audio/aiff` | .aiff | |

### 비디오

| MIME Type | 확장자 |
|-----------|-------|
| `video/mp4` | .mp4 |
| `video/mpeg` | .mpeg |
| `video/webm` | .webm |
| `video/mov` | .mov |
| `video/avi` | .avi |
| `video/x-flv` | .flv |
| `video/wmv` | .wmv |
| `video/3gpp` | .3gp |

### 문서

| MIME Type | 확장자 | 비고 |
|-----------|-------|-----|
| `application/pdf` | .pdf | **가장 권장** (레이아웃/차트 인식) |
| `text/plain` | .txt | 일반 텍스트 |
| `text/html` | .html | 웹 페이지 |
| `text/markdown` | .md | 마크다운 |
| `text/csv` | .csv | CSV 데이터 |
| `application/json` | .json | JSON 데이터 |

> **문서 TIP**: PDF만 **시각적 요소**(차트, 다이어그램, 표 레이아웃)를 제대로 인식합니다. 다른 형식은 텍스트로만 처리됩니다.

### PDF 제한사항

| 항목 | 제한 |
|-----|-----|
| 최대 파일 크기 | 50MB (Inline), 2GB (File API) |
| 최대 페이지 수 | 1,000 페이지 |
| 페이지당 토큰 | 약 258 토큰 |

---

## 멀티모달 프롬프트 전략

파일과 함께 **효과적인 프롬프트**를 작성하는 방법입니다.

### 기본 원칙

#### 1. 구체적으로 지시하기

```
❌ 나쁜 예: "이 이미지를 설명해줘"
✅ 좋은 예: "이 공항 안내판에서 시간과 도시를 목록으로 추출해줘"
```

#### 2. 단일 이미지는 텍스트보다 먼저 배치

```python
# 권장: 이미지를 먼저
contents=[image_part, "이 이미지를 분석해줘"]

# 비권장: 텍스트를 먼저
contents=["이 이미지를 분석해줘", image_part]
```

#### 3. 출력 형식 명시하기

```python
# JSON 형식 요청
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        image_part,
        "이 음식 사진에서 재료, 요리 종류, 채식 여부를 JSON 형식으로 알려줘"
    ]
)
```

#### 4. 단계별로 나누기 (복잡한 작업)

```python
# 복잡한 계산은 단계별로
contents=[
    math_image,
    """
    1. 먼저 이미지에서 수식을 파악해줘
    2. 그 다음 수식을 풀어줘
    3. 마지막으로 답을 알려줘
    """
]

# 또는 간단하게
contents=[math_image, "수열의 4번째 항을 구해줘. 단계별로 생각해봐."]
```

#### 5. Few-shot 예시 제공하기

여러 이미지와 원하는 응답 형식을 예시로 제공하면 일관된 출력을 얻을 수 있습니다.

```python
contents=[
    # 예시 1
    colosseum_image,
    "city: Rome, landmark: the Colosseum",

    # 예시 2
    forbidden_city_image,
    "city: Beijing, landmark: Forbidden City",

    # 실제 질문
    target_image,
    "위 형식대로 도시와 랜드마크를 알려줘"
]
```

---

## 실전 활용 예시

### 예시 1: PDF 문서 요약

```python
from google import genai
from google.genai import types

client = genai.Client()

# PDF 업로드 및 요약
pdf_file = client.files.upload(file="quarterly_report.pdf")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        pdf_file,
        """
        이 분기 보고서를 분석해서 다음 형식으로 요약해줘:

        ## 핵심 요약
        - 주요 성과 3가지
        - 주요 과제 2가지

        ## 재무 하이라이트
        - 매출
        - 영업이익
        - 전년 대비 성장률

        ## 향후 전망
        - 다음 분기 예측
        """
    ]
)
print(response.text)
```

### 예시 2: 이미지 속 표 데이터 추출

```python
from google import genai
from google.genai import types

client = genai.Client()

# 스크린샷에서 표 추출
with open("spreadsheet_screenshot.png", "rb") as f:
    image_data = f.read()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        types.Part.from_bytes(data=image_data, mime_type="image/png"),
        "이 이미지의 표를 마크다운 테이블로 변환해줘"
    ]
)
print(response.text)
```

### 예시 3: 비디오 분석

```python
from google import genai

client = genai.Client()

# 비디오 업로드 (대용량이므로 File API 사용)
video_file = client.files.upload(file="product_demo.mp4")

# 비디오 처리 완료 대기
import time
while video_file.state == "PROCESSING":
    time.sleep(5)
    video_file = client.files.get(name=video_file.name)

if video_file.state == "ACTIVE":
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[
            video_file,
            """
            이 제품 데모 영상을 분석해서:
            1. 주요 기능 목록을 만들어줘
            2. 각 기능이 등장하는 시간(타임스탬프)을 알려줘
            3. 전체적인 제품 장단점을 평가해줘
            """
        ]
    )
    print(response.text)
```

### 예시 4: 오디오 트랜스크립션

```python
from google import genai

client = genai.Client()

# 오디오 파일 업로드
audio_file = client.files.upload(file="meeting_recording.mp3")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        audio_file,
        """
        이 회의 녹음을 분석해서:

        1. 전체 내용을 텍스트로 변환해줘
        2. 주요 논의 사항을 정리해줘
        3. 결정된 액션 아이템을 목록으로 만들어줘
        4. 각 참석자별 발언 요약을 해줘
        """
    ]
)
print(response.text)
```

### 예시 5: 여러 이미지 비교 분석

```python
from google import genai
from google.genai import types

client = genai.Client()

# 여러 이미지 로드
def load_image(path):
    with open(path, "rb") as f:
        return types.Part.from_bytes(data=f.read(), mime_type="image/jpeg")

before_image = load_image("before.jpg")
after_image = load_image("after.jpg")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        "첫 번째 이미지 (Before):",
        before_image,
        "두 번째 이미지 (After):",
        after_image,
        """
        두 이미지를 비교해서:
        1. 변화된 점을 상세히 설명해줘
        2. 개선된 부분과 악화된 부분을 구분해줘
        3. 전체적인 변화에 대한 평가를 해줘
        """
    ]
)
print(response.text)
```

---

## 문제 해결 가이드

### 문제 1: 모델이 이미지의 관련 부분을 인식하지 못함

**해결책**: 어떤 부분에 집중해야 하는지 힌트를 주세요.

```python
# ❌ 문제가 있는 프롬프트
contents=[diaper_image, "이 기저귀가 며칠 동안 쓸 수 있을까?"]

# ✅ 개선된 프롬프트
contents=[
    diaper_image,
    """
    이 기저귀가 며칠 동안 쓸 수 있을지 계산해줘.
    박스에 표시된 체중 범위로 아이 나이를 추정하고,
    박스에 들어있는 총 기저귀 개수를 확인한 후,
    아이가 하루에 사용하는 기저귀 개수로 나눠줘.
    """
]
```

### 문제 2: 응답이 너무 일반적임

**해결책**: 먼저 이미지를 설명하게 하거나, 이미지 내용을 참조하도록 요청하세요.

```python
# ❌ 일반적인 응답이 나오는 프롬프트
contents=[images, "이 이미지들의 공통점은?"]

# ✅ 개선된 프롬프트
contents=[
    images,
    "먼저 각 이미지에 무엇이 있는지 자세히 설명해줘. 그 다음 공통점을 찾아줘."
]

# 또는
contents=[
    images,
    "이 이미지들의 공통점은? 답변에 이미지 내용을 구체적으로 언급해줘."
]
```

### 문제 3: 환각(hallucination) 발생

**해결책**: temperature를 낮추거나 짧은 설명을 요청하세요.

```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[image, "이 이미지를 간단히 설명해줘"],
    generation_config={
        "temperature": 0.7,  # 기본값 1.0보다 낮게
        "max_output_tokens": 200  # 출력 제한
    }
)
```

> **주의**: Gemini 3 모델에서는 temperature를 **1.0 (기본값)**으로 유지하는 것을 권장합니다. 낮은 temperature는 루핑이나 성능 저하를 유발할 수 있습니다.

### 문제 4: 어디서 실패했는지 모르겠음

**해결책**: 단계별로 디버깅하세요.

```python
# 1단계: 이미지 인식 확인
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[image, "이 이미지에 무엇이 보이는지 설명해줘"]
)
print("이미지 인식 결과:", response.text)

# 2단계: 추론 과정 확인
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[image, "이 이미지와 어울리는 간식은? 이유도 설명해줘."]
)
print("추론 결과:", response.text)
```

### 문제 5: External URL 오류

**오류 메시지**: `URL_RETRIEVAL_STATUS_UNSAFE`

**원인 및 해결책**:
- 로그인 필요 페이지 → 공개 URL 사용 또는 Inline Data로 전환
- 페이월 콘텐츠 → 파일을 다운로드 후 File API 사용
- 사전 서명 URL 만료 → 새 URL 생성

---

## 제한사항 및 주의점

### 파일 크기 제한

| 방법 | 일반 파일 | PDF |
|-----|---------|-----|
| Inline Data | 100MB | 50MB |
| File API | 2GB | 2GB |
| External URL | 100MB | 100MB |

### 저장 및 보존

- **File API 업로드**: 48시간 후 자동 삭제
- **GCS 등록**: 30일간 유효 (파일 자체는 GCS에 유지)
- **Inline/External URL**: 저장되지 않음 (매번 전송)

### 다운로드 불가

업로드된 파일은 **메타데이터만 조회 가능**하고, 파일 자체는 다운로드할 수 없습니다.

### 모델별 지원

- **External URL**: Gemini 2.0에서는 지원되지 않음 (2.5 이상 필요)

### 보안 주의사항

1. **사전 서명 URL**: 적절한 만료 시간 설정 필수
2. **GCS 권한**: 최소 권한 원칙 (Storage Object Viewer만 부여)
3. **민감한 정보**: API를 통해 전송되는 파일에 민감한 정보가 없는지 확인

---

## 추가 리소스

- [Google AI Studio](https://aistudio.google.com) - 웹에서 직접 테스트
- [Vision 가이드](https://ai.google.dev/gemini-api/docs/vision) - 이미지 처리 상세
- [Audio 가이드](https://ai.google.dev/gemini-api/docs/audio) - 오디오 처리 상세
- [Document 가이드](https://ai.google.dev/gemini-api/docs/document-processing) - 문서 처리 상세
- [프롬프트 전략](https://ai.google.dev/gemini-api/docs/prompting-strategies) - 효과적인 프롬프트 작성법

---

## 빠른 참조 카드

```
┌────────────────────────────────────────────────────────────────┐
│                    Gemini File API 빠른 참조                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  📥 Inline (작은 파일)                                          │
│  types.Part.from_bytes(data=bytes, mime_type="...")            │
│                                                                │
│  📤 Upload (대용량)                                             │
│  file = client.files.upload(file="path")                       │
│  contents=[file, "프롬프트"]                                    │
│                                                                │
│  ☁️ GCS 등록                                                    │
│  files = client.files.register_files(uris=["gs://..."])        │
│                                                                │
│  🔗 External URL                                                │
│  Part.from_uri(file_uri="https://...", mime_type="...")        │
│                                                                │
│  📋 파일 관리                                                    │
│  client.files.list()      # 목록                                │
│  client.files.get(name)   # 조회                                │
│  client.files.delete(name) # 삭제                               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

*이 문서는 [Google AI Gemini API 공식 문서](https://ai.google.dev/gemini-api/docs/files)를 기반으로 작성되었습니다.*
