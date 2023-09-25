![[Screenshot 2023-09-25 at 3.30.50 PM.png]]


# 개요

빌더는 복잡한 객체를 단계별로 생성할 수 있도록 하는 생성 디자인 패턴으로 이 패턴을 사용하면 같은 제작 코드를 사용하여 객체의 다양한 유형과 표현을 제작할 수 있다.

### 문제점 

먼저 빌더 패턴 이전에 사용된 생성자 패턴을 살펴보자

#### 점층적 생성자 패턴

점층적 생성자 패턴(Telescoping Constructor Pattern)은 필수 매개변수와 함께 선택 매개변수를 0개, 1개, 2개 ... 받는 형태로, 우리가 다양한 매개변수를 입력받아 인스턴스를 생성하고 싶을 때 사용하던 생정자를 오버로딩 하는 방식이다.

``` java
class Hamburger {
    // 필수 매개변수
    private int bun;
    private int patty;

    // 선택 매개변수
    private int cheese;
    private int lettuce;
    private int tomato;
    private int bacon;

    public Hamburger(int bun, int patty, int cheese, int lettuce, int tomato, int bacon) {
        this.bun = bun;
        this.patty = patty;
        this.cheese = cheese;
        this.lettuce = lettuce;
        this.tomato = tomato;
        this.bacon = bacon;
    }

    public Hamburger(int bun, int patty, int cheese, int lettuce, int tomato) {
        this.bun = bun;
        this.patty = patty;
        this.cheese = cheese;
        this.lettuce = lettuce;
        this.tomato = tomato;
    }
    

    public Hamburger(int bun, int patty, int cheese, int lettuce) {
        this.bun = bun;
        this.patty = patty;
        this.cheese = cheese;
        this.lettuce = lettuce;
    }

    public Hamburger(int bun, int patty, int cheese) {
        this.bun = bun;
        this.patty = patty;
        this.cheese = cheese;
    }

    ...
}

public static void main(String[] args) {
    // 모든 재료가 있는 햄버거
    Hamburger hamburger1 = new Hamburger(2, 1, 2, 4, 6, 8);

    // 빵과 패티 치즈만 있는 햄버거
    Hamburger hamburger2 = new Hamburger(2, 1, 1);

    // 빵과 패티 베이컨만 있는 햄버거
    Hamburger hamburger3 = new Hamburger(2, 0, 0, 0, 0, 6);
}
```

이런 방식은 클래스 인스턴스 필드들이 많으면 많을수록 생성자에 들어갈 인자의 수가 늘어나 몇번 째 인자가 어떤 필드였는지 햇갈릴 경우가 생기게 된다.
만일 여러 종류의 햄버거를 생성하기 위해 Hamburger 생성자의 몇번째 인수가 양상추인지, 토마토인지 파악할 필요가 있다.

또한 매개변수 특성상 순서를 따라야 하기 때문에 불필요한 파라미터에도 '0'을 전달해야 한다. 생성자만으로는 필드를 선택적으로 생략할 수 있는 방법이 없기 때문이다.

그리고 무엇보다 타입이 다양할 수록 생성자 메서드 수가 늘어나 가독성이나 유지보수 측면에서 좋지 않다.

#### 자바 빈(Java Beans) 패턴

점층적 생성자 패턴의 단점을 보안하기 위해 Setter 메서드를 사용한 자바 빈(Bean) 패턴이 고안 되었다.
매개변수가 없는 생성자로 객체 생성 후 Setter 메서드를 이용해 클래스 필드의 초깃값을 설정하는 방식이다.

