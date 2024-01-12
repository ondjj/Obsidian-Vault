---
sticker: lucide//text-selection
---
B-Tree는 정렬된 데이터를 유지하고 로그 시간 내에 검색, 삽입 및 삭제가 가능한 균형 트리 자료 구조로 대용량 데이터를 효율적으로 구성하고 관리하는데 주로 사용된다. 

B-Tree는 여러 키와 하위 포인터가 포함된 노드가 있는 계층 구조로 데이터 세트가 증가하더라도 효율적인 검색 작업을 제공하도록 설계되었고, 밸런싱 유지는 트리의 깊이가 로그 형식으로 유지되도록 보장하여 다양한 작업에 대해 일관된 성능을 유지하는데 도움이 된다.

데이터베이스의 맥락에서 B-Tree는 인덱스를 구현하는데 자주 사용되며, 이를 통해 인덱스된 컬럼을 기반으로 데이터를 빠르게 검색할 수 있다.

# DB Index로 B-Tree 계열이 사용되는 이유
### Self-balancing BST와 B-Tree 시간 복잡도

스스로 균형을 잡지 못하는 BST는 worst case의 경우 O(N)의 시간 복잡도를 갖을 수 있는데 이를 보안한 AVL-Tree와 Red-Black Tree는 O(lonN)의 시간 복잡도를 갖고 BST를 일반화한 B-Tree 계열 역시 조회, 삽입, 삭제 등의 작업을 진행할 때 best case 혹은 worst case 모두 동일한 O(logN)의 시간 복잡도를 갖는다. 

![](https://i.imgur.com/lAxaalI.png)

이를 통해 Self-balancing BST와 B-Tree 모두 BST를 기반으로 확장되었고, 동일한 시간 복잡도를 갖는다는 것을 알 수 있었는데 데이터베이스에서 B-Tree 계열의 자료 구조를 사용하는 이유는 무엇일까?

### Computer System

![](https://i.imgur.com/EHxzZsv.png)

데이터베이스 역시 Secondary Storage에 영구적으로 저장된다.
### Secondary storage의 특징

**1. 데이터를 처리하는 속도가 가장 느리다.**

우리가 짚고 넘어가야할 것은 RAM과 Secondary-storage의 데이터를 처리하는 속도가 상이하다는 것으로 DB가 저장되는  Secondary Storage의 데이터 처리 속도가 더 느리다.
`RAM : 40 ~ 50 GB/s, SSD : 3 ~ 5 GB/s, HDD : 0.2 ~ 0.3 GB/s`
(HDD는 물리적인 장치로 I/0에 따라 헤더가 움직이며 동작하기 때문에 그 속도가 소프트웨어로 구성된 SSD보다 더 느리다.)

**2. 데이터를 저장하는 용량이 가장 크다.**

**3. Block 단위로 데이터를 읽고 쓴다.**

여기서 말하는 block 단위란 file system이 데이터를 읽고 쓰는 논리적인 단위로 크기는 2의 승수로 표현되며 대표적인 block size는 4KB, 8KB, 16KB 등이 있고 데이터를 읽고 쓸때 위와 같은 block 단위를 사용하는 이유는 하나씩 데이터를 읽는것 보다 하나의 단위로 여러 데이터를 한 번에 읽고 사용하는것이 더 효율적이기 때문이다.

여러 개의 데이터를 하나의 단위로 묶어 한 번에 관리하는 것은 언뜻보면 장점만 존재하는것 처럼 보일 수 있지만 다음 예를 통해 반드시 그렇지는 않다는것을 알 수 있다. 

하나의 박스에 다양한 물건을 보관한다고 했을때 내가 필요한 물 컵을 꺼내야한다면
먼저, 박스가 있는 곳으로 이동하고, 여러 박스 중 필요로하는 박스를 선별한 후 잡다한 물건 틈에 끼어 
있는 물 컵을 꺼내는 과정을 거쳐야한다.
물 컵 이외에 다른 것들은 언제 쓰일지 알 수 없는 불필요한 데이터가 되는 것이다.

- Self-balancing BST와 B-Tree의 시간 복잡도는 동일하다.
- Computer System에서 DB가 실제로 저장되는 곳은 Secondary Storage이다.
- 실행 중인 프로그램의 코드와 코드 실행에 필요한 데이터는 RAM에 저장된다.
- RAM은 Secondary Storage보다 용량이 작기 때문에 자주 쓰이지 않는 데이터는 Secondary Storage에 임시 저장한다.
- Secondary Storage는 Block 단위로 데이터를 읽고 쓴다.


