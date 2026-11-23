# Study

서버 환경을 직접 구축·운영하며 겪은 일과 공부한 내용을 정리한 글 모음입니다.

- **articles**: 실제로 구축·운영한 환경의 배경, 구성, 판단 과정
- **learning**: 일반적인 개념과 원리를 공부한 학습 노트

> 글에 나오는 IP, 도메인, 계정명은 역할을 나타내는 예시 값으로 바꿔 두었습니다.

## 네트워크 운영

공인 IP 하나를 운영·개발 서비스가 함께 쓰는 서버망에서, 요청이 FortiGate와 Nginx를 거쳐 컨테이너에 도착하기까지의 구조를 다룹니다. 위에서부터 순서대로 읽으면 흐름이 이어집니다.

| # | 글 | 다루는 내용 |
| --- | --- | --- |
| 0 | [공인 IP 하나로 운영망과 개발망을 연결하는 구조 이해하기](network-infrastructure-article.md) | 전체 구조 개요 |
| 1 | [회사에서 운영 중인 환경으로 네트워크 이해하기](articles/network/network-infrastructure-understanding.md) | 회사망과 서버망의 경계 |
| 2 | [FortiGate의 SSH 포트포워딩과 접근 정책 이해하기](articles/network/fortigate-ssh-port-forwarding.md) | SSH, VIP, 방화벽 정책 |
| 3 | [FortiGate Virtual Server의 HTTPS 도메인 분기 이해하기](articles/network/fortigate-https-host-routing.md) | HTTPS, 도메인별 분기 |
| 4 | [공인 IP 하나에서 운영·개발 HTTPS를 분리하기 어려운 이유](articles/network/single-public-ip-production-development-https-routing.md) | 운영·개발 HTTPS 분리 |
| 5 | [Nginx·Docker의 요청 전달과 포트 구조 이해하기](articles/network/nginx-docker-request-routing.md) | Nginx, Docker 포트 |
| 6 | [서비스 확장과 네트워크 접속 문제 점검하기](articles/network/network-service-expansion-and-troubleshooting.md) | 서비스 추가, 접속 문제 점검 |

시리즈 목차만 따로 보려면 [네트워크 운영 경험](articles/network/README.md)을 참고하세요.

## 학습 노트

| 글 | 다루는 내용 |
| --- | --- |
| [TLS Certificates: Fundamentals, Issuance, and Renewal](learning/tls-certificates.md) | TLS 인증서 개념, 발급, 갱신 (Certbot, Nginx) |
| [FortiGate의 HTTPS 처리 흐름: TLS 종료부터 Nginx 전달까지](learning/network/fortigate-https-request-flow.md) | 그림 4장으로 보는 인증서, Host 분기, 방화벽 정책, Nginx 전달 |

## 폴더 구조

```text
.
├── README.md
├── network-infrastructure-article.md   # 네트워크 구조 개요
├── articles/
│   └── network/                        # 네트워크 운영 시리즈
└── learning/                           # 학습 노트
    └── network/                        # 네트워크 학습 노트
```
