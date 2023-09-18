
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

어떤 스레드가 큐를 수정하는 경우 여러 스레드가 동시에 PriorityQueue 인스턴스에 액세스하지 않아야 합니다. 
대신 `thread-safe`한  `java.util.current.PriorityBlockingQue 클래스`를 사용하십시오.  

구현 참고: 
이 구현은 `enqueuing, dequeuing` `method(offer, poll, remove() and add)`에 O(log(n))의 시간을 제공합니다.

대기열 및 대기열 해제 방법(offer, poll, remove() 및 add)에 대한 O(log(n)) 시간, 제거(Object) 방법 및 포함(Object) 방법에 대한 선형 시간, 검색 방법(피크, 요소 및 크기)에 대한 일정한 시간을 제공합니다
