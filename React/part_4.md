## HOOK 은 어디서 오는걸까?

- react 코어

  - React 코어 패키지 안에는 hook 에 대한 구현이 없다.
  - React -> ReactHooks -> ReactCurrentDispatcher

- react 코어는 react element에 대한 정보만 알고 있다. <br>
- react element 는 fiber로 확장되어야 hook 을 포함하게 된다.
- reconciler 가 react element를 fiber로 확장한다 즉, reconciler 가 hook을 알고 있다 <br> reconciler 가 hook을 어떻게 react 코어로 전달하는가

- react 코어 패키지는 hook 을 사용하기 위해 외부 (reconciler 패키지)에서 주입받음

- react

  - React <- ReactHooks <- ReactCurrentDispatcher <- ReactSharedInternals
    <- react/ReactSharedInternals.js 는 객체에 property 로 외부 모듈을 할당 받는다.
  - shared 는 모든 패키지가 공유하는 공통 패키지
    - shared/ReactSharedInternals.js 는 react 코어 패키지에 연결된 출입구

- reconciler 는 shared 에 hook 을 주입 ( 객체에 property로 할당 )

  - react
    - React <- ReactHooks <- ReactCurrentDispatcher <- ReactSharedInternals
    - shared
      - ReactSharedInternals
    - reconciler

- hook (useState) 의 출처를 정리
  - reconciler 패키지 -> shared/ReactSharedInternal -> react/ReactSharedInternal -> react/ReactCurrentDispatcher -> react/ReactHooks -> react -> 개발자
