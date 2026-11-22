# Community 서비스 개발 계획 (Next.js + Supabase)

## Context
`docs/requirements.md`(기능/데이터/RLS 정의)와 `docs/design.md`(Figma 분석: 화면, 컴포넌트, 토큰)를 바탕으로, 아직 코드가 없는 저장소에 모바일 웹 커뮤니티를 구현하기 위한 계획이다.
**우선순위 원칙:** 기능과 범위는 requirements를 따르고, 시각 스타일(색·간격·컴포넌트 모양)은 design을 따른다.

두 문서가 다른 부분과 결정:
| 항목 | requirements | design | 결정 |
|---|---|---|---|
| 게시글 제목 | 있음(1~50자) | 피드 카드에 제목 없음 | 제목 있음. 카드 본문 위에 Bold 14로 표시 |
| 이미지 | 1장 | 여러 장 썸네일 | 1장 (다중 업로드는 MVP 제외) |
| 홈 상단 검색바 | 검색은 MVP 제외 | 검색바 있음 | 로고 헤더로 대체 |
| 프로필 탭(게시물/좋아한 게시물) | 내가 쓴 글만 | 탭 2개 | 내가 쓴 글만 (탭은 이후 확장) |
| 탭 라벨 | 내정보 | 내 프로필 | requirements 경로 `/me` 사용, 라벨은 "내정보" |

---

## 1. 폴더 구조

```
community/
├─ docs/                          # requirements.md, design.md
├─ supabase/
│  ├─ config.toml
│  ├─ migrations/
│  │  ├─ 0001_tables.sql          # profiles, posts, comments, likes + 인덱스
│  │  ├─ 0002_functions_triggers.sql
│  │  ├─ 0003_rls.sql
│  │  └─ 0004_storage.sql         # 버킷 + storage.objects 정책
│  └─ seed.sql                    # 로컬 테스트 데이터
├─ public/                        # logo.svg, default-avatar.png, 아이콘
├─ src/
│  ├─ middleware.ts               # 세션 갱신 + 접근 제어 (Next 16이면 proxy.ts)
│  ├─ app/
│  │  ├─ layout.tsx               # 폰트, 480px 중앙 컨테이너, Toaster
│  │  ├─ globals.css              # design.md §7 토큰 → Tailwind @theme
│  │  ├─ not-found.tsx / error.tsx
│  │  ├─ (tabs)/                  # 하단 탭이 보이는 화면
│  │  │  ├─ layout.tsx            # <BottomTabBar/>
│  │  │  ├─ page.tsx              # /  홈 피드
│  │  │  ├─ me/page.tsx           # /me
│  │  │  └─ settings/page.tsx     # /settings
│  │  ├─ (stack)/                 # 뒤로가기 헤더를 쓰는 화면 (탭 없음)
│  │  │  ├─ posts/new/page.tsx
│  │  │  ├─ posts/[id]/page.tsx
│  │  │  ├─ posts/[id]/edit/page.tsx
│  │  │  ├─ me/edit/page.tsx
│  │  │  └─ users/[id]/page.tsx
│  │  ├─ (auth)/
│  │  │  ├─ login/page.tsx
│  │  │  └─ signup/page.tsx
│  │  └─ api/feed/route.ts        # (선택) 무한 스크롤 커서 API
│  ├─ features/                   # 도메인별 로직
│  │  ├─ auth/    actions.ts, schemas.ts, LoginForm.tsx, SignupForm.tsx
│  │  ├─ posts/   actions.ts, queries.ts, schemas.ts, PostCard.tsx, PostForm.tsx, PostList.tsx
│  │  ├─ comments/actions.ts, queries.ts, CommentItem.tsx, CommentInput.tsx
│  │  ├─ likes/   actions.ts, LikeButton.tsx
│  │  └─ profiles/actions.ts, queries.ts, ProfileHeader.tsx, ProfileForm.tsx
│  ├─ components/ui/              # design system: Button, Input, InputField, Header,
│  │                              # BottomTabBar, Tabs, Avatar, FloatingWriteButton,
│  │                              # FixedBottomCTA, ConfirmDialog/ActionSheet, Toast, Skeleton, EmptyState
│  ├─ lib/
│  │  ├─ supabase/ client.ts (browser) · server.ts (RSC/Action) · middleware.ts (updateSession)
│  │  ├─ storage.ts               # 업로드 경로 생성, 파일 검증(형식/용량)
│  │  ├─ relative-time.ts         # "3분 전"
│  │  └─ safe-redirect.ts         # redirect 쿼리 검증
│  └─ types/database.ts           # `supabase gen types` 생성 결과
├─ .env.local.example
└─ package.json
```

