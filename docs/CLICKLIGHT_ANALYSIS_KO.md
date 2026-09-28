# ClickLight 분석 정리 (대화 기록)

- 원본 저장소: https://github.com/aurorascharff/ClickLight
- 내 포크 저장소: https://github.com/bmshin94/ClickLight
- 공식 소개 사이트 소스: 이 저장소의 `website/` 폴더 (Next.js)
- 분석 기준 버전: v0.16.0 (`Casks/clicklight.rb`, `appcast.xml` 기준)

---

## 1. 전수조사 결과: 이게 뭐하는 건지

**ClickLight = macOS 메뉴 막대에 사는 작은 앱. 마우스를 클릭할 때마다 화면에 "빛나는 동그라미"를 띄워서, 보는 사람이 "지금 어디를 눌렀는지" 바로 알 수 있게 해 준다.**

화면 녹화 도구(Screen Studio, CleanShot 등)는 *녹화가 끝난 뒤* 클릭 효과를 붙이지만, ClickLight는 *실시간 라이브 상황*(발표, 화면 공유, 줌 회의)에서 바로 보여 주는 게 핵심이다.

### 주요 기능

| 기능 | 설명 |
| --- | --- |
| 클릭 하이라이트 | 누름(Press) / 뗌(Release) / 오른쪽 클릭 / 가운데 클릭 / 드래그를 각각 다른 모양·색으로 표시 |
| 레이저 포인터 모드 | 드래그하면 형광펜처럼 선이 그려졌다가 서서히 사라짐 |
| 화살표 모드 | 드래그한 시작점→끝점으로 화살표가 그려지고, 지울 때까지 화면에 남음 |
| 라이브 단축키 표시 | `⌘ + C` 같은 단축키를 누르면 화면 아래(또는 포인터 옆)에 크게 표시 |
| 스크린샷 처리 | `⌘ + ⇧ + 4` 스크린샷 후에는 다음 "뗌" 효과를 숨겨서 캡처에 동그라미가 안 찍히게 함 |
| 클릭 통계 | 하루 클릭 수를 기기 안에만 저장, 최근 7일 차트 + 메뉴 막대에 오늘 클릭 수 표시 가능 |
| 설정/프리셋 | 크기·강도·지속시간·색상 프리셋, 커스텀 색상, 미리보기 패드, 랜덤 색상 |
| 프로필 | Default / Workshop / Screen Recording / Presentation / Minimal 등 설정 묶음. JSON으로 가져오기·내보내기 |
| 전역 단축키 | 기본 `⌃ + ⌥ + ⌘ + L` 로 켜고 끄기, 나머지 기능도 단축키 지정 가능 |
| 자동 업데이트 | Sparkle 프레임워크로 앱 내 업데이트 |

### 어떨 때 쓰나

- 라이브 제품 데모, 온라인 강의, 워크숍, 컨퍼런스 발표
- 줌/팀즈/구글밋 화면 공유로 누군가에게 사용법을 알려줄 때
- UX 리뷰: "클릭한 순간"과 "화면이 반응한 순간" 사이의 지연을 눈으로 확인할 때 (원작자가 만든 원래 이유)
- 버그 리포트 녹화: 어떤 동작을 했는지 + 앱이 어떻게 반응했는지를 한 영상에 담을 때
- 유튜브/튜토리얼 녹화

### 폴더 구조

