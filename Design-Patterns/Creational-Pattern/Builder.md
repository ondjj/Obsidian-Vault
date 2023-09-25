---
sticker: lucide//hammer
---
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

이런 경우 같은 단계들의 집합을 다른 방식으로 구현하는 여러 다른 빌더 클래스를 생성할 수 있으며, 그 다음 프로세스(생성 단계에 대한 순서화된 호출의 집합)내에서 이러한 빌더들을 사용하여 다양한 종류의 객체를 생성할 수 있다.

``` java
public static void main(String[] args) {

    // 생성자 방식
    Hamburger hamburger = new Hamburger(2, 3, 0, 3, 0, 0);

    // 빌더 방식
    Hamburger hamburger = new Hamburger.Builder(10)
        .bun(2)
        .patty(3)
        .lettuce(3)
        .build();
}
```

빌더 패턴의 사용법을 살펴보면 빌더 클래스의 메서드를 체이닝(Chaining) 형태로 호출함으로써 자연스럽게 인스턴스를 구성하고 마지막에 builder() 메서드를 통해 최종적으로 객체를 생성하도록 되어있음을 확인할 수 있다.

빌더 패턴을 이용하면 더이상 생성자 오버로딩 열거를 하지 않아도 되며, 데이터의 순서에 상관없이 객체를 만들어내 생성자 인자 순서를 파악할 필요도 없고, 잘못된 값을 넣는 실수도 하지 않게 된다. 점층적 생성자 패턴과 자바빈즈 패턴 두 가지의 장점이 조합되었다고 볼 수 있다.

### 디렉터 (관리자)

더 나아가 제품을 생성하는 데 사용하는 빌더 단계들에 대한 일련의 호출을 디렉터라는 별도의 클래스로 추출할 수 있다. 디렉터 클래스는 제작 단계들을 실행하는 순서를 정의하는 반면 빌더는 이러한 단계들에 대한 구현을 제공한다.

프로그램에 디렉터 클래스를 포함하는 것은 필수사항은 아니고, 언제든지 클라이언트 코드에서 생성 단계들을 직접 특정 순서로 호출할 수 있다.
그러나 디렉터 클래스는 다양한 생성 루틴들을 배치하여 프로그램 전체에서 재사용 할 수 있는 좋은 장소가 될 수 있고, 디렉터 클래스는 클라이언트 코드에서 제품 생성의 세부 정보를 완전히 숨긴다. 클라이언트는 빌더를 디렉터와 연관시키고 디렉터와 생성을 시행한 후 빌더로부터 결과를 얻기만 하면 된다.

# 적용 시기

- '점층적 생성자'를 제거하기 위해 빌터 패턴을 사용할 수 있다.

- 빌더 패턴은 일부 객체를 다르게 생성할 수 있도록 하고 싶을 때 사용할 수 있다.
  객체의 다양한 표현의 생성 과정이 세부 사항만 다른 유사한 단계를 포함할 때 적용할 수 있는데
  기초 빌더 인터페이스는 가능한 모든 생성 단계들을 정의하고 구현 빌더는 이러한 단계들을 구현하여 객체를 여러 형태로 생성한다.

- 빌더를 사용하여 복합체 트리 또는 기타 복잡한 객체를 생성할 때
  객체를 단계별로 생성할 수 있으며, 최종 객체를 손상하지 않고 일부 단계들의 실행을 연기할 수 있다. 그리고 재귀적으로 단계를 호출할 수 있는데, 이는 객체 트리를 구축해야 할 때 매우 유용하다.
  
빌더는 생성 단계들을 수행하는 동안 미완성 제품을 노출하지 않으며, 이는 클라이언트 코드가 불완전한 결과를 가져오는 것을 방지한다.

### 패턴 장점

1. 객체를 단계별로 생성하거나 생성 단계들을 연기하거나, 재귀적으로 단계들을 실행할 수 있다.
2. 객체를 다양한 방식으로 만들 때 같은 생성 코드를 재사용할 수 있다.
3. 단일 책임 원칙. 비즈니스 로직에서 복잡한 생성 코드를 고립 시킬 수 있다.

