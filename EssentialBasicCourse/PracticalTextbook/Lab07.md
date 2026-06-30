# 고가용성 그룹 구성 실습

기존 AZ1에 구성한 로드밸런서의 안정성을 향상시키기 위하여 AZ2에 웹 서버, 로드밸런서를 구성하고, 고가용성 그룹을 통해 로드 밸런서를 이중화를 구성하는 실습입니다.

```mermaid
graph LR
    시작(시작) --> VM생성(다른 AZ에 Web VM 생성)
    VM생성--> LB생성(다른 AZ에 로드 밸런서 생성)
    LB생성--> HA구성(고가용성 그룹 구성)
    HA구성 --> 종료(실습 종료)
    
    %% 강조
    class VM생성,파일업로드,파일접근,권한설정,LifeCycle설정,버킷삭제 emphasized;
    %% 클래스 스타일
    classDef emphasized fill:#f9f,stroke:#333,stroke-width:4px;
    
```

## 1. 다른 AZ에 Web VM 생성


1. 카카오 클라우드 콘솔 > 전체 서비스 > VPC 접속
2. 좌측 Subnet 클릭
3. Subnet 만들기 클릭
     - VPC 선택 : `vpc_1`
     - Subnet 이름 : `main-b`
     - Availability Zone: `kr-central-2-b`
     - IPv4 CIDR 블록: `172.30.32.0/20`
4. 만들기 클릭

## 2. 다른 AZ에 로드 밸런서 생성


1. 카카오 클라우드 콘솔 > 전체 서비스 > Virtual Machine > Instance
2. 인스턴스 만들기 클릭
     - 이름 : `vm_5`
     - Image : `Ubuntu 20.04`
     - Instance 타입 : `m2a.large`
     - Volume : `30 GB`
     - Key Pair : `keypair`
     - VPC : `vpc_1`
3. 새 Security Group 생성 클릭
     - Security Group 이름 : `vm_5`
     - Inbound
          - 프로토콜: TCP, 패킷 출발지: `0.0.0.0/0`, 포트번호: `22`
          - 프로토콜: TCP, 패킷 출발지: `0.0.0.0/0`, 포트번호: `80`
     - Outbound
          - 프로토콜: `ALL`
          - 패킷 목적지: `0.0.0.0/0`
4. 새 인터페이스 클릭
     - Subnet : `main-b`(kr-cenrtral2-b의 Public 서브넷)
     - IP 할당 방식: `자동`
5. 고급설정 버튼 클릭
     - 사용자 스크립트에 아래 내용 붙여넣기
     #### **lab7-2-4**
     ```bash
     #!/bin/bash
     sudo apt update -y
     sudo apt install -y apache2
     sudo systemctl start apache2
     sudo systemctl enable apache2
     ```
6. 만들기 버튼 클릭
7. 카카오 클라우드 콘솔 > 전체 서비스 > Virtual Machine > Instance
8. 생성된 vm_5 인스턴스의 우측 메뉴바 클릭 > Public IP 연결 클릭
     - `새로운 Public IP를 자동으로 할당` 선택
9. 확인 버튼 클릭
10. vm_5의 Public IP 복사
11. 브라우저창에 입력
12. apache 웹서버 Test페이지가 나오는 것을 확인

## 3. 고가용성 그룹 구성


1. 카카오 클라우드 콘솔 > 전체 서비스 > DNS 접속
2. DNS Zone 만들기 버튼 클릭
     - DNS Zone 이름 : `kakaocloud-edu.com`
3. 만들기 버튼 클릭
4. 생성된 `kakaocloud-edu.com` DNS 클릭
5. 레코드 추가 버튼 클릭
     - 레코드 타입 : `A`
     - TTL : `60`
     - 값 : `{연결하려는 VM의 Public IP}`
     - **Note**: "{연결하려는 VM의 Public IP}" 부분을 실제 IP 주소로 교체하세요.
     _**note**: 본 실습에서는 kr-central-2-a에 위치한 Load Balancer와  kr-central-2-b에 위치한 vm_5를 연결함
6. 추가 버튼 클릭
7. 추가 설정
     - 사용 도메인의 네임서버를 카카오클라우드의 네임서버로 바꿔주어야함
     - 도메인 구입처의 도메인 설정창에서 네임서버를 변경
     - 본 실습에서는 ‘가비아’ 라는 도메인 제공 서비스를 이용하였음


