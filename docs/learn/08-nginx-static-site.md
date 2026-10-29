---
id: learn-08-nginx-static-site
title: 08 Nginx 정적 사이트·pont-coffee를 컨테이너로 띄우고 프록시에 연결
category: learn
summary: DB 없는 단순 웹사이트를 Nginx 컨테이너로 서빙하는 방법을 빌드 없는 순수 HTML 방식과 Vite 같은 빌드 도구 방식(멀티 스테이지)으로 나눠 설명하고, pont-coffee를 proxy 네트워크에 붙여 pont-coffee.hwiii.com으로 여는 계획을 정리한다
keywords: [Nginx, 정적 사이트, static site, pont-coffee, HTML, nginx:alpine, 웹서버, 바인드 마운트, 읽기 전용, ro, Vite, 빌드, 멀티 스테이지, dist, proxy 네트워크, mem_limit, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 08 Nginx 정적 사이트·pont-coffee를 컨테이너로 띄우고 프록시에 연결

> 학습 시리즈: [07 리버스 프록시](07-reverse-proxy-https.md) → **08** → [09 데이터베이스](09-database.md)
> 상태: **[예정]** pont-coffee는 DB가 필요 없는 단순 웹사이트다.

## 1. 용어

| 용어 | 한 줄 정의 |
|---|---|
| **정적 사이트** | 요청마다 계산하지 않고, 미리 만든 HTML·CSS·JS·이미지 파일을 그대로 보내는 사이트 |
| **동적 사이트** | 요청마다 서버 코드가 돌아 결과를 만든다 (로그인, 게시판 등). 보통 DB가 붙는다 |
| **웹서버** | 파일을 HTTP로 내보내는 프로그램. Nginx, Caddy, Apache |
| **Nginx** | 가장 널리 쓰이는 웹서버. 가볍고 빠르다. `nginx:alpine` 이미지는 메모리 10~20MB 수준 |

**Caddy와 Nginx 역할 나누기:** Caddy는 입구(프록시, HTTPS)를 맡고, Nginx는 각 사이트 안에서 파일을 내보낸다.
Caddy가 파일을 직접 서빙할 수도 있지만, 그러면 사이트가 프록시 설정에 묶여 "프로젝트마다 폴더와 compose를 따로 둔다"는 규칙이 깨진다.

## 2. 방식 A: 순수 HTML (빌드 없음)

```
~/services/pont-coffee/
├── compose.yml
└── public/
    ├── index.html
    ├── css/
    └── images/
```

```yaml
services:
  web:
    image: nginx:alpine
    restart: unless-stopped
    volumes:
      - ./public:/usr/share/nginx/html:ro   # ro = 읽기 전용
    mem_limit: 64m
    logging:
      options: { max-size: 10m, max-file: "3" }
    networks: [proxy]
networks:
  proxy:
    external: true
```

- Dockerfile이 필요 없다.
- 업데이트는 `git pull`만 하면 된다. 폴더를 연결해 둔 것이라 재시작 없이 바로 반영된다.

## 3. 방식 B: 빌드 도구 사용 (Vite, Astro, React 등)

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build            # → dist/

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

- compose.yml은 A에서 `image:`와 `volumes:`를 빼고 `build: .`로 바꾼다.
- 업데이트: `git pull && docker compose up -d --build`
- 빌드하는 동안 Node가 메모리를 많이 쓴다 → [10 운영](10-operations.md)의 스왑 항목.

## 4. 공통 포인트

- `ports:`를 쓰지 않는다. 외부 연결은 Caddy가 `proxy` 네트워크로 해 준다.
- 컨테이너 이름은 `폴더명-서비스명-1` → `pont-coffee-web-1`. Caddyfile에 이 이름을 쓴다.

## 5. 실제로 한 일 [예정]

```sh
# 1) 레포 만들기 (A 또는 B) → GitHub
# 2) 서버
ssh aws-ubuntu-1
cd ~/services && git clone <pont-coffee 레포> pont-coffee
cd pont-coffee && docker compose up -d
# 3) _proxy/Caddyfile에 pont-coffee 블록 추가 → reload ([07](07-reverse-proxy-https.md))
# 4) 확인
curl -I https://pont-coffee.hwiii.com
```

## 다음

[09 데이터베이스](09-database.md)