```java
class Hamburger {
    // 필수 매개변수
    private int bun;
    private int patty;

    // 선택 매개변수
    private int cheese;
    private int lettuce;
    private int tomato;
    private int bacon;
    
    public Hamburger() {}

    public void setBun(int bun) {
        this.bun = bun;
    }

    public void setPatty(int patty) {
        this.patty = patty;
    }

    public void setCheese(int cheese) {
        this.cheese = cheese;
    }

    public void setLettuce(int lettuce) {
        this.lettuce = lettuce;
    }

    public void setTomato(int tomato) {
        this.tomato = tomato;
    }

    public void setBacon(int bacon) {
        this.bacon = bacon;
    }
}

public static void main(String[] args) {
    // 모든 재료가 있는 햄버거
    Hamburger hamburger1 = new Hamburger();
    hamburger1.setBun(2);
    hamburger1.setPatty(1);
    hamburger1.setCheese(2);
    hamburger1.setLettuce(4);
    hamburger1.setTomato(6);
    hamburger1.setBacon(8);

    // 빵과 패티 치즈만 있는 햄버거
    Hamburger hamburger2 = new Hamburger();
    hamburger2.setBun(2);
    hamburger2.setPatty(1);
    hamburger2.setCheese(2);

    // 빵과 패티 베이컨만 있는 햄버거
    Hamburger hamburger3 = new Hamburger();
    hamburger3.setBun(2);
    hamburger2.setPatty(1);
    hamburger3.setBacon(8);
}
```

기존 생성자 오버로딩에서 나타났던 가독성 문제점이 사라지고 선택적인 파라미터에 대해 해당되는 Setter 메서드를 호출함으로써 유연하게 객체 생성이 가능해졌다.
하지만 이런 방식은 객체 생성 시점에 모든 값들을 주입 하지 않아 일관성(Consistency) 문제와 불변성(Immutable) 문제가 발생하게 된다.

***일관성 문제***
필수 매개변수란 객체가 초기화될때 반드시 설정되어야 하는 값이다. 하지만 개발자의 실수로 `set*()`메서드를 호출하지 않았다면 이 객체는 일관성이 무너진 상태가 된다.
즉, 객체가 유효하지 않은 것이다. 만일 다른 곳에서 이 인스턴스를 사용하게 된다면 런타임 예외가 발생할 수도 있다.

이는 객체를 생성하는 부분과 값을 설정하는 부분이 물리적으로 떨어져 있어서 발생하는 문제점이다. 물론 이는 어느정도 생성자(Constructor)와 결합하여 극복할 수 있지만
불변성 문제 때문에 자바 빈즈 패턴은 지양된다.

***불변성 문제***
자바 빈즈 패턴의 Setter 메서드는 객체를 처음 생성할때 필드값을 설정하기 위해 존재하는 메서드이다.
하지만 객체를 생성했음에도 여전히 외부적으로 Setter 메서드를 노출하고 있기 때문에 협업 과정에서 언제 어디서 누군가 Setter 메서드를 호출해 객체를 조작할 수 있게 된다.
이런 상황을 불면을 보장할 수 없다고 이야기 한다.

### 해결책

빌더 패턴은 자신의 클래스에서 객체 생성 코드를 추출해 `builders`라는 별도의 객체들로 이동하도록 제안한다.
이를 통해 복잡한 객체들을 단계별로 생성할 수 있도록 하고, 빌더는 제품이 생성되는 동안 다른 객체들이 제품에 접근하는 것을 허용하지 않는다.

이 패턴은 객체 생성을 일련의 단계로 정리하며, 객체를 생성하고 싶으면 builder 객체를 실행하면 된다. 중요한 점은 모든 단계를 호출할 필요가 없다는 것으로, 객체의 특정 설정을 제작하는 데 필요한 단계들만 호출하면 된다.

이런 경우 같은 단계들의 집합을 다른 방식으로 구현하는 여러 다른 빌더 클래스를 생성할 수 있으며, 그 다음 프로세스(생성 단계에 대한 순서화된 호출의 집합)내에서


# 적용 시기

### 패턴 장점


### 패턴 단점


# 프로토타입 패턴 구조

![](https://i.imgur.com/6QIgXi1.png)


### (1) 


### (2) 

### (3) 



![](https://i.imgur.com/2RbxYlp.png)

### (1) 


# 구현 방법


# Pattern 구현


![](https://i.imgur.com/L6QVDA7.png)


#### Shape.java

``` java

```

#### Circle.java

``` java

```

#### Rectangle.java

``` java

```

#### Demo.java

``` java

```

#### 실행 결과



#### BundledShapeCache.java

```java

```

#### Demo.java

``` java
```


### 실행 결과




# 디자인 패턴 비교

> 
>
  

> 

> 
> 
>

>


# 정리

