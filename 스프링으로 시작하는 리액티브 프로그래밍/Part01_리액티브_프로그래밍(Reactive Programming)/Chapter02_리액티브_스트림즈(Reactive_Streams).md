# 스프링으로 시작하는 리액티브 프로그래밍
## Chapter 02. 리액티브 스트림즈(Reactive Streams)
> Chatper 2에서는 리액티브 스트림즈가 무엇인지 살펴보고, 리액티브 스트림즈의 핵심 컴포넌트인 Publisher 와 Subscriber 의 동작 과정을 자세히 알아보자.
### 2.1. 리액티브 스트림즈(Reactive Streams)란?
- 리액티브 라이브러리를 어떻게 구현할지 정의해 놓은 별도의 표준 사양을 **리액티브 스트림즈**라고 부른다.
- 리액티브 스트림즈는 한마디로 **'데이터 스트림을 Non-Blocking 이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양'** 이라고 표현한다.

### 2.2. 리액티브 스트림즈 구성요소
| 컴포넌트         | 설명                                                                                                                         |
|--------------|----------------------------------------------------------------------------------------------------------------------------|
| Publisher    | 데이터를 생성하고 통지(발행, 게시, 방출)하는 역할을 한다.                                                                                         |
| Subscriber   | 구독한 Publisher 로부터 통지(발행, 게시, 방출)된 데이터를 전달받아서 처리하는 역할을 한다.                                                                  |
| Subscription | Publisher 에 요청할 데이터의 개수를 지정하고, 데이터의 구독을 취소하는 역할을 한다.                                                                       |
| Processor    | Publisher 와 Subscriber 의 기능을 모두 가지고 있다. 즉, Subscriber 로서 다른 Publisher 를 구독 할 수도 있고, Publisher 로서 다른 Subscriber 가 구독할 수 있다. |
![Publisher와 Subscriber의 동작 과정](../img/Publisher_And_Subscriber_Flow.png)
- 리액티브 스트림즈의 컴포넌트인 Publisher 와 Subscriber 간에 데이터가 전달되는 동작 과정을 그림으로 표현한 것이다.
  1. 먼저 Subscriber 는 전달받을 데이터를 구독한다.(**Subscribe**)
  2. 다음으로 Publisher 는 데이터를 통지(발행, 게시, 방출)할 준비가 되었음을 Subscriber 에 알린다.(**onSubscribe**)
  3. Publisher 가 데이터를 통지할 준비가 되었다는 알림을 받은 Subscriber 는 전달받기를 원하는 데이터의 개수를 Publisher 에게 요청한다.(**Subscription.request**)
  4. 다음으로 Publisher 는 Subscriber 로부터 요청받은 만큼의 데이터를 통지한다.(**onNext**)
  5. 이렇게 Publisher 와 Subscriber 간에 데이터 통지, 데이터 수신, 데이터 요청의 과정을 반복하다가 Publisher 가 모든 데이터를 통지하게 되면 마지막으로 데이터 전송이 완료되었음을 Subscriber 에게 알린다.(**onComplete**) 만약에 Publisher 가 데이터를 처리하는 과정에서 에러가 발생하면 에러가 발생했음을 Subscriber 에게 알린다.(**onError**)
- Q : Subscriber 가 Subscription.request 를 통해 왜 굳이 데이터의 요청 개수를 지정할까?
- A : 그림상으로 Publisher 와 Subscriber 가 마치 같은 스레드에서 동기적으로 상호작용하는 것처러 보이지만 실제로 Publisher 와 Subscriber 는 각각 다른 스레드에서 비동기적으로 상호작용하는 경우가 대부분이다. 
  - 이럴 경우 **Publisher 가 통지하는 속도가 Publisher 로부터 통지된 데이터를 Subscriber 가 처리하는 속도보다 더 빠르면 처리를 기다리는 데이터는 쌓이게 되고, 이는 결과적으로 시스템 부하가 커지는 결과를 낳는다.**
  - 이러한 문제를 방지하기 위해 Subscription.request 를 통해 데이터 개수를 제어하는 것이다.

