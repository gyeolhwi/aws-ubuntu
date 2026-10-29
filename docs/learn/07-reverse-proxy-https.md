---
id: learn-07-reverse-proxy-https
title: 07 리버스 프록시와 HTTPS·Caddy로 서브도메인마다 다른 사이트 연결
category: learn
summary: 포트 하나를 여러 사이트가 나눠 쓰기 위한 리버스 프록시 개념, Host 헤더 기반 라우팅, Let's Encrypt 자동 HTTPS를 설명하고, ~/services/_proxy에 Caddy를 두고 proxy 도커 네트워크로 각 사이트 컨테이너를 연결하는 구성과 요청 흐름을 정리한다
keywords: [리버스 프록시, reverse proxy, Caddy, Caddyfile, Nginx, Traefik, Nginx Proxy Manager, Host 헤더, 서브도메인, 라우팅, HTTPS, TLS, 인증서, Let's Encrypt, _proxy, proxy 네트워크, external network, reload, 요청 흐름, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 07 리버스 프록시와 HTTPS·Caddy로 서브도메인마다 다른 사이트 연결

> 학습 시리즈: [06 방화벽](06-firewall-ports.md) → **07** → [08 Nginx 정적 사이트](08-nginx-static-site.md)
> 상태: **[예정]** 구조와 방식만 결정했다. 실제로 구성하면 아래 "실제로 한 일"을 채운다.

## 1. 왜 필요한가

- DNS의 `*` 레코드 때문에 모든 서브도메인이 **같은 서버의 443 포트**로 들어온다.
- 포트 하나는 컨테이너 하나만 열 수 있다.
- 그래서 그 문 앞에서 요청한 이름(`pont-coffee`인지 `blog`인지)을 보고 나눠 주는 프로그램이 필요하다.

## 2. 용어

| 용어 | 한 줄 정의 | 비유 |
|---|---|---|
| **리버스 프록시** | 바깥 요청을 모두 받아 도메인 이름을 보고 알맞은 내부 서버로 넘겨주는 중계기 | 1층 안내 데스크 |
| **Host 헤더** | 브라우저가 "나는 `pont-coffee.hwiii.com`을 찾는다"고 함께 보내는 정보 | 방문객이 말하는 가게 이름 |
| **Caddy** | 설정이 짧고 HTTPS를 자동 처리하는 웹서버 겸 프록시 | 안내 데스크 직원 |
| **Caddyfile** | Caddy 설정. "도메인 → 컨테이너:포트" 목록 | 층별 안내표 |
| **HTTPS / TLS 인증서** | 통신 암호화와 "진짜 이 사이트" 증명. 주소창 자물쇠 | 사업자등록증 |
| **Let's Encrypt** | 무료 인증서 발급 기관. 90일마다 갱신해야 하는데 Caddy가 자동으로 한다 | 무료 등록증 발급처 |
| **external 네트워크** | compose 밖에서 미리 만든 도커 네트워크. 여러 프로젝트가 같이 붙을 수 있다 | 층을 가로지르는 공용 내선 |

### 다른 선택지와 비교 (왜 Caddy인가)

| 방식 | 특징 | 판단 |
|---|---|---|
| **Caddy + Caddyfile** | 파일 하나에 전체 연결이 보임. HTTPS 자동 | **선택.** 단순하고 학습에 좋다 |
| Nginx 프록시 + certbot | 가장 널리 쓰임. 인증서를 따로 설정해야 함 | 설정량이 많다 |
| Traefik + 라벨 | 각 서비스에 라벨을 달면 자동 인식 | 라벨 문법을 따로 배워야 한다 |
| Nginx Proxy Manager | 웹 화면(GUI)에서 설정 | 설정이 파일로 남지 않는다 |

## 3. 구성

```
인터넷 ── *.hwiii.com ──► <SERVER_IP>
                              │ 80/443 (이 포트만 외부에 열림)
                         ┌────▼─────┐
                         │  Caddy   │  ~/services/_proxy
                         └────┬─────┘
                    도커 네트워크 "proxy"
               ┌──────────────┼──────────────┐
      pont-coffee-web-1:80   blog-app-1:3000   …     ← 각자 ports: 없음
```

```yaml
# ~/services/_proxy/compose.yml
services:
  caddy:
    image: caddy:2
    restart: unless-stopped
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./data:/data            # 발급받은 인증서. 지우면 재발급해야 한다
      - ./config:/config
    mem_limit: 128m
    logging:
      options: { max-size: 10m, max-file: "3" }
    networks: [proxy]
networks:
  proxy:
    external: true
```

```
# ~/services/_proxy/Caddyfile
pont-coffee.hwiii.com {
    reverse_proxy pont-coffee-web-1:80
}
```

- 폴더 이름을 `_proxy`로 한 이유: `ls` 하면 맨 위에 보이고, 서비스가 아니라 기반 시설이라는 게 드러난다.
- 이 설정 파일들은 서버에만 두지 않고 이 레포에도 보관할 예정이다. 서버가 망가져도 도메인 연결 정보가 남는다.

## 4. 요청 하나가 흘러가는 길

```
1. 브라우저 → DNS: "pont-coffee.hwiii.com IP?"     → <SERVER_IP>
2. 브라우저 → <SERVER_IP>:443                       → Lightsail 방화벽 통과
3. Caddy: HTTPS 해독, Host = pont-coffee.hwiii.com 확인
4. Caddy → proxy 네트워크 → pont-coffee-web-1:80
5. 응답이 같은 길로 돌아감
```

## 5. 실제로 한 일 [예정]

예정 순서:

```sh
ssh aws-ubuntu-1
docker network create proxy
mkdir -p ~/services/_proxy && cd ~/services/_proxy
# compose.yml, Caddyfile 작성 (이 레포의 설정을 복사)
docker compose up -d
docker compose logs -f        # 인증서 발급 로그 확인
```

사이트를 추가할 때는 Caddyfile에 블록 3줄을 추가하고 다시 읽어 들인다. 다른 사이트는 멈추지 않는다.

```sh
docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
```

사전 조건: [05](05-domain-dns.md) DNS 완료, [06](06-firewall-ports.md) 443 열기.

## 다음

[08 Nginx로 정적 사이트 띄우기](08-nginx-static-site.md)
