# 1단계: 프론트엔드(React) 빌드 환경
FROM node:20-alpine AS frontend-build
WORKDIR /app/frontend

# 💡 Render가 e-commerce 폴더 내부에서 빌드를 시작하므로
# 💡 현재 위치(.)에 있는 frontend 폴더를 그대로 바라봅니다.
COPY frontend/package*.json ./
RUN npm ci

COPY frontend/ ./
RUN npm run build

# 2단계: 백엔드(Java Gradle) 빌드 환경
FROM gradle:8-jdk21 AS backend-build
WORKDIR /app/backend

# e-commerce 안의 모든 것(backend, frontend 등)을 복사
COPY . .

# 1단계에서 빌드된 React 결과물(build)을 스프링 부트 정적 자원 폴더(static)로 복사
# backend 폴더가 소문자 나란히 있으므로 아래 경로가 정확합니다.
COPY --from=frontend-build /app/frontend/build ./backend/src/main/resources/static/

# Gradle 빌드 수행을 위해 소문자 backend 폴더 안으로 진입
WORKDIR /app/backend/backend
RUN ./gradlew build -x test --no-daemon

# 3단계: 최종 실행 환경 (안정적인 런타임 이미지)
FROM eclipse-temurin:21-jre-jammy
EXPOSE 8080

# 빌드된 jar 파일을 실행 파일로 복사
COPY --from=backend-build /app/backend/backend/build/libs/*-SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]