### 2.3. 코드로 보는 리액티브 스트림즈 컴포넌트
- 리액티브 스트림즈의 컴포넌트는 실제 코드에서 인터페이스 형태로 정의되며, 이 인터페이스들을 구현해서 해당 컴포넌트를 사용하게 된다.
#### 2.3.1. Publisher
```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```
- Publisher 인터페이스 코드인데, 코드 형태는 subscribe 메서드 하나만 구현하면 되는 수준으로 매우 단순하다.
  - subscribe 메서드는 파라미터로 전달받은 Subscriber 를 등록하는 역할을 한다.
- 그런데 Publisher 인터페이스 코드를 보면서 다음과 같은 의문이 들 수 있다.
- Q: 'Publisher 는 데이터를 생성하고 통지하는 역할을 하고, Subscriber 는 Publisher 가 통지하는 데이터를 전달받기 위해 구독을 한다고 이해하고 있는데, 왜 Subscriber 가 아닌 Publisher 에 Subscribe 메서드가 정의되어 있을까?'
- A : Kafka 같은 메시지 기반의 시스템에서 Pub/Sub 모델을 이해하고 있는 분이라며, 아마도 이런 의문이 드는것이 당연하다.
  - 리엑티브 스트림즈에서의 Publisher/Subscriber 와 Kafka 에서의 Publisher/Subscriber 는 의미가 조금 다르다.
  - Kafka 의 경우 Publisher 와 Subscriber 중간에 메시지 브로커(Message Broker)가 있고 이 브로커 내에 여러 개의 토픽(Topic)이 존재하는데, Publisher 와 Subscriber 는 브로커에 있는 특정 토픽을 바라본느 구조로 이루어져 있다.
    - 그래서 Kafka 에서의 Publisher 와 Subscriber 는 각각 브로커 내의 특정 토픽만 바라보면 되기 때문에 Publisher 는 특정 토픽으로 메시지 데이터를 전송하기만 하면 되고, Subscriber 는 특정 토픽을 구독하고 해당 토픽에 전달되는 메시지 데이터를 전달받기만 하면 된다.
    - 이는 Publisher 와 Subscriber 의 느슨한 결합 구조라고 볼 수 있다.
  - 반면의 리액티프 스트림즈에서의 Publisher 와 Subscriber 는 개념상으로는 Subscriber 가 구독하는것이 맞는데 실제 코드상에서 Publisher 가 subscribe 메서드를의 파라미터인 Subscriber 를 등록하는 형태로 구독이 이루어진다.
#### 2.3.2. Subscriber
```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```
- Subscriber 인터페이스 코드이다. Subscriber 인터페이스는 총 네 개의 메서드를 구현해야 한다.
  - **onSubscribe** 메서드는 구독 시작 시점에 어떤 처리를 하는 역할을 한다.
    - 여기서의 처리는 Publisher 에게 요청할 데이터의 개수를 지정하거나 구독을 해지하는 것을 의미하는데, 이것은 onSubscribe 메서드의 파라미터로 전달되는 Subscription 객체를 통해서 이루어진다.
  - **onNext** 메서드는 Publisher 가 통지한 데이터를 처리하는 역할을 한다.
  - **onError** 메서드는 Publisher 가 데이터 통지를 위한 처리 과정에서 에러가 발생했을 때 해당 에러를 처리하는 역할을 한다.
  - **onComplete** 메서드는 Publisher 가 데이터 통지를 완료했음을 알릴 때 호출되는 메서드이다.
    - 데이터 통지가 정상적으로 완료될 경우에 어떤 후처리를 해야 한다면 onComplete 메서드에서 처리 코드를 작성하면 된다.
