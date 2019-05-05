# Clean Code 6장. 객체와 자료구조

### 자료 추상화

두 클래스는 모두 2차원 점을 표현

그런데, 한 클래스는 구현을 외부로 노출하고 다른 클래스는 구현을 완전히 숨김

```Java
// 구현을 외부로 노출
// 확실히 직교좌표계 사용
// 개별적으로 get,set 강제
public class Point {
  public double x;
  public double y;
}

// 구현을 완전히 숨김
// 직교좌표계를 사용하는지 극좌표계를 사용하는지 알 길이 없음. 그럼에도 자료구조를 명백하게 표현
// get할 때는 개별적으로, set할 때는 두 값을 한번에 설정
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

첫번째 Point 클래스의 변수를 private으로 선언하더라도 
각 값마다 get/set 함수를 제공한다면 구현을 외부로 노출하는 셈

**변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지 않음.**
구현을 감추려면 `추상화`가 필요함. 
추상 인터페이스를 제공해 **사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야** 진정한 의미의 클래스다. 



```Java
// 자동차 연료상태를 구체적인 숫자 값으로 알려줌
// 변수 값을 읽어 반환할 뿐이라는 사실이 거의 확실
public interface Vehicle {
  double getFuelTankCapacityInGallons();
  double getGallonsOfGasoline();
}

// 자동차 연료상태를 백분율이라는 추상적인 개념으로 알려줌
// 정보가 어디서 오는지 전혀 드러나지 않음.
public interface Vehicle {
  double getPercentFuelRemaining();
}
```

**자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다. **
인터페이스나 get/set 함수만으로는 추상화가 이루어지지 않음. 

**개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민**해야 함. 
아무 생각없이 get/set 함수를 추가하는 방법이 가장 나쁘다. 

</br></br>

### 자료/객체 비대칭

- 객체 : 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개
- 자료 구조 : 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다. 

```Java
// 절차적인 도형
// Geometry 클래스는 세가지 도형클래스 다룸
// 각 도형 클래스는 간단한 자료구조 => 아무 메서드도 제공하지 않음
// 도형이 동작하는 방식은 Geometry 클래스에서 구현
public class Square {
  public Point topLeft;
  public double side;
}

public class Rectangle {
  public Point topLeft;
  public double height;
  public double width;
}

public class Circle {
  public Point center;
  public double radius;
}

public class Geometry {
  public final double PI = 3.141592653589793;

  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) {
      Square s = (Square)shape;
      return s.side * s.side;
    } 
    else if (shape instanceof Rectangle) {
      Rectangle r = (Rectangle)shape;
      return r.height * r.width;
    } 
    else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius;
    }
    throw new NoSuchShapeException();
  }
}
```

클래스가 절차적이라 비판한다면 맞는 말이다. 하지만 그런 비웃음이 100% 옳다고 말하기 어렵다. 

- 만약 **Geometry 클래스에** perimeter() **함수를 추가**하고 싶다면 ?  => **도형 클래스는 아무 영향도 받지 않는다** !
- 반대로 **새 도형을 추가**하고 싶다면 ? => Geometry **클래스에 속한 함수를 모두 고쳐야 한다. **



```Java
// 객체 지향적인 도형 클래스 / 다형적인 도형
// area() 는 다형(polymorphic) 메서드 , Geometry 클래스 필요없음
public class Square implements Shape {
  private Point topLeft;
  private double side;

  public double area() {
    return side * side;
  }
}

public class Rectangle implements Shape {
  private Point topLeft;
  private double height;
  private double width;

  public double area() {
    return height * width;
  }
}

public class Circle implements Shape {
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  }
}
```

- **새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않음.**
- 반면 **새 함수를 추가**하고 싶다면 **도형 클래스 전부를 고쳐야** 함.



#### 정리

> (자료 구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 
> 반면, 객체지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다.
> 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다.

- 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉽고, 
  절차 적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.
- 복잡한 시스템을 짜다 보면 새로운 함수가 아니라 새로운 자료 타입이 필요한 경우가 생긴다. 
  이때는 클래스와 객체 지향 기법이 가장 적합하다.
- 반면, 새로운 자료타입이 아니라 새로운 함수가 필요한 경우도 생긴다. 
  이때는 절차적인 코드와 자료구조가 좀 더 적합하다. 

</br></br>

### 디미터 법칙

디미터 법칙은 **모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다**는 법칙이다.

**객체는 자료를 숨기고 함수를 공개**한다. 즉, **객체는 조회 함수로 내부 구조를 공개하면 안 된다**는 의미다. 
즉, 객체는 조회 함수로 내부구조를 공개하면 안 된다는 의미다. 그러면 내부 구조를 (숨기지 않고) 노출하는 셈이니까.

클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다.

- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체 


but, 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안됨.

</br>


#### 기차 충돌

아래와 같은 코드는 여러 객차가 한 줄로 이어진 기차처럼 보이기 때문에 *기차 충돌* 이라 부른다.

일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```



