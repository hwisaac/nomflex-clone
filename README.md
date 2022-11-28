## Routes 만들기

1. src/Routes 폴더에 Home.tsx, Search.tsx, Tv.tsx 에 컴포넌트를 만든다.
2. App.tsx 파일에서 Router > Switch > Route(path) > 컴포넌트
3. Route path 에 해당하는 주소로 갈 때 해당 컴포넌트를 랜더링 해준다.

```javascript
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";

function App() {
  return (
    <Router>
      <Header />
      <Switch>
        <Route path='/tv'>
          <Tv />
        </Route>
        <Route path='/search'>
          <Search />
        </Route>
        <Route path='/'>
          <Home />
        </Route>
      </Switch>
    </Router>
  );
}
```

## 키프레임 애니메이션 만들기

- variant 의 value 값에 배열을 넣어준다.

```javascript
const logoVariants = {
  normal: {
    fillOpacity: 1,
  },
  active: {
    fillOpacity: [0, 1, 0],
    transition: {
      repeat: Infinity, // 무한 반복
    },
  },
};

<Logo
  variants={logoVariants}
  whileHover='active'
  initial='normal'
  xmlns='http://www.w3.org/2000/svg'
  width='1024'
  height='276.742'
  viewBox='0 0 1024 276.742'></Logo>;
```

### route match

- 현재 있는 주소에 대한 정보를 사용할 때 쓴다.
- `useRouteMatch('주소')` 는 주소에 대한 정보를 반환한다. (해당 주소에 없는경우 null 반환)
- `isExact : Boolean` 해당 경로에 있는지 여부 ('/' 은 항상 true)

```javascript
import { Link, useRouteMatch } from "react-router-dom";

const homeMatch = useRouteMatch("/");
const tvMatch = useRouteMatch("/tv");

console.log(homeMatch); // {isExact: true params:{} path:"/" url:"/"}
```

## 서치바가 오른쪽에서 커져나가게 할때 : transform-origin 속성 사용

```javascript
// Header.tsx
import { motion } from "framer-motion";
import { useState } from "react";
import styled from "styled-components";

const Input = styled(motion.input)`
  transform-origin: right center;
  position: absolute;
  left: -150px;
`;
// function Header() {
const [searchOpen, setSearchOpen] = useState(false);
const toggleSearch = () => setSearchOpen((prev) => !prev);

// Header return (
<Search>
  <motion.svg
    onClick={toggleSearch}
    animate={{ x: searchOpen ? -180 : 0 }}
    transition={{ type: "linear" }}
    fill='currentColor'
    viewBox='0 0 20 20'
    xmlns='http://www.w3.org/2000/svg'>
    <path
      fillRule='evenodd'
      d='M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z'
      clipRule='evenodd'></path>
  </motion.svg>
  <Input
    transition={{ type: "linear" }}
    animate={{ scaleX: searchOpen ? 1 : 0 }}
    placeholder='Search for movie or tv show...'
  />
</Search>;
```

## useAnimation() 훅

- 보통은 animate 와 state를 결합하는 게 주를 이루지만,
- 특정 코드를 통해 애니메이션을 실행시키고 싶을 때 motion 에서 제공하는 `useAnimation()` 훅을 사용한다.
- 이를 통해 애니메이션 속성과 커맨드를 코드로부터 만들 수 있다.

1. `const someVariable = useAnimation();` 로 변수를 선언하고
2. 해당 변수를 컴포넌트의 `animate` 프롭에 넘겨줘서 연결시킨다.

```javascript
import { motion, useAnimation } from "framer-motion";

// function Header() {
const inputAnimation = useAnimation();
const navAnimation = useAnimation();
const toggleSearch = () => {
  if (searchOpen) {
    // 열려 있으면 inputAnimation을 애니메이트 해라!
    inputAnimation.start({
      scaleX: 0,
    });
  } else {
    inputAnimation.start({ scaleX: 1 });
  }
  setSearchOpen((prev) => !prev);
};
// return (
<Input
  animate={inputAnimation}
  initial={{ scaleX: 0 }}
  transition={{ type: "linear" }}
  placeholder='Search for movie or tv show...'
/>;
```

## Home screen

### query client 만들기

1. QueryClientProvider 컴포넌트로 감싸준다.
2. QueryClientProvider 에는 queryClient() 로 만든 오브젝트를 프롭으로 넘겨준다.

```javascript
// index.tsx
import { QueryClient, QueryClientProvider } from "react-query";

const client = new QueryClient();

ReactDOM.render(
  <React.StrictMode>
    <RecoilRoot>
      <QueryClientProvider client={client}>
        <ThemeProvider theme={theme}>
          <GlobalStyle />
          <App />
        </ThemeProvider>
      </QueryClientProvider>
    </RecoilRoot>
  </React.StrictMode>,
  document.getElementById("root")
);
```

3. api.ts 파일을 만들어서 관리한다.

```javascript
// api.ts
const API_KEY = "1";
const BASE_PATH = "https://api/3";

export function getMovies() {
  return fetch(`${BASE_PATH}/movie/now_playing?api_key=${API_KEY}`).then(
    (response) => response.json()
  );
}
```

4. useQuery 를 사용해준다.
   - 쿼리에는 key 를 전달해줘야 한다.(문자열, 배열 등), 두번째 인자로 fetcher 를 전달한다.
   - `useQuery(["movies", "nowPlaying], getMovies)` movies 카테고리, nowPlaying 카테고리. (이들은 식별자(id) 용도)

```javascript
// Home.tsx
import { useQuery } from "react-query";
import { getMovies } from "../api";

function Home() {
  const { data, isLoading } = useQuery(["movies", "nowPlaying"], getMovies);
  console.log(data, isLoading); // data 는 영화 데이터 객체이고 isLoading 은 boolean
  return (
    <div style={{ backgroundColor: "whitesmoke", height: "200vh" }}>home</div>
  );
}
export default Home;
```