#### 2.3.3. Subscription
```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```
- Subscription 인터페이스는 Subscriber 가 구독한 데이터의 개수를 요청하거나 또는 데이터 요청의 취소, 즉 구독을 해지하는 역할을 한다.
- Subscription 인터페이스는 두 개의 메서드를 구현해야 한다.
  - Request 메서드를 통해서 Publisher 에게 데이터의 개수를 요청할 수 있고, cancel 메서드를 통해서 구독을 해지할 수 있다.
> 정리
> - Publisher 와 Subscriber 의 동작 과정을 리액티브 스트림즈의 컴포넌트 코드 관점에서 설명
>   - Publisher 가 Subscriber 인터페이스 구현 객체를 subscribe 메서드의 파리미터로 전달한다.
>   - Publisher 내부에서는 전달받은 Subscriber 인터페이스 구현 객체의 onSubscribe 메서드를 호출하면서 Subscriber 의 구독을 의미하는 Subscription 인터페이스 구현 객체를 Subscriber 에게 전달한다.
>   - 호출된 Subscriber 인터페이스 구현 객체의 onSubscribe 메서드에서 전달 받은 Subscription 객체를 통해 전달받을 데이터의 개수를 Publisher 에게 요청한다.
>   - Publisher 는 Subscriber 로부터 전달받은 요청 개수만큼의 데이터를 onNext 메서드를 호출해서 Subscriber 에게 전달한다.
>   - Publisher 는 통지할 데이터가 더 이상 없을 경우 onComplete 메서드를 호출해서 Subscriber 에게 데이터 처리 종료를 알린다.

#### 2.3.4. Processor
```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```
- 다른 인터페이스들과 다른 점은 Subscriber 인터페이스와 Publisher 인터페이스를 상속한다는 것이다.
  - Processor 가 Publisher 와 Subscriber 의 기능을 모두 가지고 있기 때문이다.

### 2.4. 리액티브 스트림즈 관련 용어 정의
- **Signal**
  - 우리말로 해석하면 '신호'라는 의미이다.
  - **Publisher 와 Subscriber 간에 주고받는 상호작용**을 신호, 즉 Signal 이라고 표현한다.
  - Signal 을 리액티브 스트림즈에서 표현 하는 방법
    - onSubscribe, onNext, onComplete, onError 메서드 : Subscriber 인터페이스에 정의되지만 이 메서드들을 실제 호출해서 사용하는 주체를 Publisher 이기 때문에 Publisher 가 Subscriber 에게 보내는 Signal 이라고 볼 수 있다. 
    - request, cancel 메서드 : Subscriber 인터페이스 코드에 정의되지만 이 메서드들을 실제로 사용하는 주체는 Subscriber 이므로 Subscriber 가 Publisher 에게 보내는 Signal 이라고 볼 수 있다.
- **Demand**
  - 사전에서 수요, 요구 등을 의미하는데 리액티브 스트림즈에서도 의미가 이와 비슷하다.
  - 바로 Subscriber 가 Publisher 에게 요청하는 데이터를 의미한다.
    - 구체적으로 Publisher 가 아직 Subscriber 에게 전달하지 않은 Subscriber 가 요청한 데이터를 뜻한다.
- **Emit**
  - Publisher 와 Subscriber 의 동작 과정을 설명하면서 Publisher 가 Subscriber 에게 데이터를 전달하는 것을 일컬을 때 데이터를 '통지(발행, 게시, 방출)한다'라는 용어를 사용했다.
    - 그리고 '통지', '발행', '게시', '방출'이란 용어의 공통점은 입력으로 들어오는 데이터를 제공하는 의미라고도 말했다.
    - **데이터를 내보낸다** 정도로 이해하면 된다.
  - Publisher 가 emit 하는 Signal 중에서 데이터를 전달하기 위한 onNext Signal 을 줄여서 '데이터를 emit 한다'라고 표현하겠다.
