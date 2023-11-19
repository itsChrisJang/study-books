# Part02. Project Reactor
## Chapter09. Sinks
> - Chapter 9에서는 Reactor에서 사용되는 Signal을 프로그래밍 방식으로 전송할 수 있는 Sinks에 대해서 알아보자. 
### 9.1. Sinks란?
- Processor는 Publisher와 Subscriber의 기능을 모두 지니기 때문에 Subscriber로서 기능할 땐 다른 Publisher를 구독할 수 있고, Publisher로서 기능할 땐 다른 Subscriber가 구독할 수 있다.
- Reactor에서는 Processor 인터페이스를 구현한 구현 클래스인 FluxProcessor, MonoProcessor, EmitterProcessor 등을 지원한다.
- 그런데 Processor의 기능을 개선한 Sinks가 Reactor 3.4.0 버전부터 지원되기 시작했으면, Processor와 관련된 API는 Reactor 3.5.0부터 완전히 제거될 예정이다.
#### 그렇다면 Sinks는 주로 어디에 사용할 수 있을까요?
> **Sinks are constructs through which Reactive Streams signals can be programmatically pushed, with Flux or Mono semantics.** These standalone sinks expose tryEmit methods that return an Sinks.EmitResult enum, allowing to atomically fail in case the attempted signal is inconsistent with the spec and/or the state of the sink.
- 볼드체로 표시된 첫 번째 문장의 경우 'Sinks는 리액티브 스트림즈의 Signal을 프로그래밍 방식으로 푸시할 수 있는 구조이며, Flux 또는 Mono의 의미 체계를 가진다.' 정도로 해석할 수 있다.
- 지금까지 배운 방식은 모두 Flux 또는 Mono가 onNext 같은 Signal을 내부적으로 전송해 주는 방식이었는데, Sinks를 사용하면 프로그래밍 코드를 통해 명시적으로 Signal을 전송할 수 있다.
- 그렇지만 Reactor에서 프로그래밍 방식으로 Signal을 전송하는 가장 일반적인 방법은 generate() Operator나 create() Operator 등을 사용하는 것인데, 이는 Reactor에서 Sinks를 지원하기 전부터 이미 사용하던 방식이다.
#### 그렇다면 Sinks를 사용하는 것과 Operator를 사용하는 전통적인 방식에는 어떤 차이점이 있을까요?
- 일반적으로 generate() Operator나 create() Operator는 싱글스레드 기반에서 Signal을 전송하는 데 사용하는 반면, Sinks는 멀티스레드 방식으로 Signal을 전송해도 스레드 안정성을 보장하기 때문에 예기치 않은 동작으로 이어지는 것을 방지해준다.
```java
package chapter9;

import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.publisher.FluxSink;
import reactor.core.scheduler.Schedulers;

import java.util.stream.IntStream;

/**
 * create() Operator를 사용하는 예제
 *  - 일반적으로 Publisher가 단일 쓰레드에서 데이터 생성한다.
 */
@Slf4j
public class Example9_1 {
    public static void main(String[] args) throws InterruptedException {
        int tasks = 6;
        Flux
            .create((FluxSink<String> sink) -> {
                IntStream
                        .range(1, tasks)
                        .forEach(n -> sink.next(doTask(n)));
            })
            .subscribeOn(Schedulers.boundedElastic())
            .doOnNext(n -> log.info("# create(): {}", n))
            .publishOn(Schedulers.parallel())
            .map(result -> result + " success!")
            .doOnNext(n -> log.info("# map(): {}", n))
            .publishOn(Schedulers.parallel())
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(500L);
    }

    private static String doTask(int taskNumber) {
        // now tasking.
        // complete to task.
        return "task " + taskNumber + " result";
    }
}
```
- 먼저 create() Operator가 처리해야 할 작업의 개수만큼 doTask() 메서드를 호출해서 작업을 처리한 후, 결과를 리턴받는다.
- 이 처리 결과를 map() Operator를 사용해서 추가적으로 가공 처리한 후에 최종적으로 Subscriber에게 전달한다. 그런데 위 코드는 작업을 처리하는 단계, 처리 결과를 가공하는 단계, 가공된 결과를 Subscriber에게 전달하는 단계를 모두 별도의 스레드에서 실행하도록 구성했다.
  - 작업을 처리한느 스레드는 subscribeOn() Operator에서 지정하고, 처리 결과를 가공하는 스레드는 13번 라인의 publishOn()에서 지정하며, 가공된 결과를 Subscriber에게 전달하는 스레드는 16번 라인의 publishOn() Operator에서 지정한다.
