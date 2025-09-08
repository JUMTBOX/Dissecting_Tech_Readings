## Difference between requestAnimationFrame & scheduler.postTask with Vsync

### Vsync (Vertical synchronization) 란?
- 디스플레이가 한 프레임을 화면에 그려내는 주기 ( 일반적으로 60Hz -> 16.6ms,  120Hz -> 8.3ms ) 와<br> 브라우저 렌더링 파이프라인을 맞추는 메커니즘


### 브라우저의 사이클
    1. 입력 처리 -> JS 실행 -> 스타일/레이아웃 계산
    2. 페인트 명령 생성 
    3. GPU/컴포지터가 실제 프레임을 화면에 전송

### "Vsync 에 정렬된다" 라는 의미
- #### requestAnimationFrame 같은 API가 디스플레이에 새 프레임이 그려지기 직전 시점에 콜백을 실행하도록 예약된다는 의미
  - 즉, 화면 리프레시 타이밍에 맞추어 DOM 변경을 적용하면 곧바로 그 다음 화면 출력에 반영된다.

- <h4> 반면 scheduler.postTask 는 이벤트 루프 내 태스크 큐에서 우선순위에 따라 실행될 뿐이고, 디스플레이 주기와 직접 동기화 되지 않는다. <br>
  그래서 같은 DOM 변경이라도 "프레임 경계"가 보장되지 않고, 변경이 같은 프레임 안에서 합쳐지거나 지연될 수 있다.</h4>