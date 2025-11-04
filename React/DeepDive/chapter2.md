## 2. 리액트 핵심 요소 깊게 살펴 보기 ( 목차 + 28P )

### 2.1 JSX 란?
- 보통 리액트를 통해 JSX를 접하기 때문에 JSX가 리액트의 전유물이라고 오해하는 경우가 종종 있다. 
<br> 이는 반은 맞고 반은 틀리다.
<br> JSX는 리액트가 등장하면서 메타에서 소개한 새로운 구문이지만 반드시 리액트에서만 사용하라는 법은 없다.
<br> JSX는 흔히 개발자들이 알고 있는 XML과 유사한 내장형 구문이며, 리액트에 종속적이지 않은 독자적인 문법으로 보는 것이 옳다.
- 더불어 메타에서 독자적으로 개발했다는 사실에서 미루어 알 수 있듯 ECMAScript 라고 불리는 Js 표준의 일부는 아니다.

- JSX 는 반드시 트랜스파일러를 거쳐야 비로소 JS 런타임이 이해할 수 있는 의미 있는 JS 코드로 변환된다.
- JSX는 HTML이나 XML을 JS 내부에 표현하는 것이 유일한 목적은 아니다. 
<br>JSX의 설계 목적은 다양한 트랜스파일러에서 다양한 속성을 가진 트리 구조를 토큰화하여 ECMAScript 로 변환하는데 초점을 두고 있다.

- 요약하자면 JSX는 JS 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는 데 많은 도움을 주는 새로운 문법이라고 볼 수 있다.


#### 2.1.1 JSX의 정의 
- JSX는 기본적으로 JSXElement, JSXAttributes, JSXChildren, JSXStrings 라는 4가지 컴포넌트를 기반으로 구성되어 있다.
    - #### JSXElement
      - HTML의 요소와 비슷한 역할, JSXElement가 되기 위해서는 다음과 같은 형태 중 하나여야 한다.
      - JSXOpeningElement, JSXClosingElement, JSXSelfClosingElement, JSXFragment
      - JSXElementName: JSXIdentifier, JSXNamespacedName, JSXMemberExpression
      - ```jsx
        /** JSXNamespacedName 예제 */
        function valid () {
            return <foo:bar></foo:bar>
        }    
        ``` 
      - ```jsx
        /** JSXMemberExpression 예제 */
        function valid () {
            return <foo.bar></foo.bar>
        }
        ``` 
    - #### 요소명은 대문자로 시작해야만 되는 거 아닌가?
    - HTML 구문 이외에 사용자가 컴포넌트를 만들어 사용할 때에는 반드시 대문자로 시작하는 컴포넌트를 만들어야지만 사용 가능하다.
      <br> 이는 JSXElement에 명시되어 있는 표준에 없는 내용인데, 그 이유는 리액트에서 HTML 태그명과 사용자가 만든 컴포넌트 명을 구분 짓기 위함이다.
 
    - #### JSXAttributes
        - JSXElement에 부여할 수 있는 속성을 의미, 단순히 속성을 의미하기 때문에 모든 경우에서 필수값이 아니고, 존재하지 않아도 에러가 발생하지 않는다.
        - JSXSpreadAttributes: JS의 객체 전개 연산자(SpreadOperator)와 동일
        - JSXAttribute: 속성을 나타내는 키와 값으로 표현
          - JSXAttributeName: 속성의 키 값, JSXIdentifier 와 JSXNamespacedName 할당 가능
          - ```jsx
            function valid () {
                return <foo.bar foo:bar="baz"></foo.bar>
            }
            ```
        - JSXAttribute: 특이점으로는 값으로 다른 JSX 요소를 할당할 수 있다는 정도
    
    - #### JSXChildren
    - JSXElement의 자식값을 나타낸다. JSX는 속성을 가진 트리 구조를 나타내기 위해 만들어졌기 때문에 JSX로 부모와 자식 관계를 나타낼 수 있으며, 그 자식을 JSXChildren 이라고 한다.
  
    - #### JSXStrings
    - JS와 한 가지 주요한 차이점은 \로 시작하는 이스케이프 문자 형태소를 React에서는 아무런 제약 없이 사용할 수 있다.

#### 2.1.2 JSX 예제 (생략)