- **Upstream/Downstream**
  - Up은 우리말로 '위에, 위로', Down 은 '아래에, 아래로'라는 의미이다. 그리고 stream 은 '흐르다' 라는 뜻이 있다. 
  - 리액티브 세상에서 흘러가는 것은 데이터이다.
  - ```java
    public class Example2_5 {
      public static void main(String[] args){
        Flux
            .just(1, 2, 3, 4, 5, 6)
            .filter(n -> n % 2 == 0)
            .map(n -> n * 2)
            .subscribe(System.out::println);
      }
    }
    ```
    - 4번 라인의 just 메서드를 사용해서 데이터를 생성한 후 emit 하게 되는데, 여기서 just 메서드는 리액티브 스트림즈의 컴포넌트 중에서 Publisher 의 역할을 한다.
      - 그런데 잘 보면 just 메서드를 호출한 다음에 연이어서 바로 filter 메서드를 호출하고, 그 다음 연이어서 map 메서드를 호출하는 것을 볼 수 있다.
    - 이런 방식으로 메서드가 하나로 연결된 것처럼 보이는 것을 **메서드 체인**이라고 한다.
      - 이렇게 메서드 체인 방식으로 호출할 수 있는 이유는 호출하는 각각의 메서드들이 반환하는(subscribe 메서드 제외) 반환 값이 모두 Flux 타입의 객체이기 때문에 이런 식의 메서드 호출이 가능해지는 것이다.
    - 데이터 스트림의 관점에서 볼 때, 4번 라인의 just 메서드 호출을 통해 반환된 Flux 입장에서는 5번 라인의 filter 메서드 호출을 통해 반환된 Flux 가 자신보다 더 하위에 있기 때문에 Downstream이 된다.
    - 반면에 filter 메서드 호출을 통해 반환된 Flux 입장에서는 just 메서드 호출을 통해 반환된 Flux 가 자신보다 더 상위에 있기 떄문에 Upstream 이 된다.
    > - 현재 호출된 메서드에서 반환된 Flux 의 위치에서 자신보다 더 상위에 있는 Flux 는 Upstream, 하위에 있는 Flux 는 Downstream 이 된다는 사실만 기억하자.
    > - 단순히 Flux 자체를 Upstream/Downstream 이라고 말하기는 어렵지만, Flux 를 통해서 데이터의 흐름이 만들어지기 때문에 이렇게 기억하는 것이 이해하기 쉽다.
- **Sequence**
  - Publisher 가 emit 하는 데이터의 연속적인 흐름을 정의해 놓은 것 자체를 의미한다.
    - 이 Sequence 는 Operator 체인 형태로 정의된다.
    - 위 코드에와 같이 Flux 를 통해서 데이터를 생성, emit 하고 filter 메서드를 통해서 필터링한 후, map 메서드를 통해 변환하는 이러한 과정 자체를 Sequence 라고 한다. 
- **Operator**
  - 연산자 정도로 해석할 수 있다.
  - 위 코드에서 사용한 just, filter, map 같은 메서드들을 리액티브 프로그래밍에서는 연산자라고 부른다.
  - **리액티브 프로그래밍은 Operator 로 시작해서 Operator 로 끝난다고 해도 과언이 아닐 만큼 Operator 가 리액티브 프로그래밍의 핵심**이다.
- **Source**
  - 대부분 '최초의'라는 의미로 사용되고 종종 Original 이라는 용어도 사용된다.
    - 대표적으로 Data Source, Source Publisher, Source Flux 등을 볼 수 있다.

### 2.5. 리액티브 스트림즈의 구현 규칙
- 리액티브 스트림즈 표준 사양에는 리액티브 스트림즈 컴포넌트를 어떻게 구현해야 되는지에 대한 규칙들이 컴포넌트 별로 정의되어 있다.

