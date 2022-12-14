# Nov. 2. 2022 commit & merge

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Nov. 24. 2022

## 개발 내용
> ### 프록시 설정
>
> 참고자료   
> 
> [CORS 이슈 발생, 대응(Proxy설정)](https://github.com/kimhaechang1/template/blob/main/CORS%20%EC%9D%B4%EC%8A%88%20%EB%B0%9C%EC%83%9D%2C%20%EB%8C%80%EC%9D%91(Proxy%20%EC%84%A4%EC%A0%95))
>
> [CORS 와 DevServer Proxy](https://react.vlpt.us/redux-middleware/09-cors-and-proxy.html)
>
> [Proxing API requests in Development](https://create-react-app.dev/docs/proxying-api-requests-in-development/)
>
> 백엔드 프레임 워크로 스프링을 선택하였고, 개발중인 스프링 서버와 통신하기 위해서는 서로다른 도메인에 대한 데이터 전송이 가능해야 하는데
>
> 이 때 CORS 정책 이슈가 발생하기 때문에 Proxy를 설정 하여 해결하는 방식을 채택한다.
>
> 과거 나의 깃허브에서 한번 다뤘던 토픽이라서 똑같이 ```http-proxy-middleware```모듈을 설치하고 ```setupProxy.js```에 설정해주려고 했으나
>
> ```package.json``` 에서 ```react-scripts```버전이 @0.2.3 이상이면 ```"proxy" : ``` 를 통해 프록시 설정이 가능하다.

<br>

> ### 백엔드 개발자와 함께 사용 할 API 문서 작성
> 
> 스프링 백엔드서버를 개발하고 있는 친구가 api 를 문서화 시키자는 말에 찬성하였다.
>
> 사용 예시로는 다음과 같고, ```JSON요청``` 이라는 문서로 관리한다.
> ```
> 1. 
> 요청(url)
> 통신방식(GET or POST)
> 파라미터(ex) id : id)
> 응답
> { success : true }
> ```

<br>

> ### 데이터 베이스 테이블 설정
> 
> 회의를 통해 데이터베이스는 mysql을 사용하기로 하였고
> 
> 우리 홈페이지에 필요한 테이블을 고려하여 백엔드 담당하는 친구가 초기 sql 파일을 만들어 주었다.
> 
> 테이블은 총 4개로서 
> 
> 댓글을 저장 할 comment 테이블, 회원을 관리할 member 테이블, QnA 관련 글을 저장 할 post 테이블 
> 
> 마지막으로 태그검색을 위한 tag 테이블이 있다.
>
> **comment table**
> 
> 댓글기능은 QnA 글에다가 댓글을 다는것으로 일반적인 댓글 시스템과 동일하다.
>
> 테이블의 어트리뷰트 중에서는 익명댓글인지 아닌지 검사하는 부분이 있는데
>
> 이는 익명 댓글을 '에브리타임'의 익명댓글 시스템을 참고 할 예정이다.
>
> **tag table**
> 
> 태그 테이블이 좀 중요한데, 이는 다른 글들에 달린 태그들을 수집하여 중복된 태그 없이 저장 해 놓는 테이블이다.

<br>

> ### 아이디 중복검사 구현
> 
> 저번에 회원가입과 로그아웃 구현을 완료하고서 
> 
> 이번 개발을 시작할때 문득 든 생각은 "어, 아이디 중복검사 해야하는데" 였다.
> 
> 따라서 아이디 중복검사 버튼을 따로 만들고, 백엔드 통신을 통해 중복검사를 완료하도록 개발한다.
>
> 아이디 중복검사 순서
>
> 1. 사용자가 입력한 아이디를 ```axiosIdDuplicateCheck()```로 보낸다.
> 2. ```axiosIdDuplicateCheck```에서는 아이디값을 받아서 서버에 ```POST```방식으로 전송한다.
> 3. 서버에서는 DB에 member 테이블에서 중복된 아이디가 있는지 검사한다.
> 4. 서버에서 응답데이터로 중복인지 아닌지 값과 중복체크 관련 메시지를 보낸다.
> 5. 프론트에서 응답 데이터를 분석하고 중복이 아니라면 '중복체크'버튼을 비활성화 시킨다.
>
> **axiosIdDuplicateCheck.js**
> 
> 인자로 전달받은 id 값을 axios.post 를 통해 백엔드 서버로 전송한다.
> 
> 그리고 응답받은 프로미스 객체를 그대로 리턴한다.
> 
> **SignUp.js**
> 
> input type="button" 을 통해 중복검사 버튼을 만들어주고
> 
> ```onClick``` 이벤트를 연결하여 ```onClickHandler()``` 이벤트 핸들러 함수를 정의한다.
> 
> > **onClickHandler()**
> > 
> > ```useState``` 로 보관되어 있는 id값을 ```axiosIdDuplicateCheck()```의 인자로 넣는다.
> > 
> > 리턴 받아온 프로미스 객체를 ```.then```을 통해 분석하여 중복이 아니라면 '중복체크' 버튼을 비활성화 시킨다.

<br>

> ### 검색기능 70% 구현
> 
> 70% 구현이라고 한 이유는 아직 태그검색과 일반 검색을 어떻게 할지 완벽한 결정이 안났기 때문이다.
>
> 결정 난 부분은 일반검색은 드롭다운 미리보기가 안나오고, 태그 검색을 시도 할 시 드롭다운 미리보기를 제공한다는 점이다.
>
> **SearchDropDown.js**
> 
> 태그 검색일 경우 제공할 미리보기 UI를 만든다.
>
> ```useEffect``` 를 사용하여 사용자의 입력값이 props를 통해 넘어왔을 때
> 
> 검색어가 2글자 이상일 때 부터 미리보기를 통해 몇가지 해쉬태그를 나타내도록 만든다.
> 
> 이 때 태그들을 백엔드 서버에서 들고올 예정이지만 우선에 테스트용으로 만든 hashtag.json을 사용한다.
>
> **Header.js에 검색기능 추가하기**
>
> 먼저 input type="text"와 '검색' 이름의 버튼을 추가한다.
>
> 그리고 해당 input 태그의 value 값인 검색어는 ```useState```로 관리한다.
>
> 그리고 돋보기 버튼을 누르면 검색창 UI가 on & off 되도록 ```useState```로 관리한다.
> 
> <img width="100%" src="https://user-images.githubusercontent.com/81299056/203777636-2d142f9f-30ee-4961-9ffe-e72a8135a3e4.gif"/>
> 
> 또한 해쉬태그 검색에 대해서 드롭다운 미리보기 제공을 위해 ```<SearchDropDown/>``` 컴포넌트를 추가하고 
> 
> props로 ```useState```로 관리중인 검색어 값을 전달한다.
>
> '검색' 버튼에 대한 ```onClick```이벤트를 달아주고 ```onClickHandler``` 이벤트 헨들러 함수를 호출하도록 한다.
> 
> > **onClickHandler**
> > 
> > 일단은 검색기능에 대한 회의가 완결 나면 완성 시킬 ```axiosSearch.js```함수를 호출하여 인자로 검색어를 넣는다.

> ### 중복버튼 CSS 적용
> 
> 우리 팀프로젝트 프론트 팀원이 중복버튼에 CSS적용을 시켜주었다.

후기

처음에는 드롭다운 미리보기를 만들 때, jQuery에 있는 autocomplete를 사용하려고 했으나

react에서는 DOM 객체에 대한 직접적인 접근을 자제해야 하고, 또 어떻게 적용시켜야 할 지 감이 안와서 직접 만들게 되었다.

만들기전에는 저기다가 어떻게 ```<div>```를 붙여서 내용을 넣지 싶어서 고민을 하다가 다른 검색 박스가 있는 홈페이지를 많이 참고했다.

그리고 글만 주루룩 쓰는거보다 이번 처럼 움짤을 남겨서 기록하는게 더 보기 좋을것 같다.
