# React 패키지의 구성 요소들 & 용어 정의

### Recat 패키지 구성 요소

- #### react 코어
  - component 정의
  - 다른 패키지에 의존성이 없으므로 다양한 플랫폼 (브라우저, 모바일)에 올려서 사용 가능
- #### renderer
  - react-dom, react-native-renderer 등 호스트 렌더링 환경에 의존
  - 호스트와 react 를 연결 즉, 웹에서는 DOM 조작
  - reconciler 와 legacy-events 패키지 의존성
- #### evnet(legacy-events)
  - https://legacy.reactjs.org/docs/events.html
  - SyntheticEvent 라는 이름의 내부적으로 개발된 이벤트 시스템
  - 기존 웹에서의 event 를 wrapping 하여 추가적인 기능을 수행할 수 있도록 만듦
- #### scheduler
  - react 는 Task 를 비동기로 실행한다. 이 Task 를 실행하는 타이밍을 알고 있는 패키지
- #### reconciler
  - fiber 아키텍쳐에서 VDOM 재조정 담당
  - 컴포넌트를 호출하는 곳

### 용어

- #### 렌더링

  - 컴포넌트를 호출하여 react element 를 반환 -> Virtual DOM 에 적용 (재조정)하는 과정
  - 전체 과정
    - 1. 컴포넌트 호출 -> return react element
    - 2. VDOM 재조정 작업 ( 여기까지가 렌더링 )
    - 3. renderer 가 컴포넌트 정보를 DOM 에 삽입 (mount)
    - 4. 브라우저가 DOM을 paint

- #### 💥 react element

  - 컴포넌트 호출시에 return 하는 것 (JSX -> babel을 통해 react.createElement 를 호출 )
  - 컴포넌트의 정보 (결국 DOM에 삽입될 내용)를 담은 객체
    - type, key, props, ref 등의 정보

- #### fiber
  - VDOM 의 노드 객체 ( architecture 이름과 동일 )
  - react element의 내용이 DOM 에 반영되기 위해, 먼저 VDOM 에 추가되어야 하는데, <br>
    이를 위해 react element 를 확장한 객체
    - 이 fiber 객체에서 컴포넌트의 상태, life cycle, hook 이 관리된다.

## 더 알아보기

### 💥 About_React_Element

#### Managing Instance

- 기존 UI 모델의 단점 (OOP)

  - 관리할 컴포넌트의 수가 늘어나면 코드의 복잡도 증가
  - 자식 컴포넌트에게 부모 컴포넌트가 직접 접근하므로 결합도가 높아진다. (decouple 어려움)

#### Elements Describe the Trees

- Elements
  - 컴포넌트 인스턴스를 설명하며, DOM node 의 정보를 가지고 있는 plain 객체
  - "\*인스턴스가 아니며" , 리액트에게 화면에 무언가를 그려보라고 전달하는 역할
  - 두 가지의 프로퍼티(type: string | ReactClass, props: object)를 가지는 불변 객체
    - type 프로퍼티에 string 타입의 값이 할당 되는 경우 -> DOM elements 인 경우
    - type 프로퍼티에 ReactClass 타입의 값이 할당 되는 경우 -> Component elements 인 경우

```jsx
const DeleteAccount = () => ({
  type: "div",
  props: {
    children: [
      {
        type: "p",
        props: {
          children: "Are you sure?",
        },
      },
      {
        type: DangerButton,
        props: {
          children: "Yep",
        },
      },
      {
        type: Button,
        props: {
          color: "blue",
          children: "Cancel",
        },
      },
    ],
  },
});

const DeleteAccount = () => {
  return (
    <div>
      <p>Are you sure?</p>
      <DangerButton>Yep</DangerButton>
      <Button color="blue">Cancel</Button>
    </div>
  );
};
```

- element 에 의해 기존의 DOM node 와 React Component를 mixed , nested 한 구조가 가능해짐
- 그 결과, component 끼리 decoupled 가 된다.

#### Components Encapsulate Element Trees

- React 가 type 값이 class or functional 인 element(React Component element)를 만나면

  - 1. type 값을 보고, 해당 컴포넌트 함수에게 element 를 return 받는다.
  - 2. return 받은 element의 type 값이 tag name 인 element 를 만날때까지 1번으로 돌아간다.

- 위 과정을 거치면 모든 component elements 가 가진 DOM elements 를 알게 되고, React 가 해당 DOM elements 들을 적절한 때에 create, update, destroy 할 수 있게 된다.

#### Top-Down Reconcillation

```jsx
ReactDOM.render(
  {
    type: Form,
    props: {
      isSubmitted: false,
      buttonText: "OK!",
    },
  },
  document.getElementById("root")
);
```

- 위 함수 호출 시, React는 Form component 에게 props 를 전달하며 return element 를 요청
  <br> Form -> Button -> DOM node element (button) 순서로 return element 진행

- ReactDom.render, setState 호출 시 React call reconcilation <br>
  reconcilation 이 끝나면 React 는 결과물로 나온 DOM tree 를 알게 되고, <br>
  renderer(e.g. react-dom)는 필수적인 최소한의 변화를 DOM node 업데이트에 적용한다.

- 점진적인 변화 덕분에 쉽게 최적화가 가능해지고, props 가 불변이라면 변화 계산이 더 빨라진다.

### Summary

- element는 DOM Tree 생성에 필요한 정보를 담은 object 이며
  <br> React DOM node element, React Component element 두 종류가 존재한다.
- element는 property 로 다른 element 를 가질 수 있다.
  <br> DOM node 와 React component 를 mixed, nested 가능하게 함
- component 는 element를 반환(return)하는 함수이다.
- 부모 컴포넌트가 return 한 element 의 type이 자식 컴포넌트 이름일 때,
  <br> props는 해당 자식 컴포넌트의 input(props) 가 된다. -> props 는 일방향 통행
- React.createElement, JSX, React.createFactory 는 element 를 반환한다.
