# Jan. 07. 2023 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Jan. 10. 2023

## 개발 내용
>
> ### 마이페이지 구현
>
> #### 마이페이지 설계
> 
> **마이페이지 기능별 설계**
>
> 마이페이지는 ```localhost:3000/mypage``` 로 시작하며
> 
> 중첩 라우팅을 사용하여 ```edit_profile : 회원정보 수정 페이지``` 및 
>
> ```manage : 쓴글 관리 페이지``` 그리고 마이페이지 내부 페이지 이동에 필요한 사이드 메뉴를 구현한다.
>
> **/mypage**
>
> 마이페이지 내부 기본 레이아웃을 구현 (```Header.js``` etc...) 
>
> 마이페이지를 거쳐야 하는 라우팅들 (```/mypage/edit_profile``` etc...)의 각종 인증을 담당함.
> 
> - 기본적으로 로그인이 된 사용자만 사용가능해야함 
> - 어드민인증을 해야하는 이유는 어드민은 쓴글관리 페이지에서 공지사항 글까지 확인 가능해야함
>
> 사이드메뉴 컴포넌트와의 소통으로 현재 어떤 페이지가 보여지고 있는지를 나타낼 수 있음 (UI적인 측면)
> 
> **/mypage/edit_profile**
> 
> 회원정보 수정 페이지로서 다음과 같은 과정을 거친다.
> 
> 1. 로컬스토리지에 있는 유저정보를 들고와서 수정가능한 상태로 사용자에게 보여준다.
> 
> 2. 사용자가 수정을 완료 한 후 'Update'버튼을 누를 시 본인 확인용 로그인 모달창을 띄운다.
> 
> 3. 로그인 모달창에서 로그인인증이 완료 되었을 시 모달창이 자동으로 꺼진다.
> 
> 4. 로그인 인증이후 모달창이 꺼진뒤, 곧바로 서버와 통신을 하여 유저정보를 업데이트 시킨다.
>
> 이 때 수정 가능한 상태의 유저정보는 ```<SignUp/>``` 컴포넌트를 재사용 하고
>
> 로그인 모달창은 ```<SignIn/>``` 컴포넌트를 재사용한다.
> 
> **/mypage/manage**
> 
> 쓴글 관리 페이지로서 다음과 같은 과정을 거친다.
> 
> 1. 로컬스토리지에 유저 정보중 아이디를 꺼내어 서버에 전송한다.
> 
> 2. 해당 API를 통해 얻은 정보를 카테고리별 분리한다.(qna or notice)
> 
> 3. 분리된 각각의 데이터 중 어드민인 경우에만 공지사항 글을 노출시킨다.
> 
> 4. 이 때 자신의 글을 곧바로 수정할 수 있는 버튼을 추가한다.
>
> 여기서 자신이 쓴 공지사항 글 이나 QnA글은 ```<Card/>``` 컴포넌트를 재사용한다.
> 
> **SideMenu**
>
> 마이페이지 컴포넌트와 소통하여 현재 어떤 페이지가 화면에 렌더링 되어 있는지를 파악 할 수 있고
> 
> 페이지별 이동에도 사용 가능하다.
> 

<br>
  
