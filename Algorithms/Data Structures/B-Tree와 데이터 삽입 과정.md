---
sticker: lucide//waves
tistoryBlogName: ondj
tistoryTitle: "\bB-Tree와 데이터 삽입 과정"
tistoryTags: B-Tree,비트리,b-tree,B tree,B-tree
tistoryVisibility: "0"
tistoryCategory: "1096113"
tistorySkipModal: true
tistoryPostId: "158"
tistoryPostUrl: https://ondj.tistory.com/158
---
## 이진 탐색 트리 BST

**이진 탐색 트리가 가지는 특징**

모든 노드의 왼쪽 서브 트리는 해당 노드의 값보다 작은 값들만 가지고, 모든 노드의 오른쪽 서브 트리는 해당 노드의 값보다 큰 값들만 가질 수 있고 자녀 노드는 최대 두 개까지 가질 수 있다.

![](https://i.imgur.com/ojsxiPq.png)

트리 형태의 자료 구조의 특성상 깊이가 깊어질수록 소모되는 자원이 커질 수 밖에 없기 때문에 최대한 
작은 depth를 통해 많은 자녀 노드를 확보할 필요가 있다.

이 생각을 바탕으로 기존 BST를 확장해 값의 범위를 세분화하게 되면 아래와 같이 범위를 갖는 트리 구조를 얻게 된다.
![](https://i.imgur.com/NxzgeQ4.png)

이 방식을 적용하면 이진 탐색 트리와 동일한 방식으로 동작 하면서도 하나의 노드가 자녀 노드의 개 수를 두 개 이상도 가질 수 있도록 만들 수 있는것이다.

![](https://i.imgur.com/1WjtZ6v.png)

범위를 갖게된 노드를 서브 트리로 확장하면 자연스럽게 범위를 통해 대소 비교가 가능해지고 이런 방식을 통해 전체 트리를 구성하게 된다면 B-Tree 자료 구조가 만들어지게 된다.
## B-Tree 특징

![](https://i.imgur.com/EpUKjdT.png)

- 자녀 노드의 최대 개수를 늘리기 위해 부모 노드에 key를 하나 이상 저장한다.
- 부모 노드의 key들을 오름차순으로 정렬한다.
- 정렬된 순서에 따라 자녀 노드들의 key 값의 범위가 결정된다.

이런 방식을 사용한다면 자녀 노드의 최대 개수를 클라이언트가 결정해 사용할 수 있고 이는 곧 B-Tree를 사용할 때 중요한 파라미터가 된다.

![](https://i.imgur.com/kZkb2Nu.png)

> [!NOTE]
> B-Tree를 설명할 때 기준 파라미터를 어떤 것으로 하는지에 따라 설명이 다를 수 있다.
> 여기서는 주요 파라미터 M을 기준으로 설명한다.

#### B-Tree노드의 key 수와 자녀 수의 관계

- internal 노드의 key 수가 x 개라면 자녀 노드의 수는 언제나 x + 1 개이다.

![](https://i.imgur.com/7hJC8o5.png)

예시 이미지에서 루트 노드의 key는 k1과 k2로 총 두 개인데 자녀 노드 역시 두 개로 이는 B-Tree의 규칙을 위반한 형태이기 때문에 잘못된 구조를 가졌다고 볼 수 있다.

이와 반대로 internal 노드의 key 수가 1개라면 자녀 노드는 두 개가 존재한다.

![](https://i.imgur.com/X69Ea81.png)

이를 통해 우리가 유추할 수 있는 결과는 노드가 최소 하나의 key를 가지기 때문에 몇 차 B-Tree 인지와 무관하게 internal 노드는 최소 두 개의 자녀를 가질 수 있다는 것이다.

> [!NOTE]
>  M(차 수)이 정해지면 root 노드를 제외하고 internal 노드는 최소 [M/2]개의 자녀 노드를 가질 수 있게 된다.

### B-Tree 데이터 삽입 관련 두 가지 포인트

- 데이터의 추가는 항상 leaf 노드에서 이루어진다.
- 노드가 넘치면 가운데(median) key를 기준으로 좌우 key들은 분할하고 가운데 key는 승진한다.
- 노드간 이동이 발생한다면 항상 오름차순 정렬이 이뤄진다.

이 두가지 주요 포인트를 기준으로 3차 B-Tree의 데이터 삽입 과정을 간단하게 살펴보자
### 3차 B-Tree

[3차 B-Tree 데이터 삽입 예제](https://www.youtube.com/watch?v=bqkcoSm_rCs&t=1021s)

B-Tree의 데이터 삽입을 통해 트리가 만들어지는 과정에서 눈에 띄는 점으로 모든 leaf 노드들은 같은 레벨에 있다는 것이다. 

이 말은 곧 B-Tree가 Balanced Tree 라는 것이고 검색과 같은 행위에서 일어나는 시간 복잡도가 `avg/worst case` 모두 `O(logN)`의 복잡도를 갖는다는 것으로 이어진다.

> [!NOTE]
> BST 같은 이진 트리는 Balanced Tree가 아니기 때문에 최악의 경우 O(N)의 시간 복잡도를 갖는다.

[출처 : 쉬운 코드](https://www.youtube.com/watch?v=bqkcoSm_rCs)