- 원본 데이터를 생성하는 create() Operatordptjsms subcriberOn() Operator에서 지정한 스레드를 사용해서 생성한 데이터를 emit한다.
```text
16:48:24.219 [boundedElastic-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # create(): task 1 result
16:48:24.228 [boundedElastic-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # create(): task 2 result
16:48:24.228 [boundedElastic-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # create(): task 3 result
16:48:24.228 [boundedElastic-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # create(): task 4 result
16:48:24.228 [boundedElastic-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # create(): task 5 result
16:48:24.245 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 1 result success!
16:48:24.245 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 2 result success!
16:48:24.245 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 3 result success!
16:48:24.245 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 4 result success!
16:48:24.245 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 1 result success!
16:48:24.245 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 5 result success!
16:48:24.245 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 2 result success!
16:48:24.245 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 3 result success!
16:48:24.245 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 4 result success!
16:48:24.245 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 5 result success!
```
- 코드 실행 결과를 보면, doTask() 메서드의 작업처리는 boundedElastic-1 스레드에서 실행되고, map() Operator에서의 가공처리는 parallel-2 스레드에서 실행되며, Subscriber에서 전달받은 데이터의 처리는 parallel-1 스레드에서 실행되었다.
- 이 처럼 create Operator를 사용해서 프로그래밍 방식으로 signal을 전송할 수 있으며, Reactor Sequence를 단계적으로 나누어서 여러 개의 스레드로 처리할 수 있다.
- 그런데 위 코드에서 작업을 처리한 후, 그 결과 값을 반환하는 doTask() 메서드가 싱글스레드가 아닌 여러 개의 스레드에서 각각의 전혀 다른 작업들을 처리한 후, 처리 결과를 반환하는 상황이 발생할 수도 있다.
  - 이 같은 상황에서 적절하게 사용할 수 있는 방식이 바로 Sinks이다.
