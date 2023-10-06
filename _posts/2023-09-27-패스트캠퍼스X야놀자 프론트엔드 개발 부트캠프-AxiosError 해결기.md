---
published: true
title: '2023-09-27-패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프-AxiosError 해결기'
categories:
  - 패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프
tags:
toc: true
toc_sticky: true
toc_label: '패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프'
---

### 사건의 발단

때는 9월 27일 React 실시간 강의 중 디즈니 플러스 앱을 구현하고 있었다. Row 영역에 들어갈 component를 구현 도중 아래와 같이 `fectchMovieData` 함수를 구현하였다.

```javascript
const fetchMovieData = useCallback(async () => {
  const response = await axios.get(fetchUrl);

  setMovies(response.data.results);
}, [fetchUrl]);
```

`fetchMovieData` 함수는 `useCallback` 훅을 이용하여 정의되는데, 이 함수는 비동기로 동작하며 `axios`를 사용하여 `fetchUrl`로 request를 보내고, response를 `setMovies` 함수를 통해 상태를 설정한다.
<br />
<br />
이때, `useCallback` 훅은 함수를 **메모이제이션**하여 component가 렌더링될 때마다 다시 생성되지 않도록 한다. 이 함수의 의존성 배열은 `[fetchUrl]`로 지정되어 있으며, 이것은 `fetchMovieData` 함수 내에서 `fetchUrl`을 참조하는 경우에만 함수가 다시 생성됨을 의미한다.
<br />
<br />
이후, 아래와 같이 `useEffect` 훅을 짜주었다.

```javascript
useEffect(() => {
  fetchMovieData();
}, [fetchMovieData]);
```

이때, `useEffect`의 의존성 배열은 `[fetchMovieData]`로 지정되어 있는데, 이것은 `fetchMovieData` 함수가 변경될 때만 `useEffect` 내의 코드 블록이 실행되어야 함을 나타낸다. 즉, `fetchMovieData` 함수의 내용이 변경되지 않는 한, `useEffect`는 단 한 번만 실행된다. (실제로는 `fetchMovieData` 함수가 `fetchUrl` 값이 변경될 때마다 새로 생성되므로 해당 함수 내용의 변경을 감지하는 것이 아니라 함수 객체의 변경을 감지하여 실행된다.)
<br />
<br />
마지막으로, `useEffect` 내에서는 `fetchMovieData` 함수를 호출하고 있는데, 이는 컴포넌트가 렌더링될 때마다 `fetchMovieData` 함수가 실행되는 것을 의미한다.

### 에러 내용

문제가 없어 보이는 코드 같았지만 실행해보니 다음과 같은 에러가 발생하였다.

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/AxiosError_01.png?raw=true)

### 에러 해결 과정

처음에는 `get` 요청을 보내는 `fetchUrl`이 잘못된 줄 알았는데, 다른 컴포넌트에서 동일한 URL로 요청을 보낼 때는 정상적으로 response가 도착하였다. 혹시 몰라서 오류가 찍힌 주소로 postman을 사용하여 요청을 보내보았다.

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/AxiosError_02.png?raw=true)

위와 같이 직접 해당 주소로 요청을 보냈을 때는 정상적으로 response가 도착하였다. 분명히 fetch 과정에서 문제가 발생한 것은 맞는데 원인을 모르겠어서 에러를 뜯어보다가 이상한 점을 발견하였다.

```json
responseURL
:
"http://localhost:3000/trending/all
```

위와 같이 responseURL이 localhost로 설정되어있었다. 분명히 axios instance를 생성할 때 baseURL을 아래와 같이 지정해주었는데, 엉뚱한 곳에서 response가 오고 있었던 것이다.

```javascript
const axiosInstance = axios.create({
  baseURL: 'https://api.themoviedb.org/3',
  params: {
    api_key: 'API_KEY',
    language: 'ko-KR',
  },
});
```

설마 해서 axios를 import하는 부분을 보니 다음가 같이 되어있었다.

```javascript
import axios from 'axios';
```

Copilot을 사용하다 보니 import 과정에서 무심코 tab키를 쳐서 위와 같이 import를 해왔던 것이다.
<br />
<br />
다시 다음과 같이 axios instance의 경로를 잡아주었더니 에러가 해결되었다 ㅠㅠ

```javascript
import axios from '../api/axios';
```

### 마무리

이번 에러는 import시의 어이없는 해프닝으로 마무리되었지만, 실시간 강의 도중 이 에러가 해결이 안되어서 강의에 1시간 넘게 집중을 다시 하지 못했다 ㅠㅠ (나중에 다시보기 돌려볼게요...) 하지만, 이번 일로 `useCallBack` 훅과 **의존성 배열**의 역할에 대해서 공부해보는 계기가 되었다. 솔직히 해당 내용은 이론 강의로만 배워서 실제 프로젝트에서는 어떻게 사용되는지 이해가 잘 되지 않았었는데, 앞으로 data를 fetch할 때 적용해서 불필요한 함수 재생성을 막는 데에 사용해봐야겠다.