라이브러리: `next`, `@supabase/ssr`, `@supabase/supabase-js`, `tailwindcss`, `zod`(검증), `react-hook-form`(폼), `sonner`(토스트), `@tanstack/react-query`(무한 스크롤, 낙관적 업데이트) 또는 `useOptimistic`과 Server Action 조합.

---

## 2. 페이지 / 라우트 목록

| ID | 경로 | 그룹 | 권한 | 렌더링 | 비고 |
|---|---|---|---|---|---|
| S-01 | `/` | (tabs) | 전체 | RSC 첫 20개 + 클라이언트 무한 스크롤 | FAB → `/posts/new` |
| S-02 | `/login` | (auth) | 비회원 | Client form + Server Action | 로그인 상태면 → `/` |
| S-03 | `/signup` | (auth) | 비회원 | 〃 | 로그인 상태면 → `/` |
| S-04 | `/posts/new` | (stack) | 회원 | Client form | 이탈 시 확인(beforeunload + 뒤로가기 가로채기) |
| S-05 | `/posts/[id]/edit` | (stack) | 작성자 | RSC에서 소유자 확인 후 PostForm 재사용 | 작성자가 아니면 403/notFound |
| S-06 | `/posts/[id]` | (stack) | 전체 | RSC + 클라이언트 LikeButton/CommentInput | 없으면 안내 문구, 작성자는 ⋯ 메뉴 |
| S-07 | `/me` | (tabs) | 회원 | RSC | 글 수, 내 글 무한 스크롤 |
| S-08 | `/me/edit` | (stack) | 회원 | Client form | 아바타 미리보기, 기본 이미지로 되돌리기 |
| S-09 | `/users/[id]` | (stack) | 전체 | RSC | 본인 id면 `/me`로 redirect |
| S-10 | `/settings` | (tabs) | 전체 | RSC | 회원/비회원 화면 분기, 로그아웃 확인 |
| S-11 | `not-found`, `error` | – | 전체 | – | 홈 버튼 |

참고: `(tabs)/me`와 `(stack)/me/edit`처럼 route group이 달라도 URL은 충돌하지 않는다.

**Server Actions:** `signUp`, `signIn`, `signOut`, `createPost`, `updatePost`, `deletePost`, `createComment`, `deleteComment`, `toggleLike`, `updateProfile`, `checkNickname`

---

## 3. DB 테이블 개요
requirements §5를 그대로 따르고, 아래 사항을 보강한다.

| 테이블 | 핵심 컬럼 | 제약 / 인덱스 |
|---|---|---|
| `profiles` | id(uuid, FK auth.users), nickname, avatar_url, created_at, updated_at | nickname UNIQUE + `CHECK (nickname ~ '^[가-힣A-Za-z0-9_]{2,12}$')` |
| `posts` | id(bigint identity), author_id, title, content, image_url, like_count, comment_count, created_at, updated_at | `CHECK char_length`, 인덱스 `(created_at desc, id desc)`, `(author_id, created_at desc)` |
| `comments` | id, post_id, author_id, content, created_at | `CHECK 1~500`, 인덱스 `(post_id, created_at)` |
| `likes` | post_id, user_id, created_at | PK `(post_id, user_id)`, 인덱스 `(user_id)` |

