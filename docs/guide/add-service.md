---
id: guide-add-service
title: 새 서비스 추가 가이드·services 폴더 규칙과 체크리스트
category: guide
summary: ~/services에 새 프로젝트를 Docker Compose로 올릴 때 지킬 폴더·이름·비밀값·데이터·로그·포트 규칙과, 서비스가 늘기 전에 서버에 해둘 사전 준비(스왑, 로그 기본값, 리버스 프록시)를 체크리스트로 정리한다
keywords: [새 서비스, 프로젝트 추가, 배포, services, Docker Compose, compose.yml, 폴더 규칙, 프로젝트 이름, name, .env, data, 볼륨, 로그 회전, logging, mem_limit, 메모리 제한, 포트, 리버스 프록시, Caddy, Traefik, 스왑, daemon.json, builder prune]
related_files: []
last_updated: "2026-10-27"
---
# 새 서비스 추가 가이드·services 폴더 규칙과 체크리스트

현재 서버 상태는 [서버 현황](../ops/server-overview.md)을 본다. 아래 규칙은 이미 돌고 있는 `team-splitter`의 방식을 기준으로 정리했다.

## 1. 폴더와 이름 규칙

- `~/services/<프로젝트명>/` **한 단계**로 둔다. 폴더 하나에 compose.yml 하나를 둔다.
- 폴더 이름이 compose 프로젝트 이름이 된다. 한 단계 더 넣으면(`services/university/cloud`) 이름이 `cloud`가 되어 다른 프로젝트와 겹칠 수 있다. 꼭 넣어야 하면 compose.yml 맨 위에 `name: <고유이름>`을 쓴다.
- 코드는 GitHub에서 `git clone`해서 받는다. 업데이트는 `git pull && docker compose up -d --build`.

## 2. compose.yml에 넣을 것

```yaml
services:
  app:
    build: .
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./data:/app/data
    mem_limit: 512m          # 서비스 성격에 맞게
    logging:
      options:
        max-size: 10m
        max-file: "3"
```

| 항목 | 규칙 | 이유 |
|---|---|---|
| `restart` | `unless-stopped` | 재부팅·크래시 후 자동 복구 |
| `.env` | 서버 폴더에만 둔다. `.gitignore`와 `.dockerignore`에 포함 | 비밀값이 레포·이미지에 들어가지 않게 |
| 데이터 | `./data` 바인드 마운트, git 제외 | 컨테이너를 다시 만들어도 데이터 유지 |
| `logging` | 회전 설정 필수 (daemon 기본값이 생기기 전까지) | 디스크가 가득 차는 것 방지 |
| `mem_limit` | 지정 | 한 서비스가 서버 전체 메모리를 먹는 것 방지 |
| `ports` | 가능하면 쓰지 않는다. 필요하면 `127.0.0.1:<포트>:<포트>` | ufw가 꺼져 있고 포트가 겹치는 것 방지 |

## 3. 서비스가 늘기 전에 서버에 해둘 것

아직 적용하지 않은 항목이다. 적용하면 [서버 현황](../ops/server-overview.md)의 위험 요소 표를 갱신한다.

- [ ] **스왑 2GB 추가**: 빌드할 때 메모리가 부족하지 않게.
- [ ] **`/etc/docker/daemon.json` 로그 기본값**: `{"log-driver":"json-file","log-opts":{"max-size":"10m","max-file":"3"}}`. 적용하려면 docker를 재시작해야 하므로 모든 컨테이너가 잠깐 멈춘다.
- [ ] **리버스 프록시** (웹 서비스가 처음 들어올 때): `~/services/_proxy/`에 Caddy나 Traefik을 두고, 외부 네트워크(예: `proxy`)를 하나 만들어 웹 서비스가 그 네트워크에 붙도록 한다. 80/443은 프록시만 연다. Lightsail 방화벽에서도 80/443을 연다.
- [ ] **빌드 캐시 정리**: 가끔 `docker builder prune` 실행.
- [ ] **데이터 백업**: Lightsail 자동 스냅샷을 켜거나 `data/`를 주기적으로 복사.

## 4. 새 서비스 올리기 체크리스트

1. [ ] `ssh aws-ubuntu-1` → `cd ~/services && git clone <repo> <프로젝트명>`
2. [ ] 맥에서 `.env`를 `scp`로 올린다 (레포에는 `.env.example`만 둔다)
3. [ ] `mkdir -p data`
4. [ ] compose.yml에 위 2장 항목이 다 있는지 확인
5. [ ] `docker compose up -d --build`
6. [ ] `docker compose ps`, `docker compose logs -f --tail 50`으로 확인
7. [ ] `free -h`로 남은 메모리 확인
8. [ ] [서버 현황](../ops/server-overview.md)의 폴더 구조·실행 중인 것 표에 추가하고 INDEX 갱신
