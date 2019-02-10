# 자바스크립트 핵심개념 (this)
## this
자바스크립트에서는 함수를 호출하는 방식에 따라 this가 바인딩 되는 형태가 조금 씩 다르다. 그래서 한 번 정리해보았다.

  1. 객체의 메서드로 호출 될 때  
  해당 메서드를 호출한 객체로 바인딩 된다.
      ```
      var myObj = {
        name : 'foo',
        sayName : function() {
          console.log(this.name);
        }
      };
      myObj.sayName(); // foo
      ```
  2. 일반적으로 함수형태로 호출 될 때  
  전역객체에 바인딩 된다. (브라우저에서 window)  
  내부함수에서 this 역시 전역객체로 바인딩 된다.
        ```
        var test = 'This is test';
        console.log(window.test); // This is test

        var sayFoo = function() {
          console.log(this.test);
        }
        sayFoo(); // This is test
        ```
  3. 생성자 함수를 호출할 때  
  생성된 객체에 바인딩 된다.  
        - 생성자 함수 동작과정  
        빈 객체 생성 및 this 바인딩 > this를 통한 프로퍼티 생성 > 생성된 객체가 리턴됨.

        *생성자 함수는 new를 붙이지 않고 호출 시 일반 함수와 같이 this가 바인딩 되므로 주의한다.

    4. call과 apply, bind를 이용한 this 바인딩  
    Function.prototype객체의 메서드 이므로 모든 함수에서 사용할 수 있다.  
    함수의 this를 외부 객체로 명시적으로 바인딩할 수 있는 방법이다.
        - call과 apply  
        이 역시 함수를 호출하는 방법이며 사용법은 다음과 같다.
            ```
            var testObj = {
              name: 'LUKE'
            }

            function testFunc() {
              console.log(this.name)
              console.log(arguments)
            }

            testFunc() // undefined
            testFunc.call(testObj, 1, 2) // LUKE, {1, 2}
            testFunc.apply(testObj, [1, 2]) // LUKE, {1, 2}
            ```
        - bind
        bind 역시 this를 명시적으로 할 수 있는 방법이지만 위의 call, apply와 사용법에 조금 차이가 있다.
            ```
            var testObj = {
              name: 'LUKE'
            }

            function testFunc() {
              console.log(this.name)
              console.log(arguments)
            }
            // 바로 함수를 호출하지 않고, this를 설정한 함수객체를 반환한다.
            var retFunc = testFunc.bind(testObj)
            retFunc(1, 2) // LUKE, {1, 2}
            ```

    5. 화살표 함수(Arrow Function)  
    화살표 함수에는 일반 함수와 다른 점이 있다.
        - this가 없다.  
        화살표 함수에서는 this를 어떠한 방식으로 바인딩할 수 없다. 그렇기 때문에 **스코프 체인을 통해 상위 실행컨텍스트의 this를 참조**한다.

        - arguments 객체가 없다.
        관련 문법을 사용하고 싶을 경우 나머지 매개변수(Rest Parameter)를 사용하여 해결할 수 있다.
            ```
            var testfunc = (...args) => {
              console.log(this.name)
              console.log(args)
            }
            testfunc(1, 2, 3);
            ```
        - 익명함수이며 new 호출이 불가하다.

## 참조
> 인사이드 자바스크립트  
> [MDN - 화살표 함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98)