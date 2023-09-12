---
tags: []
---
  
![](https://i.imgur.com/U9SUTIN.png)

  

  

- JSP Model 1 과 Model 2의 가장 큰 차이점은 Servlet의 사용 유무로 나뉜다.
- 그동안 JSP 파일 내에서 객체를 다루고, 이를 통해 view 혹은 Model과 커넥션을 했다면 이후로는 JAVA 파일 내에서 객체와 객체를 연결하고 이를 뿌리는 형식으로 진행한다.
![](https://i.imgur.com/OpJyGQv.png)

- form 태그의 action은 annotation 기능을 사용 하기 때문에 jsp 파일이 아닌 이벤트의 느낌으로 url을 설정한다.
- 현업에서는 get/post 방식을 따로 구현하기보단, reqPro와 같이 새로운 메소드를 생성해서 처리를 요청 하는 모양으로 로직을 구성한다.

**RequestDispatcher**

- **클라이언트의 요청을 수신하여 서버의 리소스(서블릿/JSP/HTML)로 보내는 객체를 정의한다.**
- **서블릿 컨테이너는** RequestDispatcher 객체를 만든다.
- **이 오브젝트는 특정 경로에 있거나 특정 이름으로 지정된 서버 리소스 주변의 래퍼로도 사용 된다.**
- **서블릿을 래핑하기 위한 것이지만, 서블릿 컨테이너는 모든 유형의 리소스를 래핑하기 위해 RequestDispatcher 객체를 만들 수 있다.**

📌 **한 줄 요약** 

서블릿을 통해 로직 처리를 하고 jsp에서 쓸 수 있게 랩핑 해 보내려면 이 기능을 쓰면 된다. 