#### 2.5.1. Publisher 구현을 위한 주요 기본 규칙
| 번호 | 규칙                                                                                                          |
|---|-------------------------------------------------------------------------------------------------------------|
| 1 | Publisher 가 Subscriber 에게 보내는 onNext signal 의 총 개수는 항상 해당 Subscriber 의 구독을 통해 요청된 데이터의 총 개수보다 더 작거나 같아야 한다. |
| 2 | Publisher 는 요청된 것보다 적은 수의 onNext signal 을 보내고 onComplete 또는 onError 를 호출하여 구독을 종료할 수 있다.                    |
| 3 | Publisher 의 데이터 처리가 실패하면 onError signal 을 보내야 한다.                                                           |
| 4 | Publisher 의 데이터 처리가 성공적으로 종료되면 onComplete signal 을 보내야 한다.                                                  |
| 5 | Publisher 가 Subscriber 에게 onError 또는 onComplete signal 을 보내는 경우 해당 Subscriber 의 구독은 취소된 것으로 간주되어야 한다.       |
| 6 | 일단 종료 상태 signal 을 받으면(onError, onComplete) 더 이상 signal이 발생되지 않아야 한다.                                        |
| 7 | 구독이 취소되면 Subscriber 는 결국 signal 을 받는 것을 중지해야 한다.                                                            |
- 1번 규칙은 Subscriber 가 요청한 것보다 더 많은 개수의 데이터를 publisher 가 보낼 수 없음을 의미한다. 
  - 즉, Subscriber 의 요청 개수보다 더 적거나 같은 개수의 onNext signal 을 통해 데이터를 보낼 수 있다.
- 2번 규칙은 1번 규칙과 유사하지만 Subscriber 의 요청을 마지막으로 처리하는 경우의 규칙이다.
  - 예를 들어 Subscriber 의 요청 개수가 5이고 Publisher 가 emit 할 데이터 개수가 3개밖에 남지 않았다면 남은 3개를 Subscriber 에게 emit 한 후, 정상적으로 처리가 끝났음을 알리는 onComplete signal 을 보내거나 또는 처리 중 에러가 발생할 경우에는 onError signal 을 보낸다.
  - 하지만 이 규칙의 예외가 있다.
    - Publisher 가 처리할 데이터가 끊임없이 발생하는 무한 스트림의 경우, 처리 중 에러가 발생하기 전까지는 종료 자체가 없기 때문에 2번 규칙에 대한 예외적인 경우가 되겠다.
      - 예를 들어 IoT 디바이스 센서에서 발생하는 데이터처럼 끊임없이 발생하는 데이터를 떠올려 보면 쉽게 이해할 수 있을 것이다.
- 3번 규칙은 Publisher 가 처리를 진행할 수 없는 상황이 되면 Subscriber 에게 알려서 Publisher 에서 발생한 실패를 Subscriber 가 처리할 수 있는 기회를 가지도록 하는 데 주 목적이 있다.
- 4번 규칙은 Publisher 가 종료 상태가 되었음을 Subscriber 에게 알림으로써 Subscriber 가 리소스를 정리하는 등의 후처리를 할 수 있도록 하는 데 그 목적이 있다. 
- 5번 규칙의 목적은 Publisher 가 전송한 onError 또는 onComplete Signal 은 구독 취소와 동일한 기능을 한다는 것을 의미한다.
- 6번 규칙의 목적은 onError 또는 onComplete signal 을 통해 Publisher 와 Subscriber 간의 상호작용이 끝났음을 알리는 것이다.
  - 즉, 이미 상호작용이 끝났음을 알렸는데 signal 이 발생하는 것은 논리적으로 맞지 않다는 의미와 같다.
- 7번 규칙의 의미는 구독 취소를 위해 Subscription.cancel() 이 호출되었을 때 Publisher 가 구독 취소 요청을 준수해야 한다는 의미와 같다.

