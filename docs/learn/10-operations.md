---
id: learn-10-operations
title: 10 운영·메모리와 스왑, 로그, 빌드 캐시, 업데이트, 백업과 스냅샷
category: learn
summary: 4GB 서버에서 여러 프로젝트를 돌릴 때 필요한 운영 지식으로 메모리·OOM·스왑, 컨테이너 메모리 제한, 로그 회전과 daemon.json 기본값, 빌드 캐시 정리, 비밀값 파일 관리, Lightsail 스냅샷 백업을 정리하고 2026-10-27 점검에서 나온 실제 수치와 할 일을 기록한다
keywords: [운영, 메모리, RAM, OOM, 스왑, swap, swapfile, mem_limit, docker stats, 로그, 로그 회전, daemon.json, 빌드 캐시, docker builder prune, docker system df, 디스크, 업데이트, .env, 비밀값, 백업, 스냅샷, Lightsail snapshot, 학습]
related_files: []
last_updated: "2026-10-29"
---
# 10 운영·메모리와 스왑, 로그, 빌드 캐시, 업데이트, 백업과 스냅샷

> 학습 시리즈: [09 데이터베이스](09-database.md) → **10** → 다음 작업은 11번부터 이어서 기록한다
> 상태: **스왑 완료(2026-10-29). 로그 기본값·백업은 [예정]**

## 1. 메모리와 스왑

| 용어 | 한 줄 정의 |
|---|---|
| **RAM** | 실행 중인 프로그램의 작업 공간. 우리 서버는 3.7GiB |
| **OOM** (Out Of Memory) | 메모리가 바닥나면 리눅스가 프로세스를 강제로 죽인다. 컨테이너가 이유 없이 재시작된다면 의심해 볼 것 |
| **스왑** | 메모리가 부족할 때 디스크를 임시 메모리로 쓰는 것. 느리지만 OOM을 막아 준다 |
| **mem_limit** | 컨테이너 하나의 메모리 상한. 한 서비스가 서버 전체를 먹는 것을 막는다 |

메모리를 가장 많이 쓰는 순간은 **서버에서 이미지를 빌드할 때**다 (npm ci, 네이티브 모듈 컴파일).

스왑 2GB 추가 (2026-10-29 완료) **[서버]**:

```sh
ls /swapfile 2>/dev/null && echo '이미 있음: 아래 명령은 실행하지 않는다'   # 먼저 확인
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab   # 재부팅 후에도 유지
free -h                                                     # Swap: 2.0Gi 확인
```

실제 결과: `Swap: 2.0Gi`, `/etc/fstab`에 `/swapfile none swap sw 0 0` 추가 확인. 서비스 재시작은 필요 없었다.
첫 줄에서 `/swapfile`이 이미 보이면 나머지는 실행하지 않는다. 두 번 만들면 `fstab`에 줄이 중복된다.

되돌리기 **[서버]**:

```sh
sudo swapoff /swapfile
sudo sed -i '/^\/swapfile/d' /etc/fstab
sudo rm /swapfile
```

확인 명령: `free -h`, `docker stats --no-stream`

## 2. 로그

- 컨테이너 로그는 기본적으로 크기 제한 없이 쌓인다.
- 지금은 compose마다 `logging: max-size 10m, max-file 3`을 적고 있다. 새 프로젝트가 이걸 빠뜨리면 디스크가 찬다.
- 서버 전체 기본값으로 막아 두기 [예정]:

```json
// /etc/docker/daemon.json
{ "log-driver": "json-file", "log-opts": { "max-size": "10m", "max-file": "3" } }
```

적용하려면 `sudo systemctl restart docker`가 필요하고, 이때 **모든 컨테이너가 잠깐 멈춘다.** 새로 만드는 컨테이너부터 적용된다.

## 3. 디스크와 빌드 캐시

```sh
df -h /              # 디스크 전체
docker system df     # Docker가 쓰는 양
docker builder prune # 빌드 캐시 정리 (실행 중인 서비스에는 영향 없음)
```

2026-10-27 점검: 빌드 캐시 2.86GB, 그중 2.46GB가 정리 가능했다. 빌드할 때마다 쌓이니 가끔 정리한다.

## 4. 비밀값 파일

- `.env`는 서버 프로젝트 폴더에만 두고 `chmod 600`으로 잠근다.
- `.env.bak-날짜` 같은 백업 파일은 `.gitignore` 규칙(예: `.env*.local`)에 걸리지 않을 수 있다. `git add -A` 한 번에 커밋될 수 있으니, 서버 밖에 보관하거나 ignore 규칙을 `.env*`로 넓힌다.
- 이 레포에도 IP, 키 파일 이름, 토큰을 쓰지 않는다 (`<SERVER_IP>`, `<KEY>`).

## 5. 백업과 스냅샷 [예정]

| 대상 | 방법 |
|---|---|
| 서버 전체 | Lightsail 콘솔 → 인스턴스 → **Snapshots** → 자동 스냅샷 켜기 (하루 1회, 유료) |
| 앱 데이터 | [09](09-database.md)의 DB 백업 → 서버 밖에 보관 |
| 설정 | 프로젝트 코드는 각 GitHub 레포, `_proxy` 설정은 이 레포 |

## 6. 점검 루틴

```sh
ssh aws-ubuntu-1 'free -h; df -h /; docker ps; docker system df'
```

현재 서버 상태 표는 [서버 현황](../ops/server-overview.md)에 있다.
