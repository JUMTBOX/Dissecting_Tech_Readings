## React의 비동기 작업은 어떻게 구현되어 있을까? (작성 중...)
- 리액트 패키지가 담당하는 작업 중 `비동기적으로 동작`하는 작업은 옛날부터 존재해왔었다.
  - 리액트 이벤트 핸들러 안에서의 `setState` 배치 처리, `useEffect`의 실행 시점이 있다. ( 이것들에 관해서도 나중에 자세히 다룰 예정.. ) 
- 그러나 여기서 다루는 주제는 정확하게는 리액트 18부터 도입된 `동시성 렌더링`에 관한 이야기이다.

```jsx
/* 
* 진짜 구현은 'performance API'가 없는 상황을 대비해 
* Date.now() 가지고 동작하는 폴리필까지 있지만 그냥 간단하게..
*/
const getCurrentTime = () => {
    return performance.now();
}

const performWorkUntilDeadline = () => {
  if (scheduledHostCallback !== null) { 
    const currentTime = getCurrentTime();
    /* 이번 작업의 타임슬라이스가 끝나는 시간 */
    deadline = currentTime + yieldInterval; 
    const hasTimeRemaining = true;

    let hasMoreWork = false;
    try {
      /* scheduledHostCallback 내부에서 flushWork() 실행 그 내부에서 workLoop() 실행됨 */
      hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
    } finally {
      if (hasMoreWork) {
        /* 작업이 남음 .. 다음 메시지 이벤트에서 다시 자기 자신을 호출하도록 예약하는 함수 */
        schedulePerformWorkUntilDeadline();
      } else {
        /* 작업이 없으면 루프 종료 */
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      }
    }
  } else {
    isMessageLoopRunning = false;
  }
  needsPaint = false;
};
```
```jsx
/**
* @description 리액트 스케줄러 패키지의 workLoop 구현 코드
* @param {number} initialTime 
*/
function workLoop(initialTime) {
    // 중략..
    
    while( 
        currentTask !== null &&
        !( enableSchedulerDebugging && isSchedulerPaused )
    ) {
        if( currentTask.expirationTime > currentTime && shouldYieldToHost() ) {
            break;
        }
        // 생략...
    }
}
```
- 리액트는 `shoulYieldToHost()` 호출하여 작업 제어권을 브라우저에게 넘겨야하는지 판단
- `shoulYieldToHost()`는 내부적으로 `performance.now()`로 현재 시간을 측정하여 `deadline`으로 지정한 시간과 같거나 많으면 대략 5ms) `true`를 반환
- 루프 중간에 break를 호출하여 JS엔진의 호출 스택을 비워주면 브라우저에서 다른 JS 작업을 실행할 수 있음

1. `MessageChannel API` 생성자가 반환하는 두개의 `port`를 가지고.. `port1.onmessage` 핸들러로 `다음에 이어서 수행할 작업` 함수를 지정
2. `port2.postMessage(null)`을 호출하여 매크로 태스크 큐에 `다음에 이어서 수행할 작업 함수`를 등록
    - 더 자세히 말하자면 `port2.postMessage(null)`가 호출되면 `port1`의 `message`이벤트 처리를 위한 태스크가 JS엔진의 매크로 태스크 큐에 들어간다.
    - 호출 스택이 다 비워지고 나서 매크로 태스크 큐의 작업들이 호출 스택에 올려져 실행될 때가 되면 `message`이벤트 처리를 위한 태스크가 실행되어지고
      <br/> 그 `message`이벤트 처리를 위한 핸들러인 `다음에 이어서 수행할 작업 함수.. 즉, performWorkUntilDeadline`이 실행되는 것이다.

```jsx
// MessageChannel 예제
const { port1, port2 } = new MessageChannel();

port1.onmessage = () => console.log("포트1 메세지 핸들러 실행데스");

port2.postMessage(null);

console.log("호출 스택 1");

queueMicrotask(() => {
    console.log("마이크로 태스크니까 매크로 태스크보다 먼저 실행되어야 함");
});

console.log("호출 스택 2");

/**
* 호출 스택 1
* 호출 스택 2
* 마이크로 태스크니까 매크로 태스크보다 먼저 실행되어야 함
* 포트1 메세지 핸들러 실행데스
*/
```