#### 2.5.2. Subscriber 구현을 위한 주요 기본 규칙
| 번호 | 규칙                                                                                                                  |
|---|---------------------------------------------------------------------------------------------------------------------|
| 1 | Subscriber 는 Publisher 로부터 onNext signal 을 수신하기 위해 Subscription.request(n)를 통해 Demand signal 을 Publisher 에게 보내야 한다. |
| 2 | Subscriber.onComplete() 및 Subscriber.onError(Throwable t)는 Subscription 또는 Publisher 의 메서드를 호출해서는 안된다.              |
| 3 | Subscriber.omComplete() 및 Subscriber.onError(Throwable t)는 signal 을 수신한 후 구독이 취소된 것으로 간주해야 한다.                      |
| 4 | 구독이 더 이상 필요하지 않는 경우 Subscriber 는 Subscription.cancel()을 호출해야 한다.                                                    |
| 5 | Subscriber.onSubscribe()는 지정된 Subscriber 에 대해 최대 한 번만 호출되어야 한다.                                                     |
- 1번 규칙의 목적은 데이터를 언제, 얼마나 수신할 수 있는지를 결정하는 책임이 Subscriber 에게 있다는 것을 확립하는 것이다.
  - 리액티브 스트림즈에서는 한 번에 하나의 데이터를 요청하기보다는 Subscriber 가 처리할 수 있는 적절한 상한성만큼의 데이터 개수 요청을 권장한다.
- 2번 규칙은 Subscriber 가 완료 signal 또는 에러 signal 을 처리하는 동안 Publisher/Subscription 과 Subscriber 간의 순환 및 경쟁 조건(Race Condition)을 방지하기 위함입니다.
  - JPA 엔티티 간의 그래프 탐색에서 상호 재귀가 발생하는 상황을 생각해보면 이해하기 쉬울 것이다.
- 3번 규칙의 목적은 Subscriber 가 Publisher 의 종료 상태 signal 을 인정하고 받아들이도록 하는 것이다.
  - 즉, onComplete 또는 onError signal 이 수신되면 구독이 더 이상 유효하지 않게 된다는 것을 의미한다.
- 4번 규칙의 목적은 Subscriber 가 구독이 더 이상 필요하지 않을 때 명시적으로 구독을 취소함으로써 해당 구독이 유지하고 있는 리소스를 적절한 시기에 안전하게 해제할 수 있도록 하는 것이다.
- 5번 규칙의 의미는 동일한 구독자가 최대 한 번만 구독할 수 있다는 의미와 같다.

#### 2.5.3. Subscription 구현을 위한 주요 기본 규칙
| 번호 | 규칙                                                                                                                              |
|---|---------------------------------------------------------------------------------------------------------------------------------|
| 1 | 구독은 Subscriber 가 onNext 또는 onSubscribe 내에서 동기적으로 Subscription.request 를 호출하도록 허용해야 한다.                                          |
| 2 | 구독이 취소된 후 추가적으로 호출되는 Subscription.request(long n)는 효력이 없어야 한다.                                                                  |
| 3 | 구독이 취소된 후 추가적으로 호출되는 Subscription.request()는 효력이 없어야 한다.                                                                        |
| 4 | 구독이 취소되지 않은 동안 Subscription.request(long n)의 매개변수가 0보다 작거나 같으면 java.lang.IllegalArgumentException 과 함께 onError signal 을 보내야 한다. |
| 5 | 구독이 취소되지 않은 동안 Subscription.cancel()은 Publisher가 Subscriber 에게 보내는 signal을 결국 중지하도록 요청해야 한다.                                    |
| 6 | 구독이 취소되지 않은 동안 Subscription.cancel()은 Publisher에게 해당 구독자에 대한 참조를 결국 삭제하도록 요청해야 한다.                                              |
| 7 | Subscription.cancel(), Subscription.request() 호출에 대한 응답으로 예외를 던지는 것을 허용하지 않는다.                                                  |
| 8 | 구독은 무제한 수의 request 호출을 지원해야 한고 최대 2^63 - 1 개의 Demand 를 지원해야 한다.                                                                 |
- 1번 규칙의 목적은 request 와 onNext 사이의 상호 재귀로 인해 발생할 수 있는 스택 오버플로를 피하기 위해 request 가 다시 호출된다는 것을 분명히 하는것이다.
  - onNext 또는 onSubscribe 내에서 동기적으로 Subscription.request 를 호출한다는 의미는 request 를 호출하는 스레드와 onNext signal 을 보내는 스레드가 동일할 수 있다는 것이다.
