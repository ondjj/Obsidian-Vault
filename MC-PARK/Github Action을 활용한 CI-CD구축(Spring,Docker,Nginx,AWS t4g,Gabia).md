
## 목차

1. [EC2_인스턴스_생성(t4g)](#EC2_인스턴스_생성(t4g))
2. [RDS_생성하기](#RDS_생성하기)
3. [S3_버킷_생성](#S3_버킷_생성)
4. [EC2_인스턴스에_Docker_docker-compose_설치](#EC2_인스턴스에_Docker_docker-compose_설치)
5. [가비아_도메인_등록](#가비아_도메인_등록)
6. [ACM_인증서_발급받기](#ACM_인증서_발급받기)
7. [Github-Actions-Docker-compose](#Github-Actions-Docker-compose)


# 1. EC2_인스턴스_생성(t4g)

![](https://i.imgur.com/hnj1tcm.png)

## 목표
- AWS EC2(t4g) 인스턴스에 Docker를 활용한 Spring Boot 서버 배포
- Github Actions를 활용한 CI/CD 구축
- AWS RDS, S3 설정
- Nginx reverse proxy 설정


## 아키텍처

![](https://i.imgur.com/lUUzgum.jpg)
## EC2 인스턴스 생성

> [!NOTE]
> - EC2는 AWS에서 제공하는 클라우드 컴퓨팅 서비스로, 아마존이 사용자들에게 독립된 컴퓨터를 임대해주는 서비스이다.
> - EC2를 사용해 가상 서버를 구축하고, 보안 및 네트워킹을 구성해서 빠르게 애플리케이션을 배포할 수 있다.
> - 인스턴스 유형, OS, 소프트웨어 패키지 등을 선택할 수 있고, memory/cpu/storage/booting partition size 등을 선택할 수 있는 유연한 클라우드 호스팅 서비스이다.


### 1.1 AWS Region 설정

Region에 따라 인스턴스 위치가 결정되기 때문에 외국으로 하면 속도가 느릴 수 있다.
console 우측 상단 Region을 서울로 설정한다.

![](https://i.imgur.com/kILVQrc.png)

### EC2 서비스 - 인스턴스 생성

![](https://i.imgur.com/16qn0ND.png)

인스턴스 시작을 통해 EC2를 생성할 수 있다.

![](https://i.imgur.com/4fZhbm2.png)

이름은 비워놓아도 상관 없지만, 임의 값으로 나오기 때문에 프로젝트에 맞춰 설정해주자

### ✅ AMI 및 인스턴스 유형 선택


![](https://i.imgur.com/CznJ9l9.png)


현 시점(2023-09)에는 1년 프리 티어를 제외하고 무료로 사용 할 수 있는 인스턴스 유형은 `t4g.small`이 유일하기 때문에 AMI는 `t4g.small`로 결정했다.
t4g.small은 t2.micro 대비 vCPU와 REM이 2배로 기존 무료 평가판에서 스케일 업도 가능하다. 단점으로 ARM 아키텍처로 사용하는 툴이 이를 지원해야한다는 점이지만 대부분 극복 가능하다.

> [!NOTE]
> AMI(**Amazon Machine Image**)는 인스턴스를 시작하는데 필요한 정보를 제공하는 이미지로, 서버 구성을 어떻게 할지 선택하는 것이라 생각하면 된다.
> 한 AMI로 여러 인스턴스를 생성할 수 있다.

### ✅ 키 페어 생성

![](https://i.imgur.com/WoOrqbu.png)

EC2 서버에 SSH 접속을 하기 위해 키 페어를 생성하자

![](https://i.imgur.com/bYN2PRi.png)

`새 키 페어 생성`을 클릭해 원하는 이름을 입력해 생성한다.  
생성하면 자동으로 `mc-park.pem` 파일이 다운로드 되고, SSH 환경에 접속하기 위해서는 해당 키 파일이 존재하는 위치로 가서 `ssh` 명령어를 실행하면 된다.
> [!info] 
> 한 번 다운로드 받은 후에는 다시 다운로드 받을 수 없다.

### ✅ 네트워크 및 스토리지 선택

![](https://i.imgur.com/fPv2cpb.png)


네트워크 설정은 EC2에 접속을 허용하는 ACL을 생각하면 된다. 생성 이후에 `보안 그룹`을 별도로 설정할 것이기 때문에 내 IP에서만 SSH 트래픽 접속이 가능하도록 설정해준다.

![](https://i.imgur.com/c3MX5Hx.png)

프리티어는 최대 30까지만 지원하기 때문에 스토리지 부분도 변경해준다. 볼륨 유형은 범용 SSD로 선택해야 한다.
(만약 Provisioned IOPS SSD(프로비저닝된 IOPS SSD)를 선택한다면 사용하지 않아도 활성화한 기간만큼 비용이 발생하게 된다.)

다른 세부 설정은 기본 값으로 두고 요약 페이지를 통해 전체 설정을 확인 한 후 이상이 없다면 인스턴스 시작을 통해 생성한다.
### ✅ 인스턴스 시작하기

![](https://i.imgur.com/LXUgYmq.png)

인스턴스가 잘 생성되었다면 탄력적 IP 설정을 통해 고정 IP를 획득하고 보안 그룹을 추가한다.


# 1-2. 탄력적(Elastic) IP

EC2 인스턴스를 생성할 때는 항상 새로운 IP를 할당한다. 인스턴스를 중지하고 재시작하는 경우에도 새로운 IP가 할당 되기 때문에 매 번 관련 설정을 변경해야하는 번거로움이 생긴다.

`탄력적 IP`란 외부에서 인스턴스에 접근 가능한 고정 IP이다. 고정적인 IP를 가질 수 있도록 탄력적 IP 주소를 할당해주어야 한다.

> [!NOTE]
> 탄력적 IP를 만들고 EC2에 연결하지 않으면 과금이 되기 때문에 주의해야 한다. 필요한 만큼만 생성하는 것이 중요하다.


## 1-2.1 탄력적 IP 할당하기

EC2 페이지 좌측 메뉴 `네트워크 및 보안`에서 탄력적 IP를 찾을 수 있다.

![](https://i.imgur.com/pA8ARPI.png)
![](https://i.imgur.com/CPCp6cu.png)

여기서 탄력적 IP 주소 할당을 통해 새로운 IP 주소를 추가해보자

![](https://i.imgur.com/m0LHSBJ.png)

특별히 변경할 내용은 없으니 바로 할당 버튼을 클릭한다.

## 1-2.2 탄력적 IP 인스턴스와 연결하기

![](https://i.imgur.com/FMy8PiY.png)

생성한 탄력적 IP를 선택한 후 작업 - 탄력적 IP 주소 연결을 누른다.

![](https://i.imgur.com/k7MN4EY.png)

현재 내 인스턴스 목록을 선택할 수 있고 연결된 프라이빗 IP까지 선택 가능하다.

연결 버튼을 클릭한 후, 인스턴스 정보를 확인하면 `퍼블릭 IPv4 주소`와 `탄력적 IP 주소`의 값이 동일하다는 것을 확인할 수 있다.

# 1-3. SSH 클라이언트 서버 접속하기


![](https://i.imgur.com/YRofTvj.png)

1번부터 4번까지 나열된 순서대로 진행한다.

터미널을 통해 다운받은 키 페어가 있는 파일 위치로 이동하자.

![](https://i.imgur.com/LpjMwK1.png)

## 1-3.1키 파일 권한 변경

```sql
$ chmod 400 키페어이름.pem
```

## 1-3.2SSH 접속하기

SSH 접속은 퍼블릭 DNS 또는 퍼블릭 IP를 사용해 인스턴스에 접속할 수 있다.

```javascript
ssh -i "키페어이름.pem" ubuntu@<퍼블릭DNS>

ssh -i "키페어이름.pem" ubuntu@<퍼블릭IP>
```

## 1-3.3호스트를 등록해 간편 접속

### ✅ ./ssh 디렉터리로 키 페어 파일 복사하기

```javascript
$ cp 키페어이름.pem ~/.ssh/
```

### ✅ 키페어 파일의 권한 변경하기

```javascript
$ cd ~/.ssh
$ chmod 600 키페어이름.pem
```

### ✅ ~/.ssh/config 파일 생성하기

```javascript
$ vi ~/.ssh/config
```

이미 파일이 존재한다면 맨 아래에 입력하면 된다.  
User는 ubuntu를 선택했다면 `ubuntu`이고 그 외는 `ec2-user`이다.

> [!NOTE]
> - Host {원하는 호스트 이름}  
> - User {유저 이름}  
> - HostName {탄력적 IP}  
> - IdentityFile {키 페어 파일 위치}
> ![](https://i.imgur.com/H3mNhX1.png)


설정을 끝냈다면 host 이름으로 간단하게 접속할 수 있다.

```null
ssh  mc-park
```


# 1-4. 보안 그룹 설정하기

보안 그룹은 AWS에서 제공하는 방화벽 기능이다. RDS 처럼 외부에서 함부로 접근하면 안되는 인스턴스는 허용된 IP에서만 접근하도록 설정이 필요하다.

> - 인바운드 : 외부에서 EC2 인스턴스 내부로의 접근을 허용
> - 아웃바운드 : EC2 인스턴스 내부에서 외부로의 접근을 허용


## 1-4.1 현재 보안 그룹 확인

보안 그룹은 인스턴스 요약 하단 혹은 EC2 페이지 좌측 패널에서 확인 할 수 있다.
처음 인스턴스를 생성했을 때 자동으로 할당 되는 Default 보안그룹은 22번 포트의 내 IP에 대한 TCP 연결을 허용하도록 설정되어 있다.
우리는 RDS, IPv4, IPv6 그리고 Docker로 구동되는 스프링에 대한 보안을 설정해야한다.

## 1-4.2 보안 그룹 ID 리스트 확인하기

![](https://i.imgur.com/DxjOsP5.png)

![](https://i.imgur.com/jlbzAJi.png)


  
default로 생성된 보안 그룹은 `launch-wizard-1`이다. 보안 그룹은 인스턴스와 별개로 존재하기 때문에 한번 만들어두면 새로운 인스턴스에도 적용할 수 있다.

보안 그룹 생성 버튼으로 새로운 보안 그룹을 만들어보자.

## 1-4.3 보안 그룹 생성하기

![](https://i.imgur.com/B9MLtn4.png)

보안 그룹의 이름과 설명을 추가한다.

### ✅ 인바운드 규칙 편집하기

![](https://i.imgur.com/RQ6cDrM.png)


인바운드 규칙은 외부에서 EC2로 요청할 때 허용할 IP 대역을 설정하는 것이다.

로컬 PC에서 서버에 접속할 수 있게 SSH를 추가하고 소스를 내 IP로 추가한다. 소스를 선택하면 자동으로 IP가 추가되고, 특정 IP를 넣으려면 사용자 지정으로 추가할 수 있다.

여러 사람이 함께 작업하는 프로젝트라면 각각의 로컬 PC IP를 전부 추가해주어야 한다.

SSH, HTTP, HTTPS와 같은 기본 프로토콜들과 스프링 부트 프로젝트까지 사용자 지정으로 추가해준다.

### ✅ 아웃바운드 규칙 편집하기

아웃바운드 규칙은 따로 설정할 필요가 없어서 그대로 둔다.

이후 보안 그룹 생성 버튼을 눌러주어 작업을 완료한다.

## 1-4.4 보안 그룹 변경하기

![](https://i.imgur.com/esKRl99.png)


인스턴스 요약 창으로 돌아와 우측 상단 **작업 > 보안 그룹 변경 버튼**을 눌러준다.

![](https://i.imgur.com/ne3CgL4.png)
(위 이미지는 참고한 블로그에서 가져온 이미지로 보안 그룹 이름이 진행 과정과 상이하지만, 그 외에는 변동 사항이 없습니다.)

보안 그룹은 여러 개를 동시에 설정할 수 있다. 기존에 설정되었던 `launch-wizard-1`을 제거한 후 새로 생성한 보안 그룹을 추가한다.

![](https://i.imgur.com/FEm9OZF.png)
(위 이미지는 참고한 블로그에서 가져온 이미지로 보안 그룹 이름이 진행 과정과 상이하지만, 그 외에는 변동 사항이 없습니다.)


# RDS_생성하기
![](https://i.imgur.com/TNFLT3F.png)


> [!NOTE]
> RDS란 AWS에서 관계형 데이터베이스를 더 쉽게 설치, 운영 및 확장할 수 있는 웹서비스이다.
> - Amazon Aurora(MySQL 호환), Amazon Aurora(PostgreSQL 호환), MySQL, MariaDB, PostgreSQL, Oracle 등 7가지 엔진을 제공한다.
> - 인프라 프로비저닝, 데이터베이스 설정, 백업, 장애 탐지, 복구 등의 작업을 자동화하여 비용과 시간을 감소시키고, 효율적인 데이터베이스 관리가 가능하다.


## 1-1.RDS 이동 및 데이터베이스 생성

![](https://i.imgur.com/TPp9AaO.png)


![](https://i.imgur.com/98ZdfsD.png)

생성한 데이터베이스가 없다면 대쉬보드에서 바로 생성 버튼으로 접근 가능하다. 좌측 데이터베이스 메뉴를 클릭해 데이터베이스 생성 버튼으로도 접근 가능하다.

### ✅ 데이터베이스 엔진 옵션 선택하기

![](https://i.imgur.com/1kCU1cc.png)

사용할 데이터베이스 엔진을 선택한다. 여기서는 MySQL을 선택했다.

![](https://i.imgur.com/Htgyo0H.png)

템플릿은 프리 티어를 선택한다. 자동으로 가용성 및 내구성 필드가 비활성화된다.

### ✅ 데이터베이스 설정 입력하기

![](https://i.imgur.com/yixhOtp.png)

데이터베이스 이름, 마스터 이름, 마스터 암호를 입력한다.  
데이터베이스에 접근할 때 사용할 정보이므로 안전한 곳에 기록해두자

### ✅ 인스턴스 구성/스토리지 설정하기

![](https://i.imgur.com/VQ3qLhb.png)

프리 티어는 대부분 기본 설정을 사용하면 된다.

> 스토리지 자동 조정 활성화는 체크 해제해놔야 한다. 체크를 해제하지 않으면 임계값이 초과되었을 때 자동으로 스토리지가 늘어나서 과금될 가능성이 있다.

### ✅ VPC 보안 그룹 설정하기


![](https://i.imgur.com/qiSTfgl.png)

EC2랑 연결하는 옵션을 선택해서 생성하면 외부접속이 불가능하다. 그렇기에 연결을 안하는 옵션을 선택해줬다.

서브넷 그룹을 새로 생성해줄 것이기 때문에 새 VPC 생성을 선택해준다.

퍼블릭 액세는 `예`로 해주어야 외부에서 접속할 수 있다. 보안 그룹도 새로 생성해준다.

새 VPC 보안 그룹 이름은 자유롭게 작성하자.

### ✅ 추가 설정

![](https://i.imgur.com/7McqUzg.png)

- 초기 데이터베이스 이름을 설정
- 자동 백업 체크 해제

프리티어 요금에 대한 안내를 읽고, 데이터베이스 생성을 눌러 생성을 완료한다.
![](https://i.imgur.com/z87sSr8.png)

![](https://i.imgur.com/I4LQeJA.png)

**RDS > 데이터베이스** 에서 성공적으로 생성된 것을 확인할 수 있다.


## 1-2. 보안그룹 설정

RDS 인스턴스를 생성할 때 보안그룹을 생성해주었다. 데이터베이스는 서버에서 접근 가능해야 하기 때문에 보안 그룹 설정이 추가로 필요하다.

> 서버는 탄력적 IP를 뜻한다.


## 1-2.1 보안 그룹 확인하기


![](https://i.imgur.com/cBxsdMW.png)

RDS 인스턴스 정보에서 보안 그룹 규칙을 확인할 수 있다. 기본 설정은 로컬 IP만 인바운드 규칙에 추가되어 있다.  
아웃바운드는 모든 트래픽에 대해 열려있다.

> [!NOTE]
> RDS 보안 그룹은 EC2 보안 그룹이랑 별도로 관리하지 않고 같은 곳에서 관리한다. 보안 그룹을 편집하기 위해서는 EC2 대시보드로 이동하거나 저 보안 그룹 이름을 클릭해 페이지를 열어야 한다.


## 1-2.2 보안 그룹 편집하기

![](https://i.imgur.com/jj5vJ1h.png)

  
EC2 인스턴스의 탄력적 IP를 추가해준다.

## 1-3. RDS 접속 테스트

이제 RDS 인스턴스 생성 및 보안 그룹 설정이 완료되었으니 테스트를 해보자.

## 1-3.1 엔드포인트와 포트 확인하기

![](https://i.imgur.com/rlz3ytL.png)


RDS 인스턴스 정보에서 엔드포인트와 포트 번호를 확인하자. 해당 정보와 마스터 이름, 암호를 이용해 RDS에 접속할 수 있다.

## 1-3.2 MySQL Workbench로 확인하기

만약 사용하는 툴이 있다면 어떤걸 사용하든 상관 없다. 여기서는 MySQLWorkbench를 활용한다.

먼저 워크 벤치를 킨 후 아래 이미지와 같이 `+` 버튼을 누른다.

![](https://i.imgur.com/BcQAFeA.png)

![](https://i.imgur.com/UJX4tJk.png)

- Hostname : RDS 엔드포인트
- Username : RDS 생성 시 입력했던 정보
- Password : RDS 생성 시 입력했던 정보
- Connection Name : 워크 벤치에서 RDS에 접근하고자 하는 이름 (임의 작성)

정보를 입력했다면 Test Connection을 통해 핑 테스트를 해보자


## 1-4. RDS 파라미터 그룹 설정하기

초기 `AWS RDS`를 셋팅할 때 파라미터 그룹을 하나 생성하여 `character set`과 `collation` 그리고 `Amazon RDS MySQL DB 인스턴스의 함수`, 절차 및 트리거를 활성화 하기 위한 `log_bin_trust_function_creators 설정`이 필요하다.

## 1-4.1 파라미터 그룹 생성하기

![](https://i.imgur.com/Qq5WOim.png)


RDS 좌측 메뉴의 파라미터 그룹을 클릭해 생성으로 접근한다.


![](https://i.imgur.com/3ZAQi29.png)


파라미터 그룹 패밀리는 RDS에서 생성한 DB 인스턴스와 맞게 선택해주고 이름과 설명을 입력해 생성한다.

![](https://i.imgur.com/ceSw9Wk.png)

생성한 파라미터 그룹을 클릭해 편집하자.

### ✅ Time Zone

`time_zone` 검색한 후에 Asia/Seoul로 변경한다.

![](https://i.imgur.com/uyyQUcV.png)

### ✅ Character Set

`character_set` 검색한 후에 나오는 값을 모두 `utf8mb4`로 변경한다.

`utf8mb4는 이모지까지 지원해주는 utf8이다.`

![](https://i.imgur.com/hGxISNs.png)


`collation` 검색한 후에 나오는 값을 `utf8mb4_general_ci`로 변경해준다.

![](https://i.imgur.com/l9fhTqB.png)


### ✅ Max Connection

`max_connections` 검색한 후에 값을 150으로 수정해준다.

![](https://i.imgur.com/fgvpyKN.png)


### ✅ 최종 변경사항 확인하기

![](https://i.imgur.com/wyjPuHd.png)


## 4-2. RDS 파라미터 그룹 변경하기

![](https://i.imgur.com/caA9HAT.png)


RDS 인스턴스 수정을 통해 추가 구성에서 DB 파라미터 그룹을 변경한다.

![](https://i.imgur.com/SFo3i95.png)

즉시 적용을 선택해 수정한다.


# S3_버킷_생성

![](https://i.imgur.com/oyIxs9k.png)

## 1-1 S3 버킷 생성

> [!NOTE]
> S3(Simple Storage Service)란 확장성, 데이터 가용성, 보안 및 성능을 제공하는 객체 스토리지 서비스이다.
> - S3는 데이터를 버킷 내의 객체로 저장한다.
> - 객체는 해당 파일을 설명하는 메타데이터이며, 버킷은 객체에 대한 컨테이너이다.
> - REST 인터페이스를 통해 저장 / 삭제 / 조회가 가능합니다.

![](https://i.imgur.com/0Un7Hzg.png)


![](https://i.imgur.com/4Cje1nY.png)

버킷 만들기를 클릭한다.

![](https://i.imgur.com/inUiI87.png)

버킷 이름은 고유값으로 설정해야 한다.

![](https://i.imgur.com/kFLsXxP.png)


Spring을 통해 S3에 파일을 업로드하기 위해서는 ACL을 활성화해야 한다. 비활성화 상태라면 `AmazonS3Exception: The bucket does not allow ACL`이 발생한다

S3의 퍼블릭 액세스 권한을 설정한다. 외부에 S3를 공개할 경우 모든 퍼블릭 액세스 차단을 체크 해제하면 된다.


>[!note] 
>퍼블릭 액세스를 차단할 경우, IAM에서 AWSAccessKeyId, AWSSecretKey를 발급받고 키를 이용해서 S3 객체에 접근할 수 있다.


## 1-1.1 버킷 정책 설정

![](https://i.imgur.com/jCRljhV.png)


S3 인스턴스를 클릭한 후 권한 메뉴를 클릭한다. 아래에 버킷 정책에서 편집을 누르면 에딧이 가능한 창으로 변한다.

```null
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::bucket이름/*"
        }
    ]
}
```


해당 내용 입력 후 설정을 완료한다.

## 1-2. AWS IAM 사용자 생성

![](https://i.imgur.com/57p15Ts.png)

![](https://i.imgur.com/E9M4yFi.png)

사용자 추가를 누른다.

![](https://i.imgur.com/ZA6N8la.png)
사용자 이름을 설정하고 프로그래밍 방식 액세스 방식을 선택한다.

![](https://i.imgur.com/HjlUhUR.png)
기존 정책 직접 연결을 누르고, `AmazonS3FullAccess` 조회 후 선택한다.

이후는 다음을 눌러서 추가를 완료한다.

![](https://i.imgur.com/xCAhOgX.png)

액세스 키, 비밀 액세스 키는 나중에 스프링 설정에서 쓰이기 때문에 메모해두자.

# EC2_인스턴스에_Docker_docker-compose_설치


![](https://i.imgur.com/e5YpyMP.png)
### 도커 설치

docker 위에 Springboot 서버를 올려 배포할 예정이기 때문에 EC2 인스턴스에 docker와 docker-compose를 설치해보자.

```
t4g 인스턴스의 가장 큰 특징은, 로컬 컴퓨터에서 일반적으로 쓰이는 Amd64(x86-64) 아키텍처가 아닌 arm64 기반 프로세서를 사용한다는 사실입니다. 그 덕에 동일 비용으로 훨씬 뛰어난 성능을 발휘할 수 있지만, 여러 호환성 문제를 겪을 수도 있습니다.사실 겉으로 보기엔 기존에 쓰던 인스턴스들과 큰 차이를 느낄 수는 없었는데요. 문제는 도커를 설치하려 할때부터 시작됐습니다.

- 참고 블로그(velog.io/@infl_veggie) 발췌 -
```

참고한 블로그는 2021년 05월 게시글로 지금부터 약 2년전 게시글이다. 지금은 게시글 내용과 달리 amd/arm 명령어가 나뉘어있지 않고, 동일한 명령어를 통해 docker를 arm 환경에서 안전하게 설치할 수 있다.

<docker docs https://docs.docker.com/engine/install/ubuntu/>

`docker docs`를 따라 설치를 진행한다.이 과정에서 딱히 어려움은 없고, 테스트용인 `hello-world` image가 잘 빌드된다면 성공적으로 ARM 환경에서 docker를 설치한것이다.

#### Docker 실행

docker 활성화
```null
$ sudo systemctl enable docker
```

docker 실행
```null
$ sudo service docker start
```

docker 상태 확인
```null
$ service docker status
```

![](https://i.imgur.com/HFrMTC2.png)

사용자 추가
```null
$ sudo usermod -aG docker ubuntu
```
### 도커 컴포즈 설치

도커 컴포즈 설치 역시 해당 블로그를 통해 설치 방법을 확인할 수 있다.

아래 `nemchik` 의 글을 통해 docker-compose를 설치 할 수 있고 추가로 실행 권한까지 적용한다.
![](https://i.imgur.com/29Q842C.png)

이 후 우리가 사용하고하자는 이미지를 빌드하는 과정은 Github Action을 통해 자동화 할 예정이니 Docker와 Docker-compose를 설치하는 내용만 확인하고 다른 내용은 필요하다면 읽어보길 바란다.

설치 확인

```null
docker-compose --version
```

설치 확인

```null
docker-compose --version
```

![](https://i.imgur.com/Z1uIAXs.png)


# 가비아_도메인_등록


만약 토이 프로젝트나 가벼운 사이드 프로젝트를 진행한다면  [freenom](https://my.freenom.com/clientarea.php) 을 통해 도메인을 무료로 사용할 수 있지만, 제약이 따르기 때문에 여기서는 가비아를 통해 도메인을 구매하여 사용한다.
`.com co.kr ...` 과 같은 TLD는 비용이 비싸기 때문에 `info, store` 같이 저렴한 TLD를 사용하면 저렴하게 도메인을 사용할 수 있다.


### ✅ Freenom 주의사항

> 1. 제3세계 도메인은 KT DNS에서 불법적 이용으로 판단해 막는 경우가 있다고 한다.  
>     는 [이게 실제로 일어났다?](https://velog.io/@jmjmjmz732002/%EB%AC%B4%EB%A3%8C-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%93%B0%EC%A7%80%EB%A7%88%EC%9A%94-%EC%B0%B8%EC%B9%98%EB%A7%88%EC%9A%94-DNSPROBEFINISHEDNXDOMAIN)
> 2. 연장 만료일로부터 2주 안에 연장을 하지 않으면 스페셜 도메인으로 분류되어 비용을 지불해야 사용할 수 있다.
> 3. 일정 트래픽 이상을 발생시켜 인기 도메인이 되면 스페셜 도메인이 되어 유료로 강제 전환된다.
> 4. Freenom에서 제공하는 DNS 혹은 Google DNS와 같은 국제적으로 인지도가 있는 DNS 서버를 사용하면 가능하지만, 일부 국내 통신사 DNS(KT 등)를 사용하는 환경에서 접속이 안되는 경우가 종종 발생된다.


### 가비아 도메인

 - [gabia](https://domain.gabia.com/regist/regist_domain) 로 접속해 원하는 도메인을 구매 한다. 
 - My가비아로 이동하면 내가 구매한 도메인을 확인할 수 있다.

![](https://i.imgur.com/USm9sww.png)

 관리 버튼을 눌러 도메인 관리 페이지로 이동하면 네임서버를 설정하는 곳을 발견할 수 있고, 이 곳에 `AWS Route 53` 을 통해 발급 받은 네임 서버 값을 등록한다.

![](https://i.imgur.com/CRvbf2N.png)

## Route53 등록

> [!NOTE]
> AWS Route53은 클라우드 DNS 웹 서비스이다. 도메인 이름을 IP 주소로 변환한다.
> 사용자의 요청을 AWS EC2 인스턴스, Elastic Load Balancing 로드 밸런서, AWS S3 버킷 등 AWS에서 실행되는 인프라에 효과적으로 연결한다.

AWS Route53에서는 도메인 이름 등록도 지원하기 때문에 방금 발급받은 도메인 이름을 DNS 설정해볼 것이다.

![](https://i.imgur.com/vMF8Y8t.png)

![](https://i.imgur.com/1cxmFaz.png)

한번도 호스팅 영역을 생성하지 않았다면 대시보드에 `호스팅 영역 생성 버튼`이 있을 것이다. 혹은 좌측 메뉴에서 호스팅 영역으로 진입해서 생성해도 된다.

## 호스팅 영역 생성하기

![](https://i.imgur.com/3WdBRBy.png)

발급 받은 도메인 이름을 입력하고 생성한다.

![](https://i.imgur.com/Hd2tluR.png)

NS에 해당하는 라우팅 대상을 복사해고 레코드 생성을 클릭하자.

## 하위 도메인 등록하기

![](https://i.imgur.com/D2WJ5PG.png)

레코드 이름에 하위 도메인을 등록한다.  
아무것도 넣지 않아도 되고, www 추가해도 된다. 값 항목에 EC2의 탄력적 IP를 넣어준다.

> EC2 인스턴스에서 탄력적 IP 주소를 확인할 수 있다.

## DNS 등록하기

위에서 확인한 가비아 네임 서버 관리 페이지에 라우팅 대상을 네임서버 주소로 등록한다.

![](https://i.imgur.com/enycwnU.png)

# ACM_인증서_발급받기


### ACM에서 SSL 인증서 발급

먼저, AWS에서 Certificate Manager 서비스 접근 후 인증서 요청 버튼을 누른다.

![](https://i.imgur.com/HBMqFP6.png)


발급받았던 도메인을 입력하고 DNS 검증을 선택하고 태그는 선택적으로 추가한다. 그리고 입력한 내용에 틀린 부분은 없는지 확인하고 확인 및 요청을 눌러 생성하자

그러면 아래와 같이 검증을 시작하고 승인을 받으려면 추가적인 작업이 필요하다. 아래 사진처럼 Route 53에서 레코드 생성을 누르자

![](https://i.imgur.com/N8n3EGj.png)

그 뒤 Route 53에 자신의 도메인을 클릭하면 아래와 같이 레코드 세트가 추가된 것을 확인할 수 있고 최대 30분 정도 승인요청이 소요될 수 있다.

### EC2 로드 밸런서 설정

다음은 EC2 로드 밸런서 설정을 해야합니다. EC2 서비스에 들어가서 좌측 메뉴에서 로드 밸런서를 들어가주세요. 그리고 로드 밸런서 생성을 클릭합니다.

왼쪽 상단의 로드밸런서 생성을 선택합니다.

![](https://i.imgur.com/KvbPCv4.png)
`Application Load Balancer` 으로 생성해도 되지만 영문과 복잡한 구조로 되어있기 때문에, `Classic Load Balance`으로 생성한다.

Classic으로 생성한 후 마법사를 돌리면 `Application Load Balancer`과 같은 결과를 가져다 준다.

![](https://i.imgur.com/2dsdZtt.png)


Load Balancer 이름 작성 후 HTTPS(보안 HTTP)를 선택해 아래 사진과 같도록 하자


![](https://i.imgur.com/TfoOjyf.png)

그다음 EC2 인스턴스의 보안그룹을 선택해 주시고 다음을 누른다.

![](https://i.imgur.com/9KgqIrF.png)
그리고 ACM에서 인증서 선택창을 누르고 이전에 발급받았던 인증서를 선택

![](https://i.imgur.com/8YVyvXF.png)

그리고 상태 검사를 구성으로 넘어오게 되는데, 변경할 것은 없다.

![](https://i.imgur.com/LqfI8KQ.png)


![](https://i.imgur.com/JwxcDkK.png)


마지막으로 태그를 추가할 것이 있다면 추가하고 검토 및 생성한다.

클래식 로드밸런서로 생성했는데, 차세대 로드밸런서로 선택하기 위해 사진에서 보이는 것처럼 _지금 마이그레이션_을 선택하면 차세대 로드밸런서로 사용할 수 있다.(마이그레이션 후 기존의 클래식 로드밸런서는 삭제!)

![](https://i.imgur.com/U2q48zM.png)

## Route 53 설정

다시 Route 53 서비스의 등록한 도메인 호스팅 영역에서 **유형 A의 레코드 세트**를 선택하고, 별칭을 예로 선택하신 뒤, 별칭 대상에서 만든 로드 밸런서를 선택 후 저장한다.

![](https://i.imgur.com/g6nPgs8.png)

# Nginx 설정

여기까지 무탈히 왔다면 다음은 Nginx 설정을 통해 Reverse Proxy를 구현하고, 인스턴스 내에 도커로 스프링부트 이미지를 올리면 모든 과정이 완료된다.

먼저 인스턴스에 Nginx를 설치하자

- `sudo apt-get update`  
- `sudo apt install nginx -y`  
- `nginx -v` : Nginx 버전 확인  
- `sudo service nginx status` : Nginx 상태 확인

![](https://i.imgur.com/MTgvaQ4.png)

만약 active상태가 아니라면 `sudo service nginx start`커맨드를 입력해 서버를 실행한다.

## Nginx 설정 파일 생성하기

서버가 올라가는 EC2 인스턴스 `etc/nginx/sites-available/springworld.site`에서 HTTP 요청을 HTTPS 로 받아 처리할 수 있도록 코드만 추가한다. 
사진과 같이 아래의 코드(if문)를 추가

```null
sudo vi /etc/nginx/sites-available/{your domain}`
```

## Nginx 서버 연동

### Proxy 설정

```ubuntu
sudo mkdir /var/log/nginx/proxy/	# log, error 파일이 들어갈 디렉토리 생성
sudo vi /etc/nginx/proxy_params
```

포트번호가 80인 http에서 요청이 오면, Spring Boot 프로젝트에서 8088번 포트를 바라볼 수 있도록 proxy 설정을 해주겠습니다.`proxy_params`에 아래의 코드를 붙여넣어줍니다.

```null
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-NginX-Proxy true;

client_max_body_size 256M;
client_body_buffer_size 1m;

proxy_buffering on;
proxy_buffers 256 16k;
proxy_buffer_size 128k;
proxy_busy_buffers_size 256k;

proxy_temp_file_write_size 256k;
proxy_max_temp_file_size 1024m;

proxy_connect_timeout 300;
proxy_send_timeout 300;
proxy_read_timeout 300;
proxy_intercept_errors on;
```

## 서버 블록 생성

현재 가지고 있는 도메인과 jar 파일을 이용하기 위해서 기본 구성 파일을 수정해 서버 블록을 생성하자  
명령어 `sudo vi /etc/nginx/sites-available/{domain}`를 입력 후, 아래 코드를 복사해 붙여넣어준다.

```javascript
server { 
        listen 80;

    server_name mcpark.info;

    access_log /var/log/nginx/proxy/access.log;
    error_log /var/log/nginx/proxy/error.log;

    location / { # location 블록
        include /etc/nginx/proxy_params;
        proxy_pass http://{yourdomain or public ip addr};# reverse proxy의 기능
    }
}
```

이 코드를 추가하면서 location 블록의 `proxy_pass`를 통해 8088번 포트를 통해 접속해야 볼 수 있는 화면(Spring Boot 프로젝트 화면)을 80번(HTTP) 포트에 접속했을 때 확인할 수 있도록 설정, 
즉 `Reverse proxy`의 기능을 하게 할 수 있다.

Nginx는 이제 listen 지시문에 의해 포트 번호 80으로 들어오는 요청들에 대해 server_name 값과 정확하게 일치하는 서버 블록을 찾으려고 시도하고 
만약 server_name을 추가할 때 해시 버킷 메모리 문제가 발생할 수 있기에 `/etc/nginx/nginx.conf`파일에서 옵션을 조정한다.

```null
http { ...
	server_names hash_bucke_size 64;	# 주석 처리를 제거
	...
}
```

### 새로 생성한 파일 활성화

서버블록까지 생성하였으니 새로 생성한 파일을 활성화가 필요하다.

`sites-available` 디렉토리로부터 site-enabled 디렉토리에 대한 링크를 생성해 파일을 활성화한다.

```null
$ sudo ln -sf /etc/nginx/sites-available/{domain} /etc/nginx/sites-enabled/
```

**기본 구성 파일 삭제**

sites-available 디렉토리와 sites-enabled 디렉토리에서 명령어 `ls -al`을 실행해보면 원래 nginx 서버를 연결하던 _default_ 파일이 새로 생성한 파일과 함께 보이는데

이대로 연결하면 연결이 되지 않기 때문에 Default 파일을 삭제한다.

```null
sudo rm  /etc/nginx/sites-available/default
sudo rm  /etc/nginx/sites-enabled/default
```

### Nginx 재시작

Nginx에 대해 구문 오류가 없는지 테스트하고 재시작한다.

```null
sudo nginx -t
sudo service nginx reload
```

### Nginxx Http 요청을 HTTPS로 리다이렉트 설정하기

이제 거의 다 되었습니다.`/etc/nginx/sites-available/springworld.site`에서 HTTP 요청을 HTTPS 로 받아 처리할 수 있도록 코드만 추가하자 사진과 같이 아래의 코드(if문)를 추가

```null
sudo vi /etc/nginx/sites-available/springworld.site
```

위의 명령어를 입력하고 if문을 추가합니다.

```null
    if ($http_x_forwarded_proto = 'http'){
    return 301 https://$host$request_uri;
    }
```

server {
        listen 80;

    server_name mcpark.info;

    access_log /var/log/nginx/proxy/access.log;
    error_log /var/log/nginx/proxy/error.log;

    location / { # location 블록
        include /etc/nginx/proxy_params;
        proxy_pass http://3.37.60.128:8080;# reverse proxy의 기능

        if ($http_x_forwarded_proto = 'http'){
                 return 301 https://$host$request_uri;
        }
    }
}

설정값이 변경되었으니 아래의 명령어를 입력

```null
$ sudo ln -sf /etc/nginx/sites-available/{domain} /etc/nginx/sites-enabled/
 
```

nginx 리부팅

```null
$ sudo systemctl daemon-reload && sudo systemctl restart nginx
```


# Github-Actions-Docker-compose

Github Actions를 이용하여 스프링부트를 자동 배포해보자.

스프링 프로젝트를 Docker 컨테이너에 올리기 위해서는`Dokerfile`을 작성해야 한다. Spring Boot로 만들어진 Web Application을 Docker Image로 만들어서 docker hub에 push하는 과정이다.

![](https://i.imgur.com/4zO4tno.png)

스프링부트 프로젝트 루트 경로에 `Dockerfile` 이라는 파일을 생성해서 아래와 같은 내용을 입력하자.

```
FROM docker.io/library/openjdk:11  
ARG JAR_FILE=./build/libs/muscle-0.0.1-SNAPSHOT.jar  
COPY ${JAR_FILE} app.jar  
ENTRYPOINT ["java", "-jar", "/app.jar", "--spring.profiles.active=prod"]

```

`FROM openjdk:11`  
: base 이미지를 openjdk:11로 설정한다. 즉, Docker에 올릴 때 jdk11 버전을 이용해서 올리겠다고 선언해주는 커맨드이다.

`ARG JAR_FILE=./build/libs/{빌드한 jar 파일명}.jar`  
: JAR 파일의 위치를 환경변수의 형태로 선언해주는 커맨드이다. 프로젝트를 빌드하게 되면 build/libs/{프로젝트명}-0.0.1-SNAPSHOT.jar 형태로 파일이 생성된다.

`COPY ${JAR_FILE} app.jar`  
: 프로젝트 빌드 파일을 컨테이너의 루트 디렉토리의 app.jar 라는 이름으로 복사하는 커맨드이다.

`ENTRYPOINT ["java","-jar","/app.jar", "--spring.profiles.active=prod"]`  
: 컨테이너에서 java -jar /app.jar를 실행하는 커맨드이다. 이때 --spring.profiles.active=prod 라는 옵션은 properties 파일에서 profiles를 활성화하는 경우 붙여준다.

## docker-compose 설정하기

## 2-1. docker-compose 설치하기

[지난번](https://velog.io/@jmjmjmz732002/Github-Action-Docker-EC2-Nginx-%ED%99%9C%EC%9A%A9%ED%95%9C-Springboot-CICD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-4-AWS-EC2-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%EC%97%90-Docker-docker-compose-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0) EC2 인스턴스에서 docker-compose를 설치했으니 스킵하도록 하겠다.

## 2-2. docker-compose.yml 작성하기

docker-compose 파일을 작성하여 EC2 서버에 배포할 것이다. 프로젝트 루트 경로에 docker-compose.yml 파일을 생성하자.

> docker-compose.yml을 작성하여 각각 독립된 컨테이너의 실행 정의를 실시한다.

```yml
version: "3" # 버전 지정

services: # 컨테이너 설정
  database: 
    container_name: mysql # 컨테이너 이름
    image: mysql/mysql-server:latest # 컨테이너에서 사용하는 base image 지정
    environment: # 컨테이너 안의 환경변수 설정
      MYSQL_DATABASE: {database name}
      MYSQL_USER: {database user}
      MYSQL_PASSWORD: {database pwd}
      MYSQL_ROOT_HOST: '%'
      MYSQL_ROOT_PASSWORD: rootpwd
    command: # 명령어 설정
      - --default-authentication-plugin=mysql_native_password
    ports: # 접근 포트 설정 
      - 3305:3306 # Host:Container
    networks:
      - db_network
    restart: always  # 컨테이너 실행 시 재시작
  mytamla:
    build: .
    expose:
      - 8080
    depends_on:
      - database

networks: # 커스텀 네트워크 추가
  db_network: # 네트워크 이름
    driver: bridge
```

방금 작성한 `Dockerfile`로 스프링부트 컨테이너를 실행하고, mysql은 기존 이미지를 사용해 컨테이너 실행을 할 것이다.

mysql config를 따로 빼서 볼륨 설정을 해주기도 하는데 필자는 생략했다.

> 기본적으로 Docker Compose는 하나의 디폴트 네트워크에 모든 컨테이너를 연결한다. 디폴트 네트워크의 이름은 docker-compose.yml가 위치한 디렉토리 이름 뒤에 _default가 붙는다.

커스텀 네트워크  
컨테이너에 올리는 mysql은 호스트 컴퓨터에서 접속할 때는 3305포트를 사용해야 하고, 같은 디폴트 네트워크 내의 다른 컨테이너에서 접속할 때는 3306 포트를 사용해야 한다.

이렇게 해주면 database 서비스는 디폴트 네트워크 뿐만 아니라 db_network 네트워크에도 연결되게 된다.

여러가지 네트워크 드라이버가 있는데

> 1. bridge : 하나의 호스트 컴퓨터에서 여러 개의 컨테이너들이 통신할 수 있게 한다.
> 2. host : 호스트 컴퓨터와 동일한 네트워크에서 여러 개의 컨테이너들이 통신할 수 있게 한다.
> 3. overlay : 여러 호스트 컴퓨터(다른 네트워크)에서 여러 개의 컨테이너들이 통신할 수 있게 한다.


## Github Action CI/CD 작성하기

##  CI/CD

> **CI(Continuous Integration)**는 지속적 통합을 나타내는 용어이다.
> 
> 어플리케이션의 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되어 공유 레포지토리에 통합되는 것을 의미한다.
> 
> 클래스와 기능에서부터 전체 애플리케이션을 구성하는 서로 다른 모듈에 이르기까지 모든 것에 대한 테스트를 수행할 수 있으며, 코드를 병합하는 과정에서 충돌이 생긴다면 CI를 통해 버그를 수정할 수 있다.
> 
> 이로 인해 개발하는 코드의 품질을 좀 더 향상시킬 수 있으며, 새로운 업데이트의 검증 및 릴리즈의 시간을 단축시킬 수 있다.
> 
> **CD(Continuous Deliver, Countinuous Deplotment)**는 지속적인 배포를 나타내는 용어이다.  
> (* 보통은 전자인 지속적 제공의 의미가 강하다.)

![](https://i.imgur.com/hRwyLdS.png)

## 왜 CI/CD를 구축하는걸까?

어플리케이션 개발 단계를 자동화하면 보다 짧은 주기로 고객에게 제공할 수 있다. 자동화가 이루어지지 않는다면 모든 빌드와 배포 작업을 수동으로 조작해야 하는데 사소한 수정 사항 하나에도 모든 과정을 거쳐야하므로 매우 불편하고 시간 소요가 많다.

CI/CD 구축을 지원하는 툴은 시중에 많다.

- Jenkins
- Travis CI
- Circle CI
- Google Cloud Build
- AWS CodeBuild
- Github Actions

각 툴은 요금체계부터 특성이 모두 다르므로 프로젝트에 적합한 것을 선택하면 된다.

### ✅ Github Actions

Github Actions는 소프트웨어 개발 라이프사이클 안에서 PR, push 등의 이벤트 발생에 따라 자동화된 작업을 진행할 수 있게 해주는 기능이다.  
CI/CD나 Testing, Cron Job 등 작업을 수행할 수 있다.

### ✅ Github Actions의 구성 요소

Github Actions를 활용하기 위해 구성 요소를 먼저 파악해보자.

#### Workflow

- Workflow란 레포지토리에 추가할 수 있는 일련의 자동화된 커맨드 집합이다.
- 하나 이상의 Job으로 구성되어 있고, PR이나 push과 같은 이벤트에 의해 실행될 수도 있고 특정 시간대에 실행될 수 있다.
- 빌드, 테스트, 배포 등 각각의 역할에 맞는 Workflow를 추가할 수 있고, `.github/workflows` 디렉토리에 YAML 형태를 저장합니다.

#### Event

- Event란 Workflow를 실행시키는 push, PR, commit 등의 특정 행동을 의미한다.

#### Job

- Job이란 동일한 Runner에서 실행되는 여러 Step의 집합을 의미한다.
- 기본적으로 하나의 Workflow 내의 여러 Job은 독립적으로 실행되지만, 필요에 따라 의존 관계를 설정하여 순서를 지정해줄 수 있다.

#### Step

- Step이란 커맨드를 실행할 수 있는 각각의 Task를 의미한다. Shell 커맨드가 될 수도 있고, 하나의 Action이 될 수도 있다.
- 하나의 Job 내에서 각각의 Step은 다양한 Task로 인해 생성된 데이터를 공유할 수 있다.

#### Action

- Action이란 Job을 만들기 위해 Step을 결합한 독립적인 커맨드로, 재사용이 가능한 Workflow의 가장 작은 단위의 블럭이다.
- 직접 만든 Action을 사용하거나 Gihub Community에 의해 생성된 Action을 불러와 사용할 수 있다.

#### Runner

- Runner란 Github Actions Workflow 내에 있는 Job을 실행시키기 위한 어플리케이션이다.
- Runner Application은 Github에서 호스팅하는 가상 환경 또는 직접 호스팅하는 가상 환경에서 실행 가능하며, Github에서 호스팅하는 가상 인스턴스의 경우에는 메모리 및 용량 제한이 존재한다.

## CI Workflow 생성하기

필자는 팀원들과 Git을 공유하며 사용하는 환경이므로, 기능 추가와 같이 코드가 수정되고 난 후 `feat` -> `main` PR에서 CI가 동작하도록 구성하려고 한다.  
(* 혼자 백엔드를 맡아서 feat, main으로만 브랜치를 구성했었다.)

CI 작업을 수행할 프로젝트의 레포지토리의 Actions 탭을 접속한다.

> 프로젝트 루트 `.github/workflows` 디렉토리 내에 `.yml` 파일을생성해도 된다.

![](https://i.imgur.com/QSgX5q7.png)

다양한 용도별 템플릿을 제공해주고 있다. 지금은 가장 기본적인 템플릿을 사용해볼 것이다. 상단에 `set up a workflow yourself` 를 클릭한다.

![](https://i.imgur.com/Bry4lCd.png)


```yaml
# Workflow 이름은 구별이 가능할 정도로 자유롭게 적어주어도 된다. 
# 필수 옵션은 아니다.
name: Java CI with Gradle

# main 브랜치에 PR 이벤트가 발생하면 Workflow가 실행된다.
# 브랜치 구분이 없으면 on: [pull_request]로 해주어도 된다.
on:
  pull_request:
    branches: [ "main" ]

# 테스트 결과 작성을 위해 쓰기권한 추가
permissions: write-all


# 해당 Workflow의 Job 목록
jobs:
	# Job 이름으로, build 라는 이름으로 Job이 표시된다.
  build:
  	# Runner가 실행되는 환경을 정의
    runs-on: ubuntu-latest

	# build Job 내의 step 목록
    steps:
      # uses 키워드를 통해 Action을 불러올 수 있다.
      # 해당 레포지토리로 check-out하여 레포지토리에 접근할 수 있는 Acion 불러오기
    - uses: actions/checkout@v3
    # 여기서 실행되는 커맨드에 대한 설명으로, Workflow에 표시된다. 
    # jdk 세팅
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        
      # gradle 캐싱
    - name: Gradle Caching
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-
      
    ### CI
    #gradlew 권한 추가
    - name: Grant Execute Permission For Gradlew
      run: chmod +x gradlew
    
    #test를 제외한 프로젝트 빌드
    - name: Build With Gradle
      run: ./gradlew build -x test

    #test를 위한 mysql설정
    - name: Start MySQL
      uses: samin/mysql-action@v1.3
      with:
        host port: 3305
        container port: 3305
        mysql database: '{database name}'
        mysql user: '{database user}'
        mysql password: '{database pwd}'

    #테스트를 위한 test properties 설정
    - name: Make application-test.properties
      run: |
        cd ./src/test/resources
        touch ./application.properties
        echo "${{ secrets.PROPERTIES_TEST }}" > ./application.properties
      shell: bash

    #test코드 빌드
    - name: Build With Test
      run: ./gradlew test

    #테스트 결과 파일 생성
    - name: Publish Unit Test Results
      uses: EnricoMi/publish-unit-test-result-action@v1
      if: ${{ always() }}
      with:
        files: build/test-results/**/*.xml
```

## CD workflow 작성하기

CI/CD를 분리해 작성 한다.

```yaml
name: CD

on:
  push:
    branches:
      - main

permissions: write-all

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      #jdk 세팅
      - uses: actions/checkout@v3
      - name: Set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'

      - name: Gradle Caching
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('/.gradle', '/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      # CD
      - name: Make application-prod.yml
        run: |
          cd ./src/main/resources
          touch ./application-prod.yml
          echo "${{ secrets.YML_PROD }}" > ./application-prod.yml
        shell: bash

      - name: Create 'generated-snippets' Directory
        run: mkdir -p build/generated-snippets

      - name: Build With Gradle
        run: ./gradlew build -x test

      - name: Set up QEMU
        id: qemu
        uses: docker/setup-qemu-action@v1

      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1

      #도커 빌드 & 이미지 push
      - name: Docker build & Push
        run: |
          docker login -u ${{ secrets.DOCKER_ID }} -p ${{ secrets.DOCKER_PASSWORD }}
          docker buildx create --use
          docker buildx build --platform linux/amd64,linux/arm64 -t ${{ secrets.DOCKER_REPO }} --push .

      #docker-compose 파일을 ec2 서버에 배포
      - name: Deploy to Prod
        uses: appleboy/ssh-action@master
        id: deploy-prod
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          envs: GITHUB_SHA
          script: |
            docker stop mcpark
            docker rm mcpark
            sudo docker pull ${{ secrets.DOCKER_REPO }}
            docker run -d --name mcpark -p 8080:8080 ${{ secrets.DOCKER_REPO }}
            docker rmi -f $(docker images -f "dangling=true" -q)
```

CD 과정을 통해 ARM 아키텍처에 맞게 이미지를 빌드한다.

## Secrets 등록하기

workflow 내의 `${{ ~~ }}`는 외부에 공개되어서는 안되는 민감 정보를 시크릿으로 저장해 놓은 것이다.

레포지토리 Settings > Secrets > Actions 에서 등록해주면 된다.

![](https://i.imgur.com/K6zIqvI.png)

- DOCKER_ID : Docker-hub 이메일
- DOCKER_PASSWORD : Docker-hub 비밀번호
- DOCKER-REPO : Docke-hub Repository 이름
- EC2_HOST : EC2 public IP address
- EC2_PRIVATE_KEY : EC2 key pair pem 키 값을 복사한 내용 (*---BEGIN 어쩌구부터 끝까지 복붙해넣어야 한다)
- EC2_USERNAME : EC2 인스턴스가 ubuntu라면 ubuntu, linux라면 ec2-user
- PROPERTIES_xxx : properties 파일 내용