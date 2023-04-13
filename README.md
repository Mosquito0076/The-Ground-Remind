# README.md

## 0. 시작하기 앞서

- [전체 작업 내용](https://github.com/Mosquito0076/The-Ground)

<br>

 해당 프로젝트는 야구 팬들 중 '내가 감독해도 저것보단 잘 하겠다!'의 '내가' 를 담당하시는 분들을 위해, 직접 드림팀을 만들어 기록을 경쟁할 수 있도록 게임 시뮬레이션을 제공하는 서비스입니다. 팀원간의 역할 분배는 아래와 같습니다.

<br>

김주원 - BE, 선수 목록 상세 조회 및 상세 데이터 처리 담당

이대희 - BE, CI/CD 및 회원 가입(소셜 로그인), 로그인 등 유저 정보 관리 담당

이호승 - BE, Hadoop 분산 처리를 통한 선수 정보 데이터화 및 저장 담당

이정재 - FE, 구단 관리 페이지, 이적 시장 페이지, 선수 데이터 가시화, Url 정리 등 담당

홍성목 - FE, 회원가입(소셜 로그인), 랜딩 페이지, 튜토리얼, 게임 페이지 등 담당

<br>

이 프로젝트에서 저의 역할은 React 기반 Front-end 였습니다다. 이 과정에서 구글, 네이버, 카카오톡을 통한 소셜 로그인을 구현하였으며, proxy를 이용한 CORS 에러 회피 등 기술 할만한 특이사항이 많았기에 이를 기록해 두고자 합니다.

<br>

또한 이 문서의 파일 중 api 폴더는 이정재 팀원이 담당했으며, 그 외의 폴더 및 파일은 직접 작업한 파일입니다.

<br>

<br>

## 1. 소셜 로그인(구글, 네이버, 카카오톡)

 해당 프로젝트에서 제가 구현한 것 중 하나는 **구글, 네이버, 카카오톡**을 이용한 **소셜 로그인**을 입니다. 원래 또다른 FE 멤버가 진행하고 있었지만, 도중에 취업을 하는 바람에 이어받게 되었습니다. 소셜 로그인을 만든 과정과 코드는 아래와 같습니다.

<br>

### 1) Developer Page

- [Google Cloud](https://console.cloud.google.com/)
- [Naver Developers](https://developers.naver.com/main/)
- [Kakao Developers](https://developers.kakao.com/)

소셜로그인을 이용한다는 것은, 각각의 사이트에서 사용자 정보를 불러오거나 확인한다는 뜻이므로 각 사이트에서 허가를 받아야 합니다.

<br>

소셜 로그인의 과정은 크게

1. 해당 사이트 아이디 로그인 후, 지정된 Url로 API Key 등 필요한 정보를 포함하여 API 요청
2. 올바른 요청이었으면 인가 코드를 받을 수 있음
3. 인가 코드를 토대로 다시 API 요청
4. 올바른 요청이었으면 Access 토큰을 받을 수 있음
5. Access 토큰을 토대로 유저 정보 요청
6. Access 토큰 확인 후 올바르다면 유저 정보 반환
7. 받은 유저 정보를 토대로 로그인 또는 회원가입 진행

으로 이루어져 있습니다.

<br>

이 과정에서 **지정된 Url**과 **API Key**를 얻기 위해서는 각각의 사이트에 미리 **내가 소셜 로그인을 사용할 사이트에 대한 정보**를 미리 입력하고, 이를 통한 허가를 얻어야 합니다. 그 사이트들이 위의 사이트들이며, 사용 가이드가 잘 정리되어 있으므로 그를 참고하여 등록하면 큰 문제 없이 통과할 수 있습니다.

![예시](https://user-images.githubusercontent.com/95673624/231124373-92c0accf-cd6e-495d-a042-e487a292122f.png)

(네이버 사용 예시)

이 과정에서 작성한 코드는 아래와 같습니다.

<br>

### 2) 작성 코드

<br>

```js
// ./api/SocialApi.js


// 서버 환경인지 로컬 환경인지 확인
const REDIRECT_URI = window.location.href.includes("localhost")
  ? "http://localhost:3000"
  : "https://j7d109.p.ssafy.io";

// 네이버 관련 정보
const NAVER_CLIENT_ID = "PVGrBZM8vqHq_92Vh6Wx";
const NAVER_CLIENT_SECRET = "tSbysXbRL1";
const NAVER_STATE_STRING = Math.floor(
  Math.random() * (20000000 - 10000000) + 10000000
);
const NAVER_AUTH_URL = `https://nid.naver.com/oauth2.0/authorize?response_type=code&client_id=${NAVER_CLIENT_ID}&state=${NAVER_STATE_STRING}&redirect_uri=${REDIRECT_URI}`;
const NAVER_LOGIN = (CODE, STATE) =>
  `/token/?grant_type=authorization_code&client_id=${NAVER_CLIENT_ID}&client_secret=${NAVER_CLIENT_SECRET}&code=${CODE}&state=${STATE}`;

// 카카오 관련 정보
const KAKAO_REST_API_KEY = "1ae04a78365d2a5f1e2e1d4ee529fe84";
const KAKAO_AUTH_URL = `https://kauth.kakao.com/oauth/authorize?client_id=${KAKAO_REST_API_KEY}&redirect_uri=${REDIRECT_URI}&response_type=code`;
const KAKAO_LOGIN = (CODE) =>
  `https://kauth.kakao.com/oauth/token?grant_type=authorization_code&client_id=${KAKAO_REST_API_KEY}&redirect_uri=${REDIRECT_URI}&code=${CODE}`;

// 구글 관련 정보
const GOOGLE_CLIENT_ID =
  "824400159984-9lg3ubjictcle5lbsbj39s076lko1fhh.apps.googleusercontent.com";
const GOOGLE_CLIENT_SECRET = "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";
const GOOGLE_SCOPE = "https://www.googleapis.com/auth/userinfo.email";
const GOOGLE_AUTH_URL = `https://accounts.google.com/o/oauth2/v2/auth/oauthchooseacount?client_id=${GOOGLE_CLIENT_ID}&redirect_uri=${REDIRECT_URI}&response_type=code&scope=${GOOGLE_SCOPE}`;
const GOOGLE_LOGIN = (CODE) =>
  `https://www.googleapis.com/oauth2/v4/token?code=${CODE}&client_id=${GOOGLE_CLIENT_ID}&client_secret=${GOOGLE_CLIENT_SECRET}&grant_type=authorization_code&redirect_uri=${REDIRECT_URI}`;

const SocialApi = {
  naver: {
    auth: NAVER_AUTH_URL,
    login: NAVER_LOGIN,
  },
  kakao: {
    auth: KAKAO_AUTH_URL,
    login: KAKAO_LOGIN,
  },
  google: {
    auth: GOOGLE_AUTH_URL,
    login: GOOGLE_LOGIN,
  },
};
export default SocialApi;
```

미리 Secret Key, Url 등을 저장해둔 뒤, 이를 메서드로 호출하여 사용하기 편하게 되어있습니다.

이를 사용하여 로그인 또는 회원가입을 진행하는 코드는 아래와 같습니다.

<br>

```js
// ./landing/components/Modal

const Modal = (props) => {
  // 로그인 띄우는 모달
  const { closeModal, login } = props;
  const dispatch = useDispatch();

  // 버튼 클릭 시, 로그인 타입을 로컬 스토리지에 저장하고 동의 페이지로 이동
  const kakaoLogin = () => {
    dispatch(userActions.setLoginType("K"));
    window.location.href = SocialApi.kakao.auth;
  };

  const naverLogin = () => {
    dispatch(userActions.setLoginType("N"));
    window.location.href = SocialApi.naver.auth;
  };

  const googleLogin = () => {
    dispatch(userActions.setLoginType("G"));
    window.location.href = SocialApi.google.auth;
  };
```

소셜 로그인 모달에서 각각의 로그인을 클릭하면, 위와 같이 로그인 타입을 저장하며 각각의 소셜 로그인에 대해 로그인하는 페이지로 이동시킵니다.

로그인이 끝나면, 미리 등록해둔 Redirect 페이지로 이동하므로 그 페이지에서 아래와 같은 로직이 진행되도록 했습니다.

<br>

```js
// ./landing/components/LoginHandler.js


  const CODE = new URL(window.location.href).searchParams.get("code");
  const STATE = new URL(window.location.href).searchParams.get("state");

  // 무슨 소셜 로그인인지를 확인하고 주소 생성
  if (loginType === "N") {
    loginUrl = SocialApi.naver.login(CODE, STATE);
  } else if (loginType === "K") {
    loginUrl = SocialApi.kakao.login(CODE);
  } else if (loginType === "G") {
    loginUrl = SocialApi.google.login(CODE);
  }

```

우선 Redirect Page의 Url의 Parameter를 통해 인가 코드와 State가 넘어옵니다.

앞서 저장했던 login type을 토대로, 현재 로그인 하려는 과정이 네이버, 카카오톡, 구글 중 어느 것인지를 확인한 후, 요청 Url을 생성합니다.

<br>

```js
// ./landing/components/LoginHandler.js

  // 토큰 발급 요청
  const getToken = () => {
    axios
      .post(
        loginUrl,
        {
          headers: {
            "Content-Type": "application/x-www-form-urlencoded;charset=utf-8",
          },
        },
        {
          responseType: "json",
        }
      )
      .then((res) => res.data)
      .then((data) => {
        dispatch(configActions.setIsLoading(true));
        
        // 액세스 토큰으로 로그인 요청
        // 여기서 BE로 넘김
        if (data.access_token) {
          axios
            .post(
              BackApi.users.login,
              {
                accessToken: data.access_token,
                loginType,
              },
              {
                headers: {
                  "Content-type": "application/json",
                  Accept: "application/json",
                },
              }
            )
            .then((res) => {
              dispatch(configActions.setPercentage(50));
              dispatch(logosActions.getLogoAPI());
              
              // 회원가입이 된 사람이 아니라면
              if (res.data.message === "회원가입을 먼저 해주세요.") {
                dispatch(configActions.setPercentage(50));
                  
                // 회원가입을 위해 uid를 저장 후 이동
                // loginType은 로컬 스토리지에 있음
                dispatch(userActions.setUid(res.data.uid));
                dispatch(configActions.setUrl(""));
                  
              // 회원가입 한 사람이라면
              } else {
                  
                // jwt 토큰과 유저 이름을 저장 후 메인 페이지로 이동
                // 메인 페이지 이동 시 선수 전체 데이터와 유저 데이터를 요청 및 저장
                dispatch(playersActions.getPlayerAPI(res.data.jwt));
                dispatch(userActions.setJwt(res.data.jwt));
                dispatch(userActions.setLoginType(""));
                dispatch(usersActions.getUserAPI(res.data.jwt));
                dispatch(configActions.setUrl("main"));
              }
            });
        } else {
          dispatch(configActions.setPercentage(100));
          dispatch(configActions.setUrl(""));
        }
      })
      .catch((err) => console.log(err));
  };
```

인가 코드를 토대로 Access Token 발급을 요청하고, 발급 받은 Access Token을 BE로 넘겨주어 BE에서 이를 통해 유저 정보를 획득합니다.

본래 Access Token과 Secret Key, Auth Key 등을 FE에 저장하는 것은 탈취의 우려가 있어 바람직 하지 않지만, BE에서의 요청으로 인해 Access Token 획득까지 FE에서 처리하는 것으로 구현하였습니다.

또한 그 과정에서, naver API에서는 FE에서 인가코드 요청을 보내면 CORS 에러가 발생하였습니다. 따라서 이를 해결하기 위해 Proxy를 사용하였고, 그 코드는 아래와 같습니다.

<br>

```js
// ./setupProxy.js

const { createProxyMiddleware } = require("http-proxy-middleware");

module.exports = function (app) {
  app.use(
      
    // 네이버는 일부러 Url을 다르게 하여, 이를 검증함
    createProxyMiddleware("/token", {
        
      // 해당 경로
      target: "https://nid.naver.com/oauth2.0/token",
      pathRewrite: {
        
        // 경로에서 해당하는 부분을 바꿈
        "^/token": "",
      },
      changeOrigin: true,
    })
  );

  // BE 또한 해당 오류가 발생하여 수정
  app.use(
    createProxyMiddleware("/back", {
      target: "https://j7d109.p.ssafy.io/back",
      pathRewrite: {
        "^/back": "",
      },
      changeOrigin: true,
    })
  );
};
```

네이버의 CORS에러를 회피하기 위해, 일부러 SocialApi.js 에서 naver Login은 Url 자체가 "/token"으로 시작하도록 바꾸었습니다. 이를 middleware에서 확인하고, target으로 바꾼 뒤 Rewrite를 실행합니다. 마지막으로 changeOrigin을 true로 설정하여, 원하는 Url로 요청을 하되 CORS 에러가 발생하지 않도록 합니다.

그런데 BE에서도, 프로젝트 도중 Jwt 토큰에 대한 handler를 업데이트 한 이후 FE에서의 요청에서 CORS에러가 발생하였습니다. 당시 BE는 완성이 덜 되어서 이를 수정할 시간이 없었기에, BE 또한 middleware를 통해 CORS 에러 방지 처리를 해주었습니다.

<br>

<br>

## 2. Redux Toolkit & Redux Persist

FE를 두 명이서 담당하였는데, 한 사람이 받은 정보를 다른 사람이 써야 했을 뿐더러, State Drilling을 막기 위해 정보를 전역에서 상태 관리 할 필요가 있었습니다. 또한 정보 중 새로고침 등의 경우에도 휘발되지 않아야 하는 정보도 있기에, Local Storage 또는 Session Storage를 이용한 정보 저장도 필요하였습니다.따라서 Redux Toolkit과 Redux Persist 라이브러리를 사용하게 되었고, 팀원은 다른 업무를 맡게되어 이 업무는 제가 진행하였습니다.

간략하게 게임 설정과 관련된 부분만 다루어보겠습니다.

<br>

### 1) Slice

```js
// ./redux/slice/configSlice.js

// 초기 상태
const initialConfigState = {
  url: "",
  
  // 로딩 유무
  loading: {
    isLoading: false,
    percentage: 0,
  },
  
  // BGM 재생
  music: false,
  
  // 방문한 적이 있는가
  visited: false,
};

const configSlice = createSlice({
  name: "config",
  initialState: initialConfigState,
  reducers: {
    setUrl: (state, action) => {
      state.url = action.payload;
    },
    setIsLoading: (state, action) => {
      state.loading.isLoading = action.payload;
    },
    setPercentage: (state, action) => {
      const percentage = state.loading.percentage + action.payload;
      state.loading.percentage = percentage > 100 ? 100 : percentage;
    },
    resetPercentage: (state) => {
      state.loading.percentage = 0;
    },
    setMusic: (state, action) => {
      state.music = action.payload;
    },
    setVisited: (state) => {
      state.visited = true;
    },
  },
});

export const configActions = configSlice.actions;
export default configSlice.reducer;

```

게임을 하는 유저의 상태는 크게 4가지가 필요했습니다.

1. 현재 페이지 : url이 아니라 내부 상태를 통해 메인 페이지, 게임 페이지 등을 이동하도록 했습니다. 이는 게임 도중 게임을 나갈 경우 강제로 메인 페이지로 이동시키기 위함입니다
2. 로딩 유무 : 로딩을 하고 있으면 로딩 페이지를 활성화 시킵니다. 100%이면 일정 시간 이후 로딩이 종료되도록 하였습니다.
3. BGM 재생 : 회원 가입 전 안내를 읽을 때에는 BGM이 흘러 나오지 않도록 하고, 게임이 시작되면 BGM이 흘러나오도록 하였습니다.
4. 방문 여부 : 한 번이라도 방문한 적이 있다면 안내사항을 다시 보여줄 필요가 없으므로 이를 비활성화 합니다. Session Storage에 담기는 정보이기 때문에 나중에 재접속 하는 경우는 다시 확인할 수 있습니다.

따라서 Create Slice를 이용하여 유저 상태와 Reducer를 생성하였습니다.

<br>

url은 아래와 같이 활용됩니다.

<br>

```js
// App.js


function App() {
  const url = useSelector((state) => state.config.url);

  return (
    <>
      <Loading />
      {url === "" && <Landing />}
      {url === "main" && <Main />}
      {url === "guide" && <Guide />}
      {url === "manage" && <Manage />}
      {url === "market" && <Market />}
      {url === "game" && <Game />}
      {url === "match" && <Match />}
      {url === "result" && <Result />}
    </>
  );
}
```

<br>

### 2) Store

```js
// ./redux/store/index.js

// 정보 휘발 방지
const persistConfig = {
  key: "root",
  version: 1,
    
  // Session Storage에 저장(게임 가이드 유무 때문)
  storage: storageSession,
};

// Reducer 통합
const rootReducer = combineReducers({
  user: userReducer,
  logo: logoReducer,
  player: playerReducer,
  game: gameReducer,
  config: configReducer,
  test: testReducer,
});

// reducer와 persist 통합 -> reducer에 지속성을 부여
const persistedReducer = persistReducer(persistConfig, rootReducer);

// 통합 store 생성
// middleware를 통한 직렬화 체크
const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoreActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    }),
});

export const persistor = persistStore(store);
export default store;

```

파일 하나에 Reducer를 통합시켜 전역 관리가 가능하도록 합니다.

최종적으로 아래와 같이 App.js 상위에 store와 persist를 위치시켜주면 전부 적용됩니다.

<br>

```js
// index.js

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <Provider store={store}>
    <PersistGate loading={null} persistor={persistor}>
      <App />
    </PersistGate>
  </Provider>
);
```

이와 같이 store를 통한 전역 변수, persist를 통한 지속성을 둘 다 적용시킬 수 있습니다.

<br>

<br>

## 3. 마무리하며

두 번째 프로젝트는 첫 번째와는 다르게 Front-end로 작업하게 되었습니다. 이를 통해 FE에서의 데이터 흐름과, 상황에 따른 데이터의 관리, CSR과 SSR 등을 고민하며 이해해볼 수 있었습니다. 이러한 경험이 추후 BE로 일을 할 때 도움이 될 것이라 생각합니다. 마찬가지로 완성하지 못한 것에 대한 아쉬움이 남지만, 게임이기에 재밌게 개발에 임할 수 있었습니다.

<br>

<br>

## 4. 그 외 참고 자료

\<BGM 관련\>

[React-howler](https://www.npmjs.com/package/react-howler)

[브금, 효과음 넣기 (howler.js)](https://velog.io/@seungdeng17/Howler.js%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-BGM-%ED%9A%A8%EA%B3%BC%EC%9D%8C-%EB%84%A3%EA%B8%B0)

<br>

\<ERROR 관련\>

[npm audit으로 보안취약점을 발견했을 때의 조치](https://lovemewithoutall.github.io/it/npm-audit-fix/)

[Edit Post page에서 refresh 시 발생하는 Warning](https://github.com/bamichoi/hellokorea/issues/1)

<br>

\<Redux 등\>

[Redux Toolkit](https://redux-toolkit.js.org/usage/usage-guide)

[[React] Redux 기초](https://velog.io/@seungsang00/React-Redux-%EA%B8%B0%EC%B4%88)

[[Redux] Redux-Persist](https://velog.io/@bboyooning/Redux-Persist)

[redux-toolkit적용과 persist 적용](https://velog.io/@hongdol/redux-toolkit%EC%A0%81%EC%9A%A9%EA%B3%BC-persist-%EC%A0%81%EC%9A%A9)

[redux-persist 사용법](https://kyounghwan01.github.io/blog/React/redux/redux-persist/)

[220813 공통 프로젝트 개발일지](https://velog.io/@beberiche/220813-%EA%B3%B5%ED%86%B5-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EA%B0%9C%EB%B0%9C%EC%9D%BC%EC%A7%80)

<br>

\<그 외\>

[프론트에서 안전하게 로그인 처리하기 (ft. React)](https://velog.io/@yaytomato/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%90%EC%84%9C-%EC%95%88%EC%A0%84%ED%95%98%EA%B2%8C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0)

[벨로퍼트와 함께하는 모던 리액트](https://react.vlpt.us/)

[CSS Scroll snap points 알아보기](https://wit.nts-corp.com/2018/08/28/5317)