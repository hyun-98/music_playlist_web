# 🎵 Music Playlist Backend

음악 플레이리스트 웹 서비스의 백엔드 API 서버입니다.  
Spring Boot 기반으로 구축되었으며, JWT 인증과 PostgreSQL을 사용합니다.

---

## 🛠 기술 스택

| 분류 | 기술 |
|------|------|
| Language | Java 17 |
| Framework | Spring Boot 3.5.6 |
| ORM | Spring Data JPA (Hibernate) |
| Security | Spring Security + JWT (jjwt 0.11.5) |
| Database | PostgreSQL |
| Build Tool | Gradle |
| Etc | Lombok, Spring Validation, DevTools |

---

## 📁 프로젝트 구조

```
src/main/java/com/team10/music_playlist_backend/
├── config/
│   ├── ApplicationConfig.java       # 애플리케이션 설정
│   └── SecurityConfig.java          # Spring Security & CORS 설정
├── controller/
│   ├── AuthController.java          # 인증 API (회원가입, 로그인, 로그아웃)
│   └── UserController.java          # 유저 API (조회, 수정)
├── dto/
│   ├── LoginResponse.java           # 로그인 응답 DTO
│   ├── PlaylistEditRequest.java     # 플레이리스트 수정 요청 DTO
│   ├── PlaylistRequest.java         # 플레이리스트 생성 요청 DTO
│   ├── PlaylistResponse.java        # 플레이리스트 응답 DTO
│   ├── UserDto.java                 # 유저 기본 DTO
│   ├── UserRequest.java             # 유저 요청 DTO
│   ├── UserResponse.java            # 유저 응답 DTO
│   └── UserUpdateRequest.java       # 유저 수정 요청 DTO
├── entity/
│   ├── AuthProvider.java            # 인증 제공자 Enum (LOCAL, GOOGLE, GITHUB)
│   ├── Playlist.java                # 플레이리스트 엔티티
│   └── User.java                    # 유저 엔티티
├── exception/
│   ├── GlobalExceptionHandler.java  # 전역 예외 처리
│   └── ResourceNotFoundException.java
├── repository/
│   ├── PlaylistRepository.java
│   └── UserRepository.java
├── security/
│   └── JwtUtil.java                 # JWT 토큰 생성 및 검증
└── service/
    ├── AuthService.java             # 인증 비즈니스 로직
    ├── PlaylistService.java         # 플레이리스트 비즈니스 로직
    └── UserService.java             # 유저 비즈니스 로직
```

---

## 🔐 인증 방식

- **JWT (JSON Web Token)** 기반 인증
- 토큰 유효 시간: **1시간**
- 요청 시 `Authorization` 헤더에 토큰 포함
- 비밀번호는 **BCrypt** 로 암호화하여 저장

---

## 📡 API 엔드포인트

### Auth API (`/api/auth`)

| Method | URL | 설명 | 인증 필요 |
|--------|-----|------|-----------|
| POST | `/api/auth/signup` | 회원가입 | ❌ |
| POST | `/api/auth/login` | 로그인 | ❌ |
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

#### 로그인 요청
```json
{
  "email": "user@example.com",
  "password": "비밀번호"
}
```

#### 로그인 응답
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "user": {
    "id": 1,
    "username": "닉네임",
    "email": "user@example.com"
  }
}
```

---

### User API (`/api/users`)

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

---

## 🗄 데이터베이스 구조

### users 테이블

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
| provider_id | VARCHAR | OAuth 제공자 ID |
| created_at | TIMESTAMP | 생성일시 |
| updated_at | TIMESTAMP | 수정일시 |

### playlists 테이블

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | BIGINT (PK) | 자동 증가 |
| title | VARCHAR(100) | 플레이리스트 제목 |
| explanation | TEXT | 설명 |
| image_url | VARCHAR | 커버 이미지 URL |
| user_id | BIGINT (FK) | 소유자 유저 ID |

---

## ⚙️ 실행 방법

### 사전 요구사항

- Java 17 이상
- PostgreSQL 실행 중
- Gradle

### 1. 저장소 클론

```bash
git clone <repository-url>
cd music-playlist-backend
```

### 2. 데이터베이스 설정

`src/main/resources/application.properties` (또는 `application.yml`) 에서 DB 연결 정보를 설정합니다.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/music_playlist
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
```

### 3. 빌드 및 실행

```bash
# Windows
./gradlew.bat bootRun

# Linux / macOS
./gradlew bootRun
```

서버는 기본적으로 `http://localhost:8080` 에서 실행됩니다.

---

## 🌐 CORS 설정

프론트엔드 개발 서버(`http://localhost:5173`)에서의 요청을 허용합니다.  
허용 메서드: `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`

---

## 👥 팀

**Team 10** — music_playlist_web 프로젝트
