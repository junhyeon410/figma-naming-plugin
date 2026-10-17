---
name: figma-naming
description: Figma 컴포넌트·속성·레이어·변수·글자 스타일·화면 프레임 이름을 코드 대응 네이밍 규칙으로 검사하고, 미리보기 승인 후 일괄 Rename할 때 사용.
---

# Figma 네이밍 정리

원칙: **코드에서 그 이름이 쓰이는 표기법을 따른다.** Figma 이름만 보고 Claude Code가 코드 이름을 바로 맞출 수 있어야 한다.

## 규칙

| 대상 | 표기법 | 예 | 코드 대응 |
|---|---|---|---|
| 컴포넌트 (Component, Component Set) | PascalCase, 영어 | `Button`, `DecisionCard`, `SidebarItem` | React 컴포넌트 이름 |
| 아이콘 컴포넌트 | `icon/` + kebab-case 아이콘명 | `icon/close`, `icon/chevron-down` | `CloseIcon`, `ChevronDownIcon` |
| 속성 이름 (variant, boolean, text, instance swap) | camelCase | `variant`, `size`, `state`, `hasIcon` | React props |
| 속성 값 (variant 값) | 소문자, 두 단어면 kebab-case | `default`, `hover`, `icon-only` | React prop 값 (`variant="ghost"`) |
| 레이어 (컴포넌트 안, 화면 안 모두) | kebab-case | `label`, `icon-left`, `container` | CSS 클래스 모양 |
| 인스턴스 | 원본 컴포넌트 이름 그대로 | `Button`, `icon/close` | 사용하는 컴포넌트 |
| 변수 | kebab-case 세그먼트 + `/` 묶음 | `bg/canvas`, `brand/bg-hover`, `space/16` | CSS 변수 `--bg-canvas` |
| 글자 스타일 | `역할/크기`, 기본 굵기가 아니면 `-굵기` | `body/md`, `body/md-semibold`, `title/lg` | |
| 화면 프레임 (페이지 바로 아래 Frame) | `화면ID 한글이름 / 상태` | `M-01 질문하기 / empty` | 화면 목록 ID |

세부 규칙:
- 컴포넌트 이름은 영어만 쓴다. 한글 설명은 컴포넌트 description에 둔다.
- `state` 속성에는 상호작용 상태만 둔다: `default`, `hover`, `pressed`, `focus`, `disabled`, 필요하면 `loading`, `error`.
- 유지되는 의미 상태는 `state`에 넣지 않고 boolean 속성으로 나눈다: `selected`·`active` → `isSelected`, `expanded` → `isExpanded`, `on`/`off`·`checked` → `isChecked`, `completed` → `isCompleted`, `drag` → `isDragging`. `selected_hover`처럼 두 상태를 합친 값은 `isSelected=true` + `state=hover`로 나눈다. 나누는 작업은 variant 구조가 바뀌므로 항상 검토 필요로 둔다.
- boolean 속성은 `has` 또는 `is`로 시작한다: `hasIcon`, `isSelected`.
- 약어는 먼저 풀어 쓴다 (`SNB` → `SideNav`, `FAB` → `FloatingActionButton`). 남기는 약어는 PascalCase에서 대문자를 유지하고(`AIPanel`), camelCase·kebab-case에서는 소문자로 쓴다(`aiPanel`, `ai-panel`). 예외는 두지 않는다.
- 크기(`size`) 값은 `sm`, `md`, `lg`를 쓴다. 단계가 더 필요하면 `xs`, `xl`을 붙인다. `small`, `Medium`, `Large`, `S/M/L`은 자동 변환 대상이다.
- 화면 상태 값은 `default`, `loading`, `empty`, `error`.
- 화면 ID는 화면 목록 문서나 사용자가 준 목록에서만 가져온다.
- 글자 스타일 역할은 `display`, `title`, `body`, `label`. 크기는 `sm`, `md`, `lg`. 기본 굵기(regular·medium)는 접미사를 붙이지 않고, 같은 크기에 더 굵은 스타일이 있으면 `-semibold`, `-bold`를 붙인다.
- 화면 안 텍스트 레이어는 역할로 이름 붙인다: `title`, `label`, `description`, `caption`, `placeholder` 등. 이름을 바꾸면 내용 따라가기(자동 이름)는 꺼진다.
- 아이콘·그래픽 안의 벡터, 불리언 연산, 도형(`Vector`, `Union`, `Ellipse 2` 등)은 검사와 변경에서 뺀다.
- `Frame 123`, `Group 5`, `Rectangle 4` 같은 Figma 기본 이름은 남기지 않는다 (아이콘·그래픽 안 벡터는 제외).
- snake_case는 쓰지 않는다.
- 변수 이름의 `/`를 `-`로 바꾸고 앞에 `--`를 붙인 것이 CSS 변수 이름이다.

