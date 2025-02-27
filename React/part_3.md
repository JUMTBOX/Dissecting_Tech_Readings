# VDOM 과 React lifecycle

### VDOM

- 프로그래밍 컨셉 - 메모리 상에 UI와 관련된 정보를 띄우고, react-dom 과 같은 라이브러리에 의해 실제 DOM과 sync 를 맞춘다. (renderer 관여)
  <br>이 과정을 재조정이라 부른다. (reconciler 관여)
- 왜 가상인가? 실제 DOM 조작 시 (mount -> paint) 소모되는 비용이 더 크다.

#### 어떻게 구현되어 있는가

- fiber node 로 구성된 tree 형태 (더블 버퍼링 구조)

<p align="center">
    <img src="../images/React/part_3/VDOM.png" , height="500px", width="350px">
</p>

- current tree
  - DOM 에 mount 된 fiber 노드들을 일컫는다.
  - 실제 HTML 태그에 적용이 되어 있는 정보들을 담고 있는 fiber 노드로 구성된 tree
- workInProgress tree

  - render phase 에서 작업 중인 fiber
  - commit phase 에서 작업 중인 fiber

- 구현 상세
  - workInProgress tree 는 current tree 에서 자기 복제 하여 만들어짐 (서로 alternate 로 참조)
  - fiber는 첫번째 자식만을 child 로 참조, 나머지 자식들은 서로 sibling 으로 참조, 모든 자식은 부모를 return 으로 참조
- 컴포넌트 리렌더링: 컴포넌트 호출 이후 그 결과가 VDOM 에 반영 (여기까지를 렌더링이라 부르기로 약속 )
  <br>DOM 에 mount 되어 브라우저에서 paint => X

### React lifecycle

<p align="center">
    <img src="../images/React/part_3/lifecycle.png" , height="350px", width="550px">
</p>

#### render phase

- VDOM 재조정 (reconciliation)하는 단계
  - element (fiber node 객체) 추가, 수정, 삭제 되는 과정 -> WORK 를 scheduler 에 등록
    - WORK: reconciler 가 컴포넌트의 변경을 DOM 에 적용하기 위해 수행하는 일
  - reconciler가 담당 ( reconciler 설계 stack -> fiber 로 바뀌면서 abort, stop, restart <br> 즉, 렌더링 우선순위 변경이 가능해졌다. ex - useTransition hook )

#### commit phase

- 재조정한 VDOM 을 DOM 에 적용 & 라이프사이클을 실행하는 단계
  - 일관성을 위해 sync 실행 .. 즉, DOM 조작 일괄처리 후, 리액트가 콜스택을 비워준 다음
    <br> 브라우저가 paint 시작..
