# aws-ubuntu

AWS Lightsail 서울 서버 한 대(`aws-ubuntu-1`)에 Docker로 여러 프로젝트를 올리고, `서비스명.hwiii.com` 형식으로 연결하는 과정을 **학습 기록**과 **운영 문서**로 남기는 레포다.
서버 IP, SSH 키, 토큰 같은 민감정보는 쓰지 않는다 (`<SERVER_IP>`, `<KEY>`로 표시).

## 현재 상태

- 서버: Lightsail 서울, Ubuntu 24.04, 4GB 플랜, Docker + Compose
- 운영 중: `team-splitter` (디스코드 봇) 1개
- 도메인: `hwiii.com` (가비아), `*`·`@` A 레코드 연결 완료
- 프록시: `_proxy`(Caddy) 가동, 인증서 자동 발급 확인. 스왑 2GB 추가
- 다음 단계: Lightsail 443 열기 → 첫 웹사이트 `pont-coffee`

## 학습 순서

| # | 문서 | 상태 |
|---|---|---|
| 01 | [서버 개념](docs/learn/01-server-concepts.md) | 개념 |
| 02 | [Lightsail 인스턴스·고정 IP·SSH](docs/learn/02-lightsail-instance.md) | 완료 |
| 03 | [Docker 기초](docs/learn/03-docker-basics.md) | 완료 |
| 04 | [Docker Compose로 프로젝트 가동](docs/learn/04-docker-compose.md) | 완료 |
| 05 | [도메인과 DNS](docs/learn/05-domain-dns.md) | 완료 |
| 06 | [방화벽과 포트](docs/learn/06-firewall-ports.md) | Lightsail 443 열기 예정 |
| 07 | [리버스 프록시와 HTTPS](docs/learn/07-reverse-proxy-https.md) | 서버 구성 완료 |
| 08 | [Nginx 정적 사이트](docs/learn/08-nginx-static-site.md) | 예정 |
| 09 | [데이터베이스](docs/learn/09-database.md) | SQLite 운영 중, Postgres 예정 |
| 10 | [운영](docs/learn/10-operations.md) | 스왑 완료, 나머지 예정 |

새 작업을 하면 `11-...`부터 이어서 쌓는다.

## 운영 문서

- [proxy/](proxy/): 서버 `~/services/_proxy`에 올라가는 Caddy 설정 원본

- ops: [서버 현황·사양과 services 구조](docs/ops/server-overview.md)
- guide: [새 서비스 추가 가이드](docs/guide/add-service.md)

전체 목록은 [INDEX.md](INDEX.md)를 본다.
