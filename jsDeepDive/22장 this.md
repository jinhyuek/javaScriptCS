# 22장 this

### 22.1 this 키워드

**메서드가 자신이 속한 객체의 프로퍼티를 참조하려면 먼저 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야한다.**

```jsx
const circle = {
    // 프로퍼티: 객체 고유의 상태 데이터
    radius: 5,
    // 메서드: 상태 데이터를 참조하고 조작하는 동작
    getDiameter() {
        // 이 메서드가 자신이 속한 객체의 프로퍼티나 다른 메서드를 참조하려면
        // 자신이 속한 객체인 circle을 참조 할 수 있어야 한다.
        return 2 * circle.radius;
    },
};

console.log(circle.getDiameter()); // 10
```

위 예제의 객체 리터럴은 circle 변수에 할당되기 직전에 평가된다.

→ getDiameter() 호출되는 시점에는 이미 객체 리터럴의 평가가 완료되어 객체가 생성되었고 circle 식별자에 생성된 객체가 할당된 이후다.

→ 메서드 내부에서 circle 식별자를 참조할 수 있다.

하지만 자기 자신이 속한 객체를 재귀적으로 참조하는 방식은 일반적이지 않으며 바람직하지도 않다.

```jsx
function Circle(radius) {
	// 이 시점에는 생성자 함수 자신이 생성할 인스턴스를 가리키는 식별자를 알 수 없다.
	????.radius = radius;
}

Circle.prototype.getDiameter = function () {
	// 이시점에는 생성자 함수 자신이 생성할 인스턴스를 가리키는 식별자를 알 수 없다.
	return 2 * ????.radius;
}

// 생성자 함수로 인스턴스를 생성하려면 먼저 생성자 함수를 정의해야 한다.
const circle = new Circle(5);
```

생성자 함수 내부에서는 프로퍼티 또는 메서드를 추가하기 위해 자신이 생성할 인스턴스를 참조할 수 있어야 한다.

하지만 생성자 함수에 의한 객체 생성 방식은 먼저 생성자 함수를 정의한 이후 new 연산자와 함께 생성자 함수를 호출하는 단계가 추가로 필요하다.

→ 생성자 함수로 인스턴스를 생성하려면 먼저 생성자 함수가 존재해야 한다.

**this는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수다. this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있다.**

this는 자바스크립트 엔진에 의해 암묵적으로 생성되며, 코드 어디서든 참조할 수 있다.

**→ this가 가리키는 값, 즉 this 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.**

```jsx
// 객체 리터럴
const circle = {
    radius: 5,
    getDiameter() {
        // this는 메서드를 호출한 객체를 가리킨다.
        return 2 * this.radius;
    },
};

console.log(circle.getDiameter()); // 10
```

```jsx
// 생성자 함수
function Circle(radius) {
    // this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    this.radius = radius;
}

Circle.prototype.getDiameter = function () {
    // this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    return 2 * this.radius;
};

// 인스턴스 생성
const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

**자바스크립트의 this는 함수가 호출되는 방식에 따라 this에 바인딩될 값, 즉 this 바인딩이 동적으로 결정된다.**

this는 코드 어디서든지 참조 가능하다.

```jsx
// this는 어디서든지 참조 가능하다.
// 전역에서 this는 전역 객체 window를 가리킨다.
console.log(this); // window

function square(number) {
    // 일반 함수 내부에서 this는 전역 객체 window를 가리킨다.
    console.log(this); // window
    return number * number;
}
square(2);

const person = {
    name: "Son",
    getName() {
        // 메서드 내부에서 this는 메서드를 호출한 객체를 가리킨다.
        console.log(this); // {name: "Son", getName: function}
        return this.name;
    },
};
console.log(person.getName()); // Son

function Person(name) {
    this.name = name;
    // 생성자 함수 내부에서 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    console.log(this); // Person {name: "Son"}
}

const me = new Person("Son");
```

### 22.2 함수 호출 방식과 this 바인딩

**this 바인딩(this에 바인딩 값)은 함수 호출 방식, 즉 함수가 어떻게 호출되었는지에 따라 동적으로 결정된다.**

<aside>
💡 **렉시컬 스코프와 this 바인딩은 결정 시기가 다르다.**
함수 상위 스코프를 결정하는 방식인 렉시컬 스코프는 함수 정의가 평가되어 함수객체가 생성되는 시점에 상위 스코프를 결정한다. 
하지만  this바인딩은 함수 호출 시점에 결정된다.

</aside>

동일한 함수도 다양한 방식으로 호출할 수 있다.

-   일반 함수 호출
-   메서드 호출
-   생성자 함수 호출
-   Function.prototype.apply/call/bind 메서드에 의한 간접 호출

```jsx
// this 바인딩은 함수 호출 방식에 따라 동적으로 결정된다.
const foo = function () {
    console.dir(this);
};

