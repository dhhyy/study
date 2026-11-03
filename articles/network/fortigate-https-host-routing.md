---
title: "FortiGate Virtual Server의 HTTPS 도메인 분기 이해하기"
type: article
status: published
created:
updated: 2026-11-03
tags:
  - network
  - infrastructure
published_url: https://github.com/dhhyy/study/blob/develop/articles/network/fortigate-https-host-routing.md
---
<h1 style="text-align: center;">FortiGate Virtual Server의 HTTPS 도메인 분기 이해하기</h1>
<p style="text-align: center;"><em>FortiGate Virtual Server로 HTTPS 요청을 내부 서비스에 전달하는 과정</em><br>
<small>공인 IP 하나에서 도메인별 목적지를 선택하는 구성 예시</small></p>
공인 IP 하나를 공유하는 웹 서비스에 서버마다 별도 포트를 붙이면 접속 주소와 포트의 대응 관계도 함께 관리해야 한다.

VPN 등을 통해 서버망의 사설 주소로 접근할 수 있다면, 회사 내부에서 접근할 서비스마다 외부 포트포워딩을 추가하지 않는 구성도 가능하다.

Virtual Server를 검토한 목적은 공통 웹 진입점을 두고, 요청한 도메인에 따라 내부 서버를 선택하기 위해서였다. 기존 HTTP 도메인 분기와 달리, 아래 HTTPS 443 구성은 아직 적용 전인 검토 예시다.

이 글에서는 FortiGate에서 TLS를 종료하고 내부 HTTP로 전달하는 방식을 설명한다. 주소·도메인·서비스 이름과 포트는 설명용 예시이며, 실제 IP 대신 역할 이름을 사용한다.

## Virtual Server는 요청을 받고 Real Server는 내부 목적지가 된다

Virtual Server는 VM이나 물리 서버 한 대를 뜻하지 않는다.

FortiGate가 외부 요청을 받는 논리적인 진입점이며, 그 뒤에 전달할 내부 목적지를 Real Server로 등록한다.

Fortinet에서는 Virtual Server를 서버 로드밸런싱에 사용하는 VIP 유형으로 설명한다. 공통 진입점의 Nginx 등에서 도메인별 분기를 담당하게 할 수도 있으므로, 반드시 FortiGate에서만 처리해야 하는 기능은 아니다.

이 글에서 사용하는 선택 기준은 HTTP Host다. 같은 공인 IP에 도착한 웹 요청이어도 Host가 다르면 서로 다른 내부 목적지로 전달하도록 구성하는 방식이다.

| Real Server 예시 | 요청한 도메인에 대응하는 HTTP Host | 내부 IP | 내부 포트 |
| --- | --- | --- | --- |
| 개발 업데이트 서비스 | `update.example.com` | `DEV_SERVER_01_IP` | 80 |
| 개발 포털 서비스 | `portal.example.com` | `DEV_SERVER_02_IP` | 80 |

DNS에서 두 도메인이 같은 `PUBLIC_IP`를 가리키더라도, FortiGate에는 어떤 Host를 어느 서버로 보낼지 정한 설정이 별도로 필요하다. 이 예시에서는 Virtual Server가 그 역할을 담당한다.

반대로 서로 다른 도메인을 같은 내부 IP·포트로 보낼 수도 있다. 이 경우 서버에 도착한 요청을 Nginx에서 다시 구분한다면, `server_name` 등 도메인에 맞는 설정도 필요하다.

FortiGate는 서버를 선택하고, 서버 안의 Nginx는 애플리케이션을 선택하는 식으로 역할을 나눌 수 있다.

## HTTPS 요청을 FortiGate Virtual Server에서 처리한다면

HTTP에서는 Host 정보를 읽어 목적지를 고를 수 있지만, HTTPS의 HTTP 요청은 TLS로 암호화되어 있다.

단순히 TCP 443을 내부 서버로 전달하는 VIP는 HTTP Host를 해석해 목적지를 선택하는 구성이 아니다.

FortiGate에서 HTTPS를 받고 Host로 분기하는 예시에서는 FortiGate가 클라이언트와의 TLS 연결을 처리한다.

HTTPS의 Host 헤더는 TLS 안에 들어 있다. 이 구성에서 FortiGate가 HTTP Host를 읽으려면 TLS를 종료해야 하며, 이를 SSL Offloading이라고 한다.

```text
사용자
  == HTTPS ==>
FortiGate: 인증서 제공, TLS 종료, HTTP Host 확인
  == HTTP ==>
Real Server:80
```

클라이언트에 제공할 인증서와 대응하는 개인키를 FortiGate에 등록하고, 그 인증서가 요청받을 도메인을 포함하도록 구성한다.

```text
사용자: https://update.example.com 또는 https://portal.example.com
  → DNS 조회: PUBLIC_IP
  == HTTPS ==> FortiGate HTTPS Virtual Server: PUBLIC_IP:443
                 인증서 제공 → TLS 종료 → HTTP Host 확인
                 ├─ update.example.com → DEV_SERVER_01_IP:80
                 └─ portal.example.com → DEV_SERVER_02_IP:80
                                            → Nginx → 해당 애플리케이션
```

이 예시에서는 사용자와 FortiGate 사이는 HTTPS이고, FortiGate와 Real Server 사이는 HTTP다.

Fortinet이 설명하는 half offloading에 해당한다. 내부 서버와도 TLS로 통신하는 full offloading은 별도 구성으로 구분한다. [Fortinet의 두 방식 비교](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Difference-between-SSL-half-and-full-offloading/ta-p/381748)

따라서 Real Server의 내부 포트를 80으로 적었다고 외부 서비스도 HTTP인 것은 아니다. 외부에서 어떤 프로토콜로 받고 TLS를 어디에서 종료하는지, 그 뒤 내부로 어떤 프로토콜을 사용하는지를 나누어 봐야 한다.

또한 이 HTTP Host 분기는 SSH나 일반 DB 연결에 그대로 적용할 수 없다. Virtual Server가 다른 프로토콜도 지원한다는 사실과, HTTP Host로 목적지를 선택하는 방식은 구분해야 한다.
