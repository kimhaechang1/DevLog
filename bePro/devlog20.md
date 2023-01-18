# Jan. 16. 2023 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Jan. 18. 2023

## 개발 내용
>
> ### 페이지네이션 
>
> 참고자료 :
>
> [Array.fill()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)
>
> #### 페이지네이션 설계
> 
> **페이지네이션이 필요한 페이지 선정 및 기능 정의**
>
> 페이지네이션이 필요한 페이지는 
> 
> 검색결과 페이지(```"/search"```), 쓴글관리 페이지(```"/mypage/manage"```), 게시판별 글 목록 페이지(```"/{board}"```)이다.
>
> 기본적으로 쿼리스트링을 통해 소통 할 것이고 (```"?page=숫자"```)
> 
> 각각의 페이지네이션이 필요한 페이지를 첫 진입 했을 때, 
> 
> 그 페이지의 페이지네이션 개수에 따라 쿼리 스트링이 달라지며
> ```javascript
> 두 가지 페이지네이션 메뉴가 필요한 경우 : qna_page, notice_page
> -> navigate(`/{페이지명}?qna_page=1&?notice_page=1`)
> ```
> 쿼리스트링 값에 따라 서버에서 받아온 전체 글 중 페이지번호 별로 
>
> 사용자가 정의한 페이지당 글 개수에 의해 계산된 개수만큼 렌더링한다.
> 
> 여기서 중요한 것으로 내가 만든 페이지네이션 컴포넌트를 사용하기 위해서는
> 
> 페이지당 개수(```limit```)와 페이지 넘버의 최대 개수(```maxPage```), 그리고 현재 페이지 번호(```currentPage```)을 보관해야 한다.
> 
> 이 값들과 함께 페이지네이션 컴포넌트의 props로 사용하는 쿼리스트링 값(```props.query```)과 총 게시물 개수(```props.total```)를 넘겨야 
>
> 페이지네이션 컴포넌트에서 자체적으로 계산하여 현재 선택가능한 페이지번호들을 보여준다.
> 
> **페이지네이션 UI설계**
>
> UI 디자인으로는 Begin, prev, next, End, numbers 버튼이 존재한다.
>
> Begin : 시작 페이지로 이동(```"?page=1"```), prev : 이전 페이지로 이동(```current_page_num-=1```)
> 
> End : 끝 페이지로 이동(```"?page=max_page_num"```), next : 이후 페이지로 이동(```current_page_num-=1```)
>
> numbers : 각 숫자 버튼으로서, 각 숫자별 페이지이동을 담당한다.
> 

<br>

