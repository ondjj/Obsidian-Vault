---
sticker: lucide//list-ordered
---
# 온라인 SQL과 배치 SQL

온라인 SQL과 대규모 배치 SQL은 시기, 범위, 실행 특성에 따라 SQL 쿼리를 실행하는 다양한 방법을 나타내는 방식을 말한다.

### 온라인 SQL

이는 일반적으로 즉각적인 사용자 요청이나 진행 중인 시스템간 상호 작용에 대한 응답으로 실시간으로 실행되는 SQL 쿼리 작업을 의미한다.
온라인 SQL 쿼리는 사용자가 상호 작용을 기반으로 즉각적인 피드백이나 데이터 검색을 기대하는 웹 애플리케이션과 같이 사용자 작업에 대한 빠른 응답이 필요한 앱 또는 시스템과 관련되는 경우가 많기 때문에 이와 관련된 쿼리는 요청 시 필요한 특정 데이터 검색, 업데이트 또는 삽입을 포함하며 규모가 작은 편에 속한다. 이런 특성으로 온라인 SQL은 원활한 사용자 경험을 보장하기 위해 짧은 대기 시간과 응답성에 최적화되어야 한다.

### 대규모 배치 SQL

대규모 배치 SQL 작업은 일반적으로 상당한 양의 데이터를 처리하는 대규모 SQL 쿼리 실행을 이야기 한다.
이런 유형의 SQL은 데이터 마이그레이션, 데이터 웨어하우징 작업 또는 대규모 데이터 세트와 관련된 분석과 같은 대량 작업을 수행하는데 사용 되기 때문에
배치 SQL 쿼리에는 여러 테이블이나 데이터베이스에 걸친 복잡한 집계, 조인 또는 변환이 포함되는 경우가 많다. 관련된 데이터의 양이 많기 때문에 실행하는데 시간이 오래 걸릴 수 있으며 최적화는 효율적인 리소스 활용, 병렬 처리 및 전체 실행 시간 최소화에 초점을 맞추는 경우가 많다.

결과적으로 온라인 SQL은 즉각적인 사용자 상호 작용과 소규모 작업에 맞춰져 있는 반면  배치 SQL은 상당량의 데이터 볼륨을 처리하고 배치 프로세스에 자주 사용되므로 주요한 차이점은 SQL 쿼리의 규모, 목적 및 타이밍에 있다고 할 수 있다.

# 온라인 Select문 튜닝 방법론

## 1. 적절한 인덱스를 사용하여 Block I/O를 최소화 하라

조인이 없는 경우는 적절한 인덱스를 사용하는것 만으로도 상당한 효과를 볼 수 있다. 조인이 있는 경우 특히 Driving(선행) 집합에 신경 써야 한다.
왜냐하면 Nested Loop 조인을 사용했고, 선행 집합의 건수가 많다면, 후행 집합의 조인의 시도 횟 수가 증가하므로 성능이 느려지기 때문이다.
따라서 적절한 인덱스를 이용하여 선행 집합의 건수를 줄인다면, 혹은 가장 적은 집합을 선행으로 놓는다면, 후행 집합으로의 조인 건수는 줄어든다.
물론 이때에도 후행 집합의 적절한 인덱스는 필수 조건으로 Driving 집합의 Block I/O를 줄이기 위해 최적화된 인덱스가 없다면 생성하고, 있다면 그것을 사용해 최적의 Access Path를 만들어야한다.

> [!NOTE] 조인 작업 및 중첩 루프 조인
> 본질적으로 조인 시도 횟 수는 실제로 조인할 행 수에 따라 일정하게 유지 되지만 성능에 미치는 영향은 선행 세트의 크기에 따라 다르게 나타난다. 선행 세트의 크기가 클 수록 더 많은 반복이 발생하고 리소스 소비가 증가하며 잠재적으로 성능 저하를 일으키게 되는 요인이 된다.


DBMS는 쿼리를 파싱하고, 해당 쿼리를 수행하기 위한 최적의 방법을 선택하는 기능이 내장되어있다. MySQL은 엔진 내에서 SQL optimizer가 이 역할을 담당한다. 다양한 알고리즘들이 있지만 MySQL은 Nested Loop와 그 파생 알고리즘만을 지원한다고 한다.(8.0.18부터 hash join도 지원한다.) 


