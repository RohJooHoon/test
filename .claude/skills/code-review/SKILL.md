---
name: code-review
description: Review code for bugs, security, performance, and code quality
disable-model-invocation: true
---

# Code Review

## Analyze the Changes

### 분류 기준

**🔴 판정 원칙**: "프로덕션에서 사용자 또는 시스템에 실제 피해가 발생하는가?"

🔴 태그 (하나라도 해당하면 치명적): `[런타임 크래시]` `[보안]` `[데이터 오염]` `[기능 파괴]` `[레이스 컨디션]` `[빌드 파괴]` `[메모리 누수]`

- `[빌드 파괴]`에 **static export 위반** 포함 (`output: "export"` 환경에서 서버 전용 API, 런타임 의존, 요청 컨텍스트 사용)
- `[메모리 누수]`에 useEffect cleanup 누락, 이벤트 리스너/타이머/AbortController 미해제 포함
- 위 기준에 해당하지 않으면 🟡 제안 사항. 태그는 이슈당 1개만 사용.
- 린트(eslint), 타입체크(tsc)가 자동으로 잡는 이슈는 리뷰 대상 제외.

### 분석 방법

**diff의 모든 변경 파일을 빠짐없이 분석.** 주요 파일만 보고 넘어가지 않음.

분석 관점:

- **Correctness**: logic errors, null/undefined, error handling, race conditions, async
- **Security**: injection, sensitive data exposure, auth/authz
- **Performance**: unnecessary re-renders, N+1, bundle impact
- **Cross-file**: 변경된 함수/타입의 호출처 영향, hook dependency array
- **Type quality**: [docs/TYPESCRIPT.md](../../../docs/TYPESCRIPT.md) 규칙 준수 — `any`/`as`/`@ts-ignore`, `enum`, `Function`/`object` 사용, optional 남발(→discriminated union으로 갈라야 함), 외부 데이터(API·form·URL) 런타임 검증 누락, wire 타입의 도메인 표면 노출, 무의미한 generic 등. 단순 규칙 위반은 🟡, 위반이 실제 런타임 크래시·데이터 오염을 유발하면 🔴.

**심각도**: 
발견한 이슈 개수 무제한. 있는 만큼 전부 보고.
출력 전, 발견한 모든 이슈의 "최악의 시나리오"를 내부적으로 검증. 구체적 피해를 서술할 수 있으면 🔴, 없으면 🟡.

### 리뷰 완료 전 점검 (출력에 포함하지 않음)

출력 전 아래 항목을 내부적으로 확인, 누락이 있으면 분석으로 복귀:

- 모든 변경 파일을 검토했는가? (주요 파일만 보고 넘기지 않았는가)
- 삭제/변경된 함수·타입의 호출처를 확인했는가?
- 새로 추가된 상태·props의 초기값과 엣지 케이스(null, 빈 배열, 0)를 확인했는가?
- useEffect·이벤트 리스너·타이머의 cleanup이 있는가?
- review 설명과 실제 구현이 일치하는가?

## Output Format

**모든 리뷰 내용은 한국어로 작성.** (전문 용어는 영문 허용)

**문체: 전부 명사 종결(개조식).** "null 체크 누락으로 런타임 크래시", "cleanup 미등록"처럼. `~한다`·`~합니다`·해요체 금지.

```markdown
## Code Review: #<number> — <title>

### 요약

작업이 하는 일에 대한 간략한 설명.

### 🔴 치명적 이슈

각 이슈에 해당 기준 태그 표시: `[런타임 크래시]` `[보안]` `[데이터 오염]` `[기능 파괴]` `[레이스 컨디션]` `[빌드 파괴]`

### 🟡 제안 사항

개선하면 좋지만 블로커는 아닌 사항.

### 🟢 잘한 점

잘 작성된 부분에 대한 코멘트.

### 판정

- 🔴가 1개 이상 → REQUEST_CHANGES
- 🔴 없음, 🟡만 존재 → COMMENT
- 🔴, 🟡 모두 없음 → APPROVE
```