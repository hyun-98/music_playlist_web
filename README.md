# 🎵 Music Playlist Backend

---

## 1. 프로젝트 소개

음악 플레이리스트 웹 서비스의 백엔드 API 서버입니다.

사용자는 회원가입 및 로그인 후 자신만의 음악 플레이리스트를 생성하고 관리할 수 있습니다.  
Spring Boot 기반의 RESTful API로 구성되어 있으며, JWT 토큰 인증 방식을 사용합니다.

- **팀명**: Team 10
- **프로젝트명**: music_playlist_web
- **서버 기본 포트**: `8080`
- **프론트엔드 연동 포트**: `5173` (Vite 기본 포트)

---

## 2. 사용 기술

| 분류 | 기술 |
|------|------|
| Language | Java 17 ![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white) |
| Framework | Spring Boot 3.5.6 ![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white) |
| ORM | Spring Data JPA (Hibernate) ![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)|
| Security | Spring Security + JWT (jjwt 0.11.5) ![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)|
| Database | PostgreSQL ![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)|
| Build Tool | Gradle 	![Gradle](https://img.shields.io/badge/Gradle-02303A.svg?style=for-the-badge&logo=Gradle&logoColor=white)|
| Etc | Lombok, Spring Validation, DevTools ![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)|

---

## 3. 실행 방법

### 사전 요구사항

- Java 17 이상
- PostgreSQL 실행 중
- Gradle

### 1) 저장소 클론

```bash
git clone <repository-url>
cd music-playlist-backend
```

### 2) 데이터베이스 설정

`src/main/resources/application.properties` (또는 `application.yml`)에서 DB 연결 정보를 설정합니다.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/music_playlist
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
```

### 3) 빌드 및 실행

```bash
# Windows
./gradlew.bat bootRun

# Linux / macOS
./gradlew bootRun
```

서버는 기본적으로 `http://localhost:8080` 에서 실행됩니다.

---

## 4. 주요 기능

### 인증 (Auth)

| Method | URL | 설명 | 인증 필요 |
|--------|-----|------|-----------|
| POST | `/api/auth/signup` | 회원가입 | ❌ |
| POST | `/api/auth/login` | 로그인 (JWT 발급) | ❌ |
| POST | `/api/auth/logout` | 로그아웃 | ✅ |

#### 회원가입 요청
```json
{
  "username": "닉네임",
  "fullname": "이름",
  "email": "user@example.com",
  "password": "비밀번호"
}
```

#### 로그인 요청 / 응답
```json
// 요청
{
  "email": "user@example.com",
  "password": "비밀번호"
}

// 응답
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "user": {
    "id": 1,
    "username": "닉네임",
    "email": "user@example.com"
  }
}
```

### 유저 (User)

| Method | URL | 설명 | 인증 필요 |
|--------|-----|------|-----------|
| GET | `/api/users/{userId}` | 유저 정보 조회 | ✅ |
| PUT | `/api/users/{userId}` | 유저 정보 수정 | ✅ |

#### 유저 수정 요청
```json
{
  "username": "새닉네임",
  "fullName": "이름",
  "residentialArea": "서울",
  "selfIntroduction": "자기소개",
  "profileImageUrl": "https://..."
}
```

### 플레이리스트 (Playlist)

| Method | URL | 설명 | 인증 필요 |
|--------|-----|------|-----------|
| PUT | `/api/playlists/{playlistId}` | 플레이리스트 수정 | ✅ |
| DELETE | `/api/playlists/{playlistId}` | 플레이리스트 삭제 | ✅ |
| DELETE | `/api/playlists/{playlistId}/musics/{musicId}` | 플레이리스트에서 음악 제거 | ✅ |

---

## 5. 아키텍처

```
Client (React + Vite)
        │
        │ HTTP Request (JWT in Authorization Header)
        ▼
┌─────────────────────────────────────┐
│           Spring Boot App           │
│                                     │
│  Controller → Service → Repository  │
│                                     │
│  Security Filter (JWT 검증)          │
└─────────────────────────────────────┘
        │
        ▼
   PostgreSQL DB
```

### 패키지 구조

```
src/main/java/com/team10/music_playlist_backend/
├── config/         # Security, CORS 설정
├── controller/     # API 엔드포인트
├── dto/            # 요청/응답 데이터 객체
├── entity/         # JPA 엔티티 (User, Playlist)
├── exception/      # 전역 예외 처리
├── repository/     # DB 접근 레이어
├── security/       # JWT 유틸리티
└── service/        # 비즈니스 로직
```

### 인증 흐름

```
1. 클라이언트 → POST /api/auth/login
2. 서버: 이메일/비밀번호 검증 (BCrypt)
3. 서버: JWT Access Token 발급 (유효시간 1시간)
4. 클라이언트: 이후 요청 시 Authorization 헤더에 토큰 포함
5. 서버: Security Filter에서 토큰 검증 후 요청 처리
```

### 데이터베이스 구조

**users 테이블**

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | BIGINT (PK) | 자동 증가 |
| username | VARCHAR (unique) | 닉네임 |
| email | VARCHAR (unique) | 이메일 |
| password | VARCHAR | BCrypt 암호화 비밀번호 |
| full_name | VARCHAR | 실명 |
| self_introduction | TEXT | 자기소개 |
| residential_area | VARCHAR | 거주 지역 |
| profile_image_url | TEXT | 프로필 이미지 URL |
| provider | VARCHAR | 인증 제공자 (LOCAL/GOOGLE/GITHUB) |
| created_at | TIMESTAMP | 생성일시 |
| updated_at | TIMESTAMP | 수정일시 |

**playlists 테이블**

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | BIGINT (PK) | 자동 증가 |
| title | VARCHAR(100) | 플레이리스트 제목 |
| explanation | TEXT | 설명 |
| image_url | VARCHAR | 커버 이미지 URL |
| user_id | BIGINT (FK) | 소유자 유저 ID |

---

## 6. 배운 점

> 프로젝트를 진행하며 배운 점을 작성해주세요.

- Spring Security와 JWT를 직접 연동하며 인증/인가 흐름을 이해했습니다.
- JPA 연관관계 매핑(`@ManyToOne`, `@OneToMany`)과 지연 로딩(Lazy Loading)의 동작 방식을 학습했습니다.
- CORS 설정을 통해 프론트엔드와 백엔드 간 통신 문제를 해결하는 방법을 익혔습니다.
- `@Transactional`의 범위와 영속성 컨텍스트의 관계를 실습을 통해 이해했습니다.

---

## 7. 트러블슈팅

> 개발 중 겪은 문제와 해결 과정을 작성해주세요.

| 문제 | 원인 | 해결 방법 |
|------|------|-----------|
| CORS 오류 | 프론트(5173)와 백엔드(8080) 포트 불일치 | `SecurityConfig`에서 `CorsConfigurationSource` 설정으로 허용 Origin 추가 |
| JWT 검증 실패 | 토큰 만료 또는 잘못된 서명 | `JwtUtil.validateToken()`에서 예외 처리 후 `false` 반환 |
| 비밀번호 불일치 오류 | 평문 비밀번호 비교 | `BCryptPasswordEncoder.matches()` 사용으로 해결 |
