---
sticker: lucide//align-horizontal-space-between
tistoryBlogName: ondj
tistoryTitle: Server Sent Event(SSE)
tistoryVisibility: "0"
tistoryCategory: "1096106"
tistorySkipModal: true
tistoryPostId: "162"
tistoryPostUrl: https://ondj.tistory.com/162
---
### Short Polling

- 클라이언트에서 주기적으로 서버로 요청을 보내는 방법입니다.
- 서버에 대한 부담이 크지 않고 요청 주기를 넉넉하게 잡아도 될 정도로 실시간성이 중요하지 않다면 고려해 볼 만한 방법입니다.

### Long Polling

![](https://velog.velcdn.com/images/xogml951/post/337d903c-82d4-4720-b85f-44933b2c844e/image.png)

- 요청을 보내고 서버에서 변경이 일어날 때까지 대기하는 방법입니다.
- Short Polling에 비해서는 서버의 부담이 줄어들지만 한번의 서버 변경마다 다시 연결을 재요청해야 합니다.

### Server-Sent Events

![](https://velog.velcdn.com/images/xogml951/post/7c840333-6672-4cc2-954d-4b937cdb7f31/image.png)

- 서버와 한번 연결을 맺고나면 일정 시간동안 서버에서 변경이 발생할 때마다 데이터를 전송받는 방법입니다.
- Long Polling과 달리 서버에서 변경이 발생해도 다시 재연결 할 필요가 없습니다.
- 최대 연결 수에 제한이 있습니다.

### WebSocket

![](https://velog.velcdn.com/images/xogml951/post/4a3e14f3-8ff7-4924-9f96-eb9253b19618/image.png)

- 서버와 연결을 유지하면서 양방향 통신을 유지할 수 있습니다.
- SSE에 비해서 구현과 학습에 비용이 클 수 있습니다.


[Spring SSE Docs](https://www.baeldung.com/spring-server-sent-events)