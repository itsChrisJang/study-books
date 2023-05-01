# Part02. Project Reactor
## Chapter05. Reactor 개요
> - Reactor 가 무엇인지 살펴보고 Reactor 에서 사용되는 용어를 간단히 정리해보자.
### 5.1. Reactor 란?
- Reactor 는 Spring F/W 팀의 주도하에 개발된 리액티브 스트림즈의 구현체로서 Spring F/W 5 버전부터 리액티브 스택에 포함되어 Spring Webflux 기반의 리액티브 애플리케이션을 제작하기 위한 핵심 역할을 담당한다.
- 리액티브 스트림즈의 구현체인 Reactor 는 리액티브 프로그래밍을 위한 라이브러리라고 정의할 수 있다.
  - 따라서 Reactor Core 라이브러리는 Spring Webflux 프레임워크에 라이브러리로 포함되어 있다.
- Reactor 특징
  1. **Reactive Streams**
    - Reactor 는 리액티브 스트림즈(Reactive Streams) 사양을 구현한 리액티브 라이브러리이다.
  2. **Non-Blocking**
    - Reactor 는 JVM 위에서 실행되는 Non-Blocking 애플리케이션을 제작하기 위해 필요한 핵심 기술이다.
  3. **Java's functional API**
    - Reactor 에서 Publisher 와 Subscriber 간의 상호 작용은 Java 의 함수형 프로그래밍 API 를 통해서 이루어진다.
  4. **Flux[N]**
    - Reactor 의 Publisher 타입은 크게 두 가지인데, 그중 한가지가 Flux 이다.
    - Flux[N]이라는 의미는 N개의 데이터를 emit 한다는 것인데, 다시 말해서 Flux 는 0개부터 N개, 즉 무한대의 데이터를 emit 할 수 있는 Reactor 의 Publisher 이다.
  5. **Mono[0|1]**
    - Reactor 의 Publisher 타입은 크게 두 가지인데, 그중 한가지가 Mono 이다.
    - Mono[0|1]과 같이 표현된 이유는 Mono 가 데이터를 한 건도 emit 하지 않거나 단 한 건만 emit 하는 단발성 데이터 emit 에 특화된 Publisher 이기 때문이다.
  6. **Well-suited for microservices**
    - 마이크로 서비스 기반 시스템이 Non-Blocking I/O 에 적합한 시스템 중 하나라고 설명했다.
    - Non-Blocking I/O 특징을 가지는 Reactor 는 마이크로 서비스 기반 시스템에서 수많은 서비스들 간에 지속적으로 발생하는 I/O를 처리하기에 적합한 기술이다.
  7. **Backpressure-ready network**
    - Reactor 는 Publisher 로부터 전달받은 데이터를 처리하는 데 있어 과부하가 걸리지 않도록 제어하는 Backpressure 를 지원한다.
    - Backpressure 는 우리말로 배압이라고도 하는데 현실 세계에서는 배관으로 유입되는 가스나 액체 등의 흐름을 제어하기 위해 역으로 가해지는 압력을 의미한다.
    - 리액티브 프로그래밍에서의 Backpressure 역시 현실 세계에서의 배압과 비슷한 의미를 가진다.
      - Publisher 로부터 전달되는 대량의 데이터를 Subscriber 가 적절하게 처리하기 위한 제어 방법이 바로 Backpressure 이기 때문이다.

### 5.2. Hello Reactor 코드로 보는 Reactor 의 구성요소
```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;

/**
 * Hello Reactor 예제
 */
@Slf4j
public class Example5_1 {
    public static void main(String[] args) {
        Flux<String> sequence = Flux.just("Hello", "Reactor");
        sequence.map(data -> data.toLowerCase())
                .subscribe(data -> System.out.println(data));
    }
}
``` 
- just 메서드의 파라미터로 전달한 "Hello", "Reactor"는 입력으로 들어오는 데이터이다.
  - 이 데이터는 Publisher 가 최초로 제공하는 가공되지 않은 데이터로서 **데이터 소스**라고 불린다.
  - 이 데이터 소스의 데이터 개수가 둘이기 때문에 N 건의 데이터를 처리할 수 있는 Reactor 의 Publisher 타입인 Flux 를 사용했다.
- 데이터를 제공하는 Publisher 가 있으니 당연히 Subscriber 도 존재할 것이다.
  - 5번 라인의 subscribe 메서드의 파라미터로 전달된 람다 표현식인 data -> System.out.println(data) 가 바로 Subscriber 역할을 한다.
  - 이 람다 표현식은 Consumer 함수형 인터페이스이며, 내부적으로 LambdaSubscriber 라는 클래스에 전달되어 데이터를 처리하는 역할을 하게 된다.
- 그리고 3번 라인의 just(), 4번 라인의 map()은 Reactor 에서 지원하는 Operator 메서드인데, just() Operator 는 데이터를 생성해서 제공하는 역할을 하고, map() Operator 는 전달받은 데이터를 가공하는 역할을 한다.
- 그리고 3번 라인에서 just() Operator 의 리턴 값이 Flux 임을 확인할 수 있다.
  - 이를 통해 Reactor 의 Operator 는 리턴 값으로 Flux(또는 Mono)를 반환하기 때문에 Operator 체인을 형성하여 또 다른 Operator 를 연속적으로 호출할 수 있다는 사실을 알 수 있다.
> **데이터를 생성해서 제공하고(1단계), 데이터를 가공한 후에(2단계) 전달받은 데이터를 처리한다(3단계)**