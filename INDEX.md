# INDEX

> AI는 질문을 받으면 이 파일부터 읽고, 아래에서 맞는 문서만 연다.

```yaml
docs:
- path: docs/guide/add-service.md
  title: 새 서비스 추가 가이드·services 폴더 규칙과 체크리스트
  summary: ~/services에 새 프로젝트를 Docker Compose로 올릴 때 지킬 폴더·이름·비밀값·데이터·로그·포트 규칙과, 서비스가 늘기 전에 서버에 해둘 사전 준비(스왑, 로그 기본값, 리버스 프록시)를
    체크리스트로 정리한다
  keywords:
  - 새 서비스
  - 프로젝트 추가
  - 배포
  - services
  - Docker Compose
  - compose.yml
  - 폴더 규칙
  - 프로젝트 이름
  - name
  - .env
  - data
  - 볼륨
  - 로그 회전
  - logging
  - mem_limit
  - 메모리 제한
  - 포트
  - 리버스 프록시
  - Caddy
  - Traefik
  - 스왑
  - daemon.json
  - builder prune
- path: docs/learn/01-server-concepts.md
  title: 01 서버 개념·서버, VPS, 클라우드, IP, 포트, SSH
  summary: 이 학습 시리즈의 출발점으로 서버가 무엇인지, VPS와 클라우드의 차이, IP 주소와 포트, 리눅스, SSH 키 접속을 비유와 함께 설명한다
  keywords:
  - 서버
  - 개념
  - VPS
  - 클라우드
  - 호스팅
  - IP
  - 공인 IP
  - 사설 IP
  - 포트
  - 리눅스
  - Ubuntu
  - SSH
  - 키
  - pem
  - 학습
  - 입문
- path: docs/learn/02-lightsail-instance.md
  title: 02 Lightsail 인스턴스·서버 만들기, 고정 IP, SSH 접속 설정
  summary: AWS Lightsail 서울 리전에 Ubuntu 24.04 인스턴스를 만들고 고정 IP를 붙인 뒤 맥의 ~/.ssh/config에 별칭을 등록해 ssh aws-ubuntu-1로 접속하기까지의 실제
    과정과 확인 방법을 기록한다
  keywords:
  - Lightsail
  - AWS
  - 인스턴스
  - 생성
  - 리전
  - 서울
  - Ubuntu 24.04
  - 플랜
  - 요금
  - 고정 IP
  - static IP
  - SSH
  - pem
  - ssh config
  - aws-ubuntu-1
  - 접속
  - 학습
- path: docs/learn/03-docker-basics.md
  title: 03 Docker 기초·설치, 이미지, 컨테이너, 볼륨, 네트워크
  summary: Ubuntu 서버에 get.docker.com 스크립트로 Docker를 설치하고 ubuntu 사용자를 docker 그룹에 넣은 과정과, Dockerfile·이미지·컨테이너·볼륨·포트·네트워크 개념을
    건물과 방 비유로 정리한다
  keywords:
  - Docker
  - 도커
  - 설치
  - get.docker.com
  - docker 그룹
  - usermod
  - hello-world
  - Dockerfile
  - 이미지
  - image
  - 컨테이너
  - container
  - 볼륨
  - 바인드 마운트
  - 포트
  - ports
  - 네트워크
  - network
  - 멀티 스테이지
  - 학습
- path: docs/learn/04-docker-compose.md
  title: 04 Docker Compose로 프로젝트 가동·team-splitter 배포 기록
  summary: compose.yml로 컨테이너 설정을 파일에 적어 관리하는 방법과 프로젝트 이름·.env·restart·로그 회전 개념을 설명하고, 디스코드 봇 team-splitter를 ~/services에 git
    clone, .env 업로드, data 폴더 준비, docker compose up으로 가동한 실제 과정을 기록한다
  keywords:
  - Docker Compose
  - compose.yml
  - docker compose up
  - 프로젝트 이름
  - service
  - .env
  - env_file
  - scp
  - restart
  - unless-stopped
  - logging
  - 로그 회전
  - 볼륨
  - data
  - team-splitter
  - 배포
  - 업데이트
  - git pull
  - 학습
- path: docs/learn/05-domain-dns.md
  title: 05 도메인과 DNS·가비아 구매, 네임서버, A·와일드카드 레코드
  summary: 가비아에서 hwiii.com을 사고 가비아 네임서버를 유지한 채 A 레코드 *와 @를 Lightsail 고정 IP로 지정한 과정, 등록기관·네임서버·호스팅의 차이, dig로 위임과 반영을 확인하는 방법을
    기록한다
  keywords:
  - 도메인
  - DNS
  - 가비아
  - Gabia
  - hwiii.com
  - 등록기관
  - 네임서버
  - 타사 네임서버
  - A 레코드
  - 와일드카드
  - 서브도메인
  - 호스트 @
  - TTL
  - dig
  - whois
  - 반영
  - 전파
  - 학습
- path: docs/learn/06-firewall-ports.md
  title: 06 방화벽과 포트·Lightsail 방화벽, 80/443, ufw, 열린 포트 확인
  summary: Lightsail 방화벽과 서버 안 ufw의 차이, 웹에 필요한 80/443 포트, ss와 curl로 서버 안팎에서 포트 상태를 확인하는 방법을 설명하고, 연결 거부와 타임아웃으로 방화벽 상태를 추정한
    실제 점검 결과와 남은 작업(443 열기)을 기록한다
  keywords:
  - 방화벽
  - firewall
  - Lightsail 방화벽
  - IPv4 Firewall
  - ufw
  - iptables
  - 포트
  - '80'
  - '443'
  - '22'
  - HTTPS
  - 열린 포트
  - ss
  - curl
  - nc
  - connection refused
  - 연결 거부
  - timeout
  - 타임아웃
  - Docker 포트
  - 학습
- path: docs/learn/07-reverse-proxy-https.md
  title: 07 리버스 프록시와 HTTPS·Caddy로 서브도메인마다 다른 사이트 연결
  summary: 포트 하나를 여러 사이트가 나눠 쓰기 위한 리버스 프록시 개념, Host 헤더 기반 라우팅, Let's Encrypt 자동 HTTPS를 설명하고, ~/services/_proxy에 Caddy를 두고
    proxy 도커 네트워크로 각 사이트 컨테이너를 연결하는 구성과 요청 흐름을 정리한다
  keywords:
  - 리버스 프록시
  - reverse proxy
  - Caddy
  - Caddyfile
  - Nginx
  - Traefik
  - Nginx Proxy Manager
  - Host 헤더
  - 서브도메인
  - 라우팅
  - HTTPS
  - TLS
  - 인증서
  - Let's Encrypt
  - _proxy
  - proxy 네트워크
  - external network
  - reload
  - 요청 흐름
  - 학습
- path: docs/learn/08-nginx-static-site.md
  title: 08 Nginx 정적 사이트·pont-coffee를 컨테이너로 띄우고 프록시에 연결
  summary: DB 없는 단순 웹사이트를 Nginx 컨테이너로 서빙하는 방법을 빌드 없는 순수 HTML 방식과 Vite 같은 빌드 도구 방식(멀티 스테이지)으로 나눠 설명하고, pont-coffee를 proxy 네트워크에
    붙여 pont-coffee.hwiii.com으로 여는 계획을 정리한다
  keywords:
  - Nginx
  - 정적 사이트
  - static site
  - pont-coffee
  - HTML
  - nginx:alpine
  - 웹서버
  - 바인드 마운트
  - 읽기 전용
  - ro
  - Vite
  - 빌드
  - 멀티 스테이지
  - dist
  - proxy 네트워크
  - mem_limit
  - 학습
- path: docs/learn/09-database.md
  title: 09 데이터베이스·SQLite 실제 사례와 Postgres 컨테이너, 볼륨, 백업
  summary: 파일 하나로 동작하는 SQLite(team-splitter가 실제로 사용)와 별도 서버로 도는 PostgreSQL의 차이, 컨테이너에서 DB 데이터를 볼륨으로 지키는 방법, DB 포트를 외부에 열지
    않는 원칙, 백업 방법을 정리한다
  keywords:
  - 데이터베이스
  - DB
  - SQLite
  - WAL
  - PostgreSQL
  - Postgres
  - MySQL
  - 컨테이너 DB
  - 볼륨
  - named volume
  - 데이터 보존
  - 백업
  - pg_dump
  - sqlite3 .backup
  - DB 포트
  - '5432'
  - depends_on
  - healthcheck
  - 학습
- path: docs/learn/10-operations.md
  title: 10 운영·메모리와 스왑, 로그, 빌드 캐시, 업데이트, 백업과 스냅샷
  summary: 4GB 서버에서 여러 프로젝트를 돌릴 때 필요한 운영 지식으로 메모리·OOM·스왑, 컨테이너 메모리 제한, 로그 회전과 daemon.json 기본값, 빌드 캐시 정리, 비밀값 파일 관리, Lightsail
    스냅샷 백업을 정리하고 2026-10-27 점검에서 나온 실제 수치와 할 일을 기록한다
  keywords:
  - 운영
  - 메모리
  - RAM
  - OOM
  - 스왑
  - swap
  - swapfile
  - mem_limit
  - docker stats
  - 로그
  - 로그 회전
  - daemon.json
  - 빌드 캐시
  - docker builder prune
  - docker system df
  - 디스크
  - 업데이트
  - .env
  - 비밀값
  - 백업
  - 스냅샷
  - Lightsail snapshot
  - 학습
- path: docs/ops/server-overview.md
  title: 서버 현황·aws-ubuntu-1 사양과 services 구조
  summary: Lightsail 서울 서버 aws-ubuntu-1의 접속 방법, OS·Docker 버전, 메모리·디스크, ~/services 폴더 구조, 실행 중인 컨테이너와 네트워크, 확인된 운영 위험 요소를 정리한다
  keywords:
  - 서버
  - aws-ubuntu-1
  - Lightsail
  - SSH
  - 접속
  - 사양
  - 메모리
  - 스왑
  - 디스크
  - Docker
  - Docker Compose
  - services
  - team-splitter
  - 컨테이너
  - 네트워크
  - 포트
  - ufw
  - 방화벽
  - 로그
  - daemon.json
  - 빌드 캐시
  - 백업
```
