# API Gateway

Spring Boot 3 · Spring Cloud · Redis 기반 MSA API 게이트웨이 서비스입니다.  
모든 클라이언트 요청의 단일 진입점으로, JWT 인증·인가 및 마이크로서비스 라우팅을 담당합니다.

---

## 기술 스택

![Java](https://img.shields.io/badge/Java_17-007396?style=flat&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.4-6DB33F?style=flat&logo=springboot&logoColor=white)
![Spring Cloud](https://img.shields.io/badge/Spring_Cloud_2024-6DB33F?style=flat&logo=spring&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security_6-6DB33F?style=flat&logo=springsecurity&logoColor=white)
![WebFlux](https://img.shields.io/badge/WebFlux-6DB33F?style=flat&logo=spring&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=flat&logo=jsonwebtokens&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat&logo=redis&logoColor=white)
![Eureka](https://img.shields.io/badge/Eureka-6DB33F?style=flat&logo=spring&logoColor=white)
![Gradle](https://img.shields.io/badge/Gradle-02303A?style=flat&logo=gradle&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)

---

## 프로젝트 구조

```
api-gateway/
└── src/main/java/com/example/apigateway/
    ├── ApiGatewayApplication.java              # 진입점
    ├── config/
    │   └── SecurityConfig.java                 # WebFlux 보안 설정 (경로별 인증 정책)
    ├── handler/
    │   ├── CustomAccessDeniedHandler.java      # 403 권한 오류 처리
    │   └── CustomAuthenticationEntryPoint.java # 401 인증 오류 처리
    ├── jwt/
    │   ├── JwtFilter.java                      # 토큰 검증 · 만료시 재발급 필터
    │   └── JwtTokenProvider.java               # JWT 생성 · 검증 · 파싱 (HMAC-SHA256)
    └── redis/
        └── RedisConfig.java                    # Redis 연결 및 직렬화 설정
```

---

## 실행 방법

게이트웨이 실행 전 Redis와 Eureka 서버가 먼저 구동 중이어야 합니다.

```bash
# 1. Redis 컨테이너 실행
docker run -d -p 6379:6379 --name redis redis

# 2. Eureka 서버 실행 (별도 프로젝트)
#    http://localhost:8761 에서 서비스 등록 확인 가능

# 3. 게이트웨이 실행
./gradlew bootRun
# 또는 IntelliJ → ApiGatewayApplication.java 실행

# 4. http://localhost:8080 으로 모든 요청 수신
```

> `application.yml`에서 JWT 시크릿 키, Redis 호스트, Eureka URL을 환경에 맞게 수정하세요.

---

## 학습 내용

| 패키지 / 파일 | 주요 학습 내용 |
|---|---|
| `config/SecurityConfig` | `@EnableWebFluxSecurity` · `ServerHttpSecurity` 기반 경로별 인증 정책, `NoOpServerSecurityContextRepository`로 세션 비활성화 |
| `handler/` | WebFlux 전용 `ServerAccessDeniedHandler` · `ServerAuthenticationEntryPoint`로 401/403 커스텀 응답 처리 |
| `jwt/JwtTokenProvider` | HMAC-SHA256 시크릿 키 초기화(`@PostConstruct`), 액세스 토큰 생성 · 검증 · Claims 파싱 |
| `jwt/JwtFilter` | `WebFilter` 구현으로 요청 가로채기, 토큰 만료 시 Redis 리프레시 토큰으로 액세스 토큰 재발급 후 응답 헤더 갱신 |
| `redis/RedisConfig` | Lettuce 연결 팩토리, `RedisTemplate` 키/값 직렬화(`StringRedisSerializer` · `GenericJackson2JsonRedisSerializer`) |
| `application.yml` | Spring Cloud Gateway 라우팅 구성, Eureka 서비스 등록 · 헬스체크 주기, JWT 환경 변수 외부화 |

---

## 개발 환경

- **JDK 17** 이상 — [다운로드](https://download.oracle.com/java/17/archive/jdk-17.0.12_windows-x64_bin.exe)
- **IntelliJ IDEA** Ultimate (추천) / Community
- **Docker** — Redis 컨테이너 실행

```bash
docker run -d -p 6379:6379 --name redis redis
```

- **API 테스트** — Postman / IntelliJ HTTP Client
