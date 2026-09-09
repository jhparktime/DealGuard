<div align="center">

  <h1>OnGuard</h1>
  <p><strong>AI 기반 실시간 스캠 탐지 안드로이드 앱</strong></p>

  <p>
    <a href="https://kotlinlang.org">
      <img src="https://img.shields.io/badge/Kotlin-2.0+-purple.svg" alt="Kotlin">
    </a>
    <a href="https://developer.android.com">
      <img src="https://img.shields.io/badge/Android-8.0+-green.svg" alt="Android">
    </a>
    <a href="https://ai.google.dev/gemini-api">
      <img src="https://img.shields.io/badge/LLM-Gemini_Flash-orange.svg" alt="Gemini">
    </a>
    <a href="LICENSE">
      <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT">
    </a>
  </p>

  <p><em>데이콘 경진대회 출품작 · 경찰청 후원 · 데이터유니버스 주최</em></p>

  <br />

  <table>
    <tr>
      <td align="center">
        <strong>📱 어디서든</strong><br/>
        카카오톡·메신저·거래 앱까지<br/>
        한 번에 모니터링
      </td>
      <td align="center">
        <strong>🤖 똑똑한 탐지</strong><br/>
        Rule + LLM 하이브리드로<br/>
        맥락까지 이해하는 스캠 필터
      </td>
      <td align="center">
        <strong>🔐 프라이버시 우선</strong><br/>
        온디바이스 분석 중심,<br/>
        민감정보 최소 전송 & 마스킹
      </td>
    </tr>
  </table>

</div>

---

## 🌈 목차

