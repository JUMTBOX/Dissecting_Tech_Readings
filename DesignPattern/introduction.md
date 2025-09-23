## 4. JS Anti-Pattern

- 전역 컨텍스트에서 수 많은 변수를 정의하여 전역 네임스페이스를 오염시키기
- setTimeout 이나 setInterval 에 함수가 아닌 문자열을 전달하여 내부적으로 eval 실행되게 하기
- Object 클래스의 프로토타입을 수정하기(*특히 나쁜 안티 패턴).
- JS를 inline으로 사용하여 유연성을 떨어뜨리기.
- document.createElement 대신 document.write 사용하기
    ```
    document.write는 오랫동안 잘못 사용되어 왔으며, 여러 단점을 가지고 있습니다.
    만약 페이지가 로드된 뒤에 실행된다면 기존 페이지의 내용을 덮어씌우기 때문에
    XHTML에서도 동작하는 DOM 친화적인 메서드인 document.createElement 메서드를 사용하는 것이 좋습니다.
    ``` 

## 5. Current Syntax of JS
- ```import```문을 이용하면 내보내기된 모듈을 지역 변수로 가져올 수 있다. 
   <br/>(alias를 통해 기존 변수명과의 충돌도 회피 가능)
- ```export```문을 이용하면 지역 모듈을 외부에서 읽을 수 있지만 수정할 수는 없도록 만들어준다.
   <br/>(```import```문처럼 이름을 바꾸어 내보낼 수 있다.) 

```jsx
// staff.mjs
export const baker = {
    bake(item) {
        console.log(`Woo! I just baked ${item}`);
    }
}

// cakeFactory.mjs
import baker from "/modules/staff.mjs";

export const oven = {
    makeCupcake(toppings) {
        baker.bake("cupcake", toppings);
    },
    makeMuffins(mSize) {
        baker.bake("muffin", mSize);    
    }
}

// bakery.mjs
import {oven} from "/modules/cakeFactory.mjs";
oven.makeCupcake("sprinkles");
oven.makeMuffin("large");
```

- ### 정적 모듈 가져오기
  - 정적 가져오기는 메인 코드 실행 전에 모든 모듈을 먼저 다운로드하고 실행해야 하기 때문에 초기 페이지 로드 시 많은 코드를 미리 로드해야하는 상황이라면 성능에 문제가 생길 수 있다.
- ### 동적 모듈 가져오기
  - 지연 로딩(```lazy-loading```) 모듈을 사용하면 필요한 시점에 로드 할 수 있다.
  - (```import(url)```)는 요청된 모듈의 네임스페이스 객체에 대한 Promise 객체를 반환한다.
  - 이 Promise 객체는 모듈 자체와 해당 모듈의 모든 의존성을 가져온 후, 인스턴스화하고 평가한 뒤에 만들어진다.
```jsx
form.addEventListener("submit", e => {
    e.preventDefault();
    
    import("/modules/cakeFactory.js")
        .then(module => {
            module.oven.makeCupcake("sprinkles");
            module.oven.makeMuffin("large");
        });
});
```
- *동적 가져오기는 await와 함께 사용할 수 있습니다.
```jsx
let module = await import("/modules/cakeFactory.js");
```