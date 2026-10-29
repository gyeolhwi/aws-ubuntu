---
id: ops-server-overview
title: 서버 현황·aws-ubuntu-1 사양과 services 구조
category: ops
summary: Lightsail 서울 서버 aws-ubuntu-1의 접속 방법, OS·Docker 버전, 메모리·디스크, ~/services 폴더 구조, 실행 중인 컨테이너와 네트워크, 확인된 운영 위험 요소를 정리한다
keywords: [서버, aws-ubuntu-1, Lightsail, SSH, 접속, 사양, 메모리, 스왑, 디스크, Docker, Docker Compose, services, team-splitter, 컨테이너, 네트워크, 포트, ufw, 방화벽, 로그, daemon.json, 빌드 캐시, 백업]
related_files: []
last_updated: "2026-10-27"
---
# 서버 현황·aws-ubuntu-1 사양과 services 구조

2026-10-27에 서버에 접속해 직접 확인한 상태다. 값이 바뀌면 이 문서를 갱신한다.

## 1. 접속

맥의 `~/.ssh/config`에 별칭이 잡혀 있다.

```sh
ssh aws-ubuntu-1
```

| 항목 | 값 |
|---|---|
| 별칭 | `aws-ubuntu-1` |
| 사용자 | `ubuntu` (docker 그룹 포함, sudo 가능) |
| 키 | `~/.ssh/<KEY>.pem` (맥에만 있음, 레포에 쓰지 않음) |
| 고정 IP | `<SERVER_IP>` (`~/.ssh/config`의 `HostName`) |

## 2. 사양과 소프트웨어

| 항목 | 값 |
|---|---|
| 호스팅 | AWS Lightsail 서울 |
| OS | Ubuntu 24.04.4 LTS (커널 7.0.0-aws) |
| CPU | 2 vCPU |
| 메모리 | 3.7GiB, **스왑 없음** |
| 디스크 | 77G 중 7G 사용 (10%) |
| Docker | 29.8.2, Compose v5.5.1, `systemctl` enabled (부팅 시 자동 시작) |
| git | 2.43.0 |

Docker는 `get.docker.com` 스크립트로 설치했고 `ubuntu` 사용자를 docker 그룹에 넣었다.

## 3. 폴더 구조

```text
/home/ubuntu/
└── services/                 # 모든 프로젝트를 여기에 둔다
    ├── team-splitter/        # 디스코드 봇(배근이). GitHub gyeolhwi/team-splitter clone
    │   ├── compose.yml       # 서비스 bot, restart unless-stopped, 로그 10m×3
    │   ├── .env              # 비밀값 (git 제외)
    │   └── data/             # SQLite DB (./data:/app/data 바인드 마운트)
    └── university/cloud/docs/  # 빈 폴더, 서비스 아님
```

프로젝트별 배포·업데이트 절차는 각 저장소 문서를 따른다. team-splitter는 레포 안 `docs/deploy.md`, `docs/docker-guide.md`.

## 4. 실행 중인 것

| 종류 | 이름 | 비고 |
|---|---|---|
| 컨테이너 | `team-splitter-bot-1` | 외부 공개 포트 없음 (디스코드 게이트웨이로 바깥에 연결) |
| 네트워크 | `team-splitter_default` | compose가 프로젝트별로 자동 생성 |
| 호스트 포트 | 22 (sshd) | 그 밖에 127.0.0.1에서만 열린 포트: 53 (systemd-resolved), herdr-web-ui(bun) |

## 5. 격리 방식

- Compose 프로젝트 이름은 **compose.yml이 있는 폴더 이름**이 된다. 폴더가 다르면 네트워크·컨테이너 이름이 따로 생겨 서로 섞이지 않는다.
- 비밀값(`.env`)과 데이터(`data/`)는 프로젝트 폴더 안에 있고, git과 이미지에서 제외되어 있다.

## 6. 확인된 위험 요소

| 항목 | 현재 상태 | 영향 |
|---|---|---|
| 메모리 | 3.7GiB, 스왑 없음, 컨테이너 메모리 제한 없음 | 서버에서 직접 빌드(npm ci, g++)하거나 서비스가 늘면 OOM 위험 |
| 웹 입구 | 리버스 프록시 없음 | 웹 서비스가 둘 이상 80/443을 쓰면 포트가 충돌 |
| 방화벽 | ufw inactive, Lightsail 방화벽만 사용 | Docker로 공개한 포트는 Lightsail 규칙만으로 막힘 |
| 로그 기본값 | `/etc/docker/daemon.json` 없음, 회전은 compose별 설정 | 새 프로젝트가 logging 설정을 빠뜨리면 로그가 무제한으로 쌓임 |
| 빌드 캐시 | 2.86GB (2.46GB는 지워도 됨) | 빌드할 때마다 쌓임 |
| 비밀값 백업 파일 | `team-splitter/.env.bak-20261027`이 git 추적 밖에 있음 | `.gitignore`에 걸리지 않아 실수로 커밋될 수 있음 |
| 데이터 백업 | SQLite가 서버에만 있음. 스냅샷 여부는 미확인 | 서버가 망가지면 데이터 유실 |

해결 방향은 [새 서비스 추가 가이드](../guide/add-service.md)의 사전 준비 항목에 있다.
