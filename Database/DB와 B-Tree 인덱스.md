---
sticker: lucide//text-selection
tistoryBlogName: ondj
tistoryTitle: DB와 B-Tree 인덱스
tistoryTags: B-Tree,DB 인덱스,인덱스,B Tree
tistoryVisibility: "3"
tistoryCategory: "1096104"
tistorySkipModal: true
tistoryPostId: "157"
tistoryPostUrl: https://ondj.tistory.com/157
---
B-Tree는 정렬된 데이터를 유지하고 로그 시간 내에 검색, 삽입 및 삭제가 가능한 균형 트리 자료 구조로 대용량 데이터를 효율적으로 구성하고 관리하는데 주로 사용된다. 

B-Tree는 여러 키와 하위 포인터가 포함된 노드가 있는 계층 구조로 데이터 세트가 증가하더라도 효율적인 검색 작업을 제공하도록 설계되었고, 밸런싱 유지는 트리의 깊이가 로그 형식으로 유지되도록 보장하여 다양한 작업에 대해 일관된 성능을 유지하는데 도움이 된다.

데이터베이스의 맥락에서 B-Tree는 인덱스를 구현하는데 자주 사용되며, 이를 통해 인덱스된 컬럼을 기반으로 데이터를 빠르게 검색할 수 있다.

# DB Index로 B-Tree 계열이 사용되는 이유
### Self-balancing BST와 B-Tree 시간 복잡도

스스로 균형을 잡지 못하는 BST는 worst case의 경우 O(N)의 시간 복잡도를 갖을 수 있는데 이를 보안한 AVL-Tree와 Red-Black Tree는 O(lonN)의 시간 복잡도를 갖고 BST를 일반화한 B-Tree 계열 역시 조회, 삽입, 삭제 등의 작업을 진행할 때 best case 혹은 worst case 모두 동일한 O(logN)의 시간 복잡도를 갖는다. 