### 패턴 단점

- 패턴이 여러 개의 새 클래스를 생성해야하므로 코드의 전반적인 복잡성이 증가한다.
- 생성자 보다 성능이 떨어진다.
  매번 메서드를 호출하여 빌더를 거쳐 인스턴스화 하기 때문에 당연한 결과이다. 비록 생성 비용 자체는 크지 않지만, 애플리케이션의 성능이 중요한 상황이라면 문제가 될 수 있다.

클래스의 필드 개수가 4개 보다 적고, 필드의 변경 가능성이 없는 경우라면 차라리 생성자나 정적 팩토리 메소드를 이용하는 것이 더 좋을 수 있다.
빌더 패턴의 코드가 다소 장황하기 때문이다. 따라서 클래스 필드의 갯수와 필드 변경 가능성을 중점으로 보고 패턴의 적용 유무를 가려야 한다.
(다만 API 는 시간이 지날수록 많은 매개변수를 갖는 경향이 있기 때문에 애초에 빌더 패턴으로 시작하는 편이 나을 때가 많다고 말하는 경향도 있다.)


# 빌더 패턴 구조

![[Builder.png]]

### (1) 빌더

빌더 인터페이스는 모든 유형의 빌더에게 공통적인 제품 생성 단계를 선언한다.
### (2) 구현 빌더

구현 빌더는 생성 단계의 다양한 구현을 제공한다. 또 구현 빌더는 공통 인터페이스를 따르지 않는 제품도 생성할 수 있다.

### (3) 제품

제품은 그 생성 결과로 나온 객체로 다른 빌더들에 의해 생성된 제품은 같은 클래스 계층구조 또는 인터페이스에 속할 필요가 없다.


### (4) 디렉터

디렉터 클래스는 생성 단계를 호출하는 순서를 정의하므로 제품의 특정 설정을 만들고 재사용할 수 있다.

### (5) 클라이언트

클라이언트는 빌더 객체 중 하나를 디렉터와 연결해야 한다. 일반적으로 위 연결은 디렉터 생성자의 매개변수를 통해 한 번만 수행되며, 그 후 디렉터는 모든 추가 생성에 해당 빌더 객체를 사용한다. 그러나 클라이언트가 빌더 객체를 디렉터의 프로덕션 메서드에 전달할 때를 위한 대안적 접근 방식이 존재하는데 이 경우 디렉터와 함께 무언가를 만들 때마다 다른 빌더를 사용할 수 있다.

# 구현 방법

1. 실행 가능한 모든 객체 표현을 생성하는 데 필요한 표준 단계를 명확하게 설명할 수 있는지 확인한다.(그렇지 않을 경우 패턴 구현을 진행할 수 없음)
   
2. 기초 빌더 인터페이스에 이 단계를 선언 한다.
   
3. 각 제품 표현에 대한 구현 빌더 클래스를 만들고 해당 생성 단계를 구현한다.
   
   생성 결과를 가져오는 메서드를 구현하는 것은 중요하다. 빌더 인터페이스 내에서 이 메서드를 선언할 수 없는 이유는 다양한 빌더들이 공통 인터페이스가 없는 제품들을 생성할 수 있기 때문이다. 따라서 이런 메서드의 반환 유형이 무엇인지 알 수 없다. 그러나 단일 계층 구조의 제품들을 처리하는 경우 생성 결과를 가져오는 메서드를 기초 인터페이스에 안전하게 추가할 수 있다.
   
4. 디렉터 클래스를 만드는 것에 대해 고려(같은 빌더 객체를 사용하여 제품을 제작하는 다양한 방법을 캡슐화 할 수 있다.)
   
5. 클라이언트 코드는 빌더 객체들과 디렉터 객체를 모두 생성해야한다. 제작이 시작되기 전에 클라이언트는 빌더 객체를 디렉터에게 전달해야 하고, 일반적으로 클라이언트는 디렉터의 클래스 생성자의 매개변수들을 통해 위 작업을 한 번만 수행한다. 그 후 디렉터는 모든 추가 제작에서 빌더 객체를 사용한다.
   또, 대안적인 접근 방식이 있는데 이 방식은 빌더가 디렉터의 특정 제품 제작 메서드에 전달되는 것이다.
   
