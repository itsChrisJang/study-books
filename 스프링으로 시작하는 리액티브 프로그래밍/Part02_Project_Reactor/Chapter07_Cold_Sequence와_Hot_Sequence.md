# Part02. Project Reactor
## Chapter07. Cold Sequence 와 Hot Sequence
> - Chapter 7에서는 리액티브 프로그래밍에서의 Cold 와 Hot 의 의미를 살펴보고, Reactor 에서 지원하는 Cold Sequence 와 Hot Sequence 대해 알아보자. 
### 7.1. Cold 와 Hot 의 의미
- 대표적인 용어를 예시로 Hot 과 Cold 의 의미를 파악해보자.
  - Hot Swap 은 컴퓨터 시스템의 전원이 켜져 있는 상태에서 디스크 등의 장치를 교체할 경우 시스템을 재시작하지 않고서도 바로 장치를 인식하는 것을 의미한다.
  - Hot Deploy는 서버를 재시작하지 않고서 응용 프로그램의 변경 사항을 적용할 수 있는 기능을 말한다.
  - 암호화폐의 지갑을 의미하는 Wallet 은 Cold Wallet 과 How Wallet 으로 구분 지을 수 있는데 여기서도 Hot 과 Cold 라는 단어가 사용된다.
    - How Wallet 은 인터넷에 연결되어 있기 때문에 즉시 사용이 가능하지만 보안에 취약하고, Cold Wallet 은 인터넷과 단절되어 있어서 사용성은 떨어지지만 보안이 강화된다는 특성이 있다.
- Hot 은 무언가 처음부터 다시 시작하지 않고, 같은 작업이 반복되지 않는 느낌이 난다.
  - 서버나 시스템을 다시 가동할 필요가 없고, 인터넷을 다시 연결할 필요 없이 바로 사용 가능하다는 의미로 받아 들일 수 있다.
- 반면에 Cold 는 처음부터 새로 시작해야 하고, 새로 시작하기 때문에 같은 작업이 반볼될 것이다.
  - 서버나 시스템을 부팅할 때마다 초기화 작업을 매번 해야 되거나 암호 화폐의 송금을 위해서 Cold Wallet 을 사용하려면 인터넷에 다시 연결해야 하는 등의 상황을 떠올려 보면 이해되리라 생각한다.
> **Cold는 무언가를 새로 시작하고, Hot은 무언가를 새로 시작하지 않는다.**

### 7.2. Cold Sequence
- Sequence 는 Publisher 가 emit 하는 데이터의 연속적인 흐름을 정의해 놓은 것으로 코드로 표현하면 Operator 체인 형태로 정의된다.
  - 그렇다면 이런 데이터의 흐름에 Cold 나 Hot 의 개념이 포함된다면 아마도 데이터 흐름에 무언가 차이점이 있을 거라고 예상할 수 있다.
- Cold 는 무언가를 새로 시작하는 것이라고 설명했는데, 이 의미를 Cold Sequence 에 적용해 보면 아마도 Sequence 가 새로 시작한다 정도로 생각해 볼 수 있다.
- Cold Sequence 는 Subscriber 가 구독할 때마다 데이터 흐름이 처음부터 다시 시작되는 Sequence 이다.
- Subscriber의 구독 시점이 달라도 구독을 할 때마다 Publisher가 데이터를 emit하는 과정을 처음부터 다시 시작하는 데이터의 흐름을 **Cold Sequence** 라고 부릅니다.
  - 그리고 이 Cold Sequence 흐름으로 동작하는 Publisher 를 Cold Publisher 라고 합니다.

### 7.3. Hot Sequence
- Hot은 무언가를 새로 시작하지 않는 것이다.
- Cold Sequence의 경우 구독이 발생할 때마다 Sequence의 타임라인이 청므부터 새로 시작하기 때문에 Subscriber는 구독 시점과 상관없이 데이터를 처음부터 다시 전달받을 수 있다.
- 반면에 Hot Sequence의 경우 구독이 발생한 시점 이전에 Publisher로부터 emit된 데이터는 Subscriber가 전달받지 못하고 구독이 발생한 시점 이후에 emit된 데이터만 전달 받을 수 있다.
  - Hot Sequence의 경우 구독이 아무리 많이 발생해도 Publisher가 데이터를 처음부터 emit하지 않는다는 것을 의미한다.
