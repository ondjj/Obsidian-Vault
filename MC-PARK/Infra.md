---
tags: mcpark, infra, info
sticker: lucide//globe
---

- [ ] **AWS EC2 t4g (ARM architecture) 인스턴스**

| 이름         | MC-PARK             |
|:-----------|:--------------------|
| ID         | i-02e04581512f5164e |
| 플랫폼        | Ubuntu 20.04 LTS    |
| 유형         | t4g.small 2 vCPU    |
| 아키텍처       |            64비트 ARM |
| 스토리지       |  1x30 GiB gp2 루트 볼륨 |
| 탄력적 IP     | 3.37.60.128 |
|              | IPv4 SSH TCP 22 0.0.0.0/0|
| 인바운드 규칙  | IPv4 HTTP TCP 80 0.0.0.0/0|
|            | IPv4 HTTPS TCP 443 0.0.0.0/0|
|            | IPv4 사용자지정 TCP tcp 8080 0.0.0.0/0 |


- [ ] **RDS 인스턴스**

|  DB 식별자 | rds-mc-park |
|:---|:---|
| 엔진 | MySQL Community  |
| 엔드포인트 | rds-mc-park.cade1afm85pd.ap-northeast-2.rds.amazonaws.com |
| 포트 | 3306  |
| DB name | db_mc_park |
| Admin name | muscle |
| PW | musclepark0831 |
|                       | Docker 172.17.0.2/32 |
| 보안 그룹:<br>rds-mc-park | EC2 3.37.60.128/32 |  
|                       | 호석 IP Addr |
|                        | 창준 IP Addr |
| 인스턴스 연결 | O |


- [ ] S3 버킷 

| 이름 | mc-park-s3-bucket                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|:---|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 권한 | 퍼블릭                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| 정책 | <div>{</div><div>&nbsp; &nbsp; "Version": "2012-10-17",</div><div>&nbsp; &nbsp; "Statement": [</div><div>&nbsp; &nbsp; &nbsp; &nbsp; {</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "Sid": "AddPerm",</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "Effect": "Allow",</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "Principal": "*",</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "Action": [</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "s3:GetObject",</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "s3:PutObject",</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "s3:DeleteObject"</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ],</div><div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "Resource": "arn:aws:s3:::mc-park-s3-bucket/*"</div><div>&nbsp; &nbsp; &nbsp; &nbsp; }</div><div>&nbsp; &nbsp; ]</div><div>}</div> |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |  

- [ ] IAM

| 이름 | mcpark-iam-s3      |
|:---|:-------------------|
| 정책 | AmazonS3FullAccess |  


- [ ] DNS

| 호스팅 이름 | mcpark.info |
|:-------|:------------|


