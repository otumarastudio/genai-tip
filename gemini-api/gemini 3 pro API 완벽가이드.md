# Gemini 이미지 생성 API 완벽 가이드

> Gemini의 네이티브 이미지 생성 및 편집 기능을 활용한 실전 가이드

---

## 한눈에 보기

### 모델 비교

| 모델 | 용도 | 최대 해상도 | 입력 이미지 | 특징 |
|------|------|-------------|-------------|------|
| `gemini-2.5-flash-image` | 빠른 생성 | 1024px | 최대 3장 | 저지연, 고볼륨 작업 최적화 |
| `gemini-3-pro-image-preview` | 전문 제작 | 4K (4096px) | 최대 14장 | 고해상도, Google Search 그라운딩, Thinking 모드 |

### 주요 기능

| 기능 | 설명 | 지원 모델 |
|------|------|-----------|
| **텍스트→이미지** | 프롬프트로 이미지 생성 | 모두 |
| **이미지 편집** | 마스크 없이 자연어로 편집 | 모두 |
| **멀티턴 편집** | 대화형 반복 수정 | 모두 |
| **스타일 전이** | 참조 이미지 스타일 적용 | 모두 |
| **고해상도 출력** | 1K/2K/4K 선택 | gemini-3-pro만 |
| **Google Search 그라운딩** | 실시간 정보 반영 | gemini-3-pro만 |
| **Thinking 모드** | 생성 과정 시각화 | gemini-3-pro만 |

### 지원 비율

```
1:1  │ 정사각형 (기본값)
2:3  │ 세로 사진
3:2  │ 가로 사진
3:4  │ 세로 (4:3 비율)
4:3  │ 가로 (표준)
4:5  │ 인스타그램 세로
5:4  │ 인스타그램 가로
9:16 │ 세로 영상 (스토리)
16:9 │ 가로 영상 (와이드)
21:9 │ 울트라와이드
```

---

## 목차

