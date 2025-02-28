## Long Task 최적화

### 모든 작업이 메인 스레드에서 한 번에 실행된다면?

- 배열은 모든 웹 개발자가 사용하는 기본 도구이며, 배열을 반복(iterate)하는 방법은 여러 가지가 있다.
- 하지만 잘못된 방법을 선택하면 모든 처리가 동기적으로 실행되며 긴 시간 동안 블로킹(blocking)되는 작업이 발생할 수 있다. 문제는, 가장 직관적인 방법들이 대부분 잘못된 방법이라는 점이다. 예를 들어, for..of 루프는 기본적으로 동기적으로 실행되며, forEach 및 map과 같은 배열 메서드는 오직 동기적으로만 실행될 수 있다. 어쩌면 지금 당신의 코드에도 최적화가 필요한 루프가 있을 것이다.

### 긴 작업(Long Task)의 문제점은 무엇인가?

- 긴 작업은 사용자 경험을 저해하는 주요 원인 중 하나이다.
- 사용자가 페이지를 상호작용하는 시점에 긴 작업이 실행 중이라면, 브라우저는 해당 작업이 끝날 때까지 사용자 입력을 처리할 수 없다. 이는 입력 지연(Input Delay) 을 초래하고, Interaction to Next Paint(INP) 성능 을 저하시킨다.
- 이러한 긴 작업은 도로의 포트홀(pothole, 움푹 팬 곳)과 같아서 운전자가 이를 피해야 하거나 차량이 손상될 위험이 있는 것과 비슷한 문제를 유발한다. 긴 작업이 단순히 사용자 입력과 우연히 겹치는 경우라면 그나마 나을 수 있지만, 사용자의 입력에 대한 응답으로 실행된다면 문제는 더욱 심각해진다. 클릭할 때마다 긴 지연이 발생하는 것은 UX 측면에서 매우 좋지 않다.

- 배열의 크기가 커질수록 동기적으로 처리되는 루프는 긴 작업을 유발할 가능성이 커진다. 예를 들어, 한 번의 작업을 처리하는 데 0.25ms가 걸리고, 배열에 1,000개의 요소가 있다면 총 250ms 동안 블로킹되는 긴 작업이 발생한다. 이 문제를 해결하려면 긴 작업을 잘게 나누어 처리하는 방법을 고민해야 한다.

### 인터랙션 반응성 최적화하기

- 작업을 중단하여 이벤트 루프(event loop)가 계속 실행될 수 있도록 하는 기법을 "양보(yielding)" 라고 한다. 이를 수행하는 대표적인 방법은 setTimeout 을 0ms로 설정하는 것이며, 최근에는 scheduler.yield() 라는 최신 대안이 등장했다. 하지만 scheduler.yield()는 모든 브라우저에서 지원되지 않으므로, 실제 프로덕션 환경에서는 setTimeout을 폴리필(polyfill)로 사용해야 한다.

- 비동기적으로 반복문을 실행하려면 async/await을 사용해야 한다. 그러나 문제가 있다.

```javascript
function handleClick() {
  items.forEach(async (item) => {
    await scheduler.yield();
    process(item);
  });
}
```

- 이 코드가 정상적으로 작동할 것처럼 보이지만, 사실 그렇지 않다. forEach는 콜백 함수가 비동기 함수인지 여부를 신경 쓰지 않고, 배열의 모든 요소를 동기적으로 실행해버린다. 즉, await scheduler.yield()가 있어도 전체 루프는 여전히 차단(blocking)된다.

### 해결 방법: for..of 루프 사용

- forEach 대신 for..of 루프를 사용하면 문제를 해결할 수 있다.

```javascript
async function handleClick() {
  for (const item of items) {
    await scheduler.yield();
    process(item);
  }
}
```

- 이제 긴 작업이 잘게 나누어져 사용자의 클릭 이벤트에 즉시 반응할 수 있다.

- 하지만 여기에 또 다른 문제가 있다.

### 성능 최적화: 배치(Batching) 처리

