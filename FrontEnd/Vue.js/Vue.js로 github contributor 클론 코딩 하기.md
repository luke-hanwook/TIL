# Vue.js로 github contributor of insight 클론 코딩 하기

0. 소개

    [링크](https://github.com/luke-hanwook/vue-d3-github-clone-coding)를 통해 확인해 보자

1. 시작

    최근 데이터 과학 혹은 데이터 엔지니어링이란 분야가 유행이고(?) 관련 직업까지 생긴 것으로 알고 있다. 학부생때 데이터마이닝 수업도 재밌게 들었고, 현재 프론트엔드 쪽에 관심이 생기다 보니 웹에서 그러한 데이터를 시각적으로 아름답게 표현해줄 수 있는 그것도 자바스크립트로 라이브러리(대단쓰)가 있다는 것을 알게 되었다. 그래서 토이 프로젝트로 깃허브 인사이트의 공헌자 부분을 클론코딩 해보기로 하였다.

2. 준비

    우선 익숙한 [Vue.js](https://kr.vuejs.org/)를 사용하기로 했다. 데이터 시각화 자바스크립트 라이브러리인 [d3.js](https://d3js.org/) 그리고 코드로 이미지를 구현할 수 있는 [SVG](https://developer.mozilla.org/ko/docs/Web/SVG/Tutorial), 마지막으로 [Github의 Statistics API](https://developer.github.com/v3/repos/statistics/)

    정리하자면
      - [Vue.js](https://kr.vuejs.org/)
      - [d3.js](https://d3js.org/)
      - [SVG](https://developer.mozilla.org/ko/docs/Web/SVG/Tutorial)
      - [Github의 Statistics API](https://developer.github.com/v3/repos/statistics/)

3. 학습

    d3.js 라이브러리를 전혀 모르는 상태였기 때문에 기본 개념에 대해 공부하였다. **선택자** 그리고 관련 데이터를 돔으로 만들기 위해 사용되는 개념인 **가상의 DOM**과 **데이터 & 조인(data, enter, append)**, 데이터를 범위와 값으로 표현하기 위한 **스케일과 축(scale, axis)** 정도이다. SVG 같은 경우엔 온라인에 있는 몇몇의 tutorial을 참고하였다.

4. 첫 번째 난관

    **Vue.js의 Directive vs D3.js 가상 DOM의 selectAll()**
    Vue.js에는 지시문이라 하여 랜더링을 조절할 수 있는 기능이 있다. D3.js에는 selectAll()이 있어 가상으로 DOM Element를 만들 수 있다. 이 둘을 적절히 조절해야하는 것이 관건이었다. 여러 사이트들을 검색해보고 문서들을 참조하였고, **결과적으로 Vue 컴포넌트 단위의 개발을 하기 위해서 DOM Element의 생성은 Vue에게 맡기고 데이터 부분은 D3를 사용하도록 하였다.** 총 3개의 그래프를 구현하였는데 Bar 그래프의 경우 데이터를 바인딩하고 속성을 정의하는 부분까지 D3.js를 이용한 반면 Area나 Line 그래프의 경우 데이터를 형식에 맞게 변환만 하였고, 데이터의 바인딩은 Vue의 커스텀 바인딩을 사용하였다.

5. 스케일과 축

    스케일과 축을 작성하는 부분도 Vue 컴포넌트로 따로 분리하였다. 3개의 그래프에서 함께 사용될 것이기 때문에 상태관리 도구인 Vuex를 사용하여 스케일을 생성하였다. 또 공헌자 부분의 아래에 보면 선택된 부분이 확대되어 작은 그래프로 나온다. 이 부분을 고려하여 **스케일과 축이 동적인 값을 사용하여 변경**될 수 있게 하였다. 덕 분에 모든 그래프에서 **스케일 객체와 축 컴포넌트를 재 사용**할 수 있었다.

6. 두 번째 난관이라 쓰고 비동기 데이터 처리라 읽는다

    데이터의 값, 스케일의 domain 값을 설정하는 부분 등 Ajax를 이용한 한 번의 비동기 통신 설정된다. 콜백함수에선 Vuex에 action만 하고 그래프나 스케일을 만들고 데이터를 조작하는 행위는 Vue 컴포넌트에서 하기에 컴포넌트의 mounted 실행주기에선 해당 값을 핸들링 할 수 없었다. (Directive를 이용한 Bar그래프 제외) 대안으로 데이터 변경을 감지할 수 있는 watch 메서드를 사용하여 그래프를 랜더링하였다.

7. 그래프의 범위 선택

    깃허브의 공헌자 부분엔 범위를 선택하여 그래프 하단에 작은 확대 그래프를 노출시켜준다. 범위를 선택하고 이동시키고 늘이고 하는 재미있는 기능이었고, 이 부분은 마우스 이벤트를 통해 x축을 계산하여 (Y축은 고정이였음 다행쓰 :)) SVG의 x축과 width를 조작하면서 동적으로 구현되었다. (값이 동적으로 바인딩 됨에 따라 리랜더링 되는 vue 아니었으면 정말 어떻게 구현했을까요..? 매번 돔을 찾아서 값을 추가해야 했었나... 너무 아재말투인가..) 이 기능의 핵심은 선택 범위의 값을 가져와야 하는 것이었다. 이 역시 D3.js에서 제공해주고 있다. 치환된 값을 원래 값으로 변환 시켜주는 **d3.invert()** 와 변환 값을 스케일을 통해 리스트(api 값의 리스트는 배열로 되어 있었다.)의 인덱스로 변환시켜주는 **d3.bisector()** 기능을 통해 구현할 수 있다.

      ```
      getDataIndex(x) {
        const bisectDate = d3.bisector(d => {
          return new Date(d.w * 1000);
        }).left;
        const x0 = this.scale.get("x").invert(x);
        return bisectDate(this.weeks, x0, 1);
      }
      ```

8. 범위 선택된 값을 작은 그래프로

    범위 선택된 값은 Vuex로 mutation하여 하단에 보여주는 작은 그래프 컴포넌트에서 바로 접근할 수 있게 하였다. watch로 값 변경을 감지하였고, Area 그래프 컴포넌트와 axix 컴포넌트를 재사용하였다.

9. 마무리

    간단한 프로젝트라 예외처리라던지 api의 모든 정보를 클론할 수는 없어 아쉬웠지만, 짧은 기간동안 처음 사용해보는 라이브러리 및 기술을 적용하면서 앞으로 관련 기술들에 대한 관심이 높아졌다. 비슷한 내용으로 내가 좋아하는 지도에 그래프를 뿌려보는 작업을 진행하려 한다.


10. 참조
    > http://webframeworks.kr/getstarted/d3js/  
    >https://medium.com/tyrone-tudehope/composing-d3-visualizations-with-vue-js-c65084ccb686  
    >https://codepen.io/terrymun/pen/gmBdKq
