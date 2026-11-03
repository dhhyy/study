---
title: "Nginx·Docker의 요청 전달과 포트 구조 이해하기"
type: article
status: ready
created:
updated: 2026-11-03
tags:
  - network
  - infrastructure
published_url:
---
<h1 style="text-align: center;">Nginx·Docker의 요청 전달과 포트 구조 이해하기</h1>
<p style="text-align: center;"><em>FortiGate 뒤에서도 Nginx와 Docker가 필요한 이유</em><br>
<small>Fortigate에서 들어온 요청을 각 서버, 각 서비스들이 어떻게 처리하는지에 대한 상세 기록</small></p>
FortiGate가 내부 서버를 선택한 뒤에는 서버 안의 서비스까지 요청을 전달해야 함. 한 서버에서 여러 애플리케이션이 실행되므로, Nginx가 Host나 URL 경로에 따라 목적지를 다시 선택함.

```text
FortiGate
  → DEV_SERVER_02_IP:80의 Nginx
     ├─ portal.example.com → portal-app:8082
     └─ geocode.example.com → geocode-service:8087
```

설정을 단순화한 예시는 다음과 같음.

```nginx
server {
    listen 80;
    server_name portal.example.com;

    location / {
        proxy_pass http://portal-app:8082;
    }
}

server {
    listen 80;
    server_name geocode.example.com;

    location / {
        proxy_pass http://geocode-service:8087;
    }
}
```

이 예시는 Nginx와 대상 컨테이너가 이름으로 통신할 수 있는 Docker 네트워크 구성을 전제로 함. 네트워크를 공유해야 이 설정을 사용할 수 있다.

컨테이너 이름 해석과 연결이 되지 않으면 외부 방화벽을 통과한 뒤에도 요청이 실패할 수 있음.

## 호스트 포트와 컨테이너 내부 포트

Docker에서 `3002:3000`으로 공개한 Grafana를 생각하면 두 포트의 차이가 보임.

```text
서버의 공개 포트로 접근: 서버 IP:3002 → 컨테이너:3000
같은 Docker 네트워크에서 접근: grafana:3000
```

따라서 같은 Docker 네트워크에 있는 Nginx의 `proxy_pass http://grafana:3000`은 호스트 공개 포트 3002와 달라도 의도한 설정일 수 있음.

반대로 Docker에서 호스트 포트를 공개해도 회사 PC에서 그 주소까지 갈 수 있는 경로가 생기지는 않음. 직접 포트포워딩으로 접근한다면 FortiGate의 VIP와 정책이 필요하고, 기존 웹 진입점을 사용한다면 프록시 경로가 해당 서비스까지 이어져야 함. 호스트 자체의 방화벽과 바인딩 주소도 접근 범위에 영향을 줌.

## 외부 80과 내부 Nginx의 80

```text
외부 진입점: PUBLIC_IP:80
내부 목적지: DEV_SERVER_01_IP:80 또는 DEV_SERVER_02_IP:80
```

숫자는 같지만 서로 다른 주소의 포트임. 내부 Nginx가 80을 쓰는 것과 외부 공인 IP의 80을 어느 대상으로 전달할지는 별개의 문제임.

인증서 갱신은 기존 운영 서버의 Certbot standalone 사용 기록과 FortiGate의 ACME 처리 방식을 구분해 살펴봐야 함. 인증서별로 현재 어느 장비가 발급·갱신을 담당하는지는 확인이 필요한 항목임.

HTTP-01 방식이라면 외부 80의 아래 요청이 실제 인증 처리 주체까지 도달해야 함.

```text
http://도메인/.well-known/acme-challenge/...
```

따라서 개발 웹 서비스를 추가하면서 외부 80의 목적지를 바꾸면 기존 인증서 갱신 경로에 영향을 줄 수 있음. HTTP-01 경로에 대한 설명을 모든 인증 방식의 조건으로 일반화하지 않도록 주의함.
