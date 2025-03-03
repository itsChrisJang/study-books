# Chapter 02 : 대규모 데이터 처리 입문
## 메모리와 디스크, 웹 애플리케이션과 부하
---
## 강의 4. 하테나 북마크의 데이터 규모
> 데이터가 많을수록 처리에 시간이 걸린다.

## 강의 5. 대규모 데이터 처리의 어려운 점
> 메모리와 디스크
### 대규모 데이터는 어떤 점이 어려운가?
> 메모리 내에서 계산할 수 없다.
- 데이터 건수가 많으면 그만큼 입력 데이터 건수가 늘어나므로 계산량이 많아진다.
  - 문제는 '디스크를 읽고 있다'는 점이다. 
- 수 억건의 규모가 되면 데이터가 너무 많아 메모리 내에서 계산할 수 없으므로 디스크에 두고 특정 데이털르 검색하게 된다.
  - **디스크는 메모리에 비해 상당히 느리다.**

### 메모리와 디스크의 속도차
> 메모리는 10^5 ~ 10^6배 이상 고속(10만 ~ 100만배)

### 디스크는 왜 늦을까?
> 메모리와 디스크
- 메모리는 전기적인 부품이므로 물리적 구조는 탐색속도와 그다지 관계가 없다.
- `1`의 메모리 그림에서 `1` 부분에 '5' 라는 숫자가 들어있다고 하고 `2`에 '0' 이라는 숫자가 들어 있을때, `2`부분을 탐색하다가 `1`부분을 확인하고하 할 때에도 마이크로초(10^-6) 단위로 포인터를 이동시킬 수 있다.
![메모리와 디스크](image/memory_and_disk.png)
- 디스크는 동축 상에 '원반'(disk)이 쌓여 있다.
  - 이 원반이 회전하면서 데이터를 읽어낸다.
  - 즉, 메모리와 달리 회전 등의 물리적인 동작을 수반하고 있다.
    - 이 구조가 탐색 속도에 영향을 준다.

#### 탐색속도에 영향을 주는 다양한 요인
- 디스크에서는 헤드의 이동과 원반의 회전이라는 두 가지 물리적인 이동이 필요하지만, 오늘날의 기술로도 원반의 회전속도를 빛의 속도까지 근접시킬 수는 없다.
  - 디스크에서는 헤드의 이동과 원반의 회전 각각 밀리초 단위(10^-3) 단위, 합해서 수 밀리초나 걸린다. 
  - **메모리는 1회 탐색할 때 마이크로초면 되지만, 디스크는 수 밀리초가 걸리는 것이다.**
#### OS 레벨에서의 연구
- `7`과 같이 OS는 연속된 데이터를 같은 위치에 쌓는다.
  - 데이터를 읽을 때 4KB 정도를 한꺼번에 읽도록 되어 있다.
- 이렇게 비슷한 데이터를 비슷한 곳에 두어 1번의 디스크 회전으로 읽는 데이터 수를 많게 한다.
  - 그 결과로 디스크의 회전횟수를 최소화할 수 있게 된다.
  - 그러기에 디스크를 가능한 한 회전시키지 않아도 된다.
  - 그렇지만 결국 회전 1회당 밀리초 단위이므로 역시 메모리와의 속도차를 피할 수 있는 것은 아니다.
#### 전송속도, 버스의 속도차
- ㅇ

---
