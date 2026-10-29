---
id: learn-09-database
title: 09 데이터베이스·SQLite 실제 사례와 Postgres 컨테이너, 볼륨, 백업
category: learn
summary: 파일 하나로 동작하는 SQLite(team-splitter가 실제로 사용)와 별도 서버로 도는 PostgreSQL의 차이, 컨테이너에서 DB 데이터를 볼륨으로 지키는 방법, DB 포트를 외부에 열지 않는 원칙, 백업 방법을 정리한다
keywords: [데이터베이스, DB, SQLite, WAL, PostgreSQL, Postgres, MySQL, 컨테이너 DB, 볼륨, named volume, 데이터 보존, 백업, pg_dump, sqlite3 .backup, DB 포트, "5432", depends_on, healthcheck, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 09 데이터베이스·SQLite 실제 사례와 Postgres 컨테이너, 볼륨, 백업

> 학습 시리즈: [08 Nginx](08-nginx-static-site.md) → **09** → [10 운영](10-operations.md)
> 상태: **SQLite는 운영 중(team-splitter), Postgres는 [예정]** (DB가 필요한 프로젝트가 생길 때)

## 1. 용어

| 용어 | 한 줄 정의 | 비유 |
|---|---|---|
| **데이터베이스 (DB)** | 데이터를 구조적으로 저장하고 검색하는 프로그램 | 서류 보관실 |
| **SQLite** | 별도 서버 없이 **파일 하나**가 DB인 방식. 앱 안에 라이브러리로 들어간다 | 책상 서랍 장부 |
| **PostgreSQL** | 별도 프로세스(서버)로 돌고, 여러 앱이 네트워크로 접속하는 DB | 전담 직원이 있는 문서고 |
| **WAL** | SQLite가 변경 내용을 먼저 적어 두는 보조 파일 (`*.db-wal`, `*.db-shm`) | 임시 메모장 |
| **named volume** | Docker가 관리하는 이름 붙은 저장 공간. `pgdata:/var/lib/postgresql/data` | Docker 전용 창고 |

## 2. 실제 사례: team-splitter의 SQLite

```
~/services/team-splitter/data/
├── team-splitter.db       ← DB 본체
├── team-splitter.db-wal   ← 쓰기 중 변경분
└── team-splitter.db-shm
```

- compose의 `./data:/app/data` 바인드 마운트로 서버 디스크에 있다. 컨테이너를 다시 만들어도 남는다.
- 봇 하나가 혼자 쓰는 작은 데이터라 SQLite로 충분하다.
- 주의: 실행 중에 `.db` 파일만 복사하면 WAL에 있는 최신 변경이 빠질 수 있다. 백업은 아래 방법으로 한다.

## 3. 언제 무엇을 쓰나

| 상황 | 선택 |
|---|---|
| DB가 필요 없음 (pont-coffee) | 없음 |
| 앱 하나가 혼자 쓰고, 동시 쓰기가 적음 | **SQLite** |
| 동시 사용자가 많거나, 여러 서비스가 같은 데이터를 씀 | **PostgreSQL** |

## 4. Postgres를 컨테이너로 붙일 때 [예정]

프로젝트 compose.yml 안에 앱과 함께 둔다. DB는 그 프로젝트 전용이다.

```yaml
services:
  app:
    build: .
    env_file: .env                    # DATABASE_URL=postgres://app:<비밀번호>@db:5432/app
    depends_on:
      db: { condition: service_healthy }
    networks: [default, proxy]        # 웹 앱만 proxy에도 붙는다
  db:
    image: postgres:17-alpine
    restart: unless-stopped
    env_file: .env.db                 # POSTGRES_PASSWORD 등
    volumes:
      - pgdata:/var/lib/postgresql/data
    mem_limit: 256m
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 10s
    # ports: 를 쓰지 않는다
volumes:
  pgdata:
networks:
  proxy:
    external: true
```

> 핵심 원칙
> - **DB 포트(5432)는 외부에 열지 않는다.** 앱은 같은 compose의 기본 네트워크에서 `db:5432`로 접속한다.
> - DB는 `proxy` 네트워크에 붙이지 않는다. 프록시와 다른 프로젝트가 DB에 닿을 수 없게 한다.
> - 비밀번호는 `.env`에만 두고 레포에 올리지 않는다.
> - Postgres는 메모리를 꽤 쓴다(기본 수십~수백 MB). 4GB 서버에서는 프로젝트마다 DB를 띄우면 금방 빠듯해진다.

## 5. 백업

```sh
# SQLite: 실행 중에도 안전한 복사 (sqlite3가 있는 곳에서)
sqlite3 data/team-splitter.db ".backup 'backup-YYYYMMDD.db'"

# Postgres: 덤프 파일로 내보내기
docker compose exec -T db pg_dump -U app app > backup-YYYYMMDD.sql
```

백업 파일은 서버 밖(맥, S3 등)에도 보관해야 서버가 망가졌을 때 살아남는다. 서버 전체 스냅샷은 [10 운영](10-operations.md).

## 다음

[10 운영](10-operations.md)
