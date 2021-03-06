<h1>Styling in React</h1>

* React의 컴포넌트에 스타일을 정의하는 방법은 여러 가지가 있다.
  
1. css 파일 생성 후 import 해서 사용하기
  * 이 방법의 문제점은 컴포넌트와 CSS가 분리되어 있다는 점이다.   
    하나는 JS 파일, 하나는 CSS 파일로 분리되게 된다.

  * 아예 특정 컴포넌트를 위한 폴더를 만들고, 그 내에 컴포넌트 파일과   
    CSS 파일을 넣는 방법도 있다. (index.js도 추가 필요)

  * 이 방법의 또다른 문제점은 항상 className을 기억하고, 중복해야 하지 않아야 하며   
    항상 import를 해야 하기 때문이다.

2. 두 번째 방법은 css 파일이 Global(기본)이 아닌 Local이도록 하게 하는 것이다.
  * 이 방법은 `A.css`라는 파일을 `A.module.css`로 네이밍하는 것이다.   
  * 이렇게 한다면 import문도 아래와 같이 바꾸고, className을 JS Object처럼   
    사용할 수 있다.
  ```js
  import React from 'react';
  import styles from "./Header.module.css";

  export default () => (
      <header>
          <ul className={styles.nav}>
              <li>
                  <a href="/">Movies</a>
              </li>
              <li>
                  <a href="/tv">TVs</a>
              </li>
              <li>
                  <a href="/search">Search</a>
              </li>
          </ul>
      </header>
  )
  ```

  * 이렇게 하면, 실제 element는 굉장히 랜덤한 class를 갖게 된다.   
    나의 경우는 아래와 같은 class명을 갖게 되었다.
  ```html
  <ul class="Header_nav__3Xjbe"><li><a href="/">Movies</a></li><li><a href="/tv">TVs</a></li><li><a href="/search">Search</a></li></ul>
  ```

  * 이 방법의 이점은 navList라는 클래스명을 다른 파일에서도 반복해서 사용할 수   
    있게 되었기 때문이다. 즉, navList는 local 클래스명이 된 것이다.   
    하지만 여전히 className을 기억해야 한다는 문제점이 있다.

3. Styled Components
  * Styled-components 라이브러리를 활용하면, style이 내부적으로 정의된   
    컴포넌트를 생성할 수 있다.

  * `yarn add styled-components`

  * 예시는 아래와 같다.
  ```js
  import React from 'react';
  import styled from 'styled-components';

  const List = styled.ul`
      display: flex;
      &:hover {
          background-color: blue;
      }
  `;

  export default () => (
      <header>
          <List>
              <ul>
                  <li>
                      <a href="/">Movies</a>
                  </li>
                  <li>
                      <a href="/tv">TVs</a>
                  </li>
                  <li>
                      <a href="/search">Search</a>
                  </li>
              </ul>
          </List>
        </header>
  )
  ```

  * 즉, `styled.요소`를 const 변수에 할당한 후, 그 변수로 스타일을 적용할   
    컴포넌트를 감싸주면 된다.

<h2>Styled Components</h2>

* 위에서 만든 Styled-component는 local하다.   
  하지만 Styled-component는 전역(Global)으로 만들 수도 있다.   
  전역 style component를 만드는 이유는 예를 들어 해당 web application의   
  폰트와 같은 스타일은 어디서든 동일하기 때문이다.

* 전역 styled components를 사용하기 위해서는 아래의 패키지를 설치해준다.
* `yarn add styled-reset`
* styled-reset의 reset은 CSS를 초기화한다.

* 우선 전역 스타일을 지정하기 위한 컴포넌트를 만든다.
```js
import {createGlobalStyle} from 'styled-components';
import reset from 'styled-reset';

const globalStyles = createGlobalStyle`
    ${reset};
    a {
        text-decoration: none;
        color: inherit;
    }
    * {
        box-sizing: border-box;
    }
    body {
        font-family: 'Courier New', Courier, monospace;
        font-size: 14px;
        background-color: rgba(20, 20, 20, 1);
    }
`;

export default globalStyles;
```

* 위 코드의 `${reset}`이 바로 전역 스타일을 초기화하는 것이다.

* 위의 컴포넌트 사용 예시는 아래와 같다.
```js
import React, {Component} from 'react';
import Router from './Router';
import Header from "./Header";
import GlobalStyles from './GlobalStyles';

class App extends Component {

  render() {
    return(
      <>
        <GlobalStyles/>
        <Header/>
        <Router/>
      </>
    )
  }
}


export default App;
```
<hr/>

<h2>특정 위치에 스타일링 주기</h2>

* Styled component에는 props를 줄 수도 있다.   
  아래는 예시이다.
```js
const Item = styled.li`
    width: 80px;
    height: 50px;
    text-align: center;
    border-bottom: 5px solid ${props => props.current ? "#3498db" : "transparent"};
`;
```

* react-router-dom에 속해있는 `withRouter`는 다른 컴포넌트를 감싸는 컴포넌트이고,   
  `Router`에 대한 정보를 줄 수 있다.

* 아래는 예시이다.
```js
export default withRouter(() => (
    <Header>
        <List>
            <Item current={false}>
                <SLink to="/">Movies</SLink>
            </Item>
            <Item current={true}>
                <SLink to="/tv">TVs</SLink>
            </Item>
            <Item current={false}>
                <SLink to="/search">Search</SLink>
            </Item>
        </List>
    </Header>
))
```

* 위와 같이 `withRouter()`로 컴포넌트들을 감쌌기 때문에 props에 접근이 가능하다.

* `withRouter`의 props에는 `Router`에서 전달된 history, location, match가 있다.   
  location에는 pathname이라는 속성이 있어서 이를 비교해 특정 경우에만 컴포넌트에   
  속성을 줄 수 있다.

```js
const HeaderComponent = ({location: {pathname}}) => (
    <Header>
        <List>
            <Item current={pathname === "/"}>
                <SLink to="/">Movies</SLink>
            </Item>
            <Item current={pathname === "/tv"}>
                <SLink to="/tv">TVs</SLink>
            </Item>
            <Item current={pathname === "/search"}>
                <SLink to="/search">Search</SLink>
            </Item>
        </List>
    </Header>
);

export default withRouter(HeaderComponent);
```