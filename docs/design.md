# Community 디자인 분석

- 원본: [Figma – Community](https://www.figma.com/design/7e00uiPVBah6jYrxrCavAb/Community?node-id=0-1) (Page 1)
- 분석 방법: Figma MCP (메타데이터, 변수, 스크린샷)
- 기준 해상도: **390 × 844** (iPhone 12/13/14 계열 모바일 웹)

> 참고: Figma Starter 플랜 MCP 호출 한도에 걸려 일부 화면(Profile/[id], Feed/Write 등)은 스크린샷 없이 레이어 구조만 보고 분석했다. radius, 일부 색상처럼 정확한 값이 Figma 변수로 정의되지 않은 항목은 스크린샷을 보고 추정했으며 `(추정)`으로 표시했다.

---

## 1. 파일 구성

Page 1은 네 개 영역으로 나뉜다.

| 영역 | 내용 |
|---|---|
| 색상 | Orange / Red / Gray 팔레트 |
| 이미지 | `logo`(112×112), `default-avatar` |
| 컴포넌트 | Navigation, Button, Input, Feed, Setting 섹션 |
| 스크린 | 실제 화면 13종 + Flow Start 설명 카드 4개(플로우 4개) |

---

## 2. 화면 구조

### 2.1 플로우와 화면 목록

| 플로우 | 화면 (Frame) | 주요 구성 |
|---|---|---|
| **인증** | `SplashScreen` | 가운데 로고만 표시 |
| | `AuthHome` | Header("로그인") · 로고 · `Filled/Large` 버튼 "로그인하기" · 텍스트 링크 "이메일로 가입하기" |
| | `Login` | Header · 이메일/비밀번호 `InputField` ×2 · `FixedBottomCTA` |
| | `Signup` | Header · 이메일/비밀번호/비밀번호 확인 `InputField` ×3 · `FixedBottomCTA` |
| **피드** | `Home` | 상단 바(아바타 + 검색 입력) · `FeedItem` 목록 · `WriteButton`(FAB) · `BottomTab` |
| | `Feed/Search` | 뒤로가기 + `SearchInput`("글 제목 검색") · 피드 목록 · 키보드 |
| | `Feed/Detail` | Header(chevron-left) · `FeedItem` · `CommentHeader`("댓글 N개") · 댓글 목록 · 댓글 입력(`Enroll`, "등록" 버튼) |
| | `Feed/Write` | Header + 우측 "저장" 텍스트 버튼 · 제목 Input · 본문 MultiLine Input · 사진 썸네일 · `FeedWrite` Footer(카메라) · 키보드 |
| **프로필** | `My` | 커버 이미지(154h) · 프로필 아바타(152) · `ProfileDetail`(닉네임/소개) · "프로필 편집" Outlined 버튼 · `Tabs`(게시물/좋아한 게시물) · 피드 · `BottomTab` |
| | `Profile/[id]` | 다른 사용자 프로필. 투명 `CustomHeader` · 커버 · 아바타 · ProfileDetail · Tabs · 피드 |
| | `Profile/Update` | Header · 아바타 + "사진 변경" Outlined 버튼 · 닉네임/소개 `Filled/Label` Input · `FixedBottomCTA` |
| **설정** | `Setting` | Header · `ListItem`("로그아웃") · `BottomTab` |

### 2.2 공통 레이아웃 골격

```
┌──────────────────────────┐
│ StatusBar          44px  │
│ Header          56~59px  │  ← 뒤로가기 / 제목 / 우측 액션
├──────────────────────────┤
│                          │
│ Content (스크롤)         │  ← 배경 #E2E8F0 위에 흰 카드 블록 쌓기
│                          │
│                  (FAB)   │  ← WriteButton 64px, 우측 16px / 탭바 위 16px
├──────────────────────────┤
│ BottomTab          82px  │  ← 홈 / 내 프로필 / 설정 (탭 루트 화면)
│   또는 FixedBottomCTA 90px (폼 화면)                    │
│ SafeArea/Bottom    34px  │
└──────────────────────────┘
```

- **탭 루트 화면**(Home, My, Setting)에는 `BottomTab`이 있다.
- **폼 화면**(Login, Signup, Profile/Update)은 하단에 `FixedBottomCTA`를 고정한다.
- **상세 화면**(Feed/Detail, Profile/[id])에는 탭바가 없고 뒤로가기 헤더를 쓴다.

---

## 3. 주요 컴포넌트

### 3.1 Navigation

| 컴포넌트 | 크기 | 설명 |
|---|---|---|
| `StatusBar` | 390×44 | iOS 상태바 |
| `Header / Default` | 390×59 | 가운데 제목만 있음, 흰 배경 |
| `Header / TwoButton` | 390×59 | 좌측 chevron-left · 가운데 제목 · 우측 아이콘(info) |
| `CustomHeader` | 390×56 | 배경 투명, 좌측 ← · 우측 ⋯ (흰색 아이콘 버전, 검정 아이콘 버전) — 커버 이미지 위에 겹쳐 쓴다 |
| `BottomTab` | 390×82 | 3개 탭(홈 / 내 프로필 / 설정), 아이콘 24 + 라벨 12px. 선택된 탭은 Orange 600 |
| `Tabs` (Two/Three/Four) | 390×38 | 텍스트 탭, 선택된 탭은 Bold + 2px 검정 밑줄, 하단 1px 구분선 |
| `SafeArea/Bottom` | 390×34 | 홈 인디케이터 |

### 3.2 Button

| 종류 | Small | Medium | Large |
|---|---|---|---|
| **Standard** (텍스트) | 23×20 | 39×20 | – |
| **Filled** | 40×28 | 50×38 | 326×44 |
| **Outlined** | 40×28 | 50×38 (실사용 98×38) | – |

- 상태: Default(Orange 600) / **Disabled**(Gray 300 배경, 흰 글자. Outlined는 회색 테두리와 회색 글자)
- 글자: Bold 흰색(Filled), Orange 600(Standard/Outlined)
- radius: Small 약 4px, Medium·Large 약 8px (추정)
- `FixedBottomCTA` (390×90): 좌우 16px 여백, 전체 너비 Filled 버튼(358×44) + 홈 인디케이터
- `WriteButton` (64×64): Orange 600 원형 FAB, 흰색 연필 아이콘, 그림자 있음

### 3.3 Icon (24×24, 라인 아이콘)

Like(heart) · Comment · Share · Cancel(x-circle) · Reply · Camera · Vote(archive) · Setting · Eye · User
→ ionicons outline 스타일, 선 굵기 약 1.5~2px, 색 Slate 900

### 3.4 Input

| 종류 | 크기 | 스타일 |
|---|---|---|
| **Filled / Input** | 358×44 | 배경 Gray 100 `#F6F6F6`, 테두리 없음, radius 약 8px |
| **Filled / MultiLine** | 358×85 (Write 화면 188) | 같은 스타일, 여러 줄 |
| **Filled / InputField** | 358×68 | 위에 라벨(12px) + Input |
| **Filled / InputField Error** | 358×92 | 배경 Red 100 `#FFDFDF` + 아래 에러 문구(Red 500, 12px) |
| **Standard** | 358×44 | 흰 배경 + 1px Gray 300 테두리, 아이콘(x) 버전도 있음 |
| **Outlined** (Error 계열) | 358×44 | 1px Orange/Red 테두리 + 컬러 텍스트, x 아이콘 버전도 있음 |
| **SearchInput** | 306×44 (실사용 324) | pill 형태(완전 라운드), Gray 100 배경, placeholder "글 제목 검색", 오른쪽에 돋보기 |

### 3.5 Feed

| 컴포넌트 | 크기 | 구성 |
|---|---|---|
| `FeedHeader` | 390×60 | 상단 바(아바타 45 + 검색 입력) |
| `FeedItem / Basic` | 390×231 | 작성자 프로필 · 본문 3줄 말줄임 · `Action` 바 |
| `FeedItem / SinglePicture` | 390×361 | Basic + 사진 썸네일 1장 |
| `FeedItem / MultiplePicture` | 390×361 | Basic + 썸네일 여러 장을 가로로 나열 |
| `PictureItem` | 90×90 | 라운드 썸네일 (radius 약 8px, 추정) |
| `Action` (Like/Comment/eye) | 47×24 | 아이콘 + 숫자. 좋아요를 누르면 꽉 찬 하트 + Orange 600 숫자. 세 개를 좌·중·우로 배치하고 위에 1px 구분선 |
| `FeedWrite / Footer` | 390×62 | 카메라 아이콘 바 (키보드 위) |

### 3.6 Profile

| 컴포넌트 | 크기 | 설명 |
|---|---|---|
| `Avatar / Feed` | 45×45 | 원형, 연한 테두리 |
| `CommentItemProfile` / `FeedItemProfile` | 358×46 | 아바타 · 닉네임(Bold 14) · 시간("3시간 전", Gray 500 12~14) · 우측 ⋮ 더보기 |
| `Follow True/False` | 358×46 | 위와 같은 구성에 우측 Outlined Small "버튼" |
| `ProfileDetail` | 390×112 | 닉네임(H2 28 Bold) + 소개 문구 |
| `Profile` (대형) | 152×152 | 프로필 화면 아바타 |

### 3.7 Comment

| 컴포넌트 | 크기 | 설명 |
|---|---|---|
| `CommentHeader` | 390×48 | "댓글 N개" Bold |
| `CommentInput` | 390×76 | Filled Input (흰 카드 안) |
| `Enroll` | 390×110 | MultiLine Input + 오른쪽 아래 Filled Small "등록" |
| `Default` (댓글 아이템) | 390×154 | CommentItemProfile + Gray 100 말풍선 본문 |

### 3.8 Setting

- `ListItem` (390×40): 텍스트만 있는 행, 위아래 1px 구분선 (예: "로그아웃")
- `ActionSheet` (365×134): 두 버튼("삭제": Red 500 텍스트 / "취소": 검정 텍스트), 연회색 배경, radius 약 8px

---

## 4. 색상 톤

### 4.1 팔레트 (Color 섹션)

**Orange — 브랜드/Primary**

| 토큰 | HEX | 용도 |
|---|---|---|
| orange-100 | `#FFF7F1` | 아주 옅은 배경 |
| orange-200 | `#FFDEC6` | 옅은 강조 배경 |
| orange-300 | `#FFB884` | 보조 강조 |
| **orange-600** | **`#FF6B57`** | **Primary** — Filled 버튼, FAB, 선택된 탭, 좋아요, Outlined 테두리 |

**Red — 에러/위험**

| 토큰 | HEX | 용도 |
|---|---|---|
| red-100 | `#FFDFDF` | 에러 Input 배경 |
| red-500 | `#FF5F5F` | 에러 문구, "삭제" 같은 위험 액션 |

**Gray (Slate 계열 쿨그레이)**

| 토큰 | HEX | 용도 |
|---|---|---|
| gray-100 | `#F6F6F6` | Input 배경, 댓글 말풍선 |
| gray-200 | `#E2E8F0` | 페이지 배경(카드 사이 간격), 구분선 |
| gray-300 | `#D1D5DB` | Disabled 버튼 배경, Standard Input 테두리 |
| gray-500 | `#6B7280` | 보조 텍스트(시간 등), placeholder |
| gray-600 | `#4B5563` | 보조 아이콘·텍스트 |
| gray-700 | `#374151` | 진한 보조 텍스트 |

### 4.2 Figma 변수로 정의된 색

| 변수 | 값 | 비고 |
|---|---|---|
| `Slate/900` | `#0F172A` | 주 텍스트/아이콘 |
| `slate/800` | `#1E293B` | |
| `main/dark` | `#232323` | |
| `main/white` | `#FFFFFF` | 카드/헤더 배경 |
| `Light/Label/Primary` | `#000000` | |
| `greyscale/Grey 92` | `#DFE2E8` | 구분선 |

### 4.3 톤 요약

- **흰 카드 + 쿨그레이 배경**에 **코랄 오렌지(`#FF6B57`) 하나**만 포인트로 쓰는 미니멀한 톤
- 포인트 색은 CTA, FAB, 선택 상태, 좋아요에만 쓰고 나머지는 흑백·회색으로 처리
- 에러 색(Red)은 오렌지와 비슷한 계열이라 전체 톤이 따뜻하게 통일되어 있음
- 마스코트(곰/물개 일러스트 아바타)가 들어가 친근하고 캐주얼한 커뮤니티 분위기

---

## 5. 타이포그래피

| 토큰 | 폰트 | 크기 / 굵기 / 행간 | 용도 |
|---|---|---|---|
| `headline/H2` | Poppins Bold | 28 / 700 / 100% | 프로필 닉네임 |
| `body/Small/bold` | Poppins SemiBold | 14 / 600 / 100% | 닉네임, 헤더 제목, 버튼 |
| `body/Small/regular` | Poppins Regular | 14 / 400 / 160% | 본문 |
| `body small` | IBM Plex Sans Regular | 12 / 400 | 라벨, 탭바 라벨, 에러 문구 |

- 한글은 시스템 고딕(Pretendard / Apple SD Gothic Neo 계열)으로 렌더링된다. 구현할 때는 `Pretendard`를 기본으로 쓰고 숫자·영문에 Poppins를 쓰는 조합을 권장한다.
- 헤더 제목은 16~17px Bold로 보인다 (추정).

---

## 6. 간격 · 크기 · 모서리

### 6.1 간격 시스템 (4/8 기반)

| 값 | 쓰는 곳 |
|---|---|
| **16px** | **화면 좌우 여백** (콘텐츠 너비 358 = 390 − 16×2), 입력 필드 사이 세로 간격, FAB 우측/하단 여백 |
| 32px | 넓은 CTA 좌우 여백 (AuthHome의 `Filled/Large` 326 = 390 − 32×2) |
| 12px | 피드 카드 사이 간격 (회색 배경이 띠처럼 보임) |
| 8px | 썸네일 사이, 아이콘과 텍스트 사이 |
| 24px | 섹션 사이 여백 (추정) |

### 6.2 고정 높이

| 요소 | 높이 |
|---|---|
| StatusBar | 44 |
| Header | 56 (투명) / 59 (기본) |
| Tabs | 38 |
| Input / Large 버튼 | 44 |
| InputField (라벨 포함) | 68 |
| BottomTab | 82 |
| FixedBottomCTA | 90 |
| SafeArea Bottom | 34 |
| 터치 아이콘 | 24 (탭 영역은 44 이상 권장) |

### 6.3 모서리 radius (추정)

| 요소 | radius |
|---|---|
| Small 버튼 | 4px |
| Medium/Large 버튼, Input, ActionSheet | 8px |
| 사진 썸네일 | 8px |
| SearchInput | 9999px (pill) |
| 아바타, FAB | 50% (원형) |

### 6.4 그림자 · 구분선

- 그림자는 **FAB(WriteButton)에만** 있다. 나머지는 그림자 없이 평평하다.
- 영역은 **배경색 차이(흰 카드 vs `#E2E8F0`)**와 **1px 구분선(`#DFE2E8`)**으로 나눈다.

---

## 7. 구현 시 참고 (디자인 토큰 초안)

```css
:root {
  /* brand */
  --color-primary: #FF6B57;   /* orange-600 */
  --color-orange-100: #FFF7F1;
  --color-orange-200: #FFDEC6;
  --color-orange-300: #FFB884;
  /* status */
  --color-danger: #FF5F5F;    /* red-500 */
  --color-danger-bg: #FFDFDF; /* red-100 */
  /* neutral */
  --color-gray-100: #F6F6F6;
  --color-gray-200: #E2E8F0;
  --color-gray-300: #D1D5DB;
  --color-gray-500: #6B7280;
  --color-gray-600: #4B5563;
  --color-gray-700: #374151;
  --color-text: #0F172A;      /* Slate/900 */
  --color-divider: #DFE2E8;
  --color-surface: #FFFFFF;
  --color-bg: #E2E8F0;

  /* spacing */
  --space-1: 4px; --space-2: 8px; --space-3: 12px;
  --space-4: 16px; --space-6: 24px; --space-8: 32px;

  /* radius */
  --radius-sm: 4px; --radius-md: 8px; --radius-full: 9999px;

  /* layout */
  --app-max-width: 390px;
  --header-h: 56px; --tabbar-h: 82px; --input-h: 44px;
}
```

### 확인이 필요한 부분

- `AuthHome` 헤더 제목이 "로그인"으로 되어 있다. 화면 역할(인증 첫 화면)에 맞는 제목인지 확인이 필요하다.
- 컴포넌트 variant 이름이 일관되지 않다(`Stardard` 오타, `View=View2`, `Property 2=…`). 코드 컴포넌트와 연결(Code Connect)하기 전에 정리하는 것이 좋다.
- 색상 팔레트가 Figma 변수가 아니라 컴포넌트(심볼)로만 정의되어 있다. 토큰을 동기화하려면 Variables로 옮기는 것을 권장한다.
- 다크 모드 디자인은 없다.
