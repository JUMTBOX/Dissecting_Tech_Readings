# Fiber vs Stack 18v

### fiber가 뭔데?

- 리액트 v16 에서 처음 등장한 아키텍쳐 ( 이전에는 stack 아키텍쳐 )

- fiber 는 스택과 달리 가장 나중에 들어간 fiber 노드를 가장 먼저 꺼낼 필요가 없음 ...
  들어간 순서와 무관하게 꺼낼 수 있는 것이 fiber 아키텍쳐

<p align="center">
    <img src="../images/React/part_1/fiber.png" , height="200px", width="350px">
</p>

- 현재 문맥에서 "꺼낸다"라는 의미는 리액트에서의 렌더링(DOM에 적용한다)의 의미

- 리액트 v18부터 fiber 아키텍쳐가 본격적으로 사용되기 시작했다! ( <b>💥useTransition</b> Hook의 등장과 함께!)

### fiber 를 파악하기 위해 구조 분석 ...

- useState 훅의 코드레벨부터 <br>
  즉, 리액트 훅의 동작을 분석하면서 리액트 패키지 내부 구조 까보기 -> fiber 파악!

- 개요
  - 리액트라는 큰 그림을 대략 설명, 이후 각 패키지를 좀 더 자세히 살펴보며 <br>
    패키지들끼리의 상호작용을 설명.
  - 리액트의 패키지 구성 요소들 , react, shared, scheduler, reconciler 등
  - 각 구성 요소들이 어떻게 상호작용하여 hook 을 실행하는지
    - useState, useReducer, useEffect, useLayoutEffect 등 hook 부터 시작
    - hoo -> scheduler -> reconciler

## 더 알아보기

### 💥 useTransition 훅

- UI를 block시키지 않고 상태(state)를 업데이트할 수 있게 해주는 리액트 훅이다.
- 리액트 버전18에서 소개된 Transitions라는 개념에 대해서 이해할 필요가 있다.

- Transitions은 리액트에서 긴급한 업데이트와 긴급하지 않은 업데이트를 구분해서 처리하기 위한 새로운 개념

- 긴급한 업데이트와 긴급하지 않은 업데이트의 예시

  - 긴급한 업데이트 <br>
    사용자와 직접적인 인터랙션이 일어나는 경우 (ex. 글자 입력, 버튼 클릭 등)
  - 긴급하지 않은 업데이트 <br>
    사용자와 직접적인 인터랙션이 일어나지 않는 경우 (ex. 서버로부터 결과를 받아와서 보여주는 경우)

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState("about");

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

- 리액트 버전18에서는 이렇게 업데이트의 종류를 나누고 긴급한 업데이트를 먼저 처리함으로써 사용자에게 더 나은 경험을 제공하도록 한 것

- 참고로 Transition 업데이트는 긴급하지 않은 것으로 처리되기 때문에 그 사이에 더 긴급한 업데이트가 들어오면 중단될 수 있다.
