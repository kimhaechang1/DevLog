# Oct. 24. 2022 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Nov. 21. 2022

## 개발 내용
> ### be전공자 프론트 프로젝트 생성
> ```npx create-react-app```을 통해 ```react```프로젝트 생성

<br>

> ### be전공자 라우팅 연결
> #### 참고자료
> + [template프로젝트](https://github.com/kimhaechang1/template/blob/main/React%20Router%20DOM)
> + [@velopert/react-router-dom v6](https://velog.io/@velopert/react-router-v6-tutorial)
> #### be전공자 페이지 요소 4가지 작성
> + ```MainPage.js``` : 메인 페이지
> + ```GongJi.js``` : 공지사항 페이지
> + ```QnABoard.js``` : QnA 페이지
> + ```Dic.js``` : 용어사전 페이지
> #### be전공자의 페이지들을 ```react-router-dom```을 사용하여 연결
> ```App.js```에서 라우팅


<br>

> ### be전공자 홈페이지의 틀 그리기
> #### 메인페이지에 들어갈 요소들을 채워넣기 위해 상단바 및 내용 부분의 프레임을 구성
> #### 참고자료
> + [이번에야 말로 CSS Flex를 익혀보자](https://studiomeal.com/archives/197)
> 
> ##### Header.js 
> 
> 홈페이지의 상단바를 이루는 컴포넌트
> 
> 로그인 로그아웃 기능 및 여러 페이지로 이동을 위한 초기 UI 생성
> 
> ```react-router-dom```의 ```Link```컴포넌트를 사용
> 
> ```Link```컴포넌트의 ```to```속성값을 통해 알맞은 페이지로 연결
> 
> ##### Card.js
> 메인페이지의 컨텐츠 영역을 다루는 컴포넌트
> 
> 메인 페이지의 **공지사항**, **최신 QnA**, **최근 등록된 용어**, **조회수 높은 순** 에 해당하는 요소의 위치 및 프레임을 형성
> 
> ```name``` 속성으로 각각의 글 제목들을 넘겨받아서 ```useState```로 상태저장
> 


