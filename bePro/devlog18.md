# Jan. 03. 2023 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Jan. 04. 2023

## 개발 내용
>
> ### 검색 결과 페이지 구현
>
> #### 라우팅 변경
> 
> **공통 컴포넌트 Header 묶기 & 검색결과 페이지 라우팅**
>
> 기존의 라우팅 방식은
>
> 개별 라우트 : 
> 
> 메인페이지(```"/"```), 마이페이지(```"/mypage"```), 글 수정/쓰기 페이지(```"/write"```)
> 
> 게시판 별 라우트 : 
> 
> 게시판 리스트(```"/게시판"```), 게시판 게시글 상세보기(```"/게시판/:id"```), 검색결과 페이지(```"/게시판/search"```)
>
> 로 구성되어 있는데,
> 
> 여기서 개별 페이지나 게시판별 페이지 모두 공통 컴포넌트인 Header가 있으므로
> 
> ```<Root/>```컴포넌트를 만들어서 그 내부에 ```<Outlet/>```을 활용하여 공통 레이아웃을 처리해준다. 
>
> 그리고 검색결과페이지는 현재 개발된 API가 통합검색 기능으로 구현되어 있으므로
> 
> 특정 게시판 별 검색결과 가 아닌 모든 결과값을 출력 할 페이지로 구현해야 한다.
> 
> 따라서 ```"/search"```주소를 기준으로 통합 검색결과 페이지 ```<Search/>``` 컴포넌트를 불러오도록 라우팅 한다.
> ```javascript
> // in Root.js...
> const Root = () =>{
>   return(
>     <>
>       <Header/> // 중첩된 헤더 컴포넌트를 렌더링
>       <Outlet/> // 중첩된 라우팅 구조상 자식 라우트의 컴포넌트를 렌더링
>     </>
>   )
> }
> ```
> ```javascript
> function App(){
>   return(
>     ...
>     <Routes>
>       <Route path="/" element={<Root/>}> // 다른 모든 라우팅을 루트 컴포넌트 하위로 둠으로서 공통 레이아웃 처리
>        <Route index element={<MainPage/>}/>
>         <Route path="/mypage/*" element={<MyPage/>}/>
>         <Route path="/qna" element={<QnABoard/>}>
>          <Route index element={<QnABoardList/>}/>
>          <Route path=":id" element={<QnAView/>}/>
>          <Route path="*" element={<Cod404/>}/>
>         </Route>
>         <Route path="/write" element={<Write/>}/>
>         <Route path="/notice" element={<NoticeBoard/>}>
>          <Route index element={<NoticeBoardList/>}/>
>          <Route path=":id" element={<NoticeView/>}/>
>          <Route path="*" element={<Cod404/>}/>
>         </Route>
>         <Route path="/search" element={<Search/>}/> // 통합 검색결과 페이지로서의 기능을 할 페이지
>       <Route/>
>     </Routes>
>     ...
>   )
> }
> ```
> 
> #### 검색창 위치변경 & navigate 변경
> 
> 기존의 검색창은 초기 설계 때 메인페이지에 임시로 둔 검색창이다.
> 
> <img src="https://user-images.githubusercontent.com/81299056/210476413-0095fa39-b7a1-45c4-b1ff-1cbb2f0b3a54.png" width="100%" height="100%"/>
>
> 과거에 구현했던 검색 API가 통합 검색의 기능을 수행하고 있으므로
> 
> 어디서든 사용자가 검색 할 수 있게 헤더에 아이콘을 추가하여
> 
> 검색창을 토글방식으로 on/off 가능하도록 구현하였다.
> <img src="https://user-images.githubusercontent.com/81299056/210476747-cec7fcd2-3a41-402a-9c9e-bfa1c76e8fbd.gif" width="100%" height="100%"/>
> 
> "검색"버튼을 누를 시 기존의 ```useNavigate```위치를 라우팅으로 잡아준 ```"/search"```로 변경하고
>
> 쿼리스트링으로 사용자가 입력한 검색어 및 태그를 추가한다.
> ```javascript
> onClickHandler = () =>{
>   // useState로 저장되어 있는 value(검색어 값)와 hashTag(추가된 태그 배열)사용
>   navigate(`/search?q=${value}&tags=${hashTag.join(',')}`);
> }
> ```
> #### 검색 결과 페이지 구현
>
> **설계**
>
> > **레이아웃 설계**
> > 
> > 기본적인 레이아웃 구성은 메인페이지와 동일하기에 ```<Card/>, <SideBar/>```컴포넌트를 재사용 할 것이다.
> >
> > 통합검색 결과를 사용자에게 보여주어야 하기 때문에
> >
> > 검색 API의 결과로 얻은 글 목록을 응답 데이터 내부 category값에 따라 분류하여 
> >
> > 위쪽에는 QnA 결과값, 아래쪽에는 공지사항 결과값을 렌더링 할 것이다.
>
> <br>
>
> > **작동 순서**
> > 
> > 사용자가 검색창에 검색을 하면 ```/search```라우터에 쿼리스트링으로
> > 
> > ```?q=검색어&tags=태그들```이 붙은채로 ```useNavigate```훅을 통해 페이지를 이동하게되고
> > 
> > 페이지 초기 렌더링 후 ```useEffect```를 통해 쿼리스트링 값을 받아서 검색 API 요청을 통해 
> > 
> > 검색 결과값을 받은 후 category별 분류를 통해 <Card/> 컴포넌트에 각각 렌더링 하게 된다.
>
> **구현**
> 
> > **Search 컴포넌트 구현**
> > 
> > 렌더링 부분에서 ```<MainPage/>```컴포넌트의 렌더링 일부를 가져온다.
> > 
> > 총 결과값 개수를 알려줄 부분과 ```<Card/>``` 컴포넌트 재사용을 위해 props를 수정한다.
> > ```javascript
> > const Search = () => {
> >   ...
> >     return(
> >         <div className="App">
> >           <div className="mainContentFrame">
> >             <div className="innerArea border">
> >               <div className="contentFrame">
> >                 <SideBar name={"# 실시간 태그순위"}/>
> >                 <div className="cardFrame">
> >                   // resOfQnA : 검색API 응답데이터 중 category==="qna"에 해당하는 것
> >                   // resOfNoti : 검색API 응답데이터 중 category==="notice"에 해당하는 것
> >                   <div className="subject">총 {resOfQna.length+resOfNoti} 건</div> // 총 결과값 개수
> >                   <div className="cardArea">
> >                   // name : Card컴포넌트 제목, data : 메인페이지용이 아닌 경우 props로 데이터를 넘겨줌
> >                   // referrer : 지금 Card컴포넌트를 사용중인 페이지 종류
> >                     <Card name={"QnA 검색결과"} data={resOfQna} referrer={"search"}/>
> >                   </div>
> >                   <div className="cardArea">
> >                     <Card name={"공지사항 검색결과"} data={resOfNoti} referrer={"search"}/>
> >                   </div>  
> >                 </div>
> >               </div>
> >             </div>
> >           </div>
> >         </div>
> >     )
> > }
> > ```
> > 초기 렌더링 이후 알맞은 검색어와 함께 검색 API를 호출해야 하므로
> > 
> > ```useEffect```를 사용하며, ```useSearchParams```훅을 통해 
> > 
> > 쿼리스트링을 검색어 부분과 태그 부분을 분리하여 검색 API 함수 인자로 넘겨주고
> > 
> > 응답데이터를 분석하여 카테고리별로 분리하여 두 개의 분리된 배열을 useState로 저장한다.
> > 
> > ```javascript
> > const Search = () =>{
> >     const [searchParams] = useSearchParams();
> >     const [resOfQna, setResOfQna] = useState([]); 
> >     const [resOfNoti, setResOfNoti] = useState([]); 
> >     useEffect(()=>{
> >         let qna = [];
> >         let noti = [];
> >         let tags = [];
> >         const q = searchParams.get("q");
> >         if(searchParams.get("tags")>1){
> >             tags = [...searchParams.get("tags").split(",")];
> >         }
> >         const res = axiosSearch(q, tags); // 검색 api 호출
> >         res.then(data=>{
> >             data.map((context)=>{
> >             // 카테고리 별 분리
> >                 if(context.category ==="qna"){
> >                    return qna.push(context);
> >                 }else{
> >                    return noti.push(context);
> >                 }
> >             })
> >             // useState로 저장
> >             setResOfQna(qna);
> >             setResOfNoti(noti);
> >         })
> >     },[searchParams])
> >     ...
> > }
> > ```
>
> <br/>
>
> > **Card 컴포넌트 변경**
> > 
> > 기존의 카드 컴포넌트는 메인에서만 사용하였으므로 
> > 
> > 목적에 따라 ```bindCardContent```에서 메서드를 받아서 응답데이터를 렌더링 하였다.
> > 
> > 하지만 이번 검색결과페이지에서 재사용하기 위해서는 
> > 
> > ```bindCardContent```에 새로운 ```<Card/>```컴포넌트의 제목을 추가하고
> > 
> > 추가한 제목에 따른 리턴값에는 메서드가 없으므로 props로 넘어온 데이터를 setContent로 저장한다.
> > 
> > ```props.referrer```값을 통해 search인 경우 제목에 결과 값 개수를 함께 렌더링하도록 setTitle로 변경한다.
> > 
> > 검색결과가 없는경우 검색결과값이 없다는 안내문구를 렌더링한다.
> > ```javascript
> > const bindCardContent = (type) =>{
> >     const method = {
> >         "조회수 높은 순" : "qna",
> >         "최신 QnA" : "qna",
> >         "공지사항" : "notice",
> >         // 검색결과 페이지 추가
> >         "QnA 검색결과" : "qna",
> >         "공지사항 검색결과" :"notice"
> >     }
> >     // 목적이 추가 됨에 따라 조건 추가
> >     if(type ==="조회수 높은 순"){
> >         return {
> >             boardType : method[type],
> >             method : axiosGetViewTitle   
> >         }
> >     }else if(type==="QnA 검색결과" || type==="공지사항 검색결과"){
> >         return{
> >             // 메서드가 없음 => props.data로 받아오기 때문
> >             boardType : method[type]
> >         }
> >     }
> >     else{
> >         return {
> >             boardType : method[type],
> >             method : axiosMainBoard
> >         }
> >     }
> > }
> > ```
> > ```javascript
> > const Card = (props) => {
> >     const [title, setTitle] = useState("");
> >     const [content, setContent] = useState([]); // 기존에 내용을 저장하던 contentTitle에서 이름만 변경
> >     const [boardType, setBoardType] = useState("");
> >     const navigate = useNavigate();
> > 
> >     useEffect(() => {
> >       setTitle(props.name);
> >       const obj = bindCardContent(props.name);
> >       setBoardType(obj.boardType);
> >       if(obj.method){ // 메서드가 있다면 메인페이지에서 사용하는 목적
> >         const method =obj.method;
> >         method(obj.boardType)
> >         .then( data =>{
> >           setContent(data);
> >         })
> >       }else{ // 검색결과 페이지 등에서 사용 할 목적
> >         if(props.referrer==="search"){
> >           // 검색결과 페이지에서는 카테고리별 개수를 제목에 붙임
> >           setTitle(`${props.name} ${props.data.length} 건`);
> >         }
> >         setContent(props.data);
> >       }
> >     },[props.name,props.data,props.referrer])
> > 
> >     const onLinkHandler = (e) =>{
> >       if(e.target.innerHTML !== "조회수 높은 순"){
> >         // 기존의 navigate방식보다는 boardType을 그대로 navigate에 사용하면 된다.
> >         navigate(`/${boardType}`);
> >       }
> >     }
> > 
> >     return (
> >         <div className="card">
> >           <div onClick={ onLinkHandler} className="cardTitle">{title}</div>
> >           <div className="cardDetail">
> >             {content.length > 0 ? 
> >             content.map((data, index)=>{
> >               return <div onClick={()=>{navigate(`/${boardType}/${data['id']}`)}} className="cardContentTitle" key={data.id}>제목 : {data.title}|조회수 : {data.view}|글쓴 날짜 : { data.uploadtime }</div>
> >             }) : 
> >             // 검색결과가 없는 경우
> >             <div>
> >               <div>검색결과가 없어요!</div>
> >               <div>다른 검색어나 태그로 검색 해 보세요.</div>
> >             </div>}
> >           </div>
> >         </div>
> >     )
> > }
> > ```

