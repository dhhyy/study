---
title: "TLS Certificates: Fundamentals, Issuance, and Renewal"
type: learning
status: published
created:
updated: 2026-11-02
tags:
  - certificate
  - tls
  - https
  - certbot
  - nginx
published_url: https://github.com/dhhyy/study/blob/develop/learning/tls-certificates.md
---

# TLS Certificates: Fundamentals, Issuance, and Renewal

## 인증서란?

인증서란, 이 도메인의 공개키가 이것이다, 라는 정보를 인증기관(CA)이 서명한 문서

|구분|역할|관리 방식|
|---|---|---|
|인증서|도메인·공개키·발급자·유효기간 등의 정보 제공|접속 상대에게 전달합니다|
|공개키|서명 검증이나 키 교환 등에 사용|공개할 수 있습니다|
|개인키|서버의 신원 증명 등에 사용|외부에 노출되지 않도록 보관합니다|
- 인증서 자체가 통신 내용을 암호화하는 것 X
- TLS는 인증서로 상대를 확인하고, 키 교환을 통해 만든 대칭키로 실제 데이터를 암호화

## 현재 회사 인프라 구조로 인증 과정 요약

1. 서버가 자신의 인증서와 중간 인증서를 전달한다.
2. 장치가 인증서의 도메인, 유효기간, 발급기관의 서명과 신뢰 경로를 검증한다.
3. 서버가 인증서에 대응하는 개인키를 가지고 있음을 TLS 과정에서 증명한다.
4. 양쪽이 통신에 사용할 키를 도출하고 암호화된 데이터를 주고 받는다.

```
장치가 미리 신뢰하는 루트 CA
          ↓ 서명
       중간 CA
          ↓ 서명
     서버 인증서
```

## 현재 사용 중인 인증서 구조

- example.com이라는 도메인이 존재하고, 여기에 서브 도메인을 계속 추가하면서 사용 중.
- 운영 서버 기준으로, 하나의 nginx가 들어오는 모든 요청에 대한 처리를 하고 있음. HTTPS -> nginx의 443번 입구(인증서와 개인키 사용) -> HTTP로 애플리케이션의 특정 포트, 특정 컨테이너로 통신 전달(proxy_pass)

## 인증서의 갱신
### 서버에서 확인할 수 있는 인증서 갱신 방법

[Remote Linux] 아래는 설정 확인 명령과 출력 예시이며, 이번 발행 작업에서 서버 명령을 실행한 결과는 아닙니다.

```text
$ sudo grep '^authenticator' /etc/letsencrypt/renewal/app.example.com.conf
authenticator = standalone
```
### standalone 방식과 webroot 방식의 비교

```
인증서 발급·갱신 시
인증기관 → FortiGate 80번 → 운영 서버에서 Certbot이 응답

평소 HTTPS 접속 시
사용자 → FortiGate 443번 → 운영 서버에서 Nginx가 응답
```

Certbot은 인증서를 발급, 갱신하고 nginx는 그 인증서를 사용해 https 서비스를 제공

두 방식의 가장 큰 차이는 **“인증기관의 확인 요청에 누가 응답하느냐”**이다.

| 비교                     | 현재 standalone               | webroot                 |
| ---------------------- | --------------------------- | ----------------------- |
| 확인 요청에 응답하는 프로그램       | Certbot의 임시 웹서버             | 기존 Nginx                |
| Certbot의 역할            | 직접 웹서버를 열어 응답               | 확인용 파일을 지정 폴더에 생성       |
| 외부 80번 접속              | 필요                          | 필요                      |
| 기존 Nginx가 80번을 사용 중이라면 | 포트가 겹쳐 잠시 중지하는 처리가 필요할 수 있음 | Nginx를 계속 실행한 채 확인 가능   |
| 준비할 설정                 | Certbot이 80번을 사용할 수 있도록 구성  | Nginx가 확인용 파일을 제공하도록 구성 |
- **standalone:** Certbot의 임시 웹서버
- **webroot:** 실행 중인 Nginx

쉽게 비유하면 다음과 같다.

**현재 standalone**

> Certbot이 임시 안내소를 열고, 인증기관에 직접 확인 자료를 보여줍니다.

**webroot**

> Certbot이 기존 안내소인 Nginx에 확인 자료를 놓고, Nginx가 인증기관에 보여줍니다.

두 방식 모두 **HTTP-01로 도메인 관리 권한을 확인**하며, webroot로 바꿔도 FortiGate의 외부 80번 전달 경로는 필요합니다.

**같은 서버의 같은 IP에서 Nginx와 Certbot이 동시에 80번 포트를 사용할 수 없다.**
### Q: 만약, 제공하는 API키 없이 갱신을 해야한다면?

A: API 없이도 TXT 기록을 직접 입력해 발급·갱신할 수는 있지만, 매번 사람이 작업해야 한다.
## 와일드카드 인증서를 사용하면 안되는지?
### 와일드카드 인증서란?

와일드카드는 `*.example.com`처럼 여러 서브도메인에 사용할 수 있는 인증서.
특정 웹사이트 한 곳에서 파일을 보여주는 것보다 **도메인의 DNS를 관리할 권한이 있는지**를 확인하도록 Let’s Encrypt가 요구하기 때문에 DNS-01을 사용.

### DNS 관리 업체에서 제공하는 API를 이용한 갱신 방법

지금 사용 중인 **Certbot + Let’s Encrypt 기준으로**, `*.example.com` 인증서는 **webroot가 아닌 DNS-01 방식**으로 확인을 받아와야 하는데, 자동 갱신하려면 DNS 확인용 TXT 기록을 자동으로 변경할 수 있는 수단이 필요함. 보통 DNS API와 Certbot 플러그인을 연결함.

1. Certbot이 **DNS 관리 서비스에 확인용 TXT 기록을 자동 등록**.
2. 인증기관이 그 기록을 확인하고 인증서를 갱신.
3. 갱신된 인증서를 사용하는 Nginx에 **reload로 반영**.

핵심 조건은 **DNS 기록 변경도 자동화되어야 한다는 것**임. 문서 작성 당시 사용 중인 DNS 업체에서는 필요한 API를 제공하지 않는 것으로 파악했음. 다만 API가 없다는 이유만으로 모든 자동화가 불가능한 것은 아니며, `_acme-challenge`를 다른 DNS 영역으로 위임하는 방식도 검토할 수 있음. 이 문서에서는 위임 방식을 실제로 구성·검증하지 않았음.

또한 여러 서버에서 같은 인증서를 사용한다면, 갱신된 인증서를 각 서버에 전달하고 반영하는 과정도 자동화가 되어야 한다(fortigate에서 하나의 인증서를 사용하도록 바꾼다면, 가능해 보이나 지금의 구조와 너무 다르다)

- **webroot → HTTP-01**: Nginx가 **확인용 파일**을 보여줘서 증명합니다.
- **와일드카드 → DNS-01**: DNS에 **확인용 TXT 기록**을 넣어서 증명합니다.

### 와일드카드 인증서의 내용

- **두 주소를 나열한 인증서 하나** → 두 주소 각각을 확인받아 **webroot로 발급·갱신 가능**
- **와일드카드 인증서 하나** → **DNS-01로 발급·갱신**

## 참고 자료

- [Certbot 사용 안내](https://eff-certbot.readthedocs.io/en/stable/using.html): standalone·webroot와 수동 DNS 검증의 갱신 조건
- [Let’s Encrypt 인증 방식](https://letsencrypt.org/docs/challenge-types/): HTTP-01·DNS-01과 DNS 검증 위임
