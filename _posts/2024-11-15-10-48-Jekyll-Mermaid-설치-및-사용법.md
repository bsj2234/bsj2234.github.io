---
layout: post
title: "Jekyll에서 Mermaid 다이어그램 사용하기"
date: 2024-11-15
categories: [Blog/Jekyll]
tags: [jekyll, mermaid, diagram, documentation]
mermaid: true
media_subpath: /medias/posts
---

# Jekyll에서 Mermaid 다이어그램 사용하기

## 1. Mermaid 설치하기

### _config.yml 설정
먼저 `_config.yml` 파일에 다음 설정을 추가합니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight yaml %}
markdown: kramdown
kramdown:
  math_engine: mathjax
    {% endhighlight %}
</div>

### head.html 수정
`_includes/head.html` 파일에 Mermaid 스크립트를 추가합니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<!-- Mermaid -->
<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
<script>
  document.addEventListener("DOMContentLoaded", function () {
    mermaid.initialize({
      startOnLoad: true,
      theme: 'default',
      securityLevel: 'loose'
    });
  });
</script>
    {% endhighlight %}
</div>

## 2. Mermaid 사용하기

### 기본 사용법
포스트에서 다이어그램을 그릴 때는 `<div class="mermaid">` 태그를 사용합니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<div class="mermaid">
graph TD
    A[시작] --> B[중간]
    B --> C[끝]
</div>
    {% endhighlight %}
</div>

실제 렌더링 결과:
<div class="mermaid">
graph TD
    A[시작] --> B[중간]
    B --> C[끝]
</div>

### 화살표에 텍스트 추가
화살표에 텍스트나 순서를 표시할 수 있습니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<div class="mermaid">
graph TD
    A[시작] -->|1| B[중간]
    B -->|2| C[끝]
</div>
    {% endhighlight %}
</div>

실제 렌더링 결과:
<div class="mermaid">
graph TD
    A[시작] -->|1| B[중간]
    B -->|2| C[끝]
</div>

### 다양한 화살표 스타일
Mermaid는 여러 종류의 화살표를 지원합니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<div class="mermaid">
graph LR
    A-->B      %% 기본 화살표
    B==>C      %% 두꺼운 화살표
    C-.->D     %% 점선 화살표
    D===>E     %% 더 두꺼운 화살표
</div>
    {% endhighlight %}
</div>

실제 렌더링 결과:
<div class="mermaid">
graph LR
    A-->B
    B==>C
    C-.->D
    D===>E
</div>

### 노드 스타일링
노드의 스타일을 지정할 수 있습니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<div class="mermaid">
graph TD
    A[시작] --> B[처리]
    B --> C[끝]
    style A fill:#f9f,stroke:#333
    style C fill:#9f9,stroke:#333
</div>
    {% endhighlight %}
</div>

실제 렌더링 결과:
<div class="mermaid">
graph TD
    A[시작] --> B[처리]
    B --> C[끝]
    style A fill:#f9f,stroke:#333
    style C fill:#9f9,stroke:#333
</div>

## 3. 주의사항

1. **Front Matter 설정**
   - 머메이드를 사용하는 포스트의 Front Matter에 `mermaid: true`를 추가하면 좋습니다.

2. **렌더링 문제**
   - 코드 블록(\`\`\`mermaid)보다 `<div class="mermaid">` 사용을 권장합니다.
   - Jekyll 테마에 따라 추가 설정이 필요할 수 있습니다.

3. **성능 고려**
   - 너무 복잡한 다이어그램은 렌더링 성능에 영향을 줄 수 있습니다.
   - 필요한 페이지에만 Mermaid를 로드하는 것이 좋습니다.

## 4. 유용한 팁

1. **방향 설정**
   - TD: 위에서 아래로 (Top Down)
   - LR: 왼쪽에서 오른쪽으로 (Left Right)
   - RL: 오른쪽에서 왼쪽으로 (Right Left)
   - BT: 아래에서 위로 (Bottom Top)

2. **주석 추가**
   - `%%`를 사용하여 다이어그램 코드에 주석을 추가할 수 있습니다.

3. **복잡한 다이어그램**
   - [Mermaid Live Editor](https://mermaid.live/)를 사용하여 미리 테스트해볼 수 있습니다.

## 5. 보안 설정 (Security Level)

Mermaid 초기화 시 설정하는 `securityLevel` 옵션은 다이어그램의 기능과 보안을 제어합니다:

<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight javascript %}
mermaid.initialize({
    startOnLoad: true,
    theme: 'default',
    securityLevel: 'loose'  // 'strict', 'loose', 'antiscript' 중 선택
});
    {% endhighlight %}
</div>

### 보안 레벨 종류

1. **strict (기본값)**
   - 가장 엄격한 보안 설정
   - 외부 URL 사용 불가
   - 클릭 이벤트 비활성화
   - XSS 공격 방지에 중점

2. **loose**
   - 더 유연한 보안 설정
   - 다음 기능들이 활성화됨:
     - 외부 이미지 URL 사용
     - 클릭 이벤트
     - 확장된 CSS 스타일링
     - 외부 링크

3. **antiscript**
   - strict와 loose의 중간 수준
   - 스크립트 실행은 방지
   - 일부 기능적 요소 허용

### 사용 예시

1. **strict 모드에서는 작동하지 않는 예:**
<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<div class="mermaid">
graph TD
    A[시작] --> B[중간]
    click B href "https://example.com" _blank    %% strict에서는 작동 안 함
    style B fill:url(#gradient)                  %% strict에서는 제한됨
</div>
    {% endhighlight %}
</div>

2. **loose 모드에서 가능한 고급 기능:**
<div class="code-block-container">
    <button class="code-toggle">Expand</button>
    {% highlight html %}
<div class="mermaid">
graph TD
    A[시작] --> B[중간]
    click B href "https://example.com" _blank
    style B fill:#f9f,stroke:#333,stroke-width:4px
    linkStyle 0 stroke:#ff3,stroke-width:4px,color:red
</div>
    {% endhighlight %}
</div>

실제 렌더링 결과:
<div class="mermaid">
graph TD
    A[시작] --> B[중간]
    click B href "https://example.com" _blank
    style B fill:#f9f,stroke:#333,stroke-width:4px
    linkStyle 0 stroke:#ff3,stroke-width:4px,color:red
</div>

### 보안 레벨 선택 가이드

1. **strict 사용 시나리오**
   - 사용자 입력을 받아 다이어그램을 생성하는 경우
   - 공개 웹사이트에서 보안이 중요한 경우
   - XSS 취약점 방지가 필요한 경우

2. **loose 사용 시나리오**
   - 개인 블로그나 문서
   - 신뢰할 수 있는 콘텐츠만 사용하는 경우
   - 고급 스타일링이나 인터랙션이 필요한 경우

3. **antiscript 사용 시나리오**
   - 부분적인 기능 확장이 필요한 경우
   - 스크립트 실행은 방지하고 싶은 경우
   - strict와 loose의 중간 수준의 보안이 필요한 경우

### 주의사항
- 보안 레벨 변경 시 기존 다이어그램의 작동 여부를 반드시 확인
- 공개 사이트에서는 가능한 strict 사용을 권장
- 외부 링크 사용 시 보안 위험 고려