- 2번 규칙은 상싱적인 선에서 생각해 봐도 당연한 규칙이라고 볼 수 있다.
  - 예를 들어 어떤 독자가 잡지 정기 구독을 신청하면 매달 한 권의 잡지를 우편으로 전달받는다 가정해보자.
    - 현실 세계에서는 구독자가 매달 잡지를 보내 달라고 요청하지는 않겠지만 암묵적으로 매달 잡지를 보내 주기를 요청한다고 가정하자.
    - 이번 달 잡지를 전달받고 구독을 해지했다면 다음 달에 잡지를 또 보내 달라고 요청하더라도 잡지사에서 잡지를 보내 주지는 않을 것이다.
- 3번 규칙 역시 2번 규칙과 비슷한 의미라고 볼 수 있다.
  - 구독을 취소했으니 또 다시 구독을 취소하더라도 이미 취소할 구독이 없기 때문에 효력이 없어야 하는 것이 자연스러울 것이다.
- 4번 규칙의 목적은 잘못된 구현으로 인해 예외 발생 없이 계속 작업이 진행되는 것을 방지하는 것이다.
  - 다시 말해 요청 개수가 조건에 부합하지 않을 경우 onError signal 을 전달받아서 예외 처리를 할 수 있음을 알 수 있다.
- 5번 규칙을 통해 Subscription.cancel()의 의미가 구독을 취소한다는 의미에서 그치는 것이 아니라 구독 취소를 하니 signal 보내는 것을 중지하라고 Publisher 에게 요청하다는 의미까지 포함함을 알 수 있다.
- 6번 규칙은 5번 규칙에 대한 좀 더 구체적인 내용이라고 볼 수 있다.
  - Subscription.cancel() 을 호출했을 때, Publisher 가 signal 전송을 중지할 뿐만 아니라 구독 취소를 요청한 구독자에 대한 참조까지 삭제한다는 것을 알 수 있다.
  - 이 규칙을 통해서 가비지 컬렉터(Garbage Collector)가 더 이상 유효하지 않은 구독자의 객체를 수집하여 메모리를 확보할 수 있도록 해 준다.
- 7번 규칙은 Subscription 의 메서드를 호출했을 때 메서드 내부로 예외가 던져지지 않도록 규정한다.
  - Java 에서 일반적으로 어떤 메서드를 호출하여 예외가 발생하면 메서드를 호출한 쪽으로 예외를 던지는데, 리엑티브 스트림즈에서는 예외가 발생하면 해당 예외를 onError signal 과 함께 보내도록 규정한다.
  - > 리액티브 스트림즈에서는 메서드 호출 시, 유효한 값 이외에는 어떠한 예외도 던지지 않는다는 의미로 '**Return normally**'라는 용어를 사용한다.
- 8번 규칙을 통해 구독자는 한번 요청할 때 무한한 개수의 데이터를 요청할 수 있고, 그 요청을 끝없이 호출할 수 있음을 알 수 있다.
  - 이렇게 무한히 발생하는 데이터의 흐름을 **무한 스트림(Unbounded Stream)** 이라고 한다.