> #### 마이페이지 기능별 구현
>
> **라우팅 구현**
>
> 중첩 라우팅을 사용하여 ```/mypage```로 시작하도록 구현한다.
>
> ```javascript
> // App.js...
> function AppRoute() {
>  return (
>    <Router>
>      <div>
>        <Routes>
>          <Route path="/" element={<Root/>}>
>            <Route index element={<MainPage/>}/>
>            <Route path="/mypage/" element={<MyPage/>}>
>              <Route index element={<Cod404/>}/>
>              <Route path="edit_profile" element={<EditProfile/>}/>
>              <Route path="manage" element={<Manage/>}/>
>              <Route path="*" element={<Cod404/>}/>
>            </Route>
>       ...
>  )
> }
> ```
> 
> **마이페이지 구현**
>
> 참고자료 : 
>
> [React-Context](https://ko.reactjs.org/docs/context.html)
>
> [ReactRouter-useOutletContext](https://reactrouter.com/en/main/hooks/use-outlet-context)
>
> ```useEffect```훅을 사용하여 초기 렌더링 이후 프론트 로컬스토리지 검사를 통해
>
> 일차적으로 로그인 상태의 유저가 접근한건지 검사하고,
>
> 로그인 상태로 파악되면 서버에 Auth API를 통해 실제 로그인 상태인지 와 어드민 유저인지를 검사한다.
>
> ```useState```훅을 통해 메뉴 리스트(```manage```와 ```edit_profile```), 현재 선택된 페이지,
>
> 그리고 Auth API를 통해 받은 데이터를 저장한다.
> 
> 랜더링에서 기본 프레임은 메인페이지의 프레임을 재활용하고 
>
> ```<SideMenu/>```컴포넌트와의 소통을 위해 메뉴리스트와 현재 선택된 페이지를 props로 보낸다.
>
> 마지막으로 다른 중첩된 컴포넌트에 공통레이아웃을 렌더링 하기 위해 ```<Outlet/>```를 활용한다.
>  
> 이 때 ```<Outlet/>```컴포넌트로 렌더링 될 다른 컴포넌트들에 공통적인 값을 ```prop```로 전달하기 위해 ```context```를 사용한다.
>```javascript
> function MyPage(){
>    // 사이드 메뉴에 전달 할 메뉴리스트로 menu_value는 라우터 명과 동일, render는 실제 렌더링되어 보여질 값으로 전달한다.
>    const [menuList] = useState([{menu_value : "manage",render: "쓴글 관리"},{menu_value : "edit_profile", render : "회원정보 관리"}]);
>    const [currentSelected, setCurrentSelected] = useState(""); // 현재 어떤 페이지인지 전역으로 관리할 state
>    const [isCurrentUserAdmin, setIsCurrentUserAdmin] = useState(false); // 현재 로그인된 유저가 어드민인지
>    const [userAuth, setUserAuth] = useState(false); // 현재 해당유저가 실제 로그인 된 상태인지
>    const navigate = useNavigate();
>
>    useEffect(()=>{
>       // 프론트 상 로그인 유저인지 검사
>        if(!JSON.parse(localStorage.getItem("token"))){
>            alert("권한이 없습니다.");
>            return navigate("/");
>        }
>        const lsValue = JSON.parse(localStorage.getItem("token"));
>       // 서버 AuthAPI 사용
>        const res = axiosAuth({
>            id : lsValue.id,
>            token  : lsValue.value,
>            isAdmin : true,
>            isSignin : true
>        })
>        res.then(data=>{
>            if(!data.signinAuth){
>                alert("권한이 없습니다.");
>                return navigate("/");
>            }
>            setUserAuth(data.signinAuth);
>            setIsCurrentUserAdmin(data.adminAuth);
>        })
>    },[])
>
>    return(
>        <div className="App">
>        <div className="mainContentFrame">
>          <div className="innerArea border">
>            <div className="contentFrame">
>                 // 메뉴리스트와 현재 선택된 페이지를 넘겨받는다
>                <SideMenu menuList={menuList} current={currentSelected}/>
>                 // 나중에 useOutletContext를 통해 인증값과 현재 선택된 페이지 값을 부모컴포넌트와 소통
>                <Outlet context={{setCurrentSelected, isCurrentUserAdmin, userAuth}}/>
>            </div>
>          </div>
>        </div>
>      </div>
>    )
>}
>```
> **사이드 메뉴 구현**
>
> 사이드메뉴는 마이페이지 컴포넌트에서 전역으로 관리하고 있는 메뉴리스트를 ```props```로 받아서 렌더링 한다.
>
> 이 때 각각의 메뉴 요소를 클릭 할 시 해당하는 페이지가 렌더링 되고, 
>
> 현재 렌더링 된 페이지가 어떤 것인지를 나타내야 한다.
>
> ```javascript
> const SideMenu = (props) =>{
>    const [menuList, setMenuList] = useState([]);
>
>    const navigate = useNavigate();
>
>    useEffect(()=>{
>        setMenuList(props.menuList); // 마이페이지에서 받은 메뉴리스트를 저장
>    },[props.current])
>
>    return(
>        <div className="sideMenuFrame">
>            {
>            menuList.map((data,index)=>{ // Array.map((value, index))함수를 통해 메뉴리스트 렌더링
>                return(
>                     // 각각의 리스트 요소를 클릭 할 시 해당하는 페이지로 연결되도록 링크
>                    <div className="sideMenuElement" key={data.menu_value}>
>                        <div onClick={()=>{
>                            navigate(`/mypage/${data.menu_value}`);
>                         // 특히 현재 선택된 페이지는 CSS적으로 구분이 되도록 렌더링
>                        }} className={props.current === data.menu_value ? "sideMenuElementInside currentSelected" : "sideMenuElementInside"}>{data.render}</div>
>                    </div>
>                )    
>            })
>            }
>        </div>
>    )
>}
> ```
> 사이드메뉴 테스트
>
> 위의 url과 사이드메뉴를 비교하면서 확인
>
> <img src="https://user-images.githubusercontent.com/81299056/211449945-54d296b3-b4c5-447e-b297-620d38143f0b.gif" width="100%" height="100%"/>
>
> **쓴글 관리 페이지 구현**
> 
> useOutletContext를 활용하여 context로 넘겨준 값들 받음
>
> useEffect 를 활용하여 초기렌더링 이후 서버 통신을 통해 받아야 할 값들을 받음
>
> context로 넘겨받은 것들 중 setCurrentSelected함수에 
> 
> 현재 url주소 중 현재 페이지를 나타내는 부분만 잘라서 인자로 전달
>
> axiosGetPostings() 함수를 통해 현재 해당 id를 통해 조회된 모든 글 데이터를 받음
>
> 받은 데이터를 ```category``` 값에 따라 분리하고 useState로 저장
>
> ```<Card/>```컴포넌트의 재 사용으로 name값을 포함하여 referrer과 data를 ```props```로 전달 
>
> data : 현재 글 데이터, referrer : 현재 ```Card```컴포넌트 사용위치 
>
> 랜더링 부분에서 기본 ```<Card/>```컴포넌트의 틀을 구성하고
> 
> 넘겨받은 ```isCurrentUserAdmin```값에 따라 공지사항 글 까지 렌더링 할 것인지 여부를 결정
>
> ```javascript
> const Manage = (props) =>{
>    const [resOfQna, setResOfQna] = useState([]);
>    const [resOfNoti, setResOfNoti] = useState([]);
>    // Outlet 컴포넌트에서 context 로 넘겨받은 값들 임시저장
>    const { setCurrentSelected , isCurrentUserAdmin, userAuth} = useOutletContext();
>    const location = useLocation();
>    useEffect(()=>{
>       // 현재 url값을 pathname으로 받으면 /mypage/manage인데 여기서 두번째면 manage
>        setCurrentSelected((location.pathname.split("/"))[2]);
>        if(!localStorage.getItem("token")){
>            return;
>        }
>        let body = {
>            id : JSON.parse(localStorage.getItem("token")).id
>        }
>       
>        const res =  axiosGetPostings(body);
>        res.then(data=>{
>            let qna = [];
>            let notice = [];
>            if(data.length>0){
>                data.map((value)=>{
>                    if(value.category ==="qna"){
>                        return qna.push(value);
>                    }else{
>                        return notice.push(value);
>                    }
>                })
>            }
>            setResOfQna(qna);
>            setResOfNoti(notice);
>        })
>    },[isCurrentUserAdmin, userAuth])
>
>    return (
>        <div className="cardFrame">
>            <div className="cardArea">
>                <Card name={"내가 쓴 QnA"} referrer={"mypage"} data={resOfQna} isCurrentUserAdmin={isCurrentUserAdmin}/>
>            </div>
>            { isCurrentUserAdmin ? 
>            <div className="cardArea">
>                <Card name={"내가 쓴 공지사항"} referrer={"mypage"} data={resOfNoti} isCurrentUserAdmin={isCurrentUserAdmin}/>
>            </div>  : null}
>        </div>
>    )
>}
> ```
> > **Card 컴포넌트의 확장**
> >
> > 기본적인 확장의 틀은 저번 Search 컴포넌트를 만들 때 확장 해 두었으므로
> >
> > bindCardContent에서 추가적인 ```name```값에 대한 ```boardType``` 매핑을 추가하기만 하면 된다.
> > 
> > 추가적으로 각각의 본인 글에 대해 바로 수정할 수 있도록 버튼을 추가하며, 
> > 
> > 글이 없을 경우 예외 렌더링을 referrer 별로 나눈다.
> > 
> > ```javascript
> > //bindCardContent.js
> > const bindCardContent = (type) =>{
> >     const method = {
> >         ...
> >         "내가 쓴 QnA" : "qna",
> >         "내가 쓴 공지사항" : "notice"
> >     }
> >     ...
> >     }else if(type==="QnA 검색결과" || type==="공지사항 검색결과" || type ==="내가 쓴 QnA" || type ==="내가 쓴 공지사항"){
> >         return{
> >             boardType : method[type]
> >         }
> >     }
> >     ....
> > }
> > ```
> > ```javascript
> > const Card = (props) => {
> >  ...
> >     return (
> >         <div className="card">
> >           <div onClick={ onLinkHandler} className="cardTitle">{title}</div>
> >           <div className="cardDetail">
> >             {content.length > 0 ? 
> >             content.map((el, index)=>{
> >               return (
> >                 <div className="cardElement">
> >                   <div onClick={()=>{navigate(`/${boardType}/${el['id']}`)}} className="cardContentTitle" key={el.id}>제목 : {el.title}|조회수 : {el.view}|글쓴 날짜 : { el.uploadtime }</div> 
> >                   { props.referrer === "mypage" ?
> >                   <button onClick={()=>{navigate(`/write?board=${boardType}&type=edit`,{
> >                     state : {
> >                       data : el,
> >                       isCurrentUserAdmin : props.isCurrentUserAdmin
> >                     }
> >                   })}}>수정</button> 
> >                   : null}
> >                 </div>
> >               )
> >             }) : 
> >             <div>
> >               { props.referrer ==="search" ? 
> >               <>
> >                 <div>검색결과가 없어요!</div>
> >                 <div>다른 검색어나 태그로 검색 해 보세요.</div>
> >               </> : props.referrer ==="mypage" ? 
> >               <>
> >                 <div>아직 글을 쓴적이 없네요!</div>
> >                 <div>상단바에 글쓰기를 통해 자유롭게 글을 써 보세요</div>
> >               </> : props.referrer === "main" ? 
> >               <>
> >                 <div>bePro는 여러분의 글을 기다리고 있어요</div>
> >               </> : null
> >               }
> >             </div>}
> >           </div>
> >         </div>
> >       )
> > ```
>
> **회원정보 수정 페이지 구현**
>
> 회원정보 수정 페이지는 ```SignIn```컴포넌트와 ```SignUp```를 하여 구현한다.
> 
> ```SignUp``` : 사용자가 자신의 정보를 수정 할 수 있도록 컴포넌트 제공
> 
> ```SignIn``` : 사용자 인증용 로그인 모달창
>
> 쓴글 관리 페이지와 동일하게 ```context```로 넘겨받은 함수를 처리 해 준다.
> 
> SignIn 모달창 on/Off 상태관리와 SignUp을 활용해 업데이트 할 유저정보를 저장하기위해 useState를 사용한다.
> 
> ```javascript
> const EditProfile = () =>{
>     
>     const [signInModal, setSignInModal] = useState(false); // 모달창 on/off
>     const [updateUserData, setUpdateUserData] = useState() // 업데이트 할 유저 정보
>      // 쓴글 관리 페이지에서도 동일했던 과정
>     const { setCurrentSelected } = useOutletContext();
>     const location = useLocation();
>     const param = (location.pathname.split("/"))[2];
>     useEffect(()=>{
>         setCurrentSelected(param);
>     },[])
> 
>     return(
>         <>
>           // SignIn을 확장하기 위해 referrer : 사용한 컴포넌트, userData : 업데이트 할 유저정보를 넘긴다.
>             {signInModal ? <SignIn referrer={param} forClose={setSignInModal} userData={updateUserData}/> : null}
>           // SignUP을 확장하기 위해 referrer : 사용한 컴포넌트, setUpdateUserData : 업데이트 할 유저정보를 설정할 함수를 넘긴다.
>             <SignUp referrer={param} forSetModal={setSignInModal} setUpdateUserData={setUpdateUserData}/>
>         </>
>   )
> }
> ```
> > **SignUp 컴포넌트의 확장**
> > 
> > 초기 렌더링을 한 후 각 input값에 기존의 유저정보를 미리 입력시켜야 하므로
> > 
> > useEffect를 통해 referrer 검사 후 유저정보를 set함수를 통해 입력시킨다.
> >
> > 렌더링 부분에서 CSS값을 referrer 값 기준으로 회원가입용 과 회원정보 수정용으로 나눈다.
> > 
> > 특히 form 태그의 이벤트 헨들러함수도 referrer값을 기준으로, 회원가입용과 회원정보 수정용으로 나뉜다.
> > 
> > 회원정보 수정용 함수에서는 submit이벤트 작동 시 
> > 
> > 업데이트 할 회원정보를 props로 넘어온 setUpdateUserData함수에 인자로 넣고서
> > 
> > 회원정보 로그인 인증을 위해 로그인 모달창을 true로 설정한다.
> > ```javascript
> > // SignUp.js
> > function SignUp(props){
> >     ...
> >     useEffect(()=>{
> >         if(props.referrer ==="edit_profile"){
> >         // referrer 검사 후 로컬스토리지에서 유저정보를 들고와서 각 state에 저장한다.
> >             if(!JSON.parse(localStorage.getItem("token")) || !JSON.parse(localStorage.getItem("token")).id){
> >                 return;
> >             }
> >             const userData = JSON.parse(localStorage.getItem("token"));
> >             if(userData.isPro){
> >                 setMajorInput(true);
> >                 setMajor(userData.major);
> >             }
> >             setId(userData.id);
> >             setNick(userData.nick);
> >             setEmail(userData.email);
> >         }
> >     },[props.referrer])
> >     ...
> >     // referrer===edit_profile의 경우의 submit 핸들러 함수
> >     // 전역 state로서 updateUserData에 업데이트 할 유저정보를 입력시키고 
> >     // 로그인 인증용 모달창을 on 한다.
> >     const onUpdateHandler = (e) =>{
> >         if(!isDupliCheck){
> >             alert("아이디 중복확인 해 주세요");
> >             e.preventDefault();
> >             return;
> >         }else{
> >             let updateData ={
> >                 id : id,
> >                 email : email,
> >                 pw : pw,
> >                 nick : nick,
> >                 isPro : majorInput,
> >                 major : major
> >             }
> >             e.preventDefault();
> >             props.forSetModal(true);
> >             props.setUpdateUserData(updateData);
> >         }
> >     }
> >     ...
> >     // 렌더링 부분은 referrer 값에 따라 두 가지 css로 나눈다.
> >     return(
> >         <div className={props.referrer ==="edit_profile" ? "signUpdateArea" : "modal"}>
> >             {props.referrer === "edit_profile" ? <div className="signUpdateTitle">회원정보 수정</div> : null}
> >             <div className={props.referrer ==="edit_profile" ? "signUpdateFrame" : "SignUpModal"}>
> >                 {props.referrer === "edit_profile" ? null : <span className="close" onClick={()=>{props.forClose(false)}}>x</span>}
> >                 <div className={props.referrer ==="edit_profile" ? "signUpdateContents" : "modalContents"}>
> >                     <form onSubmit={ props.referrer==="edit_profile" ? onUpdateHandler : onSubmitHandler}>
> >     ...
> >     )
> >}
> > ```

> > **SignIn 컴포넌트의 확장**
> >
> > 기존의 ```<SignIn/>``` 컴포넌트로서의 렌더링 부는 그대로 두고
> >
> > ```<form/>```태그의 메서드만 referrer에 따라 나뉘게 확장시킨다.
> > 
> > 또한 referrer이 edit_profile의 경우 로컬스토리지에 있는 유저 id 값을 받아서
> > 
> > 로그인 인증시 id입력란에 미리 적혀있도록 setId함수를 사용한다.
> > 
> > 추가적으로 각각의 본인 글에 대해 바로 수정할 수 있도록 버튼을 추가하며, 
> > 
> > 글이 없을 경우 예외 렌더링을 referrer 별로 나눈다.
> > 
> > 특히 ```axiosSignInData()```함수의 경우 인자값에 두번째로 auth를 넣어서 
> >
> > 로그인 인증용 모달창에서 사용 할 경우 페이지 리로딩을 하지 않도록 구분한다.
> > ```javascript
> > function SignIn(props){
> >     ...
> >     useEffect(()=>{
> >         // referrer검사후 edit_profile의 경우 인증용 oldId값 꺼내서 미리 입력
> >         if(props.referrer==="edit_profile"){
> >             setId(JSON.parse(localStorage.getItem("token")).id);
> >         }
> >     })
> >     ...
> >     const onAuthSubmitHandler = (e) =>{
> >         let loginData = {
> >             id : id, // 여기서 사용되는건 기존의 id값 즉 oldID
> >             pw : pw
> >         }
> >         e.preventDefault();
> >         // 두번째 인자에 true값을 보내어 로그인 인증용이라는것을 명시
> >         const result = axiosSignInData(loginData,true);
> >         result.then(data => {
> >             // 인증에 성공 한 경우 모달창을 닫고 곧 바로 업데이트 진행
> >             if(data.loginSuccess){
> >                 alert("인증되었습니다.");
> >                 props.forClose(false);
> >               // 여기서 중요한 점은 올드(기존의) 아이디 값을 함께 보내야만 한다는 점
> >                 let updateData = {
> >                     oldId : JSON.parse(localStorage.getItem("token")).id,
> >                     ...props.userData
> >                 }
> >                 const res = axiosSignUpdate(updateData);
> >                 res.then(data=>{
> >                     alert(data.msg);
> >                     // 또한 정보 수정시 새로 로그인 하도록 로그아웃 유도
> >                     localStorage.removeItem("token");
> >                     alert("다시 로그인 해 주세요.");
> >                     navigate('/');
> >                     window.location.reload();
> >                     props.forClose(true);
> >                 })
> >             }else{
> >                 alert("비밀번호를 확인 해 주세요");
> >             }
> >         }) 
> >     }
> > }
> > ```
> > ```javascript
> > // in axiosSignInData.js
> > // 파라미터로 auth 추가 및 auth값에 따라 리로딩 여부 결정
> > function axiosSignInData(data, auth){
> >     ...
> >     request.then(token => {
> >         if(token.loginSuccess){
> >             localStorage.setItem("token",JSON.stringify({
> >                 value:token.cookie, 
> >                 nick:token.nick, 
> >                 id:data.id,
> >                 isPro : token.isPro,
> >                 major : token.major,
> >                 email : token.email
> >             }))
> >             if(!auth){
> >                 window.location.reload();
> >             }
> >         }
> >     })
> >     return request;
> > }
> > ```
  
<br>

> ### 버그 고치기 & if문 최적화
> 
> #### Write 컴포넌트 권한 예외처리 및 if문 최적화
> 
> 기존의 ```<Write/>``` 컴포넌트에서 ```location.state```로 넘기는 값이 없는 경우에
> 
> 페이지 에러가 발생하던 것을 권한 예외처리에 추가하여 해결 하였고,
> 
> 기존의 권한 예외처리 if문이 복잡한 것을 한 줄로 해결
>
> 또한 ```location.state```로 넘겨 받은 데이터들은 useState로 관리
>
> ```javascript
> const Write = (props)=>{
>   ...
>  useEffect(()=>{
>        ...
>        if(!JSON.parse(localStorage.getItem("token"))||(!location.state.isCurrentUserAdmin && searchParams.get("board")==="notice") || !location.state){
>             alert("권한이 없습니다.");
>             return navigate(-1);
>         }
>         const locData = location.state.data;
>         const isCurUserAdmin = location.state.isCurrentUserAdmin;
>         setAllData(locData);
>         setIsAdmin(isCurUserAdmin);
>         ...
>  })
>  ...
> }
> ```
> #### axiosSearch 컴포넌트에서 검색어 길이 예외처리 중 페이지에러 발생
>
> 기존의 검색기능은 검색어 길이가 2미만이면서 해쉬태그또한 없다면 발생시키는 예외인데
>
> 이 때 리턴값이 false로 남아있는 부분을 처리하지 못해 페이지 에러가 발생함
> 
> 따라서 리턴값 자체를 없애버리고 ```<Search/>```컴포넌트에서 리턴값이 없다면
> 
> 검색결과 글 데이터를 저장하지 않도록 리턴 시켰다.
>
>```javascript
>   const axiosSearch = (inputValue, hashTags=[]) =>{
>     if(inputValue.length<2 && hashTags.length<1){
>         alert("검색어 길이는 최소 2글자 이상이어야 합니다.");
>         // 아무것도 리턴하지 않음
>         return;
>     }
>     ...
>```
> ```javascript
>   const Search = () =>{
>     ...
>     useEffect(()=>{
>         ...
>         const res = axiosSearch(q, tags);
>         // 응답 데이터자체가 비어있다면 useEffect 진행을 멈춤
>         if(!res){
>             return;
>         }
>```

<br/>

후기

컴포넌트를 거의 무조건적으로 재사용하고 확장이 가능하도록 구현해야 한다는 생각에
 
너무 맹목적으로 확장시킨 탓에 뭔가 몇몇 컴포넌트들의 사이즈가 너무 커졌다는 생각이든다.
  
다음 프로젝트를 진행할 때에는 너무 재사용과 확장에 목매어 있지 말아야겠다.
