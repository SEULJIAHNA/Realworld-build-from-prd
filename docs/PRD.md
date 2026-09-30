# RealWorld (Conduit) 제품 요구사항 문서 (PRD)

| 항목 | 내용 |
| --- | --- |
| 문서 버전 | v1.0 |
| 작성일 | 2026-09-30 |
| 제품명 | Conduit (RealWorld 사양 기반 Medium.com 클론) |
| 기준 사양 | [RealWorld 공식 문서](https://realworld-docs.netlify.app/implementation-creation/introduction/) |
| 목적 | 바이브 코딩 학습을 위한 풀스택 RealWorld 앱 구현 |

---

## 1. 개요

### 1.1 제품 소개
**Conduit**은 Medium.com과 유사한 **소셜 블로깅 플랫폼**입니다. 사용자는 가입 후 글을 작성·수정·삭제하고, 다른 사용자의 글에 댓글을 달거나 좋아요(favorite)를 누르고, 작성자를 팔로우할 수 있습니다. 인증을 포함한 모든 요청은 RealWorld 사양이 정의한 커스텀 REST API를 통해 이루어집니다.

### 1.2 배경 및 목적
- RealWorld는 "모든 데모 앱의 어머니(The mother of all demo apps)"로, 동일한 사양을 다양한 프레임워크로 구현해 비교할 수 있는 표준 프로젝트입니다.
- 본 프로젝트는 이 사양을 바탕으로 **바이브 코딩(AI 협업 개발)** 방식을 학습하고, **실제 운영 가능한 수준의 MVP**를 완성하는 것을 목표로 합니다.

### 1.3 목표 (Goals)
1. RealWorld 사양의 **필수 기능을 모두 구현**한다.
2. 백엔드는 공식 **API 명세 및 응답 포맷을 100% 준수**하여, 다른 RealWorld 프론트엔드와도 호환되게 한다.
3. 프론트엔드는 공식 **라우팅·템플릿 구조**를 따라 동일한 사용자 경험을 제공한다.
4. 새로운 개발자가 **10분 안에 아키텍처를 이해**할 수 있을 만큼 단순하게 유지한다.

### 1.4 비목표 (Non-Goals)
- 회원 탈퇴(사용자 삭제) 기능 — 사양상 불필요
- 댓글 수정 기능 — 사양상 생성/조회/삭제만 요구
- 이미지 파일 업로드 — 프로필 이미지는 URL 문자열로만 관리
- 실시간 알림, 검색, 관리자 기능 등 사양 외 기능
- 과도한 추상화·마이크로서비스 등 불필요한 오버엔지니어링

---

## 2. 사용자 및 시나리오

### 2.1 사용자 유형
| 유형 | 설명 | 가능한 행동 |
| --- | --- | --- |
| 비로그인 사용자 (Guest) | 가입하지 않았거나 로그인하지 않은 방문자 | 글로벌 피드 조회, 태그별 조회, 글 상세·댓글 조회, 프로필 조회, 회원가입/로그인 |
| 로그인 사용자 (Member) | JWT로 인증된 사용자 | Guest 기능 + 글 작성, 댓글 작성, 좋아요, 팔로우, 개인 피드, 설정 변경 |
| 작성자 (Owner) | 특정 글/댓글의 작성자 | Member 기능 + 본인 글 수정·삭제, 본인 댓글 삭제 |

### 2.2 핵심 사용자 스토리
- **US-01** 방문자로서, 회원가입을 해서 나만의 계정을 만들고 싶다.
- **US-02** 사용자로서, 이메일과 비밀번호로 로그인하고 설정 페이지에서 로그아웃하고 싶다.
- **US-03** 사용자로서, 내 프로필 이미지·사용자명·소개·이메일·비밀번호를 수정하고 싶다.
- **US-04** 사용자로서, 마크다운으로 글을 작성하고 태그를 붙여 발행하고 싶다.
- **US-05** 작성자로서, 내가 쓴 글을 수정하거나 삭제하고 싶다.
- **US-06** 방문자로서, 전체 글 목록을 최신순으로 페이지를 넘기며 보고 싶다.
- **US-07** 방문자로서, 인기 태그를 눌러 해당 태그의 글만 보고 싶다.
- **US-08** 사용자로서, 내가 팔로우한 작성자의 글만 모아 보는 피드를 보고 싶다.
- **US-09** 사용자로서, 마음에 드는 글에 좋아요를 누르고 취소하고 싶다.
- **US-10** 사용자로서, 글에 댓글을 달고 내가 단 댓글을 삭제하고 싶다.
- **US-11** 사용자로서, 다른 사용자를 팔로우/언팔로우하고 그의 프로필에서 작성한 글과 좋아요한 글을 보고 싶다.

---

## 3. 기능 요구사항

> 우선순위: **P0** = MVP 필수(RealWorld 사양 요구), **P1** = 품질 향상

### 3.1 인증 및 사용자 관리 (Authentication & Users)
| ID | 요구사항 | 우선순위 |
| --- | --- | --- |
| F-AUTH-01 | 사용자명·이메일·비밀번호로 회원가입할 수 있다. 가입 성공 시 JWT 토큰을 포함한 User 객체를 반환한다. | P0 |
| F-AUTH-02 | 이메일·비밀번호로 로그인할 수 있으며, 성공 시 JWT 토큰을 발급한다. | P0 |
| F-AUTH-03 | 인증은 `Authorization: Token <jwt>` 헤더로 수행한다. | P0 |
| F-AUTH-04 | 현재 로그인한 사용자 정보를 조회할 수 있다. | P0 |
| F-AUTH-05 | 이메일·사용자명·비밀번호·이미지·소개(bio)를 수정할 수 있다. 전달된 필드만 갱신한다. | P0 |
| F-AUTH-06 | 설정 페이지의 로그아웃 버튼으로 로그아웃할 수 있다(클라이언트 토큰 삭제). | P0 |
| F-AUTH-07 | 이메일·사용자명은 중복될 수 없으며, 비밀번호는 해시(bcrypt 등)로 저장한다. | P0 |
| F-AUTH-08 | 사용자 삭제 기능은 제공하지 않는다. | — |

### 3.2 프로필 및 팔로우 (Profiles & Follow)
| ID | 요구사항 | 우선순위 |
| --- | --- | --- |
| F-PROF-01 | 사용자명으로 프로필(username, bio, image, following)을 조회할 수 있다. 비로그인도 가능. | P0 |
| F-PROF-02 | 로그인 사용자는 다른 사용자를 팔로우/언팔로우할 수 있다. | P0 |
| F-PROF-03 | `following` 값은 현재 로그인 사용자 기준으로 계산되며, 비로그인 시 `false`이다. | P0 |

### 3.3 게시글 (Articles)
| ID | 요구사항 | 우선순위 |
| --- | --- | --- |
| F-ART-01 | 로그인 사용자는 제목·설명·본문(마크다운)·태그 목록으로 글을 작성할 수 있다. 제목·설명·본문은 필수, 태그는 선택. | P0 |
| F-ART-02 | 글은 제목에서 생성한 **slug**(URL 식별자)로 식별하며 고유해야 한다. | P0 |
| F-ART-03 | slug로 글 상세를 조회할 수 있다. 본문은 마크다운으로 렌더링한다. | P0 |
| F-ART-04 | 작성자만 글의 제목·설명·본문을 수정할 수 있다. 제목이 변경되면 slug도 갱신한다. | P0 |
| F-ART-05 | 작성자만 글을 삭제할 수 있다. 권한이 없으면 403을 반환한다. | P0 |
| F-ART-06 | 글 목록을 **최신순**으로 조회하며, `tag`, `author`, `favorited`(사용자명) 필터와 `limit`(기본 20), `offset`(기본 0) 페이지네이션을 지원한다. | P0 |
| F-ART-07 | 로그인 사용자는 팔로우한 작성자들의 글만 모은 **피드**를 최신순으로 조회할 수 있다(`limit`/`offset` 지원). | P0 |
| F-ART-08 | 목록 응답에는 `articlesCount`(전체 개수)를 포함하고, 성능을 위해 **본문(body)은 제외**한다. | P0 |

### 3.4 좋아요 (Favorites)
| ID | 요구사항 | 우선순위 |
| --- | --- | --- |
| F-FAV-01 | 로그인 사용자는 글에 좋아요를 누르거나 취소할 수 있다. | P0 |
| F-FAV-02 | 글 응답에는 `favorited`(현재 사용자 기준)와 `favoritesCount`를 포함한다. | P0 |

### 3.5 댓글 (Comments)
| ID | 요구사항 | 우선순위 |
| --- | --- | --- |
| F-CMT-01 | 로그인 사용자는 글에 댓글을 작성할 수 있다. | P0 |
| F-CMT-02 | 누구나 글의 댓글 목록을 조회할 수 있다. | P0 |
| F-CMT-03 | 댓글 작성자만 해당 댓글을 삭제할 수 있다. | P0 |

### 3.6 태그 (Tags)
| ID | 요구사항 | 우선순위 |
| --- | --- | --- |
| F-TAG-01 | 사용 중인 태그 목록을 조회할 수 있다(홈 사이드바 "Popular Tags"). | P0 |
| F-TAG-02 | 태그를 클릭하면 해당 태그로 필터링된 글 목록 탭이 열린다. | P0 |

---

## 4. 백엔드 API 명세

### 4.1 공통 규칙
- **Base Path**: `/api`
- **Content-Type**: `application/json; charset=utf-8`
- **인증 헤더**: `Authorization: Token jwt.token.here`
- 날짜는 ISO 8601 형식(`2016-02-18T03:22:56.637Z`)
- **CORS**: 프론트엔드와 백엔드의 호스트/포트가 다르므로 `OPTIONS` 프리플라이트 요청을 처리하고 `Access-Control-Allow-Origin`, `Access-Control-Allow-Headers`(`Content-Type`, `Authorization`) 헤더를 반환한다.

### 4.2 엔드포인트 목록
| # | 메서드 | 경로 | 인증 | 요청 본문 | 응답 |
| --- | --- | --- | --- | --- | --- |
| 1 | POST | `/api/users/login` | 불필요 | `{ "user": { "email", "password" } }` | User |
| 2 | POST | `/api/users` | 불필요 | `{ "user": { "username", "email", "password" } }` | User |
| 3 | GET | `/api/user` | 필수 | — | User |
| 4 | PUT | `/api/user` | 필수 | `{ "user": { "email"?, "username"?, "password"?, "image"?, "bio"? } }` | User |
| 5 | GET | `/api/profiles/:username` | 선택 | — | Profile |
| 6 | POST | `/api/profiles/:username/follow` | 필수 | — | Profile |
| 7 | DELETE | `/api/profiles/:username/follow` | 필수 | — | Profile |
| 8 | GET | `/api/articles` | 선택 | 쿼리: `tag`, `author`, `favorited`, `limit`, `offset` | Multiple Articles |
| 9 | GET | `/api/articles/feed` | 필수 | 쿼리: `limit`, `offset` | Multiple Articles |
| 10 | GET | `/api/articles/:slug` | 불필요 | — | Single Article |
| 11 | POST | `/api/articles` | 필수 | `{ "article": { "title", "description", "body", "tagList"? } }` | Single Article |
| 12 | PUT | `/api/articles/:slug` | 필수 | `{ "article": { "title"?, "description"?, "body"? } }` | Single Article |
| 13 | DELETE | `/api/articles/:slug` | 필수 | — | — |
| 14 | POST | `/api/articles/:slug/comments` | 필수 | `{ "comment": { "body" } }` | Single Comment |
| 15 | GET | `/api/articles/:slug/comments` | 선택 | — | Multiple Comments |
| 16 | DELETE | `/api/articles/:slug/comments/:id` | 필수 | — | — |
| 17 | POST | `/api/articles/:slug/favorite` | 필수 | — | Single Article |
| 18 | DELETE | `/api/articles/:slug/favorite` | 필수 | — | Single Article |
| 19 | GET | `/api/tags` | 불필요 | — | List of Tags |

> "선택" 인증: 토큰이 없어도 동작하지만, 토큰이 있으면 `following`/`favorited` 값을 현재 사용자 기준으로 계산한다.

### 4.3 응답 포맷

**User** (인증 관련 응답)
```json
{
  "user": {
    "email": "jake@jake.jake",
    "token": "jwt.token.here",
    "username": "jake",
    "bio": "I work at statefarm",
    "image": null
  }
}
```

**Profile**
```json
{
  "profile": {
    "username": "jake",
    "bio": "I work at statefarm",
    "image": "https://api.realworld.io/images/smiley-cyrus.jpg",
    "following": false
  }
}
```

**Single Article**
```json
{
  "article": {
    "slug": "how-to-train-your-dragon",
    "title": "How to train your dragon",
    "description": "Ever wonder how?",
    "body": "It takes a Jacobian",
    "tagList": ["dragons", "training"],
    "createdAt": "2016-02-18T03:22:56.637Z",
    "updatedAt": "2016-02-18T03:48:35.824Z",
    "favorited": false,
    "favoritesCount": 0,
    "author": {
      "username": "jake",
      "bio": "I work at statefarm",
      "image": "https://i.stack.imgur.com/xHWG8.jpg",
      "following": false
    }
  }
}
```

**Multiple Articles** (목록에서는 `body` 제외)
```json
{
  "articles": [
    {
      "slug": "how-to-train-your-dragon",
      "title": "How to train your dragon",
      "description": "Ever wonder how?",
      "tagList": ["dragons", "training"],
      "createdAt": "2016-02-18T03:22:56.637Z",
      "updatedAt": "2016-02-18T03:48:35.824Z",
      "favorited": false,
      "favoritesCount": 0,
      "author": {
        "username": "jake",
        "bio": "I work at statefarm",
        "image": "https://i.stack.imgur.com/xHWG8.jpg",
        "following": false
      }
    }
  ],
  "articlesCount": 1
}
```

**Single Comment**
```json
{
  "comment": {
    "id": 1,
    "createdAt": "2016-02-18T03:22:56.637Z",
    "updatedAt": "2016-02-18T03:22:56.637Z",
    "body": "It takes a Jacobian",
    "author": {
      "username": "jake",
      "bio": "I work at statefarm",
      "image": "https://i.stack.imgur.com/xHWG8.jpg",
      "following": false
    }
  }
}
```

**Multiple Comments**
```json
{
  "comments": [
    {
      "id": 1,
      "createdAt": "2016-02-18T03:22:56.637Z",
      "updatedAt": "2016-02-18T03:22:56.637Z",
      "body": "It takes a Jacobian",
      "author": {
        "username": "jake",
        "bio": "I work at statefarm",
        "image": "https://i.stack.imgur.com/xHWG8.jpg",
        "following": false
      }
    }
  ]
}
```

**List of Tags**
```json
{
  "tags": ["reactjs", "angularjs"]
}
```

### 4.4 에러 처리
| 상태 코드 | 상황 |
| --- | --- |
| **422** Unprocessable Entity | 요청 검증 실패 (필수값 누락, 중복 이메일/사용자명 등) |
| **401** Unauthorized | 인증이 필요한데 토큰이 없거나 유효하지 않음 |
| **403** Forbidden | 인증은 되었으나 권한 없음 (타인의 글/댓글 수정·삭제) |
| **404** Not Found | 리소스를 찾을 수 없음 (존재하지 않는 slug, 사용자, 댓글) |

검증 에러 응답 형식:
```json
{
  "errors": {
    "body": ["can't be empty"]
  }
}
```
`errors`의 키는 문제가 된 필드명, 값은 메시지 배열이다.

---

## 5. 데이터 모델

```
users ──< articles ──< comments
  │  │        │  └──< article_tags >── tags
  │  │        └──< favorites >── users
  │  └──< follows (follower_id, following_id) >── users
```

| 테이블 | 주요 컬럼 | 제약 |
| --- | --- | --- |
| `users` | id, email, username, password_hash, bio, image, created_at, updated_at | email·username UNIQUE |
| `articles` | id, slug, title, description, body, author_id, created_at, updated_at | slug UNIQUE, author_id FK |
| `tags` | id, name | name UNIQUE |
| `article_tags` | article_id, tag_id | PK(article_id, tag_id) |
| `favorites` | user_id, article_id | PK(user_id, article_id) |
| `follows` | follower_id, following_id | PK(follower_id, following_id), 자기 자신 팔로우 불가 |
| `comments` | id, body, article_id, author_id, created_at, updated_at | article 삭제 시 CASCADE |

---

## 6. 프론트엔드 요구사항

### 6.1 라우팅
| 경로 | 페이지 | 인증 | 설명 |
| --- | --- | --- | --- |
| `/` | 홈 | 선택 | 태그 목록 + Your Feed / Global Feed / 태그 피드 탭, 페이지네이션 |
| `/login` | 로그인 | 비로그인 전용 | 이메일·비밀번호 폼 |
| `/register` | 회원가입 | 비로그인 전용 | 사용자명·이메일·비밀번호 폼 |
| `/settings` | 설정 | 필수 | 프로필 수정 폼 + 로그아웃 버튼 |
| `/editor` | 글 작성 | 필수 | 새 글 작성 |
| `/editor/:slug` | 글 수정 | 필수(작성자) | 기존 글 수정 |
| `/article/:slug` | 글 상세 | 선택 | 마크다운 렌더링된 본문, 댓글, 작성자 전용 수정/삭제 버튼 |
| `/profile/:username` | 프로필 | 선택 | 사용자 정보 + 작성한 글 목록 |
| `/profile/:username/favorites` | 프로필(좋아요) | 선택 | 사용자 정보 + 좋아요한 글 목록 |

> 공식 사양은 해시 라우팅(`/#/`)을 예시로 들지만, 본 프로젝트는 TanStack Router의 브라우저 라우팅을 사용한다. 라우트는 코드로 명시적으로 정의하며, 파일 기반 라우팅(`$param.tsx` 형식)은 사용하지 않는다.

### 6.2 페이지별 UI 요구사항

**공통 레이아웃**
- **헤더**: 비로그인 → Home / Sign in / Sign up. 로그인 → Home / New Article / Settings / 사용자 아바타+이름(프로필 링크). 현재 페이지 링크는 active 스타일.
- **푸터**: "conduit" 로고와 출처 문구.

**홈 페이지**
- 배너: "conduit" 로고와 "A place to share your knowledge." 문구 (비로그인 시 노출)
- 피드 탭: Your Feed(로그인 시), Global Feed, 선택한 태그 탭(`# 태그명`)
- 글 미리보기: 작성자 아바타·이름, 작성일, 좋아요 버튼+개수, 제목, 설명, "Read more...", 태그 목록
- 사이드바: Popular Tags (태그 pill 클릭 시 필터링)
- 하단 페이지네이션 (페이지당 10개 권장)

**로그인 / 회원가입**
- 가운데 정렬 폼, 상단에 서버 검증 에러 목록 표시
- "Need an account?" / "Have an account?" 전환 링크
- 성공 시 토큰 저장 후 홈으로 이동

**설정 페이지**
- 프로필 이미지 URL, 사용자명, 소개, 이메일, 새 비밀번호 입력 필드
- "Update Settings" 버튼, 에러 목록
- 폼 하단 "Or click here to logout." 로그아웃 버튼

**글 작성/수정 페이지**
- 제목, 설명("What's this article about?"), 본문(마크다운), 태그 입력
- 태그는 Enter로 추가, pill 형태로 표시되며 X 아이콘으로 제거
- "Publish Article" 버튼, 에러 목록
- 성공 시 해당 글 상세 페이지로 이동

**글 상세 페이지**
- 배너: 제목, 작성자 메타(아바타·이름·작성일)
- 액션 버튼: 작성자가 아니면 Follow / Favorite, 작성자면 Edit / Delete Article
- 본문 마크다운 렌더링, 태그 목록
- 본문 아래 동일한 액션 영역 반복
- 댓글 작성 폼(로그인 시) / "Sign in or sign up to add comments" 안내(비로그인 시)
- 댓글 목록: 작성자 정보, 작성일, 본인 댓글에 삭제(휴지통) 아이콘

**프로필 페이지**
- 아바타, 사용자명, 소개
- 본인 → "Edit Profile Settings" 버튼 / 타인 → Follow/Unfollow 버튼
- 탭: My Articles / Favorited Articles, 글 미리보기 + 페이지네이션

### 6.3 상태 및 인증 처리
- JWT 토큰은 `localStorage`에 저장하고, 앱 시작 시 `GET /api/user`로 세션을 복원한다.
- 모든 인증 요청에 `Authorization: Token <jwt>` 헤더를 자동 첨부한다.
- 401 응답 시 토큰을 삭제하고 로그인 페이지로 이동한다.
- 인증이 필요한 페이지(설정, 에디터)는 비로그인 시 로그인 페이지로 리다이렉트한다.
- 좋아요/팔로우 등 비로그인 사용자의 인증 필요 액션은 회원가입(또는 로그인) 페이지로 유도한다.

---

## 7. 비기능 요구사항

| 구분 | 요구사항 |
| --- | --- |
| **단순성** | 초기 스타트업 MVP 수준 — 기능적으로 완전하고 안정적이되 과도하게 설계하지 않는다. 새 개발자가 10분 안에 구조를 이해할 수 있어야 한다. |
| **호환성** | 백엔드 API는 공식 사양과 호환되어 다른 RealWorld 프론트엔드와 연동 가능해야 한다. |
| **보안** | 비밀번호 해시 저장, JWT 서명 검증, SQL 파라미터 바인딩(SQL Injection 방지), 권한 검사(403). 렌더링된 마크다운은 XSS 방지를 위해 sanitize 한다. |
| **성능** | 목록 API는 body 제외 + 페이지네이션. 목록 조회 시 N+1 쿼리를 피한다. |
| **반응형** | 모바일·태블릿·데스크톱에서 사용 가능해야 한다. |
| **테스트** | 최소 1개 이상의 단위 테스트(공식 기대사항). 권장: 핵심 핸들러 테스트 + 공식 Postman/API 테스트 컬렉션 통과. |
| **문서화** | README에 설치·실행 방법을 명시한다. |
| **의존성** | 사용하는 프레임워크/라이브러리는 GitHub 스타 300개 이상의 검증된 것을 사용한다. |

---

## 8. 기술 스택

### 8.1 선정 원칙
기술 스택은 Armin Ronacher의 [Agentic Coding Recommendations](https://lucumr.pocoo.org/2025/6/12/agentic-coding/)를 기준으로 정한다. 핵심은 **AI 에이전트(Claude Code)가 읽고, 실행하고, 고치기 쉬운 스택**이다.

- **단순함**: "에이전트가 동작하는 가장 단순한 방법(the dumbest possible thing that will work)을 하게 하라."
- **마법 최소화**: 암묵적 동작(fixture, 복잡한 async, ORM, 파일 이름 규칙 기반 라우팅)보다 명시적인 코드를 선호한다.
- **빠른 피드백 루프**: 컴파일·테스트·도구 응답이 빨라야 에이전트의 생산성이 올라간다.
- **의존성보다 코드**: 라이브러리를 늘리기보다 코드를 생성해 저장소에 두는 쪽을 택한다.
- **관찰 가능성**: 에이전트가 스스로 문제를 진단할 수 있도록 로그를 파일로 남긴다.

### 8.2 기술 스택
| 영역 | 기술 | 선정 이유 |
| --- | --- | --- |
| 백엔드 언어 | **Go 1.22+** | context가 실행 경로를 따라 명시적으로 전달되고, 테스트가 빠르고 증분 실행되며, 구조적 인터페이스가 LLM이 이해하기 쉽다. 생태계가 하위 호환성을 중시한다. |
| HTTP 서버 | **`net/http` 표준 라이브러리** + 커스텀 미들웨어 | 외부 프레임워크 없이 Go 1.22 패턴 라우팅(`GET /api/articles/{slug}`)으로 충분하다. |
| 데이터베이스 | **SQLite** + `database/sql` | 별도 DB 서버 없이 파일 하나로 동작하는 가장 단순한 선택. |
| DB 접근 | **순수 SQL (ORM 미사용)** | 에이전트는 좋은 SQL을 작성하고, 작성한 SQL과 실제 로그를 대조할 수 있다. |
| 인증 | JWT (`Authorization: Token <jwt>`) | RealWorld 사양 요구사항. |
| 로깅 | Go `log/slog` → **`logs/*.log` 파일** | 에이전트가 로그 파일을 읽어 스스로 원인을 찾을 수 있다. |
| 프론트엔드 | **React 18+ / TypeScript** | 글에서 권장하는 프론트엔드 조합. |
| 라우팅 | **TanStack Router (코드 기반 라우트 정의)** | 파일 기반 라우팅의 `$param.tsx` 같은 파일명은 에이전트가 자주 잘못 다루므로 라우트를 코드로 명시한다. |
| 서버 상태 | **TanStack Query** (+ 인증 상태만 최소 Context) | 서버 데이터 캐싱·갱신을 표준화하고, 전역 상태를 최소화한다. |
| 스타일/UI | **Tailwind CSS + shadcn/ui** | Tailwind는 글에서 "문제없음"으로 평가. shadcn/ui는 패키지가 아니라 컴포넌트 코드를 저장소에 생성하는 방식이라 "의존성보다 코드" 원칙에 부합. 공식 템플릿의 Bootstrap 테마 CSS는 사용하지 않고 동일한 레이아웃을 Tailwind로 재현한다. |
| 빌드 | **Vite** | 빠른 개발 서버와 HMR. |
| 실행 환경 | **Docker 컨테이너 + Docker Compose** | 모든 개발·실행은 컨테이너에서 이루어진다. 환경 차이를 없애고, 서비스당 컨테이너 1개로 중복 실행을 방지한다. |
| 작업 도구 | **Makefile** | 에이전트와 사람이 같은 명령(`make dev`, `make test` 등)을 사용한다. |
| 테스트 | Go 표준 `testing`, Vitest, RealWorld 공식 API 테스트(Postman/Newman) | 표준 도구로 빠르게 실행. |
| 린터 | golangci-lint, ESLint | 코드 품질 기본선. |
| AI 도구 | Claude Code | 바이브 코딩 도구. |

### 8.3 실행 환경 (Docker Compose)
| 서비스 | 내용 | 포트 |
| --- | --- | --- |
| `backend` | Go API 서버. SQLite DB 파일은 볼륨(`data/`)에 저장해 컨테이너를 다시 띄워도 유지 | 8080 |
| `frontend` | Vite 개발 서버 (HMR) | 3000 |

- 두 서비스 모두 공유 볼륨 `logs/`에 로그 파일(`logs/backend.log`, `logs/frontend.log`)을 남긴다.
- 소스 코드는 볼륨으로 마운트하여 코드 변경이 컨테이너에 즉시 반영되게 한다(백엔드 자동 재시작, 프론트엔드 HMR).

**Makefile 주요 타깃**
| 명령 | 동작 |
| --- | --- |
| `make dev` | `docker compose up`으로 전체 서비스 기동 |
| `make down` | 전체 서비스 중지 |
| `make logs` | `logs/` 파일 로그 확인 |
| `make test` | 컨테이너 안에서 백엔드·프론트엔드 테스트 실행 |
| `make migrate` | DB 마이그레이션 적용 |
| `make lint` / `make fmt` | 린트 / 포맷 |

### 8.4 에이전트 친화 개발 원칙
1. **가장 단순한 구현을 먼저**: 추상화·디자인 패턴보다 동작하는 직관적인 코드를 우선한다.
2. **순수 SQL**: 쿼리는 SQL 문자열로 명시적으로 작성하고, 파라미터 바인딩을 사용한다.
3. **길고 설명적인 함수명**: `getArticleBySlugWithAuthor`처럼 이름만 보고 역할을 알 수 있게 한다. 클래스·상속 대신 평범한 함수를 쓴다.
4. **권한 검사는 핸들러 안에서 명시적으로**: "작성자만 수정 가능" 같은 검사를 다른 파일에 숨기지 않는다. 숨겨진 권한 검사는 에이전트가 빠뜨리기 쉽다.
5. **항상 파일로 로깅**: 요청·에러·SQL 오류를 `logs/`에 기록한다.
6. **빠르고 명확한 도구 응답**: Makefile 명령은 빠르게 끝나고, 실패 시 원인을 분명히 출력한다. 멈추거나 대기하는 명령을 만들지 않는다.
7. **의존성 최소화**: 표준 라이브러리로 충분하면 외부 라이브러리를 추가하지 않는다.
8. **리팩터링 시점**: 너무 이르지도, 너무 늦지도 않게 — 중복이 명확해지고 코드가 에이전트에게 복잡해지기 시작할 때 정리한다.

---

## 9. 마일스톤

| 단계 | 범위 | 완료 기준 |
| --- | --- | --- |
| **M1. 프로젝트 설정** | 폴더 구조, Makefile, Docker Compose, DB 마이그레이션 | `make dev`로 빈 백엔드/프론트엔드 실행 |
| **M2. 백엔드 MVP** | 인증·사용자 → 프로필/팔로우 → 게시글/좋아요/태그 → 댓글, CORS·에러 처리 | 19개 엔드포인트 구현, 공식 API 테스트 통과 |
| **M3. 프론트엔드 MVP** | 레이아웃, 인증 페이지, 홈, 글 상세, 에디터, 프로필, 설정 | 모든 라우트 동작, 사용자 스토리 US-01~11 수행 가능 |
| **M4. 통합 및 테스트** | 프론트-백엔드 연동, E2E 점검, 문서 정리 | 수용 기준(10장) 전체 충족 |

세부 작업은 `docs/tasks.md`에서 관리한다.

---

## 10. 수용 기준 (Acceptance Criteria)

- [ ] 회원가입·로그인 후 JWT가 발급되고, 새로고침 후에도 로그인 상태가 유지된다.
- [ ] 설정 페이지에서 프로필을 수정하면 헤더·프로필 페이지에 즉시 반영되고, 로그아웃할 수 있다.
- [ ] 글을 작성·수정·삭제할 수 있으며, 타인의 글은 수정·삭제할 수 없다(UI 미노출 + API 403).
- [ ] 글 본문이 마크다운으로 렌더링된다.
- [ ] 홈에서 Global Feed / Your Feed / 태그 피드가 최신순·페이지네이션으로 동작한다.
- [ ] 좋아요/취소 시 개수와 상태가 즉시 갱신된다.
- [ ] 팔로우/언팔로우가 동작하고, Your Feed에 팔로우한 사용자의 글만 표시된다.
- [ ] 댓글 작성·조회·삭제(본인 댓글만)가 동작한다.
- [ ] 프로필 페이지에서 작성한 글 / 좋아요한 글 탭이 동작한다.
- [ ] 검증 실패 시 422와 `{ "errors": { ... } }` 형식으로 응답하고, 프론트엔드가 에러 목록을 표시한다.
- [ ] 401 / 403 / 404 상태 코드가 사양대로 반환된다.
- [ ] 공식 RealWorld API 테스트 컬렉션을 통과한다.
- [ ] 단위 테스트가 1개 이상 존재하고 `make test`로 실행된다.

---

## 11. 참고 자료

- [RealWorld 소개](https://realworld-docs.netlify.app/implementation-creation/introduction/)
- [기능 목록](https://realworld-docs.netlify.app/implementation-creation/features/)
- [구현 기대사항](https://realworld-docs.netlify.app/implementation-creation/expectations/)
- [백엔드 엔드포인트](https://realworld-docs.netlify.app/specifications/backend/endpoints/)
- [API 응답 포맷](https://realworld-docs.netlify.app/specifications/backend/api-response-format/)
- [에러 처리](https://realworld-docs.netlify.app/specifications/backend/error-handling/)
- [CORS](https://realworld-docs.netlify.app/specifications/backend/cors/)
- [프론트엔드 템플릿](https://realworld-docs.netlify.app/specifications/frontend/templates/)
- [프론트엔드 라우팅](https://realworld-docs.netlify.app/specifications/frontend/routing/)
- [RealWorld GitHub](https://github.com/gothinkster/realworld) / [라이브 데모](https://demo.realworld.io/)
