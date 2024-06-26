# 20장 strict mode

### 20.1 strict mode란?

정답은 무엇일까??

```jsx
function foo() {
    x = 10;
}
foo();

console.log(x); // ?
```

자바스크립트 엔진은 foo 함수의 스코프에서 `x` 변수의 선언을 검색하고 없다면 전역 스코프에서 `x` 변수 선언을 검색한다. 이때 `x`는 전역 변수로 동적 생성하고, 전역 객체의 `x` 프로퍼티는 마치 변수처럼 사용할 수 있다. (원래라면 대부분은 reference error가 발생할것이라고 생각)

→ 이러한 현상은 **암묵적 전역**(implicit global)이라고 한다.

**암묵적 전역**은 오류를 발생시키는 원인이 될 가능성이 크다. 따라서 반드시 `var`, `let`, `const` 키워드를 사용하자.

하지만 언제나 오타나 문법 지식의 미비로 인한 실수는 언제나 발생한다.

→ 이를 막기 위해 ES5부터 **strict mode**(**엄격 모드**)가 추가되었다. **strict mode**는 자바스크립트 언어의 문법을 좀 더 엄격히 적용하여 오류를 발생시킬 가능성이 높거나 자바스크립트 엔진의 최적화 작업에 문제를 일으킬 수 있는 코드에 대해 명시적인 에러를 발생시킨다.

<aside>
💡 ESLint 같은 도구를 사용해도 strict mode와 유사한 효과를 얻을 수있다. 린트 도구는 strict가 제안하는 오류는 물론 코딩 컨벤션을 설정 파일 형태로 정의하고 강제할 수 있기 때문에 더욱 강력한 효과를 얻을 수 있다.
따라서 필자는 strict mode보다 린트 도구의 사용을 선호한다.

</aside>

### 20.2 strict mode의 적용

**strict** **mode**를 적용하려면 전역의 선두 또는 함수 몸체의 선두에 `use strict;` 를 추가한다.

```jsx
// 전역
"use strict";

function foo() {
    x = 10;
}
foo(); // ReferenceError

// 함수 몸체의 선두
function foo() {
    "use strict";
    x = 10;
}
foo(); //ReferenceError
```

코드의 선두에 `use strict;` 를 위치시키지 않으면 **strict** **mode**가 제대로 동작하지 않는다.

```jsx
function foo() {
    x = 10("use strict");
}
foo();

console.log(x); // 10
```

### 20.3 **전역에 strict mode를 적용하는 것은 피하자**

전역에 적용한 **strict** **mode**는 스크립트 단위로 적용된다.

```html
<!DOCTYPE html>
<html lang="en">
    <body>
        <script>
            "use strict";
        </script>
        <script>
            x = 1; // 에러가 발생하지 않음
            console.log(x);
        </script>
        <script>
            "use strict";
            y = 1; // ReferenceError
            console.log(y);
        </script>
    </body>
</html>
```

스크립트 단위로 적용된 **strict mode**는 다른 스크립트에 영향을 주지 않고 해당 스크립트에 한정되어 적용된다.

하지만 **strict** **mode** 스크립트와 **non-strict** **mode** 스크립트를 혼용하는 것은 오류를 발생시킬 수 있다.

특히 외부 서드파티 라이브러리를 사용하는 경우 라이브러리가 **non-strict** **mode**인 경우도 있기 때문에 전역에 **strict** **mode**를 적용하는 것은 바람직하지 않다.

```jsx
// 즉시 실행 함수의 선두에 strict mode 적용
(function () {
    "use strict";
    // Do something
})();
```

즉시 실행 함수로 스크립트 전체를 감싸서 스코프를 구분하고 즉시 실행 함수의 선두에 strict mode를 적용한다.

### 20.4 **함수 단위로 strict mode를 적용하는 것도 피하자**

어떤 함수는 **strict mode**를 적용하고 어떤 함수는 **strict mode**를 적용하지 않는 것은 바람직하지 않으며 모든 함수에 일일이 **strcit mode**를 적용하는 것은 번거로운 일이다.

**strict mode**가 적용된 함수가 참조할 함수 외부의 컨텍스트에 **strict mode**를 적용하지 않는다면 문제가 발생할 수 있다.

```jsx
(function () {
    // non-strict mode
    var let = 10; // 에러 발생 x
    function foo() {
        "use strict";
        let = 20; // SyntaxError
    }
    foo();
})();
```

따라서 **strict** **mode**는 즉시 실행 함수로 감싼 스크립트 단위로 적용하는 것이 바람직하다.\*\*\*\*

### **20.5. strict mode가 발생시키는 에러**

다음은 **strict** **mode**를 적용했을 때 에러가 발생하는 대표적인 사례다.\*\*\*\*

**1. 암묵적 전역**

선언하지 않은 변수를 참조하면 ReferenceError가 발생한다.

```jsx
(function () {
    "use strict";
    x = 1;
    console.log(x); // ReferenceError
})();
```

**2. 변수, 함수, 매개변수의 삭제**

`delete` 연산자로 변수, 함수, 매개변수를 삭제하면 SyntaxError가 발생한다.

```jsx
(function () {
    "use strict";
    var x = 1;
    delete x; // SyntaxError

    function foo(a) {
        delete a; // SyntaxError
    }
    delete foo; // SyntaxError
})();
```

**3. 매개변수 이름의 중복**

중복된 매개변수 이름을 사용하면 SyntaxError가 발생한다.

```jsx
(function () {
    "use strict";

    // SyntaxError
    function foo(x, x) {
        return x + x;
    }
    console.log(foo(1, 2));
})();
```

**4. with 문의 사용**

`with` 문을 사용하면 SyntaxError가 발생한다. `with` 문은 전달된 객체를 스코프 체인에 추가한다.

`with` 문은 동일한 객체의 프로퍼티를 반복해서 사용할 때 객체 이름을 생략할 수 있어서 코드가 간단해지는 효과가 있지만 성능과 가독성이 나빠지는 문제가 있다.

→ `with` 문은 사용하지 않는 것이 좋다.

```jsx
(function () {
    "use strict";

    // SyntaxError
    with ({ x: 1 }) {
        console.log(x);
    }
})();
```

### **20.6 strict mode 적용에 의한 변화**

**1. 일반 함수의 this**

strict mode 에서 함수를 일반 함수로서 호출하면 this에 undefined가 바인딩된다. 생성자 함수가 아닌 일반 함수 내부에서는 this를 사용할 필요가 없기 때문이다. → 이때 에러는 발생하지 않음

```jsx
(function () {
    "use strict";

    function foo() {
        console.log(this); // undefined
    }
    foo();

    function Foo() {
        console.log(this); // Foo
    }
    new Foo();
})();
```

**2. arguments 객체**

strict mode에서는 매개변수에 전달된 인수를 재할당하여 변경해도 arguments 객체에 반영되지 않는다.

```jsx
(function (a) {
    "use strict";
    // 매개변수에 전달된 인수를 재할당하여 변경
    a = 2;

    // 변경된 인수가 arguments 객체에 반영되지 않는다.
    console.log(arguments);
})(1);

// [Arguments] { '0': 1 }
```
