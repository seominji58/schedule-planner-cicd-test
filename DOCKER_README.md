# Docker 배포 가이드

이 프로젝트는 Docker를 사용하여 프론트엔드(Next.js)와 백엔드(Node.js)를 컨테이너로 실행할 수 있습니다.

## 프로젝트 구조

```
schedule-planner-main/
├── frontend/          # Next.js 프론트엔드 (포트 3000)
│   ├── Dockerfile     # 프로덕션용 Dockerfile
│   └── Dockerfile.dev # 개발용 Dockerfile
├── backend/           # Node.js 백엔드 (포트 3001)
│   ├── Dockerfile     # 프로덕션용 Dockerfile
│   └── Dockerfile.dev # 개발용 Dockerfile
├── docker-compose.yml # 프로덕션용 Docker Compose
├── docker-compose.dev.yml # 개발용 Docker Compose
├── .dockerignore      # Docker 빌드 시 제외할 파일들
├── env.example        # 환경변수 예시 파일
└── DOCKER_README.md   # 이 파일
```

## 환경 변수

### .env 파일 설정
프로젝트 루트 디렉토리에 `.env` 파일을 생성하여 환경변수를 설정할 수 있습니다.

```bash
# .env 파일 생성
cp env.example .env
# 또는 직접 .env 파일을 생성하고 필요한 환경변수를 설정
```

### .dockerignore 파일
Docker 이미지 빌드 시 불필요한 파일들이 포함되지 않도록 `.dockerignore` 파일이 설정되어 있습니다:
- `node_modules/`, `.next/`, `dist/` 등 빌드 결과물
- `.env*` 파일들 (보안상 제외)
- `.git/`, `docs/` 등 개발 관련 파일들
- IDE 설정 파일들

### Frontend 환경 변수
- `BACKEND_URL`: 백엔드 API 서버 주소 (기본값: http://localhost:3001)
- `NEXT_PUBLIC_API_URL`: 클라이언트에서 접근할 API URL (기본값: http://localhost:3001)

### Backend 환경 변수
- `PORT`: 서버 포트 (기본값: 3001)
- `NODE_ENV`: 실행 환경 (기본값: production)
- `CORS_ORIGIN`: CORS 허용 도메인 (기본값: http://localhost:3000)

### 공통 환경 변수 (docker-compose에서 자동 주입)
- `JWT_SECRET`: JWT 토큰 시크릿 키
- `FIREBASE_PROJECT_ID`: Firebase 프로젝트 ID
- `GOOGLE_CLIENT_ID`: Google OAuth 클라이언트 ID
- `OPENAI_API_KEY`: OpenAI API 키
- 기타 데이터베이스, 이메일, 파일 저장소 관련 설정

## 빌드 및 실행

### 1. 전체 애플리케이션 실행
```bash
# 모든 서비스 빌드 및 실행
docker-compose up --build

# 백그라운드에서 실행
docker-compose up -d --build
```

### 2. 개별 서비스 실행
```bash
# 프론트엔드만 실행
docker-compose up frontend

# 백엔드만 실행
docker-compose up backend
```

### 3. 서비스 중지
```bash
# 모든 서비스 중지
docker-compose down

# 볼륨까지 삭제
docker-compose down -v
```

## 접속 주소

- **프론트엔드**: http://localhost:3000
- **백엔드 API**: http://localhost:3001
- **헬스 체크**: http://localhost:3001/health

## Jenkins 배포 설정

Jenkins에서 이 프로젝트를 배포할 때는 다음과 같이 설정하세요:

### 1. Jenkins Job 설정
```bash
# 빌드 스크립트
docker-compose build

# 배포 스크립트
docker-compose up -d
```

### 2. 환경 변수 설정
Jenkins에서 다음 환경 변수를 설정하세요:

#### 방법 1: .env 파일 사용 (권장)
```bash
# Jenkins 서버에 .env 파일을 업로드하거나 생성
# docker-compose.yml에서 자동으로 .env 파일을 읽어서 환경변수 주입
```

#### 방법 2: Jenkins 환경변수 직접 설정
- `BACKEND_URL`: 실제 백엔드 서버 주소
- `NEXT_PUBLIC_API_URL`: 클라이언트에서 접근할 API URL
- `JWT_SECRET`: JWT 토큰 시크릿 키
- `FIREBASE_PROJECT_ID`: Firebase 프로젝트 ID
- 기타 필요한 환경변수들

## 문제 해결

### 1. 포트 충돌
포트가 이미 사용 중인 경우:
```bash
# 사용 중인 포트 확인
netstat -tulpn | grep :3000
netstat -tulpn | grep :3001

# 컨테이너 중지
docker-compose down
```

### 2. 빌드 오류
```bash
# 캐시 삭제 후 재빌드
docker-compose build --no-cache

# 이미지 삭제 후 재빌드
docker system prune -a
docker-compose up --build
```

### 3. 로그 확인
```bash
# 모든 서비스 로그
docker-compose logs

# 특정 서비스 로그
docker-compose logs frontend
docker-compose logs backend

# 실시간 로그
docker-compose logs -f
```

## 개발 환경

개발 환경에서는 다음과 같이 실행할 수 있습니다:

```bash
# 개발 모드로 실행 (볼륨 마운트)
docker-compose -f docker-compose.dev.yml up
```

## 보안 고려사항

1. **환경 변수**: 민감한 정보는 환경 변수로 관리
2. **네트워크**: 컨테이너 간 통신은 내부 네트워크 사용
3. **사용자**: 컨테이너 내에서 root가 아닌 사용자로 실행
4. **헬스 체크**: 컨테이너 상태 모니터링

## 성능 최적화

1. **멀티스테이지 빌드**: 불필요한 파일 제거
2. **캐시 활용**: Docker 레이어 캐시 사용
3. **이미지 크기**: Alpine Linux 사용으로 이미지 크기 최소화
4. **메모리 제한**: 컨테이너별 메모리 제한 설정 