> 
> #### 페이지네이션 구현
> 
> **페이지네이션이 필요한 페이지 공통요소 추가**
> 
> 페이지네이션이 필요한 페이지들을 이동할 수 있는 버튼에 대하여
> 
> 심어져 있는 navigate함수 첫번째 인자 주소 쿼리스트링에 사용할 쿼리값을 넣는다.
>
> ```javascript
> // qna 게시판 글 목록, 쓴글 관리 페이지 in Header.js
>  const UI = {
>         // 쓴글관리 페이지의 경우 어드민의 한해서만 공지사항 쓴글 목록을 보여주어야 한다.
>        loginSuccess1 : <Link to={isCurrentUserAdmin ? `/mypage/manage?qna_page=1&notice_page=1` : '/mypage/manage?qna_page=1'}><li>{nick}</li></Link> // 변경점
>  }
>  <Link to="/qna?page=1"><div>글목록</div> // 변경점
>  
> // 제목으로 넘어갈 수 있는 게시판별 글 목록 in Card.js
>
> const onLinkHandler = (e) =>{
>      if(e.target.innerHTML !== "조회수 높은 순"){
>        navigate(`/${boardType}?page=1`); // 변경점
>      }
>    }
>
> // 검색결과 목록 in SearchBox.js
>
> const onClickHandler = () =>{
>   navigate(`/search?q=${value}&tags=${hashTag.join(',')}&qna_page=1&notice_page=1`); // 변경점
> }
> // 쓴글 관리 페이지 in SideMenu.js
> // 변경점으로 기본적으로 QnA 쓴 글 목록을 보여주며, 어드민의 경우 공지사항 쓴 글 목록도 확인해야 함
> navigate(`/mypage/${data.menu_value}${data.menu_value==="manage" ? 
>  "?qna_page=1":''}${props.isCurrentUserAdmin && data.menu_value==="manage" ? 
>  "&notice_page=1" : '' }`);
> ```
> 이 다음으로는 페이지네이션 컴포넌트가 필요한 props와 글 목록을 자르기 위한 currentPage를 useState로 보관하고
>
> 보여주어야 할 글 데이터들을 알맞게 자르는 로직을 구현한다. 페이지네이션 컴포넌트도 추가한다.
> ```javascript
> // 게시판 글 목록 페이지의 경우는 모두 동일
> const QnABoardList = () =>{
>   ...
>    const [maxPage] = useState(5); // 최대 페이지네이션 번호 개수
>    const [limit] = useState(15);  // 페이지당 글 개수
>    const [currentPage, setCurrentPage] = useState(); // 현재 쿼리스트링 페이지번호
>    const [searchParams] = useSearchParams();
>    useEffect(()=>{
>        setCurrentPage(parseInt(searchParams.get("page")));
>        ...
>    },[searchParams])
>    return(
>        <>
>        <div style={{alignItems:"center",display:"flex",flexDirection:"column"}}>
>            {
>           // 페이지 번호 별로 배열을 잘라서 넘겨줌
>            list.slice(0+((currentPage)-1)*(limit),(limit*currentPage)).map((data, index)=>{
>                return (
>                    <div onClick={()=>{
>                        navigate(`/qna/${data['id']}`);
>                    }} style={{cursor:"pointer"}}>{data['title']}</div>
>                )
>            })
>            }
>        </div>
>        <div className="PageArea">
>            <Page query={"page"} maxPage={maxPage} limit={limit} total={list.length}/>
>        </div>
>        </>
>    )
>}
> // 검색결과 페이지 in Search.js
> const Search = () =>{
>    const [searchParams] = useSearchParams();
>    const [resOfQna, setResOfQna] = useState([]);
>    const [resOfNoti, setResOfNoti] = useState([]);
>    const [pagesNumber, setPagesNumber] = useState({});
>    const [maxPage] = useState(5);
>    const [limit] = useState(10); 
> 
>    const navigate = useNavigate();
>    useEffect(()=>{
>         const qna_page  = parseInt(searchParams.get("qna_page"));
>         const notice_page = parseInt(searchParams.get("notice_page"));
>         setPagesNumber({
>           qna_page : qna_page,
>           notice_page : notice_page
>    });
>    ...
>         return(
>             <div className="cardFrame">
>               <div className="subject">총 {resOfQna.length+resOfNoti.length} 건</div>
>               <div className="cardArea">
>                 <Card name={"QnA 검색결과"} total={resOfQna.length} data={resOfQna.slice(0+((pagesNumber['qna_page']-1)*(limit)), (limit*pagesNumber['qna_page']))} referrer={"search"}/>
>               </div>
>               <div className="PageArea">
>                 <Page query={"qna_page"} referrer={"search"} maxPage={maxPage} limit={limit} total={resOfQna.length} queryNameList={Object.keys(pagesNumber)}/>
>               </div>
>               <div className="cardArea">
>                 <Card name={"공지사항 검색결과"} total={resOfNoti.length} data={resOfNoti.slice(0+((pagesNumber['notice_page']-1)*(limit)), (limit*pagesNumber['notice_page']))} referrer={"search"}/>
>               </div>  
>               <div className="PageArea">
>                 <Page query={"notice_page"} referrer={"search"} maxPage={maxPage} limit={limit} total={resOfNoti.length} queryNameList={Object.keys(pagesNumber)}/>
>               </div>
>             </div>
>           </div>
>         </div>
> // 쓴글 관리 페이지 in Manage.js 
> // 어드민의 경우에 공지사항 쓴 글 확인하기 위해서 쿼리스트링 에 notice_page 값이 추가되어있다.
> const Manage = (props) =>{
>     const [resOfQna, setResOfQna] = useState([]);
>     const [resOfNoti, setResOfNoti] = useState([]);
>     const [maxPage] = useState(5);
>     const [limit] = useState(6);  
>     const [currentPage, setCurrentPage] = useState({});
>     const [searchParams] = useSearchParams();
> 
>     useEffect(()=>{
>         const qna_page  = parseInt(searchParams.get("qna_page"));
>         const notice_page = parseInt(searchParams.get("notice_page"));
>         const pageObj = {
>             qna_page : qna_page,
>         }
>         if(notice_page){
>             pageObj.notice_page = notice_page;
>         }
>         setCurrentPage(pageObj);
>         setCurrentSelected((location.pathname.split("/"))[2]);
>         ...
>         setResOfQna(qna);
>         setResOfNoti(notice);
>         })
>    },[isCurrentUserAdmin, userAuth,searchParams])
>
>    return (
>        <div className="cardFrame">
>            <div className="cardArea">
>                <Card name={"내가 쓴 QnA"} referrer={"mypage"} data={resOfQna.slice(0+((currentPage['qna_page']-1)*(limit)), (limit*currentPage['qna_page']))} isCurrentUserAdmin={isCurrentUserAdmin}/>
>            </div>
>            <div className="PageArea">
>                <Page query={"qna_page"} maxPage={maxPage} limit={limit} total={resOfQna.length}/>
>            </div>
>            { isCurrentUserAdmin ? 
>            <>
>            <div className="cardArea">
>                <Card name={"내가 쓴 공지사항"} referrer={"mypage"} data={resOfNoti.slice(0+((currentPage['notice_page']-1)*(limit)), (limit*currentPage['notice_page']))} isCurrentUserAdmin={isCurrentUserAdmin}/>
>            </div>
>            <div className="PageArea">
>                <Page query={"notice_page"} maxPage={maxPage} limit={limit} total={resOfNoti.length}/>
>            </div>
>            </>  : null}
>        </div>
>    )
>}
> ``` 
>
> **페이지네이션 컴포넌트 구현**
>
> props로 넘어온 자료를 토대로 useEffect속에서 연산하고
> 
> 페이지 UI 설계 한 것에 기능을 부여하여 구현한다.
> 
> ```javascript
> 
> const Page = (props) =>{
>
>    const [currentMaxPage, setCurrentMaxPage] = useState(); // 현재 번호들 중 최대값
>    const [currentNumberOfPage, setCurrentNumberOfPage] = useState(); // 현재 나타내어진 번호 개수
>    const [searchParams,setSearchParams] = useSearchParams(); // url쿼리스트링 들고오기
>    const [totalP, setTotalP] = useState(); // 총 페이지 수
>    const [startNumber, setStartNumber] = useState(1); // 현재 시작번호
>
>    useEffect(()=>{
>       // 렌더링에 쓰일 useState값 초기화
>        let endText = 0; // 총 게시물 수를 페이지당 게시글 개수로 나누고 남은 페이지들
>        let totalPage = 1; // 총 페이지 수
>        let current_max_page = 0; // 현재 번호들 중 최대값
>        const current_page_num = parseInt(searchParams.get(`${props.query}`)); // 현재 쿼리스트링 페이지변호
>
>       // 페이지 목록은 항상 startNumber를 기준으로 생성되기 때문에 
>       // (startNumber=1,props.maxPage=5 : |1|2|3|4|5|)
>       // 현재 쿼리스트링 번호가 처음 페이지 최대 숫자를 넘어섰을 때 부터 startNumber를 항상 계산함
>        if(current_page_num > props.maxPage){
>            let newStartPage = 1 + (props.maxPage*Math.ceil(current_page_num/props.maxPage-1));
>            setStartNumber(newStartPage);
>        }
>         // maxPage가 5라면 6번 페이지넘버가 필요한가? 를 검사함
>        if(props.limit<props.total){
>            totalPage = Math.ceil(props.total/props.limit); 
>            endText = props.total % props.limit; 
>        }
>       // 현재 startNumber 기준으로 아직 남은 페이지수가 설정해둔 props.maxPage보다 큳다면
>       // 현재 나타낼 페이지 번호 수는 props.max만큼이다.
>       // 작다면 몇개 남았는지 연산해야한다.
>        if(totalPage - startNumber + 1 >= props.maxPage){
>            current_max_page = startNumber + props.maxPage - 1;
>            setCurrentNumberOfPage(props.maxPage);
>
>        }else{
>            current_max_page = startNumber + totalPage % props.maxPage -1;
>            setCurrentNumberOfPage(totalPage % props.maxPage);
>        }
>
>        setTotalP(totalPage);
>        setCurrentMaxPage(current_max_page);
>
>    },[props, searchParams, startNumber])
>    // prev 와 next는 항상 페이지 쿼리스트링을 건들기 전 넘어가도 되는 페이지번호인지 검사해야한다.
>    // 만약에 현재 설정된 startNumber보다 작아지게 된다면 새로운 페이지 번호들을 렌더링 해야하므로 
>    // startNumber를 재설정 한다.
>    const onPrevClickHandler = () =>{
>        if(parseInt(searchParams.get(`${props.query}`))-1>=1){
>            if(parseInt(searchParams.get(`${props.query}`))-1 < startNumber){
>                setStartNumber(startNumber-props.maxPage);
>            }
>            const queryValue = parseInt(searchParams.get(props.query));
>            searchParams.set(props.query, queryValue-1);
>            setSearchParams(searchParams);
>        }
>    }
>
>    const onNextClickHandler = () =>{
>        if(parseInt(searchParams.get(`${props.query}`))+1<=totalP){
>            if(parseInt(searchParams.get(`${props.query}`))+1 > currentMaxPage){
>                setStartNumber(startNumber+props.maxPage);
>            }
>            const queryValue = parseInt(searchParams.get(props.query));
>            searchParams.set(props.query, queryValue+1);
>            setSearchParams(searchParams);
>        }
>
>    }
>   // Begin과 End의 경우에는 알맞은 쿼리값(props.query)에 알맞은 페이지넘버만 넘기면 된다.
>    const onBeginClickHandler = () =>{
>        searchParams.set(props.query, 1);
>        setSearchParams(searchParams);
>        setStartNumber(1);
>    }
>
>    const onEndClickHandler = () =>{
>        searchParams.set(props.query, totalP);
>        setSearchParams(searchParams);
>        setStartNumber(Math.ceil(totalP/props.maxPage)*props.maxPage-props.maxPage+1);
>
>    }
>
>    return (
>        <div className="PageFrame">
>            <div onClick={onBeginClickHandler} className="onCursor">Begin</div>
>            <div onClick={onPrevClickHandler} className="PagePrev onCursor">Prev</div>
>            <div className="PageList">
>                {Array(currentNumberOfPage).fill(startNumber).map((value, index)=>{
>                    return <div onClick={()=>{
>                        searchParams.set(props.query,value+index);
>                        setSearchParams(searchParams);
>                    }}className={parseInt(searchParams.get(props.query)) === value+index? "PageEle selected onCursor" : `PageEle onCursor`}>{value+index}</div>
>                })}
>            </div>
>            <div onClick={onNextClickHandler} className="PageNext onCursor">Next</div>
>            <div onClick={onEndClickHandler} className="onCursor">End</div>
>        </div>
>    )
>}
> ```

<br>
  
> #### 조그마한 수정사항
>
> **css import를 각자 컴포넌트 및 페이지로 이동시킴**
>
> css 가 import  되어있는 방식이 초창기에는 메인페이지에 다 박아두다가
> 
> 나중에는 또 컴포넌트나 페이지별로 import하고 있기에
>
> 어짜피 각자 컴포넌트나 페이지별로 css가 존재하기에 각각 필요한 css를 가지는 것으로 바꿈
>

<br/>

후기

재사용성 높은 페이지네이션을 구현할 당시에 초창기에는 if문 정리하느라 시간을 정말 많이 쏟은것 같다.

그래도 다 구현하니까 여기저기 쓸 수 있을 것이라고 생각이 든다.

이번 프로젝트를 쭉 진행하면서 너무 시간에 쫒기듯이 한것 같은데

아무래도 프로젝트 팀장으로서 시간 조율이나, 팀 협업을 거의 못한것 같다는 생각이 든다.

다음 프로젝트에서는 체계적으로 관리 툴을 쓰던지, 시간 관련햇는 체계적으로 다뤄야 할 것 같다.