### 2.6. 리액티브 스트림즈 구현체
> 리액티브 스트림즈의 구현체는 어떤 것들이 있는지 알아보자!
- **RxJava**
  - Rx 는 Reactive Extensions(리액티브 확장)이라는 의미이다.
  - RxJava 는 .NET 환경의 리액티브 확장 라이브러리를 넷플릭스에서 Java 언어로 포팅하여 만든 JVM 기반의 대표적인 리액티브 확장 라이브러리이다.
  - RxJava 는 2.0 부터 리액티브 스트림즈 사양을 지원하기 시작헀는데, 이 때문에 리액티브 스트림즈 사양을 지원하지 않았던 RxJava 1.x와 2.0의 기능들이 함께 사용되고 있따.
    - 1.x 버전과 2.0 이후 버전의 근본적인 동작구조는 크게 차이가 나지 않지만 두 버전에는 **Backpressure 를 지원하느냐 그렇지 않느냐 하는 차이점**이 있다.
- **Project Reactor**
  - Project Reactor(줄여서 Reactor)는 Spring Framework 팀에 의해 주도적으로 개발된 리액티브 스트림즈의 구현체이다.
    - Reactor 3.x 가 Spring F/W 5 버전부터 리액티브 스택에 포함되어 리액티브 프로그래밍의 핵심 역할을 담당한다.
- **Akka Streams**
  - JVM 상에서의 동시성과 분산 애플리케이션을 단순화해 주는 오픈소트 툴킷이다.
  - Akka 는 Actor 로 시작해서 Actor 로 끝난다고 해도 과언이 아닐 정도로 Actor Model 을 적극적으로 사용하는 대표적인 기술이다.
    - 이러한 Actor 들 간의 통신은 메시지를 통해서만 이루어지고 Actor 들을 서로 독립적이기 때문에 느슨한 결합과 높은 응집력이 보장된다.
  - 이 Akka 라는 Actor 기반의 동시성 모델을 사용하는 툴킷 위에 리액티브 스트림즈를 구현한 것이 바로 Akka Streams 이다.
- **Java Flow API**
  - Java 9 부터 Flow API 를 사용하여 리액티브 스트림즈를 지원하게 되었다.
    - 그렇지만 Flow API 는 리액티브 스트림즈를 구현한 다른 구현체들과 차이점이 한 가지 있다.
      - Flow API 는 Reactor, RxJava, Akka Streams 처럼 리액티브 스트림즈를 구현한 구현체가 아니라 리액티브 스티름즈의 표준 사양이 **SPI(Service Provider Interface)** 로써 Java API 에 정의 되어 있다. 
- **그 외 리액티브 확장(Reactive Extension)**
  - RxJava 외에도 현재 구현되어 있는 Reactive Extension 은 많이 있다.
    - 대표적인 예로 Android 플랫폼에서 사용하는 RxAndroid 가 있으면, Javascript 에서 사용하는 RxJS 가 있다.
  - 언어별로 특성이 있긴 하지만 어떤 Reactive Extension 을 학습하더라도 리액티브 프로그래밍의 기본 원리는 동일하기 떄문에 한 가지만 잘 학습하면 Reactive Extension 을 학습하는데 큰 어려움은 없을 것이다.
> 정리
> - 리액티브 스트림즈는 데이터 스트림을 Non-Blocking 이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리 표준사양이다.
> - 리액티브 스트림즈는 Publisher, Subscriber, Subscription, Processor 라는 네 개의 컴포넌트로 구성되어 있다.
>   - 리액티브 스트림즈의 구현체는 이 네 개의 컴포넌트를 사양과 규칙에 맞게 구현해야 한다.
> - Publisher 와 Subscriber 의 동작 과정과 리액티브 스트림즈 컴포넌트의 구현 규칙은 리액티브 프로그래밍을 큰 틀에서 이해하고 올바르게 사용하기 위해 기억해야 하는 중요한 내용이다.
> - 리액티브 스트림즈의 구현체 중에서 어떤 구현체를 학습하든지 핵심 동작 원리는 같다.