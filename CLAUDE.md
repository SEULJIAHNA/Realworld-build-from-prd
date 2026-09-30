# CLAUDE.md

이 파일은 Claude Code가 이 저장소에서 작업할 때 따르는 규칙입니다.

## 소통
- 사용자에게 하는 질문, 진행 상황 안내, 최종 답변은 모두 **한국어**로 작성한다. (코드·명령어·고유명사는 원문 유지)
- 커밋 메시지도 한국어로 작성한다. 형식: `<type>: <요약>` (예: `feat: 게시글 목록 API 구현`, `docs: PRD 수정`)
- 커뮤니케이션 및 문서화에는 한국어를 사용한다.

## 프로젝트
- RealWorld 사양(Conduit, Medium.com 클론)을 구현하는 바이브 코딩 학습 프로젝트.
- **요구사항의 기준은 [docs/PRD.md](docs/PRD.md)**. 기능·API·화면을 구현하기 전에 해당 장을 먼저 확인한다.
  - API 명세·응답 포맷: 4장 / 데이터 모델: 5장 / 프론트엔드 라우팅·UI: 6장 / 기술 스택: 8장 / 수용 기준: 10장
- 작업 목록은 `docs/tasks.md`에서 관리한다.
- 백엔드 API는 RealWorld 공식 사양과 **정확히 호환**되어야 한다(경로, JSON 키 이름, 상태 코드).

## 기술 스택
| 영역 | 기술 |
| --- | --- |
| 백엔드 | Go 1.22+, `net/http` 표준 라이브러리(패턴 라우팅), `log/slog` |
| DB | SQLite + `database/sql`, **순수 SQL (ORM 금지)** |
| 인증 | JWT, 헤더 `Authorization: Token <jwt>` |
| 프론트엔드 | React 18+ / TypeScript, Vite |
| 라우팅 | TanStack Router — **코드 기반 라우트 정의** (파일 기반 `$param.tsx` 라우팅 금지) |
| 서버 상태 | TanStack Query (전역 Context는 로그인 사용자 정보에만) |
| 스타일 | Tailwind CSS + shadcn/ui (Bootstrap 테마 CSS 사용 안 함) |
| 실행 환경 | Docker 컨테이너 + Docker Compose (`backend`:8080, `frontend`:3000) |
| 테스트 | Go `testing`, Vitest, RealWorld 공식 API 테스트 |
| 린터 | golangci-lint, ESLint |

새 외부 의존성은 표준 라이브러리로 해결이 안 될 때만, 이유를 설명하고 추가한다.

## 명령어
모든 실행은 Docker 컨테이너 안에서 이루어지며 Makefile로 통일한다.

| 명령 | 동작 |
| --- | --- |
| `make dev` | 전체 서비스 기동 (`docker compose up`) |
| `make down` | 전체 서비스 중지 |
| `make logs` | `logs/` 로그 파일 확인 |
| `make test` | 백엔드·프론트엔드 테스트 실행 |
| `make migrate` | DB 마이그레이션 적용 |
| `make lint` / `make fmt` | 린트 / 포맷 |

> Makefile과 docker-compose.yml은 M1(프로젝트 설정) 단계에서 만든다. 아직 없으면 명령을 추측해 실행하지 말고 먼저 만든다.

## 코딩 원칙 (Agentic Coding)
1. **가장 단순하게 동작하는 코드**를 먼저 쓴다. 불필요한 추상화·레이어·디자인 패턴을 만들지 않는다.
2. **SQL은 문자열로 명시적으로** 작성하고 항상 파라미터 바인딩(`?`)을 쓴다.
3. **길고 설명적인 함수명**을 쓴다 (예: `getArticleBySlugWithAuthor`). 클래스·상속 대신 평범한 함수.
4. **권한 검사는 핸들러 안에서 명시적으로** 한다. "작성자만 수정/삭제 가능" 같은 검사를 다른 파일에 숨기지 않는다.
5. **로그는 항상 `logs/` 파일에** 남긴다. 문제가 생기면 먼저 로그 파일을 읽고 원인을 찾는다.
6. 명령은 빠르게 끝나야 한다. 멈추거나 계속 대기하는 명령을 포그라운드로 실행하지 않는다.
7. 리팩터링은 중복이 명확해졌을 때 한다. 너무 이르게 하지 않는다.

## 작업 방식
- 기능 하나를 구현하면 관련 테스트를 추가하고 `make test`로 확인한 뒤 완료로 본다.
- API 에러는 PRD 4.4장 형식(`422` + `{"errors": {"필드": ["메시지"]}}`, `401`/`403`/`404`)을 따른다.
- PRD와 다르게 구현해야 할 이유가 생기면 임의로 바꾸지 말고 사용자에게 먼저 묻는다.
- 커밋·푸시는 사용자가 요청할 때만 한다.