#### 2.1.3 JSX는 어떻게 JS에서 변환될까
- 우선 JS에서 JSX가 변환되는 방식을 알려면 @babel/plugin-transform-reac-jsx 플러그인을 알아야 한다. 

- 다음과 같은 JSX 코드가 있고 가정해 보자.
```jsx
const ComponentA = <A required={true}> Hello Wolrd </A>
const ComponentB = <>Hello World</>

const ComponentC = (
    <div>
        <span>hello world</span>
    </div>
)
```
- 이를 변환한 결과는 다음과 같다.
```jsx
'use strict'
var ComponentA = React.createElement(
    A,
    {
        required: true,
    },
    "Hello World",
)

var ComponentB = React.createElement(React.Fragment, null, "Hello World");

var ComponentC = React.createElement(
    'div',
    null,
    React.createElement('span', null, "hello world"),
)
```
- React 17, 바벨 7.9.0 이후 버전에서 추가된 자동 런타임으로 트랜스파일한 결과는 다음과 같다

```jsx
'use strict'

var _jsxRuntime = require('custom-jsx-library/jsx-runtime');

var ComponentA = (0, _jsxRuntime.jsx)(A,{
    required: true,
    children: "Hello World",
});

var ComponentB = (0, _jsxRuntime.jsx)(_jsxRuntime.Fragment,{
    children: "Hello World",
});

var ComponentC = (0, _jsxRuntime.jsx)('div', {
    children: (0, _jsxRuntime.jsx)('span',{
        children: "hello world"
    })
})
```

### 2.2 Virtual DOM(가상 DOM) 과 React Fiber (리액트 파이버)

#### 2.2.1 DOM과 브라우저 렌더링 과정
- DOM은 웹페이지에 대한 인터페이스로 브라우저가 웹페이지의 콘텐츠와 구조를 어떻게 보여줄지에 대한 정보를 담고 있다.
<br> 본격적인 DOM의 정의를 다루기에 앞서 브라우저가 웹사이트 접근 요청을 받은 이후 화면을 그리는 과정에서 어떠한 일이 일어나는지 살펴보자. 

1. 브라우저가 사용자가 요청한 주소를 방문해 HTML 파일을 다운로드
2. 브라우저의 렌더링 엔진은 HTML을 파싱해 DOM 노드로 구성된 (DOM)트리를 만든다.
3. 2번 과정에서 CSS 파일을 만나면 해당 CSS 파일도 다운로드
4. 브라우저의 렌더링 엔진은 이 CSS도 파싱해 CSS 노드로 구성된 (CSSOM)트리를 만든다.
5. 브라우저는 2번 과정에서 만든 DOM 노드를 순회하는데, 여기서 모든 노드를 방문하는 것이 아니고 사용자 눈에 보이는 노드만 방문한다.
   <br> 즉, display: none 과 같이 사용자 화면에 보이지 않는 요소는 방문해 작업하지 않는다.  
6. 5번에서 제외되지 않은, 눈에 보이는 노드를 대상으로 해당 노드에 대한 CSSOM 정보를 찾고 여기서 발견한 CSS 스타일 정보를
<br> 이 노드에 적용한다. 이 DOM노드에 CSS를 적용하는 과정은 크게 두 가지로 나눌 수 있다. 
    - **레이아웃(layout, reflow)**: 각 노드가 브라우저 화면의 어느 좌표에 정확히 나타나야 하는지 계산하는 과정,
    <br> 이 레이아웃 과정을 거치면 반드시 페인팅 과정도 거치게 된다.
    - **페인팅 (painting)**: 레이아웃 단계를 거친 노드에 색과 같은 실제 유효한 모습을 그리는 과정

#### 2.2.2 Virtual DOM의 탄생 배경
- 브라우저가 웹 페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 든다. 
<br/> 또한 요즘 대다수의 앱은 렌더링된 이후 정보를 보여주는 데 그치치 않고 사용자의 인터랙션을 통해 다양한 정보를 노출하기 때문에
<br/> 렌더링이 완료된 이후에도 사용자의 인터랙션으로 웹페이지가 변경되는 상황 또한 고려해야한다..

- 특정한 요소의 색상이 변경되는 경우
  - 이때는 ```페인팅(painting)``` 과정만 진행되므로 비교적 빠르게 처리할 수 있다.