<br>

> ### 버그 고치기
> 
> #### 글 수정 상태에서 글 쓰기를 시도 할 때, 수정할 때의 값이 남아있는 버그 수정
> 
> 원인은 같은 페이지를 사용하는데 있어서 상태 변경 시 state값을 지우지 않아서 생긴 문제
>
> 기존의 쿼리스트링에 type값을 더하고, navigate의 location.state값에 type값을 제거하여 
>
> 해당 type값이 new 라면 state값을 초기화 하는걸로 해결
> ```javascript
> // in Header.js...
> function Header(){
> ...
>   return(
>     ...
>      // 쿼리스트링에 type값 추가 및 state값에 type, board 제거
>      
>     {isCurrentUserAdmin ? <div style={{cursor:"pointer"}} onClick={()=>{ 
>     navigate('/write?board=notice&type=new',{state:{ isCurrentUserAdmin : isCurrentUserAdmin }}) }}>공지사항 글쓰기</div>
>      : null }
>      
>     {token ? <div style={{cursor:"pointer"}} onClick={()=>{     
>       navigate('/write?board=qna&type=new',{state:{ isCurrentUserAdmin : isCurrentUserAdmin }}) }}>글쓰기</div> 
>       : null }
>     ...
>   )
> }
> ```
> ```javascript
> // in QnAView.js...
> const QnAView = () =>{
> ...
>   onClickHandler = () =>{
> // 쿼리스트링에 type값 추가 및 state값에 board 제거
>     navigate('/write?board=qna&type=edit', {
>            state:{
>                data : content,
>                isCurrentUserAdmin : isCurrentUserAdmin
>            }
>      })
>   }
> }
> ```
> ```javascript
> // in Write.js...
> const Write = (props) => {
>   ...
>   // 쿼리스트링으로 넘어온 값 저장용
>   const [type, setType] = useState(""); 
>   const [board, setBoard] = useState("");
>   ...
>   // useSearchParams 활용
>   setType(searchParams.get("type"));
>   setBoard(searchParams.get("board"));
>       if(searchParams.get("type")==="edit"){
>           setContentTitle(obj.title);
>           setContext(obj.detail);
>           setTag(obj.tags);
>       // 쿼리스트링의 type값이 new인 경우 state값 초기화
>       }else if(searchParams.get("type")==="new"){
>           setContentTitle("");
>           setContext("");
>           setTag([]);
>       }
>   ...
>    },[searchParams, obj,location.state.isCurrentUserAdmin])
> ```
>
> #### whitelabel 에러 고치기
>
> 기존의 API명과 라우팅 명이 겹쳐서 또 에러가 발생하였다.
>
> axiosSearch에서 사용하는 API를 ```"/search"```에서 ```"/post/searchquery"```로 변경 하였다.

<br/>

후기

역시나 페이지별 서버를 거치는 인증을 할 때

초기 설계의 중요성을 다시금 알게 되었다.

이번에 버그를 고치면서 분기문들이나 조건들을 좀 최적화 하면서도 느끼지만

너무 쓸데없이 한 곳에 통합하여 구현하려 했던게 잘못된것 같기도 한다.
