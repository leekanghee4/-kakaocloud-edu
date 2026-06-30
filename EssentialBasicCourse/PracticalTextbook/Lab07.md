# 고가용성 그룹 구성 실습

기존 AZ1에 구성한 로드밸런서의 안정성을 향상시키기 위하여 AZ2에 웹 서버, 로드밸런서를 구성하고, 고가용성 그룹을 통해 로드 밸런서를 이중화를 구성하는 실습입니다.

```mermaid
graph LR
    시작(시작) --> VM생성(다른 AZ에 Web VM 생성)
    VM생성--> LB생성(다른 AZ에 로드 밸런서 생성)
    LB생성--> HA구성(고가용성 그룹 구성)
    HA구성 --> 종료(실습 종료)
    
    %% 강조
    class VM생성,LB생성,HA구성 emphasized;
    %% 클래스 스타일
    classDef emphasized fill:#f9f,stroke:#333,stroke-width:4px;
    
```

## 1. 다른 AZ에 Web VM 생성


1. 모든 서비스 > Compute > Beyond Compute Service > 인스턴스 클릭
2. [인스턴스 생성] 버튼 클릭
3. 인스턴스 정보 입력
     - 이름 : web_server_3
     - 개수 : 1
     - 이미지 : Ubuntu 20.04
     - 인스턴스 유형 : m2a.xlarge
     - 볼륨 : 30 GB
     - 키페어 :  keypair
     - 서브넷 : vpc_1_public_sn2
     - 네트워크 인터페이스 구분 : 새 인터페이스
     - IP 할당 방식  : 자동 할당
     - 인스턴스 유형 : m2a.xlarge
     - 보안 그룹 : webserver
     - CPU 멀티스레딩: 사용

4.고급 설정을 클릭한 후, ‘사용자 스크립트’에 아래 스크립트를 복사/붙여넣기 후 [생성] 버튼 클릭

     ```bash
     #!/bin/bash
     sudo apt-get update
     sudo apt-get -y remove mariadb-server mariadb-client
     sudo apt-get -y install apache2 php mysql-client php-mysql wget
     sudo systemctl enable apache2
     cd /var/www/html
     sudo rm -f index.html
     wget https://github.com/kakaocloud-edu/tutorial/raw/main/EssentialBasicCourse/src/kakao.tar.gz -O kakao.tar.gz
     tar -xvf kakao.tar.gz
     sudo mv kakao/{index.php,get_user_list.php,add_user.php} /var/www/html/
     sudo systemctl restart apache2

     ```
5. 인스턴스가 Active 상태로 변경되면, 더보기 및 ‘퍼블릭 IP 연결’ 클릭
6. 퍼블릭 IP의 생성 자동 할당을 선택 후 [연결] 버튼 클릭
7. web_server_3의 퍼블릭 IP릍 통해 접근한 (41P 참조) 웹 브라우저 화면으로 이동
8. 호스트 입력 칸에 53p에서 복사해 놓은 ‘엔드포인트 URL’ 붙여 넣고 [유저 이름 가져오기] 버튼을 클릭하여 DB 연결을 확인

## 2. 다른 AZ에 로드 밸런서 생성


1. ‘모든 서비스’ > Networking  > Load Balancing > 로드 밸런서 클릭
2. [로드 밸런서 생성] 버튼 클릭
3. ‘Application Load Balancer’ 설정 후 [생성] 버튼 클릭
4. Load Balancing > 대상 그룹 클릭
5. [대상 그룹 생성] 버튼 클릭
6. ‘1단계 대상 그룹 구성’ 정보 입력
     - 로드 밸런서 : App_LB_B
     - 리스너 : HTTP : 80
     - 대상 그룹 이름 : App_Target_B
     - 프로토콜 : HTTP
7. ‘1단계 대상 그룹 구성’ 정보 입력 후 [다음] 버튼 클릭
     - 알고리즘 : 라운드 로빈
     - 고정세션 : 미사용
     - 상태 확인 : 사용
     - 상태 확인 프로토콜 : HTTP
     - 나머지는 기본 값
8. ‘대상 추가(선택)’ 정보 선택 후, 대상 2개 선택 후, [다음] 버튼 클릭
     - 대상 : web_server_3
     - 트래픽 포트 : 80
     - [선택한 대상 추가] 버튼 클릭
     - [다음 버튼 클릭
9. ‘고급설정(선택)’에서 다음 버튼 클릭
10.검토 화면에서 [생성] 버튼 클릭
11. 로드 밸랜서에 퍼블릭 IP를 연결
     - 더보기를 클릭
     - 새로운 퍼블릭 IP 자동 생성 및 연결 선택
     - [연결] 버튼 클릭
12. App_LB_B의 퍼블릭 IP로 정상 접근 확인

## 3. 고가용성 그룹 구성


1. Load Balancing > 고가용성 그룹 클릭
2. [고가용성 그룹 생성] 버튼을 클릭
3. 고가용성 그룹  정보 등록
     - Application Load Balancer 선택
     - 체계 : 인터넷 경계
4. 고가용성 그룹  정보 등록(계속) 후 [생성] 버튼 클릭
     -고가용성 그룹 이름 : App_HA
     - 서브넷 / LB 노드 : vpc_1_public_sn1 /  App_LB_A, vpc_1_public_sn2 / App_LB_B
     - [생성] 버튼 클릭
5. App_HA 고가용성 그룹의 DNS 이름을 복사한 후, 웹 브라우저에서 테스트