## 작업 흐름

### 0. 준비
- Figma MCP가 연결돼 있는지 확인한다. 연결이 안 돼 있으면 그 사실을 알리고 멈춘다. 이름을 추측해서 목록을 만들지 않는다.
- `use_figma`를 쓰기 전에 반드시 `figma-use` skill을 먼저 불러온다.
- 사용자에게 받을 것: 파일 URL, 범위(선택 영역 / 페이지 / 파일 전체), 대상 유형, 모드(검사만 / 검사 후 Rename). 말하지 않았으면 범위는 현재 페이지, 대상은 전체, 모드는 검사 후 Rename으로 둔다.

### 1. 스캔 (읽기만)
`use_figma`로 이름과 맥락만 모은다. 이 단계에서는 아무것도 바꾸지 않는다.
- 노드: id, type, name, 부모 경로, 컴포넌트 안인지, 인스턴스 안인지, `remote` 여부, 잠금·숨김 여부.
- Component Set: `componentPropertyDefinitions` 전체 (키, type, variant 값 목록). Component Set에 에러가 있으면 이 값을 읽을 때 예외가 난다. try/catch로 감싸고, 그 세트는 "Figma에서 먼저 고칠 것"으로 보고한다.
- 변수: `getLocalVariableCollectionsAsync()`, `getLocalVariablesAsync()` 결과의 이름, description, 컬렉션, 모드 이름.
- 글자 스타일: `getLocalTextStylesAsync()`.
- 페이지마다 `use_figma`를 한 번씩 호출하고, 그 안에서 `figma.setCurrentPageAsync(page)`를 한 번만 한다. MCP 환경에서는 `loadAllPagesAsync()`를 쓸 수 없다.
- 결과는 전체 목록이 아니라 분류별 개수와 예시로 돌려받는다. 레이어가 수천 개인 페이지는 전체 목록을 받으면 응답 한도를 넘는다.

### 2. 분류와 제안
이름마다 셋 중 하나로 나눈다.
- **자동 변환**: 의미는 맞고 표기만 틀린 것. 규칙대로 기계적으로 바꾼다 (`Icon Left` → `icon-left`, `primary-button` → `PrimaryButton`, `has_icon` → `hasIcon`, `BG/Canvas` → `bg/canvas`, `size=Large` → `size=lg`, `ic_close` → `icon/close`, `body/body_sb_14` → `body/md-semibold`). 이름을 바꿔서 원본 컴포넌트와 달라진 인스턴스는 원본 이름으로 되돌린다.
- **검토 필요**: 이름의 의미를 판단해야 하는 것. 기본 이름(`Frame 123`), 한글 레이어명, 컴포넌트 이름이 한글인 경우, 상태 이름이 정해진 값 목록에 없는 경우(`Active`, `Selected` 등), boolean 접두어가 없는 경우(`icon` → `hasIcon`인지 판단), 의미 상태를 boolean으로 나누는 경우, 화면 안 텍스트의 역할 이름, 약어를 풀어 쓰는 경우. 레이어 구조, 자식 텍스트, 위치, 필요하면 스크린샷을 보고 이름을 제안하되 근거를 한 줄 붙인다.
- **건너뜀**: 바꾸면 안 되거나 바꿀 수 없는 것. 이유를 남긴다.

