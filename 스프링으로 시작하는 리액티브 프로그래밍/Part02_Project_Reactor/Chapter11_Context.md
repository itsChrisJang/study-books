# Part02. Project Reactor
## Chapter 11. Context
> Chapter 11에서는 Reactor Sequence에서 상태 값을 젖아하고 저장된 상태 값을 사용할 수 있게 해 주는 Context에 대해 알아보자.
### 11.1. Context란?
- 프로그래밍 학습을 하다 보면 종종 듣게 되는 용어 중 하나가 바로 Context이다.
  - Context를 번역하면 "문맥", "맥락"이다.
  - 그렇지만 "문맥", "맥락"이라는 의미 만으로 프로그래밍에서 사용하는 Context를 이해하려면 도저히 매치가 잘 되지 않는다.
  - "어떤 것을 이해하는 데 도움이 될 만한 관련 정보나 이벤트, 상황" 정도로 해석할 수 있다.
  - **즉, Context는 어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보**라고 할 수 있다.
- Reactor에서의 Context란 무엇을 뜻하는 것일까?
  - Reactor API 공식 문서
    - > A key/value store that is propagated between components such as operators via the context protocol.
    - 즉, Reactor의 Context는 Operator 같은 Reactor 구성요소 간에 전파되는 key/value 혀애의 저장소라고 정의한다.
      - 여기서 '전파'는 Downstream에서 Upstream으로 Context가 전파되어 Operator 체인상의 각 Operator가 해당 Context의 정보를 동일하게 이용할 수 있음을 의미한다.
  - Reactor의 Context는 ThreadLocal과 다소 유사한 면이 있지만, 각각의 실행 스레드와 매핑되는 ThreadLocal과 달리 실행 스레드와 매핑되는 것이 아니라 **Subscriber와 매핑**된다.
  - 즉, **구독이 발생할 때마다 해당 구독과 연결된 하나의 Context가 생긴다**라고 보면 된다.
```java
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Mono;
import reactor.core.scheduler.Schedulers;

/**
 * Context 기본 예제
 *  - contextWrite() Operator로 Context에 데이터 쓰기 작업을 할 수 있다.
 *  - Context.put()으로 Context에 데이터를 쓸 수 있다.
 *  - deferContextual() Operator로 Context에 데이터 읽기 작업을 할 수 있다.
 *  - Context.get()으로 Context에서 데이터를 읽을 수 있다.
 *  - transformDeferredContextual() Operator로 Operator 중간에서 Context에 데이터 읽기 작업을 할 수 있다.
 */
@Slf4j
public class Example11_1 {
    public static void main(String[] args) throws InterruptedException {
        Mono
            .deferContextual(ctx ->
                Mono
                    .just("Hello" + " " + ctx.get("firstName"))
                    .doOnNext(data -> log.info("# just doOnNext : {}", data))
            )
            .subscribeOn(Schedulers.boundedElastic())
            .publishOn(Schedulers.parallel())
            .transformDeferredContextual(
                    (mono, ctx) -> mono.map(data -> data + " " + ctx.get("lastName"))
            )
            .contextWrite(context -> context.put("lastName", "Jobs"))
            .contextWrite(context -> context.put("firstName", "Steve"))
            .subscribe(data -> log.info("# onNext: {}", data));

        Thread.sleep(100L);
    }
}
```
- Reactor Sequence상에서 Context를 사용하는 방법을 이해하기 위한 기본 예제이다.
- Context에 key/value 형태의 데이터를 저장할 수 있다는 의미는 Context에 데이터의 쓰기와 읽기가 가능하다는 의미인데, 위 코드는 사실상 Context에 데이터를 쓰고 읽는 방법을 코드로 구현한 것이 전부이다.
#### 컨텍스트에 데이터 쓰기
p. 240
#### Context에 쓰인 데이터 읽기