# uml
## Description
프로젝트의 유스케이스와 관련된 유스케이스·클래스·시퀀스 다이어그램을 Mermaid.js 문법으로 생성하고, SVG 이미지를 포함한 HTML 보고서를 생성합니다.

## Usage
/uml [유스케이스명]

## Workflow
1. `./docs/uml/[유스케이스명]` 폴더가 없으면 생성 여부를 묻습니다.
2. 폴더가 비어있거나 새로 만든 경우, 코드를 분석해 **세 종류의 .mmd 파일**을 생성합니다:
   - `usecase.mmd` — 유스케이스 다이어그램 (액터와 기능 관계)
   - `class.mmd`   — 클래스 다이어그램 (구조)
   - `sequence.mmd` — 시퀀스 다이어그램 (주요 흐름)
   - 필요 시 `interactive.mmd` 등 추가 시퀀스도 생성 가능
3. 폴더 내의 모든 `.mmd` 파일을 찾아 `mmdc`로 SVG로 변환합니다.
4. `analysis.md`가 있다면 내용을 읽어 HTML 보고서 상단에 통합합니다.
5. 모든 다이어그램이 포함된 HTML 보고서를 생성하고 브라우저로 엽니다.

## 유스케이스 다이어그램 (usecase.mmd) 작성 규칙

Mermaid는 전용 usecase 타입이 없으므로 **flowchart LR** 으로 구현합니다.

### 노드 표기 규칙
| 요소 | Mermaid 표현 | 예시 |
|------|-------------|------|
| 액터 (사람) | `Actor["ActorName"]:::actor` | `Dev["👤 Developer"]:::actor` |
| 시스템 경계 | `subgraph SYS["SystemName"]` | `subgraph SYS["🔧 RouterMapping"]` |
| 유스케이스 (타원) | `UC1(("UseCase")):::uc` | `UC1(("타입 조건 등록")):::uc` |
| 포함 관계 | `UC1 --> UC2` | |
| 액터→유스케이스 | `Dev -->` \| 선언\| SYS` | |

### 레이아웃 원칙
- 액터는 시스템 경계 **밖**에 배치
- 유스케이스는 `subgraph` **안**에 배치
- 방향: `flowchart LR` (액터 왼쪽, 시스템 오른쪽)
- 스타일: `classDef actor fill:#dbeafe,stroke:#3b82f6` / `classDef uc fill:#f0fdf4,stroke:#22c55e`

### 예시 골격
```
flowchart LR
    classDef actor fill:#dbeafe,stroke:#3b82f6,color:#1e3a8a
    classDef uc    fill:#f0fdf4,stroke:#22c55e,color:#14532d

    Dev([👤 Developer]):::actor

    subgraph SYS["🔧 MVC Generator Plugin"]
        UC1(("generate\nMVC Code")):::uc
        UC2(("select\nTables")):::uc
        UC3(("configure\nDSL")):::uc
        UC1 -->|<<include>>| UC2
    end

    Dev --- UC1
    Dev --- UC3
```

## 클래스 다이어그램 (class.mmd) 작성 규칙

### 노드 정의 순서
**참조되는 클래스를 먼저 정의하고, 메인 클래스(의존하는 클래스)를 마지막에 정의합니다.**

1. 기본 타입 / Java 기본 클래스 (`Method`, `Class` 등)
2. 상위 클래스 / 인터페이스 (`RequestMappingHandlerMapping` 등)
3. 도메인 클래스 / 애노테이션 (`RoutingCondition` 등)
4. 메인 클래스 / 핵심 구현체 (`RouterMappingHandlerMapping` 등)

### 관계 표현 규칙
**화살표 방향을 역으로** 하여 참조 관계를 명확하게 표현:

| 관계 | Mermaid 표현 | 의미 |
|------|-------------|------|
| 상속 | `Parent <\|-- Child` | Parent 상위, Child 하위 |
| 의존 | `Dependency <-- Dependent` | Dependency 상위, Dependent 하위 |
| 합성 | `Whole *-- Part` | Whole이 Part를 포함 |

