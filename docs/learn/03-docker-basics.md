---
id: learn-03-docker-basics
title: 03 Docker 기초·설치, 이미지, 컨테이너, 볼륨, 네트워크
category: learn
summary: Ubuntu 서버에 get.docker.com 스크립트로 Docker를 설치하고 ubuntu 사용자를 docker 그룹에 넣은 과정과, Dockerfile·이미지·컨테이너·볼륨·포트·네트워크 개념을 건물과 방 비유로 정리한다
keywords: [Docker, 도커, 설치, get.docker.com, docker 그룹, usermod, hello-world, Dockerfile, 이미지, image, 컨테이너, container, 볼륨, 바인드 마운트, 포트, ports, 네트워크, network, 멀티 스테이지, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 03 Docker 기초·설치, 이미지, 컨테이너, 볼륨, 네트워크

> 학습 시리즈: [02 Lightsail](02-lightsail-instance.md) → **03** → [04 Compose](04-docker-compose.md)
> 상태: **완료** (2026-10-02 무렵)

## 1. 왜 Docker를 쓰나

서버 한 대에 프로젝트를 여러 개 올리면, 프로젝트마다 필요한 Node·Python 버전과 라이브러리가 서로 부딪친다.
Docker는 프로그램을 **실행 환경째로 포장**해서, 서로 섞이지 않는 칸에서 돌린다.

비유: **서버 = 건물, Docker = 서로 섞이지 않는 방을 찍어내는 시스템.**

## 2. 용어

| 용어 | 한 줄 정의 | 비유 |
|---|---|---|
| **Dockerfile** | 이미지를 만드는 레시피. "Node 22 설치 → 코드 복사 → 빌드" 순서를 적는다 | 방 설계도 |
| **이미지 (image)** | Dockerfile로 만든 완성품. 읽기 전용이고 바뀌지 않는다 | 조립식 방 키트 |
| **컨테이너 (container)** | 이미지를 실제로 실행한 것. 이미지 하나로 여러 개를 띄울 수 있다 | 키트로 지은 실제 방 |
| **레지스트리** | 이미지를 올리고 받는 저장소. Docker Hub(`nginx`, `caddy`, `node` 등 공식 이미지), GHCR | 키트 판매처 |
| **볼륨 / 바인드 마운트** | 서버 디스크의 폴더를 컨테이너 안에 연결한 것. `./data:/app/data` | 방 밖 창고. 방을 부숴도 짐은 남는다 |
| **포트 공개 (`-p`, `ports:`)** | 서버 포트와 컨테이너 포트를 연결. `"8080:3000"` = 서버 8080 → 컨테이너 3000 | 정문에서 방까지 바로 가는 복도 |
| **네트워크** | 컨테이너끼리 **이름으로** 서로 부를 수 있는 내부 통신망 | 건물 안 내선 전화 |

> 핵심 1: 컨테이너는 **언제든 지우고 다시 만드는 것**이다. 남겨야 할 데이터는 반드시 볼륨에 둔다.
> 핵심 2: 같은 네트워크에 있는 컨테이너는 `http://컨테이너이름:포트`로 서로 접근한다. 외부에 포트를 열 필요가 없다.

### Dockerfile 예시 (멀티 스테이지)

team-splitter가 실제로 쓰는 방식이다. 빌드 도구는 첫 번째 단계에만 두고, 실행 이미지는 가볍게 만든다.

```dockerfile
FROM node:22-bookworm-slim AS build     # 1단계: 빌드용
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY src ./src
RUN npm run build && npm prune --omit=dev

FROM node:22-bookworm-slim              # 2단계: 실행용 (결과물만 복사)
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
USER node                               # root가 아닌 사용자로 실행
CMD ["node", "dist/index.js"]
```

## 3. 실제로 한 일: 설치

**[맥]** `ssh aws-ubuntu-1`로 접속한 뒤 **[서버]**에서 실행한다.

```sh
ssh aws-ubuntu-1                              # [맥]
curl -fsSL https://get.docker.com | sudo sh   # Docker 공식 설치 스크립트
sudo usermod -aG docker ubuntu                # sudo 없이 docker 쓰도록 그룹 추가
exit                                          # 그룹 적용을 위해 재접속
ssh aws-ubuntu-1
docker run --rm hello-world                   # 설치 확인
mkdir -p ~/services                           # 모든 프로젝트를 둘 폴더
```

- `usermod` 뒤에는 **재접속해야** 그룹이 적용된다. 안 하면 `permission denied ... docker.sock` 오류가 난다.
- `docker run --rm hello-world`가 "Hello from Docker!"를 출력하면 성공이다.

## 4. 확인

**[서버]**

```sh
docker --version                 # Docker version 29.8.2 (2026-10-27 확인)
systemctl is-enabled docker      # enabled → 서버 재부팅 시 Docker 자동 시작
docker ps                        # 실행 중인 컨테이너
docker images                    # 받아 둔 이미지
```

## 5. 자주 쓰는 명령

| 명령 | 하는 일 |
|---|---|
| `docker ps -a` | 멈춘 것까지 모든 컨테이너 |
| `docker logs -f <컨테이너>` | 로그 실시간 보기 |
| `docker exec -it <컨테이너> sh` | 컨테이너 안으로 들어가기 |
| `docker stats --no-stream` | 컨테이너별 CPU·메모리 사용량 |
| `docker network ls` | 네트워크 목록 |
| `docker system df` | 이미지·컨테이너·빌드 캐시가 차지하는 디스크 |

컨테이너를 하나씩 `docker run`으로 띄우면 옵션이 길어진다. 그래서 다음 단계에서 Compose를 쓴다.

## 다음

[04 Docker Compose로 프로젝트 가동](04-docker-compose.md)
