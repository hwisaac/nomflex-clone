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

## 슬라이더 만들기

- 모션을 이용하면 Box를 모두 랜더링 할 필요 없다. key 값을 바꿔주기만 해도 된다.
- Row 하나를 무한히 이어붙이는 효과를 낼 수 있다.
- AnimatePresence 는 컴포넌트가 render되거나 destroy 될 때 애니메이트 한다.

```javascript
// Home.tsx
import { motion, useAnimation, useViewportScroll } from "framer-motion";
// styled-components
const Slider = styled.div`
  position: relative;
  top: -100px;
`;
const Row = styled(motion.div)`
  display: grid;
  gap: 10px;
  grid-template-columns: repeat(6, 1fr);
  position: absolute;
  width: 100%;
`;

const Box = styled(motion.div)`
  background-color: white;
  height: 200px;
  color: red;
  font-size: 66px;
`;
// variants
const rowVariants = {
  hidden: {
    x: window.outerWidth + 10,
  },
  visible: {
    x: 0,
  },
  exit: {
    x: -window.outerWidth - 10,
  },
};
// function Home() {
// index 키값 컨트롤
const [index, setIndex] = useState(0);
const incraseIndex = () => setIndex((prev) => prev + 1);
// return(
<Slider>
  <AnimatePresence>
    <Row
      variants={rowVariants}
      initial='hidden'
      animate='visible'
      exit='exit'
      transition={{ type: "tween", duration: 1 }}
      key={index}>
      {[1, 2, 3, 4, 5, 6].map((i) => (
        <Box key={i}>{i}</Box>
      ))}
    </Row>
  </AnimatePresence>
</Slider>;
```

#### 이 코드의 버그

- 연속으로 클릭하면 두번째로 나오는 Row 가 exit 되면서 슬라이드 사이의 간격이 멀어진다.
- state 를 하나 만들어주자 : `leaving`
- `leaving`이면 아무것도 작동 안하고, `false` 면 원래대로 작동한다.
- `leaving` 을 다시 false 로 만들기 위해 `AnimatePresence` 에서 `onExitComplete` 프롭을 사용하자
- onExitComplete 에 함수를 넣으면 exit 이 끝났을 때 실행해준다.

```javascript
// Home.tsx
const rowVariants = {
  hidden: {
    x: window.outerWidth + 5,
  },
  visible: {
    x: 0,
  },
  exit: {
    x: -window.outerWidth - 5,
  },
};

// function Home() {
const [leaving, setLeaving] = useState(false);
const incraseIndex = () => {
  if (data) {
    if (leaving) return;
    toggleLeaving();
    const totalMovies = data.results.length - 1;
    const maxIndex = Math.floor(totalMovies / offset) - 1;
    setIndex((prev) => (prev === maxIndex ? 0 : prev + 1));
  }
};
const toggleLeaving = () => setLeaving((prev) => !prev);

// return (
<AnimatePresence onExitComplete={toggleLeaving}>
  <Row
    variants={rowVariants}
    initial='hidden'
    animate='visible'
    exit='exit'
    transition={{ type: "tween", duration: 1 }}
    key={index}>
    {data?.results
      .slice(1)
      .slice(offset * index, offset * index + offset)
      .map((movie) => (
        <Box
          key={movie.id}
          bgPhoto={makeImagePath(movie.backdrop_path, "w500")}
        />
      ))}
  </Row>
</AnimatePresence>;
```

#### 또다른 버그

- TvShows 에 갔다가 Home 으로 돌아오면 오른쪽에서 들어온다.
- 초기 애니메이션 `{hidden = x: window.outerWidth + 5}` 가 적용되기 때문이다.
- `AnimatePresence` 컴포넌트에 `initial ={false}` 를 주자
- 이로인해 컴포넌트가 처음 마운트 될 때 `initial = {hidden}` 히든 애니메이션이 작동되지 않는다.

```javascript
<AnimatePresence initial={false} onExitComplete={toggleLeaving}>
```

#### 영화를 잘라서 보여주기

- 슬라이스로 잘라준다.
- 무한정 index가 증가하지 않게, 영화를 보여줄 수 있는 한계를 보여주면 다시 `page = 0` 으로 만들어준다.

```javascript
const offset = 6;
let page = 0;
const movies = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18];

movies.slice(page * offset, page * offset + offset);
```

```javascript
// function Home() {
const incraseIndex = () => {
  if (data) {
    if (leaving) return;
    toggleLeaving();
    const totalMovies = data.results.length - 1;
    const maxIndex = Math.floor(totalMovies / offset) - 1;
    setIndex((prev) => (prev === maxIndex ? 0 : prev + 1)); // 최대 인덱스면 0으로 초기화
  }
};

// return (
<Row
  variants={rowVariants}
  initial='hidden'
  animate='visible'
  exit='exit'
  transition={{ type: "tween", duration: 1 }}
  key={index}>
  {data?.results
    .slice(1)
    .slice(offset * index, offset * index + offset)
    .map((movie) => (
      <Box
        key={movie.id}
        bgPhoto={makeImagePath(movie.backdrop_path, "w500")}
      />
    ))}
</Row>;
```

