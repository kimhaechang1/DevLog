# Dec. 22. 2022 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Dec. 22. 2022

## 개발 내용
>
> ### 프록시 재설정 & API 주소변경 및 수정
>
> #### http-proxy-middleware 사용
>
> 참고자료 :
>
> [npm http-proxy-middleware](https://www.npmjs.com/package/http-proxy-middleware)
>
> [과거에 사용해본 http-proxy-middleware](https://github.com/kimhaechang1/template/blob/main/CORS%20%EC%9D%B4%EC%8A%88%20%EB%B0%9C%EC%83%9D%2C%20%EB%8C%80%EC%9D%91(Proxy%20%EC%84%A4%EC%A0%95))
> 
> 본가에 내려와서 코딩을 딱 시작하려고 했는데
>
> 프록시 에러가 엄청나게 뜨던 것 이었다.
>
> 인터넷에 검색하여 살펴보았더니, 유선 랜을 사용 할 경우 package.json에서
> 
> ```json
> "proxy" : "localhost:8080"
> ```
> 위와 같이 설정한 것이 작동 안할수도 있다는 것 이었다.
>
> 그래서 이거 때문이겠다 싶어서, 전에 사용 한 적이 있는 http-proxy-middleware를 설치하고 사용하였다.
>
> #### API 주소변경
>
> 우선에 http-proxy-middleware를 설치하여 setupProxy를 설정한 코드는 다음과 같다.
>
> ```js
> const { createProxyMiddleware } = require('http-proxy-middleware');
> module.exports = function(app) {
>  app.use(
>    createProxyMiddleware(['/api/**','/qna/**','/notice/**','/user/**','/board/**','/get/**',"/auth/**"],{ 
>      target: 'http://localhost:8080',
>      changeOrigin: true,
>    })
>  );
>};
> ```
>
> 여기서 이제 저번에 수정하였던 url 파라미터에 직접 넣어서 이동하려 하는데
>
> 무슨 화이트라벨 에러가 뜨는 것 이었다.
>
> 여기서 이제 생각 해 볼 수 있는 경우의 수는
>
> 서버와 통신 할 url과 단순 페이지 이동목적의 url이 겹쳐서 그럴 수 있다고 생각이 들었다.
>
> 당시에 겹치던 부분이 /qna/ 로 시작하는 부분과 /notice/ 로 시작하는 부분이었는데
>
> 이 겹치는 부분들의 url 앞 뒤를 교환 해 주었다.
>
> 그리고 그것들에 해당하는 proxy 화이트 리스트 설정을 해 주었다.
>
> #### /user/signin 응답 데이터 추가
>
> 마이페이지에서 사용할 기능중에 하나인 유저 데이터 변경에 쓰일 유저 데이터에 대한 부분을
>
> 로그인 할 때 미리 전부 로컬스토리지에 저장하도록 하기 위해서
>
> 서버로 부터 추가 데이터를 받도록 변경 해 주었다.

<br>

> ### 공지사항 게시판 기능 구현 & 인증 시스템 구현
>
> #### 인증시스템 구현
> 
> 우리홈페이지의 게시판 컴포넌트 구조는 다음과 같다.
> ```
> QnABoardList : localhost:3000/qna
> QnAView : localhost:3000/qna/:id
> NoticeBoardList : localhost:3000/notice
> NoticeView : localhost:3000/notice/:id
>
> Cod404 : localhost:3000/* // 아무런 url일 경우 404페이지로 이동
> Write : localhost:3000/write // 모든 게시판 쓰기와 수정을 담당하고 있음
> 게시판에 따라 달라지고, 수정 or 새 글 쓰기에 따라 달라져야한다.
> ```
> 
> 해당 컴포넌트에 따라 어떤 인증이 필요한지는 다음과 같다.
> ``` 
> QnABoardList, NoticeBoardList, QnAView, NoticeView : 인증이 필요없음
> write : 
> 1. qna게시판인 경우 : 
> 1-1. 새글쓰기 : 로그인이 필요함, 
> 1-2. 기존 글 수정 : 로그인인증 및 해당 글 글쓴이 인지 검증
> 2. notice게시판인 경우 : 
> 2-1. 새글쓰기 : 로그인 인증 및 어드민 인증, 
> 2-2. 기존 글 수정 : 로그인 인증 및 어드민 인증
> ```
>
> 따라서 총 필요한 인증시스템의 종류는\
> 
> **1. 로그인 인증, 2. 어드민 인증, 3. 해당 글 글쓴이 인증** 이 필요하다. 
>
> 이 모든 인증은 ```localhost:3000/auth```를 통해 구현되며
>
> option 오브젝트를 통해 제어되도록 설계한다.
> 
> **로그인 인증의 설계 및 구현**
>
> 로그인 인증의 설계는 
> 
> 이 User가 합당한 방법으로 실제 로그인이 되어있는지 검증해야 하는데
>
> 여기서 우리는 토큰 무결성을 검사하는것으로 채택하였다.
> 
> 따라서 서버에 현재 사용자의 토큰, 아이디값 을 보내어 
>
> 서버는 아이디값에 따른 유저 토큰정보를 DB에서 받아서
> 
> 클라이언트에서 POST 요청으로 보낸 토큰값과 DB상의 토큰값에 대한 문자열 검사(```String.equal()```)를 진행한다.
>
> 실제 사용되는 코드는 다음과 같다.
>
>```jsx
> const axiosAuth = (option) =>{
>  // option 객체의 내부 값은 { id : 아이디값, index : 글 고유인덱스, token: 토큰값, cate : post(게시판) or comment(댓글), 
>  // isEdit : true or false, isAdmin : true or false, isSignin : true or false }
>   const res = axios.post('/auth',option)
>    .then(response => response.data)
>    .catch( err =>{
>        return {
>            msg : `통신 오류(${err.request.status}) 발생하였습니다.`
>        }
>    })
> }
>```
>
> 위의 코드가 기본형이고 option 객체를 통해 어떤 인증을 하고싶은건지
> 
> 그리고 그 인증에 따라 어떤 값을 보낼 것인지를 결정한다.
> 
> 위의 로그인 인증 같은 경우는 토큰 검증 후 ```authSignin : true or false``` 를 보내어
>
> 최종 인증 결과값을 클라이언트로 전송한다.
>
> 위의 로그인 인증은 추가적으로 글쓴이 검증과 어드민 인증에서도 사용되는데
>
> 왜냐하면 무심코 해커가 로컬스토리지에 있는 값을 변경하여 서버에 변조된 값을 보낸다 하더라도
>
> 해당되는 유저가 실제로 로그인 되어있는지에 대한 값은 전부 서버 및 DB에 기록되어있기에
>
> 변조된 값을통해 글을 수정한다던지 하는 일은 일어날 수 없다.
> 
> **어드민 인증의 설계와 구현**
>
> 어드민 인증은 우선 하기전 대부분의 경우에서 로그인 인증과 함께 진행된다.
>
> 왜냐하면 로그인인증이 우선적으로 해당 유저가 로그인 실제로 되어있는지 하기 때문에
>
> 만약에 해커가 로컬스토리지 값을 변조하여 어드민 아이디 값을 보내더라도
>
> 어드민 인증에선 통과될지 언정, 로그인 인증에서는 통과가 안되기에 접근제한을 걸 수 있다.
> 
> 어드민 인증에서 필요한 값은 현재 로그인된 유저의 id값만 있으면 된다.
>
> 클라이언트에서 현재 로그인된 id값을 넘기면 
> 
> 서버에서 id값을 받아서 조회한 후 admin값이 1이면 adminAuth : true
> 
> 0이면 adminAuth : false를 진행시킨다.
>
> 이러한 어드민 인증은 주로 어드민 페이지나 공지사항 글쓰기 및 수정 할 때 쓰이며
>
> 일반 페이지에서는 페이지로드와 동시에 어드민 검사를 하여 현재 유저가 어드민이면
>
> 글 수정과 같은 행위에 접근제한을 해제시키는 목적으로 쓰이기도 한다.
> 
> **글쓴이 인증 설계 및 구현**
> 
> 위의 글쓴이 인증도 마찬가지로 로그인 인증과 함께 작동한다.
>
> 글쓴이 인증에 필요한 값들은 현재 로그인된 유저 id, 현재 보고있는 글 고유인덱스(댓글인 경우 댓글인덱스)
>  
> 게시판 인지 댓글인지를 구분짓는 cate값이 필요하다.
> 
> 그리고 글쓴이 인증에 앞서서 글 수정(댓글 수정)버튼의 활성화 및 비활성화는
> 
> 앞서서 페이지 로드 시 하는 어드민 인증을 통해 현재 유저가 어드민이라면 모든 수정버튼을 활성화 시키고
>
> 기본적으로 로그인 된 유저인 경우 로컬스토리지에서 꺼낸 아이디 값과 동일하다면 수정버튼을 활성화 시킨다.
>
> 결국 유저가 수정을 끝낸 후 submit 버튼을 눌렀다면, 서버로 필요한 데이터가 이동되고
>
> 서버에서는 글 고유 인덱스번호를 통해 조회하여 클라이언트에서 넘어온 아이디 값과 조회된 글쓴이를 비교하고
> 
> 두 값이 동일하다면 응답데이터로 editAuth : true를 보내어 인증을 완료 시킨다.
>
> #### 인증시스템 삽입 및 공지사항 게시판 기능 구현
>
> 위와 같이 만든 인증시스템을 필요한 부분에 넣어줌과 동시에 공지사항 관련 페이지를 구현한다.
>
> **QnAView.js**
> 
> 먼저 QnAView 페이지에서 axiosBoardView를 통해 넘어온 데이터의 길이가 0인경우
>
> 미리 만들어 둔 404페이지로 넘어가게 만든다.
>
> 또한 현재 유저가 로그인 된 상태라면 현재 아이디와 글쓴이 아이디를 비교하여
>
> 동일하다면 수정 버튼을 활성화 시킨다.
>
> 여기서 추가적으로 어드민 인증을 통해 현재 로그인 된 사용자가 어드민 권한이 있다면
>
> 굳이 글쓴이와 아이디가 동일하지않더라도 수정버튼이 활성화 되도록 구현한다.
>
> 인증시스템이 삽입된 구현부 코드이다.
> ```jsx
> const [isEditable, setIsEditable] = useState(false); // 글수정 버튼 토글용
>    const [isCurrentUserAdmin, setIsCurrentUserAdmin] = useState(false); // 현재 유저가 어드민인지 저장
>
>    useEffect(()=>{
>        const res = axiosBoardView(params.id, "qna");
>        res.then(data=>{
>            if(data.length === 0){ // 아무런 데이터도 안넘어온 경우
>                navigate("/404");
>            }
>            setContent(data);
>            setHashTag(data.tags);
>            if(JSON.parse(localStorage.getItem("token"))){
>                const userId = JSON.parse(localStorage.getItem("token")).id;
>                if(data.uploaderId === userId){ // 유저아이디와 글쓴이 아이디 검증
>                    setIsEditable(true);
>                }
>            }
>        })
>        if(JSON.parse(localStorage.getItem("token"))){
>            const userToken = JSON.parse(localStorage.getItem("token"));
>            const resOfAuth = axiosAuth({
>                id : userToken.id,
>                token : userToken.value, // signin인증에 필요한 토큰값
>                isSignin : true, // 어드민 인증 시 필요한 인증 
>                isAdmin : true 
>            })
>            resOfAuth.then(data =>{
>                if(data.adminAuth && data.signinAuth){ // 토큰무결성 통과 및 어드민 인증 통과 시
>                    setIsEditable(true);
>                    setIsCurrentUserAdmin(true);
>                }
>            })
>        }
>    },[])
> ```
> 그리고 기존의 글 수정 버튼시 ```useNavigate```의 state를 통해 이동되었던 데이터 값에
>
> 현재 유저가 어드민인지에 대한 값을 넘겨준다.
>
> 이로서 굳이 글 수정 페이지(/write)에서 추가적인 어드민 검증은 할 필요 없어진다.
>
> 그리고 쿼리스트링을 통해 어떤 게시판의 글을 수정하는지 url에 남긴다.
> 
> **NoticeView.js**
>
> 기본 구조는 위와 동일하나, 공지사항 게시판이기에 
>
> 글수정 버튼 활성화 조건은 무조건 어드민 인증이 되어야 한다.
>
> **NoticeBoardList.js**
>
> 별 다른 인증이 필요없고 ```QnaBoardList.js```와 동일한 구조를 가진다.
>
> **Write.js**
>
> 참고자료 : 
>
> [useSearchParams](https://reactrouter.com/en/main/hooks/use-search-params)
>
> [url 파라미터와 쿼리스트링](https://matdongsane.tistory.com/65)
>
> 우선에 이 페이지에 넘어온 후 로그아웃을 한다면 
>
> 글 수정 권한에서 벗어나야 하므로, 이전 페이지로 넘어가게 만든다.
> 
> ```jsx
> if(!JSON.parse(localStorage.getItem("token"))){
>            alert("로그아웃 상태입니다.");
>            return navigate(-1);
> }
> ```
>
> 또한 현재 로그인된 유저가 admin이 아닌데, 수정하려는 게시판이 notice인 경우도 접근을 막아야하며
>
> 이전에 넘어온 현재 사용자가 어드민인지에 대한 값은 useState를 통해 저장한다.
>
>```jsx
> const [isAdmin, setIsAdmin] = useState(false);
> 
> setIsAdmin(location.state.isCurrentUserAdmin);
> if(!location.state.isCurrentUserAdmin && location.state.board==="notice"){
>            alert("권한이 없습니다.");
>            return navigate(-1);
> }
>```
> 마지막으로 현재 어떤 게시판에 있는 글을 수정하는지에 대한 값을 
> 
> ```useSearchParams()```를 통해 url쿼리스트링에서 꺼내고
>
> if문과 useState를 통해 저장하여 필요할때 사용한다.
> ```
> const [title, setTitle] = useState("") // 어떤 게시판인지 나타내는 제목을 저장함
> ```
> 이제 사용자가 수정을 모두 마치고 submit버튼을 눌렀을 시 이다.
>
> submit버튼은 onClickHandler를 통해 처리되는데
>
> 우선에 이 순간에서도 로그아웃인 경우를 걸러내고
>
> 전송 할 body값 object를 만들어 둔다.
>
> 그리고 어떤 게시판이냐에 따라 우선에 갈리고 
> 
> 그 내부적으로 어떤 목적으로 ```Write.js```를 사용하느냐에 따라 분기되어야 한다.
>
> ```jsx
> if(location.state.board ==="qna") {
>   // 게시판이 qna인 경우
>   if(location.state.type === "edit"){
>    // qna 게시판에서 글 수정인 경우
>    // isEdit 검사를 통해 글쓴이 인증이 통과되면
>    // axiosPostUpdate() 함수를 통해 글을 업데이트 시킨다.
>   }else if(location.state.type ==="new"){
>    // 새 qna 글인 경우
>    // 따로 인증 할 것 없이 axiosPost()를 통해 글을 저장한다.
>   }
> }else if(location.state.board ==="notice"){
>    if(!isAdmin){
>       // 공지사항 게시판인 경우 수정 및 쓰기에 대해 어드민 권한이 필요하다.
>       return alert("권한이 없습니다.")
>    }else{
>       // 이전의 검사에서 어드민 권한을 얻은 경우 문제없이 수정 및 쓰기를 진행한다.
>    }
> }
> ```
>
> 위의 설계대로 구현된 모습은 아래와 같다.
> ```jsx
> const onClickHandler = (e)=>{
>        e.preventDefault();
>        if(!JSON.parse(localStorage.getItem("token"))){
>            return alert("로그아웃 상태입니다.");
>        }
>        const userToken = JSON.parse(localStorage.getItem("token"));
>        const body = {
>            "title" : contentTitle,
>            "tag" : [...tag],
>            "detail" : context,
>            "uploaderId" : userToken.id
>        }
>        if(location.state.board ==="qna"){
>            if(location.state.type ==="edit"){
>                const auth = axiosAuth({
>                    id : userToken.id,
>                    token : userToken.value,
>                    index : obj.id,
>                    isSignin : true,
>                    cate : "post",
>                    isEdit : true
>                })       
>                auth.then(data =>{
>                    if(data.signinAuth && data.editAuth){
>                        body['id'] = obj.id; // 몇번째 글을 수정하는 것 인지 body에 실어야함
>                        const resQnAEdit = axiosPostUpdate(body, location.state.board);
>                        resQnAEdit.then(data =>{
>                            alert(data.msg);
>                        })
>                    }else{
>                        alert("권한이 없습니다.");
>                        return navigate(-1);
>                    }
>                })
>            }
>            else if(location.state.type==="new"){
>                const resQnANew = axiosPost(body, location.state.board);
>                resQnANew.then(data=>{
>                    alert(data.msg);
>                })
>            }
>      
>       }else if(location.state.board ==="notice"){
>            if(!isAdmin){
>                return alert("권한이 없습니다.");
>            }else{
>                if(location.state.type ==="edit"){
>                    body['id'] = obj.id;
>                    const resNotiEdit = axiosPostUpdate(body, location.state.board);
>                    resNotiEdit.then(data =>{
>                        alert(data.msg);
>                    })
>                }else if(location.state.type ==="new"){
>                    const resNotiNew = axiosPost(body, location.state.board);
>                    resNotiNew.then(data =>{
>                        alert(data.msg);
>                    })
>                }
>            }
>        }
>    }
>```
> 위와 같이 구현 한 후 
>
> 공지사항 글 쓰기/수정인경우엔 태그가 필요없기에 분기문으로 분리하고
> 
> 최종적으로 render부분에서 ```title``` 값을 사용하여 동적인 제목을 구현하려 했는데
> 
> 이상하게 아무리 건드려도 title값에 변화가 없는 것 이었다.
>
> 정확하게는 title 변수의 변화는 있는데, 이를 react가 따로 감지하지 못하여 리렌더링이 일어나지 않은 문제였다.
>
> 따라서 useEffect 콜백함수의 두번째 인자인 depth 배열에
>
> ```searchParams.get("board")```를 넣어줌으로서
> 
> 쿼리스트링 값에 변화를 useEffect 훅이 감지하도록 구현하였다.
> ```jsx
> useEffect(()=>{
>   ...
>   if(searchParams.get("board")==="notice"){
>      setTitle("공지사항");
>   }else if(searchParams.get("board")==="qna"){
>      setTitle("QnA");
>    }
>    // 게시판 늘어날때 추가 해야 할 부분 : 게시판 제목
>    
> },[searchParams.get("board")])
> ...
> return(
>   <div className="App">
>        <Header/>
>        <div className="mainContentFrame">
>          <div className="innerArea border">
>            <div className="contentFrame writeFrame">
>                <div>{title} 게시글 작성</div> // 동적인 제목 UI
>                    <div>
>                        <div>
>                            <div>제목</div><input type="text" value={contentTitle} onChange={(e)=>{setContentTitle(e.target.value)}} name="contentTitle" required/>
>                        </div>
>                         // 토글형식의 태그 input UI
>                        { title === "QnA" ? 
>                        <div className="tagFrame">
>                            <div className="tagArea" id="tags">
>                           <div className="tagArea" id="tags">
>                                { tag.length > 0 ? tag.map(name =>(<HashTag forDelTag={tag} setHashTag={setTag} tagName={name} key={name}>{name}</HashTag>) ) : null }
>                            </div>
>                            </div> 
>                            <input className="tagInput" type="text" placeholder="태그를 입력해주세요" onKeyUp={onEnterKeyUpHandler} name="contentTitle"/>
>                        </div>
>                        : null }  
>  ...
> )
> ```
  
<br>
  
> #### Header.js 수정
>
> 기존의 헤더는 임시방편으로 글 목록으로 이동하고 글쓰기 페이지로 이동하도록 구현되어있다.
> 
> 이 부분에 대해서 정해진 정책에 따라 변경한다.
> 
> 먼저 글 쓰기 페이지는 일반적인 qna글쓰기 페이지를 뜻 하므로 
> 
> 로그인 된 사용자만이 볼 수 있도록 하고
> 
> 공지사항 글쓰기 에 대한 버튼을 어드민인증을 통해 어드민 유저가 로그인 된 경우
> 
> 공지사항 글쓰기 버튼을 헤더에 활성화 시키도록 구현한다.
>
> 구현된 코드는 다음과 같다.
>```jsx
> function Header(){
>   useEffect(() => {
>        if(localStorage.getItem("token")){
>        ...
>            const res = axiosAuth({
>                id : userData.id,
>                token : userData.value,
>                isSignin : true,
>                isAdmin : true
>            })
>            res.then(data =>{
>                if(data.adminAuth && data.signinAuth){
>                    setIsCurrentUserAdmin(true);
>                }
>            })
>        }
>    }, [token])
>   ...
>   return (
>     ...
>      // 현재 유저가 어드민이라면 나타내어질 헤더 요소
>      {isCurrentUserAdmin ? <div style={{cursor:"pointer"}} onClick={()=>{ navigate('/write?board=notice',{state:{ type:"new", board:"notice", isCurrentUserAdmin : isCurrentUserAdmin }}) }}>공지사항 글쓰기</div> : null }
>      {isCurrentUserAdmin ? <span>|</span> : null}
>      // 현재 로그인 되어있다면 나타내어질 헤더 요소
>      {token ? <div style={{cursor:"pointer"}} onClick={()=>{ navigate('/write?board=qna',{state:{ type:"new", board:"qna", isCurrentUserAdmin : isCurrentUserAdmin }}) }}>글쓰기</div> : null }
>      {token ? <span>|</span> : null}
>   )
>```

후기

이제 백엔드 로직또한 내가 건드리며 수정 해야하는 일이 생겼는데

물론 이제 백엔드 기존의 친구가 많이 도와주었지만, 지금하고 있는 프로젝트를 재대로 리더로서 담당하려면

스프링도 잘 이해하고 있어야 할 것 같다.

지금은 간단간단하게 API 및 DB 쿼리문 수정이라 큰 문제는 없었다.

코로나 이슈로 일주일정도 휴식 후 이제 게시판을 완성 해 볼까 싶어서 코드를 실행하는데

프록시 에러로 실행이 안되었을때 정말 많이 당황했다.

구글에 검색했을때 정보가 거의 없었지만, 
  
유일하게 유선에서 안되는 현상이 있다는점이 나와 공통점이라 참 다행스럽게 생각한다.

수동 프록시 화이트리스트 설정은 정말 정말 짜증이 났지만, 설정하면서 동시에 API설계는 중요하다는 점을 느꼈다.

쓸데없는 url 파라미터를 늘리지 않아야 화이트리스트도 깔끔해지기 때문이다.

















