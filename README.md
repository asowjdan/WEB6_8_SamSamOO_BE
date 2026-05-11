# 👨‍💻 나의 기여 내역 — WEB6_8_SamSamOO_BE (Balaw - AI 법률 서비스)

> Member 도메인 전체와 OAuth 소셜 로그인, JWT 인증 체계를 전담하였으며,
> CI/CD 환경 설정 및 Swagger 문서 구성도 담당하였습니다.

---

## 🛠 기술 스택

`Java` `Spring Boot` `Spring Security` `JPA` `Redis` `JWT` `OAuth2` `GitHub Actions (CI/CD)`

---

## 📁 담당 작업 내역

### 1. Member 도메인 구현 (feat/41-member, feat/154-member)
- 회원 가입 / 로그인 / 이메일 인증 로직 구현
- 비밀번호 재설정 기능 구현 (Redis에 인증 여부 저장)
- 회원 탈퇴 시 연관 데이터 cascade 삭제 로직 구현
- Swagger 컨트롤러 순서 지정 및 전체 API 카테고리 정렬 코드 작성

### 2. OAuth 소셜 로그인 (feat/oauth)
- 백엔드 소셜 로그인 콜백 로직 구현
- 소셜 로그인 후 JWT 토큰 생성 및 쿠키 응답 처리
- 소셜 로그인 테스트 코드 작성 및 리팩토링

### 3. JWT 인증 체계 (feat/JWT-token, fix/jwt)
- JWT 토큰 생성 및 인증 로직 구현 (`JwtAuthenticationFilter`, `CookieUtil`)
- 토큰 재발급 로직 구현
- Redis를 활용한 비밀번호 재설정 인증 여부 저장 로직 구현

### 4. CI/CD 환경 구성 (ci[Github-Action])
- GitHub Actions CI-CD 테스트 환경 yml 설정
- CI 환경에서 embedded-redis `RedisAutoConfiguration` 자동 구성 비활성화
- Redis 컨테이너 연결 테스트 설정

### 5. 인프라 설정 (chore/33-infra)
- 개발/운영 환경별 Redis 설정 파일 분리
- h2 console 설정 및 로컬 테스트 yml 구성
- 배포 환경 Swagger 설정

---

## 🚧 문제 상황과 해결

### 1. 소셜 로그인 후 쿠키가 프론트엔드로 전달되지 않는 문제
- **상황**: 소셜 로그인 완료 후 발급된 JWT 토큰 쿠키가 프론트엔드(`www.trybalaw.com`)로 전달되지 않아 member_id 조회가 불가능한 버그가 발생했습니다. 원인은 `CookieUtil`에서 `SECURE_IN_PRODUCTION = false`, `SAME_SITE = "Lax"`가 코드에 하드코딩되어 있어 운영 환경(HTTPS, 크로스 도메인)에서 쿠키가 차단된 것이었습니다.
- **해결**: 하드코딩된 쿠키 설정값을 `@Value("${custom.cookie.secure:false}")`, `@Value("${custom.cookie.same-site:Lax}")`로 외부화하고, `application-dev.yml`에는 `secure: false / same-site: Lax`, `application-prod.yml`에는 `secure: true / same-site: None`을 환경별로 분리하여 적용했습니다.
- **배운 점**: 쿠키의 `SameSite`와 `Secure` 속성이 로컬/운영 환경에 따라 다르게 설정되어야 하며, 이를 코드에 하드코딩하면 배포 환경에서 인증이 완전히 깨질 수 있다는 것을 배웠습니다.

### 2. 로컬/소셜 회원 기본 키(PK) 중복 문제
- **상황**: 로컬 회원과 소셜 회원이 각각 독립적인 방식으로 PK를 생성하면서 두 테이블 간 id 값이 겹치는 문제가 발생했습니다. 이로 인해 post, poll_vote, history 등 연관 테이블에서 잘못된 회원을 참조하는 데이터 정합성 문제가 생겼습니다.
- **해결**: `member_id_sequence` 테이블을 신규 생성해 단일 시퀀스에서 id를 발급받도록 변경하고, 기존 데이터 마이그레이션 스크립트(`migrate_member_id_sequence.sql`)를 작성했습니다. `oauth2_member_id_mapping` 임시 테이블로 구 id와 신 id 매핑 후 `FOREIGN_KEY_CHECKS = 0`으로 제약 조건을 비활성화한 상태에서 관련 테이블을 일괄 업데이트했습니다.
- **배운 점**: 다중 인증 방식 설계 시 회원 식별자 전략을 초기에 단일화해야 하며, 데이터 마이그레이션 시 FK 제약 조건과 관련 테이블 전체를 함께 고려해야 한다는 것을 배웠습니다.