- 즉, Publisher가 데이터를 emit하는 과정이 한 번만 일어나고, Subscriber가 각각의 구독 시점 이후에 emit된 데이터만 전달받는 데이터의 흐름을 Hot Sequence라고 한다.
  - Hot Sequence의 경우 Cold Sequence와 반대로 구독 횟수와 상관없이 Sequence 타임라인이 하나만 생긴다고 볼 수 있다.
> share<br/>
> public final Flux`<T>` share() <br/>
> <br/><br/>
> **Returns a new Flux that multicasts (shares) the original Flux.** As long as there is at least one Subscriber this Flux will be subscribed and emitting data. When all subscribers have cancelled it will cancel the source Flux.<br/>
> - **여러 Subscriber가 하나의 원본 Flux를 공유한다**라는 의미이다.<br>
> - 하나의 원본 Flux를 공유해서 다 같이 사용하기 때문에 어떤 Subscriber가 이 원본 Flux를 먼저 구독해 버리면 데이터 emit을 시작하게 되고, 이후에 다른 Subscriber가 구독하는 시점에는 원본 Flux에서 이미 emit된 데이터를 전달받을 수 없게 된다.

### 7.4. HTTP 요청과 응답에서 Cold Sequence와 Hot Sequence의 동작 흐름
```java
package chapter7;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.util.UriComponentsBuilder;
import reactor.core.publisher.Mono;

import java.net.URI;

@Slf4j
public class Example7_3 {
    public static void main(String[] args) throws InterruptedException {
        URI worldTimeUri = UriComponentsBuilder.newInstance().scheme("http")
                .host("worldtimeapi.org")
                .port(80)
                .path("/api/timezone/Asia/Seoul")
                .build()
                .encode()
                .toUri();

        Mono<String> mono = getWorldTime(worldTimeUri);
        mono.subscribe(dateTime -> log.info("# dateTime 1: {}", dateTime));
        Thread.sleep(2000);
        mono.subscribe(dateTime -> log.info("# dateTime 2: {}", dateTime));

        Thread.sleep(2000);
    }

    private static Mono<String> getWorldTime(URI worldTimeUri) {
        return WebClient.create()
                .get()
                .uri(worldTimeUri)
                .retrieve()
                .bodyToMono(String.class)
                .map(response -> {
                    DocumentContext jsonContext = JsonPath.parse(response);
                    String dateTime = jsonContext.read("$.datetime");
                    return dateTime;
                });
    }
}
```
- 12번 라인에서 getWorldTime() 메서드를 호출해서 리턴 값으로 Mono를 전달받는다.
  - Cold Sequence에서는 구독이 발생하지 않으면 데이터의 emit이 일어나지 않기 때문에 13번 라인에서 첫 번째 구독을 발생시키고, 2초의 지연 시간 후에 15번 라인에서 두 번째 구독을 발생기켰다.