다음과 같이 나누는 편이 좋다. 

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScrathDir();
final String outputDir = scratchDir.getAbsolutePath();
```



##### 위의 코드 둘 다 디미터 법칙을 위반할까?

ctxt, Options, ScratchDir 이 객체인지 아니면 자료 구조인지에 달렸다. 

- **객체**라면 내부 구조를 숨겨야 하므로 확실히 **디미터 법칙을 위반.**
- **자료 구조**라면 당연히 내부 구조를 노출하므로 **디미터 법칙이 적용되지 않음. **

그런데 위 예제는 조회 함수를 사용하는 바람에 혼란을 일으킨다. 
아래와 같이 구현 했다면 디미터 법칙을 거론할 필요가 없어진다.

```Java
final String outputDir = ctxt.options.scratchDir.absolutePath
```

</br>

#### 잡종 구조

이런 혼란으로 말미암아 때때로 **절반은 객체, 절반은 자료 구조인 잡종구조**가 나온다.

잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개변수는 공개 get/set 함수도 있다. 공개 get/set함수는 비공개 변수를 그대로 노출한다.  

이런 잡종구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다. 양쪽 세상에서 단점만 모아놓은 구조이다. 
그러므로 잡종구조는 되도록 피하는 편이 좋다.

</br>

#### 구조체 감추기

만약 ctxt, options, scratchDir이 진짜 객체라면? 그렇다면 앞서 코드처럼 줄줄이 사탕으로 엮여서는 안된다. 
객체라면 내부구조를 감쳐야 하니까. 

그렇다면 임시 디렉터리의 절대 경로는 어떻게 얻어야 좋을까? 

ctxt가 객체라면 뭔가를 하라고 말해야지 속을 드러내라고 말하면 안된다. 
임시 디렉터리의 절대 경로가 왜 필요할까? 절대 경로를 얻어 어디에 쓰려고? => ex. 임시 파일을 생성하기 위해

ctxt 객체에 임시 파일을 생성하라고 시키기! 

```Java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

ctxt는 내부 구조를 드러내지 않으며, 모듈에서 해당함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다. 
따라서 디미터 법칙을 위반하지 않는다.

</br></br>

### 자료 전달 객체

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스 (== 자료 전달 객체 ==DTO )
데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용

- private 비공개 변수를 get/set 함수로 조작
- 일종의 사이비 캡슐화로, 별다른 이익을 제공하지 않음

</br>

#### 활성 레코드

DTO의 특수한 형태 

공개 변수가 있거나 비공개 변수에 get/set 함수가 있는 자료 구조지만, 대개 save나 find와 같은 탐색 함수도 제공

불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔하다. 하지만 이는 바람직하지 않다. 그러면 자료 구조도 아니고 객체도 아닌 잡종 구조가 나오기 때문

해결책 => 활성 레코드는 자료구조로 취급해야 함. 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성.

</br></br>

### 결론

객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다. 

자료 구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다. 

시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다. 다른 경우로 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다. 

우수한 소프트웨어 개발자는 편견없이 이 사실을 이해해 직면한 문제해 최적인 해결책을 선택한다.

#### TMI

흠.. 몇 번더 읽어보고 학습해야 할 장인것 같다.

객체와 자료구조,, 이 장에서 저자가 뭘 말하고 싶은건지는 알겠지만, 이걸 어떻게 코드에 적용해야하고 내가 어떤 식으로 고려해 객체냐 자료구조냐를 선택할 지 사실 아직 잘 모르겠다.  

사실 나는 get/set 함수와 DTO를 정말 많이 사용했다. private 변수에 접근하기 위해 사용했었는데, 이걸 어떻게 사용해야 코드에 더 도움이 되는지, 뭐가 잘못된건지, DTO 를 사용하지 않고 다른 방법은 무엇이있는지 등에 대해 더 생각해봐야겠다. 