- [주요 기능](#주요-기능)
- [차별점](#차별점)
- [기술 아키텍처](#기술-아키텍처)
- [지원 앱 목록](#지원-앱-목록)
- [시작하기](#시작하기)
- [프로젝트 구조](#프로젝트-구조)
- [보안 & 프라이버시](#보안--프라이버시)
- [최신 업데이트](#최신-업데이트)

---

## 프로젝트 한눈에 보기

**규칙 기반 탐지와 LLM을 Android 앱에 통합한 스캠 위험 알림 팀 프로젝트**입니다. 키워드만으로 판단하기 어려운 메시지의 맥락을 분석하고 사용자에게 경고 이유를 전달합니다.

- **AI 구현:** 규칙 기반 분석과 Gemini의 맥락 분석·경고 문구 생성을 연결합니다.
- **jhparktime의 기여:** Gemini 연동, 프롬프트 수정, 탐지 점수 조정, 경고 문구 생성 및 리팩터링. [기여 커밋](https://github.com/jhparktime/OnGuard/commits/main/?author=jhparktime)에서 확인할 수 있습니다.
- **협업:** 초기 구조·아키텍처는 Zaeewang, AI/LLM 통합·리팩터링은 jhparktime이 담당했습니다.
- **확인 가능한 산출물:** 앱 소스와 탐지 엔진 테스트 코드. 탐지 정확도나 응답 시간의 보편적인 성능 보장을 의미하지 않습니다.

**먼저 볼 코드:** [탐지 엔진](app/src/main/java/com/onguard/detector/) · [Gemini 분석](app/src/main/java/com/onguard/detector/LLMScamDetector.kt) · [탐지 테스트](app/src/test/java/com/onguard/detector/HybridScamDetectorTest.kt)

**데이터 흐름:** LLM 분석 경로에서는 `PiiMasker`를 적용한 메시지·대화 맥락과 분석 정보를 외부 Gemini API로 전달합니다. 실행에는 Android 환경과 사용할 외부 서비스의 API 설정이 필요합니다.

---

## 주요 기능

### 🛡️ 플랫폼 무관 실시간 모니터링
- **18개 이상의 메신저/거래 앱 지원**
  - 메신저: 카카오톡, 텔레그램, 왓츠앱, 페이스북 메신저, 인스타그램, 라인, 디스코드 등
  - SMS/MMS: Google Messages, Samsung Messages, 기본 메시지 앱
  - 거래 플랫폼: 당근마켓, 네이버
- AccessibilityService 기반 - 특정 앱에 종속되지 않음

### 🤖 하이브리드 AI 탐지 시스템
OnGuard는 3단계 하이브리드 탐지 시스템으로 **높은 정확도**와 **빠른 응답**을 동시에 제공합니다:

#### 1단계: Rule-based 필터 (< 100ms)
- **KeywordMatcher**: 150개 이상의 스캠 키워드 패턴 매칭
  - HIGH 위험: "급전", "선입금", "고수익 보장", "원금 보장"
  - MEDIUM 위험: "입금", "송금", "계좌", "비밀번호"
  - 가중치 기반 점수 계산
- **UrlAnalyzer**: URL 위험도 분석
  - KISA 피싱사이트 DB 조회
  - 무료 도메인 (.tk, .ml, .ga 등) 탐지
  - 단축 URL 감지 (bit.ly, goo.gl 등)
  - **강화된 은행 도메인 검증** - 서브도메인 속임수 방지
- **PhoneAnalyzer**: 전화번호 검증
  - Counter Scam 112 DB 조회
  - 보이스피싱/스미싱 신고 이력 확인
  - 070/050 의심 대역 탐지
- **AccountAnalyzer**: 계좌번호 검증
  - 경찰청 사기계좌 DB 조회
  - 다수 신고 이력 체크

#### 2단계: 외부 LLM 분석 (애매한 경우)
- **Google Gemini 2.5 Flash** 통합
- Rule-based 탐지가 애매한 경우(30~70% 신뢰도)에만 호출
- 문맥 기반 분석 + 위험 이유 설명 생성
- 스캠 유형 자동 분류 (투자/중고거래/피싱 등)
- 무료 티어 범위 내 일일 호출 제한

#### 3단계: 결과 통합
- Rule-based + LLM 결과를 가중 평균하여 최종 판정
- **스레드 안전한 점수 누적** - AtomicInteger/AtomicReference 사용
- **조기 종료 최적화** - 최대 위험도 도달 시 추가 분석 스킵

### ⚡ 즉시 경고 오버레이
위험 감지 시 화면 상단에 상세 경고 표시:
- 위험도 퍼센트 (0~100%)
- AI 생성 경고 메시지
- 탐지 이유 목록 (예: "KISA 피싱사이트 등록", "보이스피싱 신고 이력")
- 의심 문구 하이라이트
- 스캠 유형별 맞춤 조언

### 🔐 프라이버시 우선
- **모든 분석은 온디바이스에서 수행** (LLM API 호출 제외)
- LLM 분석 경로에서는 마스킹한 메시지·대화 맥락과 분석 정보를 외부 Gemini API로 전송
- 사용자 명시적 동의 후에만 모니터링 시작
- Google Play Prominent Disclosure 준수

### 🗄️ 사기 DB 연동
- **Counter Scam 112 API**: 전화번호 사기 이력 조회
- **경찰청 사기계좌 조회 API**: 계좌번호 신고 이력
- **KISA 피싱사이트 DB**: 공공 피싱 URL 데이터베이스
- **LRU 캐시 (TTL 15분)**: 중복 API 호출 방지

---

## 차별점

| 기존 앱 | OnGuard |
|--------|---------|
| 특정 앱에만 동작 | **모든 메신저 지원 (18개+)** |
| 서버로 데이터 전송 | **온디바이스 + 외부 LLM 하이브리드** |
| 단순 키워드 매칭 | **AI 문맥 분석 + 이유 설명** |
| 사후 신고 | **실시간 경고 (<100ms)** |
| 오탐률 높음 | **강화된 검증 로직 (은행명/도메인)** |
| 스레드 안전성 미흡 | **AtomicInteger/Mutex 사용** |

---

## 기술 아키텍처

### 탐지 파이프라인

```
채팅 메시지 수신
      │
      ▼
┌─────────────────────────────┐
│  AccessibilityService       │
│  - 18개 앱 모니터링         │
│  - 텍스트 추출              │
└─────────────────────────────┘
      │
      ▼
┌─────────────────────────────┐
│  Rule-based 1차 필터 (빠름) │
│  - KeywordMatcher (0.1ms)   │
│  - UrlAnalyzer + KISA DB    │
│  - PhoneAnalyzer + 112 API  │
│  - AccountAnalyzer + 경찰청 │
└─────────────────────────────┘
      │
      ├── 명확한 위험 (>70%) ──────► 즉시 경고
      │
      ▼ 애매한 경우 (30~70%)
┌─────────────────────────────┐
│  Gemini 2.5 Flash LLM       │
│  - 문맥 기반 탐지           │
│  - 위험 이유 설명 생성      │
│  - 스캠 유형 분류           │
└─────────────────────────────┘
      │
      ▼
┌─────────────────────────────┐
│  결과 통합 (가중 평균)      │
│  - Thread-safe 점수 누적    │
│  - 조기 종료 최적화         │
└─────────────────────────────┘
      │
      ▼
    Overlay 경고 표시
```

### 기술 스택

```
Language:       Kotlin 2.0+
Min SDK:        26 (Android 8.0)
Target SDK:     34 (Android 14)
Architecture:   MVVM + Clean Architecture
DI:             Hilt
Async:          Kotlin Coroutines + Flow
UI:             Jetpack Compose + XML (Overlay)
External LLM:   Google Gemini 2.5 Flash API
Network:        Retrofit2 + OkHttp
Local DB:       Room
Concurrency:    AtomicInteger, AtomicReference, Mutex
Cache:          LruCache with TTL
Build:          Gradle Kotlin DSL
```

---

## 지원 앱 목록

### 메신저 앱 (9개)
- 카카오톡 (com.kakao.talk)
- 텔레그램 (org.telegram.messenger)
- 왓츠앱 (com.whatsapp)
- 페이스북 메신저 (com.facebook.orca)
- 인스타그램 (com.instagram.android)
- 라인 (jp.naver.line.android)
- 위챗 (com.tencent.mm)
- 디스코드 (com.discord)
- 스냅챗 (com.snapchat.android)

### SMS/MMS 앱 (3개)
- Google Messages (com.google.android.apps.messaging)
- Samsung Messages (com.samsung.android.messaging)
- 기본 메시지 앱 (com.android.mms)

### 거래 플랫폼 (2개)
- 당근마켓 (kr.co.daangn)
- 네이버 (com.nhn.android.search)

### 기타 (4개)
- 바이버 (com.viber.voip)
- 킥 (kik.android)
- 스카이프 (com.skype.raider)

**총 18개 앱 지원** - 지속적으로 추가 예정

---

## 시작하기

### 1. 환경 요구사항

- Android Studio Hedgehog (2023.1.1) 이상
- JDK 17 (Java 25는 AGP 8.13.2와 호환 문제 가능)
- Android SDK 34

### 2. 프로젝트 클론

```bash
git clone --recurse-submodules https://github.com/jhparktime/OnGuard.git
cd OnGuard
git checkout Backend  # 백엔드 브랜치
```

### 3. API 키 설정

`local.properties` 파일 생성:

```properties
# SDK Location (Android Studio auto-generates)
sdk.dir=/path/to/your/Android/sdk

# API Keys
THECHEAT_API_KEY=your_api_key_here
KISA_API_KEY=your_kisa_api_key_here

# Gemini API 설정 (선택사항)
ENABLE_LLM=true
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MAX_CALLS_PER_DAY=100
```

**API 키 발급**:
- **Counter Scam 112 API**: [Counter Scam 112](https://counterscam112.com) 회원가입
- **KISA API**: [공공데이터포털](https://www.data.go.kr/)
- **Gemini API**: [Google AI Studio](https://ai.google.dev/) (무료 티어 사용 가능)

### 4. 빌드 & 실행

```bash
# Android Studio에서 열기 권장
# 또는 CLI:
./gradlew assembleDebug
./gradlew installDebug
```

### 5. 권한 설정 (앱 설치 후)

OnGuard 실행 후:
1. **접근성 서비스 활성화**
   - 설정 → 접근성 → OnGuard → 활성화
2. **오버레이 권한 허용**
   - 설정 → 앱 → 특수 앱 접근 → 다른 앱 위에 표시 → OnGuard 허용

---

## 프로젝트 구조

```
app/src/main/java/com/onguard/
├── di/                     # Hilt DI modules
│   ├── AppModule.kt
│   ├── NetworkModule.kt
│   └── DatabaseModule.kt
│
├── data/                   # Data Layer
│   ├── local/              # Room Database
│   │   ├── dao/            # ScamAlertDao
│   │   ├── entity/         # ScamAlertEntity
│   │   └── AppDatabase.kt
│   ├── remote/             # Retrofit API
│   │   ├── api/            # CounterScam112Api, PoliceFraudApi
│   │   ├── dto/            # Data Transfer Objects
│   │   └── interceptor/    # CookieJar, Logging
│   └── repository/         # Repository Implementations
│       ├── CounterScamRepositoryImpl.kt  # LRU 캐시, 세션 관리
│       └── PoliceFraudRepositoryImpl.kt  # Thread-safe 캐시
│
├── domain/                 # Domain Layer
│   ├── model/              # ScamAnalysis, ScamAlert, ScamType
│   ├── repository/         # Repository Interfaces
│   └── usecase/            # Business Logic
│
├── presentation/           # Presentation Layer
│   ├── ui/
│   │   ├── main/           # Main Activity + Compose
│   │   ├── onboarding/     # Permission & consent flow
│   │   └── settings/       # App settings
│   └── viewmodel/          # ViewModels with StateFlow
│
├── service/                # Android Services
│   ├── ScamDetectionAccessibilityService.kt  # 채팅 모니터링
│   └── OverlayService.kt                     # 경고 UI 표시
│
├── detector/               # Scam Detection Engine (핵심)
│   ├── HybridScamDetector.kt     # 하이브리드 탐지 통합
│   ├── KeywordMatcher.kt         # Rule-based 키워드 매칭
│   ├── UrlAnalyzer.kt            # URL 분석 + 도메인 검증
│   ├── PhoneAnalyzer.kt          # 전화번호 검증 + 점수 캡핑
│   ├── AccountAnalyzer.kt        # 계좌번호 검증 + 점수 캡핑
│   └── LLMScamDetector.kt        # Gemini LLM 탐지 (Thread-safe)
│
└── util/                   # Utilities
    ├── PermissionHelper.kt
    ├── NotificationHelper.kt
    └── DebugLog.kt
```

---

## 보안 & 프라이버시

OnGuard는 사용자 프라이버시를 최우선으로 설계되었습니다:

### 데이터 보호
- **외부 전송 범위:** LLM 분석 경로에서 `PiiMasker`를 적용한 메시지·대화 맥락과 분석 정보를 Gemini API에 전달
- ✅ **모든 Rule-based 분석은 온디바이스에서 수행**
- **마스킹 경로:** `HybridScamDetector`에서 `ScamLlmRequest`의 메시지·맥락·분석 이유에 `PiiMasker` 적용
- ✅ **로그에 민감 정보 마스킹** (전화번호, 계좌번호)

### 권한 관리
- ✅ **사용자 동의 없이 모니터링 시작 금지**
- ✅ **Google Play Prominent Disclosure 준수**
- ✅ **명시적 권한 요청 UI**

### 코드 보안
- ✅ **API 키 하드코딩 금지** (BuildConfig 사용)
- ✅ **스레드 안전한 구현** (AtomicInteger, Mutex)
- ✅ **SQL Injection 방지** (Room Parameterized Query)
- ✅ **메모리 누수 방지** (WeakReference 사용)

자세한 내용은 [CLAUDE.md](CLAUDE.md#-security-requirements) 참조

---

## 최신 업데이트

### v1.1.0 (2026-02-08) - 리팩터링 및 안정성 개선

**스레드 안전성 향상**:
- LLMScamDetector에 `AtomicInteger`/`AtomicReference` 적용
- Repository 세션 관리에 `Mutex` 사용
- 캐시 변수 `@Volatile` 처리

**오탐률 감소**:
- 은행명 키워드 제거 (카카오뱅크, 국민은행 등 8개)
- 주민등록번호/여권번호 패턴 제거 (API 조회 불가)
- 강화된 은행 도메인 검증 (서브도메인 속임수 방지)

**성능 최적화**:
- PhoneAnalyzer/AccountAnalyzer 조기 종료 (최대 점수 도달 시)
- UrlAnalyzer Set 사용으로 중복 URL 자동 제거
- 중간 점수 캡핑으로 불필요한 연산 제거

**에러 처리 개선**:
- Repository API 실패 시 `Result.failure` 명시적 반환
- 호출자가 에러와 "결과 없음" 구분 가능
- Thread-safe 세션 무효화

**코드 품질**:
- Dead code 제거 (HybridScamDetector 중복 함수)
- KeywordMatcher 별칭 함수 삭제
- 주석 개선 및 KDoc 추가

자세한 내용은 [리팩터링 로그](docs/refactoring-log-2026-02-08.md) 참조

---

## 개발 로드맵

자세한 개발 계획은 [DEVELOPMENT_ROADMAP.md](DEVELOPMENT_ROADMAP.md) 참조

**현재 진행 상황**: Backend 브랜치 개발 완료

- [x] 프로젝트 초기 설정
- [x] Clean Architecture 구조 생성
- [x] 기본 도메인 모델 정의
- [x] 접근성 서비스 구현 - 18개 앱 지원
- [x] Rule-based 탐지 엔진 (KeywordMatcher, UrlAnalyzer)
- [x] 외부 API 연동 (Counter Scam 112, 경찰청, KISA)
- [x] Room 데이터베이스 구현
- [x] Overlay 경고 시스템
- [x] **Gemini LLM 통합**
  - [x] Google Gemini 2.5 Flash API 연동
  - [x] 투자/중고거래 스캠 탐지 프롬프트
  - [x] AI 경고 메시지 + 이유 설명 생성
  - [x] HybridScamDetector 통합
- [x] **리팩터링 및 안정성 개선**
  - [x] 스레드 안전성 향상
  - [x] 오탐률 감소
  - [x] 성능 최적화

---

## 테스트

### 단위 테스트

```bash
./gradlew test
```

### 테스트 커버리지

```bash
./gradlew testDebugUnitTest jacocoTestReport
```

**목표 커버리지**: 전체 70%, detector 패키지 90%

### 통합 테스트

```bash
./gradlew connectedAndroidTest
```

---

## 문제 해결

### Java 버전 호환성
현재 시스템에 Java 25가 설치된 경우 AGP 8.13.2와 호환 문제가 발생할 수 있습니다. Java 17 또는 21 사용을 권장합니다.

### 빌드 실패
```bash
# Clean build
./gradlew clean assembleDebug

# Gradle 캐시 초기화
rm -rf .gradle/
```

### 접근성 서비스가 동작하지 않음
1. 설정 → 접근성 → OnGuard 확인
2. 앱 재시작
3. Logcat에서 "OnGuardAccessibility" 태그 확인

---

## 기여하기

이 프로젝트는 데이콘 경진대회 출품작입니다. 기여를 환영합니다!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'feat: add some amazing feature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

**커밋 컨벤션**: [Conventional Commits](https://www.conventionalcommits.org/) 준수

---

## 라이선스

이 프로젝트는 데이콘 경진대회 출품작입니다.

Gemini API 사용 시 해당 모델의 라이선스 및 이용 약관 준수 필요

---

## 개발자

- **Zaeewang** - Initial work & Architecture
- **jhparktime** - AI/LLM Integration & Refactoring

---

## 감사의 말

- **경찰청 & 데이터유니버스** - 경진대회 주최
- **Counter Scam 112** - 전화번호 사기 DB API 제공
- **경찰청** - 사기계좌 조회 API 제공
- **KISA** - 피싱사이트 공공 DB 제공
- **Google** - Gemini Flash API 무료 티어 제공

---

## 관련 문서

- [개발 가이드](CLAUDE.md) - Claude Code를 위한 프로젝트 가이드
- [개발 로드맵](DEVELOPMENT_ROADMAP.md) - 단계별 개발 계획
- [리팩터링 로그](docs/refactoring-log-2026-02-08.md) - 최신 리팩터링 내역

---

## 문의

프로젝트 관련 문의사항은 [Issues](https://github.com/jhparktime/OnGuard/issues) 탭을 이용해주세요.

---

**Made with ❤️ for a safer digital world**
