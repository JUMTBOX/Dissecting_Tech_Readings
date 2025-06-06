## 1. 리액트 개발을 위해 꼭 알아야 할 자바스크립트 ( 목차 + 28P )

### 1.1 자바스크립트의 동등 비교

- 리액트 함수 컴포넌트와 훅을 반복적으로 작성하다 보면 의존성 배열에 대해 고민해 본 적이 있을 것이다. 보통은 리액트에서 제공하는 eslint-react-config 에 선언되어 있는 react-hooks/exhaustive-deps 의 도움을 받아 해당 배열을 채우곤 하는데, 실제로 이것이 어떤 식으로 작동하는지, 또한 왜 이러한 변수들을 넣어야 하는지 이해하지 못하는 경우가 많다. 또 리액트 컴포넌트의 렌더링이 발생하는 이유 중 하나가 바로 props 의 동등 비교에 따른 결과다. 그리고 이 props 의 동등 비교는 객체의 얕은 비교를 기반으로 이루어지는데, 이 얕은 비교가 리액트에서 어떻게 작동하는지 이해하지 못하면 렌더링 최적화에 어려움을 겪을 가능성이 크다.

- 리액트의 가상 DOM 과 실제 DOM 의 비교, 리액트 컴포넌트가 렌더링할지를 판단하는 방법, 변수나 함수의 메모이제이션 등 모든 작업은 자바스크립트의 동등 비교를 기반으로 한다. 자바스크립트의 이러한 동등 비교는 어떻게 수행되는지, 또 이를 리액트에서 어떻게 활용하고 있는지 살펴보자.

#### 1.1.1 자바스크립트의 데이터 타입

- Primitive type (원시 타입) - typeof 키워드드로 타입 검사시 사용 가능

  - boolean, null, undefined, number, string, symbol, bigint
  - ```jsx
    const isUndefined = undefined;
    const isString = "str";

    console.log(typeof isUndefined === "undefined"); // true
    console.log(typeof isString === "number"); // false
    ```

- Object/reference type (객체/참조 타입)

  - object

- 원시타입

  - 자바스크립트에서 원시 타입이란 객체가 아닌 다른 모든 타입을 의미, 객체가 아니므로 이러한 타입들은 메서드를 가지지 않는다. (\*래퍼 객체 개념...)
  - ES2022 와 같은 최신 자바스크립트에서는 총 7개의 원시 타입이 있다.

- undefined

  - 선언한 이후 값을 할당하지 않은 변수 또는 값이 주어지지 않은 인수(parameter)에 자동으로 할당되는 값이다.
  - 후술할 원시 값 중 null 과 undefined 는 오직 각각 null 과 undefined라는 값만 가질 수 있으며, 그 밖의 타입은 가질 수 있는 값이 두개 이상 존재한다.

- null

  - 아직 값이 없거나 비어 있는 값을 표현할 때 사용한다.

  - ```jsx
    typeof null === "object"; // true?
    ```
  - 특별한 점 하나는 다른 원시값과 다르게 typeof 로 null을 확인했을 때 null 타입이 아닌 "object"라는 결과가 반환된다는 것이다.
  - undefined는 '선언됐지만 할당되지 않은 값'이고, null은 '명시적으로 비어 있음을 나타내는 값'으로 사용하는 것이 일반적이다.

- Symbol
  - ES6 에서 새롭게 추가된 7번째 타입으로, 중복되지 않는 어떠한 고유한 값을 나타내기 위해 만들어졌다. 심벌은 심벌 함수를 이용해서만 만들 수 있다.
  - ```jsx
    // Symbol 함수에 같은 인수를 넘겨주더라도 이는 동일한 값으로 인정되지 않는다.
    // symbol 함수 내부에 넘겨주는 값은 Symbol 생성에 영향을 미치지 않는다 (Symbol.for 제외)
    const key = Symbol("key");
    const key2 = Symbol("key");
    ```

