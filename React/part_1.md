### Fiber vs Stack 18v

#### fiber가 뭔데?

- 리액트 v16 에서 처음 등장한 아키텍쳐 ( 이전에는 stack 아키텍쳐 )

- fiber 는 스택과 달리 가장 나중에 들어간 fiber 노드를 가장 먼저 꺼낼 필요가 없음 ...
  들어간 순서와 무관하게 꺼낼 수 있는 것이 fiber 아키텍쳐

<p align="center">
    <img src="../images/React/part_1/fiber.png" , height="150px", width="250px">
</p>

- 현재 문맥에서 "꺼낸다"라는 의미는 리액트에서의 렌더링(DOM에 적용한다)의 의미
