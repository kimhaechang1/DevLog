# Dec. 28. 2022 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Dec. 31. 2022

## 개발 내용
>
> ### 댓글 기능 구현
>
> #### 댓글 설계
>
> 댓글 기능은 크게 쓰기, 수정, 삭제로 구현 할 것이다.
>
> 댓글은 익명댓글과 유저댓글로 나뉘며, 유저댓글인 경우는 닉네임과 아이디값을 공개한다.
>
> 단, 아이디값은 전체를 공개하지 않는다.
>
> 익명댓글은 수정 및 삭제가 불가능하다.
>
> 본 프로젝트에서 사용한 컴포넌트로 구분하면
>
> 댓글쓰기는 ```<QnAView/>```에서, 댓글 수정/삭제는 ```<Comment/>```에서 구현한다.
>
> ex) 유저 댓글
>```
> 테스트유저(te**)
> 댓글 내용
>```
>
> ex) 익명 댓글
>```
> 익명
> 댓글 내용 
>```
> > **댓글 쓰기**
> >
> > 댓글 쓰기는 크게 로그인 유저와 비 로그인 유저로 나뉘며
> >
> > 댓글 종류로 익명댓글과 유저댓글로 나뉜다.
> > 
> > if(로그인 유저)
> > 
> > - 댓글 작성 시 체크박스(```<input type="checkbox"/>```)를 통해 익명여부 선택가능
> > - 서버로 넘어가는 데이터에 사용자 데이터가 함께 넘어감
> >
> > else
> >
> > - 내부적으로 익명 댓글인 상태로 작성하게됨
> > - 서버로 넘어가는 데이터에 사용자 데이터는 안 넘어감
> >
> > 로그인/비로그인 여부는 페이지 로드 후 곧 바로 체크한다.
> > 
> 
> > **댓글 수정/삭제**
> >
> > 댓글 수정/삭제는 <Comment/> 컴포넌트에서 이루어지며
> > 
> > 기본적으로 댓글 데이터를 ```<QnAView/>```가 로드 된 후 바로 받아오고
> > 
> > ```<QnAView/>```컴포넌트에서 이미 인증된 데이터(```isCurrentUserAdmin``` 등등)가 있으므로
> >
> > ```props```를 활용하여 댓글데이터 및 인증받은 데이터를 넘긴다.
> >
> > 유저가 수정 가능한 댓글인지 아닌지는 수정/삭제 버튼이 보이냐/안보이냐 로 나눈다.
> > 
> > ```<Comment/>```컴포넌트는 두 가지 모드가 있으며
> >
> > 뷰 모드와 수정 모드에 따라 토글방식으로 달라지게 된다.
> >
> > if(mode === view)
> > 
> > 뷰 모드에서는 현재 해당 게시글에 달려있는 댓글의 데이터를 볼 수 있으며
> > 
> > 현재 로그인된 사용자라면, 본인 댓글에 대한 수정/삭제버튼을 통해 수정모드 토글 및 삭제가 가능하다.
> >
> > else if(mode === edit)
> > 
> > 수정 모드에서는 수정 가능한 댓글에 대하여 내용을 ```textarea```를 통해 수정 가능하도록 담고
> > 
> > 사용자가 수정된 내용을 서버에 보낼 수 있도록 제출 버튼을 활성화 시킨다.
> >
> > 댓글삭제의 경우에는 어드민이 아닌경우엔 추가 검증을 하여(본인 댓글인지)
> > 
> > 서버에 댓글 인덱스번호를 보내어 삭제한다.
> >
>
> #### 댓글 구현
>
> 댓글 구현은 1. 댓글 수정/삭제 2. 댓글 쓰기 순으로 구현한다.
>
> > **댓글 수정/삭제**
> > 
> > ```<Comment/>``` 컴포넌트를 먼저 만들어 두고 (그냥 함수 틀만)
> > 
> > ```<QnAView/>```에서 map을 통해 위의 설계 대로 ```<Comment/>```렌더링 하도록 구현한다.
> >
> > ```javascript
> > return (
> > ...
> > {content.comment ? 
> >  <div className="commentArea">
> >   { (content.comment).map((value) => {return <Comment commentData={value} admin={isCurrentUserAdmin} />}) }
> >  </div> 
> > : null}
> > )
> > ```
> > 
> > ```<Comment/>```컴포넌트에서는 모드, 댓글내용, 댓글 전체데이터, 
> > 
> > 어드민여부 그리고 수정/삭제버튼 활성화 여부 를 저장하기 위해 ```useState```를 사용한다.
> >
> > ```javascript
> > const Comment = (props) =>{
> >   const [mode, setMode]= useState("view");
> >   const [content, setContent] = useState("");
> >   const [commentData, setCommentData] = useState({});
> >   const [isAdmin, setIsAdmin] = useState(false);
> >   const [editButtonToggle, setEditButtonToggle] = useState(false);
> > }
> > ```
> > 
> > 위의 설계대로 구현하려면 
> > 
> > 첫 렌더링 이후 ```props```로 넘어온 값을 통해 ```state``` 초기값을 ```set``` 해주어야 한다.
> >
> > 여기서 중요 한 점은 현재 유저의 어드민 여부, 익명 댓글인지 
> >
> > 그리고 본인댓글인지에 따라 수정/삭제 버튼 활성화 여부를 잘 구분지어야 한다.
> > ```javascript
> > const Comment = (props) =>{
> > ...
> >   useEffect(()=>{
> >         setCommentData(props.commentData);
> >         setIsAdmin(props.admin);
> >         setContent(props.commentData.commentDetail);
> >         if(props.admin){
> >             setEditButtonToggle(true); // 어드민이면 익명 댓글에 상관없이 수정/삭제 가능
> >         }else if(!props.commentData.isAnony){ // 어드민이 아니면 익명 댓글에 수정/삭제 불가능
> >             if(JSON.parse(localStorage.getItem("token")) && JSON.parse(localStorage.getItem("token")).id === props.commentData.commentId){
> >             // 현재 로그인 된 유저이면서 댓글 쓴 사람이면 수정/삭제 가능
> >                 setEditButtonToggle(true);
> >             }else{
> >                 setEditButtonToggle(false);
> >             }
> >         }else{
> >          // 비 로그인유저라면 수정/삭제 불가능
> >             setEditButtonToggle(false);
> >         }
> >   },[props]) // useEffect
> > } // const Comment 
> > ```
> > 기본적인 렌더링은 모드에 따라 달라지도록 UI값을 저장할 ```object```타입 변수와
> > 
> > ```return``` 문 내부에서 state에 저장된 댓글 데이터에 따라 달라진다.
> > 
> > 그리고 익명이 아닌 경우 닉네임과 아이디를 노출하는데 
> >
> > 이 때 아이디 길이의 절반을 '*'로 노출시킨다.
> > ```javascript
> > const Comment = (props) =>{
> > ...
> > 
> >    const onToggleHandler = () =>{
> >        if(mode === "view"){
> >           setMode("edit");
> >        }else{
> >           setMode("view");
> >        }
> >    }
> >    const contentUI = {
> >     // 수정모드 : textarea에 데이터를 옮겨서 수정 가능하게 만듦, 뷰 모드 : 단순 글로 수정 불가능
> >        "view" : <div>{content}</div>,
> >        "edit" : <textarea value={content} onChange={(e)=>{ setContent(e.target.value) }}></textarea>
> >    }
> >    const buttonUI = {
> >      // 수정 모드 일 경우 취소를 통해 뷰 모드로 전환가능
> >        "view" : "수정",
> >        "edit" : "취소"
> >    }
> >
> >    return(
> >        <div className="commentBox">
> >            <div className="commentTitle">
> >                // 익명이면 "익명", 유저댓글이면 유저 닉(아이디) 단, 아이디 절반 길이계산 후 *표시
> >                {commentData.isAnony ? "익명" : `${commentData.commentNick}(${commentData.commentId ? 
> >                    commentData.commentId.slice(0,Math.ceil(commentData.commentId.length/2))+
> >                    "*".repeat(commentData.commentId.length - Math.ceil(commentData.commentId.length/2)) 
> >                    : commentData.commentId})`}
> >                {editButtonToggle ? // 수정/삭제버튼 활성화/비활성화
> >                <div className="commentButtons">
> >                    <button onClick={onToggleHandler}>{buttonUI[mode]}</button> 
> >                    <button onClick={onDeleteHandler}>삭제</button>
> >                </div>
> >                 : null}
> >            </div>
> >            <div className="commentMain">
> >             { contentUI[mode] }
> >            </div>
> >            {mode ==="edit" ? <button onClick={onUpdateHandler}>수정</button> : null}
> >        </div>
> >    )
> >}
> > ```
> > 수정의 경우에는 이벤트 핸들러 함수에서 state값을 통해 어드민인지 먼저 검사하고
> > 
> > 어드민이 아닌경우에만 본인 댓글인지 검사한 후 수정된 댓글 데이터와 인덱스번호를
> > 
> > 댓글 수정용 통신 함수 ```axiosCommentUpdate()```로 보내어 서버에 전송한다.
> > ```javascript
> > const Comment = (props) =>{
> > ...
> >     const onUpdateHandler = () =>{
> >         if(!isAdmin){
> >         // 어드민이 아니라면 수정 가능한지, 해당유저가 진짜 로그인상태인지 검사한다.
> >             const userToken = JSON.parse(localStorage.getItem("token"));
> >             const authRes = axiosAuth({
> >                 id : userToken.id,
> >                 index : commentData.commentIndex,
> >                 token : userToken.value,
> >                 cate : "comment",
> >                 isSignin : true,
> >                 isEdit : true
> >             })
> >             authRes.then(data =>{
> >                 if(!data.signinAuth || !data.editAuth){
> >                 // 비 로그인 이거나 본인댓글 아니라면
> >                     alert("권한이 없습니다.");
> >                     return window.location.reload();
> >                 }
> >             })
> >         }
> >         const updateData = {
> >         // 업데이트 된 내용 및 수정된 댓글 인덱스번호
> >             commentIndex : commentData.commentIndex,
> >             commentDetail : content
> >         }
> >         const res = axiosCommentUpdate(updateData);
> >         res.then(data =>{
> >             alert(data.msg);
> >             window.location.reload();
> >         })
> >     }
> > ...
> > }
> > ```
> > ```javascript
> > import axios from "axios";
> > 
> > const axiosCommentUpdate = (post) =>{
> >     const res = axios.post('/comment/update',post)
> >     .then( response => response.data)
> >     .catch(err =>{
> >         return {
> >             msg : `통신 오류(${err.request.status}) 발생하였습니다.`
> >         }
> >     })
> > 
> >     return res;   
> > }
> > 
> > export default axiosCommentUpdate;
> > ```
> > 삭제의 경우에는 인증시스템이 수정의 경우와 비슷하고, 
> >
> > ```axiosCommentDelete()```로 넘기는 데이터가 댓글 인덱스 번호면 된다.
> > ```javascript
> > const Comment = (props) =>{
> > ...
> >     const onDeleteHandler = () =>{
> >        if(window.confirm("삭제 하시겠습니까?")){
> >         // 유저 확인용 안내문구
> >            if(!isAdmin){
> >             // 어드민이 아니라면 수정 가능한지, 해당유저가 진짜 로그인상태인지 검사한다.
> >                const userToken = JSON.parse(localStorage.getItem("token"));
> >                const authRes = axiosAuth({
> >                    id : userToken.id,
> >                    index : commentData.commentIndex,
> >                    token : userToken.value,
> >                    cate : "comment",
> >                    isSignin : true,
> >                    isEdit : true
> >                })
> >                authRes.then(data =>{
> >                    if(!data.signinAuth || !data.editAuth){
> >                   // 비 로그인 이거나 본인댓글 아니라면
> >                        alert("권한이 없습니다.");
> >                        return window.location.reload();
> >                    }
> >                })
> >            }
> >            const deleteData = {
> >                commentIndex : commentData.commentIndex,
> >            }
> >            const res = axiosCommentDelete(deleteData);
> >            res.then(data =>{
> >                alert(data.msg);
> >                window.location.reload();
> >            }) 
> >        } // isAdmin
> >     } // window.comfirm
> >   } // onDeleteHandler
> > } // Comment
> > ```
> > ```javascript
> > import axios from 'axios';
> > 
> > const axiosCommentDelete = (post) =>{
> >     const res = axios.post('/comment/delete',post)
> >     .then( response => response.data)
> >     .catch(err =>{
> >         return {
> >             msg : `통신 오류(${err.request.status}) 발생하였습니다.`
> >         }
> >     })
> > 
> >     return res;    
> > }
> > 
> > export default axiosCommentDelete;
> > ```
> > 
>
> > **댓글 쓰기**
> > 
> > 렌더링 위치는 ```<QnAView/>```에서 게시글 내용 바로 아래에 오도록 구현하고
> > 
> > 위의 설계에서 초기 렌더링 이후 로그인/비로그인에 따라 유저에게 제공되는 UI가 달라져야 한다.
> >
> > ```javascript
> > const QnAView = () =>{
> >     ...
> >     // 해당 게시글에 쓸 댓글 내용 값
> >     const [userCommentData, setUserCommentData] = useState(""); 
> >     // 익명 체크박스 상태값과 익명댓글이냐 아니냐 : 기본적으로 비로그인의 경우를 기준으로 삼는다.
> >     const [isInputAnonyActive, setIsInputAnonyActive] = useState(false); 
> >     const [isAnony, setIsAnony] = useState(1);
> >     ...
> >     useEffect(()=>{
> >       ...
> >       if(JSON.parse(localStorage.getItem("token"))){
> >           const userToken = JSON.parse(localStorage.getItem("token"));
> >           const resOfAuth = axiosAuth({
> >               id : userToken.id,
> >               token : userToken.value,
> >               isSignin : true,
> >               isAdmin : true
> >           })
> >           resOfAuth.then(data =>{
> >             if(data.adminAuth && data.signinAuth){
> >               ...
> >             }else if(data.signinAuth){
> >             // 로그인 만 되어있다면 익명 댓글 체크박스를 사용 가능케 하고 초기값을 false
> >               setIsAnony(false);
> >               setIsInputAnonyActive(true);
> >             }else{
> >             // 그게 아니라면 체크박스 비활성화 및 기본 익명여부 true
> >               setIsAnony(true);
> >               setIsInputAnonyActive(false);
> >             }
> >           })
> >       }
> >     },[])
> >   ...
> > return (
> >   ...
> >     // 아래의 isInputAnonyActive, isAnony는 state값 확인용
> >     <div>isInputAnonyActive : {isInputAnonyActive ? "true" : "false"}</div>
> >       <div>isAnony : {isAnony ? "true" : "false"}</div>
> >         <div className = "commentWriteArea">
> >            <div className ="commentWriteFrame">
> >               <div className="commentWriteHeader">
> >                     {isInputAnonyActive ? <div><input type="checkbox" value={isAnony} onChange={(e)=>{
> >                     // 익명 댓글이라면 익명 체크박스가 필요 없음
> >                         if(isAnony){
> >                           setIsAnony(false);  
> >                         }else{
> >                           setIsAnony(true);
> >                         }
> >                     }}></input>익명</div> 
> >                    : <div>익명</div>}
> >               </div>
> >               <div className="commentWriteContent">
> >                     <textarea rows="2" value={userCommentData} onChange={(e)=>{setUserCommentData(e.target.value)}}></textarea>
> >                     <button onClick={onCommentSubmitHandler}>제출</button>
> >               </div> 
> >            </div> // <div className ="commentWriteFrame">
> >         </div> // <div className = "commentWriteArea">
> >     ...
> > )
> > ``` 
> > 
> > 제출시에는 익명 댓글인 경우 삭제/수정이 불가하다는 안내문구를 사용자에게 보여주며
> > 
> > 어드민이 아닐경우에는 newComment라는 ```object```타입 변수에
> >
> > 현재 게시글 인덱스값, 코멘트 내용, 익명여부, 댓글 작성자 아이디 그리고 닉네임을 담는다.
> >
> > 단, 작성자 아이디와 닉네임은 빈 문자열로 비워두고 아래의 분기문에 따라 달라진다.
> > 
> > if(로그인 된 유저)
> > 
> > 로컬스토리지에 있는 유저 데이터를 활용하여 newComment에 저장
> >
> > else
> > 
> > 빈 문자열 그대로
> > 
> > 마지막으로 ```axiosComment()```함수를 통해 서버에 전송된다.
> > ```javascript
> > const QnAView = () =>{
> >   ...
> >   const onCommentSubmitHandler = () =>{
> >     if(isAnony){
> >       if(!window.confirm("익명 댓글은 수정/삭제 할 수 없습니다.\n 제출 하시겠습니까?")){
> >         return;
> >       }
> >     }
> >     const newComment = {
> >       id : content.id,
> >       commentDetail : userCommentData,
> >       isAnony : isAnony,
> >       commentNick : "",
> >       commentId : ""
> >     }
> >     // 전송 할 댓글 데이터에는 익명여부 상관없이 로그인 되어있으면 commentNick과 commentId에 값이 저장됨
> >     if(JSON.parse(localStorage.getItem("token"))){
> >       const userToken = JSON.parse(localStorage.getItem("token"));
> >       newComment['commentNick'] = userToken.nick;
> >       newComment['commentId'] = userToken.id;
> >     }
> >     const res =  axiosComment(newComment);
> >       res.then(data =>{
> >         alert(data.msg);
> >         window.location.reload();
> >     })
> >   } // onCommentSubmitHandler
> > } // QnAView
> > ```
> > ```javascript
> > import axios from "axios";
> > const axiosComment = (post) =>{
> >     console.log(post);
> >     const res = axios.post('/comment/new',post)
> >     .then( response => response.data)
> >     .catch(err =>{
> >         return {
> >             msg : `통신 오류(${err.request.status}) 발생하였습니다.`
> >         }
> >     })
> > 
> >     return res;
> > }
> > 
> > export default axiosComment;
> > ```

<br/>

후기

저번에 설계 해둔 인증 쪽 함수에서 cate값 덕분에

댓글 구현 시 인증 문제 때 큰 막힘 없이 진행되었다.

하지만 JSON Web Token을 활용한 인증 시스템을 이번에 다시 공부하게 되었는데

지금같이 단순히 토큰 비교로 구현하고 로컬스토리지에 남겨 놓는 방식은

언제든 공격의 위험이 있고, 현재로서는 무수한 if 분기문을 통해 철저히 변조를 방어 할 수 밖에 없다.

이부분은 공부부족이므로 나중에 리펙토링 하여 해결 할 것이다.