토큰화: 공백, `-`, `_`, `.`, camelCase 경계, 숫자 경계에서 단어를 자른다. `iOS`, `URL`, `ID`, `CTA`, `AI` 같은 약어는 한 단어로 자르고, 약어 규칙에 따라 대문자를 유지한다 (`url_input` → `URLInput`).

### 3. 미리보기
대상 유형별로 묶은 표로 보여준다: `현재 이름 → 제안 이름 | 분류 | 근거`. 맨 위에 유형별 개수 요약을 둔다. 항목이 많으면 자동 변환은 개수와 대표 예 몇 개만, 검토 필요는 전부 보여준다.

사용자가 승인한 항목만 적용한다. "전부", "자동 변환만", "이 번호 빼고" 같은 답을 받는다. 검사만 모드면 여기서 끝낸다.

### 4. 적용
대상별로 바꾸는 방법이 다르다.
- 레이어, 화면 프레임, 컴포넌트, Component Set: `node.name`.
- variant 속성 이름: Component Set의 `editComponentProperty(키, { name })`.
- boolean, text, instance swap 속성 이름: 키에 `#123:0` 같은 ID가 붙어 있다. 정의에서 읽은 전체 키로 `editComponentProperty`를 호출한다.
- variant 값: 각 variant 컴포넌트의 이름(`size=Large, state=Default`)을 새 값으로 다시 조립해 `name`에 넣는다. 속성 이름과 값 순서를 유지하고, 한 세트 안에서 같은 조합이 두 번 생기지 않는지 먼저 확인한다.
- 변수: `variable.name`. 컬렉션 이름은 `collection.name`, 모드 이름은 `collection.renameMode(modeId, name)`.
- 글자 스타일: `style.name`.

한 번의 적용은 하나의 `use_figma` 호출로 묶어서 Figma에서 되돌리기 한 번에 돌아가게 한다. 범위가 너무 커서 나눠야 하면 나눈다고 미리 알린다.

### 5. 확인
같은 범위를 다시 스캔해서 남은 위반 개수를 보고한다. 바뀐 개수, 건너뛴 항목과 이유를 짧게 정리한다.

## 건너뛰거나 주의할 것
- 라이브러리에서 가져온 컴포넌트·변수·스타일(`remote`)은 수정할 수 없다. 원본 라이브러리 파일에서 고치라고 안내한다.
- 인스턴스 안의 레이어는 바꾸지 않는다. 이름을 바꾸면 오버라이드가 생긴다. 원본 컴포넌트를 고친다.
- Text 레이어는 내용을 따라 이름이 정해지는 자동 이름 상태일 수 있다. 역할 이름을 붙이면 자동 이름이 꺼진다는 점을 미리보기에 표시한다.
- 같은 부모 아래, 같은 컬렉션 안, 같은 Component Set 안에서 이름이 겹치면 적용하지 않고 검토 필요로 돌린다. 변수는 `bg/canvas`와 `bg-canvas`처럼 CSS 이름이 같아지는 경우도 충돌로 본다.
- 화면 ID(`M-01` 등)는 만들어내지 않는다. 화면 목록 문서에서 확인되거나 사용자가 알려준 ID만 쓰고, 모르면 검토 필요로 남긴다.
- 배포된 라이브러리의 컴포넌트·속성·변수 이름 변경은 Publish 후 구독 파일과 코드 쪽 이름에 영향을 준다. 적용 전에 이 점을 한 줄로 알린다.
- 편집 권한이 없는 파일은 검사만 한다.

## 답하는 방식
- 결과는 짧게. 표와 개수 위주로 보여주고, 설명은 검토 필요 항목의 근거에만 쓴다.
- 규칙에 없는 경우를 만나면 임의로 정하지 말고, 기본안을 제안하면서 물어본다. 사용자가 정한 내용은 이 규칙에 추가할 후보로 따로 모아 마지막에 알려준다.