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