```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Sinks;
import reactor.core.scheduler.Schedulers;

import java.util.stream.IntStream;

/**
 * Sinks를 사용하는 예제
 *  - Publisher의 데이터 생성을 멀티 쓰레드에서 진행해도 Thread safe 하다.
 */
@Slf4j
public class Example9_2 {
    public static void main(String[] args) throws InterruptedException {
        int tasks = 6;

        Sinks.Many<String> unicastSink = Sinks.many().unicast().onBackpressureBuffer();
        Flux<String> fluxView = unicastSink.asFlux();
        IntStream
                .range(1, tasks)
                .forEach(n -> {
                    try {
                        new Thread(() -> {
                            unicastSink.emitNext(doTask(n), Sinks.EmitFailureHandler.FAIL_FAST);
                            log.info("# emitted: {}", n);
                        }).start();
                        Thread.sleep(100L);
                    } catch (InterruptedException e) {
                        log.error(e.getMessage());
                    }
                });

        fluxView
                .publishOn(Schedulers.parallel())
                .map(result -> result + " success!")
                .doOnNext(n -> log.info("# map(): {}", n))
                .publishOn(Schedulers.parallel())
                .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(200L);
    }

    private static String doTask(int taskNumber) {
        // now tasking.
        // complete to task.
        return "task " + taskNumber + " result";
    }
}
```
- 코드 9-2는 코드 9-1과 달리 doTask() 메서드가 루프를 돌 때마다 새로운 스레드에서 실행된다.
- 그리고 doTask()메 메서드의 작업 처리 결과를 Sinks를 통해서 Downstream에 emit한다.(`unicastSink.emitNext(doTask(n), Sinks.EmitFailureHandler.FAIL_FAST);`)
```text
16:57:33.585 [Thread-0] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # emitted: 1
16:57:33.660 [Thread-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # emitted: 2
16:57:33.765 [Thread-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # emitted: 3
16:57:33.870 [Thread-3] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # emitted: 4
16:57:33.975 [Thread-4] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # emitted: 5
16:57:34.197 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 1 result success!
16:57:34.197 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 2 result success!
16:57:34.197 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 3 result success!
16:57:34.197 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 4 result success!
16:57:34.197 [parallel-2] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # map(): task 5 result success!
16:57:34.197 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 1 result success!
16:57:34.197 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 2 result success!
16:57:34.197 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 3 result success!
16:57:34.197 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 4 result success!
16:57:34.197 [parallel-1] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # onNext: task 5 result success!
```
- 코드 실행 결과를 보면, doTask() 메서드는 루프를 돌 때마다 새로운 스레드에서 실행되기 때문에 총 5개의 스레드(Thread-0 ~ Thread-4)에서 실행되고, map() Operator에서의 가공 처리는 parallel-2 스레드, Subscriber에서 전달 받은 데이터의 처리는 parallel-1 스레드에서 실행되어 코드 9-2는 총 7개의 스레드가 실행되었따.
- 이처럼 Sinks는 프로그래밍 방식으로 Signal을 전송할 수 있으며, 멀티스레드 환경에서 **스레드 안전성(Thread Safety)을 보장받을 수 있는 장점이 있다.**
> ### 스레드 안정성(Thread Safety) 이란?
> - 스레드 안전성이란 함수나 변수 같은 공유 자원에 동시 접근할 경우에도 프로그램의 실행에 문제가 없음을 의미한다.
>   - 공유 변수에 동시에 접근해서 올바르지 않은 값이 할당된다거나 공유 함수에 동시에 접근함으로써 교착 상태, 즉 Dead Lock에 빠지게 되면 스레드 안전성이 깨지게 된다.
> - Processor에서는 onNext, onComplete, onError 메서드를 직접적으로 초훌함으로써 쓰레드 안전성이 보장되지 않을 수 있는데, Sinks의 경우에는 동시 접근을 감지하고, 동시 접근하는 스레드 중 하나가 빠르게 실패함으로써 스레드 안전성을 보장한다.

### 9.2. Sinks 종류 및 특징
- Reactor에서 Sinks를 사용하여 프로그래밍 방식으로 Signal을 전송할 수 있는 방법은 크게 두 가지이다.
  - 첫째는 Sinks.One을 사용하는 것이고, 둘째는 Sinks.Many를 사용하는 것이다.
#### Sinks.One
- Sinks.One은 Sinks.one() 메서드를 사용해서 한 건의 데이터를 전송하는 방법을 정의해 둔 기능 명세라고 말할 수 있다.
  - Sinks.One을 이해하기 위해서는 Sinks.one() 메서드의 내부를 들여다볼 필요가 있다.