#### 1.1.2 값을 저장하는 방식의 차이

- 원시 타입과 객체 타입의 가장 큰 차이점이라고 한다면, 바로 값을 저장하는 방식의 차이다. </br>
  이 값을 저장하는 방식의 차이가 동등 비교를 할 때 차이를 만드는 원인이 된다.

- 먼저 원시 타입은 불변 형태의 값으로 저장되고, 이 값은 변수 할당 시점에 메모리 영역을 차지하고 저장된다.

```jsx
let h = "hello world";
let h2 = "hello world";
console.log(h === h2); // true
```

- 반면 객체는 프로퍼티를 추가, 삭제, 수정할 수 있으므로 원시 값과 다르게 변경 가능한 형태로 저장되며, 값을 복사할 때도 값이 아닌 참조를 전달하게 된다.

```jsx
let h = {
  greet: "hello world",
};

let h2 = {
  greet: "hello world",
};
// 참조 타입인 두 객체를 비교하면 false
console.log(h === h2); // false
// 원시 값인 내부 greet 속성값을 비교하면 true
console.log(h.greet === h2.greet); // true
```

- 객체는 값을 저장하는 것이 아니라 참조를 저장하기 때문에 앞서 동일하게 선언했던 객체라 하더라도 저장하는 순간 다른 참조를 바라보기 때문에 false 를 반환한다.

#### 1.1.3 자바스크립트의 또 다른 비교 공식, Object.is

- 자바스크립트에서는 비교를 위한 또 한 가지 방법을 제공하는데, 바로 Object.is 다. <br/>
  Object.is 가 == 나 === 와 다른 점은 다음과 같다.

- == vs Object.is

  - == 비교는 같음을 비교하기 전에 양쪽이 같은 타입이 아니라면 비교할 수 있도록 강제로 형 변환(type casting)을 한 후에 비교한다. 하지만 Object.is 는 이러한 작업을 하지 않는다.

- === vs Object.is

  - 이 방식에도 차이가 있다. 예제를 보면 알 수 있듯, Object.is 가 좀 더 개발자가 기대하는 방식으로 정확히 비교한다.

  - ```jsx
    -0 === +0; // true
    Object.is(-0, +0); // false

    Number.NaN === NaN; // false
    Object.is(Number.NaN, NaN); // true

    NaN === 0 / 0; //false
    Object.is(NaN, 0 / 0); // true
    ```

  - 이렇듯 == 과 ===가 만족하지 못하는 몇 가지 특이 케이스를 추가하기 위해, Object.is가 나름의 알고리즘으로 작동하는 것을 알 수 있다. 한 가지 주의해야 할 점은, Object.is를 사용한다 하더라도 객체 비교에는 별 차이가 없다는 것이다.
  - ```jsx
    Object.is({}, {}); // false

    const a = { hello: "hi" };
    const b = a;

    Object.is(a, b); //true
    a === b; //true
    ```

  - Object.is 는 ES6 에서 새로이 도입된 비교 문법으로, 위와 같이 몇 가지 특별한 사항에서 동등 비교 === 가 가지는 한계를 극복하기 위해 만들어졌다. 그러나 여전히 객체 간 비교에 있어서는 자바스크립트의 특징으로 인해 ===와 동일하게 동작하는 것을 알 수 있다.

#### 1.1.4 리액트에서의 동등 비교

- 그렇다면 리액트에서는 동등 비교가 어떻게 이루어질까? 리액트에서 사용하는 동등 비교는 Object.is 이다. ES6 에서 제공하는 기능이기 때문에 리액트에서는 이를 구현한 폴리필(polyfill)을 함께 사용한다.

```tsx
function is(x: any, y: any) {
  return (
    // eslint-disable-line no-self-compare
    (x === y && (x !== 0 || 1 / x === 1 / y)) || (x !== x && y !== y)
  );
}
// 런타임에 Object.is 가 있다면 그것을 사용하고, 아니라면 위에서 정의한 함수를 사용
const objectIs: (x: any, y: any) => boolean =
  typeof Object.is === "function" ? Object.is : is;

export default objectIs;
```

