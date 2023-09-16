# Longest Increasing Subsequence

[나무위키](https://namu.wiki/w/%EC%B5%9C%EC%9E%A5%20%EC%A6%9D%EA%B0%80%20%EB%B6%80%EB%B6%84%20%EC%88%98%EC%97%B4)


최장 증가 부분 수열 문제는 동적 계획법으로 풀 수 있는 유명한 알고리즘 문제이다.

어떤 임의의 수열이 주어질 때, 이 수열에서 몇 개의 수들을 제거해서 부분수열을 만들 수 있다. 이때 만들어진 부분수열 중 오름차순으로 정렬된 가장 긴 수열을 최장 증가 부분 수열이라 한다.

3 5 7 9 2 1 4 8

위 수열에서 몇 개의 수를 제거해 부분수열을 만들 수 있다.

3 7 9 1 4 8 (5, 2 제거)
7 9 1 8 (3, 5, 2, 4 제거)
3 5 7 8 (9, 2, 1, 4 제거)
1 4 8 (3, 5, 7, 9, 2 제거)

이들 중 첫번째, 두번째 수열은 일부가 오름차순으로 정렬되어 있다.
반면에 세번째, 네번째 수열은 전체가 오름차순으로 정렬되어 있다.
위와 같이 일부, 혹은 전체가 오름차순인 수열을 '증가 부분 수열'이라고 한다.

그리고 이런 증가 부분 수열 중 가장 긴 수열을 '최장 증가 부분 수열 (LIS)'이라 한다.
즉, 위의 수열의 집합에선 부분수열 3 5 7 8이 LIS이다.


한 수열에서 여러 개의 LIS가 나올 수도 있다. 예를 들어 수열

5 1 6 2 7 3 8 에서 부분수열

1 2 3 8
5 6 7 8 은 모두 길이가 4인 LIS이다.

---

## 예제

N개의 자연수로 이루어진 수열이 주어졌을 때, 그 중에서 가장 길게 증가하는(작은 수에서 큰 수로) 원소들의 집합을 찾는 프로그램을 작성하라. 예를 들어, 원소가 2, 7, 5, 8, 6, 4, 7, 12, 3 이면 가장 길게 증가하도록 원소들을 차례대로 뽑아내면 2, 5, 6, 7, 12를 뽑아내어 길 이가 5인 최대 부분 증가수열을 만들 수 있다.

▣ 입력설명
첫째 줄은 입력되는 데이터의 수 N(3≤N≤1,000, 자연수)를 의미하고, 둘째 줄은 N개의 입력데이터들이 주어진다.

▣ 출력설명
첫 번째 줄에 부분증가수열의 최대 길이를 출력한다.

▣ 입력예제 1
8 
53786294

▣ 출력예제 1 
4

``` java

전체 풀이 

import java.util.*;
class Main{
	static int[] dy;
	public int solution(int[] arr){
		int answer=0;
		dy=new int[arr.length];
		dy[0]=1;
		for(int i=1; i<arr.length; i++){
			int max=0;
			for(int j=i-1; j>=0; j--){
				if(arr[j]<arr[i] && dy[j]>max) max=dy[j];
			}
			dy[i]=max+1;
			answer=Math.max(answer, dy[i]);
		}
		return answer;
	}

	public static void main(String[] args){
		Main T = new Main();
		Scanner kb = new Scanner(System.in);
		int n=kb.nextInt();
		int[] arr=new int[n];
		for(int i=0; i<n; i++){
			arr[i]=kb.nextInt();
		}
		System.out.print(T.solution(arr));
	}
}

```

전체 흐름은 위와 같다. solution 메서드에 풀이 로직이 담겨 있으니 살펴보자
```java
static int[] dy;
	public int solution(int[] arr){
		int answer=0;
		dy=new int[arr.length];
		dy[0]=1;
		for(int i=1; i<arr.length; i++){
			int max=0;
			for(int j=i-1; j>=0; j--){
				if(arr[j]<arr[i] && dy[j]>max) max=dy[j];
			}
			dy[i]=max+1;
			answer=Math.max(answer, dy[i]);
		}
		return answer;
	}
```

1.
먼저 입력으로 받은 리스트의 LIS를 구하는 것이 목표이기 때문에 dy 역시 arr와 같은 길이로 선언한다.

2. dy[0]의 길이는 먼저 등장하는 수가 없기 때문에 확실하게 1로 크기를 잡을 수 있다.
3. 내부 반복문은 현재 비교 대상 i와 이전에 등장한 수를 j로 하여 검증하는데 이 때 dy[j]와 max 값을 검사하는 로직을 추가하는 것으로 하나의 정의를 내릴 수 있다.
`arr[i]의 값이 이전 값들 보다 크면서 동시에 부분 수열의 크기가 가장 큰 값을 찾는다.`
4. 이렇게 가장 큰 수열의 값을 찾았다면 자기 자신까지 포함해 `dy[i]=max+1`의 값이 된다.
5. 마지막으로 answer 값과 부분 증가 수열을 비교해 가장 큰 수열의 크기를 찾을 수 있다.

---

## 예제

``` java
import java.util.*;
class Brick implements Comparable<Brick>{
    public int s, h, w;
    Brick(int s, int h, int w) {
		this.s = s;
        this.h = h;
        this.w = w;
    }
    @Override
    public int compareTo(Brick o){
        return o.s-this.s;
    }
}
class Main{
	static int[] dy;
	public int solution(ArrayList<Brick> arr){
		int answer=0;
		Collections.sort(arr);
		dy[0]=arr.get(0).h;
		answer=dy[0];
		for(int i=1; i<arr.size(); i++){
			int max_h=0;
			for(int j=i-1; j>=0; j--){
				if(arr.get(j).w > arr.get(i).w && dy[j]>max_h){
					max_h=dy[j];
				}
			}
			dy[i]=max_h+arr.get(i).h;
			answer=Math.max(answer, dy[i]);
		}
		return answer;
	}

	public static void main(String[] args){
		Main T = new Main();
		Scanner kb = new Scanner(System.in);
		int n=kb.nextInt();
		ArrayList<Brick> arr=new ArrayList<>();
		dy=new int[n];
		for(int i=0; i<n; i++){
			int a=kb.nextInt();
			int b=kb.nextInt();
			int c=kb.nextInt();
			arr.add(new Brick(a, b, c));
		}
		System.out.print(T.solution(arr));
	}
}
```