- 위 코드에서 우리는 모든 반복에서 yield를 호출하고 있다. 이로 인해 scheduler.yield() 를 지원하지 않는 브라우저에서는 setTimeout(0) 을 사용하게 되는데, 이 경우 작업이 지나치게 느려진다. 예를 들어, 1000개의 요소를 처리하는데 scheduler.yield()를 사용할 경우 1초 정도 걸리지만, setTimeout(0)을 사용하면 2분 이상 소요될 수 있다. 이는 setTimeout이 중첩 호출 시 최소 4ms의 지연 을 추가하기 때문이다.

- 이 문제를 해결하려면 일정량의 작업을 처리한 후에만 yield를 실행하는 "배칭(Batching)" 기법 을 적용해야 한다.

### 시간 기반 배칭 처리

- 배칭을 적용할 때 단순히 일정 개수의 요소를 처리한 후 yield를 호출하는 방식은 최적의 방법이 아니다. 대신, 작업을 실행하는 데 걸리는 시간을 기준으로 배치를 나누는 것이 더 효과적이다.

```javascript
const BATCH_DURATION = 50; // 50ms 동안만 작업 실행
let timeOfLastYield = performance.now();

function shouldYield() {
  const now = performance.now();
  if (now - timeOfLastYield > BATCH_DURATION) {
    timeOfLastYield = now;
    return true;
  }
  return false;
}

async function handleClick() {
  for (const item of items) {
    if (shouldYield()) {
      await scheduler.yield();
    }
    process(item);
  }
}
```

- 이 방식에서는 50ms 동안 작업을 수행한 후 yield를 실행하여 브라우저가 다른 작업(예: 사용자 입력 처리, UI 업데이트)을 수행할 수 있도록 한다.

### 배치 크기의 선택

- 배치 크기는 처리 속도와 반응성 사이의 절충(trade-off) 관계를 가진다.

- 배치 크기가 크면: 처리 속도가 빠르지만, 최악의 경우 사용자가 100ms 이상 기다릴 수 있음.
- 배치 크기가 작으면: 반응성이 좋아지지만, 총 처리 시간이 증가할 수 있음.
  일반적으로 50ms 배치 크기는 적절한 절충안이 될 수 있으며, 애플리케이션의 성격에 따라 조정하는 것이 좋다.

### 프레임 속도(Frame Rate) 최적화

- UI가 부드럽게 업데이트되려면 프레임 속도(Frame Rate)를 유지하는 것이 중요하다. scheduler.yield() 를 사용할 경우, 작업이 너무 우선순위가 높아져서 프레임 렌더링이 지연될 수 있다. 이를 해결하려면 requestAnimationFrame을 활용하여 프레임이 갱신될 때 yield가 실행되도록 조정할 수 있다.

```javascript
const BATCH_DURATION = 1000 / 30; // 30 FPS 유지
let timeOfLastYield = performance.now();

function shouldYield() {
  const now = performance.now();
  if (now - timeOfLastYield > BATCH_DURATION) {
    timeOfLastYield = now;
    return true;
  }
  return false;
}

async function handleClick() {
  for (const item of items) {
    if (shouldYield()) {
      await new Promise(requestAnimationFrame);
      await scheduler.yield();
    }
    process(item);
  }
}
```

- 이렇게 하면 작업을 실행하는 동안 최소 30FPS를 유지하여 UI가 끊기지 않고 부드럽게 실행될 수 있다.

### 결론

- 긴 작업(Long Task)을 방지하고 반응성을 최적화하기 위해 비동기 반복문(for..of + await)을 사용한다.
  <br> 일정 시간(50ms) 단위로 작업을 배칭(Batching)하여 처리한다.
  <br> 프레임 속도를 유지하기 위해 requestAnimationFrame을 활용한다.
  <br> 이러한 기법을 적용하면 대량의 데이터를 처리하면서도 UI 반응성을 유지할 수 있다.
  <br> 최적의 yield 전략을 선택하는 것이 애플리케이션의 성능을 결정짓는 중요한 요소가 될 것이다.
