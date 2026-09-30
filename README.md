# test

Turborepo 기반의 테스트용 모노레포입니다. 여러 실험·테스트 작업을 앱 단위로 나눠 진행할 수 있도록 기본 공간을 세팅해 두었습니다.

## 기술 스택

- 모노레포: [Turborepo](https://turborepo.com) + pnpm workspace
- 앱: React 19 + Vite 8 + TypeScript 6
- 린트: oxlint

## 구조

```text
apps/
  a/                  # 테스트 앱 A
  b/                  # 테스트 앱 B
  c/                  # 테스트 앱 C
packages/             # 공유 패키지 (필요 시 추가)
package.json          # 루트 스크립트 (turbo run ...)
pnpm-workspace.yaml   # 워크스페이스 범위: apps/*, packages/*
turbo.json            # 태스크 파이프라인 정의
```

`apps/a`, `apps/b`, `apps/c`는 동일한 빈 React 앱으로 시작합니다. 각 앱은 독립적으로 실행·빌드되므로 서로 영향을 주지 않고 테스트할 수 있습니다.

## 시작하기

```bash
pnpm install
```

## 커맨드

루트에서 실행하면 모든 앱에 대해 Turborepo가 태스크를 병렬로 실행합니다.

```bash
pnpm dev           # 전체 앱 개발 서버 실행
pnpm build         # 전체 앱 빌드
pnpm lint          # 전체 앱 린트
pnpm check-types   # 전체 앱 타입 검사
```

특정 앱만 실행하려면 `--filter`를 사용합니다.

```bash
pnpm dev --filter a
pnpm build --filter b
```

## 새 테스트 앱 추가

1. 기존 앱 폴더를 복사합니다. 예: `cp -R apps/a apps/d`
2. `apps/d/package.json`의 `name`을 `d`로 바꿉니다.
3. 루트에서 `pnpm install`을 실행합니다.

여러 앱에서 함께 쓰는 코드는 `packages/` 아래에 패키지로 만들고, 앱의 `dependencies`에 `"<패키지명>": "workspace:*"`로 추가합니다.
