**ConectionPool**
DBCP

- JDBC를 사용하여 DB에 접속하는건 생각보다 자원을 많이 소비 하는 일이다. 사용자가 많은 환경에서 요청에 따라 매 번 드라이버를 로드하고 커넥션 객체를 생성하여 연결하고 닫는것은 데이터베이스에 과부하를 줄 수 있는데 이를 해결 하기 위해 커넥션 풀을 적용한다.

- Conection Pool이란  웹 컨테이너가 실행되면서 커넥션 객체를 미리 pool에 생성해 두고 필요에 따라 가져다 쓰고 반환 하는 개념으로, 미리 생성해두기 때문에 데이터베이스에 부하를 줄이고 유동적으로 연결을 관리 할 수 있다.

📌Servers/server.xml
![](https://i.imgur.com/kfegZjK.png)


- 만약 JSP1/JSP2 프로젝트도 커넥션 풀을 적용 하고 싶다면 Database 패턴처럼 바꿔준다.

📌MemberDAO.class
![](https://i.imgur.com/UEoWTNQ.png)

- 주석 oracle에 접속하는 소스 부분은 server.xml으로 넘어가있기 때문에 더 이상 필요하지 않고, 기존에 있던 try 구문은 db Driver와 커넥션 부분이 존재하는 반면 새롭게 작성 한 구문은 환경에 접근 하는 패턴으로 바뀐것을 확인 할 수 있다.

#DataBase