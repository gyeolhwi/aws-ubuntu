---
id: learn-06-firewall-ports
title: 06 방화벽과 포트·Lightsail 방화벽, 80/443, ufw, 열린 포트 확인
category: learn
summary: Lightsail 방화벽과 서버 안 ufw의 차이, 웹에 필요한 80/443 포트, ss와 curl로 서버 안팎에서 포트 상태를 확인하는 방법을 설명하고, 연결 거부와 타임아웃으로 방화벽 상태를 추정한 실제 점검 결과와 남은 작업(443 열기)을 기록한다
keywords: [방화벽, firewall, Lightsail 방화벽, IPv4 Firewall, ufw, iptables, 포트, "80", "443", "22", HTTPS, 열린 포트, ss, curl, nc, connection refused, 연결 거부, timeout, 타임아웃, Docker 포트, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 06 방화벽과 포트·Lightsail 방화벽, 80/443, ufw, 열린 포트 확인

> 학습 시리즈: [05 도메인·DNS](05-domain-dns.md) → **06** → [07 리버스 프록시](07-reverse-proxy-https.md)
> 상태: **점검 완료, Lightsail 443 열기는 [예정]** (서버 쪽 Caddy는 준비됨)

## 1. 용어

| 용어 | 한 줄 정의 | 비유 |
|---|---|---|
| **방화벽** | 들어오는 연결을 포트 기준으로 허용하거나 막는 장치 | 출입문 경비 |
| **Lightsail 방화벽** | 서버 **바깥**(AWS 네트워크)에 있는 방화벽. 콘솔에서 설정 | 건물 담장의 경비 |
| **ufw** | 서버 **안**에 있는 리눅스 방화벽 | 건물 내부 경비 |
| **열려 있다 (LISTEN)** | 어떤 프로그램이 그 포트에서 연결을 기다리는 상태 | 문 뒤에 직원이 있음 |

두 방화벽을 모두 통과하고, 그 포트를 듣는 프로그램까지 있어야 연결된다.

## 2. 실제 점검 (2026-10-29)

### 서버 안: 누가 포트를 듣고 있나

```sh
ssh aws-ubuntu-1 'sudo ss -tlnp'
```

외부에서 접근 가능한 주소(`0.0.0.0`)로 듣는 것은 **22(sshd)뿐**이었다. 80/443을 쓰는 프로그램은 없다.
`sudo ufw status` → `inactive`. 서버 안 방화벽은 꺼져 있고, Lightsail 방화벽만 쓰고 있다.

### 서버 밖(맥)에서: 문이 열려 있나

```sh
for p in 80 443 8080; do curl -sS -m 5 -o /dev/null http://<SERVER_IP>:$p; done
```

| 포트 | 결과 | 해석 |
|---|---|---|
| 80 | 15ms 만에 `Couldn't connect` (연결 거부) | 방화벽은 **통과**했고, 듣는 프로그램이 없어 서버가 바로 거절 |
| 443 | 5초 타임아웃 | Lightsail 방화벽이 패킷을 **버림** → 막혀 있음 |
| 8080 | 5초 타임아웃 | 막혀 있음 (정상) |

> 배운 점: **빠른 거부 = 방화벽은 열림, 프로그램 없음** / **타임아웃 = 방화벽이 막음.** 콘솔을 안 봐도 대략 판단할 수 있다.

### 실수에서 배운 것

처음에 웹서버 설치 여부를 이렇게 확인했다가 전부 "active"로 잘못 나왔다.

```sh
systemctl is-active nginx | grep -q active && echo active   # 틀림
```

`inactive`라는 글자 안에도 `active`가 들어 있어서 grep이 걸린 것이다. 출력값 전체를 직접 보거나 `grep -qx active`처럼 줄 전체가 일치하는지 확인해야 한다.

## 3. 해야 할 일 [예정]

Lightsail 콘솔 → 인스턴스 → **Networking** → **IPv4 Firewall** → **Add rule** → `HTTPS (TCP 443)`

- 80은 이미 열려 있다. HTTPS 인증서를 발급받을 때 80도 필요하니 닫지 않는다.
- 2026-10-29 Caddy를 띄운 뒤: 80은 정상 응답(308 → HTTPS), 443은 여전히 **타임아웃**이다. 서버 안에서 `curl --resolve`로 443에 붙으면 200이 나오므로, 남은 원인은 Lightsail 방화벽뿐이다.
- 열고 나면 맥에서 `curl -I https://hello.hwiii.com`이 200을 돌려준다.
- 참고: 인증서는 443이 막힌 상태에서도 발급됐다. Let's Encrypt의 HTTP-01 방식은 **80 포트로만** 소유를 확인하기 때문이다.

## 4. Docker와 방화벽 주의점

- `ports: ["8080:80"]`처럼 공개한 포트는 Docker가 iptables를 직접 고친다. 그래서 **ufw를 켜도 막히지 않는 경우가 있다.**
- 원칙: 외부 포트는 프록시(80/443)만 연다. 나머지는 `ports:`를 쓰지 않거나 `127.0.0.1:8080:80`처럼 서버 안에서만 접근 가능하게 묶는다.

## 다음

[07 리버스 프록시와 HTTPS](07-reverse-proxy-https.md)