#### 배경이미지 가운데로 하기

```css
background-image: url(...);
background-size: cover;
background-position: center center;
```

#### 영화 클릭시 모달 띄우기

1. 박스 클릭시 작동할 함수 `onBoxClicked` 선언

```javascript
// function Home() {
const onBoxClicked = (movieId: number) => {
  // push 를 하면 해당 URL 로 이동
  history.push(`/movies/${movieId}`);
};
```

##### history 객체

- useHistory 훅을 사용하면 URL 을 왔다갔다 할 수 있다.

```javascript
import { useHistory, useRouteMatch } from "react-router-dom";
const history = useHistory();
```

##### useRouteMatch 훅

- 해당 라우트에 있는지 체크할 때 사용한다.

```javascript
// 이 객체의 isExact: (boolean) 으로 체크가능
const bigMovieMatch = useRouteMatch < { movieId: string } > "/movies/:movieId";

// bigMovieMatch 가 null 이 아니면 모달창을 띄운다.
{
  bigMovieMatch ? <motion.div layoutId={bigMovieMatch.params.movieId} /> : null;
}
```

```javascript
// path 를 배열로 만들면 '/' 또는 '/movies/:movieId' 일 때 Home을 랜더링한다.
<Route path={["/", "/movies/:movieId"]}>
  <Home />
</Route>
```

##### 같은 layoutId 로 두 요소를 연결해주면 모션이 애니메이트를 적용해준다.

```javascript
// function Home() {
const onBoxClicked = (movieId: number) => {
  // push 를 하면 해당 URL 로 이동
  history.push(`/movies/${movieId}`);
};

// return (
  <Slider>
    <AnimatePresence initial={false} onExitComplete={toggleLeaving}>
      <Row
        variants={rowVariants}
        initial='hidden'
        animate='visible'
        exit='exit'
        transition={{ type: "tween", duration: 1 }}
        key={index}>
        {data?.results
          .slice(1)
          .slice(offset * index, offset * index + offset)
          .map((movie) => (
            <Box
              layoutId={movie.id + ""}
              key={movie.id}
              whileHover='hover'
              initial='normal'
              variants={boxVariants}
              onClick={() => onBoxClicked(movie.id)}
              transition={{ type: "tween" }}
              bgPhoto={makeImagePath(movie.backdrop_path, "w500")}>
              <Info variants={infoVariants}>
                <h4>{movie.title}</h4>
              </Info>
            </Box>
          ))}
      </Row>
    </AnimatePresence>
  </Slider>
    <AnimatePresence>
      {bigMovieMatch ? (
        <motion.div
          layoutId={bigMovieMatch.params.movieId}
          style={{
            position: "absolute",
            width: "40vw",
            height: "80vh",
            backgroundColor: "red",
            top: 50,
            left: 0,
            right: 0,
            margin: "0 auto",
          }}
        />
      ) : null}
    </AnimatePresence>
```

- 박스가 클릭하면 url 이 바뀌는데 useRouteMatch훅으로 만든 bigMovieMatch 객체의 params.movieId 에 해당 영화의 id 정보가 있다. 이걸 사용한다.
-

#### 모달 바깥 누르면 모달창 없애기

- fragment <></> 로 서로 분리된 컴포넌트를 묶어준다. 만들어준다. (하나는 모달BigMovie이고 하나는 Overlay 로 클릭시 닫힐 영역이다.)
- 오버레이를 클릭하면 홈('/') 으로 돌아간다.

```javascript
{
  // function Home() {
  const onOverlayClick = () => history.push("/");
  // return (
  bigMovieMatch ? (
    <>
      <Overlay
        onClick={onOverlayClick}
        exit={{ opacity: 0 }}
        animate={{ opacity: 1 }}
      />
      <BigMovie
        style={{ top: scrollY.get() + 100 }}
        layoutId={bigMovieMatch.params.movieId}>
        hello
      </BigMovie>
    </>
  ) : null;
}
```

-

#### 스크롤이 어디에 있든 그 화면에 맞게 모달창을 가운데에 나오게 만들기

- `useViewportScroll` 훅을 사용한다.
- `scrollY` 를 받아와서 모달의 위치를 조절한다.

```javascript
import { useViewportScroll } from "framer-motion";

//function Home() {
const { scrollY } = useViewportScroll();

<BigMovie
  style={{ top: scrollY.get() + 100 }}
  layoutId={bigMovieMatch.params.movieId}>
  hello
</BigMovie>;
```
