# figma-naming

Figma 파일의 컴포넌트, 속성, 레이어, 변수, 글자 스타일, 화면 프레임 이름을 코드에서 쓰는 표기법에 맞게 검사하고 고치는 Claude Code 플러그인.

## 설치

Claude Code에서:

```
/plugin marketplace add junhyeon410/figma-naming-plugin
/plugin install figma-naming@junhyun-plugins
```

설치 후 Claude Code를 다시 켜고 `/mcp`에서 figma를 골라 로그인한다.

## 사용

```
/figma-naming:figma-naming <Figma 링크> 검사만 해줘
```

처음에는 이름을 바꾸지 않고 미리보기 표만 보여준다. 승인한 항목만 실제로 바뀐다.

## 구성

- `skills/figma-naming/SKILL.md`: 네이밍 규칙과 작업 순서
- `.mcp.json`: Figma MCP 서버 연결
- `.claude-plugin/`: 플러그인, 마켓플레이스 정보

## 규칙 수정

`skills/figma-naming/SKILL.md`를 고치고 `plugin.json`의 `version`을 올린 다음 푸시한다. 쓰는 쪽에서는 `/plugin update figma-naming`으로 받는다.