// 1. 일반 함수 호출
foo(); // window

// 2. 메서드 호출
const obj = { foo };
obj.foo();

// 3. 생성자 함수 호출
new foo(); // foo {}

// 4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출
const bar = { name: "bar" };
foo.call(bar); // bar
foo.apply(bar); // bar
foo.bind(bar)(); // bar
```

**1. 일반 함수 호출**

**기본적으로 this에는 전역 객체 가 바인딩된다.**

```jsx
function foo() {
    console.log("foo's this: ", this); // window
    function bar() {
        console.log("bar's this: ", this); // window
    }
    bar();
}
foo();
```

**일반 함수로 호출하면 함수 내부의 this에는 전역 객체가 바인딩된다.**

this는 객체의 프로퍼티나 메서드를 자기 참조 변수이므로 객체를 생성하지 않는 일반 함수에서 this는 의미가 없다. → strict mode가 적용된 일반 함수 내부의 this에는 undefined가 바인딩된다.

```jsx
function foo() {
    "use strict";

    console.log("foo's this: ", this); // undefined
    function bar() {
        console.log("bar's this: ", this); // undefined
    }
    bar();
}
foo();
```

```jsx
// var 키워드로 선언한 전역 변수 value는 전역 객체의 프로퍼티다.
var value = 1;
// const 키워드로 선언한 전역 변수 value는 전역 객체의 프로퍼티가 아니다.
// const value = 1;

const obj = {
    value: 100,
    foo() {
        console.log("foo's this: ", this); // {value: 100, foo: function}
        console.log("foo's this.value: ", this.value); // 100

        // 메서드 내에서 정의한 중첩 함수
        function bar() {
            console.log("bar's this: ", this); // window
            console.log("bar's this.value: ", this.value); // 1
        }

        // 메서드 내에서 정의한 중첩 함수도 일반 함수로 호출되면 중첩 함수 내부의 this에는
        // 전역 객체가 바인딩된다.
        bar();
    },
};

obj.foo();
```

콜백 함수가 일반 함수로 호출된다면 콜백 함수 내부의 this에도 전역 객체가 바인딩된다.

어떠한 함수라도 일반 함수로 호출되면 this에 전역 객체가 바인딩된다.

```jsx
var value = 1;

const obj = {
    value: 100,
    foo() {
        console.log("foo's this: ", this); // {value: 100, foo: function}

        // 콜백 함수 내부의 this에는 전역 객체가 바인딩된다.
        setTimeout(function bar() {
            console.log("bar's this: ", this); // window
            console.log("bar's this.value: ", this.value); // 1
        }, 100);
    },
};

obj.foo();
```

```jsx
var value = 1;

const obj = {
    value: 100,
    foo() {
        // 콜백 함수에 명시적으로 this를 바인딩한다.
        setTimeout(
            function bar() {
                console.log(this.value); // 100
            }.bind(this),
            100
        );
    },
};

obj.foo();
```

또는 화살표 함수를 사용해서 this 바인딩을 일치시킬 수도 있다.

```jsx
var value = 1;

const obj = {
    value: 100,
    foo() {
        // 화살표 함수 내부의 this는 상위 스코프의 this를 가리킨다.
        setTimeout(() => console.log(this.value), 100); // 100
    },
};

obj.foo();
```

**2. 메서드 호출**

메서드 내부의 this에는 메서드를 호출한 객체, 즉 메서드를 호출할 때 메서드 이름 앞의 마침표(.) 연산자 앞에 기술한 객체가 바인딩된다.

메서드 내부의 this는 메서드를 소유한 객체가 아닌 메서드를 호출한 객체에 바인딩되는 것이다.

```jsx
const person = {
    name: "Son",
    getName() {
        // 메서드 내부의 this는 메서드를 호출한 객체에 바인딩된다.
        return this.name;
    },
};

// 메서드 getName을 호출한 객체는 person이다.
console.log(person.getName()); // Son
```

→ getName 프로퍼티가 가리키는 함수 객체, 즉 getName 메서드는 다른 객체의 프로퍼티에 할당하는 것으로 다른 객체의 메서드가 될 수도 있고 일반 변수에 할당하여 일반 함수로 호출될 수도 있다.

```jsx
const anotherPerson = {
    name: "Son",
};
// getName 메서드를 anotherPerson 객체의 메서드로 할당
anotherPerson.getName = person.getName;

// getName 메서드를 호출한 객체는 anotherPerson이다.
console.log(anotherPerson.getName()); // Son

// getName 메서드를 변수에 할당
const getName = person.getName;