- **트리거:** `handle_new_user`, `update_like_count`, `update_comment_count`, `set_updated_at`
- **함수(RPC, 선택):** `is_nickname_available(text)`
- **Storage:** `avatars`, `post-images` (public read, 첫 폴더 = `auth.uid()`인 경우에만 쓰기 허용)
- **피드 조회:** `posts` + `profiles!author_id(nickname, avatar_url)` 조인. 로그인 사용자는 내가 누른 좋아요를 `likes`에서 `post_id in (...)`로 한 번에 조회해 합친다.
- **페이지네이션:** 커서 `(created_at, id)`로 `lt` 조건, 20개씩

---

## 4. 구현 순서

| 단계 | 내용 | 완료 기준 |
|---|---|---|
| **0. 기반** | create-next-app(TS, App Router, Tailwind), Supabase 프로젝트와 로컬 CLI(`supabase init/start`), `.env.local`, `lib/supabase/*` 3종, middleware 세션 갱신, 타입 생성 스크립트 | 로컬에서 `/` 렌더링, 세션 쿠키 갱신 동작 |
| **1. DB** | migrations 0001~0004 작성 → `supabase db reset` → `gen types` | 테이블, 트리거, RLS, 버킷 생성 확인 |
| **2. 디자인 시스템** | 토큰(design.md §7), 480px 컨테이너, `components/ui/*` (Button 3종×3사이즈, Input 변형, Header 2종, BottomTabBar, FAB, FixedBottomCTA, Avatar, Skeleton, Toast, ConfirmDialog) | 임시 `/dev/ui` 페이지에서 Figma와 비교 |
| **3. 인증** | 회원가입(zod 검증, 닉네임 중복 확인, metadata.nickname 전달), 로그인(redirect 처리), 로그아웃(설정 탭), middleware 접근 제어 | F-01~03, F-16 |
| **4. 피드/게시글** | 홈 피드(RSC + 무한 스크롤, 빈 상태, 스켈레톤), 글 작성(이미지 업로드), 상세, 수정/삭제, 상대시간, "(수정됨)" 표시 | F-04~07 |
| **5. 댓글·좋아요** | 상세 하단 고정 댓글 입력, 목록(오래된 순), 삭제, LikeButton 낙관적 토글(상세는 필수, 피드는 권장), 비회원은 로그인 유도 | F-08~11 |
| **6. 프로필** | `/me`(글 수, 내 글), `/me/edit`(아바타 업로드/미리보기/기본 이미지 복귀, 닉네임 변경), `/users/[id]` | F-12~14 |
| **7. 공통 마감** | 404/에러, safe-area, aria-label, 44px 터치 영역, `next/image` remotePatterns(Supabase 도메인), 홈 탭 다시 누르면 최상단으로 이동 + 새로고침 | 비기능 요구사항 |
| **8. 배포** | Supabase 원격 프로젝트에 `db push`, Auth 설정(이메일 인증 OFF, Site URL), Vercel 연결과 환경변수, 프리뷰/프로덕션 확인 | 프로덕션 URL에서 E2E 시나리오 통과 |

---

## 5. 주의점

### RLS · DB
- **모든 테이블에 RLS를 켠다.** 정책은 requirements §5.4대로 작성하고, `with check`(INSERT/UPDATE)와 `using`(SELECT/UPDATE/DELETE)을 모두 지정한다.
- **카운터 컬럼 변조 방지:** posts UPDATE 정책은 작성자에게 행 전체 수정 권한을 주므로 작성자가 `like_count`를 직접 바꿀 수 있다. → `revoke update on posts from authenticated; grant update (title, content, image_url) on posts to authenticated;`처럼 컬럼 단위로 권한을 준다. `author_id`, `created_at`도 수정되지 않게 막는다.
- **카운터 트리거는 `security definer`로 만든다.** 다른 사람 글에 좋아요/댓글을 달 때 트리거가 posts를 UPDATE해야 하는데, RLS에 막히면 안 된다. `set search_path = ''`로 고정하고 스키마를 명시한다.
- **`handle_new_user`**도 `security definer`로 만든다. 닉네임 중복이나 형식 오류로 트리거가 실패하면 signUp이 "Database error saving new user"라는 일반 오류만 반환한다. → 가입 전에 `is_nickname_available`로 미리 확인하고, 그 사이 다른 사람이 같은 닉네임을 가져가는 경우에 대비해 실패 시 닉네임 중복 메시지로 바꿔 보여준다.
- **profiles INSERT 정책**은 트리거로만 생성할 경우 필요 없다. 닫아두는 편이 안전하다.
- **Storage 정책:** `(storage.foldername(name))[1] = auth.uid()::text`로 INSERT/UPDATE/DELETE를 제한한다. 파일 형식과 용량은 클라이언트 검증만 믿지 말고 버킷 설정(`allowed_mime_types`, `file_size_limit`)으로도 막는다.
- 이미지 교체나 글 삭제 시 이전 Storage 파일을 정리한다. CASCADE는 Storage까지 지우지 않는다.

