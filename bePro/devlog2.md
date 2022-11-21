# Oct. 25. 2022 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Nov. 21. 2022

## 개발 내용
> ### 페이지 및 라우팅 수정
> #### 다양한 전공자와 비전공자가 모여서 QnA를 이루는 것이 홈페이지 주요 목적에 맞게 변경
>
> 검색기능을 나중에 추가 할 것이므로
> 
> 용어사전 페이지 ```Dic.js```를 검색결과 페이지 용 ```Result.js```로 변경    
> 
> 검색결과 페이지 기능 :
> 
> + 검색된 글 목록들을 글 제목을 기준으로 형성
> 
> + 해당 글 제목 누를 시 글에 대한 상세 내용 페이지로 넘어감
>
> 공지사항 페이지인 ```GongJi.js```는 추 후 검토를 위해 라우팅 해제    
> 
> ```QnABoard.js```페이지는 검색 결과 페이지인 ```Result.js```와 기능이 비슷하므로  
>  
> 마이페이지용 ```MyPage.js```로 변경 및 라우팅 변경
> 
> 마이페이지 기능 :   
> 
> + 개인정보 수정
> + 본인이 쓴 글 확인   
> 
> 추 후 기능이 추가 될 수도 있음   
> 

<br>

> ### 로그인 및 회원가입 컴포넌트 추가
> 
> #### 참고자료
> 
> + [React modal 로그인창 만들기](https://velog.io/@7p3m1k/React-modal-%EB%A1%9C%EA%B7%B8%EC%9D%B8%EC%B0%BD-%EB%A7%8C%EB%93%A4%EA%B8%B0)
> 
> #### SignUp.js
> 
> 회원가입 모달 창으로서 구성요소는 다음과 같다
> 
> + 아이디 : input type="text"
> + 비밀번호 : input type="password"
> + 이메일 : input type="email"
> + 닉네임 : input type="text"
> + 전공자 or 비전공자 : input type="radio"
> + 전공자라면 전공입력 input type="text"
> 
> 전공자 or 비전공자 에 따라 전공입력 input이 생겨야 하기 때문에 ```useState```로 관리
> 
> #### SignIn.js
> 
> 로그인 모달 창으로서 구성요소는 다음과 같다
> 
> + 아이디 : input type="text"
> 
> + 비밀번호 : input type="password"
> 
> #### 상단바 요소 변경 및 모달창 on off 기능
> 
> 상단바의 기존에 있던 페이지들이 변경 및 삭제 되었으므로 수정
> 
> 로그인 로그아웃을 각각 Sign in, Sign up으로 이름을 붙이고 상단바의 요소로 추가
> 
> 로그인 로그아웃은 모달 창 이므로 on off 상태를 ```useState```로 관리
> 
> - 상태값을 변경하는 함수를 각각의 컴포넌트 속성(props)으로 전달
> 
