---
title: Terraform Docker 컨테이너 상태 출력 정리

description: Terraform state show에 기록된 Docker 컨테이너 속성과 옵션을 정리한 학습 기록

document_type: standard

status: active

project1: terraform

tags:
  - infra
  - terraform
  - 배포

aliases:
  - 테라폼 설정값 정리

연관:
  - server
  - ansible
  - jenkins
  - CI/CD

updated: 2026-10-14
---

# Terraform Docker 컨테이너 상태 출력 정리

## 개요

Terraform이 관리하는 Docker 컨테이너의 `terraform state show` 출력값을 항목별로 정리합니다.

- `terraform state show`에는 .tf에서 가져온 값만 있는 것이 아니라 컨테이너의 기본값, Docker에서 읽혀온 값들이 존재
- `null`은 기록된 값이 없음
- `[]`는 빈 목록
- `terraform state show`는 실시간 조회 상태가 아님

## 학습 내용

### 리소스와 실행 설정

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `docker_container.iot_server` | Terraform에서 이 리소스를 가리키는 주소 docker_container는 리소스 종류, iot_server는 Terraform 내부 이름 |
| `name = "app-kiosk-iot-server"` | Docker에 표시되는 실제 컨테이너 이름 |
| `id = "c257…fa75"` | Docker가 부여한 컨테이너 ID |
| `image = "app-kiosk-iot-server:latest"` | 컨테이너 생성에 사용된 이미지 이름과 태그입니다. |
| `entrypoint = ["app-entrypoint.sh"]` | 시작할 때 실행할 진입점입니다. |
| `command = []` | 별도의 시작 명령 인자가 이 목록에 기록되어 있지 않습니다. 최종 실행 명령 전체는 이미지의 CMD 등도 함께 봐야 알 수 있습니다. |
| `working_dir = "/app"` | 컨테이너 프로세스의 작업 디렉터리입니다. |
| `user = "1000:1000"` | 첫 프로세스를 UID 1000, GID 1000으로 실행합니다. |
| `hostname = "c257811a2f1a"` | 컨테이너 내부 호스트 이름입니다. 여기서는 컨테이너 ID 앞부분과 같습니다. |
| `domainname = null` | 별도 도메인 이름이 기록되지 않았습니다. |
| `env = []` | 이 리소스의 환경변수 목록은 비어 있습니다. 이미지에 정의된 ENV까지 없다는 뜻은 아닙니다. |
| `attach = false` | Terraform이 컨테이너 생성 후 표준 입출력에 연결해 종료를 기다리지 않습니다. |
| `stdin_open = false` | 표준 입력을 열린 상태로 유지하지 않습니다. |
| `tty = false` | 가상 터미널을 할당하지 않습니다. |
| `init = false` | 별도의 init 프로세스를 주입하지 않는 것으로 기록되어 있습니다. |
| `start = true` | 컨테이너를 생성한 뒤 시작하도록 설정된 값입니다. |
| `must_run = true` | Terraform이 컨테이너를 실행 상태로 유지하려는 설정입니다. 이 값 자체가 지금 실행 중이라는 실시간 증명은 아닙니다. |
| `wait = false` | 생성 후 healthy 상태가 될 때까지 Terraform이 기다리지 않습니다. |
| `wait_timeout = 60` | wait를 사용할 때의 최대 대기 시간인 60초입니다. 현재는 wait = false이므로 그 대기를 수행하지 않습니다. |
| `container_read_refresh_timeout_milliseconds = 15000` | 제공자가 컨테이너 상태를 읽을 때 running 상태에 도달하기를 기다리는 제한 시간 15초입니다. |
| `rm = false` | 컨테이너가 종료되어도 자동 삭제하지 않습니다. |
| `restart = "unless-stopped"` | 예상치 못한 종료나 Docker 데몬 재시작 시 다시 시작하는 정책입니다. 수동으로 중지한 컨테이너는 그 중지 상태를 유지합니다. |
| `max_retry_count = 0` | restart = "on-failure" 정책에서 사용하는 재시도 횟수 값입니다. 현재 정책에는 적용되지 않습니다. |
| `stop_signal = null` | 별도 중지 신호가 기록되지 않았습니다. |
| `stop_timeout = 0` | 중지 대기 시간이 0초로 기록되어 있습니다. |
| `runtime = "runc"` | 컨테이너 실행에 사용하는 런타임입니다. |