```java
public final class Sinks {
    ...
    ...
    public static <T> Sinks.One<T> one() {
        return SinksSpecs.DEFAULT_ROOT_SPEC.one();
    }
}
```
- 위 코드는 Sinks.one() 메서드 내부 코드의 구성 모습이다.
- Sinks.One은 한 건의 데이터를 프로그래밍 방식으로 emit하는 역할을 하기도 하고, Mono 방식으로 Subscriber가 데이터를 소비할 수 있도록 해 주는 Sinks 클래스 내부에 인터페이스로 정의된 Sinks의 스펙 또는 사양으로 불 수 있다.
- 즉, **Sinks.one() 메서드를 호출하는 것은 한 건의 데이터를 프로그래밍 방식으로 emit하는 기능을 사용하고 싶으니 거기에 맞는 적당한 기능 명세를 달라고 요청하는 것과 같다.**
```java
package chapter9;

import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Mono;
import reactor.core.publisher.Sinks;

import static reactor.core.publisher.Sinks.EmitFailureHandler.FAIL_FAST;

/**
 * Sinks.One 예제
 *  - emit 된 데이터 중에서 단 하나의 데이터만 Subscriber에게 전달한다. 나머지 데이터는 Drop 됨.
 */
@Slf4j
public class Example9_4 {
    public static void main(String[] args) throws InterruptedException {
        Sinks.One<String> sinkOne = Sinks.one();
        Mono<String> mono = sinkOne.asMono();

        sinkOne.emitValue("Hello Reactor", FAIL_FAST);
        sinkOne.emitValue("Hi Reactor", FAIL_FAST);
        sinkOne.emitValue(null, FAIL_FAST);

        mono.subscribe(data -> log.info("# Subscriber1 {}", data));
        mono.subscribe(data -> log.info("# Subscriber2 {}", data));
    }
}
```
- 위 코드는 Sinks.one() 메서드를 사용하는 예제이다.
- Sinks.one() 메서드를 호출하면 Sinks.One이라는 기능 명세를 리턴하며(`Sinks.One<String> sinkOne = Sinks.one();`), Sinks.One 객체로 데이터를 emit할 수 있다.(`sinkOne.emitValue("Hello Reactor", FAIL_FAST);`)
- (`sinkOne.emitValue("Hello Reactor", FAIL_FAST);`) 라인에서 emitValue() 메서드의 두 번째 파라미터는 emit 도중에 에러가 발생할 경우 어떻게 처리할 것인지에 대한 핸들러를 나타낸다.
- 얼핏 봐서는 마치 상수 변수 같은 "FAIL_FAST" 라는 값을 지정했지만 Sinks 내부는 아래 코드로 구성되어 있다.
```java
public final class Sinks {
    ...
    ...
    public interface EmitFailureHandler {
        EmitFailureHandler FAIL_FAST = (signalType, emission) -> false;
        boolean onEmitFailure(SignalType signalType, EmitResult emitResult);
    }
}
```
- 위 코드를 보면, FAIL_TEST는 람다 표현식으로 표현한 EmitFailureHandler 인터페이스의 구현 객체이다.
  - 이 EmitFailureHandler 객체를 통해서 emit 도중 발생한 에러에 대해 빠르게 실패 처리한다.
  - 빠르게 실패 처리한다는 의미는 **에러가 발생했을 때 재시도를 하지 않고 즉시 실패 처리를 한다**는 의미이다.
  - 이렇게 빠른 실패 처리를 함으로써 쓰레드 간의 경합 등으로 발생하는 교착 상태 등을 미연에 방지할 수 있는데, 이는 결과적으로 스레드 안전성을 보장하기 위함이다.
