# JavaScript Prototype

### Prototype 이란?

- 자바스크립트의 모든 **객체**(Object)는 자신과 연결된 **[프로토타입 객체](#prototype-chain)**(prototype object)를 가집니다.
- 프로토타입은 `상속`을 위한 매커니즘입니다.
- 프로토타입은 공유 가능한 속성과 함수를 담고 있는 **단일 객체**입니다.
- 객체에서 어떤 `속성`이나 `함수`를 찾을 때, 그 **객체에 없으면 프로토타입 체인을 따라 올라가면서 찾습니다.**

---

### 기본 예시

```js
function Person(name, age, address) {
  this.name = name;
  this.age = age;
  this.address = address;
}

Person.prototype.getInfo = function () {
  return `Name: ${this.name}, Age: ${this.age}, Address: ${this.address}`;
};

const person1 = new Person("종원", 29, "서울시 금천구");

console.log(person1.getInfo());
```

[prototype-exp.js](./example/prototype-exp.js)

```
> 코드 진행 순서

1. `new Person("종원")`로 `person1`이라는 객체가 메모리에 새로 생성됨
2. `person1`은 `Person.prototype`과 연결됨
3. `person1`에는 `getInfo`직접 정의를 찾을 수 없음
4. JS 엔진이 `person1.__proto__`에서 `getInfo`를 찾아서 실행
```

---

### Prototype Chain

```js
function Person(name, age, address) {
  this.name = name;
  this.age = age;
  this.address = address;
}

const person1 = new Person("종원", 29, "서울시 금천구");

console.log(
  "person1은 Person 프로토타입의 상속? ",
  person1.__proto__ === Person.prototype
); // true
console.log(
  "person1의 프로토타입은 Object 프로토타입의 상속?",
  Person.prototype.__proto__ === Object.prototype
); // true
console.log("person1은 null?", Object.prototype.__proto__ === null); // true
```

[prototype-chain.js](./example/prototype-chain.js)

프로토타입 체인은 **프로토타입 객체들의 연결**입니다.

그래서 `Person` 객체의 프로토타입은 어떤 상속이 일어났는지 확인해보면 다음과 같습니다.

```js
// person1 -> Person.prototype -> Object.prototype -> null

console.log("person1의 프로토타입 체인: ");
console.log(person1);
console.log(person1.__proto__); // Person.prototype
console.log(person1.__proto__.__proto__); // Object.prototype
console.log(person1.__proto__.__proto__.__proto__); // null
```

[prototype-chain.js](./example/prototype-chain.js)

그런데 위의 예제에서도 그렇듯 Object의 프로토타입을 상속시키지 않았는데, 왜 `Object.prototype이 상속`되고 있을까요?

그 이유는 **자바스크립트에서 모든 것이 객체**라는 철학을 가지고 있기 때문입니다.
모든 객체가 공통으로 가져야 할 **기본 기능**들을 제공할 공통 조상이 필요해 **Object.prototype이 기본적으로 들어가 있습니다**.

Object.prototype에서 제공해주는 속성과 함수를 살펴보면 다음과 같습니다.

```js
const obj = new Object();

console.log(Object.getOwnPropertyNames(obj.__proto__));
```

```js
// 실행 결과
[
  "constructor",
  "__defineGetter__",
  "__defineSetter__",
  "hasOwnProperty",
  "__lookupGetter__",
  "__lookupSetter__",
  "isPrototypeOf",
  "propertyIsEnumerable",
  "toString",
  "valueOf",
  "__proto__",
  "toLocaleString",
];
```

그래서 `Date`, `Array`, `Number` 등 여러 객체에서 공통적으로 보이거나 쓰이는 함수와 속성이 포함되어 있는 이유입니다.

```js
const date = new Date();
console.log(date.toString());

const array = new Array(1, 2, 3);
console.log(array.constructor.name);

const number = new Number("12345");
console.log(number.toLocaleString());

console.log();

function Person(name, age) {
  this.name = name;
  this.age = age;
}

const person1 = new Person("종원", 29);
console.log(person1.toString()); // [object Object]
```

[prototype-object.js](./example/prototype-object.js)

그런데 다른 객체들은 toString()이 잘 출력 되는데, 왜 person1의 toString()만 `[object Object]`로 나올까요?

이유는 다른 객체들이 각 객체에 맞는 함수(toString())로 오버라이드 되어 있기 때문입니다.

그래서 Person객체에서 toString()을 사용하고 싶다면 따로 오버라이드 해줘야 합니다.

```js
Person.prototype.toString = function () {
  return `이름: ${this.name}, 나이: ${this.age}`;
};
console.log(person1.toString());
```

[prototype-object.js](./example/prototype-object.js)

---

### 프로토타입은 어떤 객체와 연결될까?

- 모든 `객체`에는 각자에 맞는 `프로토타입`이 연결되어 있습니다.

```js
const div = document.createElement("div");

// HTMLElement
let proto = div;

// Div HTML Element Prototype Chain
while (proto) {
  console.log(proto.constructor.name);
  proto = Object.getPrototypeOf(proto);
}

// Div HTML Element Prototype Object Properties and Methods
console.log(Object.getPrototypeOf(div));
```

[div-prototype.html](./example/div-prototype.html)

HTML의 기본적인 태그들도 모두 `Object`로, 태그에 맞는 프로토타입을 상속받고 있습니다.

따라서 HTML 태그 `속성(Attribute)`들은 `프로토타입`에 상속받아 사용되고 있습니다.

대표적으로 `class`, `style`, `onload` 등의 속성 등이 있습니다.

`onload`같은 이벤트 핸들러(Event Handler)의 경우 함수를 **할당(assignment)**하여 이벤트를 사용할 수 있게 되는겁니다.

> 💡 **참고사항**
>
> `HTML DOM`에서 사용되는 속성과 `JS DOM`에서 사용되는 이름이 다른 경우가 있는데, 거의 대부분이 자바스크립트에서 `예약어`로 사용되고 있기 때문에 이름이 다르게 설계 된겁니다.
>
> 대표적으로 _DOM에서는 class로 사용_ 되지만, *JS DOM에서는 className으로 사용*됩니다.

<br/>

```js
const array = [];

console.log(array instanceof Array); // true

// Array.prototype
console.log(Object.getPrototypeOf(array));

// Object.prototype
console.log(Object.getOwnPropertyNames(Array.prototype));
```

[array-prototype.js](./example/array-prototype.js)

Array Object의 경우에도 `push`, `find`, `sort` 등의 함수를 사용할 수 있는 이유는 `Array` 프로토타입에 상속받고 있기 때문입니다.

<br/>

그렇다면 null도 객체(Object)인데 null에도 프로토타입이 있을까요?

```js
const _null = null;

console.log(typeof _null); // object
```

```js
console.log(_null instanceof Object); // false
```

null의 타입은 object지만, `_null instanceof Object`의 결과는 `false`입니다.

> instanceof는 `객체`가 *특정 생성자 함수로 만들어졌는지 확인*할 때 사용하는 연산자입니다.

이유는 과거 자바스크립트 초기 설계의 오류로 null을 내부적으로 객체처럼 처리하는 **언어적 결함**이 있습니다. 하지만 하위 호환성을 이유로 수정되지 않아 typeof null의 경우 object로 나옵니다.

null은 자바스크립트에서 string, number, bigint, boolean, undefined, symbol, null 총 `7개의 원시타입` 중 하나이지만, **값이 없는** `특수한 원시타입`으로 분류 돼서 다른 원시타입 처럼 객체를 만들수도 없고, 프로토타입을 만들수도 없습니다.

---

### class 문법과의 관계

> ES6 `class` 문법은 **프로토타입 기반**으로 동작

```js
// class
class Person {
  say() {
    console.log("안녕하세요");
  }
}

const man = new Person();
man.say();
```

```
> 실행 결과

"안녕하세요"
```

---

### 프로토타입을 사용하는 이유

#### 1. **메모리 효율성**

```js
function Person(name, age, address) {
  this.name = name;
  this.age = age;
  this.address = address;

  // ❌ Bad Case
  // 인스턴스마다 메모리에 새로운 함수 생성
  this.getInfo = function () {
    return `name: ${this.name}, age: ${this.age}, address: ${this.address}`;
  };
}

---

function Person(name, age, address) {
  this.name = name;
  this.age = age;
  this.address = address;
}

// ✅ Good Case
// 모든 인스턴스가 하나의 함수 메모리 주소를 참조
Person.prototype.getInfo = function() {
  return `name: ${this.name}, age: ${this.age}, address: ${this.address}`;
}
```

위 예시와 같이 객체에서 함수를 만들게 되면, 인스턴스를 생성(new Person) 할 때 마다 변경 사항이 없는 함수도 같이 메모리를 할당하게 됩니다.
하지만, 아래 예제와 같이 `Person.prototype.getInfo`로 하게 되면 **인스턴스를 생성하더라도 하나의 함수만 공유하기 때문에 메모리 효율적**입니다.

#### 2. **상속 구현**

```js
function Person(name) {
  this.name = name;
}

Person.prototype.walk = function () {
  console.log(`${this.name}이(가) 걷고 있습니다.`);
};

function Man(name, age) {
  Person.call(this, name); // Person 생성자를 Man의 컨텍스트에서 실행
  this.age = age;
}

Man.prototype = Object.create(Person.prototype);

const man = new Man("종원", 29);
man.walk(); // 종원이(가) 걷고 있습니다.
```

[prototype-inheritance.js](./example/prototype-inheritance.js)

프로토타입 체인을 통해 객체 간 상속 관계를 구현할 수 있고, 부모 객체의 속성과 함수를 자식 객체가 사용 가능하게 할 수 있습니다.

#### 3. **동적 확장성**

```js
const person1 = new Person("종원", 29);
const person2 = new Person("철수", 30);

Person.prototype.hello = function () {
  console.log(`안녕하세요. ${this.name}입니다!`);
};

person1.hello();
person2.hello();
```

런타임에 프로토타입 수정 시 모든 인스턴스(기존 객체 포함)에서 즉시 사용할 수 있게 됩니다.

#### 4. **코드 재사용성**

```js
// Object.prototype의 메서드들을 모든 객체가 공유
person1.toString();
person1.hasOwnProperty("name");
person1.valueOf();
```

한 번 프로토타입을 작성해두고, 상속을 하면 코드를 재작성 하지 않아도 해당 코드를 사용할 수 있어 코드의 재사용성을 높일 수 있다.

#### 5. **언어 차원의 통일성**

```js
const div = document.createElement("div");
const array = [1, 2, 3];
const date = new Date();
const person = new Person("종원", 29);

// 모든 객체가 동일한 기본 메서드를 가짐
console.log(div.toString()); // HTML 요소도 toString() 사용 가능
console.log(array.toString()); // 배열도 toString() 사용 가능
console.log(date.toString()); // 날짜도 toString() 사용 가능
console.log(person.toString()); // 커스텀 객체도 toString() 사용 가능

// 모든 객체에서 동일한 방식으로 타입 검사 가능
console.log(div instanceof Object); // true
console.log(array instanceof Object); // true
console.log(date instanceof Object); // true
console.log(person instanceof Object); // true
```

모든 객체가 Object.prototype을 최상위로 하는 **일관된 구조**를 가지므로, `toString()`같은 공통 함수를 사용할 수 있으며, **예측 가능한 구조**로 되어있어 어떤 객체든 프로토타입 체인을 따라 Object.prototype에 도달 가능합니다.
