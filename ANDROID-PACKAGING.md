# 데스크핏 · APK / Play Store 패키징 가이드

## 0. 먼저 정할 것 — APK냐 AAB냐

| | 파일 | 용도 |
|---|---|---|
| **내 폰(또는 지인)에 직접 설치** | `.apk` | 사이드로딩. Play 등록 불필요, 바로 됨 |
| **Play Store 정식 등록** | `.aab` | 신규 앱은 **AAB만** 업로드 가능 (APK 불가) |

PWABuilder가 주는 zip 안에 **둘 다** 들어 있습니다. "APK만 만들고 싶다"면 1~3단계만 하면 끝이에요.

---

## 1. HTTPS로 배포 (필수 선행조건)

TWA는 실제 웹 주소를 감싸는 방식이라 **배포가 먼저**입니다. GitHub Pages에 올리고
`https://<사용자명>.github.io/deskfit/` 가 폰 크롬에서 정상 동작하는지 먼저 확인하세요.

체크:
- [ ] https 로 열림 (http 아님)
- [ ] 설정 맨 아래 `build 10` 보임
- [ ] 크롬 메뉴에 "앱 설치" 뜸 (= manifest + SW 정상)

## 2. PWABuilder에서 패키징

1. https://www.pwabuilder.com 접속 → 배포한 URL 입력 → **Start**
2. 점수 리포트 확인 (manifest/SW/보안 항목 통과해야 함)
3. **Package for stores** → **Android** → **Generate Package**
4. 옵션에서 아래 두 개만 신경 쓰면 됩니다 (나머지 기본값 OK)

| 항목 | 값 |
|---|---|
| Package ID | `com.kwangseok.deskfit` ← **아래 함정 A 참고** |
| App name | 데스크핏 |
| Signing key | **Create new** (처음이면) |

5. 다운로드된 zip 안:
```
├── app-release-signed.apk   ← 폰에 직접 설치할 파일
├── app-release-bundle.aab   ← Play Store 업로드용
├── signing.keystore         ← ★ 절대 잃어버리면 안 됨(백업!)
├── signing-key-info.txt     ← 비밀번호. 같이 백업
└── assetlinks.json          ← 함정 B 참고
```

> ⚠️ **keystore를 잃어버리면 그 앱은 영원히 업데이트를 못 합니다.** Play에 올릴 거면
> `signing.keystore` + `signing-key-info.txt`를 반드시 안전한 곳에 백업하세요.

## 3. APK를 폰에 설치

- **간단한 방법**: `app-release-signed.apk`를 구글 드라이브/카톡으로 폰에 보내서 탭 →
  "출처를 알 수 없는 앱 설치 허용" 한 번 켜주면 설치됩니다.
- **adb 방법**: `adb install app-release-signed.apk`

여기까지가 APK 끝입니다.

---

## ⚠️ 함정 A — Package ID (이미 한 번 겪으신 것)

Java 패키지 규칙이라 **숫자로 시작 불가, 하이픈(-) 불가**입니다.
GitHub 사용자명이 `00kwang-droid` 같은 형태면 그대로 못 씁니다:

```
✗ io.github.00kwang-droid.deskfit   숫자 시작 + 하이픈
✓ com.kwangseok.deskfit
✓ io.github.kwangdroid.deskfit
```

한 번 정하면 **Play에서 영구 고정**(변경 불가)이니 신중히 정하세요.

## ⚠️ 함정 B — assetlinks.json 위치 (GitHub Pages에서 제일 많이 걸림)

이걸 안 하면 앱 안에 **주소창이 그대로 보입니다** (앱이 아니라 브라우저처럼 보임).

Digital Asset Links는 **도메인 루트(origin)** 에서만 검증됩니다. 즉:

```
✗ https://<사용자명>.github.io/deskfit/.well-known/assetlinks.json   ← 안 됨!
✓ https://<사용자명>.github.io/.well-known/assetlinks.json           ← 여기여야 함
```

deskfit 저장소가 아니라 **`<사용자명>.github.io` 라는 이름의 별도 저장소**를 만들어서
그 루트에 `.well-known/assetlinks.json`을 올려야 합니다.

- 이미 다른 PWA(스트링웍스 등)도 같은 origin에 있다면, **한 파일에 배열로 여러 앱을 같이** 넣으세요.
- 반영까지 몇 분~몇 시간 걸릴 수 있고, 앱 재설치하면 확인됩니다.
- 검증 도구: https://developers.google.com/digital-asset-links/tools/generator

## ⚠️ 함정 C — Target API 마감 (2026-08-31)

- 현재: API 35(Android 15) 이상
- **2026년 8월 31일부터: API 36(Android 16) 이상**이어야 신규 등록 및 업데이트 가능

PWABuilder(내부적으로 Bubblewrap)는 과거에도 API 상향 대응이 늦어 문제가 된 전례가 있습니다.
Play에 올릴 거면 패키징 시점에 PWABuilder가 뽑아주는 targetSdk를 확인하고,
거부당하면 PWABuilder 최신 버전으로 다시 뽑거나 Bubblewrap CLI로 직접 빌드하세요.

---

## 4. Play Store 등록까지 간다면 추가로 필요한 것

- **개발자 계정** $25 (1회)
- **AAB** 업로드 (`app-release-bundle.aab`)
- **스크린샷** — 이 폴더에 만들어 뒀습니다 (1080×1920):
  `screenshot-today.png`, `screenshot-report.png`, `screenshot-settings.png`
- **512×512 아이콘** — `icon-512.png` 그대로 사용 가능
- **1024×500 피처 그래픽** — 별도 제작 필요
- **개인정보처리방침 URL** — 필수. GitHub Pages에 `privacy.html` 하나 올리면 됨
- **데이터 안전(Data Safety) 양식** — 아래 내용을 정확히 신고해야 함:
  - 운동/식단 기록은 **기기에만 저장**되고 서버로 전송되지 않음
  - 단, 사용자가 **AI 코칭을 요청할 때만** 그날의 운동/식단 요약이 Google Gemini API로 전송됨
  - Gemini API 키는 사용자가 직접 입력하며 기기에만 저장됨
- **정책 4.3 (최소 기능)** — 단순 웹뷰 래핑은 거부되지만, **정상 PWA를 TWA로 감싸는 건
  구글이 공식 권장하는 방식**이라 통과합니다. 데스크핏은 SW/manifest/오프라인 동작이 다 있어 해당됨.

### Play 배포 시 꼭 생각해볼 것 — Gemini 키

지금 구조는 **사용자가 각자 자기 Gemini 키를 발급받아 입력**해야 AI 코칭이 됩니다.
개인용이면 문제없지만, 일반 사용자에게 배포하면 대부분 이 단계에서 이탈해요. 선택지:

1. **현재 유지** — AI 코칭을 "고급 기능"으로 두고, 키 없으면 로컬 평가 문구 표시 (이미 그렇게 동작함)
2. **프록시 서버** — Cloudflare Workers 등에 내 키를 숨기고 앱은 그 엔드포인트만 호출.
   `app.js`의 `GEMINI_API` 상수만 바꾸면 됨. 단, 사용량 비용을 내가 부담

---

## 5. 업데이트할 때

웹을 고치면 **앱은 자동으로 갱신**됩니다 (TWA는 실제 웹을 띄우는 구조라
APK를 다시 만들 필요 없음). 단 아래는 재패키징이 필요합니다:
- 앱 이름/아이콘 변경
- targetSdk 상향
- manifest의 name/theme_color 등 변경 반영