- 다시 코드 9-4의 설명으로 돌아가 보면, emit한 데이터를 구독하여 전달받기 위해서 asMono()라는 메서드를 사용해서 Mono 객체로 변환한다.
> Reactor API 문서에서는 asMono() 메서드를 통해 Sinks.One에서 Mono 객체로 변환할 수 있는 것을 'Mono의 의미 체계를 가진다(with Mono semantic)'라고 표현한다.
- 이렇게 변환된 Mono 겍체를 emit된 데이터를 전달받을 수 있다.(`mono.subscribe(data -> log.info("# Subscriber1 {}", data)); mono.subscribe(data -> log.info("# Subscriber2 {}", data));`)
```text
17:32:46.052 [main] DEBUG reactor.core.publisher.Operators - onNextDropped: Hi Reactor
17:32:46.057 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 Hello Reactor
17:32:46.058 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber2 Hello Reactor
```
- Sinks.One으로 emit한 데이터가 두 구독자에게 모두 전달되었다.
- 그렇다면 `sinkOne.emitValue("Hi Reactor", FAIL_FAST);`를 주석 해제하고 다시 실행해보겠다.
```text
17:32:46.052 [main] DEBUG reactor.core.publisher.Operators - onNextDropped: Hi Reactor
17:32:46.057 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 Hello Reactor
17:32:46.058 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber2 Hello Reactor
```
- 결과는 동일하지만, Reactor 내부에서 아주 친절하게 "Hi Reactor" 문자열이 Drop되었다는 디버그 로그를 출력해주고 있다.
- 위 실행 결과를 통해 Sinks.One으로 아무리 많은 수의 데이터를 emit한다 하더라도 **처음 emit한 데이터는 정상적으로 emit되지만 나머지 데이터들은 Drop된다**는 사실을 알 수 있다.
#### Sinks.Many
- Sinks.Many는 Sinks.mamy() 메서드를 사용해서 여러 건의 데이터를 여거 가지 방식으로 전송하는 기능을 정의해 둔 기능 명세라고 볼 수 있다.
```java
public final class Sinks {
    ...
    ...
    public static ManySpec many() {
        return SinksSpecs.DEFAULT_ROOT_SPEC.many();
    }
}
```
- Sinks.one() 메서드의 경우 Sinks.One을 리턴하는 반면에, Sinks.many() 메서드의 경우 Sinks.Many를 리턴하지 않고 ManySpec이라는 인터페이스를 리턴한다.
- Sinks.One의 경우, 단순히 한 건의 데이터를 emit하는 한 가지 기능만 가지기 때문에 별도의 Spec이 정의되는게 아니라 Default Spec(SinksSpecs.DEFAULT_ROOT_SPEC)을 사용한다.
  - 반면에 Sinks.Many의 경우, 데이터를 emit을 위한 여러 가지 기능이 정의된 ManySpec을 리턴한다.
```java
public final class Sinks {
    ...
    ...
    public interface ManySpec {
        UnicastSpec unicast();
        MulticastSpec multicast();
        MulticastReplaySpec replay();
    }
}
```
- ManySpec은 총 세 가지 기능을 정의하는데, 이 세 기능은 각각의 기능을 또다시 별도의 Spec(UnicastSpec, MulticastSpec, MulticastReplaySpec)으로 정의해 두고 있다.
```java
@Slf4j
public class Example9_8 {
  public static void main(String[] args) throws InterruptedException {
    Sinks.Many<Integer> unicastSink = Sinks.many().unicast().onBackpressureBuffer();
    Flux<Integer> fluxView = unicastSink.asFlux();

    unicastSink.emitNext(1, Sinks.EmitFailureHandler.FAIL_FAST);
    unicastSink.emitNext(2, Sinks.EmitFailureHandler.FAIL_FAST);

    fluxView.subscribe(data -> log.info("# Subscriber1 : {}", data));

    unicastSink.emitNext(3, Sinks.EmitFailureHandler.FAIL_FAST);

//    fluxView.subscribe(data -> log.info("# Subscriber2 : {}", data));
  }
}
```
- ManySpec의 구현 메서드인 unicast() 메서드를 호출했다.
  - unicast() 메서드를 호출하면 리턴 값으로 UnicastSpec을 리턴하고 최종적으로 UnicastSpec에 정의된 기능을 사용한다.
  - UnicastSpec에 정의된 기능은 onBackpressureBuffer() 메서드를 호출함으로써 사용하게 된다.