### 예시 골격
```
classDiagram
    class Method {
        <<java>>
    }
    
    class RequestMappingHandlerMapping {
        <<abstract>>
        -getCustomTypeCondition(Class)
    }
    
    class RoutingCondition {
        <<annotation>>
    }
    
    class RoutingRequestCondition {
        -annotation: RoutingCondition
    }
    
    class RouterMappingHandlerMapping {
        +getCustomTypeCondition(Class)
    }
    
    RequestMappingHandlerMapping <|-- RouterMappingHandlerMapping : extends
    RoutingCondition <-- RouterMappingHandlerMapping : reads
    RoutingRequestCondition <-- RouterMappingHandlerMapping : creates
    Method <-- RouterMappingHandlerMapping : inspects
```

## HTML 보고서 — 다이어그램 뷰어 요구사항

생성하는 `index.html`의 각 다이어그램 영역은 다음 UX를 반드시 포함해야 합니다.

### 마크다운 렌더링
- `analysis.md` 파일의 마크다운을 HTML로 변환하여 보고서 상단에 통합.
- 정규표현식을 사용한 마크다운-to-HTML 변환:
  - `# 제목` → `<h1>제목</h1>`
  - `## 소제목` → `<h2>소제목</h2>`
  - `**굵은 텍스트**` → `<strong>굵은 텍스트</strong>`
  - `` `코드` `` → `<code>코드</code>`
  - `- 목록항목` → `<ul><li>목록항목</li></ul>`
  - ` ``` 코드블록 ``` ` → `<pre><code>코드블록</code></pre>`

### SVG 임베딩 방식
- `<object>` 또는 `<img>` 태그 금지.
- 각 `.svg` 파일 내용을 Python(`open().read()`)으로 읽어 HTML에 **인라인** 삽입.
- SVG 내부의 `id="mySvg"` 속성을 `id="svg-{name}"`으로 교체하고, SVG `<style>` 내 `#mySvg` 셀렉터도 `#svg-{name}`으로 모두 치환 — 여러 SVG를 인라인으로 넣을 때 CSS 충돌 방지.

### 뷰어 레이아웃 — 화면 꽉 채우기
- 뷰어 높이는 고정값 금지. CSS로 `height: calc(100vh - <header높이> - <nav높이> - <기타 UI 높이>)` 로 설정.
- `min-height: 320px` 보장.
- `ResizeObserver`로 브라우저 창 크기 변경 시 자동 재fit.

### 초기 표시 — fit-to-viewer
- 페이지 로드 후 각 다이어그램을 **뷰어 크기에 맞게 자동 스케일** (fit-to-viewer).
- SVG의 자연 크기는 `svg.viewBox.baseVal` → fallback `width/height` 속성 순으로 읽는다.
- fit 스케일 = `Math.min(viewerW / svgW, viewerH / svgH) * 0.96` (4% 여백).
- 가로세로 중앙 정렬: `tx = (vW - svgW * scale) / 2`, `ty = (vH - svgH * scale) / 2`.
- 더블클릭도 fit-to-viewer로 동작 (reset이 아님).

### 줌/패닝 동작
- **마우스 휠**: 커서 위치 기준 확대/축소. `e.preventDefault()` 로 페이지 스크롤 방지. factor = 1.1 / (1/1.1).
- **클릭+드래그**: `pointerdown/move/up` + `setPointerCapture`. 패닝.
- **더블클릭**: fit-to-viewer로 복귀.
- 줌 범위: 최소 0.05×, 최대 10×.
- 줌/패닝은 CSS `transform: translate(...) scale(...)` 로 구현. SVG DOM 직접 수정 금지.

### 컨트롤 버튼 (diagram-header 오른쪽에 배치)
- `+`  : 뷰어 중앙 기준 1.25× 확대
- `%`  : 현재 줌 표시 (zoom-label, 읽기 전용)
- `−`  : 뷰어 중앙 기준 0.8× 축소
- `≡`  : fit-to-viewer (화면에 맞추기)
- `1:1`: 100% 원본 크기로, 뷰어 중앙에 배치

### JavaScript 구현 원칙
- 외부 라이브러리 사용 금지 — 순수 vanilla ES5 호환 JS.
- 각 다이어그램마다 독립 state `{ scale, tx, ty }` 유지.
- IIFE로 전체 로직 캡슐화.
- HTML 생성 시 Python f-string을 사용하면 `{`/`}` 충돌이 생기므로 일반 문자열 연결(+) 방식으로 작성.
