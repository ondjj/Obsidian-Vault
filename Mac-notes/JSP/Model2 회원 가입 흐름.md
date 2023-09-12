**Model2 회원 가입 흐름**

주요 포인트
- MemberBean - DB 설계에 맞게 private 타입으로 필드 생성 - Getter / Setter 생성
- MemberDAO - 회원 가입 / 탈퇴 / 글 생성 등 목적에 맞게 모델을 생성 하고 DB와 연결
- MemberJoin.jsp - 클라이언트에게 보여지는 View 더 이상 <% %> .. 등을 사용 하지 않고 taglib를 사용한다.
- MemberList.jsp - 클라이언트에게 보여지는 View 객체 자체를 가져와 사용 하는 습관을 들여야한다. ${ bean. * }

📌 **action**
![](https://i.imgur.com/7YSgCy9.png)

- 기존 모델 1 방식의 form 처리는 jsp 파일 간의 이동으로 이루어졌지만, 모델 2 방식은 서블릿의 도입으로 jsp와 servlet의 소통으로 이루어진다.

📌**servlet - proc.do**
![](https://i.imgur.com/ofwgkte.png)

- Get / Post 방식을 따로 처리하지않고 reqPro 메서드를 선언하여 일괄적으로 처리 할 수 있게 구조를 잡는다.

- 기존 jsp에서 처리하던 로직을 모두 서블릿으로 끌어와 control 한다는 점이 모델을 구분하는 가장 큰 차이점

- 주의 할 점으로, 페이지 전환 처리 방법이 있는데 모델 1에서 jsp - jsp로 이동했다면 모델 2 방식은 서블릿 - 서블릿으로 이동한다는점이 있다.

📌 **더이상 로직을 처리하지않는다면?**
![](https://i.imgur.com/8QLcNsb.png)

- 모든 로직을 처리했고, 더는 서블릿 코드를 쓸 필요 없는 즉, 결과 화면 / 메인 화면 / 게시판 .. 등 클라이언트에게 view를 제공하기만 한다면 그 때 jsp로 넘어간다.

#Model2