```
ClickLight/
├── Sources/ClickLight/         ← 앱 본체 (Swift, 약 6,000줄)
│   ├── main.swift / AppDelegate.swift    앱 시작, 모든 부품 연결
│   ├── ClickEventTap.swift               전역 마우스/키보드 감지 (CGEventTap + NSEvent 예비)
│   ├── ClickEvent.swift                  클릭 종류(ClickKind) 데이터
│   ├── OverlayCoordinator.swift          모니터마다 투명 오버레이 창 1개씩 관리, 중복 이벤트 제거
│   ├── ClickOverlayWindow.swift          투명·클릭 통과·최상단 창
│   ├── ClickOverlayView.swift            CoreGraphics로 동그라미/레이저/화살표/단축키 그리기
│   ├── SettingsStore.swift               설정값(UserDefaults) 저장
│   ├── ClickProfileStore.swift           프로필 저장, JSON 가져오기/내보내기
│   ├── ClickActivityStore.swift          일별 클릭 수 기록 (로컬 전용)
│   ├── StatusController.swift            메뉴 막대 아이콘과 메뉴
│   ├── SettingsWindowController.swift / ClickLightSettingsView.swift   설정 창 (SwiftUI)
│   ├── HotKeyManager.swift / HotKeyBinding.swift / ShortcutRecorderField.swift   전역 단축키
│   ├── PermissionController.swift        손쉬운 사용 권한 확인
│   ├── LaunchAtLoginController.swift     로그인 시 자동 실행
│   └── UpdateChecker.swift               Sparkle 자동 업데이트
├── website/                    ← 홍보 사이트 (Next.js 16 + React 19 + TypeScript)
│   ├── app/page.tsx                      브라우저에서 클릭 효과를 체험하는 인터랙티브 데모
│   └── profiles/index.ts                 앱에서 가져올 수 있는 프로필 JSON 다운로드
├── docs/                       ← 로컬 개발, 수동 설치, 릴리스 방법 문서
├── Casks/clicklight.rb         ← Homebrew 설치 정의
├── appcast.xml                 ← Sparkle 업데이트 피드
├── build-app.sh                ← Xcode 없이 .app 만드는 빌드 스크립트
├── Package.swift               ← Swift Package Manager 설정 (의존성: Sparkle 하나)
├── Info.plist / AppIcon.icon   ← 앱 정보, 아이콘
├── .github/workflows/          ← PR 빌드 테스트, 보안 점검, 태그 푸시 시 자동 서명·공증·배포
├── AGENTS.md                   ← AI 코딩 에이전트용 작업 안내서
└── CLAUDE.md                   ← 포크 후 추가한 대화 스타일 설정
```

### 동작 흐름

```
마우스 클릭 → ClickEventTap (전역 감지)
          → NotificationCenter 로 이벤트 전달
          → AppDelegate (설정 창 안 클릭은 걸러냄)
          → OverlayCoordinator (해당 모니터 찾기, 3px/0.1초 이내 중복 제거)
          → ClickOverlayView (애니메이션으로 동그라미 그림)
```

### 나한테 무슨 도움이 되나

1. **바로 쓰는 도구**: 강의·데모·화면 공유·튜토리얼 녹화 품질이 올라간다.
2. **공부 교재**: 작고 깔끔한 네이티브 macOS 앱 + Next.js 사이트 + 자동 배포 파이프라인이 한 저장소에 다 있다.
3. **개조 가능한 베이스**: MIT 라이선스라 고쳐 쓰거나 파생 제품을 만들 수 있다.

---

## 2. 더 쉽게 설명하면

- **비유**: 발표자가 들고 있는 "레이저 포인터"를 마우스에 달아 준 것.
- 컴퓨터 화면 위에 **보이지 않는 투명 유리판**을 한 장 깔아 둔다. 유리판은 클릭을 막지 않고 그대로 통과시킨다.
- 마우스를 누를 때마다 앱이 "방금 여기 눌렀어!" 신호를 받아서 유리판 위 그 자리에 **동그라미가 퐁 하고 퍼졌다가 사라지게** 그린다.
- 결과적으로 화면을 보는 사람은 커서를 눈으로 쫓지 않아도 "아, 저기 눌렀구나"를 바로 안다.
- 조건: **Mac 전용** (macOS 14 Sonoma 이상). 윈도우/리눅스에서는 동작하지 않는다.
- 설치하고 "손쉬운 사용" 권한만 주면 끝. 인터넷 계정·로그인·결제 없음.