- 구독이 발생할 때마다 데이터의 emit 과정이 처음부터 새로 시작되는 Cold Sequence의 특징으로 인해, 두 번의 구독이 발생했으므로 두 번의 새로운 HTTP 요청이 발생한다.
```java
package chapter7;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.util.UriComponentsBuilder;
import reactor.core.publisher.Mono;

import java.net.URI;

@Slf4j
public class Example7_4 {
    public static void main(String[] args) throws InterruptedException {
        URI worldTimeUri = UriComponentsBuilder.newInstance().scheme("http")
                .host("worldtimeapi.org")
                .port(80)
                .path("/api/timezone/Asia/Seoul")
                .build()
                .encode()
                .toUri();

        Mono<String> mono = getWorldTime(worldTimeUri).cache();
        mono.subscribe(dateTime -> log.info("# dateTime 1: {}", dateTime));
        Thread.sleep(2000);
        mono.subscribe(dateTime -> log.info("# dateTime 2: {}", dateTime));

        Thread.sleep(2000);
    }

    private static Mono<String> getWorldTime(URI worldTimeUri) {
        return WebClient.create()
                .get()
                .uri(worldTimeUri)
                .retrieve()
                .bodyToMono(String.class)
                .map(response -> {
                    DocumentContext jsonContext = JsonPath.parse(response);
                    String dateTime = jsonContext.read("$.datetime");
                    return dateTime;
                });
    }
}
```
- cache() Operator를 추가함으로써 Cold Sequence가 Hot Sequence로 동작하게 된다.
> cache<br/>
> public final Mono`<T>` cache() <br/>
> <br/><br/>
> **Turn this Mono into a hot source and cache last emitted signals for further Subscriber.** Completion and Error will alse be replayed.<br/>
- 위 설명에서는 hot source라고 표현했는데 Hot Sequence와 동일한 의미라고 생각하면 된다.
- Cache() Operator는 Cold Sequence로 동작하는 Mono를 Hot Sequence로 변경해 주고 emit된 데이터를 캐시한 뒤, 구독이 발생할 때마다 캐시된 데이터를 전달한다.
- 결과적으로 캐시된 데이터를 전달하기 때문에 구독이 발생할 때마다 Subscriber는 동일한 데이터를 전달받게 된다.
- 이처럼 cache() Operator를 잘 활용할 수 있는 대표적인 예로는 바로 REST API 요청을 위해서 인증 토큰이 필요한 경우를 들 수 있다.
  - 예를 들어 getAuthToken()이라는 메서드를 호출해서 API 서버로부터 인증 토큰을 전달받는다면 토큰이 만료되기 전까지 해당 인증 토큰을 사용해서 인증이 필요한 API 요청에 사용할 수 있다.
  - 그런데 getAuthToken() 메서드를 호출할 때마다 API 서버 쪽에서 매번 새로운 인증 토큰을 전송하게 되어 불필요한 HTTP 요청이 발생할 것이다.
  - 이를 방지하기 위해 Cache() Operator를 사용해서 캐시된 인증 토큰을 사용하여 효율적인 동작 과정을 구성할 수 있을 것이다.
> **Reactor에서 Hot의 두 가지 의미**
> - Reactor에서의 Hot에는 사실 두 가지 의미가 있다 볼 수 있다.
> - 최초 구독이 발생하기 전까지는 데이터의 emit이 발생하지 않는 것과 구독 여부와 상관없이 데이터가 emit 되는것 이렇게 두 가지로 구분할 수 있다.
> - Reactor에서는 전자의 의미를 **Warm up**이라 표현하고, 후자의 의미를 Hot이라 표현한다.
>   - share()의 경우, 최초 구독이 발생했을 때 데이터를 emit 하는 Warm up의 의미를 가지는 Hot Sequence라고 할 수 있다.

> **기억해야 할 것!**
> - Subscriber의 구독 시점이 달라도 구독을 할 때마다 Publisher가 데이터를 처음부터 emit하는 과정을 Cold Sequence라고 한다.
> - Cold Sequence 흐름으로 동작하는 Publisher를 Cold Publisher라고 한다.
> - Publisher가 데이터를 emit하는 과정이 한 번만 일어나고, Subscriber가 각각의 구독 시점 이후에 emit된 데이터만 전달받는 것을 Hot Sequence라고 한다.
> - Hot Sequence 흐름으로 동작하는 Publisher를 Hob Publisher라고 한다.
> - share(), cache() 등의 Operator를 사용해서 Cold Sequence를 Hot Sequence로 변환할 수 있다.
> - Hot Sequence는 Subscriber의 최초 구독이 발생해야 Publisher가 데이터를 emit하는 Warm up과 Subscriber의 구독 여부와 상관없이 데이터를 emit하는 Hot으로 구분할 수 있다.