// getName 메서드를 일반 함수로 호출
console.log(getName()); // ""
// 일반 함수로 호출된 getName 함수 내부의 this.name은 브라우저 환경에서 window.name과 같다.
```

프로토타입 메서드 내부에서 사용된 this도 일반 메서드와 마찬가지로 해당 메서드를 호출한 객체에 바인딩 된다.

```jsx
function Person(name) {
    this.name = name;
}

Person.prototype.getName = function () {
    return this.name;
};

const me = new Person("Son");

// getName 메서드를 호출한 객체는 me다.
console.log(me.getName()); // Son

Person.prototype.name = "Sonny";

// getName 메서드를 호출한 객체는 Person.prototype이다.
console.log(Person.prototype.getName()); // Sonny
```

**3. 생성자 호출**

생성자 함수 내부의 this에는 생성자 함수가 생성할 인스턴스가 바인딩된다.

```jsx
// 생성자 함수
function Circle(radius) {
    // 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    this.radius = radius;
    this.getDiameter = function () {
        return 2 * this.radius;
    };
}

const circle1 = new Circle(5);
const circle2 = new Circle(10);

console.log(circle1.getDiameter()); // 10
console.log(circle2.getDiameter()); // 20

// new 연산자와 함께 호출하지 않으면 생성자 함수로 동작하지 않는다.
const circle3 = Circle(15);

// 일반 함수로 호출된 Circle에는 반환문이 없으므로 암묵적으로 undefined를 반환한다.
console.log(circle3); // undefined

// 일반 함수로 호출된 Circle 내부의 this는 전역 객체를 가리킨다.
console.log(radius); // 15
```

**4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출**

apply, call 메서드는 this로 사용할 객체와 인수 리스트를 인수로 전달받아 함수를 호출한다.

```jsx
function getThisBinding() {
    return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

console.log(getThisBinding()); // window

// getThisBinding 함수를 호출하면서 인수로 전달한 객체를 getThisBinding 함수의 this에 바인딩
console.log(getThisBinding.apply(thisArg)); // {a: 1}
console.log(getThisBinding.call(thisArg)); // {a: 1}
```

**apply와 call 메서드의 본질적인 기능은 함수를 호출하는 것이다.**

apply와 call 메서드는 호출할 함수에 인수를 전달하는 방식만 다를 뿐 동일하게 동작한다.

```jsx
function getThisBinding() {
    console.log(arguments);
    return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

// apply 메서드는 호출할 함수의 인수를 배열로 묶어 전달한다.
console.log(getThisBinding.apply(thisArg, [1, 2, 3]));

// call 메서드는 호출할 함수의 인수를 배열로 묶어 전달한다.
console.log(getThisBinding.call(thisArg, 1, 2, 3));

// [Arguments] { '0': 1, '1': 2, '2': 3 }
// { a: 1 }
// [Arguments] { '0': 1, '1': 2, '2': 3 }
// { a: 1 }
```

apply와 call 메서드의 대표적인 용도는 arguments 객체와 같은 유사 배열 객체에 배열 메서드를 사용하는 경우다. arguments 객체는 배열이 아니기 때문에 Array.prototype.slice 같은 배열의 메서드를 사용할 수 없으나 apply와 call 메서드를 이용하면 가능하다.

```jsx
function convertArgsToArray() {
    console.log(arguments);

    // Array.prototype.slice를 인수 없이 호출하면 배열의 복사본을 생성한다.
    const arr = Array.prototype.slice.call(arguments);
    // const arr = Array.prototype.slice.apply(arguments);
    console.log(arr);

    return arr;
}

convertArgsToArray(1, 2, 3); // [1, 2, 3]
```

Function.prototype.bind 메서드는 apply와 call 메서드와 달리 함수를 호출하지 않고 this로 사용할 객체만 전달한다.

```jsx
function getThisBinding() {
    return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

// bind 메서드는 함수에 this로 사용할 객체를 전달한다.
// bind 메서드는 함수를 호출하지는 않는다.
console.log(getThisBinding.bind(thisArg));
// bind 메서드는 함수를 호출하지는 않으므로 명시적으로 호출해야 한다.
console.log(getThisBinding.bind(thisArg)());

// [Function: bound getThisBinding]
// { a: 1 }
```

```jsx
const person = {
    name: "Son",
    foo(callback) {
        setTimeout(callback, 100);
    },
};

person.foo(function () {
    console.log(`Hi! my name is ${this.name}.`); // Hi! my name is .
});
```

binding 하지 않으면 person.foo의 콜백 함수 내부에서 `this.name` 은 `window.name` 과 같다.

```jsx
const person = {
    name: "Son",
    foo(callback) {
        // bind 메서드로 callback 함수 내부의 this 바인딩을 전달
        setTimeout(callback.bind(this), 100);
    },
};

person.foo(function () {
    console.log(`Hi! my name is ${this.name}.`); // Hi! my name is Son.
});
```