---

## 3. 질문과 답

### 3-1. 설치 및 사용법

**방법 A — Homebrew (추천)**

```bash
brew tap aurorascharff/clicklight https://github.com/aurorascharff/ClickLight
brew install --cask aurorascharff/clicklight/clicklight
# 업데이트
brew upgrade --cask clicklight
```

**방법 B — 직접 다운로드**: 원본 저장소 GitHub Releases 에서 `ClickLight.zip` 받아서 압축 풀고 응용 프로그램 폴더로 이동.

**방법 C — 소스에서 빌드 (포크를 고쳐 쓸 때)**

```bash
xcode-select --install          # 최초 1회
git clone https://github.com/bmshin94/ClickLight.git
cd ClickLight
./build-app.sh
cp -R ClickLight.app "$HOME/Applications/ClickLight.app"
open "$HOME/Applications/ClickLight.app"
```

**처음 실행 후 필수 설정**

1. 시스템 설정 → 개인정보 보호 및 보안 → **손쉬운 사용** → ClickLight 켜기
2. (단축키 표시/스크린샷 처리까지 쓰려면) **입력 모니터링**도 켜기
3. 메뉴 막대에서 ClickLight **종료 후 다시 실행**
4. 메뉴 막대 아이콘 → **Test Pulse at Pointer** 로 동그라미 나오는지 확인

**사용 팁**

- `⌃ + ⌥ + ⌘ + L` 로 켜기/끄기
- 메뉴에서 크기·강도·색상 빠르게 변경, 자세한 건 Settings 창
- 발표용이면 "Presentation" 프로필, 녹화용이면 "Screen Recording" 프로필
- 시스템 설정 → 손쉬운 사용 → 디스플레이 → 포인터 에서 커서를 크게 키우면 더 잘 보임
- 삭제: `brew uninstall --cask --zap clicklight`

### 3-2. 플러그인? 스킬? MCP?

**셋 다 아님. 독립 실행형 macOS 네이티브 앱**이다.

| 구분 | 의미 | ClickLight |
| --- | --- | --- |
| 플러그인 | 다른 프로그램 안에 끼워 쓰는 확장 | ❌ 혼자 실행됨 |
| 스킬 | AI 에이전트가 읽는 작업 지침 묶음 | ❌ (단, `AGENTS.md` 는 AI가 이 코드를 고칠 때 읽는 안내서) |
| MCP | AI가 외부 도구를 호출하는 서버 프로토콜 | ❌ AI와 통신하는 부분 없음 |

### 3-3. API 토큰이 필요한가?

**필요 없다.** 앱은 인터넷 API를 호출하지 않는다. 네트워크를 쓰는 건 Sparkle 업데이트 확인(`appcast.xml` 읽기)뿐이다. 클릭 통계도 기기 안(UserDefaults)에만 저장된다.

단, **내가 직접 서명된 정식 버전을 배포**하려면 Apple 개발자 계정(연 $99)과 인증서, App Store Connect API 키, Sparkle 서명 키를 GitHub Secrets 에 넣어야 한다. 이건 배포자용이고 사용자는 전혀 신경 쓸 필요 없다.

### 3-4. 왜 GitHub 에서 유명할까? (추정)

1. **누구나 겪는 불편 하나를 정확히 해결** — "화면 공유할 때 내가 뭘 눌렀는지 안 보인다"
2. **무료 + 오픈소스(MIT) + 가벼움** — 유료 대안(Presentify, Mouseposé 등)이 있는 분야에서 공짜로 깔끔하게 동작
3. **설치가 쉬움** — brew 명령 두 줄
4. **완성도** — 서명·공증된 배포, 자동 업데이트, 잦은 릴리스(v0.16.0까지)
5. **체험형 웹사이트** — 브라우저에서 바로 클릭 효과를 눌러볼 수 있음
6. **"개인용 소프트웨어 + AI 에이전트로 고쳐 쓰기" 트렌드** — 코드가 작고 `AGENTS.md` 로 AI가 수정하기 쉽게 설계됨
7. **제작자 인지도** — 원작자 Aurora Scharff 는 React/Next.js 커뮤니티에서 활발히 발표·글쓰기를 하는 개발자로, 발표/데모를 많이 하는 사람들 사이에 퍼지기 좋았음

