# Nov. 10. 2022 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Nov. 29. 2022

## 개발 내용
> ### 검색결과 페이지인 Result.js -> Board.js로 변경
> 
> 검색결과 페이지라는 목적보다는 여러 게시판으로 활용할 목적을 생각하여 Board.js로 변경하였다.
 
<br>

> ### Login -> SignIn 으로 명칭변경
>
> 초창기에 로그인은 Login으로 해놓고 회원가입은 SignUp으로 하고 뒤죽박죽이었던 것을
>
> 회의를 통해 통합시키기로 하였다.

<br>

> ### 회원가입 통신 테스트
> 
> 회원가입 부분을 백엔드 담당이 완성 하였다고 하여 함께 테스트를 하였다.
> 
> "이게 한번만에 되네" 하는 생각이 들 정도로 잘 작동하였다.

<br>

>
> ### 자잘한 버그 고치기
>
> 저번에 메인페이지에서 ```useState```로 저장되어 있던 태그리스트들을 
>
> ```<SearchBox>``` 컴포넌트로 props를 통해 넘겼다고 생각했는데
>
> 지금보니 안넘겨 져 있었다.
>

후기

항상 props를 사용하여 부모 컴포넌트와 자식컴포넌트가 소통할 때 어떤게 필요한지 잘 살펴 볼 필요가 있다.
  