- 리액트에서는 이 Object.is 를 기반으로 동등 비교를 하는 shallowEqual 이라는 함수를 만들어 사용한다. 이 shallowEqual은 의존성 비교 등 리액트의 동등 비교가 필요한 다양한 곳에서 사용된다.

```tsx
const { hasOwnProperty, is, keys } = Object;

function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) {
    return true;
  }

  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }
  // 각 키 배열을 꺼낸다.
  const [keysA, keysB] = [keys(objA), keys(objB)];
  // 배열의 길이가 다르다면 false
  if (keysA.length !== keysB.length) {
    return false;
  }

  for (let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i];
    if (
      !hasOwnProperty.call(objB, currentKey) ||
      !is(objA[currentKey], objB[currentKey])
    ) {
      return false;
    }
  }
  return true;
}
```

- 리액트에서의 비교를 요약하자면 Object.is로 먼저 비교를 수행한 다음에 Object.is에서 수행하지 못하는 비교, 즉 객체 간 얕은 비교를 한번 더 수행하는 것을 알 수 있다. 객체 간 얕은 비교란 객체의 첫 번째 깊이에 존재하는 속성 값만 비교한다는 것을 의미한다.

- 이렇게 객체의 얕은 비교까지만 구현한 이유는 무엇일까? 먼저 리액트에서 사용하는 JSX props는 객체이고, 그리고 여기에 있는 props만 일차적으로 비교하면 되기 때문이다. 예제 코드를 보자

```tsx
type Props = {
  hello: string;
};

function HelloComponent(props: Props) {
  return <h1>{props.hello}</h1>;
}

// ...

function App() {
  return <HelloComponent hello={"hi"} />;
}
```

- 위 코드에서 props 는 객체다. 그리고 기본적으로 리액트는 props 에서 꺼내온 값을 기준으로 렌더링을 수행하기 때문에 일반적인 케이스에서는 얕은 비교로 충분하다. 이러한 특성을 안다면 props에 또 다른 객체를 넘겨준다면 리액트 렌더링이 예상치 못하게 동작한다는 것을 알 수 있다.

#### 1.1.5 정리

### 1.2 함수

#### 1.2.1 함수란 무엇인가?

#### 1.2.2 함수를 정의하는 4가지 방법

#### 1.2.3 다양한 함수 살펴보기

#### 1.2.4 함수를 만들 때 주의해야 할 사항

#### 1.2.5 정리

### 1.3 클래스

#### 1.3.1 클래스란 무엇인가?

#### 1.3.2 클래스와 함수의 관계

#### 1.3.3 정리

### 1.4 클로저

#### 1.4.1 클로저의 정의

#### 1.4.2 변수의 유효 범위, 스코프

#### 1.4.3 클로저의 활용

#### 1.4.4 주의할 점

#### 1.4.5 정리

### 1.5 이벤트 루프와 비동기 통신의 이해

#### 1.5.1 싱글 스레드 자바 스크립트

#### 1.5.2 이벤트 루프란?

#### 1.5.3 태스크 큐와 마이크로 태스크 큐

#### 1.5.4 정리

### 1.6 리액트에서 자주 사용하는 자바스크립트 문법

#### 1.6.1 구조 분해 할당

#### 1.6.2 전개 구문

#### 1.6.3 객체 초기자

#### 1.6.4 Array 프로토 타입의 메서드: map, filter, reduce, forEach

#### 1.6.5 삼항 조건 연산자

#### 1.6.6 정리

### 1.7 선택이 아닌 필수, 타입스크립트

#### 1.7.1 타입스크립트란?

#### 1.7.2 리액트 코드를 효과적으로 작성하기 위한 타입스크립트 활용법

#### 1.7.3 타입스크립트 전환 가이드

#### 1.7.4 정리

```

```

```

```
