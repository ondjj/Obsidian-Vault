**Paging**

카운터링

- 흔히 사용 되는 기능이고, 대다수의 웹 사이트에서 사용된다.

- DB를 통해 전체 게시글을 가져오는 방식 -> 1 - 10 Page 10개의 게시글 단위로 끊어 가져오는 방식

- 이를 위해 선언해야하는 변수가 몇 가지 존재하고, Model 1 방식에선 표현식(<%= %>) Model 2 방식에선 서블릿을 사용 한다.

👻 **초기 설정 및 변수 선언**

  ![](https://i.imgur.com/xqoZXHu.png)

![](https://i.imgur.com/LSey1yl.png)
![](https://i.imgur.com/MpuvPqb.png)

  ![](https://i.imgur.com/o9Ma9Wr.png)

🎃 **전체 글 개 수를 리턴 하는 메소드**
![](https://i.imgur.com/NxdFjw3.png)

😾**이해가 안되면 외워야 하는 부분**
![](https://i.imgur.com/x6guutI.png)

🙈 **startRow와 endRow를 매개 변수로 게시글을 가져오는 메소드 작성**
![](https://i.imgur.com/EgHysVo.png)
- Rownum은 오라클에 존재하는 개념으로 예를 들어 1 부터 상위 5까지 구할 때 사용 된다. Rnum이 자동으로 만들어짐.
- 내부 쿼리를 사용해서 구현한다.

- 이 후 view 에 감소식을 사용해 number를 뿌려준다. ex: <%= number — %>

👀 **페이지 카운터링 소스 작성**

  ![](https://i.imgur.com/5rUIk7H.png)
————————————————————————————————————————————————————————————

#JSP-게시판