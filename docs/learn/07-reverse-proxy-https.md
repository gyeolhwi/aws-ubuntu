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
> 상태: **완료 (2026-10-29)**. 외부에서 `https://hello.hwiii.com` 접속 확인

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
- 설정 원본은 이 레포의 [`proxy/`](../../proxy/)에 있다. 서버가 망가져도 도메인 연결 정보가 남는다. 실제 파일에는 HTTP/3용 `443:443/udp`도 열려 있다.

## 4. 요청 하나가 흘러가는 길

```
1. 브라우저 → DNS: "pont-coffee.hwiii.com IP?"     → <SERVER_IP>
2. 브라우저 → <SERVER_IP>:443                       → Lightsail 방화벽 통과
3. Caddy: HTTPS 해독, Host = pont-coffee.hwiii.com 확인
4. Caddy → proxy 네트워크 → pont-coffee-web-1:80
5. 응답이 같은 길로 돌아감
```

## 5. 실제로 한 일 (2026-10-29)

사전 조건: [05](05-domain-dns.md) DNS 완료, 이 레포를 맥에 clone해 둔 상태.

**1. [맥] 공용 네트워크와 폴더 만들기**

```sh
ssh aws-ubuntu-1 'docker network create proxy && mkdir -p ~/services/_proxy ~/services/hello'
```

기대 결과: 긴 네트워크 ID 한 줄. `already exists`가 나오면 이미 만든 것이라 괜찮다.

**2. [서버] 테스트 사이트 만들기**: 프록시가 컨테이너까지 연결되는지 확인하려고 띄우는 nginx 기본 페이지다.

```sh
ssh aws-ubuntu-1
cat > ~/services/hello/compose.yml <<'EOF'
services:
  web:
    image: nginx:alpine
    restart: unless-stopped
    mem_limit: 64m
    networks:
      - proxy
networks:
  proxy:
    external: true
EOF
cd ~/services/hello && docker compose up -d
```

기대 결과: `Container hello-web-1 Started`

**3. [맥] Caddy 설정을 서버로 복사** (이 레포 폴더에서 실행)

처음 [`proxy/Caddyfile`](../../proxy/Caddyfile)에는 테스트용 블록 하나만 넣었다. 지금은 [08](08-nginx-static-site.md)에서 pont-coffee로 바꿨다.

```
hello.hwiii.com {
	reverse_proxy hello-web-1:80
}
```

```sh
scp proxy/compose.yml proxy/Caddyfile aws-ubuntu-1:services/_proxy/
```

**4. [서버] Caddy 실행**

```sh
cd ~/services/_proxy && docker compose up -d
docker compose logs -f caddy        # Ctrl+C로 빠져나온다
```

기대 결과: 로그에 `certificate obtained successfully`. 1분 안쪽이면 나온다.

## 6. 확인

| 확인 | 명령 | 결과 |
|---|---|---|
| 컨테이너 | `docker ps` | `proxy-caddy-1`이 80/443 공개, `hello-web-1`은 공개 포트 없음 |
| 인증서 발급 | Caddy 로그 | `certificate obtained successfully` (Let's Encrypt, 2027-01-27 만료, 자동 갱신) |
| HTTP | 맥에서 `curl -I http://hello.hwiii.com` | `308` → `https://hello.hwiii.com/` 자동 리다이렉트 |
| HTTPS (서버 안) | `curl --resolve hello.hwiii.com:443:127.0.0.1 https://hello.hwiii.com` | `200`, 인증서 검증 통과, "Welcome to nginx!" |
| HTTPS (맥) | `curl -I https://hello.hwiii.com` | 처음엔 타임아웃(Lightsail 443 닫힘) → [06](06-firewall-ports.md)에서 연 뒤 `200`, HTTP/2 |
| 메모리 | `docker stats --no-stream` | Caddy 13MiB, 테스트 nginx 3MiB |

`--resolve`는 DNS를 거치지 않고 "이 이름은 이 IP"라고 지정해서 접속하는 옵션이다. 방화벽 바깥을 거치지 않고 서버 안에서 HTTPS가 되는지만 확인할 때 쓴다.

## 7. 겪은 일

| 관찰 | 이유 |
|---|---|
| 폴더는 `_proxy`인데 컨테이너 이름이 `proxy-caddy-1` | Compose 프로젝트 이름은 영문 소문자나 숫자로 시작해야 해서 앞의 `_`가 빠진다. 공용 네트워크 이름(`proxy`)과 이름이 같지만 서로 다른 것이라 문제는 없다 |
| 443이 막혀 있는데 인증서가 발급됨 | Let's Encrypt HTTP-01 방식은 80 포트로 소유를 확인한다. 80은 열려 있었다 |
| 로그에 `HTTP/2 skipped because it requires TLS` | 80 포트(HTTP) 서버에 대한 안내다. 443 서버에서는 HTTP/2·3가 켜진다. 오류가 아니다 |

## 8. 되돌리기

```sh
# [서버] 테스트 사이트만 내리기
cd ~/services/hello && docker compose down && cd .. && rm -r hello
# → 이 레포 proxy/Caddyfile에서 hello 블록을 지우고, 아래 9장 방법으로 반영

# [서버] 프록시 전체 내리기 (80/443이 닫히고 모든 웹 사이트가 끊긴다)
cd ~/services/_proxy && docker compose down
docker network rm proxy             # proxy에 붙은 컨테이너가 없어야 지워진다
```

`_proxy/data/`(인증서)는 지우지 않으면 다시 올릴 때 재사용된다.

## 9. 사이트 추가하는 법

1. 사이트 컨테이너의 compose에 `networks: [proxy]`(external)를 넣고 `ports:`는 쓰지 않는다.
2. 이 레포 `proxy/Caddyfile`에 블록 3줄을 추가하고 커밋한다.
3. **[맥]** 서버에 반영하고 다시 읽어 들인다. 다른 사이트는 멈추지 않는다.

```sh
scp proxy/Caddyfile aws-ubuntu-1:services/_proxy/
ssh aws-ubuntu-1 'cd ~/services/_proxy && docker compose exec -w /etc/caddy caddy caddy reload'
```

## 다음

[08 Nginx로 정적 사이트 띄우기](08-nginx-static-site.md)