### 3-5. 로컬 에이전트 구축에 도움이 될까?

**직접적인 AI 부품은 아니지만 간접적으로 도움이 된다.**

- **컴퓨터 사용(computer-use) 에이전트 시연**: 에이전트가 화면을 자동 클릭할 때 ClickLight를 켜 두면 에이전트가 어디를 눌렀는지 사람이 눈으로 확인하기 쉽다 (녹화/디버깅/데모용). 합성 이벤트가 어느 단계에 주입되는지에 따라 감지 여부가 달라질 수 있으니 직접 확인 필요.
- **macOS 전역 입력 감지 참고 코드**: `ClickEventTap.swift` 는 사용자 행동을 관찰하는 에이전트(예: "사용자가 한 작업을 기록해서 자동화 스크립트로 바꾸기")를 만들 때 좋은 예제다.
- **화면 위 오버레이 참고 코드**: 에이전트가 "여기를 누를게요"라고 미리 표시하는 UI를 만들 때 `ClickOverlayWindow` 방식을 그대로 응용할 수 있다.
- **AI 친화적 저장소 구조 예시**: `AGENTS.md` 작성 방식 참고.

### 3-6. 수익화 아이디어 요약 → 4장에서 자세히

### 3-7. React 나 PHP 로 만들 수 있나?

| 기술 | 가능 여부 | 설명 |
| --- | --- | --- |
| React (웹) | ⭕ 부분적 | **브라우저 탭 안** 클릭만 하이라이트 가능. 웹앱 데모·온보딩·크롬 확장으로 충분히 제작 가능. 이 저장소 `website/` 가 이미 React로 만든 웹 데모다. |
| React + Electron/Tauri | ⭕ 가능 | 투명·최상단·클릭통과 창 + 전역 마우스 훅(예: uiohook 계열 네이티브 모듈)으로 **윈도우/맥 공용 데스크톱 앱** 제작 가능. 앱 용량과 CPU 사용량은 네이티브보다 큼. |
| PHP | ❌ 앱 자체는 불가 / ⭕ 서버는 가능 | PHP는 서버에서 돌기 때문에 사용자 화면의 클릭을 감지할 수 없다. 대신 **라이선스 판매, 회원·결제, 프로필 공유, 통계 대시보드 백엔드**로 적합. |

**추천 조합**: 데스크톱 앱(Swift 포크 또는 Electron+React) + 웹 판매/관리 페이지(React) + 결제·라이선스 서버(PHP 또는 Node).

---

## 4. 수익화 아이디어 (상세)

> 라이선스: MIT → 상업적 이용·수정·판매 모두 가능. 단, **원본 저작권·라이선스 고지를 유지**해야 하고, "ClickLight" 이름·아이콘을 그대로 써서 원작 공식 제품인 것처럼 보이게 하면 안 된다. 파생 제품은 새 이름·새 브랜드로 만드는 게 안전하다.

### 아이디어 1. 윈도우용 버전 (가장 현실적) ⭐

- **왜**: ClickLight는 Mac 전용. 한국은 윈도우 사용자가 압도적으로 많고, 온라인 강의·사내 교육·유튜브 강의 제작자 수요가 크다.
- **어떻게**: Electron(또는 Tauri) + React UI + 전역 마우스 훅 + 투명 오버레이 창.
- **경쟁**: MS PowerToys "마우스 유틸리티"에 무료 클릭 강조 기능이 있음 → 차별화 필요 (레이저·화살표·단축키 표시·프로필·한국어 UI·녹화 연동을 한 앱에).
- **가격 예시**: 기본 무료 + Pro 1회 결제 1~2만 원, 또는 기업/학교 라이선스.

