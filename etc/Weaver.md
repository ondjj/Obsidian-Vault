---
sticker: emoji//1f918
---
# 24.04.16Ver

## 변경 사항
#### JWT
- Token sub : 기존 (user id) -> 변경 (member uuid)
- LoginType 항목 추가 (Weaver || OAUTH)

![](https://i.imgur.com/uypyl5p.png)


기대 사항
- LoginType에 따라 알맞은 UI 표출(OAuth 유저는 비밀 번호, 이메일 .. 수정이 불가능)

### Auth*Controller
- MemberController로 명칭 변경 End-point (api/v1/auth/* -> api/v1/member/*)
- 관리 대상
  - OAuth 로그인 유저, 일반 로그인 유저 공통 관심사
  - Ex. 토큰 재발급, 로그아웃, Member UUID를 통한 조회 ..

### UserController
- End-point 기존 동일 (api/v1/users/*)
- 관리 대상
  - 일반 로그인 사용자 단독 관심사
  - Ex. 닉네임 중복 체크, 이메일 중복 체크 ..
- 고려 사항
  - Member 정보 업데이트 로직 분할 여부
    - 현재 UserService를 통해 OAuth, 일반 로그인 유저 정보를 한 곳에서 업데이트 중

### 진행 상황

- Member Controller를 통한 기능 [v]
- User Controller를 통한 기능 [v]
- OauthMember Controller 통한 기능 [v]
- Project Controller를 통한 기능 [v]
- Issue Controller [x]
- Task Controller [x]
- Comment Controller[x]

만약 [x] 표시 되어 있는 컨트롤러의 기능이 Member 도메인과 무관하다면 정상 작동할 수 있습니다.