![](https://i.imgur.com/lAxaalI.png)

이를 통해 Self-balancing BST와 B-Tree 모두 BST를 기반으로 확장되었고, 동일한 시간 복잡도를 갖는다는 것을 알 수 있는데 데이터베이스에서 Self-balancing BST가 아닌 B-Tree 계열의 자료 구조를 사용하는 이유는 무엇일까?

이를 이해하기 위한 기반 지식으로 컴퓨터 시스템에 대해 알아야할 필요가 있다.
### Computer System

![](https://i.imgur.com/EHxzZsv.png)

데이터베이스 역시 Secondary Storage에 영구적으로 저장된다.
### Secondary storage의 특징

**1. 데이터를 처리하는 속도가 가장 느리다.**

우리가 짚고 넘어가야할 것은 RAM과 Secondary-storage의 데이터를 처리하는 속도가 상이하다는 것으로 DB가 저장되는  Secondary Storage의 데이터 처리 속도가 더 느리다.
`RAM : 40 ~ 50 GB/s, SSD : 3 ~ 5 GB/s, HDD : 0.2 ~ 0.3 GB/s`
(HDD는 물리적인 장치로 I/0에 따라 디스크 헤더가 움직이며 동작하기 때문에 그 속도가 소프트웨어로 구성된 SSD보다 더 느리다.)

**2. 데이터를 저장하는 용량이 가장 크다.**

**3. Block 단위로 데이터를 읽고 쓴다.**

여기서 말하는 block 단위란 file system이 데이터를 읽고 쓰는 논리적인 단위로 크기는 2의 승수로 표현되며 대표적인 block size는 4KB, 8KB, 16KB 등이 있고 데이터를 읽고 쓸때 위와 같은 block 단위를 사용하는 이유는 하나씩 데이터를 읽는것 보다 하나의 단위로 여러 데이터를 한 번에 읽고 사용하는것이 더 효율적이기 때문이다.

여러 개의 데이터를 하나의 단위로 묶어 한 번에 관리하는 것은 언뜻보면 장점만 존재하는것 처럼 보일 수 있지만 다음 예를 통해 반드시 그렇지는 않다는것을 알 수 있다. 

하나의 박스에 다양한 물건을 보관한다고 했을때 내가 필요한 물 컵을 꺼내야한다면
먼저, 박스가 있는 곳으로 이동하고 여러 박스 중 필요로하는 박스를 선별한 후 잡다한 물건 틈에 끼어 
있는 물 컵을 꺼내는 과정을 거쳐야한다.

물 컵 이외에 다른 것들은 언제 쓰일지 알 수 없는 불필요한 데이터가 되는 것이다.

지금까지 내용을 정리하면 다음과 같다.
- Self-balancing BST와 B-Tree의 시간 복잡도는 동일하다.
- Computer System에서 DB가 실제로 저장되는 곳은 Secondary Storage이다.
- 실행 중인 프로그램의 코드와 코드 실행에 필요한 데이터는 RAM에 저장된다.
- RAM은 Secondary Storage보다 용량이 작기 때문에 자주 쓰이지 않는 데이터는 Secondary Storage에 임시 저장한다.
- Secondary Storage의 데이터 처리 속도는 가장 느리기 때문에 최대한 적게 접근하는 것이 성능 면에서 좋다.
- Secondary Storage는 block 단위로 읽고 쓰기 때문에 연관된 데이터를 모아서 저장하면 더 효율적으로 읽고 쓸 수 있다.

### B-Tree 계열의 자료 구조를 사용하는 이유

여기서는 AVL Tree 와 B B-Tree로 각각 인덱스를 만들어 데이터를 찾는 과정을 통해 두 자료 구조의 차이에 대해 알아본다.

- Tree의 각 노드는 서로 다른 block에 있다고 가정한다.
- 초기에는 root 노드를 제외한 모든 노드는 secondary storage에 있다고 가정한다.
- 초기에는 데이터 자체도 모두 secondary storage에 있다고 가정한다.

#### AVL Tree Index

![](https://i.imgur.com/hJdwqG1.png)

`b` 컬럼을 기준으로 AVL Tree를 사용하여 인덱스를 생성한다면 아래와 같은 구조를 갖게된다.
(`*, asterisk(애스터리스크)`으로 표시된 부분은 실제 데이터를 가르키는 포인터 역할을 한다.)

![](https://i.imgur.com/BCaXAb1.png)

위 가정에서 루트 노드(6)를 제외한 나머지 데이터는 모두 `secondary storage`에 저장 되어 있다고 가정했고, `5`라는 데이터를 검색한다면 다음과 같은 과정을 거치게 된다.

![](https://i.imgur.com/mZjoICD.png)

찾고자 하는 데이터와 노드의 값이 같지 않다면 일치하는 값을 찾을 때 까지 `secondary storage`에 접근하기 때문에 응답을 포함해 총 4번의 접근이 필요하다.

#### B-Tree Index

다음은 B-Tree 기반으로 인덱스를 만들어 조회하는 과정으로 5차 B-Tree를 통해 조회한다.
AVL 트리와 같이 루트 노드를 제외한 데이터는 모두 `secondary storage`에 저장되어있고 B-Tree의 특성상 루트 노드의 데이터들은 모두 오름차순으로 정렬되어있다.

![](https://i.imgur.com/QwAbJbY.png)

- 먼저 조회하고자 하는 데이터와 루트 노드를 순차적으로 비교한다.
- 루트 노드 4는 조회하고자 하는 데이터 5보다 작다.
- 4가 가르키는 데이터는 조회하지 않는다.
- 루트 노드 8은 조회하고자 하는 데이터 5보다 크다.
- 8이 가르키는 데이터 셋을 조회한다.
- 조회 대상 발견

![](https://i.imgur.com/yPnsBOS.png)

B-Tree를 통해 데이터를 읽을 경우 총 두 번의 접근을 통해 데이터를 조회할 수 있었다.

**주요 차이점**
- AVL 트리는 노드의 데이터 수가 1개로 고정되어있지만, B-Tree는 2~4개이다.
- AVL 트리는 1 ~ 2개의 자녀 노드를 가질 수 있었지만, B-Tree는 3 ~ 5개의 자녀 노드를 가질 수 있었다.
- Secondary Storage는 block 단위로 읽고 쓴다, B-Tree는 연속된 데이터를 가지기 때문에 저장 공간에 대한 활용도가 더 좋다.

결과적으로 B-Tree는 데이터를 찾을 때 탐색 범위를 빠르게 좁힐 수 있고, 저장 공간을 연관된 데이터로 채울 수 있기 때문에 활용도 측면에서도 AVL Tree 보다 더 좋다고 볼 수 있는 것이다.

## About Think

##### Self-balancing BST의 노드들도 Block 안에 최대한 담는다면?

![dm6zShG.png](https://i.imgur.com/dm6zShG.png)

동일한 시간 복잡도를 가지는 AVL 트리를 DB Index에 사용하지 않는 이유 중 하나로 Block, 공간 활용도가 떨어진다는것이 있었다. 

그렇다면 AVL 트리의 노드 역시 Block 안에 담으면 되는것 아닌가? 라는 생각이 들 수 있는데 결국 이 방식이 B-Tree가 동작하는 방식이기 때문에 이는 불필요한 자원의 낭비라고 볼 수 있다.

##### Hash Index의 사용

Hash Index는 삽입, 삭제, 조회의 시간 복잡도가 모두 O(1)로 성능면에서 뛰어나기 때문에 MySQL 등은 Hash Index를 제공하기도 한다.

하지만 Hash Index는 equality(=) 조회만 가능하고 범위 기반 검색이나 정렬에는 사용될 수 없다는 단점이 있다.

[출처 : 쉬운 코드](https://www.youtube.com/watch?v=liPSnc6Wzfk)