### 아이디어 2. "Pro" 발표 도구 (Mac/Win)

- ClickLight 기능 + **화면 확대(줌), 스포트라이트(주변 어둡게), 화면에 펜 그리기, 타이머, 웹캠 원형 오버레이**를 한 번에.
- 대상: 강사, 세일즈 데모, 개발 컨퍼런스 발표자.
- 모델: 1회 결제(Gumroad, Lemon Squeezy, Paddle) 또는 연 구독.

### 아이디어 3. 웹 전용 클릭 하이라이터 (React 로 바로 가능)

- **크롬 확장 프로그램**: 웹 서비스 데모·QA 녹화 시 브라우저 안 클릭·키 입력을 표시. 설치 쉬움, 유료 Pro 기능(커스텀 스타일, 팀 공유 프로필).
- **npm 라이브러리 / 스크립트 태그**: SaaS 회사가 자기 제품 데모 페이지·온보딩 투어에 붙이는 위젯. 무료 오픈소스 + 상업용 라이선스 판매.

### 아이디어 4. 클릭 기반 "자동 매뉴얼 생성" SaaS (가장 큰 시장, 난이도 높음)

- 사용자가 작업하는 동안 **클릭 위치 + 스크린샷**을 기록 → "1단계: 여기를 클릭하세요" 형태의 **단계별 가이드 문서/인터랙티브 데모**를 자동 생성.
- 해외 유사 서비스: Scribe, Tango, Arcade, Supademo (월 구독 모델로 성업 중).
- 한국 틈새: 한국어 가이드, 사내 ERP/그룹웨어 매뉴얼, 공공기관·학교 업무 매뉴얼, AI로 설명 문장 자동 작성.
- 구조: 캡처 앱(데스크톱 또는 크롬 확장, React) + 웹 편집기(React) + 백엔드(PHP/Laravel 또는 Node) + 구독 결제.

### 아이디어 5. UX 리서치 / 사용성 테스트 도구

- ClickLight 원래 동기가 "클릭→반응 지연 확인". 이를 확장해 **클릭 로그 + 응답 시간 측정 + 히트맵 리포트**를 만들어 기획자·QA팀에 판매.
- B2B 구독, 팀 단위 가격.

### 아이디어 6. 콘텐츠·서비스형 수익

- 이 코드를 교재로 "Swift로 macOS 메뉴바 앱 만들기", "React로 크롬 확장 만들기" 강의 (인프런, 클래스101, 유튜브).
- 강의 영상 제작 대행, 기업 교육 영상 제작 시 차별화 포인트로 활용.
- 발표/녹화 용도별 **프리미엄 프로필 팩**(색상·효과 세트) 판매는 단독으로는 작지만 Pro 앱의 부가 상품으로 가능.

### 추천 순서 (작게 시작 → 확장)

1. **React 크롬 확장 클릭 하이라이터** (1~2주, 시장 반응 테스트)
2. **윈도우 데스크톱 버전** (Electron + React, 한국어 UI)
3. 사용자 모이면 **자동 매뉴얼 생성 SaaS** 로 확장 (PHP/Node 백엔드 + 구독 결제)

---

## 5. 이 문서에 대해

- 1~4번 대화 내용을 정리한 문서이며, 저장소 코드(`Sources/`, `website/`, `docs/`, `.github/`, `README.md`, `AGENTS.md` 등)를 직접 확인한 내용을 기반으로 작성했다.
- "왜 유명한가", "경쟁 제품", "시장" 관련 내용은 저장소 밖 정보를 포함한 추정이므로 실제 사업 판단 전에는 최신 정보를 따로 확인할 것.