![](https://i.imgur.com/0BksjP7.png)

Nested Loop 는 이름에서 알 수 있듯 중첩 루프 구조를 가지고 있다. A 테이블의 레코드 하나 하나에 대해 B 테이블의 레코드를 스캔하고, 조인 조건에 맞는 레코드를 리턴한다. 테이블의 레코드 수를 R(테이블)이라고 한다면 `R(A) * R(B)`를 소요하는 것이다.

선행 집합의 수와 후행 집합의 수에 무관하게 총 조인 횟수는  `R(A) * R(B)` 만큼의 연산을 수행함에도 선행 테이블(Driving Table)의 선택이 중요한데,
이는 반복이 미치는 영향과 리소스 활용, 조인 성능과 연결된다. 위에서도 언급 했듯이 총 반복 횟수가 일정하게 유지되더라도 선행 집합의 크기는 반복 프로세스가 발생해야하는 횟수에 영향을 미친다. 선행 테이블이 작을수록 반복 횟수가 줄어들어 조인 프로세스가 더 빠르고 효율적으로 동작하는것이다.
또한 선행 세트가 클수록 더 많은 리소스가 필요하다. 선행 집합이 큰 경우 엔진은 후행 집합의 각 레코드에 대해 이를 반복해야하고, 이로 인해 리소스 소비가 증하여 처리 시간이 길어지게 된다.

선행 테이블의 선택은 조인 작업에 전반적인 성능에 영향을 준다. 조인 시도 횟수는 일정하게 유지되지만 더 작고 잘 인덱싱된 선행 집합을 선택하면 개별 조인 시도의 효율성과 속도가 크게 향상되어 전반적인 성능이 향상될 수 있다. 따라서 총 반복 횟수는 고정되어 있지만 리소스 활용도를 최적화 하고 인덱스를 효과적으로 활용하며 중첩 루프 구조 내에서 개별 조인 시도의 효율성을 향상 시키려면 최적의 선행 집합을 선택하는 것이 중요하다고 할 수 있다.

단순히 중첩 반복을 통해 동작하지만, 최적화를 하기 위해  `BKA(Batched Key Access), BNL(Block Nested Loop)`과 같은 방식들이 사용되고 있다. 또한 기본적으로는 `Index`를 활용하는 방법도 있다. `Driving Table`의 각 레코드 마다 조인 조건에 부합하는 레코드를 찾아낼 때 `Index`를 사용할 수 있도록 하면, 자원 관리를 더 용이하게 할 수 있다.

지금까지의 내용을 요약하면 전체를 검색 대상에 올리는 선행 집합(Driving Table)은 작게 유지하고 선행 집합의 `Join Key`는 `Index`를 사용하도록 하는 것이 가장 기본적인 튜닝 방법 중 하나이다.

이 때 `Index`를 사용해 스캔하는 것도 해당 `Index`의 유니크 여부에 따라 나뉘게 된다.
`Join key`로 사용되는 `Index가 Unique` 하다는 것이 보장된다면, `Index unique scan`을 사용한다. 이 경우가 가장 빠르고 최적화된 상태라고 볼 수 있다. 다음으로 `Index`가 `Unique` 하지 않다면 `Index range scan`을 사용한다. `unique scan` 보다는 느리지만, 테이블 전체를 `full scan`하는 것보단 효율적이다.

> [!note] 
> Driven table이 비대해지면 결국 지연이 발생하게되는데 이 때는 오히려 driving table을 크게 만드는 것도 방법이 된다고 한다. 
> 선행 집합을 전체 full scan하기는 하겠지만, driven table에 대응 하는 레코드를 찾는 것이 얼마 안걸리기 때문이다. 
> 점포, 주문을 예시로 들자면, 점포 ID는 unique하므로, index unique scan이 적용되어 최적화가 가능하다.  다른 방법으로는 hash join을 사용하는 방법이 있다고 한다.


## 2.조인 방법과 조인 순서를 최적화 하라

온라인 SQL에서 사용하는 `Select문`은 좁은 범위를 검색하는 경우가 많다. 이럴 때는 대부분 `Nested Loop Join`이 유리하기 때문에 조인 건수가 소량인 SQL에 `Hash Join`이나 `Sort Merge Join`이 발견되면 `Nested Loop Join`으로 변경하는 것이 더 유리한지 검토할 필요가 있다.

`Nested Loop Join`에서 가장 중요한 것은 조인 순서이다. `From절`에 테이블(집합)이 두 개라면 후행 집합의 관점에서는 적절한 인덱스만 존재한다면 충분하다. 만약 `From절`에 테이블이 세 개 이상이라면 조인 순서를 변경할 수 있는지에 대해 고려해야한다. 이 때 적용할 수 있는 두 가지 원리가 있고 조인할 테이블이 세 개, 네 개 이상이 되더라도 이 두 가지 원리는 동일하게 적용할 수 있다.

- 후행 집합에 적절한 인덱스가 없는 경우 조인 순서를 바꾸면 최적의 인덱스를 사용할 수 있는 경우가 많다.
  튜닝 전의 조인 순서가 `A - B - C `라고 한다면, 중간 집합인 `B`에 적절한 인덱스가 없고 오히려 `C`에 적절한 인덱스가 존재하는 경우가 있다.
  이럴때는 `B`에 인덱스를 무작정 생성하는것이 아닌 조인 순서를 `A - C - B`로 바꿀 수 있는지, 바꿀 수 있다면 더 효율적인지 검증해야한다.
  조인 순서만 바꾸어도 불필요한 리소스 낭비가 줄어드는 케이스이다. 만약 조인 순서를 바꿀 수 없거나 `C`를 중간 집합으로 변경하는 것이 비효율적이라면
  `B`에 적절한 인덱스를 추가해준다.
  
- 조인되는 집합 중 특정 인덱스에서 `Block I/O`가 증가하는 경우엔 조인 순서 변경을 고려할 필요가 있다.
  튜닝 전의 조인 순서가 `A - B - C` 이고, `B`에서 `Block I/O량`이 증가하면 `A - C - B`로 변경한다. 
  `C`를 먼저 조인(Filter)하여 선행 집합의 건수를 줄이고 `B`에 조인하여 성능 향상을 기대하는 것이다.

> [!help]  Hash Join과 Sort Merge Join
> Hash Join은 메모리를 통해 해시 테이블을 구축한다. 
> 이는 대규모 배치 SQL의 경우 효율적일 수 있으나 빠른 응답이 중요한 온라인 SQL에서는 과도한 메모리 사용으로 시스템 응답성에 악영향을 끼칠 수 있다.
> 좁은 범위 처리와 선택적 대상 검색이 주요한 온라인 SQL의 경우 해시 테이블 작성에 따른 이점 보단, 오버 헤드로 인해 효율이 떨어진다.
> 
> Sort Merge Join에는 두 입력 집합을 정렬하는 작업이 포함되어있다. 이는 CPU 및 메모리 리소스 측면에서 비용 소모가 수반되는 작업이고 
> 이미 정렬되어 있거나 소규모 데이터 세트에서 Sort Merge Join의 사용은 오버헤드의 원인이 될 수 있다.
> 

## 3. Table Access(Random Access)를 최소화 하라

`Random Access`란 `rowid`로 테이블을 엑세스하는 것을 말한다. 1 - 2번 작업을 진행했다면 `Random Access` 역시 많이 줄어든 상태이지만 여전히 만족스러운 성능이 나오지 않는다면 `Random Access` 횟 수를 의심해야한다.

인덱스를 사용하면 `rowid`가 자동으로 획득 되고, 만약 인덱스에 없는 컬럼을 `Select` 해야 한다면 `rowid`로 테이블을 액세스 해야 한다. 
이 때 테이블로 엑세스 해야 할 건수가 많고, 인덱스의 컬럼순으로 테이블이 정렬 되어 있지 않다면 성능이 크게 저하된다. 테이블이 인덱스 기준으로 정렬 되어 있지 않기 때문에 테이블을 방문할 때마다 서로 다른 블럭을 읽어야 하기 때문이다.

디스크 헤드를 떠올려 보자 인덱스의 `rowid`로 테이블을 방문할 때, 테이블이 인덱스를 기준으로 정렬되어 연달아 붙어있다면 연속적으로 스캔이 가능하기 때문에 성능이 빠르고 그 반대의 경우 성능이 느려진다. 바로 이런 이유로 `Index scan`보다 `Table scan`의 속도가 더 느린 것이다.
결과적으로 `Random Access`의 부하를 최소화 해야 한다.

`Random Access`의 부하를 줄이는 방법은 네 가지가 존재한다.

- 테이블의 종류를 변경한다.
  `IOT`나 클러스터를 이용하면 `Clustering Factor`가 극단적으로 좋아진다. 또 한 파티션을 이용하면 같은 범위의 데이터를 밀집 시킬 수 있다.
  
- 효율적인 인덱스를 사용하거나 조인 방법과 순서를 조정하여 `Table Access`를 최소화 한다.(1 - 2번 참고)

- 인덱스에 컬럼을 추가하여 `Table Access`를 방지한다.
  예를 들어 `Select 절`의 특정 컬럼 때문에 테이블에 액세스 된다면, 인덱스의 마지막에 그 컬럼을 추가하면 된다.
  
- 인덱스만 엑세스 하고 테이블로의 엑세스는 모든 조인을 끝낸 후 마지막에 시도하여 `Random Access`를 줄인다.[모든 상식을 의심하라 ]( https://scidb.tistory.com/entry/%EB%AA%A8%EB%93%A0-%EC%83%81%EC%8B%9D%EC%9D%84-%EC%9D%98%EC%8B%AC%ED%95%98%EB%9D%BC)

1 - 3번을 통해 최적의 `Access Path`와 `Join`을 사용했다면, `Block I/O` 관점에서 튜닝은 완료했다.

> [!note] 
> 데이터베이스 관리 측면에서 IOT는 인덱스 구성 테이블(Index-Organized Table)을 의미한다.
> 인덱스 구성 테이블은 인덱스 구조 내에 데이터가 저장되는 테이터베이스의 테이블 유형으로 데이터가 인덱스와 별도로 저장되는 기존 테이블과 달리
> IOT에서는 데이터 자체가 인덱스 구조 내에 할당된다. 즉, 인덱스는 행에 대한 포인터 뿐만 아니라 실제 행 데이터도 저장하므로 단일 구조에서 인데스과 데이터 정보 모두 다 효율적으로 검색할 수 있다. IOT는 빠른 인덱스 기반 액세스와 I/O 작업 감소가 필수적인 작업에서 주로 사용된다.
> 
> 클러스터링은 클러스터링된 테이블 또는 인덱스의 개념을 말한다. 데이터베이스의 클러스터링은 하나 이상의 열 값을 기반으로 물리적으로 테이블 데이터를 구성하는 것을 의미하고, 클러스터링된 테이블은 클러스터링된 열에 유사한 값을 가진 행을 동일하거나 가까운 데이터 블록에 함께 저장하게된다.
> 이렇게 구성된 배열은 관련 데이터를 검색할 때 분산된 I/O 작업의 필요성을 줄여 성능을 향상시킨다. 테이블이 클러스터링되면 클러스터링 열에서 공통 값을 공유하는 행이 함께 저장되므로 해당 열을 기반으로 쿼리를 날릴 때 효율적인 액세스가 가능하다.


## 4. Sort나 Hash 작업을 최소화 하라

`Order by`나 `Group By` 때문에 성능이 저하될 수 있는데 특히 결과가 많을 경우 `Sort`는 치명적이다.
인덱스가 `Sort` 되어있다는 특성을 이용하면 `order by` 작업을 대신할 수 있다. `Group By`도 `sort`가 발생되는데 `group by` 단위와 인덱스의 컬럼이 
동일 하다면 `sort`는 발생하지 않는다. 최적의 인덱스를 사용하면 `Access Path`를 개선하는 효과뿐만 아니라 `Sort`의 부하도 없어지는것이다.

`Union All`을 제외한 집합 연산(Union, Minus, Intersect)를 사용하면 `sort unique` 혹은 `hash unique`가 발생되는데, `Union`은 `Union All`로 변경할 수 없는지 고려해야 하고, `Minus`는 `Not Exists` 서브 쿼리를 이용하여 `Anti Join`으로 바꿀 수 없는지 고려해야 한다. `Intersect`는 교집합이므로 
조인으로 바꿀 수 있는지 검토해야 한다. 간혹 `Distinct`를 사용한 SQL을 볼 수 있는데 이 역시 `sort unique` 혹은 `hash unique`를 발생시키기 때문에 
모델러 혹은 설계자와 상의하여 `Distinct`를 제거할 방법을 찾아야한다.

Oracle 10g부터는 `Hash Group By`가 발생할 수 있는데, 이미 적절한 인덱스를 사용한 경우라면 `Hash Group By`를 사용할 이유가 없다.
이런 경우 `NO_USE_HASH_AGGREGATION` 힌트를 사용하면 `Sort Group By`로 변경할 수 있고 실행 계획에 `“SORT GROUP BY NOSORT” Operation`이 
발생하며, `Sort`나 `Hashing` 작업이 전혀 발생하지 않는다. 

`Group By`의 부하를 해결하는 또 다른 방법은 스칼라 서브 쿼리를 사용하는것으로 `Group By`를 사용하지 않고도 `sum 이나 min/max 값`을 구할 수 있다.
또한 분석함수의 `Ranking Family(rank, dens_rank, row_number)`를 최적화된 인덱스와 같이 사용하면 `Group By나 Sort`를 쓰지 않고도 `min/max 값`을 구할 수 있다. 조인을 사용하면 `Group By`가 필수적인것과 대비적이다. 이 때는 실행 계획에 `“WINDOW NOSORT” Operation`이 발생 된다.[관련 글](https://scidb.tistory.com/entry/%EB%B6%84%EC%84%9D%ED%95%A8%EC%88%98%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-TOP-SQL%EC%9D%80-%ED%8A%9C%EB%8B%9D%EC%9D%B4-%EB%B6%88%EA%B0%80%ED%95%9C%EA%B0%80)

> [!note] Sort Unique 와 Hash Unique
> Sort Unique 와 Hash Unique 중 어떤 전략이 사용되는지는 데이터베이스 시스템, 사용 가능한 시스템 리소스, 데이터 배포 및 쿼리 최적화 프로그램 결정과 같은 다양한 요소에 따라 달라지고 서로 다른 메커니즘을 사용하지만, 두 방법 모두 집합 작업으로인해 발생하는 중복 행을 효율적으로 제거하는 것을 목표로 한다.  데이터베이스의 쿼리 최적화 프로그램은 쿼리의 특성과 최적의 성능을 위해 사용 가능한 리소스를 기반으로 어떤 방법을 사용할지 결정한다.
> 


## 5. 한 블록은 한 번만 Scan 하라

당연하게도 같은 데이터를 반복적으로 Scan하는 SQL은 성능에 영향을 준다. 대표적인 경우로 Union All로 분리되어있지만 실제로는 그럴 필요가 없는 경우이다. 예를 들어 Where 절에 구분 코드가 1일 때, 2일 때, 3일 때 별로 SQL이 나누어져 있다면 Where 절을 구분 코드 in (1,2,3)으로 처리하고, Select 절에서 Decode나 Case 문을 사용하여 구분 코드 별로 처리해준다면 Union All은 사용하지 않아도 된다.

```SQL
SELECT 'Type 1 Processing: ' || value AS processed_value 
FROM your_table 
WHERE separator_code = 1 

UNION ALL 

SELECT 'Type 2 Processing: ' || value AS processed_value 
FROM your_table 
WHERE separator_code = 2 

UNION ALL 

SELECT 'Type 3 Processing: ' || value AS processed_value 
FROM your_table 
WHERE separator_code = 3;
```

```sql
SELECT 
	CASE 
		WHEN separator_code = 1 THEN 'Type 1 Processing: ' || value 
		WHEN separator_code = 2 THEN 'Type 2 Processing: ' || value 
		WHEN separator_code = 3 THEN 'Type 3 Processing: ' || value 
		ELSE 'Other Processing: ' || value 
	END AS processed_value 
FROM 
	your_table 
WHERE 
	separator_code IN (1, 2, 3);
```

UnionAll을 사용하는 또 한가지의 경우는 Sub Total(소계)과 Grand Total(총계)를 구해야 하는 경우이다. 이 경우도 Rollup/Cube나 Grouping sets를 Group by 절에 사용한다면 소계나 총계를 위한 별도의 Select문을 실행 시킬 필요가 없다. 

```sql
SELECT 
	region, 
	product, 
	SUM(sales_amount) AS total_sales 
FROM sales 
GROUP BY region, product 
	
UNION ALL  

SELECT 
	region, 
	'Total for ' || region AS product, 
	SUM(sales_amount) AS total_sales 
	
FROM sales 

GROUP BY region 

UNION ALL 

SELECT 
	'Grand Total' AS region, 
	'All Products' AS product, 
	SUM(sales_amount) AS total_sales 
FROM sales;
```

```sql
SELECT 
	CASE 
		WHEN GROUPING(region) = 1 THEN 'Grand Total' 
		ELSE COALESCE(region, 'Unknown') 
	END AS region, 
	CASE 
		WHEN GROUPING(product) = 1 THEN 'Total for ' || COALESCE(region, 'All Regions') 
		ELSE COALESCE(product, 'All Products')   
	END AS product, 
	SUM(sales_amount) AS total_sales 
FROM sales 
GROUP BY ROLLUP(region, product);
```

1 - 4번의 과정은 SQL문의 변경이 없거나 최소화 되지만 5번의 경우 SQL을 통합시켜야 하기 때문에 시간이 많이 소모되며, 많은 사고가 요구된다.
이것으로 원본 SQL 자체의 튜닝은 완료되었다고 볼 수 있다.

## 6. 페이징 처리

온라인 SQL에서 주요한 포인트는 부분 범위 처리이다. 물론 전체 건을 처리해야 하는 경우도 있겠지만, 조회 화면이라면 몇 십 혹은 몇 만 건의 결과를 모두 볼 수 없다. 따라서 볼 수 있는 단위로 끊어서 출력해야 한다. 조회 결과 건수가 10만 건이라도 해도 최초의 50건을 화면에 먼저 뿌린다면 1 - 4번에서 언급했던
모든 부하(Block I/O, 조인, Random Access, sort 부하)를 한 번에 감소 시킬 수 있다.

페이징 처리를 해도 효과를 볼 수 없는 몇가지 예외 상황이 있다. 분석 함수를 사용하거나 `Connect By + Start With`를 사용한다면 페이징 처리의 효과는 없다. 분석 함수의 경우 인라인 뷰의 외부로 뺄 수 있다면 부분 범위 처리가 가능하다.[관련 글](https://scidb.tistory.com/entry/Pagination%EA%B3%BC-%EB%B6%84%EC%84%9D%ED%95%A8%EC%88%98%EC%9D%98-%EC%9C%84%ED%97%98%ED%95%9C-%EC%A1%B0%ED%95%A9) 
`Connect By + Start With`를 사용한 경우는 부분 범위 처리가 불가능 하지만 `Recursive With` 절을 사용한다면 페이징 처리의 효과를 볼 수 있다. 이때, `Recursive With`절에 Search절을 사용한다면 Connect By와 마찬가지로 페이징 처리의 효과가 없으니 주의가 필요하다.

## 7. 답이 틀려선 안된다 SQL을 검증하라

튜닝을 하기 전과 한 후 혹은 튜닝을 하였음에도 답이 틀리다면, 튜닝을 하지 않은 것 보다 못하다. 그러므로 튜닝 후에 답이 올바른지 항상 검증해야한다.

----

**정리**

1. 적절한 인덱스를 사용하여 Block I/O를 최소화 하라.
2. 조인방법과 조인순서를 최적화 하라.
3. Table Access(Random Access)를 최소화 하라
4. Sort나 Hash 작업을 최소화 하라
5. 한 블록은 한번만 Scan하고 끝내라
6. 온라인의 조회화면이라면 페이징처리는 필수이다
7. 답이 틀리면 안 된다. SQL을 검증하라


1번부터 7번을 모두 적용할 수 있는 경우임에도 불구하고 하나라도 빠진다면 그것은 최적화된 SQL이라고 볼 수 없다.
튜닝을 진행할 때 항상 모든 사항을 적용할 수 있는 것은 아니지만 모든 과정을 적용할 수 있는지 꼼꼼하게 살펴보는것이 중요하다.


**참고 자료**
[Science of Database](https://scidb.tistory.com/entry/%EB%B6%84%EC%84%9D%ED%95%A8%EC%88%98%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-TOP-SQL%EC%9D%80-%ED%8A%9C%EB%8B%9D%EC%9D%B4-%EB%B6%88%EA%B0%80%ED%95%9C%EA%B0%80)
[gonnnnn.log](https://velog.io/@gonnnnn/JOIN%EC%9D%98-%EC%A2%85%EB%A5%98%EC%99%80-%EA%B8%B0%EB%B3%B8-%EC%9E%91%EB%8F%99-%EB%B0%A9%EC%8B%9DNested-Loop#nested-loops)