1. [빠른 시작](#1-빠른-시작)
2. [이미지 생성 (Text-to-Image)](#2-이미지-생성-text-to-image)
3. [이미지 편집 (Image Editing)](#3-이미지-편집-image-editing)
4. [멀티턴 편집 (Multi-turn Editing)](#4-멀티턴-편집-multi-turn-editing)
5. [Gemini 3 Pro 고급 기능](#5-gemini-3-pro-고급-기능)
6. [프롬프트 전략](#6-프롬프트-전략)
7. [설정 옵션](#7-설정-옵션)
8. [실전 예제](#8-실전-예제)
9. [제한사항 및 팁](#9-제한사항-및-팁)

---

## 1. 빠른 시작

### 설치

```bash
# Python
pip install google-genai

# JavaScript/Node.js
npm install @google/genai
```

### 기본 이미지 생성

**Python**
```python
from google import genai
from google.genai import types

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash-image",
    contents="고양이가 우주복을 입고 달 위를 걷는 모습"
)

# 결과 처리
for part in response.parts:
    if part.text:
        print(part.text)
    elif part.inline_data:
        image = part.as_image()
        image.save("output.png")
```

**JavaScript**
```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash-image",
  contents: "고양이가 우주복을 입고 달 위를 걷는 모습",
});

// 결과 처리
for (const part of response.parts) {
  if (part.text) {
    console.log(part.text);
  } else if (part.inlineData) {
    const buffer = Buffer.from(part.inlineData.data, "base64");
    fs.writeFileSync("output.png", buffer);
  }
}
```

---

## 2. 이미지 생성 (Text-to-Image)

프롬프트만으로 이미지를 생성합니다.

### 기본 구조

```python
response = client.models.generate_content(
    model="gemini-2.5-flash-image",  # 또는 gemini-3-pro-image-preview
    contents="생성할 이미지에 대한 설명"
)
```

### 응답 처리 패턴

응답에는 텍스트와 이미지가 섞여서 올 수 있습니다:

```python
for part in response.parts:
    if part.text is not None:
        # 텍스트 응답 (이미지 설명 등)
        print(part.text)
    elif part.inline_data is not None:
        # 이미지 데이터
        image = part.as_image()  # PIL Image 객체
        image.save("output.png")
```

### 팁: 이미지만 받기

텍스트 없이 이미지만 받고 싶다면:

```python
response = client.models.generate_content(
    model="gemini-2.5-flash-image",
    contents="프롬프트",
    config=types.GenerateContentConfig(
        response_modalities=['Image']  # 이미지만 응답
    )
)
```

---

## 3. 이미지 편집 (Image Editing)

기존 이미지를 자연어 명령으로 편집합니다. **마스크가 필요 없습니다!**

### 기본 편집

```python
from PIL import Image

# 원본 이미지 로드
original = Image.open("photo.jpg")

# 편집 요청
response = client.models.generate_content(
    model="gemini-2.5-flash-image",
    contents=[
        original,
        "이 사진에서 배경을 해변으로 바꿔줘"
    ]
)
```

### 편집 가능한 작업들

| 작업 | 프롬프트 예시 |
|------|--------------|
| **배경 변경** | "배경을 산으로 바꿔줘" |
| **요소 추가** | "테이블 위에 꽃병을 추가해줘" |
| **요소 제거** | "사진에서 사람을 제거해줘" |
| **스타일 변환** | "이 사진을 수채화 스타일로 바꿔줘" |
| **색상 변경** | "차 색깔을 빨간색으로 바꿔줘" |
| **조명 조절** | "더 따뜻한 조명으로 바꿔줘" |

---

## 4. 멀티턴 편집 (Multi-turn Editing)

대화형으로 이미지를 점진적으로 수정할 수 있습니다.

### Python - Chat 세션 활용

```python
from google import genai
from google.genai import types
from PIL import Image
from io import BytesIO
import base64

client = genai.Client()

# 채팅 세션 시작
chat = client.chats.create(model="gemini-2.5-flash-image")

# 1단계: 초기 이미지 생성
response = chat.send_message("넓은 녹색 잔디밭에 빨간 집을 그려줘")
image = None
for part in response.parts:
    if part.inline_data:
        image = part.as_image()
        image.save("step1.png")

# 2단계: 요소 추가
response = chat.send_message("집 앞에 흰색 울타리를 추가해줘")
for part in response.parts:
    if part.inline_data:
        image = part.as_image()
        image.save("step2.png")

# 3단계: 추가 수정
response = chat.send_message("하늘에 무지개를 추가해줘")
for part in response.parts:
    if part.inline_data:
        image = part.as_image()
        image.save("step3.png")
```

### JavaScript - Chat 세션 활용

```javascript
const chat = ai.chats.create({ model: "gemini-2.5-flash-image" });

// 1단계
let response = await chat.sendMessage("넓은 녹색 잔디밭에 빨간 집을 그려줘");
saveImage(response, "step1.png");

// 2단계
response = await chat.sendMessage("집 앞에 흰색 울타리를 추가해줘");
saveImage(response, "step2.png");

// 3단계
response = await chat.sendMessage("하늘에 무지개를 추가해줘");
saveImage(response, "step3.png");
```

### 중요: 멀티턴의 장점

1. **컨텍스트 유지**: 이전 대화 내용을 기억
2. **점진적 수정**: 작은 변경을 단계별로 적용
3. **실험 용이**: 다양한 변형을 빠르게 시도

---

## 5. Gemini 3 Pro 고급 기능

`gemini-3-pro-image-preview` 모델만의 특별한 기능들입니다.

### 5.1 고해상도 출력 (1K/2K/4K)

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="도시의 야경 파노라마",
    config=types.GenerateContentConfig(
        image_config=types.ImageConfig(
            aspect_ratio="21:9",
            image_size="4K"  # 1K, 2K, 4K 중 선택
        )
    )
)
```

**해상도 표**

| 이미지 크기 | 1:1 해상도 | 16:9 해상도 | 토큰 |
|-------------|------------|-------------|------|
| 1K | 1024×1024 | 1376×768 | 1210 |
| 2K | 2048×2048 | 2752×1536 | 1210 |
| 4K | 4096×4096 | 5504×3072 | 2000 |

### 5.2 이미지 내 텍스트 렌더링

Gemini 3 Pro는 이미지 안에 텍스트를 정확하게 렌더링할 수 있습니다.

```python
# 팁: 텍스트를 먼저 생성하고, 그 다음 이미지에 포함시키면 더 정확합니다
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="""
    카페 간판을 만들어줘.
    - 카페 이름: "Morning Brew"
    - 스타일: 빈티지 나무 간판
    - 텍스트는 필기체로
    """
)
```

### 5.3 Google Search 그라운딩

실시간 정보를 반영한 이미지를 생성합니다.

```python
from google.genai import types

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="2024년 파리 올림픽 마스코트 캐릭터를 그려줘",
    config=types.GenerateContentConfig(
        tools=[types.Tool(google_search=types.GoogleSearch())]
    )
)
```

**그라운딩 활용 예시**:
- 최신 유행 패션 스타일
- 실존하는 유명 장소
- 현재 시즌의 트렌드 컬러
- 최신 제품 디자인 참조

### 5.4 Thinking 모드 (사고 과정 시각화)

모델이 이미지를 생성하는 과정을 볼 수 있습니다.

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="복잡한 기계 내부 구조를 보여주는 단면도",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True,
            include_thought_images=True  # 중간 이미지도 포함
        )
    )
)

# 응답에서 사고 과정과 최종 이미지 분리
for part in response.parts:
    if hasattr(part, 'thought') and part.thought:
        print("사고 과정:", part.text)
    elif part.inline_data:
        image = part.as_image()
        image.save("final.png")
```

### 5.5 다중 참조 이미지 (최대 14장)

여러 이미지를 참조하여 새로운 이미지를 생성합니다.

**제한사항**:
- 최대 14장의 참조 이미지
- 객체: 최대 6개
- 인물: 최대 5명

```python
from PIL import Image

# 여러 참조 이미지 로드
style_ref = Image.open("style_reference.jpg")
object_ref = Image.open("object_reference.jpg")
pose_ref = Image.open("pose_reference.jpg")

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[
        style_ref,
        object_ref,
        pose_ref,
        """
        첫 번째 이미지의 스타일로,
        두 번째 이미지에 있는 제품을,
        세 번째 이미지와 같은 구도로 배치해줘.
        """
    ]
)
```

### 5.6 Thought Signature (멀티턴 최적화)

멀티턴 편집 시 이전 사고 과정을 재사용하여 일관성을 유지합니다.

```python
# 첫 번째 요청: thought signature 받기
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="미래적인 자동차 컨셉 디자인",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True,
            include_thought_signature=True  # signature 포함
        )
    )
)

# thought signature 추출
thought_signature = None
for part in response.parts:
    if hasattr(part, 'thought_signature'):
        thought_signature = part.thought_signature

# 두 번째 요청: signature 재사용
response2 = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[
        {"thought_signature": thought_signature},
        "같은 자동차를 다른 각도에서 보여줘"
    ]
)
```

---

## 6. 프롬프트 전략

### 6.1 사실적 사진 스타일

```
템플릿:
A [detailed subject] in [location], photographed with [camera/lens],
[lighting conditions], [mood/atmosphere], [technical settings]

예시:
"A weathered fisherman mending nets on a wooden dock at sunset,
photographed with a Sony A7R IV and 85mm f/1.4 lens,
golden hour backlighting creating rim light,
shallow depth of field, ISO 400, 1/500s"
```

### 6.2 예술적/스타일라이즈드

```
템플릿:
A [subject] in the style of [artist/movement],
[medium], [color palette], [mood]

예시:
"A cyberpunk street market in the style of Syd Mead,
digital painting, neon blues and magentas against dark backgrounds,
bustling with vendors selling holographic wares"
```

### 6.3 이미지 내 텍스트

```
템플릿:
A [object type] with the text "[exact text]" displayed [position],
[style details], [context]

예시:
"A vintage movie poster with the text 'STELLAR ODYSSEY'
in bold art deco lettering at the top,
a spaceship soaring through a nebula below"
```

### 6.4 제품 목업

```
템플릿:
A [product type] with [design elements] displayed in [setting],
[lighting], [angle], [style]

예시:
"A minimalist skincare bottle with a matte white finish
and gold accent cap, displayed on a marble surface
with soft diffused lighting, hero shot angle"
```

### 6.5 미니멀 아이콘/로고

```
템플릿:
A minimalist [object] icon, [number] colors,
[style], [background], vector-style

예시:
"A minimalist mountain icon, two colors (dark blue and white),
geometric shapes, transparent background,
flat design suitable for app icon"
```

### 6.6 편집 프롬프트 예시

| 작업 | 프롬프트 |
|------|----------|
| **요소 추가** | "Add a hot air balloon floating in the sky above the landscape" |
| **요소 제거** | "Remove all the cars from the street, leaving an empty road" |
| **인페인팅** | "Replace the wall art with a large abstract painting in blues and golds" |
| **스타일 전이** | "Transform this photo into a Studio Ghibli animation style" |
| **고품질 보존** | "Upscale this image to 4K while preserving all details exactly" |
| **생동감 부여** | "Bring this pencil sketch to life as a photorealistic image" |
| **360도 뷰** | "A studio portrait of this person against white, in profile looking right" |

---

## 7. 설정 옵션

### 7.1 응답 유형 (Response Modalities)

```python
# 텍스트 + 이미지 (기본값)
config = types.GenerateContentConfig(
    response_modalities=['TEXT', 'IMAGE']
)

# 이미지만
config = types.GenerateContentConfig(
    response_modalities=['IMAGE']
)
```

### 7.2 비율과 크기

**Python**
```python
# Gemini 2.5 Flash
config = types.GenerateContentConfig(
    image_config=types.ImageConfig(
        aspect_ratio="16:9"
    )
)

# Gemini 3 Pro (크기 지정 가능)
config = types.GenerateContentConfig(
    image_config=types.ImageConfig(
        aspect_ratio="16:9",
        image_size="2K"
    )
)
```

**JavaScript**
```javascript
// Gemini 2.5 Flash
const config = {
  imageConfig: {
    aspectRatio: "16:9"
  }
};

// Gemini 3 Pro
const config = {
  imageConfig: {
    aspectRatio: "16:9",
    imageSize: "2K"
  }
};
```

### 7.3 Thinking 설정

```python
config = types.GenerateContentConfig(
    thinking_config=types.ThinkingConfig(
        include_thoughts=True,        # 텍스트 사고 과정
        include_thought_images=True,  # 이미지 사고 과정
        include_thought_signature=True  # 멀티턴용 signature
    )
)
```

### 7.4 Google Search 그라운딩

```python
config = types.GenerateContentConfig(
    tools=[types.Tool(google_search=types.GoogleSearch())]
)
```

---

## 8. 실전 예제

### 예제 1: 제품 사진 생성

```python
from google import genai
from google.genai import types

client = genai.Client()

prompt = """
고급 향수병 제품 사진:
- 투명한 크리스탈 병에 금색 캡
- 병 표면에 "AURORA" 텍스트 각인
- 대리석 테이블 위에 배치
- 부드러운 스튜디오 조명
- 45도 각도에서 촬영
- 배경에 살짝 흐린 꽃잎들
"""

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=prompt,
    config=types.GenerateContentConfig(
        image_config=types.ImageConfig(
            aspect_ratio="4:5",
            image_size="2K"
        )
    )
)

for part in response.parts:
    if part.inline_data:
        part.as_image().save("perfume_product.png")
```

### 예제 2: 로고 여러 버전 생성

```python
from google import genai

client = genai.Client()
chat = client.chats.create(model="gemini-2.5-flash-image")

# 기본 로고 생성
response = chat.send_message("""
테크 스타트업 'NovaSpark' 로고를 만들어줘:
- 미니멀한 기하학적 디자인
- 파란색과 주황색 그라데이션
- 현대적이고 혁신적인 느낌
""")
save_image(response, "logo_v1.png")

# 변형 1: 단색 버전
response = chat.send_message("같은 로고를 단색 흰색 버전으로 만들어줘")
save_image(response, "logo_white.png")

# 변형 2: 다크 모드
response = chat.send_message("어두운 배경용 버전도 만들어줘")
save_image(response, "logo_dark.png")

# 변형 3: 아이콘만
response = chat.send_message("텍스트 없이 심볼만 추출해서 앱 아이콘으로 만들어줘")
save_image(response, "logo_icon.png")
```

### 예제 3: 사진 스타일 변환

```python
from google import genai
from PIL import Image

client = genai.Client()
original = Image.open("photo.jpg")

styles = [
    ("지브리 애니메이션 스타일로 변환해줘", "ghibli.png"),
    ("유화 스타일로 변환해줘", "oil_painting.png"),
    ("사이버펑크 스타일로 변환해줘", "cyberpunk.png"),
    ("미니멀한 라인아트로 변환해줘", "lineart.png"),
]

for prompt, filename in styles:
    response = client.models.generate_content(
        model="gemini-2.5-flash-image",
        contents=[original, prompt]
    )
    for part in response.parts:
        if part.inline_data:
            part.as_image().save(filename)
```

### 예제 4: 캐릭터 일관성 유지 (360도 뷰)

```python
from google import genai
from PIL import Image

client = genai.Client()
character = Image.open("character_front.png")

angles = [
    "정면을 바라보는 모습",
    "오른쪽을 바라보는 측면",
    "왼쪽을 바라보는 측면",
    "뒷모습",
    "45도 각도에서 본 모습"
]

for i, angle in enumerate(angles):
    response = client.models.generate_content(
        model="gemini-3-pro-image-preview",
        contents=[
            character,
            f"이 캐릭터를 흰 배경 스튜디오에서 {angle}으로 그려줘. 의상과 외모를 정확히 유지해줘."
        ]
    )
    for part in response.parts:
        if part.inline_data:
            part.as_image().save(f"character_angle_{i}.png")
```

### 예제 5: 최신 트렌드 반영 (그라운딩)

```python
from google import genai
from google.genai import types

client = genai.Client()

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="2024년 가을 뉴욕 패션위크에서 인기있는 스타일의 여성 패션 일러스트를 그려줘",
    config=types.GenerateContentConfig(
        tools=[types.Tool(google_search=types.GoogleSearch())],
        image_config=types.ImageConfig(
            aspect_ratio="3:4",
            image_size="2K"
        )
    )
)
```

---

## 9. 제한사항 및 팁

### 제한사항

| 항목 | 제한 |
|------|------|
| **오디오/비디오 입력** | 지원 안 함 |
| **입력 이미지 (Flash)** | 최대 3장 |
| **입력 이미지 (3 Pro)** | 고품질 5장, 총 14장까지 |
| **객체 참조** | 최대 6개 |
| **인물 참조** | 최대 5명 |
| **정확한 출력 개수** | 보장 안 됨 |

### 최적화 언어

최상의 결과를 위해 다음 언어 사용을 권장합니다:
- EN, ar-EG, de-DE, es-MX, fr-FR, hi-IN, id-ID, it-IT
- ja-JP, ko-KR, pt-BR, ru-RU, ua-UA, vi-VN, zh-CN

### 베스트 프랙티스

1. **구체적으로 작성하기**
   - ❌ "판타지 갑옷"
   - ✅ "은빛 나뭇잎 문양이 새겨진 엘프 판금 갑옷, 높은 깃과 매의 날개 모양 어깨보호대"

2. **목적과 맥락 설명하기**
   - ❌ "로고 만들어줘"
   - ✅ "고급 미니멀 스킨케어 브랜드를 위한 로고를 만들어줘"

3. **반복적으로 개선하기**
   ```
   "좋아, 근데 조명을 좀 더 따뜻하게 해줄래?"
   "그대로 유지하고, 캐릭터 표정만 더 진지하게 바꿔줘"
   ```

4. **단계별로 설명하기** (복잡한 장면)
   ```
   "먼저, 새벽 안개가 낀 고요한 숲 배경을 만들어줘.
   그 다음, 전경에 이끼 낀 고대 돌 제단을 추가해줘.
   마지막으로, 제단 위에 빛나는 검 하나를 올려줘."
   ```

5. **부정적 표현 대신 긍정적 묘사**
   - ❌ "차가 없는 거리"
   - ✅ "텅 빈 한적한 거리, 차량 통행 흔적 없음"

6. **카메라/촬영 용어 활용**
   - `wide-angle shot` (광각)
   - `macro shot` (접사)
   - `low-angle perspective` (로우앵글)
   - `bird's eye view` (조감도)
   - `bokeh background` (배경 흐림)

### 워터마크 안내

모든 생성된 이미지에는 [SynthID 워터마크](https://ai.google.dev/responsible/docs/safeguards/synthid)가 포함됩니다.

---

## 모델 선택 가이드

### Gemini 3 Pro Image Preview를 선택해야 할 때

- 4K 고해상도가 필요할 때
- 실시간 정보(트렌드, 인물, 장소)를 반영해야 할 때
- 복잡한 구도나 다수의 참조 이미지가 필요할 때
- 전문적인 에셋 제작이 목적일 때

### Gemini 2.5 Flash Image를 선택해야 할 때

- 빠른 프로토타이핑이 필요할 때
- 대량의 이미지를 생성해야 할 때
- 비용 효율성이 중요할 때
- 1024px 해상도로 충분할 때

### Imagen을 고려해야 할 때

- 극도로 사실적인 이미지가 필요할 때
- 특정 아트 스타일(인상주의, 애니메이션 등)이 중요할 때
- 브랜딩, 로고, 제품 디자인에 특화된 결과물이 필요할 때
- 고급 타이포그래피가 필요할 때
- 저지연이 최우선일 때

---

## 참고 자료

- [Cookbook 가이드](https://colab.research.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_Started_Nano_Banana.ipynb)
- [Veo 비디오 생성 가이드](https://ai.google.dev/gemini-api/docs/video)
- [Gemini 모델 목록](https://ai.google.dev/gemini-api/docs/models/gemini)
- [Imagen 가이드](https://ai.google.dev/gemini-api/docs/imagen)

---

*마지막 업데이트: 2026-01*