### 로그인 · 세션
- `@supabase/ssr`의 `createServerClient`와 cookie `getAll/setAll` 패턴을 쓰고, **middleware에서 매 요청마다 세션을 갱신**한다.
- 서버에서 인증을 판단할 때는 **`supabase.auth.getUser()`**(또는 `getClaims()`)를 쓴다. `getSession()`은 쿠키를 검증하지 않으므로 신뢰하면 안 된다.
- **middleware는 화면 이동 편의용일 뿐이다.** 모든 Server Action 안에서 다시 `getUser()`로 확인하고, 최종 권한은 RLS가 강제한다.
- **Open redirect 방지:** `redirect` 쿼리는 `/`로 시작하고 `//`로 시작하지 않는 내부 경로만 허용한다(`lib/safe-redirect.ts`).
- 로그인 실패 메시지는 하나로 통일한다. 이메일 중복은 signUp 에러 "User already registered"를 받아 안내 문구로 바꾼다(이메일 인증 OFF 기준).
- 로그인/로그아웃 후에는 `revalidatePath('/', 'layout')`로 서버 컴포넌트 캐시를 갱신한다.
- service role 키는 사용하지 않는다(필요하면 서버 전용 환경변수로 두고 `NEXT_PUBLIC_` 접두사를 붙이지 않는다).

### 기타
- XSS: 본문은 `whitespace-pre-wrap` 텍스트로만 렌더링하고 `dangerouslySetInnerHTML`은 쓰지 않는다.
- 좋아요 연타로 인한 경쟁 상태: 토글 요청을 직렬화하거나 마지막 상태를 기준으로 처리한다. PK 중복(23505)은 성공으로 간주한다.
- 피드 캐시: 글 작성/삭제/수정 후 `revalidatePath('/')`, 상세 경로, `/me`를 갱신한다.
- Next.js 버전 확인: 16부터는 `middleware.ts` 대신 `proxy.ts`를 쓴다. 설치한 버전 문서를 기준으로 한다.

---

## 검증
1. **DB/RLS:** `supabase db reset` 후 SQL로 확인한다. anon은 SELECT만 되는지, 다른 사람 글 UPDATE/DELETE가 0행인지, 남의 `like_count`를 직접 UPDATE하면 거부되는지, 좋아요/댓글 추가·삭제 시 카운트가 맞는지 본다.
2. **Storage:** 다른 사용자 폴더에 업로드하면 거부되는지, 5MB를 넘거나 허용되지 않는 MIME 타입이면 거부되는지 본다.
3. **수동 E2E 시나리오** (requirements §3 흐름 3.1~3.7): 비회원 열람 → 글쓰기 시도 → 로그인 → redirect 복귀 → 작성 → 상세 → 댓글/좋아요 → 프로필 수정 → 다른 유저 프로필 → 로그아웃.
4. (선택) Playwright로 위 시나리오 중 인증·작성·좋아요를 자동화한다.
5. `npm run build`, `lint`, 타입 체크를 통과하고, Vercel 프리뷰에서 같은 시나리오를 다시 확인한다.
