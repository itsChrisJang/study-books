# 스프링으로 시작하는 리액티브 프로그래밍
## Chapter03. 리액티브 프로그래밍을 위한 사전 지식
> - 리액티브 프로그래밍을 잘 사용하기 위해서 기본적으로 알아야 하는 최소한의 사전 지식이 함수형 프로그래밍 기법이다.
>   - Java 8 이전까지는 함수형 프로그래밍이라는 개념이 없었으나, Java 8에 람다 표현식이 도입됨으로써 Java 에서도 함수형 프로그래밍 기법을 사용할 수 있게 되었다.
>   - 더 정확히 얘기하자면 함수형 인터페이스를 이용해서 함수형 프로그래밍과 같은 효과를 낼 수 있게 되었다.
### 4.1. 함수형 인터페이스(Functional Interface)
- Java 에서 인터페이스는 명세 또는 사양이라고 표현할 수 있다.
  - 리액티브 스트림즈에서 정의해 놓은 네 개의 인터페이스 처럼, Java 의 인터페이스는 몸체가 없는 추상 메서드로만 이루어져 있다.
  - 물론 Java 8 부터 지원하는 default 메서드는 예외이다.
  - 아무튼 이 인터페이스를 사용하기 위해서는 해당 인터페이스에 정의된 메서드들을 구현하는 구현 클래스가 필요하다.
> - Q : Java 8 부터 지원하는 함수형 인터페이스란 무엇일까?
> - 함수형 인터페이스 역시 인터페이스이다.
>   - 다만 기존 인터페이스에 비해 함수형 인터페이스는 단 하나의 추상 메서드만 정의되어 있다.
>   - 그렇다면 왜 그냥 인터페이스라고 부르지 않고 굳이 함수형 인터페이스라고 부를까?
>     - 함수형 프로그래밍 세계에서 함수를 일급 시민으로 취급한다.
>     - 여기서 말하는 입급 시민이란 함수를 값으로 취급한다는 의미를 조금 더 고급스럽게 표현한 것인데, 함수를 값으로 취급하기 때문에 어떤 함수를 호출할 때 함수 자체를 파라미터로 전달할 수 있다.
>   - Java 에서도 이렇게 함수를 값으로 취급할 수 있는 기능이 Java 8 부터 추가되었는데, 그것이 바로 함수형 인터페이스이다.
```java
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

/**
 * 기존에 사용되던 단 하나의 추상 메서드를 가지는 인터페이스 예제
 */
public class Example4_1 {
    public static void main(String[] args) {
        List<CryptoCurrency> cryptoCurrencies = SampleData.cryptoCurrencies;

        Collections.sort(cryptoCurrencies, new Comparator<CryptoCurrency>() {
            @Override
            public int compare(CryptoCurrency cc1, CryptoCurrency cc2) {
                return cc1.getUnit().name().compareTo(cc2.getUnit().name());
            }
        });

        for(CryptoCurrency cryptoCurrency : cryptoCurrencies)
            System.out.println("암호 화폐명: " + cryptoCurrency.getName() +
                    ", 가격: " + cryptoCurrency.getUnit());
    }
}
```
- Comparator 인터페이스는 Java 1.2 버전부터 많이 사용해 오던 인터페이스인데, 이 Comparator 인터페이스가 바로 compare() 라는 단 하나의 추상 메서드를 가지는 인터페이스, 바로 함수형 인터페이스가 되는 것이다.
```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    ...
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    
    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T>) & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        }
    }
    ...
}
```
- Comparator 내부에는 compare()라는 단 한나의 추상 메서드가 정의된 것이 보인다.
- compare() 외에 reversed(), thenComparing() 같은 메서드들이 보이는데 이 메서드들은 Java 8 부터 지원하는 이미 구현된 default 메서드이기 때문에, 추상 메서드는 하나만 정의되어 있으므로 함수형 인터페이스가 맞다.
  - @FunctionalInterface 애너테이션을 추가해서 이 인터페이스는 함수형 인터페이스라고 아예 명시적으로 지정해 둔 것을 볼 수 있다.
- 그리고 인터페이스 자체를 익명 구현 객체로 전달받는 방식은 코드 자체가 길어져서 지저분하게 보이기도 한다.
  - 그래서 Java 8 버전부터 기존에 이미 사용하던 인터페이스의 익명 구현 객체를 전달하던 방식을 조금 더 함수형 프로그래밍 방식에 맞게 표현할 수 있는 방법을 추가했는데 그것이 **람다 표현식(Lambda Expression)** 이다.
### 4.2. 람다 표현식(Lambda Expression)
-
