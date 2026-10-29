---
id: learn-02-lightsail-instance
title: 02 Lightsail 인스턴스·서버 만들기, 고정 IP, SSH 접속 설정
category: learn
summary: AWS Lightsail 서울 리전에 Ubuntu 24.04 인스턴스를 만들고 고정 IP를 붙인 뒤 맥의 ~/.ssh/config에 별칭을 등록해 ssh aws-ubuntu-1로 접속하기까지의 실제 과정과 확인 방법을 기록한다
keywords: [Lightsail, AWS, 인스턴스, 생성, 리전, 서울, Ubuntu 24.04, 플랜, 요금, 고정 IP, static IP, SSH, pem, ssh config, aws-ubuntu-1, 접속, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 02 Lightsail 인스턴스·서버 만들기, 고정 IP, SSH 접속 설정

> 학습 시리즈: [01 개념](01-server-concepts.md) → **02** → [03 Docker](03-docker-basics.md)
> 상태: **완료** (2026-10-02 무렵)

## 1. 개념

| 용어 | 한 줄 정의 |
|---|---|
| **인스턴스** | 클라우드에서 빌린 가상 서버 한 대 |
| **리전** | 서버가 실제로 있는 지역. 가까울수록 빠르다 |
| **블루프린트** | 처음 설치되어 나오는 OS 이미지 |
| **플랜** | CPU·메모리·디스크 크기와 월 요금 묶음 |
| **고정 IP (Static IP)** | 인스턴스에 붙여 두는, 바뀌지 않는 공인 IP |

## 2. 실제로 한 일

[Lightsail 콘솔](https://lightsail.aws.amazon.com) → **Create instance**

| 항목 | 선택 | 이유 |
|---|---|---|
| Region | Seoul (ap-northeast-2) | 사용자가 한국이라 응답이 빠르다 |
| Platform / Blueprint | Linux/Unix → OS Only → **Ubuntu 24.04 LTS** | 장기 지원, 자료가 가장 많다 |
| Network | Dual-stack (IPv4 포함) | IPv6 전용은 접속이 안 되는 곳이 있다 |
| Plan | **$24 (4GB RAM, 2 vCPU, 80GB SSD)** | 여러 프로젝트를 같이 올릴 서버 |

그다음:

1. 인스턴스 → **Networking** 탭 → **Attach static IP** → 새 고정 IP를 만들어 붙인다.
   - 붙이지 않으면 재부팅할 때마다 IP가 바뀌어서 도메인 연결이 끊긴다.
   - 인스턴스에 붙어 있는 고정 IP는 무료다. **인스턴스를 지울 때는 고정 IP도 같이 지운다.** 남겨 두면 요금이 나간다.
2. 계정 페이지에서 SSH 키(`.pem`)를 내려받아 맥의 `~/.ssh/`에 두고 권한을 잠근다.
   ```sh
   chmod 400 ~/.ssh/<KEY>.pem
   ```
3. `~/.ssh/config`에 별칭 등록 ([01](01-server-concepts.md#4-ssh-서버에-원격-접속하기) 참고).

## 3. 확인

```sh
ssh aws-ubuntu-1 'hostname; whoami; lsb_release -ds; free -h; df -h /'
```

2026-10-27에 확인한 값:

| 항목 | 값 |
|---|---|
| 사용자 | `ubuntu` |
| OS | Ubuntu 24.04.4 LTS |
| CPU / 메모리 | 2 vCPU / 3.7GiB (4GB 플랜에서 커널 몫을 뺀 값) |
| 디스크 | 77G 중 7G 사용 |
| 스왑 | 없음 → [10 운영](10-operations.md)에서 다룬다 |

## 4. 알아둘 것

- Lightsail 방화벽은 콘솔의 **Networking → IPv4 Firewall**에 있다. 기본으로 22, 80이 열려 있다 → [06 방화벽](06-firewall-ports.md).
- 서버 IP, `.pem` 파일 이름, 계정 정보는 이 문서와 레포에 쓰지 않는다. `<SERVER_IP>`, `<KEY>`로 표시한다.

## 다음

[03 Docker 기초](03-docker-basics.md)에서 서버에 Docker를 설치한다.
