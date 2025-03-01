## MDN -> window.scheduler 프로퍼티

### 1. API 개요 및 필요성

- 기존의 setTimeout()이나 requestIdleCallback()을 이용한 작업 스케줄링은 브라우저 렌더링 성능을 고려하지 않고 단순히 작업을 지연시키거나, 비효율적인 방식으로 실행하는 경우가 많았습니다.
- Prioritized Task Scheduling API(window.scheduler)는 작업의 우선순위를 명시적으로 <br> 지정하여 실행 순서를 제어하고, 메인 스레드의 부하를 최소화하면서도 사용자 경험(UX)을 유지하는 데 초점을 맞춘 API입니다.

### 2. 핵심 개념 및 Scheduler 객체

- window.scheduler는 Scheduler 객체를 반환하며, 이 객체의 핵심 기능은 다음과 같습니다.

- 메서드 설명
  scheduler.postTask(task, options) 우선순위 기반으로 작업을 예약
  scheduler.yield() 현재 실행 중인 작업을 중단하고 다른 대기 작업을 실행하도록 양보
  scheduler.getPriority() 현재 실행 중인 코드의 우선순위를 확인

### 3. postTask() 메서드와 우선순위 전략

- (1) postTask() 사용 예시

```javascript
if ("scheduler" in window) {
  window.scheduler.postTask(
    () => {
      console.log("High priority task executed");
    },
    { priority: "user-blocking" }
  );

  window.scheduler.postTask(
    () => {
      console.log("Low priority task executed");
    },
    { priority: "background" }
  );
}
```

- 이 경우, **"High priority task executed"**가 **"Low priority task executed"**보다 먼저 실행됩니다.

- (2) 우선순위 개념 (Prioritization Levels)

  - 'user-blocking': 가장 높은 우선순위. 입력 이벤트와 관련된 작업에 적합 (예: 클릭 이벤트 처리).
  - 'user-visible': 사용자 인터페이스를 업데이트하는 데 필요한 작업. 중간 정도의 우선순위 (예: UI 업데이트).
  - 'background': 낮은 우선순위의 작업. 메인 스레드의 부하가 없을 때만 실행됨 (예: 분석 데이터 로깅, 캐시 프리패칭).

### 4. 기존 API와의 차이점 및 비교

- 기존 방식 window.scheduler
  setTimeout(callback, delay) 정확한 우선순위 지정 불가, 일정 지연 후 실행
  requestIdleCallback(callback) 메인 스레드가 유휴 상태일 때 실행, 우선순위 지정 불가
  window.scheduler.postTask(task, { priority }) 명확한 우선순위 지정 가능, 작업 실행 최적화 5. 실전 적용 예시: React UI 최적화
  React 앱에서 window.scheduler를 활용하여 UI 성능을 향상할 수 있습니다.

- ### 비동기 데이터 요청 시 user-visible 활용

```jsx
import { useEffect } from "react";

function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("데이터 로딩 완료"), 2000);
  });
}

export default function App() {
  useEffect(() => {
    if ("scheduler" in window) {
      window.scheduler.postTask(
        async () => {
          const data = await fetchData();
          console.log(data);
        },
        { priority: "user-visible" }
      );
    }
  }, []);

  return <div>데이터 로딩 중...</div>;
}
```

- 여기서는 네트워크 요청과 같은 중요하지만 UI 블로킹이 필요 없는 작업을 user-visible 우선순위로 설정하여 실행합니다.

- ### scheduler.yield()를 활용한 긴 작업 분할 <br>
  yield()를 활용하면 브라우저의 메인 스레드를 블로킹하지 않으면서 작업을 실행할 수 있습니다.

```jsx
async function heavyComputation() {
  for (let i = 0; i < 1000000; i++) {
    if (i % 10000 === 0) {
      await window.scheduler.yield(); // 중간에 실행을 양보
    }
    // 연산 수행
  }
  console.log("연산 완료");
}
```

- 이렇게 하면 긴 연산이 브라우저를 멈추지 않도록 조정할 수 있습니다.
