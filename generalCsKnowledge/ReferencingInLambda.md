## 람다식 내부에서의 외부 스코프 변수 값 참조 관련

### Why?
- Java 8 환경에서 단순 합을 계산하려는 *예제*와 같은 코드에서 에러를 마주치게 되었다.
- ```Variable is accessed from within inner class needs to be final or effectively final```

```java
	private int makeAllLineCombinedHistory(List<PeriodicApprovalStats> allLineHistoryList) {
            int sum = 0;
            
            allLineHistoryList.forEach(lineHistory -> sum += Integer.parseInt(lineHistory.getTotalCount()));
            
            return sum;
	}
```
- 💥 ***왜 이런 에러가 발생하는가***?
  - ```자바8 람다는 “캡처한 로컬 값을 람다 인스턴스의 private final 필드로 복사”합니다. 이 때문에 캡처 대상 로컬 변수는 final/효과상 final이어야 합니다. 그렇지 않으면 로컬 값과 람다 내부 사본이 어긋납니다. 스펙과 구현이 이를 금지합니다. (Oracle Docs)```
  - Java 8의 람다식은 컴파일 시점에 ```invokedynamic``` 호출 지점으로 변환되고, runtime에 부트스트랩 메서드 ```LambdaMetaFactory.metafactory```가
  "람다 인스턴스 생성용 팩토리"를 반환
  - 이후 해당 팩토리를 호출하여 ```캡처 값```이 주입된 함수형 인터페이스의 인스턴스를 만들고, 그 인스턴스의 ```SAM 메서드```가 실제 람다 본문을 실행 
      - *여기서 ```캡처 값```이란 람다식 내부에서 참조하는 ```외부 변수 값```*
      - *정확하게는 람다 생성 시점에 외부 스코프의 ```로컬 변수*매개변수(및 this)의 현재 값```을 ```캡처```하여 람다 인스턴스의 ```final```필드에 보관한다.*


#### 💥 ```invokedynamic```, ```LambdaMetaFactory.metafactory```,```SAM 메서드```가 뭐길래?

#### 1. ```invokedynamic```란?
- JVM 바이트코드의 ```동적 호출``` 명령이다, 실행 시점에 부트스트랩 메서드를 통해 호출 지점을 링크하고, 그 결과로 얻은 ```CallSite```의 ```MethodHandle``` 타깃을 호출한다.
  -  ```CallSite```? : ```invokedynamic``` 호출 지점과 연결되는 "가변 타깃 홀더", 해당 바이트코드의 실행은 항상 이 *CallSite*의 **현재 타깃(MethodHandle)** 에 위임된다.<br> 
    상수형/가변형/휘발성 변형이 있다.
  - ```MethodHandle```? : JVM이 직접 실행할 수 있는 **"타입이 있는 저수준 함수 참조"** 이다. ```invokeExact/invoke```로 호출하며, 대상은 메서드,생성자,필드 접근 등이 있다.
#### 2. ```LambdaMetaFactory.metafactory```란?
- ```invokedynamic```의 부트스트랩 메서드로, 람다*메서드 참조를 **함수형 인터페이스 인스턴스** 로 만들어내는 ```CallSite```를 반환한다. 이후에 이 ```CallSite```의 타깃을 호출하면 캡처 값이 주입된 함수 객체가 생성된다.  
#### 3. ```SAM 메서드```란?
- 단일 추상 메서드(Single Abstract Method)를 뜻하며, 함수형 인터페이스가 가지는 유일한 추상 메서드를 말한다.
- 람다식은 이 메서드의 구현으로 간주된다. (??)


### 💥 [실행 순서]
1. 소스의 람다식 자리에 ```invokedynamic``` 바이트코드가 배치됨, 이 호출의 부트스트랩 메서드로 ```LambdaMetaFactory.metafactory```가 기록됨
2. 최초 실행 시 부트스트랩
    - ```metafactory```가 호출되어 ```CallSite```를 만들고, 그 타깃은 람다 객체를 생성하는 ```MethodHandle```이다. 이때 SAM 시그니처, 구현 메서드, 타입 적응 정보가 전달된다.
3. 캡처와 인스턴스 생성
    - 람다식 평가 시, 캡처해야 할 로컬 값들이 팩토리의 인수로 넘어가 람다 인스턴스 내부의 final 필드로 고정된다.<br/>
      캡처가 없다면 동일 인스턴스를 재사용할 수 있다.
4. 본문 실행
    - 생성된 인스턴스의 SAM 메서드를 호출하면, 컴파일러가 생성해둔 구현 메서드(보통 주변 클래스의 private static/private 메서드)에 캡처값과 인자가 결합되어 실행된다.
5. JIT 최적화
    - ```생성된 람다의 SAM 메서드는 최종 메서드로 간주되어 HotSpot이 인라이닝하기 용이합니다. invokedynamic 경유 구조는 메서드 핸들 최적화 이점을 활용합니다.``` (<a href="https://openjdk.org/jeps/126">openjdk.org<a/>)


### 💥 그렇다면 JS의 람다식에서는 왜 허용되는가?