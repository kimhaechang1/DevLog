# Nov. 09. 2022 commit

적용 프로젝트 : [bePro](https://github.com/kimhaechang1/bePro)

작성 날짜 : Nov. 28. 2022

## 개발 내용
> ### 검색창 컴포넌트화
>
> 검색창이 다른 페이지에서도 사용 될 수 있으므로, 검색창을 컴포넌트화 시켜 재사용성을 늘린다.   
> 
> 메인페이지로 옮겼던 검색창의 내용을 ```SearchBox.js```로 옮겨서 ```<SearchBox>```가 되도록 컴포넌트 화 시킨다.   
> 
> 그리고 메인페이지에서 ```useState```로 관리하던 input태그의 value 값을 ```props```를 통해 컴포넌트끼리 소통하도록 만든다.
 
<br>

> ### 검색 데이터 통신과 ```useNavigate```활용
> 
> 참고자료
>
> [React Router v6: useNavigate() 파라미터 전달&취득 방법 - useLocation](https://curryyou.tistory.com/477)
> 
> [react-router](https://reactrouter.com/en/main/hooks/use-navigate)
> 
> 이제 저번 회의를 통해 한 문장이 아닌 분리된 두 데이터(태그들, 검색어)를 서버에 전송하는 방법을 확정 지었다.
> 
> 따라서 두가지의 데이터를 
>  
> 이전과 같은 방식으로 ```axiosOOOO.js```를 만들어서   
>
> 그곳에 object 타입으로 데이터를 변형하여 서버에 전송한다.
> 
> 우선엔 서버쪽에서 아직 준비가 안되었기 때문에
> 
> 통신을 하는부분은 주석처리를 해 두기로 한다.
>
> **axiosSearch.js**
>
> 다른 ```axios```함수들과 동일하게 넘겨받은 데이터를 ```object``` 타입으로 묶고
>
> ```POST```방식 통신으로 서버에 전송 한 다음 응답 프로미스 객체를 리턴시키는데,
>
> 이 때 검색어의 길이를 계산하여 한 글자 이하 일 경우에는 검색자체가 안되도록 한다.
>
> **SearchBox.js**
>
> ```<SearchBox>``` 컴포넌트는 ```props```를 통해 넘겨받은 
>
> 해쉬태그 배열과 검색어를 ```axiosSearch()```의 인자로 넘긴다.
>
> 리턴 받아온 프로미스 객체는 다른 경우와 마찬가지로 ```.then```을 통해 분석해 나가는데,
> 
> 여기서 받은 데이터를 검색결과 페이지(/Result)에서 사용해야 한다.
>
> 처음에는 쿼리스트링을 활용해야 하나? 어떻게 해야할까 고민을 많이 하면서
>
> 구글에 검색하다가 위의 참고자료에 있는 홈페이지를 발견하고는
>
> ```useNavigate```가 저런 기능이 있구나 하고 바로 응용했다.
> 
> **useNavigate**
>
> ```useNavigate``` 는 ```react-router-dom```의 ```hook```으로서
>
> 어떠한 조건에 따른 페이지 이동이나 옵션 값을 통해 특정 페이지로 파라미터를 넘길 수 있다.
>
> 주로 사용하는 인자를 살펴보면
>
> 첫 번째 인자는 ```to``` 로서 ```react-router-dom```에서 
>
> 쓰이는 ```<Link>``` 태그의 속성 ```to```와 역할이 동일하다.
> 
> 두 번째 인자는 ```option``` 값으로서 ```replace```와 ```state```를 주로 사용하는데
>
> ```replace```는 소위말해 뒤로가기 를 허용할지 말지 로서
>
> true 면 뒤로가기를 막게 된다. 기본값은 false이다.
> 
> ```state```는 첫 번째 인자로 준 링크에다가 어떤 값을 넘기고 싶을 때 ```object```타입으로 넘긴다.
> 
> 값을 받을 때는 받는 페이지에서 ```useLocation```훅을 사용하여 받는다.
>
> ```jsx
> import { useLocation } from 'react-router-dom';
>
> const location = useLocation();
> const contentData = location.state.content // 만약에 state의 key값을 content로 해놓았을 경우
> ```
>
> 따라서 응답 프로미스 객체를 그대로 content라는 key 값으로 useNavigation을 통해 넘겨준다.
>
> 그러면 앞으로 검색결과 페이지에서 넘겨받은 데이터를 어떻게 나타낼지 고민하면 된다.

<br>

후기

검색창을 만들어 갈 때 처음에 설계를 한 방식부터가 잘못되었다는 생각이 들었다.

그도 그럴게 지금 역할을 생각 했을 때, 검색창의 데이터를 관리하는 state가 메인페이지에 존재하고 있는점이
  
상당히 불편하다. 하지만 ```<SearchDropDown>```에서도 사용해야 하기 때문에 어쩔수 없이 메인페이지에 있긴하다.
  
이부분은 다시한번 생각하면서 나중에 리펙토링을 진행해야겠다.
 
또한 리액트 훅에서도 ```useNavigate```를 너무 단순한 기능으로서 생각했다.
 
파라미터 취득도 할 수 있는지는 몰랐는데, 너무 좋은 기능 같다.
  
그리고 ```react-router```에서 너무 설명이 짧은 것 같아서 인터넷의 글을 좀 읽어서 이해했는데
  
이해를 잘 하고 알맞게 쓰고 있는지는 모르겠다.
  