### 3. CI 환경에서 embedded-redis 자동 구성 충돌
- **상황**: GitHub Actions CI 파이프라인에서 실제 Redis 컨테이너를 사용하는 테스트를 실행할 때 Spring Boot가 `embedded-redis`의 `RedisAutoConfiguration`을 자동으로 활성화하면서 실제 Redis 연결과 충돌이 발생했습니다.
- **해결**: `application-test-ci.yml`에 `autoconfigure.exclude: org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration`을 추가해 CI 환경에서 embedded-redis 자동 구성을 명시적으로 비활성화했습니다.
- **배운 점**: Spring Boot의 자동 구성 메커니즘이 테스트 환경에서 의도치 않게 동작할 수 있으며, CI 전용 yml에서 명시적으로 제외 설정을 해야 한다는 것을 배웠습니다.

---

# Balaw (바로)
"Balaw"는 복잡하고 어려운 법률 문제에 직면한 일반인들을 위해 AI 기술을 활용하여 법률 정보의 문턱을 낮추는 서비스입니다. 


## 주요 기능
### **회원 및 인증 기능**


- **회원가입 및 로그인**: 이메일 또는 소셜 계정(카카오, 네이버) 간편 가입
- **로그아웃**: 안전한 계정 정보 보호 및 서비스 종료


### **AI 및 분석 기능**


> **AI 법률 분석** : 서비스의 핵심 기능
> 
> 
> 사용자가 자신의 상황을 텍스트로 입력하면 AI가 실시간으로 문맥을 분석하여 종합적인 분석 결과를 즉시 제공
> 
> **제공 정보**: 관련 법률 조항 | 유사 판례 | 핵심 쟁점 | 이해하기 쉬운 조언
> 


### **투표(유저 배심원단) 기능**


| 기능 | 설명 |
| --- | --- |
| **투표 생성 및 관리** | 법률적 고민을 익명으로 공유하고 투표 생성/수정/삭제 |
| **투표 참여 및 조회** | 다양한 사례를 조회하고 배심원으로 참여 |
| **투표 결과 통계** | 시각적 통계(막대그래프, 원형 차트)로 여론 흐름 확인 |
| **선택 수정 및 알림** | 투표 마감 전 선택 수정 가능, 새로운 반응 시 알림 제공 |


### **법률 정보 및 검색 기능**


- **법률 조항/판례 통합 검색**: 키워드 하나로 법률 조항, 판례 검색
- **법률 용어 인라인 검색 및 해설**: 방대한 데이터베이스에서 정확한 정보 검색 + 클릭 한 번으로 쉬운 해설 제공


### **UI/UX 및 사용자 경험**


- **반응형 UI/UX**: PC, 태블릿, 모바일 최적화 디자인
- **빠르고 쾌적한 사용 환경**: 페이지 로딩 속도 최소화, 부드러운 화면 전환
- **로딩 스켈레톤**: 데이터 로딩 중 콘텐츠 윤곽 먼저 표시로 체감 로딩 시간 단축


## 실행방법
### 1. 저장소 클론
```
git clone https://github.com/prgrms-web-devcourse-final-project/WEB6_8_SamSamOO_BE.git
cd WEB6_8_SamSamOO_BE/backend
```


### 2. 의존 서비스 실행 (PostgreSQL, Redis 등)
```
docker-compose up -d
```


### 3. Spring Boot 실행
```
./gradlew bootRun
```


## **기술 스택**


### **백엔드 (Backend)**


| **기술** | **세부 내용 / 역할** |
| --- | --- |
| **Java 21** | LTS 버전, 가상 스레드(Project Loom) 등 최신 언어 기능 활용 |
| **Spring Boot 3.5** | 빠른 애플리케이션 개발 및 설정 자동화, Java 21 완벽 지원 |
| **Spring Batch** | 대용량 데이터의 배치 처리 및 스케줄링 |
| **Gradle** | Groovy/Kotlin 기반의 유연하고 빠른 빌드 자동화 도구 |
| **Spring Data JPA** | 객체-관계 매핑(ORM)을 통해 생산성 향상 및 SQL 중심 개발 탈피 |
| **Spring Security / JWT** | 인증(Authentication) 및 인가(Authorization) 처리, 상태 비저장(Stateless) API를 위한 토큰 기반 인증 구현 |
| **OAuth 2.0** | 카카오, 네이버 소셜 로그인을 통한 간편한 사용자 인증 |
| **Springdoc (Swagger)** | Swagger UI를 활용한 API 명세 자동화 및 테스트 환경 제공 |
| **Elasticsearch** | 검색 기능 고도화 및 로그 데이터 분석/모니터링 |
| **Spring RAG** | LLM과 외부 데이터베이스(Vector DB 등)를 연동한 검색 증강 생성(RAG) 구현 |
