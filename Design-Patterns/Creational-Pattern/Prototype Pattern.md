---
sticker: lucide//align-horizontal-justify-center
tags:
  - design-patterns
  - creational
  - pattern
Prototype-Pattern:
---


![](https://i.imgur.com/XsR7lc9.png)

# 개요

- 프로토타입 패턴은 객체 생성을 최적화하고 새로운 객체를 기존 객체의 복사를 통해 생성하는 디자인 패턴이다.

### 문제점 

객체가 있고 그 객체를 외부에서부터 복사하는 것은 객체의 필드 중 일부가 비공개일 경우 항상 가능하지 않다. 
또한 객체의 복제본을 생성하려면 객체의 클래스를 알아야 하므로, 코드가 해당 클래스에 의존하게 되고, 메서드의 매개 변수가 일부 인터페이스를 따르는 모든 객체를 수락한다면 우리는 그 객체가 따르는 인터페이스만 알고, 
그 객체의 구상 클래스는 알지 못 한다.

### 해결책

프로토타입 패턴은 실제로 복제되는 객체에 복제 프로세스를 위임한다.
패턴은 복제를 지원하는 모든 객체에 대한 공통 인터페이스를 선언하고 이 인터페이스를 사용하면 코드를 객체의 클래스에 결합하지않고도 해당 객체를 복제할 수 있다.
일반적으로 이런 인터페이스에는 단일 `clone` 메서드만 포함된다.

`clone` 메서드의 구현은 모든 클래스에서 유사한데 이 메서드는 현재 클래스의 객체를 만든 후 이전 객체의 모든 필드 값을 새 객체로 전달한다.
대부분의 프로그래밍 언어는 객체들이 같은 클래스에 속한 다른 객체의 비공개 필드에 접근 할 수 있도록 하므로 비공개 필드를 복사하는 것도 가능하다.
이 때 복제를 지원하는 객체를 프로토타입이라고 한다.

만약 사용하는 객체에 수십 개의 필드와 수백 개의 메서드가 있는 경우 이를 복제해 사용하는 것이 서브 클래싱의 대안이 될 수 있다.

# 적용 시기

- **복사해야 하는 객체의 구현 클래스에 코드가 의존하면 안될 때 사용할 수 있다.** 
  
	- 이 경우는 코드가 특정 인터페이스를 통해 외부 API에서 전달된 객체들과 함께 작동할 때 많이 발생한다.
	  이런 객체들의 구현 클래스를 알 수 없기 때문에 의존 할 수 없다.
	  
	- 프로토타입 패턴은 클라이언트 코드에 복제를 지원하는 모든 객체와 작업할 수 있도록 일반 인터페이스를 제공한다.
	  이 인터페이스는 클라이언트 코드가 복제하는 객체의 구현 클래스에서 클라이언트 코드를 독립적으로 유지할 수 있게 한다.
	  
- **각자의 객체를 초기화하는 방식만 다른 자식 클래스들의 수를 줄이고 싶을 때 사용할 수 있다.**
  
	- 프로토타입 패턴은 다양한 방식으로 설정된 미리 만들어진 객체들의 집합을 프로토타입으로 사용할 수 있도록하고
	  일부 설정과 일치하는 자식 클래스를 인스턴스화하는 대신 클라이언트는 간단하게 적절한 프로토타입을 찾아 복제한다.

### 패턴 장점

1. 객체를 구현 클래스에 결합하지 않고 복제할 수 있다.
2. 반복되는 초기화 코드를 제거한 후 미리 만들어진 프로토타입을 복제하는 방법을 사용할 수 있다.
3. 복잡한 객체를 쉽게 생성할 수 있다.
4. 복잡한 객체들에 대한 사전 설정을 처리할 때 상속을 대신 해 사용할 수 있는 방법이다.

### 패턴 단점

- 순환 참조가 있는 복잡한 객체를 복제할 때 매우 까다롭다.

# 프로토타입 패턴 구조

![](https://i.imgur.com/6QIgXi1.png)


### (1) Prototype

프로토타입 인터페이스는 복제 메서드를 선언하며, 이 메서드의 대부분은 단일 `clone` 메서드이다.


### (2) ConcretePrototype

구현 프로토타입 클래스는 복제 메서드를 구현한다.
원본 객체의 데이터를 복제본에 복사하는 것 외에도 이 메서드는 복제 프로세스와 관련된 일부 예외적인 경우도 처리할 수 있다.
(ex. 연결된 객체 복제, 재귀 종속성 제거)

### (3) Client

클라이언트는 프로토타입 인터페이스를 따르는 모든 객체의 복사본을 생성할 수 있다.


![](https://i.imgur.com/2RbxYlp.png)

### (1) PrototypeRegistry

- 프로토타입 레지스트리는 자주 사용하는 프로토타입에 쉽게 접근하는 방법을 제공한다.
- 이 레지스트리에는 복사될 준비가 된 미리 만들어진 객체의 집합을 저장한다.
- 가장 간단한 프로토타입 레지스티리는 `name -> prototype` 해시맵 형태이지만 
  단순히 이름을 검색하는 것보다 더 나은 검색 기준을 설정하는 것으로 보다 더 탄탄한 레지스트리를 구축할 수 있다.

# 구현 방법

1. 프로토타입 인터페이스 혹은 추상 클래스를 생성한 후 `clone`메서드를 선언한다.
   (만약 기존 계층 구조가 있는 경우, `clone`메서드를 계층 구조의 모든 클래스에 추가)
   
2. 프로토타입 클래스는 이 클래스의 객체를 인수로 받는 대체 생성자를 반드시 정의해야 한다.
   또 생성자는 이 클래스에 정의된 모든 필드의 값을 전달된 객체에서 새로 생성된 인스턴스로 복사해야한다.
   자식 클래스를 변경할 때에는 부모 생성자를 호출해 부모 클래스가 부모 클래스의 비공개 필드를 복제하도록 해야 한다.
   
3. 복제 메서드는 일반적으로 한 줄로 구성된다.
   이 한 줄은 생성자의 프로토타입 버전으로 `new`연산자를 실행하고 모든 클래스는 복제 메서드를 오버라이딩한 후 `new` 연산자와 함께 자체 클래스 이름을 사용해야 한다.
   (이렇게 하지 않으면 복제 메서드가 부모 클래스의 객체를 생성할 가능성이 있다.)
   
4. 추가 옵션으로 자주 사용하는 프로토타입을 저장할 중앙 관리형 프로토타입 레지스트리를 생성할 수 있다.

# Prototype Pattern 구현


![](https://i.imgur.com/L6QVDA7.png)

모든 shape 클래스는 같은 인터페이스를 따르며, 이 인터페이스는 복제 메서드를 제공한다.
자식 클래스는 자신의 필드 값을 생성된 객체에 복사하기 전에 부모의 복제 메서드를 호출할 수 있다.

#### Shape.java

``` java
package com.design.pattern.creational.prototype.shapes;  
  
import java.util.Objects;  
  
public abstract class Shape {  
    public int x;  
    public int y;  
    public String color;  
  
    public Shape() {  
    }  
  
    public Shape(Shape target) {  
        if (target != null) {  
            this.x = target.x;  
            this.y = target.y;  
            this.color = target.color;  
        }  
    }  
  
    public abstract Shape clone();  
  
    @Override  
    public boolean equals(final Object obj) {  
        if (!(obj instanceof Shape)) return false;  
        Shape shape = (Shape) obj;  
        return shape.x == x && shape.y == y && Objects.equals(shape.color, color);  
    }  
}
```

#### Circle.java

``` java
package com.design.pattern.creational.prototype.shapes;  
  
public class Circle extends Shape{  
  
    public int radius;  
  
    public Circle() {  
    }  
  
    public Circle(Circle target) {  
        super(target); // 부모 클래스 생성자 호출  
        if (target != null) {  
            this.radius = target.radius;  
        }  
    }  
  
    @Override  
    public Shape clone() {  
        return new Circle(this);  
    }  
  
    @Override  
    public boolean equals(final Object obj) {  
        if (!(obj instanceof Circle) || !super.equals(obj)) return false;  
        Circle shape = (Circle) obj;  
        return shape.radius == radius;  
    }  
}
```

#### Rectangle.java

``` java
package com.design.pattern.creational.prototype.shapes;  
  
public class Rectangle extends Shape{  
    public int width;  
    public int height;  
  
    public Rectangle() {  
    }  
  
    public Rectangle(Rectangle target) {  
        super(target); // 부모 클래스 생성자 호출  
        if (target != null) {  
            this.width = target.width;  
            this.height = target.height;  
        }  
    }  
  
    @Override  
    public Shape clone() {  
        return new Rectangle(this);  
    }  
  
    @Override  
    public boolean equals(final Object obj) {  
        if (!(obj instanceof Rectangle) || !super.equals(obj)) return false;  
        Rectangle shape = (Rectangle) obj;  
        return shape.width == width && shape.height == height;  
    }  
}
```

#### Demo.java

``` java
package com.design.pattern.creational.prototype;  
  
import com.design.pattern.creational.prototype.shapes.Circle;  
import com.design.pattern.creational.prototype.shapes.Rectangle;  
import com.design.pattern.creational.prototype.shapes.Shape;  
import java.util.ArrayList;  
import java.util.List;  
  
public class Demo {  
    public static void main(String[] args) {  
        List<Shape> shapes = new ArrayList<>();  
        List<Shape> shapesCopy = new ArrayList<>();  
  
        Circle circle = new Circle();  
        circle.x = 10;  
        circle.y = 20;  
        circle.radius = 15;  
        circle.color = "red";  
        shapes.add(circle);  
  
        Circle anotherCircle = (Circle) circle.clone();  
        shapes.add(anotherCircle);  
  
        Rectangle rectangle = new Rectangle();  
        rectangle.width = 10;  
        rectangle.height = 20;  
        rectangle.color = "blue";  
        shapes.add(rectangle);  
  
        cloneAndCompare(shapes, shapesCopy);  
    }  
  
    private static void cloneAndCompare(final List<Shape> shapes, final List<Shape> shapesCopy) {  
        for (Shape shape : shapes) {  
            shapesCopy.add(shape.clone());  
        }  
        for (int i = 0; i < shapes.size(); i++) {  
            if (shapes.get(i) != shapesCopy.get(i)) {  
                System.out.println(i + ": Shapes are different objects");  
                if (shapes.get(i).equals(shapesCopy.get(i))) {  
                    System.out.println(i + ": And they are identical");  
                } else {  
                    System.out.println(i + ": But they are not identical");  
                }  
            } else {  
                System.out.println(i + ": Shape objects are the same");  
            }  
        }  
    }  
}
```

#### 실행 결과

![](https://i.imgur.com/rrDzYlG.png)

## 프로토타입 레지스트리

미리 정의된 프로토타입 객체의 집합을 포함한 중앙 집중식 프로토타입 레지스트리​(또는 팩토리)​를 구현하면 이를 통해 이름 또는 다른 매개변수들을 전달하여 팩토리로부터 새로운 객체들을 가져올 수 있다. 
팩토리는 적절한 프로토타입을 검색한 후 복제를 통해 복사본을 반환한다.

#### BundledShapeCache.java

```java
package com.design.pattern.creational.prototype.caching.cache;  
  
import com.design.pattern.creational.prototype.shapes.Circle;  
import com.design.pattern.creational.prototype.shapes.Rectangle;  
import com.design.pattern.creational.prototype.shapes.Shape;  
import java.util.HashMap;  
import java.util.Map;  
  
public class BundledShapeCache {  
    private Map<String, Shape> cache = new HashMap<>();  
  
    public BundledShapeCache() {  
        Circle circle = new Circle();  
        circle.x = 5;  
        circle.y = 7;  
        circle.radius = 45;  
        circle.color = "Green";  
  
        Rectangle rectangle = new Rectangle();  
        rectangle.x = 6;  
        rectangle.y = 9;  
        rectangle.width = 8;  
        rectangle.height = 10;  
        rectangle.color = "blue";  
  
        cache.put("Big green circle", circle);  
        cache.put("Medium blue rectangle", rectangle);  
    }  
  
    public Shape put(String key, Shape shape) {  
        cache.put(key, shape);  
        return shape;  
    }  
  
    public Shape get(String key) {  
        return cache.get(key).clone();  
    }  
}
```

#### Demo.java

``` java
package com.design.pattern.creational.prototype.caching;  
  
import com.design.pattern.creational.prototype.caching.cache.BundledShapeCache;  
import com.design.pattern.creational.prototype.shapes.Shape;  
  
public class Demo {  
    public static void main(String[] args) {  
        BundledShapeCache cache = new BundledShapeCache();  
  
        Shape shape1 = cache.get("Big green circle");  
        Shape shape2 = cache.get("Medium blue rectangle");  
        Shape shape3 = cache.get("Medium blue rectangle");  
  
        if (shape1 != shape2 && !shape1.equals(shape2)) {  
            System.out.println("Big green circle != Medium blue rectangle");  
        } else {  
            System.out.println("Big green circle == Medium blue rectangle");  
        }  
  
        if (shape2 != shape3) {  
            System.out.println("Medium blue rectangles are two different objects");  
            if (shape2.equals(shape3)) {  
                System.out.println("And they are identical");  
            } else {  
                System.out.println("But they are not identical");  
            }  
        } else {  
            System.out.println("Rectangle objects are the same");  
        }  
    }  
}
```


### 실행 결과

![](https://i.imgur.com/Z3lxwFJ.png)


# 디자인 패턴 비교

> 프로토타입을 사용하여 추상 팩토리의 구상 클래스의 생성 메서드를 구현할 수 있다.
>
   프로토타입은 커맨드 패턴의 복사본을 저장해야 할 때 도움이 될 수 있다.

> 데코레이터 및 복합체 패턴을 많이 사용하는 패턴은 프로토타입을 사용해 이득을 볼 수 있다.
> (프로토타입 패턴을 적용하면 복잡한 구조들을 처음부터 다시 건축하는 대신 복제할 수 있기 때문이다.)

> 프로토타입은 상속 기반이 아니기 때문에 상속과 관련된 단점이 없다. 대신 복제된 객체의 복잡한 초기화가 필요하다.
> 이와 반대로 팩토리 메서드는 상속을 기반으로 하지만 초기화 단계가 필요하지않다.

> 때로는 프로토타입이 메멘토 패턴의 간단한 대안이 될 수 있다.

> 추상 팩토리, 빌더 및 프로토타입은 모두 싱글턴으로 구현할 수 있다.


# 정리

- **프로토타입**은 객체들​(복잡한 객체 포함)​을 특정 클래스들에 결합하지 않고 복제할 수 있도록 하는 생성 디자인 패턴이다.
  
- 모든 프로토타입 클래스들은 객체들의 구상 클래스들을 알 수 없는 경우에도 해당 객체들을 복사할 수 있도록 하는 공통 인터페이스가 있어야 한다.
  
- 프로토타입 객체들은 전체 복사본을 생성할 수 있다.
  (같은 클래스의 객체들은 서로 비공개 필드에 접근할 수 있기 때문이다.)
  
- 자바에서 프로토타입 패턴은 `Cloneable` 인터페이스를 통해 바로 사용할 수 있다.