6. 모든 제품이 같은 인터페이스를 따르는 경우에만 디렉터로부터 직접 생성 결과를 얻을 수 있다. (그렇지 않으면 클라이언트는 빌더에서 결과를 가져와야 한다.)
# Pattern 구현


빌더 패턴은 자바 개발자에게 잘 알려진 패턴으로 가능한 설정 옵션이 많은 객체를 만들어야 할 때 특히 유용하다.

- 빌더는 자바 코어 라이브러리에서 널리 사용된다.

- [`java.lang.StringBuilder#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html#append-boolean-) (`unsynchronized`)
- [`java.lang.StringBuffer#append()`](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-) (`synchronized`)
- [`java.nio.ByteBuffer#put()`](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-) (또 다음에도 사용됩니다: [`Char­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html#put-char-), [`Short­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/ShortBuffer.html#put-short-), [`Int­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/IntBuffer.html#put-int-), [`Long­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/LongBuffer.html#put-long-), [`Float­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/FloatBuffer.html#put-float-) and [`Double­Buffer`](http://docs.oracle.com/javase/8/docs/api/java/nio/DoubleBuffer.html#put-double-))
- [`javax.swing.GroupLayout.Group#addComponent()`](http://docs.oracle.com/javase/8/docs/api/javax/swing/GroupLayout.Group.html#addComponent-java.awt.Component-)
- [`java.lang.Appendable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)의 모든 구현

**식별법**
빌더 패턴은 하나의 생성 메서드와 결과 객체를 설정하기 위한 여러 메서드가 있는 클래스가 있고, 빌더 메서드는 체이닝 연결을 지원한다.

### UML

![[Screenshot 2023-09-25 at 8.46.59 PM.png]]

![[Screenshot 2023-09-25 at 8.45.55 PM.png]]
## Package : builders
#### Builder.java

``` java
package com.design.pattern.creational.builder.example.builders;  
  
import com.design.pattern.creational.builder.example.cars.CarType;  
import com.design.pattern.creational.builder.example.components.Engine;  
import com.design.pattern.creational.builder.example.components.GPSNavigator;  
import com.design.pattern.creational.builder.example.components.Transmission;  
import com.design.pattern.creational.builder.example.components.TripComputer;  
  
/**  
 * Builder 인터페이스는 제품을 구성하는 모든 가능한 방법을 정의합니다.  
 */
 public interface Builder {  
    void setCarType(CarType type);  
    void setSeats(int seats);  
    void setEngine(Engine engine);  
    void setTransmission(Transmission transmission);  
    void setTripComputer(TripComputer tripComputer);  
    void setGPSNavigator(GPSNavigator gpsNavigator);  
}
```

#### CarBuilder.java

``` java
package com.design.pattern.creational.builder.example.builders;  
  
import com.design.pattern.creational.builder.example.cars.Car;  
import com.design.pattern.creational.builder.example.cars.CarType;  
import com.design.pattern.creational.builder.example.components.Engine;  
import com.design.pattern.creational.builder.example.components.GPSNavigator;  
import com.design.pattern.creational.builder.example.components.Transmission;  
import com.design.pattern.creational.builder.example.components.TripComputer;  
  
/**  
 * 구현 빌더는 공통 인터페이스에 정의된 단계를 구현합니다.  
 */  
public class CarBuilder implements Builder{  
    private CarType type;  
    private int seats;  
    private Engine engine;  
    private Transmission transmission;  
    private TripComputer tripComputer;  
    private GPSNavigator gpsNavigator;  
  
    @Override  
    public void setCarType(final CarType type) {  
        this.type = type;  
    }  
  
    @Override  
    public void setSeats(final int seats) {  
        this.seats = seats;  
    }  
  
    @Override  
    public void setEngine(final Engine engine) {  
        this.engine = engine;  
    }  
  
    @Override  
    public void setTransmission(final Transmission transmission) {  
        this.transmission = transmission;  
    }  
  
    @Override  
    public void setTripComputer(final TripComputer tripComputer) {  
        this.tripComputer = tripComputer;  
    }  
  
    @Override  
    public void setGPSNavigator(final GPSNavigator gpsNavigator) {  
        this.gpsNavigator = gpsNavigator;  
    }  
  
    public Car getResult() {  
        return new Car(type, seats, engine, transmission, tripComputer, gpsNavigator);  
    }  
}
```

#### CarManualBuilder.java

``` java
package com.design.pattern.creational.builder.example.builders;  
  
  
import com.design.pattern.creational.builder.example.cars.CarType;  
import com.design.pattern.creational.builder.example.cars.Manual;  
import com.design.pattern.creational.builder.example.components.Engine;  
import com.design.pattern.creational.builder.example.components.GPSNavigator;  
import com.design.pattern.creational.builder.example.components.Transmission;  
import com.design.pattern.creational.builder.example.components.TripComputer;  
  
/**  
 * 다른 생성 패턴과 달리 Builder는 관련 없는 제품을 제작할 수 있고, 공통적인 인터페이스가 없다.  
 */
 public class CarManualBuilder implements Builder{  
    private CarType type;  
    private int seats;  
    private Engine engine;  
    private Transmission transmission;  
    private TripComputer tripComputer;  
    private GPSNavigator gpsNavigator;  
  
    @Override  
    public void setCarType(final CarType type) {  
        this.type = type;  
    }  
  
    @Override  
    public void setSeats(final int seats) {  
        this.seats = seats;  
    }  
  
    @Override  
    public void setEngine(final Engine engine) {  
        this.engine = engine;  
    }  
  
    @Override  
    public void setTransmission(final Transmission transmission) {  
        this.transmission = transmission;  
    }  
  
    @Override  
    public void setTripComputer(final TripComputer tripComputer) {  
        this.tripComputer = tripComputer;  
    }  
  
    @Override  
    public void setGPSNavigator(final GPSNavigator gpsNavigator) {  
        this.gpsNavigator = gpsNavigator;  
    }  
  
    public Manual getResult() {  
        return new Manual(type, seats, engine, transmission, tripComputer, gpsNavigator);  
    }  
}
```

## Package : cars
#### Car.java

``` java
package com.design.pattern.creational.builder.example.cars;  
  
import com.design.pattern.creational.builder.example.components.Engine;  
import com.design.pattern.creational.builder.example.components.GPSNavigator;  
import com.design.pattern.creational.builder.example.components.Transmission;  
import com.design.pattern.creational.builder.example.components.TripComputer;  
  
/**  
 * Car is a product class. */
public class Car {  
    private final CarType carType;  
    private final int seats;  
    private final Engine engine;  
    private final Transmission transmission;  
    private final TripComputer tripComputer;  
    private GPSNavigator gpsNavigator;  
    private double fuel = 0;  
  
    public Car(CarType carType, int seats, Engine engine, Transmission transmission,  
               TripComputer tripComputer, GPSNavigator gpsNavigator) {  
        this.carType = carType;  
        this.seats = seats;  
        this.engine = engine;  
        this.transmission = transmission;  
        this.tripComputer = tripComputer;  
        if (this.tripComputer != null) {  
            this.tripComputer.setCar(this);  
        }  
        this.gpsNavigator = gpsNavigator;  
    }  
  
    public CarType getCarType() {  
        return carType;  
    }  
  
    public double getFuel() {  
        return fuel;  
    }  
  
    public void setFuel(double fuel) {  
        this.fuel = fuel;  
    }  
  
    public int getSeats() {  
        return seats;  
    }  
  
    public Engine getEngine() {  
        return engine;  
    }  
  
    public Transmission getTransmission() {  
        return transmission;  
    }  
  
    public TripComputer getTripComputer() {  
        return tripComputer;  
    }  
  
    public GPSNavigator getGpsNavigator() {  
        return gpsNavigator;  
    }  
}
```

#### CarType
```java
package com.design.pattern.creational.builder.example.cars;  
  
public enum CarType {  
    CITY_CAR, SPORTS_CAR, SUV  
}
```

#### Manual
```java
package com.design.pattern.creational.builder.example.cars;  
  
import com.design.pattern.creational.builder.example.components.Engine;  
import com.design.pattern.creational.builder.example.components.GPSNavigator;  
import com.design.pattern.creational.builder.example.components.Transmission;  
import com.design.pattern.creational.builder.example.components.TripComputer;  
  
/**  
 * 카 매뉴얼은 또 다른 제품입니다. 참고로 카와 조상이 같지 않습니다. 관련이 없습니다.  
 */
public class Manual {  
    private final CarType carType;  
    private final int seats;  
    private final Engine engine;  
    private final Transmission transmission;  
    private final TripComputer tripComputer;  
    private final GPSNavigator gpsNavigator;  
  
    public Manual(CarType carType, int seats, Engine engine, Transmission transmission,  
                  TripComputer tripComputer, GPSNavigator gpsNavigator) {  
        this.carType = carType;  
        this.seats = seats;  
        this.engine = engine;  
        this.transmission = transmission;  
        this.tripComputer = tripComputer;  
        this.gpsNavigator = gpsNavigator;  
    }  
  
    public String print() {  
        String info = "";  
        info += "Type of car: " + carType + "\n";  
        info += "Count of seats: " + seats + "\n";  
        info += "Engine: volume - " + engine.getVolume() + "; mileage - " + engine.getMileage() + "\n";  
        info += "Transmission: " + transmission + "\n";  
        if (this.tripComputer != null) {  
            info += "Trip Computer: Functional" + "\n";  
        } else {  
            info += "Trip Computer: N/A" + "\n";  
        }  
        if (this.gpsNavigator != null) {  
            info += "GPS Navigator: Functional" + "\n";  
        } else {  
            info += "GPS Navigator: N/A" + "\n";  
        }  
        return info;  
    }  
}
```

### Package : components
#### Engine
```java
package com.design.pattern.creational.builder.example.components;  
  
/**  
 * Just another feature of a car. */
public class Engine {  
    private final double volume;  
    private double mileage;  
    private boolean started;  
  
    public Engine(double volume, double mileage) {  
        this.volume = volume;  
        this.mileage = mileage;  
    }  
  
    public void on() {  
        started = true;  
    }  
  
    public void off() {  
        started = false;  
    }  
  
    public boolean isStarted() {  
        return started;  
    }  
  
    public void go(double mileage) {  
        if (started) {  
            this.mileage += mileage;  
        } else {  
            System.err.println("Cannot go(), you must start engine first!");  
        }  
    }  
  
    public double getVolume() {  
        return volume;  
    }  
  
    public double getMileage() {  
        return mileage;  
    }  
}
```

#### GPSNavigator
```java
package com.design.pattern.creational.builder.example.components;  
  
public class GPSNavigator {  
    private String route;  
  
    public GPSNavigator() {  
        this.route = "221b, Baker Street, London  to Scotland Yard, 8-10 Broadway, London";  
    }  
  
    public GPSNavigator(String manualRoute) {  
        this.route = manualRoute;  
    }  
  
    public String getRoute() {  
        return route;  
    }  
}
```

#### Transmission
```java
package com.design.pattern.creational.builder.example.components;  
  
/**  
 * Just another feature of a car. */
public enum Transmission {  
    SINGLE_SPEED, MANUAL, AUTOMATIC, SEMI_AUTOMATIC  
}
```

#### TripComputer
```java
package com.design.pattern.creational.builder.example.components;  
  
import com.design.pattern.creational.builder.example.cars.Car;  
  
/**  
 * Just another feature of a car. */public class TripComputer {  
    private Car car;  
  
    public void setCar(Car car) {  
        this.car = car;  
    }  
  
    public void showFuelLevel() {  
        System.out.println("Fuel level: " + car.getFuel());  
    }  
  
    public void showStatus() {  
        if (this.car.getEngine().isStarted()) {  
            System.out.println("Car is started");  
        } else {  
            System.out.println("Car isn't started");  
        }  
    }  
}
```

### Package : director

#### Director
```java
package com.design.pattern.creational.builder.example.director;  
  
import com.design.pattern.creational.builder.example.builders.Builder;  
import com.design.pattern.creational.builder.example.cars.CarType;  
import com.design.pattern.creational.builder.example.components.Engine;  
import com.design.pattern.creational.builder.example.components.GPSNavigator;  
import com.design.pattern.creational.builder.example.components.Transmission;  
import com.design.pattern.creational.builder.example.components.TripComputer;  
  
/**  
 * 디렉터는 빌딩 단계의 순서를 정의합니다. 빌딩 객체와 함께 작동합니다  
 * common Builder 인터페이스를 통해서 어떤 제품을 만들고 있는지 알 수 없습니다.  
 */public class Director {  
  
    public void constructSportsCar(Builder builder) {  
        builder.setCarType(CarType.SPORTS_CAR);  
        builder.setSeats(2);  
        builder.setEngine(new Engine(3.0, 0));  
        builder.setTransmission(Transmission.SEMI_AUTOMATIC);  
        builder.setTripComputer(new TripComputer());  
        builder.setGPSNavigator(new GPSNavigator());  
    }  
  
    public void constructCityCar(Builder builder) {  
        builder.setCarType(CarType.CITY_CAR);  
        builder.setSeats(2);  
        builder.setEngine(new Engine(1.2, 0));  
        builder.setTransmission(Transmission.AUTOMATIC);  
        builder.setTripComputer(new TripComputer());  
        builder.setGPSNavigator(new GPSNavigator());  
    }  
  
    public void constructSUV(Builder builder) {  
        builder.setCarType(CarType.SUV);  
        builder.setSeats(4);  
        builder.setEngine(new Engine(2.5, 0));  
        builder.setTransmission(Transmission.MANUAL);  
        builder.setGPSNavigator(new GPSNavigator());  
    }  
}
```

### Demo.java
```java
public class Demo {  
    public static void main(String[] args) {  
        Director director = new Director();  
  
        // Director는 클라이언트로부터 구현 빌더 객체를 가져옵니다  
        // (애플리케이션 코드) 애플리케이션이 특정 제품을 얻기 위해 사용할 빌더입니다.  
        CarBuilder builder = new CarBuilder();  
        director.constructSportsCar(builder);
        
        // 최종 제품은 종종 빌더 오브젝트에서 검색됩니다.  
		// Director는 구현 빌더와 제품에 의존하지 않고 인지하고 있지 않습니다.  
		Car car = builder.getResult();  
		System.out.println("Car built:\n" + car.getCarType());
		        
		// Director may know several building recipes.  
        director.constructSportsCar(manualBuilder);  
        Manual carManual = manualBuilder.getResult();  
        System.out.println("\nCar manual built:\n" + carManual.print());  
    }  
}
```

#### 실행 결과

``` java
> Task :Demo.main()
Car built:
SPORTS_CAR

Car manual built:
Type of car: SPORTS_CAR
Count of seats: 2
Engine: volume - 3.0; mileage - 0.0
Transmission: SEMI_AUTOMATIC
Trip Computer: Functional
GPS Navigator: Functional
```


# 디자인 패턴 비교

>빌더는 복잡한 객체를 단계별로 생성하는 데 중점을 두고, 추상 팩토리는 관련된 객체들의 패밀리들을 생성하는데 중점을 둔다.
>추상 팩토리는 제품을 즉시 반환하지만 빌더는 제품을 가져오기 전에 몇 가지 추가 생성 단계들을 실행할 수 있도록 한다. 
  

> 복잡한 복합체 패턴 트리를 생성할 때 빌더를 사용할 수 있다. (빌더의 생성 단계들을 재귀적으로 작동하도록 프로그래밍 할 수 있기 때문)

> 빌더를 브릿지와 조합할 수 있다. 디렉터 클래스는 추상화의 역할을 하고 다양한 빌더들은 구현의 역할을 한다.

> 추상 팩토리, 빌더, 프로토타입은 모두 싱글턴으로 구현할 수 있다.
