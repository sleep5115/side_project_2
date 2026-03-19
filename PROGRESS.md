# Pickty 진행 현황

> 웹 AI(Gemini)에게 컨텍스트를 전달하기 위한 파일.
> 작업할 때마다 Cursor AI가 이 파일을 업데이트한다.

---

## 전체 진행 상태

| 영역 | 상태 | 비고 |
|---|---|---|
| 개발 환경 세팅 | ✅ 완료 | |
| Frontend 기초 | ✅ 완료 | |
| Backend 기초 | ✅ 완료 | |
| Auth — 엔티티/도메인 설계 | ✅ 완료 | User, SocialAccount 엔티티 완성 |
| Auth — 로그인/회원가입 UI | ✅ 완료 | 소셜 로그인 버튼 포함 |
| Auth — 백엔드 API 연동 | ⬜ 미시작 | Spring Security OAuth2 구현 필요 |
| Tier Maker | ⬜ 미시작 | |
| Ideal Type World Cup | ⬜ 미시작 | |

---

## 완료된 작업

### 개발 환경
- Docker Compose로 PostgreSQL 17 + Valkey 9 로컬 구동 (`CursorProjects/docker-compose.yml`)
- DB 접속 정보는 `application-local.yaml`로 분리 후 gitignore 처리

### Frontend (`side_project_1`)
- Next.js 16 + React 19 + Tailwind CSS v4 + TypeScript 기본 세팅 완료
- `next-themes` + `zustand` 설치
- 다크/라이트 모드 토글 구현 (`ThemeProvider`, `ThemeToggle` 컴포넌트)
- **페이지 목록**
  - `/` — 메인 랜딩 (티어/월드컵 진입 버튼)
  - `/tier` — 더미 티어 대시보드
  - `/worldcup` — 더미 월드컵 대시보드
  - `/login` — 로그인 페이지 (이메일/비밀번호 + 소셜 로그인 UI)
  - `/signup` — 회원가입 페이지 (이메일/닉네임/비밀번호)
- **폼 검증**: `react-hook-form` + `zod` 사용 (`src/lib/schemas/auth.ts`에 스키마 정의)
- **소셜 로그인 버튼 구현 완료** (UI only, API 미연동)
  - 주요: Google, 네이버, 카카오 (와이드 버튼)
  - 방송: Twitch, 치지직, SOOP (원형 버튼)

### Backend (`side_project_2`)
- Spring Boot 4.0.3 + Kotlin 2.2.21 + JPA + Security + Validation 기본 세팅 완료
- `application.yaml` 구성: PostgreSQL, Valkey(Redis 호환), JPA 설정
- `spring-boot-starter-data-redis` 의존성 추가
- JVM 타겟 불일치 해결: JDK 25 toolchain + 컴파일 타겟 JVM 24 (Kotlin 2.4.0 이후 25로 업그레이드 예정)
- 서버 정상 기동 확인 (`localhost:8080`)
- **패키지 구조**: `com.pickty.server`
  - `domain/user/` — User, SocialAccount, Provider, Role
  - `global/common/` — BaseTimeEntity
  - `global/config/` — JpaAuditingConfig

---

## 핵심 데이터베이스 스키마

### BaseTimeEntity (공통 상속)
| 필드 | 타입 | 제약 |
|---|---|---|
| `created_at` | `TIMESTAMP` | NOT NULL, 수정 불가 |
| `updated_at` | `TIMESTAMP` | NOT NULL |

### users 테이블 (`User.kt`)
| 필드 | 타입 | 제약 | 비고 |
|---|---|---|---|
| `id` | `BIGINT` | PK, AUTO_INCREMENT | |
| `email` | `VARCHAR` | UNIQUE, NULL 허용 | 소셜 전용 유저는 null. PostgreSQL은 NULL 중복 허용 |
| `password` | `VARCHAR` | NULL 허용 | 소셜 전용 유저는 null. BCrypt 인코딩 예정 |
| `nickname` | `VARCHAR` | NOT NULL | |
| `profile_image_url` | `VARCHAR` | NULL 허용 | |
| `role` | `VARCHAR` | NOT NULL | Enum: `USER`, `ADMIN` |
| `created_at` | `TIMESTAMP` | NOT NULL | BaseTimeEntity 상속 |
| `updated_at` | `TIMESTAMP` | NOT NULL | BaseTimeEntity 상속 |

### social_accounts 테이블 (`SocialAccount.kt`)
| 필드 | 타입 | 제약 | 비고 |
|---|---|---|---|
| `id` | `BIGINT` | PK, AUTO_INCREMENT | |
| `user_id` | `BIGINT` | FK → users.id, NOT NULL | LAZY 로딩 |
| `provider` | `VARCHAR` | NOT NULL | Enum 값 (아래 참고) |
| `provider_id` | `VARCHAR` | NOT NULL | |
| `created_at` | `TIMESTAMP` | NOT NULL | BaseTimeEntity 상속 |
| `updated_at` | `TIMESTAMP` | NOT NULL | BaseTimeEntity 상속 |
- **복합 UNIQUE 제약**: `(provider, provider_id)`

### 연관 관계
- `User` 1 ↔ N `SocialAccount` (OneToMany / ManyToOne)
- cascade: ALL, orphanRemoval: true

### Provider Enum
| 그룹 | 값 |
|---|---|
| 주요 소셜 | `GOOGLE`, `NAVER`, `KAKAO` |
| 방송 플랫폼 | `CHZZK`, `SOOP`, `TWITCH` |
| 차후 검토 | `DISCORD`, `X`, `APPLE` |

---

## 핵심 코드 구조

### Frontend 주요 파일
```
src/
├── app/
│   ├── layout.tsx              # ThemeProvider 적용, suppressHydrationWarning
│   ├── globals.css             # Tailwind v4, @custom-variant dark, CSS 변수
│   ├── page.tsx                # 메인 랜딩
│   ├── login/page.tsx          # 로그인 (react-hook-form + zod)
│   ├── signup/page.tsx         # 회원가입 (react-hook-form + zod)
│   ├── tier/page.tsx           # 더미 티어 대시보드
│   └── worldcup/page.tsx       # 더미 월드컵 대시보드
├── components/
│   ├── ThemeToggle.tsx         # 다크/라이트 토글 버튼
│   └── providers/
│       └── ThemeProvider.tsx   # next-themes 래퍼
└── lib/
    └── schemas/
        └── auth.ts             # zod 스키마 (loginSchema, signupSchema)
```

### Backend 주요 파일
```
src/main/kotlin/com/pickty/server/
├── ServerApplication.kt
├── domain/
│   └── user/
│       ├── User.kt             # users 테이블 엔티티
│       ├── SocialAccount.kt    # social_accounts 테이블 엔티티
│       ├── Provider.kt         # 소셜 플랫폼 Enum
│       └── Role.kt             # 권한 Enum (USER, ADMIN)
└── global/
    ├── common/
    │   └── BaseTimeEntity.kt   # createdAt, updatedAt 공통 상속
    └── config/
        └── JpaAuditingConfig.kt
```

---

## 다음 작업 예정

- [ ] Backend: Spring Security + OAuth2 설정 (SecurityConfig)
- [ ] Backend: UserRepository, SocialAccountRepository
- [ ] Backend: OAuth2 로그인 흐름 구현 (Chzzk, Google)
- [ ] Backend: JWT 발급/검증 필터
- [ ] Frontend: 로그인/회원가입 API 실제 연동
