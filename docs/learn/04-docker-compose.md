---
id: learn-04-docker-compose
title: 04 Docker Compose로 프로젝트 가동·team-splitter 배포 기록
category: learn
summary: compose.yml로 컨테이너 설정을 파일에 적어 관리하는 방법과 프로젝트 이름·.env·restart·로그 회전 개념을 설명하고, 디스코드 봇 team-splitter를 ~/services에 git clone, .env 업로드, data 폴더 준비, docker compose up으로 가동한 실제 과정을 기록한다
keywords: [Docker Compose, compose.yml, docker compose up, 프로젝트 이름, service, .env, env_file, scp, restart, unless-stopped, logging, 로그 회전, 볼륨, data, team-splitter, 배포, 업데이트, git pull, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 04 Docker Compose로 프로젝트 가동·team-splitter 배포 기록

> 학습 시리즈: [03 Docker](03-docker-basics.md) → **04** → [05 도메인·DNS](05-domain-dns.md)
> 상태: **완료** (2026-10-02 무렵, 현재 가동 중)

## 1. 용어

| 용어 | 한 줄 정의 | 비유 |
|---|---|---|
| **Docker Compose** | 컨테이너 설정을 파일 하나에 적어 두고 명령 한 줄로 올리고 내리는 도구 | 방 배치표 |
| **compose.yml** | 그 설정 파일 | 배치표 종이 |
| **service** | compose.yml 안의 컨테이너 단위 이름 (`bot`, `web`, `db`) | 배치표의 방 이름 |
| **프로젝트 이름** | compose.yml이 있는 **폴더 이름**. 컨테이너 이름이 `<프로젝트>-<service>-1`이 된다 | 층 이름 |
| **.env** | 토큰·비밀번호 같은 비밀값 파일. git에 올리지 않는다 | 방마다 따로 둔 금고 |
| **restart: unless-stopped** | 죽거나 서버가 재부팅돼도 자동으로 다시 켠다 (직접 멈춘 경우는 제외) | 정전 후 자동 복구 |
| **logging** | 로그 파일 크기 제한. 없으면 로그가 디스크를 채운다 | 쓰레기통 크기 제한 |
| **mem_limit** | 컨테이너 하나의 메모리 상한 | 방마다 전기 차단기 |

## 2. 서버 폴더 규칙

```
~/services/
└── <프로젝트>/          ← 폴더 하나 = compose.yml 하나 = 프로젝트 하나
    ├── compose.yml
    ├── .env             ← 서버에만 있음
    └── data/            ← 남겨야 할 데이터
```

폴더가 다르면 Compose가 네트워크와 컨테이너 이름을 따로 만들어서 프로젝트끼리 섞이지 않는다.
예: team-splitter 폴더 → 네트워크 `team-splitter_default`, 컨테이너 `team-splitter-bot-1`.

## 3. 실제 compose.yml (team-splitter)

```yaml
services:
  bot:
    build: .                    # 같은 폴더의 Dockerfile로 이미지 빌드
    restart: unless-stopped
    env_file: .env              # 비밀값은 파일에서 읽음
    environment:
      DB_PATH: /app/data/team-splitter.db
    volumes:
      - ./data:/app/data        # SQLite DB를 서버 디스크에 보관
    logging:
      options:
        max-size: 10m
        max-file: "3"
```

이 봇은 디스코드 서버에 **바깥으로** 연결하는 방식이라 `ports:`가 없다. 외부에서 들어올 문이 필요 없다.

## 4. 실제로 한 일: 배포

```sh
# [맥] 접속 확인
ssh aws-ubuntu-1 'whoami && docker compose version && ls ~/services'

# [서버] 코드 받기
ssh aws-ubuntu-1
cd ~/services
git clone https://github.com/gyeolhwi/team-splitter.git
cd team-splitter
mkdir -p data          # docker 실행 전에 꼭 만든다
exit

# [맥] 비밀값 파일 올리기 (화면·채팅에 토큰을 붙여넣지 않고 파일째 보낸다)
scp .env.local aws-ubuntu-1:services/team-splitter/.env
ssh aws-ubuntu-1 'chmod 600 ~/services/team-splitter/.env && cut -d= -f1 ~/services/team-splitter/.env'
# → 키 이름만 출력되고 값은 보이지 않는다

# [서버] 빌드하고 실행
ssh aws-ubuntu-1 'cd ~/services/team-splitter && docker compose up -d --build'
```

## 5. 확인

```sh
docker compose ps                 # STATUS가 Up
docker compose logs -f --tail 50  # 정상 로그 확인 후 Ctrl+C
```

## 6. 겪은 문제와 이유

| 증상 | 원인 | 해결 |
|---|---|---|
| 봇이 DB를 못 만들고 재시작을 반복 | `data/`가 없으면 Docker가 **root 소유**로 만든다. 컨테이너는 `node` 사용자라 쓸 수 없다 | 실행 전에 `mkdir -p data` |
| 명령이 두 번 처리되며 꼬임 | 맥에서 `npm run dev`로 켠 봇과 서버 봇이 동시에 실행 | 서버에 올리기 전에 로컬 봇을 끈다 |

## 7. 업데이트와 자주 쓰는 명령

```sh
# 코드가 바뀌었을 때
ssh aws-ubuntu-1 'cd ~/services/team-splitter && git pull && docker compose up -d --build'
```

| 명령 | 하는 일 |
|---|---|
| `docker compose up -d --build` | 이미지를 다시 빌드하고 백그라운드로 실행 |
| `docker compose ps` | 상태 확인 |
| `docker compose logs -f` | 로그 보기 |
| `docker compose restart` | 재시작 |
| `docker compose down` | 멈추고 컨테이너 삭제 (볼륨 데이터는 남는다) |

새 프로젝트를 추가할 때의 규칙과 체크리스트는 [새 서비스 추가 가이드](../guide/add-service.md).

## 다음

[05 도메인과 DNS](05-domain-dns.md)
