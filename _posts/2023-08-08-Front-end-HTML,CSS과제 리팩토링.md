---
published: true
title: '2023-08-08-Front-End-HTML,CSS과제 리팩토링'
categories:
  - Front-End
tags:
  - Front-End
toc: true
toc_sticky: true
toc_label: 'Front-End'
---

# 이전 코드 VS 리펙토링 코드

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/airbnb_clone1.png?raw=true)

이전 코드에서는 헤더 부분이 fixed로 설정되어있지 않고, relative로 설정하여 스크롤을 위로 올려야지만 헤더가 보였다. 헤더 고정을 위해 fixed를 사용해봤지만 width를 지정해주지 않아 헤더가 깨지는 이슈가 있었다. 해당 이슈를 그룹스터디에서 공유하였고, 그룹스터디 조원분이 width를 지정해줘야지 깨지지 않는다고 말씀해주셨다. 이후, 피드백을 반영하여 position을 fixed로, width를 100%로 설정해주었고, 헤더가 가장 위에 보일 수 있게 z-index도 설정해주었다.

<br>

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/airbnb_clone2.png?raw=true)

헤더 바로 아레 있는 explorer-section도 헤더 아래 위치하여 고정될 수 있게 설정해주었다. 또한, background-color도 배경과 잘 어울리도록 설정해주었다.

<br>

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/airbnb_clone3.png?raw=true)

마지막으로, 이전 코드에서는 nav bar의 구분선을 li tag에 가상 요소를 추가하여 표현하였는데, 원본 사이트와 비교해보니 원하는 모양의 구분선이 그려지지 않아 리팩토링을 진행하였다. a tag에 border를 설정해 보다 간단하면서도 아래 사진과 같이 원본 사이트와 비슷한 nav bar를 구현할 수 있었다.

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/airbnb_clone4.png?raw=true)

<br>

# 추후 개선할 점

```javascript
prevButton.addEventListener('click', () => {
  const curLeft = parseInt(swiperWrapperEl.style.left) || 0;
  swiperWrapperEl.style.left = curLeft + swiperElStyleWidth / 2 + 'px';
  if (curLeft <= 0 && curLeft >= -swiperElStyleWidth) {
    swiperWrapperEl.style.left = '0';
    prevButtonWrap.style.display = 'none';
  }
  if (curLeft >= -swiperWrapperElStyleWidth + swiperElStyleWidth) {
    nextButtonWrap.style.display = 'flex';
  }
});
```

javascript를 이용하여 직접 구현한 swiper 코드 중 prevButton 엘리먼트의 이벤트 리스너 부분에서 left 속성을 사용했었다. 해당 코드에서 버튼 클릭시,

1. `swiperWrapperEl` 엘리먼트의 현재 왼쪽 위치를 한다.
2. `swiperWrapperEl` 엘리먼트의 왼쪽 위치를 현재 위치에 `swiperElStyleWidth` 엘리먼트의 너비의 절반을 더한다.
3. `swiperWrapperEl` 엘리먼트의 현재 왼쪽 위치가 0과 `swiperElStyleWidth` 엘리먼트의 너비의 절반 사이이면 `swiperWrapperEl` 엘리먼트의 왼쪽 위치를 0으로 설정하고 `prevButtonWrap` 엘리먼트를 숨긴다.
4. `swiperWrapperEl` 엘리먼트의 현재 왼쪽 위치가 `swiperWrapperEl` 엘리먼트의 너비와 `swiperElStyleWidth` 엘리먼트의 너비의 절반의 합보다 크거나 같으면 `nextButtonWrap` 엘리먼트를 표시한다.

위와 같은 과정을 수행하기 위해 left 속성을 사용했는데, 코드리뷰 중 움직이는 모션의 경우는 left 속성 사용 시 재랜더링을 일으켜서 translate속성이나 translate3d 속성 사용을 고려해보라는 리뷰가 있었다. 같은 기능을 하는 속성들 사이에서도 성능적인 차이가 있다는 것을 알려 주셔서 너무 감사했다. 추후에는 모션 구현 시, left 속성보다는 translate 속성을 이용하여 성능을 개선해볼 예정이다.  
<a> https://mygumi.tistory.com/238 </a>

또한, 아직 javascript를 완벽히 숙지하지 못해서 반복되는 데이터 처리를 해주지 않았었는데, 멘토님께서 해주시면서 javascript의 반복문을 활용해서 반복되는 데이터에 대한 렌더링을 해보라는 리뷰를 남겨 주셨다. 실제로 그룹스터디원분 중 데이터를 json 형태로 직접 만드셔서 렌더링하신 분이 계셨는데, javascript에 좀 더 익숙해지면 그 방법을 활용해서 랜더링을 해볼 예정이다.

<br>

# 느낀점

이번 프로젝트에서의 핵심 사항이라고 한다면 swiper를 라이브러리를 사용하지 않고 vanilla로 구현한 점인 것 같다. 하지만 멘토님께서 리뷰해 주셨듯이 안정적으로 release된 라이브러리를 사용하는 것이 오히려 프로젝트 전체적인 측면에서는 공수를 덜어줄 수 있을 것 같다. 라이브러리 사용을 무조건 지양하지 말고 라이브러리 사용 시 해당 레퍼런스를 읽으면서 이해하고 사용한다면 실력 향상과 함께 프로젝트 진행 시 효율도 높일 수 있을 것 같다.