### 자원·격리·보안 설정

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `cpu_set = null` | 사용할 CPU 코어를 특정 번호로 제한하지 않았습니다. |
| `cpu_shares = 0` | CPU 상대 가중치가 별도로 지정되지 않은 상태로 기록되어 있습니다. CPU를 전혀 쓰지 못한다는 뜻은 아닙니다. |
| `memory = 0` | 컨테이너 메모리 제한을 별도로 지정하지 않은 값입니다. 단위는 MB입니다. |
| `memory_reservation = 0` | 메모리 소프트 제한을 지정하지 않았습니다. 단위는 MB입니다. |
| `memory_swap = 0` | 메모리와 스왑을 합친 제한을 별도로 지정하지 않은 값입니다. 단위는 MB입니다. |
| `shm_size = 64` | 컨테이너의 /dev/shm 크기가 64MB입니다. |
| `ipc_mode = "private"` | IPC 네임스페이스를 다른 컨테이너와 공유하지 않습니다. |
| `pid_mode = null` | PID 네임스페이스 공유 방식이 별도로 기록되지 않았습니다. |
| `userns_mode = null` | 사용자 네임스페이스 모드가 별도로 기록되지 않았습니다. |
| `privileged = false` | Docker의 특권 모드로 실행하지 않습니다. |
| `security_opts = []` | 별도 보안 옵션 목록이 없습니다. |
| `read_only = false` | 컨테이너 루트 파일시스템을 읽기 전용으로 설정하지 않았습니다. |

### 로그 설정

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `log_driver = "json-file"` | Docker가 컨테이너 로그를 JSON 파일 형식으로 저장합니다. |
| `log_opts["max-size"] = "50m"` | 로그 파일 하나가 약 50MB에 이르면 순환하도록 설정했습니다. |
| `log_opts["max-file"] = "5"` | 순환 로그 파일을 최대 5개 유지합니다. |
| `logs = false` | Terraform 제공자가 `attach`를 통해 컨테이너 로그를 저장하지 않습니다. **Docker 로그 기록 자체가 꺼졌다는 뜻은 아닙니다.** |

### 네트워크 설정

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `network_mode = "bridge"` | 네트워크 모드 값이 `bridge`입니다. 아래 `network_data`에는 실제 연결 정보로 두 사용자 정의 네트워크가 표시됩니다. |
| `bridge = null` | Docker 네트워크 설정에서 읽는 별도 브리지 이름 값이 기록되지 않았습니다. |
| `publish_all_ports = false` | 이미지 등에 선언된 모든 포트를 자동으로 호스트에 공개하지 않습니다. 아래 `ports`에 지정된 포트는 별도로 공개됩니다. |

### 추가 네트워크 설정

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `name` | 첫 블록은 `kiosk-db-network`, 둘째 블록은 `mqtt_iot-net`에 연결합니다. |
| `aliases = ["app-kiosk-iot-server", "iot-server"]` | 각 네트워크에서 사용할 컨테이너의 네트워크 별칭 두 개입니다. |
| `ipv4_address = null` | 이 블록에서 고정 IPv4 주소를 지정하지 않았습니다. `network_data`의 IPv4 주소는 실제 연결 정보로 기록된 값입니다. |
| `ipv6_address = null` | 고정 IPv6 주소를 지정하지 않았습니다. |
| `mac_address = null` | 이 블록에서 MAC 주소를 직접 지정하지 않았습니다. `network_data`에는 실제 연결의 MAC 주소가 기록되어 있습니다. |

### 헬스체크 설정

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `test = ["CMD-SHELL", "wget -qO- http://127.0.0.1:9200/health \|\| exit 1"]` | 컨테이너 **내부**에서 셸로 `wget`을 실행해 자신의 `9200/health`에 요청합니다. 명령이 실패하면 `exit 1`로 검사 실패를 알립니다. `-q`는 조용히 실행, `-O-`는 응답을 표준 출력으로 보내는 옵션입니다. |
| `interval = "30s"` | 검사 간격은 30초입니다. |
| `timeout = "5s"` | 한 번의 검사에 허용하는 시간은 5초입니다. |
| `retries = 3` | 검사가 연속 3번 실패하면 Docker가 `unhealthy`로 판단합니다. |
| `start_period = "0s"` | 시작 직후 실패 횟수 계산을 미루는 유예 기간이 없습니다. |
| `start_interval = "0s"` | 시작 유예 기간용 별도 검사 간격이 0초로 기록되어 있습니다. 여기서는 `start_period`도 0초입니다. |

### 마운트와 볼륨

| 항목과 출력값 | 의미와 메모 |
| --- | --- |
| `mounts.type = "bind"` | Docker 호스트의 경로를 컨테이너에 직접 연결하는 바인드 마운트입니다. |
| `mounts.source = "/home/nrobotics/app-kiosk-iot-server"` | Docker 호스트 쪽 디렉터리입니다. |
| `mounts.target = "/app"` | 컨테이너 안에서 그 디렉터리가 보이는 위치입니다. 이 마운트가 적용된 동안 이미지에 들어 있던 `/app`의 기존 내용은 해당 경로에서 가려질 수 있습니다. |
| `remove_volumes = true` | 컨테이너 제거 시 연결된 **익명 볼륨**을 제거하는 설정입니다. 위 `source`의 호스트 디렉터리를 삭제한다는 뜻은 아닙니다. |

## 정리

- state에 기록된 값과 컨테이너의 실시간 상태를 구분합니다.
- 빈 값이나 기본값을 해석할 때 이미지 설정과 Docker에서 읽은 정보도 함께 고려합니다.
- 로그 수집, 헬스체크, 볼륨 제거 옵션의 관리 범위를 확인합니다.
