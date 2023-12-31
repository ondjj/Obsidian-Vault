---
sticker: lucide//factory
tistoryBlogName: ondj
tistoryTitle: Abstract Factory
tistoryTags: design-pattern,디자인패턴,추상팩토리
tistoryVisibility: "0"
tistoryCategory: "1096109"
tistorySkipModal: true
tistoryPostId: "118"
tistoryPostUrl: https://ondj.tistory.com/118
---


![](https://i.imgur.com/0HILFjq.png)


# 개요

추상 팩토리 패턴은 연관성이 있는 객체 군이 여러 개 있을 경우 이들을 묶어 추상화하고, 어떤 구체적인 상황이 주어지면 팩토리 객체에서 집합으로 묶은 객체 군을 구현화 하는 생성 패턴이다.
클라이언트에서 특정 객체를 사용할 때 패토리 클래스만을 참조하여 특정 객체에 대한 구현부를 감추어 역할과 구현을 분리시킬 수 있다.

즉, 추상 팩토리의 핵심 제품군 집합을 타입 별로 찍어낼 수 있다는 점이 포인트이다. 예를들어 모니터, 마우스, 키보드를 묶은 전자 제품군을 삼성 제품인지, 애플 제품인지에 따라 집합이 브랜드 명으로 여러 갈래로 나뉘게 될 때, 복잡하게 묶이는 이러한 제품들을 관리와 확장에 용이하게 패턴화 한것이 추상 팩토리이다.



![](https://i.imgur.com/3s3tg0Q.png)

### 문제점 

가구 판매장을 위한 프로그램을 만들고 있다고 가정하고, 우리가 만든 코드는 다음을 나타내는 클래스들로 구성된다.

1. 관련 제품들로 형성된 패밀리(제품군) : 의자, 소파, 커피 테이블 ..
2. 해당 제품군의 여러 가지 형식 : Modern, Victorian, ArtDeco ..

이제 새로운 가구 객체를 생성했을 때, 이 객체들이 기존의 같은 패밀리 내에 있는 다른 가구 객체들과 일치하는 형식(스타일)을 가지도록 할 방법이 필요하다.
또, 공급 업체들의 카탈로그는 자주 변경되기 때문에 새로운 제품 또는 제품군을 추가할 때마다 기존 코드를 변경해야 하는 번거로움이 존재한다.

### 해결책

추상 팩토리 패턴의 첫 번째 방안은 각 제품 패밀리(제품군)에 해당하는 개별적인 인터페이스를 명시적으로 선언하는 것이다.(의자, 소파, 커피 테이블 ...)
그 다음 제품의 모든 클래스가 위 인터페이스를 따르도록 한다. `예를 들어, 모든 의자의 클래스는 Chair 인터페이스를 구현한다.` 등의 규칙을 명시한다.

![](https://i.imgur.com/hjIx0mo.png)


> [!NOTE]
**> 같은 객체의 모든 모델은 단일 클래스 계층구조로 옮겨져야 한다.**
> 이 말은 객체 지향 프로그래밍에서 중요한 개념 중 하나인 단일 책임 원칙(Single Responsibility Principle, SRP)에 관련된 내용으로
> 원칙을 더 구체적으로 설명하면, "같은 객체의 모든 모델"이라는 것은 비슷한 역할을 하는 객체나 클래스들이 있다면 이들을 단일 클래스 계층 구조로 묶어
> 동일한 부모 클래스나 인터페이스를 공유하도록 설계해 코드의 재사용성을 높이고 유지보수를 용이하게 함을 나타낸다. 

다음 단계는 추상 팩토리 패턴을 선언하는 것으로 추상 팩토리 패턴은 제품 패밀리 내의 모든 개별 제품들의 생성 메서드들이 목록화되어있는 인터페이스이다.
(예 : `create­Chair-의자생성, create­Sofa-소파생성, create­Coffee­Table-커피 테이블 생성` ..)

![](https://i.imgur.com/MwvfgBI.png)


> [!info] 
> 각 구상 패토리는 특정 제품의 변형에 해당한다.

다음은 제품 모델을 다룰 차례로 제품 패밀리의 각 모델에 대해 `Abstract­Factory` 추상 팩토리 인터페이스를 기반으로 별도의 팩토리 클래스를 생성한다.
팩토리는 특정 종류의 제품을 반환하는 클래스로 예를 들어 `Modern­Furniture­Factory(현대식 가구 팩토리)` 에서는 `Modern­Chair`​(현대식 의자), `Modern­Sofa`​(현대식 소파) 및 `Modern­Coffee­Table`​(현대식 커피 테이블) 객체만 생성할 수 있다.

클라이언트 코드는 자신에 해당하는 추상 인터페이스를 통해 팩토리와 제품 모두와 함께 동작해야 한다. 그래야 클라이언트 코드에 넘기는 팩토리 종류와 제품 모델을 클라이언트 코드를 손상하지 않으며 자유자재로 변경할 수 있다.
이 때, 클라이언트는 함께 작업하는 팩토리의 구현 클래스에 대해 전혀 알지 못하도록 해야 한다.

클라이언트가 팩토리에 의자를 주문했다고 가정해 보자. 클라이언트는 팩토리의 클래스를 알 필요가 없으며, 팩토리가 어떤 모델의 의자를 생성할지 전혀 신경 쓰지 않는다.
`abstract Chair 인터페이스`를 사용하여 현대식 의자이든 빅토리아식 의자이든 종류에 상관 없이 모든 의자를 항상 동일한 방식으로 주문하며, 클라이언트가 의자에 대해 아는 유일한 정보는 `sitOn 메서드`를 구현한다는 것뿐이다.
그러나, 생성된 의자의 모델은 항상 같은 팩토리 객체에서 생성된 소파 또는 커피 테이블의 모델과 같을 것이다.

여기서 명확히 짚고 넘어가야 할 점은 클라이언트가 추상 인터페이스에만 노출된다면 실제 팩토리 객체를 생성하는 것은 무엇일까?
일반적으로 프로그램은 초기화 단계에서 구상 팩토리 객체를 생성하고, 그 직전에 프로그램은 환경 또는 구성 설정에 따라 팩토리 유형을 선택한다.

# 적용 시기

- **추상 팩토리는 코드가 관련된 제품군의 다양한 패밀리들과 작동해야 하지만, 해당 제품들의 구현 클래스에 의존하고 싶지 않을 때 사용할 수 있다.**
  
	- 추상 팩토리는 제품 패밀리의 각 클래스에서부터 객체들을 생성할 수 있는 인터페이스를 제공한다.
	  위 인터페이스를 통해 코드가 객체를 생성하는 한, 앱에서 이미 생성된 제품들과 일치하지 않는 잘못된 제품 모델을 생성하지 않을지 걱정할 필요가 없다.

- **코드에 클래스가 있고, 이 클래스의 팩토리 메서드들의 집합의 기본 책임이 뚜렷하지 않을 때 추상 팩토리 구현을 고려한다.**
	- 잘 설계된 프로그램에서는 각 클래스는 하나의 책임만 가진다. 클래스가 여러 제품 유형을 상대할 경우, 
	  클래스의 팩토리 메서드들을 독립 실행형 팩토리 클래스 또는 완전한 추상 팩토리 구현으로 추출할 가치가 있을 수 있다.

### 패턴 장점

- 팩토리에서 생성되는 제품의 상호 호환을 보장할 수 있다.
- 구현 제품과 클라이언트 코드 사이의 단단한 결합을 피할 수 있다.
- SRP, 제품 생성 코드를 한 곳으로 추출하여 코드를 더 쉽게 유지보수 할 수 있다.
- OCP, 기존 클라이언트 코드를 훼손하지 않고 제품의 새로운 모델을 생성할 수 있다.

### 패턴 단점

- 패턴과 함께 새로운 인터페이스들과 클래스들이 많이 도입되기 때문에 코드가 필요 이상으로 복잡해질 수 있다.


# 패턴 구조

![](https://i.imgur.com/6wapxCA.png)

### (1)  Abstract Product

- 추상 제품은 제품 패밀리를 구성하는 개별 연관 제품의 집합에 대한 인터페이스를 선언한다.

### (2) Concrete Product

- 구현 제품은 모델들로 그룹화된 추상 제품의 다양한 구현으로 각 추상 제품은 주어진 모든 모델에 구현되어야 한다.

### (3) Abstact Factory

- 추상 팩토리 인터페이스는 각각의 추상 제품을 생성하기 위한 여러 메서드들의 집합을 선언한다.

### (4),(5) Concrete Factory

- 구현 팩토리는 추상 팩토리의 생성 메서드를 구현한다. 각 구현 팩토리는 제품들의 특정 모델들에 해당하며 해당 특정 모델만 생성한다.
- 구현 팩토리는 구현 제품들을 인스턴스화 하지만, 그 제품들의 생성 메서드의 시그니처는 그에 해당하는 추상 제품을 반환해야 한다.
  그래야 팩토리를 사용하는 클라이언트 코드가 팩토리에서 받은 제품의 특정 모델과 결합되지 않는다.
- 클라이언트는 추상 인터페이스를 통해 팩토리/제품 변형의 객체들과 소통하는 한 모든 구현 팩토리/ 제품 모델과 작업할 수 있다.

# 구현 방법


1. 고유한 제품 유형과 모델 제품을 나타내는 매트릭스를 매핑한다.
   
2. 모든 제품 모델들에 대한 추상 제품 인터페이스를 선언하고 모든 구현 제품 클래스들이 위 인터페이스를 구현하도록 한다.
   
3. 추상 팩토리 인터페이스를 모든 추상 제품들에 대한 생성 메서드의 집한과 함께 선언한다.
   
4. 각 제품 모델에 대해 각각 하나의 구현 팩토리 클래스 집합을 구현한다.
   
5. 앱 어딘가에 팩토리 초기화 코드를 생성한다.
   (초기화 코드는 앱 설정 또는 현재 환경에 따라 구현 팩토리 클래스 중 하나를 인스턴스화 해야 한다. 이 팩토리 객체를 제품을 생성하는 모든 클래스에 전달해야한다.)
   
6. 코드를 검토해서 제품 생성자에 대한 모든 직접 호출을 찾고, 이 호출을 팩토리 객체에 대한 적절한 생성 메서드 호출로 교체한다.


# Pattern 구현


아래 예시에선 버튼과 체크박스는 제품 역할을 하며, `mac` 과 `window` 두 가지 모델이 존재한다.
추상 팩토리는 버튼과 체크박스를 생성하기 위한 인터페이스를 정의하고, 또 두 개의 구현 팩토리들이 있으며, 이들은 모든 제품을 단일 모델로 반환한다.

클라이언트 코드는 추상 인터페이스를 사용하여 팩토리 및 제품과 함께 작동하며, 이는 팩토리 객체의 유형에 따라 같은 클라이언트 코드가 많은 제품 모델과 함께 작동하도록 한다.

![](https://i.imgur.com/0GbPzjY.png)

## **buttons:** 첫 번째 제품의 계층구조

#### **buttons/Button**.java

``` java
package com.design.pattern.creational.abstractfactory.example.buttons;  
  
  
/**  
 * 추상 공장에서는 여러 제품군을 보유하고 있다고 가정합니다,  
 * 별도의 클래스 계층 구조(Button/Checkbox)로 구성되어 있습니다.  
 * 모든 제품의 같은 패밀리는 공통 인터페이스를 가지고 있습니다.  
 * 버튼 제품군의 공통 인터페이스입니다.  
 */
 public interface Button {  
    void paint();  
}
```

#### **buttons/MacOSButton**.java

``` java
package com.design.pattern.creational.abstractfactory.example.buttons;  
  
/**  
 * 모든 제품군은 동일한 종류(MacOS/Windows)를 가지고 있습니다.  
 * MacOS 모델 버튼입니다.  
 */
 public class MacOSButton implements Button{  
    @Override  
    public void paint() {  
        System.out.println("You have created MacOSButton");  
    }  
}
```

#### **buttons/WindowsButton**.java

``` java
package com.design.pattern.creational.abstractfactory.example.buttons;  
  
/**  
 * 모든 제품군은 동일한 종류(MacOS/Windows)를 가지고 있습니다.  
   버튼의 또 다른 모델입니다.  
 */
 public class WindowsButton implements Button{  
    @Override  
    public void paint() {  
        System.out.println("You have created WindowsButton.");  
    }  
}
```

## **checkboxes:** 두 번째 제품의 계층구조
#### **checkboxes/Checkbox**.java

``` java
package com.design.pattern.creational.abstractfactory.example.checkboxes;  
  
/**  
 * 체크박스는 두번째 제품군입니다.  
 * 버튼과 동일한 모델을 가지고 있습니다.  
 */
 public interface Checkbox {  
  
    void paint();  
}
```

#### **checkboxes/MacOSCheckbox**.java

```java
package com.design.pattern.creational.abstractfactory.example.checkboxes;  
  
/**  
 * 모든 제품군은 동일한 종류(MacOS/Windows)를 가지고 있습니다.  
 * 체크박스의 모델입니다.  
 */
 public class MacOSCheckbox implements Checkbox{  
    @Override  
    public void paint() {  
        System.out.println("You have created MacOSCheckbox.");  
    }  
}
```

#### **checkboxes/WindowsCheckbox**.java

```java
package com.design.pattern.creational.abstractfactory.example.checkboxes;  
  
/**  
 * All products families have the same varieties (MacOS/Windows). *
 * This is another variant of a checkbox. */
 public class WindowsCheckbox implements Checkbox{  
    @Override  
    public void paint() {  
        System.out.println("You have created WindowCheckbox.");  
    }  
}
```

## **factories**

### **factories/GUIFactory.java:** 추상 팩토리
```java
package com.design.pattern.creational.abstractfactory.example.factories;  
  
import com.design.pattern.creational.abstractfactory.example.buttons.Button;  
import com.design.pattern.creational.abstractfactory.example.checkboxes.Checkbox;  
  
/**  
 * Abstract factory knows about all (abstract) product types. */
public interface GUIFactory {  
    Button createButton();  
    Checkbox createCheckbox();  
}
```

### **factories/MacOSFactory.java:** 구상 팩토리 (맥)
```java
package com.design.pattern.creational.abstractfactory.example.factories;  
  
import com.design.pattern.creational.abstractfactory.example.buttons.Button;  
import com.design.pattern.creational.abstractfactory.example.buttons.MacOSButton;  
import com.design.pattern.creational.abstractfactory.example.checkboxes.Checkbox;  
import com.design.pattern.creational.abstractfactory.example.checkboxes.MacOSCheckbox;  
  
/**  
 * 각 구현 팩토리는 추상 팩토리를 확장하고 생산을 담당합니다.  
 * 단일 모델 제품  
 */  
public class MacOSFactory implements GUIFactory{  
    @Override  
    public Button createButton() {  
        return new MacOSButton();  
    }  
  
    @Override  
    public Checkbox createCheckbox() {  
        return new MacOSCheckbox();  
    }  
}
```

### **factories/WindowsFactory.java:** 구상 팩토리 (윈도우)
```java
package com.design.pattern.creational.abstractfactory.example.factories;  
  
import com.design.pattern.creational.abstractfactory.example.buttons.Button;  
import com.design.pattern.creational.abstractfactory.example.buttons.WindowsButton;  
import com.design.pattern.creational.abstractfactory.example.checkboxes.Checkbox;  
import com.design.pattern.creational.abstractfactory.example.checkboxes.WindowsCheckbox;  
  
/**  
 * 각 구현 팩토리는 추상 팩토리를 확장하고 생산을 담당합니다.  
 * 단일 모델 제품  
 */  
public class WindowsFactory implements GUIFactory{  
    @Override  
    public Button createButton() {  
        return new WindowsButton();  
    }  
  
    @Override  
    public Checkbox createCheckbox() {  
        return new WindowsCheckbox();  
    }  
}
```


## app

### app/Application.java : 클라이언트 코드

``` java
package com.design.pattern.creational.abstractfactory.example.app;  
  
import com.design.pattern.creational.abstractfactory.example.buttons.Button;  
import com.design.pattern.creational.abstractfactory.example.checkboxes.Checkbox;  
import com.design.pattern.creational.abstractfactory.example.factories.GUIFactory;  
  
/**  
 * 클라이언트는 어떤 구현 팩토리를 사용하든 상관하지 않습니다.  
 * 추상적인 인터페이스를 통한 공장과 제품.  
 */
 public class Application {  
  
    private Button button;  
    private Checkbox checkbox;  
  
    public Application(GUIFactory factory) {  
        button = factory.createButton();  
        checkbox = factory.createCheckbox();  
    }  
  
    public void paint() {  
        button.paint();  
        checkbox.paint();  
    }  
}
```

## Demo.java : 앱 설정 및 실행

``` java
package com.design.pattern.creational.abstractfactory.example;  
  
  
import com.design.pattern.creational.abstractfactory.example.app.Application;  
import com.design.pattern.creational.abstractfactory.example.factories.GUIFactory;  
import com.design.pattern.creational.abstractfactory.example.factories.MacOSFactory;  
import com.design.pattern.creational.abstractfactory.example.factories.WindowsFactory;  
  
/**  
 * Demo class. Everything comes together here. 
 */
public class Demo {   
    /**  
     * 응용 프로그램이 팩토리 유형을 선택하고 실행 시간 내에 생성합니다(보통 초기화 단계에서)  
     * 구성 또는 환경 변수에 따라 달라질 수 있습니다.  
     */  
    private static Application configureApplication() {  
        Application app;  
        GUIFactory factory;  
        String osName = System.getProperty("os.name").toLowerCase();  
        if (osName.contains("mac")) {  
            factory = new MacOSFactory();  
        } else {  
            factory = new WindowsFactory();  
        }  
        app = new Application(factory);  
        return app;  
    }  
  
    public static void main(String[] args) {  
        Application app = configureApplication();  
        app.paint();  
    }  
}
```
#### 실행 결과

```java
Starting Gradle Daemon...
Gradle Daemon started in 1 s 739 ms
> Task :compileJava
> Task :processResources UP-TO-DATE
> Task :classes

> Task :Demo.main()

You have created MacOSButton
You have created MacOSCheckbox.
```


![](https://i.imgur.com/HZkw8iB.png)

# 다른 패턴과의 관계

> 많은 디자인은 복잡성이 낮고 자식 클래스를 통해 더 많은 커스터마이징이 가능한 팩토리 메서드로 시작해 더 유연하면서도 더 복잡한 추상 팩토리, 프로토타입 또는 빌더 패턴으로 발전해 나간다.

> 빌더는 복잡한 객체들을 단계별로 생성하는데 중점을 두고, 추상 팩토리는 관련된 객체들의 패밀리들을 생성하는데 중접을 둔다.
> 추상 팩토리는 제품을 즉시 반환하지만 빌더는 제품을 가져오기 전에 사용자가 몇 가지 추가 생성 단계들을 실행할 수 있도록 한다. 

> 추상 팩토리 클래스는 팩토리 메서드의 집합을 기반으로 하는 경우가 많지만, 프로토타입을 사용하여 추상 팩토리의 구현 클래스의 생성 메서드를 구현할 수도 있다.

> 추상 팩토리는 하위 시스템 객체들이 클라이언트 코드에서 생성되는 방식만 숨기고 싶을 때 퍼사드 대신 사용할 수 있다.

> 추상 팩토리를 브리지와 함께 사용할 수 있다. 이 조합은 브리지에 의해 정의된 어떤 추상화들이 특정 구현에 의해 작동할 수 있을 때 유용하다.
> 이런 경우 추상 팩토리는 이러한 관계들을 캡슐화하고 클라이언트 코드에서부터 복잡성을 숨길 수 있다.


# 정리

- 추상 팩토리는 고유한 제품을 생성하기 위해 인터페이스를 정의하지만 실제 제품 생성은 구현 팩토리 클래스에 맡긴다.
  (이 때, 각 팩토리 유형은 특정 제품군에 해당한다.)
  
- 클라이언트 코드는 생성자 호출(`new 연산자`)로 직접 제품을 생성하는 대신 팩토리 객체의 생성 메서드를 호출한다.
  (팩토리는 단일 제품 모델에 해당하므로 해당 팩토리의 모든 제품이 호환된다.)
  
- 클라이언트 코드는 추상 인터페이스를 통해서만 팩토리 및 제품과 함께 작동하며, 이렇게 하면 클라이언트 코드가 팩토리 객체에 의해 생성된 모든 제품 모델과 함께 작동할 수 있다.
  새로운 구현 팩토리 클래스를 생성한 후 클라이언트 코드에 전달한다.