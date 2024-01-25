---
sticker: lucide//hash
tistoryBlogName: ondj
tistoryTitle: Set과 Hash Set
tistoryTags: Set, Hash Set, list와 set의 차이
tistoryVisibility: "0"
tistoryCategory: "1096113"
tistorySkipModal: true
tistoryPostId: "161"
tistoryPostUrl: https://ondj.tistory.com/161
---
# Set

실무에서 자주 쓰이는 자료구조는 대표적으로 List와 Set이 존재한다. 그 중 Set은 데이터를 저장하는 추상 자료형(ADT)로 순서를 보장하지 않고 데이터 중복을 허용하지 않는다는 특징이 있다.
데이터 조회(search)에 있어서 List 보다 더 빠른 속도를 가지는데 Set 자료형은 `O(1)`의 시간 복잡도를 갖는다.

중복된 데이터를 제거해야 할 때 Set을 사용하기 좋으며 데이터의 존재 여부를 확인할 필요가 있을때 역시 Set 사용을 고려할 수 있다.(이 경우 필터링이 필요한 상황을 떠올려보자)

## Set의 구현체

set을 구현한 구현체는 크게 세 가지로 `Hash Set, Linked Hash Set, Tree Set`이 존재하는데
그 중 `Hash Set`에 대해 다뤄보자 `Hash Set`은 해시 테이블(Hash Table)을 사용하여 구현되고
해시 테이블은 배열과 Hash Function을 통해 구현된다.

![](https://i.imgur.com/yQZHACU.png)

Hash Table은 구조상 일반적으로 테이블의 크기에 상관 없이 Key를 통해 상수 시간동안 빠르게 데이터에 접근이 가능하고 Hash Set 역시 Hash Table을 사용하여 구현하기 때문에 크기와 무관하게 데이터 조회 속도가 빠르다.

### Java에서의 Hash Set

![](https://i.imgur.com/i94b502.png)

자바에서 제공하는 `Hash Set` 클래스의 생성자를 살펴보면 내부적으로 `Hash Map`을 사용하고있기 때문에 `Hash Set과 Hash Map`은 동일하다. 

결국 `Key : Value` 구조인 `Hash Map`과의 차이점은 `value 값`의 유무이고 `Hash Set`의 경우 `value 값`으로 더미 값을 넣는 형태로 이루어진다.
(`Hash Set`은 key의 값에 데이터가 들어가게된다.)

![](https://i.imgur.com/ggno21E.png)

![](https://i.imgur.com/wIOzHhs.png)

조회의 경우 Key에 데이터가 들어가는 특성에 따라 Key값이 존재하는지 확인하는 과정을 통해 조회할 수 있다.

![](https://i.imgur.com/FzDBqGX.png)

> [!NOTE]
> CPython의 경우 python이 제공하는 딕셔너리를 통해 구현하는것이 아닌 해쉬 테이블을 사용하는것과 동시에 최적화된 형태로 set만의 자체 구조를 가지고 있다.

![](https://i.imgur.com/9rMcEev.png)

### List와 Set 중 무엇을 사용해야 할까?

Set을 사용하는게 더 적절한 상황이 아니라면 거의 대부분 List를 사용한다고 봐도 무방하다.
그렇다면 데이터들 자체에 이미 중복이 없고, 순서 역시 상관 없이 데이터를 순회하면서 특정 작업만을 반복해야 하는 경우에는 List와 Set 중 어떤걸 사용하는게 좋을까?

이 경우 List가 메모리도 적게 사용하고, 구현 특성 상 List가 단순하여 iteration이 더 빠르기 때문에 이 경우에도 List를 사용하는것을 추천한다.

해쉬 테이블로 구현된 Set의 경우 List와 달리 데이터가 연속적이지않고, 2차원 배열 형태의 표 구조로 row를 탐색해야 하기 때문에 `over head`가 발생할 여지가 존재한다.

자바의 경우 이러한 문제를 해결하기 위해 `Linked Hash Set`을 통해 해당 문제를 해결 하지만 유효한 데이터간의 연결을 위해 Linked List의 추가 구현이 필요하기 때문에 더 많은 메모리를 사용하게 된다. 
트리 셋 역시 추가적인 비용이 필요하다.

![](https://i.imgur.com/T3b0bW0.png)


> [!NOTE]
> 같은 시간복잡도 O(1)인데 hashTable을 이용한 get이 ArrayList를 get보다 빠른 이유가 무엇인가요?
> 
> hash table의 get과 list의 get은 같은 get이 아니다. 
> 리스트의 get은 파라미터로 인덱스를 받는다. 즉, 리스트에서 몇 번째 위치에 있는 데이터를 가져오라는 의미로 해석할 수 있고, 해시테이블에서 get은 파라미터로 key를 받기 때문에 그 key에 대응하는 value를 가지고 오라는 의미이다.
> 
> 단순히 Index 번호를 통해 데이터를 찾는것과 Key 값에 대응하는 value를 가져오는것에서 나오는 차이라고 볼 수 있다.