> 네트워크 통신에서 사용하는 Broadcast라는 용어는 네트워크에 연결된 모든 시스템이 정보를 전달받는(One to All) 방식이다.<br/>
> 반면에 Unicast는 하나의 특정 시스템만 정보를 전달받는(One to One) 방식이고, Multicast는 일부 시스템들만 정보를 전달받는(One to Many) 방식이다.<br/>
> 따라서 Unicast의 의미를 가지는 UnicastSpec의 기능은 단 하나의 Subscriber에게만 데이터를 emit하는 것입니다.
- asMono() 메서드를 사용해서 Mono 객체로 변환한 반면, 여기서는 asFlux() 메서드를 사용해서 Flux 객체로 변환한다.
#### 실행 결과
```text
20:37:07.175 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 1
20:37:07.178 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 2
20:37:07.179 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 3
```
#### 실행 결과(15번 라인을 주석 해제한 결과)
```text
20:37:07.175 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 1
20:37:07.178 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 2
20:37:07.179 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 3
20:37:07.195 [main] ERROR reactor.core.publisher.Operators - Operator called default onErrorDropped
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.IllegalStateException: UnicastProcessor allows only a single Subscriber
Caused by: java.lang.IllegalStateException: UnicastProcessor allows only a single Subscriber
	at reactor.core.publisher.UnicastProcessor.subscribe(UnicastProcessor.java:537)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8642)
	at reactor.core.publisher.Flux.subscribeWith(Flux.java:8815)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8608)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8532)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8475)
	at com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test.main(Test.java:25)
```
- 15번 라인의 주석을 해제하고 코드를 실행하면, IllegalStateException이 발생한다.
- 그리고 UnicastProcessor는 하나의 Subscriber만 허용한다는 친절한 메시지까지 보여준다.
  - 이 메시지를 통해서 UnicastSpec에서 단 하나의 Subscriber에게 데이터를 emit하기 위해서 내부적으로 UnicastProcessor를 사용한다는 것을 알 수 있다.
```java
@Slf4j
public class Example9_9 {
    public static void main(String[] args)  {
        Sinks.Many<Integer> multicastSink =
                Sinks.many().multicast().onBackpressureBuffer();
        Flux<Integer> fluxView = multicastSink.asFlux();
      
        multicastSink.emitNext(1, Sinks.EmitFailureHandler.FAIL_FAST);
        multicastSink.emitNext(2, Sinks.EmitFailureHandler.FAIL_FAST);
      
        fluxView.subscribe(data -> log.info("# Subscriber1 : {}", data));
      
        fluxView.subscribe(data -> log.info("# Subscriber2 : {}", data));
      
        multicastSink.emitNext(3, Sinks.EmitFailureHandler.FAIL_FAST);
    }
}
```
- 이번에는 ManySpec의 구현 메서드 중에서 multicast() 메서드를 호출했다.
  - multicast() 메서드를 호출하면 리턴 값으로 MulticastSpec을 리턴하고 MulticastSpec의 구현 메서드인 onBackpressureBuffer() 메서드를 호출한다.
