## 목표

- Java에서 사용되는 우선 큐에 대해 알아보고 구현 한다.

## 목차

1. [Priority Queue란?](#Priority Queue란?)
2. [생성자](#생성자)
3. [주요 메서드](#주요 메서드)
4. [백준 1927번 최소 힙 문제](# 백준 1927번 최소 힙 문제)
5. [Priority Queue 구현](#Priority Queue 구현)


# Priority Queue란?

우선 순위 힙을 기반으로 하는 무제한 우선 순위 큐입니다. 

- 우선 순위 큐의 요소는 사용되는 생성자에 따라  `natural ordering`에 따라 정렬되거나 큐 생성 시 제공되는 `Comparator`에 의해 정렬됩니다. 
- 우선 순위 큐는 null 요소를 허용하지 않습니다.  `natural ordering`에 의존하는 우선 순위 큐는 비교할 수 없는 개체의 삽입도 허용하지 않습니다(이렇게 하면 ClassCastException이 발생할 수 있음).  
- 큐의 `head` 부분은 지정된 순서에 대한 최소 요소입니다. 여러 요소가 최소값으로 묶여 있으면 `head` 부분이 해당 요소 중 하나입니다. (ties are broken arbitrarily). 
- 큐 검색 작업은 큐의 `head` 부분에 있는 요소를 poll, remove, peek 를 통해 요소에 액세스합니다.  
  
우선 순위 큐의 크기는 제한이 없지만 큐의 요소를 저장하는 데 사용되는 배열의 크기를 제어하는 내부 용량이 있고 이는 항상 큐 크기만큼 유지됩니다. 
(요소가 우선 순위 큐에 추가되면 해당 용량이 자동으로 증가합니다. `growth policy`에 대한 세부 정보는 지정되지 않았습니다.)
  
이 `class`와 `iterator implement`는 `Collection 및 Iterator 인터페이스`의 모든 옵션 메서드를 구현합니다. 
`Iterator provided` 의 메서드 `iterator()`와 `Spliterator provided`의 메서드 `spliterator()`는 우선순위 큐의 요소를 특정 순서로 적용하는 것을 보장하지 않습니다.
  
**이 구현은 동기화되지 않습니다. **

어떤 스레드가 큐를 수정하는 경우 여러 스레드가 동시에 `PriorityQueue` 인스턴스에 액세스하지 않아야 합니다. 
대신 `thread-safe`한  `java.util.current.PriorityBlockingQue 클래스`를 사용하십시오.  

**구현 참고: **

이 구현은 `enqueuing, dequeuing` `method(offer, poll, remove() and add)`에 O(log(n))의 시간을 제공합니다.
`remove(Object)와 contains(Object) method`에는 `linear time`을 보장하고 `retrieval methods(peek, element, and size)`는 `constant time`이 적용됩니다.

> [!NOTE]
> 위 내용은 Java Doc에서 발췌한 내용으로 자바에서 최소 힙을 구현할 때 사용하는 Priority Queue에 대해 상세하게 기재되어 있다.
> 핵심 내용을 추려 알아보자

`PriorityQueue`는 다음과 같은 특징을 가지고 있다.

![[Screenshot 2023-09-18 at 6.32.10 PM.png]]

- 데이터 삽입 시 우선 순위에 따라 정렬되어 저장된다.
  (우선순위 큐가 `comparator`에 의해 정렬되거나, 만약 `comparator`가 `null`이면 원소의 자연 순서에 따라 정렬된다.)

- 가장 우선 순위가 높은 데이터는 항상 큐의 맨 앞에 위치하며, 이 데이터를 꺼낼 때 가장 먼저 나온다.
  (우선순위 큐가 비어있지 않다면 가장 낮은 값을 가진 원소가 배열 `queue[0]`에 위치한다는 것을 말해줍니다. 이것은 최소 힙에서의 특성입니다.)

- 내부적으로 `balanced Binary Heap`을 사용하여 구현되어 있으므로 삽입과 삭제 연산의 시간 복잡도가 O(log N)이다.
  (`balanced binary heap` 특성 중 하나로 원소들이 정렬되는 자료구조로 각 노드 'n'의 자식 노드들이 배열 'queue'의 인덱스 '2*n+1'과 '2*(n+1)'에 위치한다는 것을 설명하고 있다.)

`PriorityQueue`는 기본적으로 작은 값을 우선 순위가 높은 것으로 간주하며, 이를 변경하려면 `Comparator`를 사용하여 정렬 방식을 커스터마이즈 할 수 있다.

다양한 알고리즘 문제를 해결하는 데 유용하지만, 주로 다익스트라 알고리즘과 같은 그래프 알고리즘에서 사용된다.


# 생성자

``` java
/**  
 * Creates a {@code PriorityQueue} with the default initial  
 * capacity (11) that orders its elements according to their * {@linkplain Comparable natural ordering}.  
 */
 public PriorityQueue() {  
    this(DEFAULT_INITIAL_CAPACITY, null);
}

`PriorityQueue()`: 
기본 생성자로, 초기 용량을 11로 설정하고, 원소들을 자연 순서에 따라 정렬합니다. 
즉, 원소들은 `Comparable` 인터페이스에 정의된 자연 순서에 따라 정렬됩니다.
```


```java
/**  
 * Creates a {@code PriorityQueue} with the specified initial  
 * capacity that orders its elements according to their * {@linkplain Comparable natural ordering}.  
 * * @param initialCapacity the initial capacity for this priority queue  
 * @throws IllegalArgumentException if {@code initialCapacity} is less  
 *         than 1 */
public PriorityQueue(int initialCapacity) {  
    this(initialCapacity, null);  
}
``PriorityQueue(int initialCapacity)`: 
초기 용량을 지정할 수 있는 생성자로, 원소들을 자연 순서에 따라 정렬합니다. 
이 생성자는 `initialCapacity` 매개변수를 통해 초기 큐의 용량을 설정할 수 있습니다. 
`initialCapacity`가 1보다 작으면 `IllegalArgumentException`이 발생합니다.

```

```java
/**  
 * Creates a {@code PriorityQueue} with the default initial capacity and  
 * whose elements are ordered according to the specified comparator. * * @param  comparator the comparator that will be used to order this  
 *         priority queue.  If {@code null}, the {@linkplain Comparable  
 *         natural ordering} of the elements will be used. * @since 1.8  
 */
public PriorityQueue(Comparator<? super E> comparator) {  
    this(DEFAULT_INITIAL_CAPACITY, comparator);  
}

`PriorityQueue(Comparator<? super E> comparator)`: 
기본 생성자와 같지만, 원소들을 지정된 `comparator`를 사용하여 정렬합니다. 
만약 `comparator`가 `null`이면 원소들은 `Comparable` 인터페이스에 정의된 자연 순서에 따라 정렬됩니다. 
이 생성자는 Java 1.8부터 사용 가능합니다.
```

```java
/**  
 * Creates a {@code PriorityQueue} with the specified initial capacity  
 * that orders its elements according to the specified comparator. * * @param  initialCapacity the initial capacity for this priority queue  
 * @param  comparator the comparator that will be used to order this  
 *         priority queue.  If {@code null}, the {@linkplain Comparable  
 *         natural ordering} of the elements will be used. * @throws IllegalArgumentException if {@code initialCapacity} is  
 *         less than 1 */
public PriorityQueue(int initialCapacity,  
                     Comparator<? super E> comparator) {  
    // Note: This restriction of at least one is not actually needed,  
    // but continues for 1.5 compatibility    if (initialCapacity < 1)  
        throw new IllegalArgumentException();  
    this.queue = new Object[initialCapacity];  
    this.comparator = comparator;  
}

`PriorityQueue(int initialCapacity, Comparator<? super E> comparator)`: 
초기 용량과 `comparator`를 모두 설정할 수 있는 생성자입니다. 
이 생성자는 초기 큐의 용량을 `initialCapacity`로 설정하고, 원소들을 지정된 `comparator`를 사용하여 정렬합니다. 
`initialCapacity`가 1보다 작으면 `IllegalArgumentException`이 발생합니다.
```

```java
/**  
 * Creates a {@code PriorityQueue} containing the elements in the  
 * specified collection.  If the specified collection is an instance of * a {@link SortedSet} or is another {@code PriorityQueue}, this  
 * priority queue will be ordered according to the same ordering. * Otherwise, this priority queue will be ordered according to the * {@linkplain Comparable natural ordering} of its elements.  
 * * @param  c the collection whose elements are to be placed  
 *         into this priority queue * @throws ClassCastException if elements of the specified collection  
 *         cannot be compared to one another according to the priority *         queue's ordering * @throws NullPointerException if the specified collection or any  
 *         of its elements are null */
public PriorityQueue(Collection<? extends E> c) {  
    if (c instanceof SortedSet<?>) {  
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;  
        this.comparator = (Comparator<? super E>) ss.comparator();  
        initElementsFromCollection(ss);  
    }  
    else if (c instanceof PriorityQueue<?>) {  
        PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;  
        this.comparator = (Comparator<? super E>) pq.comparator();  
        initFromPriorityQueue(pq);  
    }  
    else {  
        this.comparator = null;  
        initFromCollection(c);  
    }  
}

`PriorityQueue(Collection<? extends E> c)`: 
지정된 컬렉션 `c`의 요소를 포함하는 우선순위 큐를 생성합니다. 
만약 `c`가 `SortedSet`의 인스턴스이거나 다른 `PriorityQueue`의 인스턴스인 경우, 이 우선순위 큐는 동일한 정렬 방식에 따라 정렬됩니다. 
그렇지 않으면 원소들은 자연 순서에 따라 정렬됩니다. `c`의 요소들은 `PriorityQueue`에 추가될 때 정렬 기준에 따라 적절하게 배치됩니다.
```

```java
/**  
 * Creates a {@code PriorityQueue} containing the elements in the  
 * specified priority queue.  This priority queue will be * ordered according to the same ordering as the given priority * queue. * * @param  c the priority queue whose elements are to be placed  
 *         into this priority queue * @throws ClassCastException if elements of {@code c} cannot be  
 *         compared to one another according to {@code c}'s  
 *         ordering * @throws NullPointerException if the specified priority queue or any  
 *         of its elements are null */
public PriorityQueue(PriorityQueue<? extends E> c) {  
    this.comparator = (Comparator<? super E>) c.comparator();  
    initFromPriorityQueue(c);  
}

`PriorityQueue(PriorityQueue<? extends E> c)`: 
지정된 다른 `PriorityQueue` `c`의 요소를 포함하는 우선순위 큐를 생성합니다. 
이 우선순위 큐는 `c`와 동일한 정렬 방식에 따라 정렬됩니다.
```

``` java
/**  
 * Creates a {@code PriorityQueue} containing the elements in the  
 * specified sorted set.   This priority queue will be ordered * according to the same ordering as the given sorted set. * * @param  c the sorted set whose elements are to be placed  
 *         into this priority queue * @throws ClassCastException if elements of the specified sorted  
 *         set cannot be compared to one another according to the *         sorted set's ordering * @throws NullPointerException if the specified sorted set or any  
 *         of its elements are null *
 */
public PriorityQueue(SortedSet<? extends E> c) {  
    this.comparator = (Comparator<? super E>) c.comparator();  
    initElementsFromCollection(c);  
}

`PriorityQueue(SortedSet<? extends E> c)`: 
지정된 정렬된 집합(`SortedSet`) `c`의 요소를 포함하는 우선순위 큐를 생성합니다. 
이 우선순위 큐는 `c`와 동일한 정렬 방식에 따라 정렬됩니다.
```

# 주요 메서드

1. `add(E e)` 또는 `offer(E e)`: 원소를 큐에 추가합니다. 원소는 우선순위에 따라 정렬됩니다.
    
2. `poll()`: 가장 우선순위가 높은 원소를 큐에서 제거하고 반환합니다.
    
3. `peek()`: 큐에서 가장 우선순위가 높은 원소를 반환하지만 제거하지 않습니다. 큐가 비어있을 경우 `null`을 반환합니다.
    
4. `remove(Object o)`: 지정된 원소를 큐에서 제거합니다.
    
5. `isEmpty()`: 큐가 비어있는지 여부를 확인합니다.
    
6. `size()`: 큐에 저장된 원소의 개수를 반환합니다.
    
7. `clear()`: 큐를 비웁니다.
    
8. `toArray()`: 큐에 저장된 원소를 배열로 반환합니다. 반환된 배열은 정렬되지 않은 상태입니다.
    
9. `toArray(T[] a)`: 큐에 저장된 원소를 주어진 배열 `a`에 복사합니다. 만약 주어진 배열의 크기가 충분하지 않다면 새로운 배열을 생성하여 복사합니다.
    
10. `comparator()`: 우선순위 큐의 정렬을 수행하는 `Comparator` 객체를 반환합니다. 만약 `PriorityQueue`가 기본 정렬 기준을 사용하는 경우 `null`을 반환합니다.


# 백준 1927번 최소 힙 문제

[백준 최소 힙](https://www.acmicpc.net/problem/1927)

```java
package backjun;  
  
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
import java.util.PriorityQueue;  
  
public class MinHeap {  
  
    public static void main(String[] args) throws IOException {  
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));  
        int n = Integer.parseInt(br.readLine());  
        PriorityQueue<Integer> priQueue = new PriorityQueue<>();  
  
        for (int i = 0; i < n; i++) {  
            int temp = Integer.parseInt(br.readLine());  
            if (temp != 0) {  
                priQueue.add(temp);  
            } else if (priQueue.isEmpty()) {  
                System.out.println(0);  
            }else {  
                System.out.println(priQueue.poll());  
            }  
        }  
    }  
}
```

문제 자체가 최소 힙을 사용하기만 하면 쉽게 풀 수 있는 문제로 구성되어있다.

위에서 알아본 `Comparator`를 적용해 우선 큐의 오더 순서를 변경해보자

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Comparator;
import java.util.PriorityQueue;

public class MaxHeap {

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());

        // Comparator를 사용하여 내림차순으로 정렬되는 PriorityQueue 생성
        PriorityQueue<Integer> priQueue = new PriorityQueue<>(Comparator.reverseOrder());

        for (int i = 0; i < n; i++) {
            int temp = Integer.parseInt(br.readLine());
            if (temp != 0) {
                priQueue.add(temp);
            } else if (priQueue.isEmpty()) {
                System.out.println(0);
            } else {
                System.out.println(priQueue.poll());
            }
        }
    }
}

```

# Priority Queue 구현

[ST-LAB](https://st-lab.tistory.com/219)