- 어떤 특정 요소의 노출 여부가 변경되거나 사이즈가 변경되는 등 요소의 위치와 크기를 재계산하는 경우
  - 이 경우에는 ```레이아웃(layout, reflow)``` 이 발생하고 이어서 필연적으로 ```리페인팅```이 발생하기 때문에 더 많은 비용이 든다.

- 가상 DOM은 웹 페이지가 표시해야 할 DOM을 일단 메모리에 저장하고 리액트가 실제 변경에 대한 준비가 완료되었을 때 실제 브라우저의 DOM에 반영한다.
  - (여기서 말하는 리액트는 정확하게 이야기하자면 package.json에 있는 react가 아닌 react-dom을 의미) 

#### 2.2.3 가상DOM을 위한 아키텍처, 리액트 파이버
- 가상 DOM과 렌더링 과정 최적화를 가능하게 해주는 것이 바로 리액트 파이버(React Fiber)다.

#### 리액트 파이버(React Fiber)란?
- 리액트에서 관리하는 평범한 자바스크립트 객체다. 파이버 재조정자(fiber reconciler)라는 패키지가 관리하는데, 이는 앞서 이야기한 가상 DOM과 실제 DOM을 비교해 변경 사항을 수집하며,
<br/>만약 이 둘 사이에 차이가 있다면 변경에 관련된 정보를 가지고 있는 파이버 객체를 기준으로 화면에 렌더링을 요청하는 역할을 한다.

- 리액트 파이버의 목표는 리액트 어플리케이션에서 발생하는 애니메이션, 레이아웃, 그리고 사용자 인터랙션에 올바른 결과물을 만드는 반응성 문제를 해결하는 것이다. 이를 위해 파이버는 다음과 같은 일을 할 수 있다.
  - 작업을 작은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다.
  - 이러한 작업을 일시 중지하고 나중에 다시 시작할 수 있다.
  - 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우에는 폐기할 수 있다.
- ```그리고 한 가지 중요한 것은 이러한 모든 과정이 비동기로 일어난다는 것이다!!``` 과거 리액트의 조정 알고리즘은 스택 알고리즘으로 이루어져 있었다.
- 이 하나의 스택에 렌더링에 필요한 작업들이 쌓이면 이 스택이 빌 때까지 동기적으로 작업이 이루어졌다.
- 싱글 스레드 기반인 JS의 특징으로 인해 이 동기적인 작업은 중단될 수 없었고, 이는 비효율성으로 이루어졌다.

- 그렇다면 파이버는 어떻게 구현되어 있을까? 파이버는 일단 하나의 작업 단위로 구성되어 있다. 리액트는 이러한 작업 단위를 하나씩 처리하고 finishWork()라는 작업으로 마무리한다.
<br/> 그리고 이 작업을 커밋해 실제 브라우저 DOM에 가시적인 변경 사항을 만들어 낸다. 그리고 이러한 단계는 아래 두 단계로 나눌 수 있다.
  1. 렌더 단계에서 리액트는 사용자에게 노출되지 않는 모든 비동기 작업을 수행한다. 그리고 이 단계에서 앞서 언급한 파이버의 작업, 우선순위를 지정하거나 중지시키거나 버리는 등의 작업이 일어난다.
  2. 커밋 단계에서는 앞서 언급한 것처럼 DOM에 실제 변경 사항을 반영하기 위한 작업, commitWork()가 실행되는데, 이 과정은 앞서와 다르게 동기식으로 일어나고 중단될 수도 없다.

## 더 알아보기

### 💥 React19 에서의 JSX 트랜스파일링
- 19 에서 Babel 은 기본적으로 "새로운 JSX 트랜스폼" (automatic runtime)을 사용해 JSX를 ```react/jsx-runtime``` 의 함수 호출로 변환
- 개발 빌드에서는 ```jsxDEV```, 프로덕션 빌드에서는 ```jsx/jsxs```가 사용된다.

```jsx
/** 프로덕션 변환 결과 */

/* input */
function App() {
    return <h1>Hello</h1>
}

import { jsx as _jsx } from "react/jsx-runtime";

/* output */
function App() {
  return _jsx("h1", { children: "Hello" });
}
```
- babel 설정을 고의로 ```runtime:"classic"``` 으로 바꾸면 예전처럼 ```React.createElement(...)```로도 변환할 수는 있으나
<br> React 19 권장 사항에 어긋남