- Multicast가 하나 이상의 일부 시스템들만 정보를 전달받는 방식이라고 설명한 것처럼, MulticastSpec의 기능은 하나 이상의 Subscriber에게 데이터를 emit하는 것이다.
```text
20:42:27.410 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 1
20:42:27.418 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 2
20:42:27.419 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 3
20:42:27.420 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber2 : 3
```
- 실행 결과를 보면 Subscriber1은 emit된 세 개의 데이터 모두를 전달받은 반면에, Subscriber2는 세 번째 데이터만 전달받았다.
- Sinks가 Publisher의 역할을 할 경우 기본적으로 Hot Publisher로 동작하며, 특히 onBackpressureBuffer() 메서드는 Warm up의 특징을 가지는 Hot Sequence로 동작하기 때문에 첫 번째 구독이 발생한 시점(11번 라인)에 Downstream 쪽으로 데이터가 전달되는 것이다.
```java
@Slf4j
public class Example9_10 {
    public static void main(String[] args)  {
        Sinks.Many<Integer> replaySink = Sinks.many().replay().limit(2);
        Flux<Integer> fluxView = replaySink.asFlux();
    
        replaySink.emitNext(1, Sinks.EmitFailureHandler.FAIL_FAST);
        replaySink.emitNext(2, Sinks.EmitFailureHandler.FAIL_FAST);
        replaySink.emitNext(3, Sinks.EmitFailureHandler.FAIL_FAST);
    
        fluxView.subscribe(data -> log.info("# Subscriber1 : {}", data));
    
        replaySink.emitNext(4, Sinks.EmitFailureHandler.FAIL_FAST);
    
        fluxView.subscribe(data -> log.info("# Subscriber2 : {}", data));
    }
}
```
- 코드 9-10에서 ManySpec의 구현 메서드 중에서 replay() 메서드를 호출했다.
- MP3 플레이어와 같은 기기로 replay 버튼을 누르거나 replay 모드 설정을 하면 현재 듣고 있는 음악이 계속 재생되는 것처럼 Sinks에서의 replay 역시 같은 기능을 한다.
- 4번 라인의 replay() 메서드를 호출하면 리턴 값으로 MulticastReplaySpec을 리턴하고, 이 MulticastReplaySpec의 구현 메서드 중 하나의 limit() 메서드를 호출한다.
- MulticastReplaySpec에는 emit된 데이터를 다시 replay해서 구독 전에 이미 emit된 데이터라도 Subscriber가 전달받을 수 있게 하는 다양한 메서드들이 정의되어 있다.
- 대표적으로 all()이라는 메서드가 있는데, all() 메서드는 MP3 플레이어의 replay 버튼과 같은 기능을 한다고 볼 수 있다.
  - 즉, 구독 전에 이미 emit된 데이터가 있더라도 처음 emit된 데이터부터 모든 데이터들이 Subscriber에게 전달된다.
- limit() 메서드는 emit된 데이터 중에서 파라미터로 입력한 개수만큼 가장 나중에 emit된 데이터부터 Subscriber에게 전달하는 기능을 한다.
  - 즉, emit된 데이터 중에서 2개만 뒤로 돌려서(replay) 전달하겠다는 의미이다.
```text
20:58:16.583 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 2
20:58:16.584 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 3
20:58:16.584 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber1 : 4
20:58:16.586 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber2 : 3
20:58:16.586 [main] INFO com.itvillage.rxjavachris.example.reactiveprogrammingdeeply.Test - # Subscriber2 : 4
```
- 실행 결과를 보면, 첫 번째 Subscriber의 입장에서는 구독 시점에 이미 세 개의 데이터가 emit되었기 때문에 마지막 2개를 뒤로 되돌린(replay) 숫자가 2이므로 2부터 전달된다.
- 하지만 두 번째 Subscriber의 경우, 구독 전에 숫자 4의 데이터가 한 번 더 emit되었기 때문에 두 번째 Subscriber의 구독 시점에 마지막 2개를 뒤로 돌린 숫자가 3이므로 3부터 전달된다.
> #### 기억하기
> - Sinks는 Publisher와 Subscriber의 기능을 모두 지닌 Processor의 향상된 기능을 제공한다.
> - 데이터를 emit하는 Sinks에는 크게 Sinks.One과 Sinks.Many가 있다.
> - Sinks.Many는 여러 건의 데이터를 프로그래밍 방식으로 emit한다.
> - Sinks.Many는 여러 건의 데이터를 프로그래밍 방식으로 emit한다.
> - Sinks.Many의 UnicastSpec은 단 하나의 Subscriber에게만 데이터를 emit한다.
> - Sinks.Many의 MulticastSpec은 하나 이상의 Subscriber에게 데이터를 emit한다.
> - Sinks.Many의 MulticastReplaySpec은 emit된 데이터 중에서 특정 시점으로 되돌린(replay) 데이터부터 emit한다.