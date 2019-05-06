# Clean Code 8장. 경계

시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다.
때로는 패키지를 사고, 때로는 오픈소스를 이용한다. 때로는 사내 다른 팀이 제공하는 컴포넌트를 사용한다.

어떤 식으로든 이 **외부 코드를 우리 코드에 깔끔하게 통합** 해야만 한다. 

**소프트웨어 경계를 깔끔하게 처리하는 기법과 기교**를 알아보자. 

</br>

### 외부 코드 사용하기

- 인터페이스 제공자와 사용자 사이에는 특유의 긴장이 존재

  - 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다. (더 많은 환경에서 돌아가야 더 많은 고객)
  - 반면, 사용자는 자신의 요구에 집중하는 인터페이스를 바란다.

- 예시 - java.util.Map

  - Map은 굉장히 다양한 인터페이스로 수많은 기능을 제공
  - Map이 제공하는 기능성과 유연성은 확실이 유용하지만 그만큼 위험도 크다. 

  - 다양한 기능과 유연성은 유용하지만 그만큼 위험하다.
    - ex. 누구나 clear() 할 권한이 있다는 것
    - 설계시 Map에 특정 객체 유형만 저장하기로 결정했어도, Map은 객체 유형을 제한하지 않음.

  | Map 이 제공하는 메서드                           |
  | ---------------------------------------- |
  | clear() void – Map                       |
  | containsKey(Object key) boolean – Map    |
  | containsValue(Object value) boolean – Map |
  | clear() void – Map                       |
  | containsKey(Object key) boolean – Map    |
  | containsValue(Object value) boolean – Map |
  | entrySet() Set – Map                     |
  | equals(Object o) boolean – Map           |
  | get(Object key) Object – Map             |
  | getClass() Class<? extends Object> – Object |
  | hashCode() int – Map                     |
  | isEmpty() boolean – Map                  |
  | keySet() Set – Map                       |
  | notify() void – Object                   |
  | notifyAll() void – Object                |
  | put(Object key, Object value) Object – Map |
  | putAll(Map t) void – Map                 |
  | remove(Object key) Object – Map          |
  | size() int – Map                         |
  | toString() String – Object               |
  | values() Collection – Map                |
  | wait() void – Object                     |
  | wait(long timeout) void – Object         |
  | wait(long timeout, int nanos) void – Object |

</br>

Sensor라는 객체를 담는 Map을 만드려면 다음과 같이 Map 을 생성한다

```java
Map sensors = new HashMap();
```

Sensor 객체가 필요한 코드는 다음과 같이 Sensor 객체를 가져온다.

```java
Sensor s = (Sensor)sensors.get(sensorId);
```

- 위와 같은 코드가 한 번이 아니라 여러 차례 사용된다.
- Map이 반환하는 Object를 **올바른 유형으로 변환할 책임이 Map을 사용하는 클라이언트에 있다.**
  - 깨끗한 코드라 보기 어렵다.
  - 의도도 분명히 드러나지 않는다.

</br>

**제네릭**을 사용하면 코드 가독성이 크게 높아진다.

```java
Map<String, Sensor> sensors = new HashMap<Sensor>();
...
Sensor s = sensor.get(sensorId);
```

하지만 "Map<String, Sensor>이 **사용자에게 필요하지 않은 기능까지 제공**한다."는 **문제**는 해결하지 못한다.

</br>

> 캡슐화가 제일 좋은 방법 ! 

```java
public class Sensors {
  private Map sensors = new HashMap();
  
  public Sensor getById(String id) {
    return (Sensor) sensor.get(id);
  }
}
```

- 경계 인터페이스인 Map을 Sensors 안으로 숨긴다.
  - Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 끼치지 않는다.
    - 변할 가능성이 거의 없다고 여길지도 모르지만, 자바 5가 제네릭을 지원하면서 Map 인터페이스가 변했다는 사실을 명심해야 함.  
  - Sensors 클래스 안에서 객체 유형을 관리하고 변환하기 때문에 제네릭을 사용하든 말든 문제가 안된다.
- Sensors 사용자는 제네릭이 사용되었는지 여부에 신경 쓸 필요가 없다.
  - 제네릭 사용 여부는 Sensors 안에서 결정
- Sensors 클래스는 프로그램에 필요한 인터페이스만 제공한다.
  - 코드를 이해하기 쉽고 오용하기 어렵다.
  - (나머지 프로그램이) 설계 규칙과 비즈니스 규칙을 따르도록 강제할 수 있다.
- Map을 사용할 때마다 위와 같이 캡슐화 하라는 소리가 아니다.
  - Map을(혹은 유사한 경계 인터페이스를) **여기저기 넘기지 말라**는 것이다.
  - **Map과 같은 경계 인터페이스를 사용할 때**는 이를 이용하는 클래스나 클래스 계열 **밖으로 노출되지 않도록 주의**한다.
  - Map 인스턴스를 **공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.**



</br></br>

### 경계 살피고 익히기

> 외부 코드를 익히기는 어렵다. 외부 코드를 통합하기도 어렵다. 두 가지를 동시에 하기는 두 배나 어렵다.

`학습 테스트` : (곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신) **먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히는 것**

- 학습 테스트는 프로그램에서 사용하려는 방식대로 외부 API를 호출한다. 
- 통제된 환경에서 API를 제대로 이해하는지를 확인하는 셈이다.
- 학습테스트는 API를 사용하려는 목적에 초점을 맞춘다.



</br></br>

### log4j 익히기

```java
// 1. "hello"를 출력하는 테스트 케이스
@Test
public void testLogCreate() {
  Logger logger = Logger.getLogger("My Logger");
  logger.info("hello");
}


// Appender라는 뭔가가 필요하다는 오류 발생
```



```java
// 2. 그래서 ConsoleAppender를 생성 후 테스트 케이스
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  ConsoleAppender appender = new ConsoleAppender();
  logger.addAppender(appender);
  logger.info("hello");
}

// 이번에는 Appender에 출력 스트림이 없다는 사실 발견
```



```java
// 3. 출력 스트림이 있어야 정상 아닌가? 구글링 후 다음과 같이 시도
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  logger.removeAllAppenders();
  logger.addAppender(new ConsoleAppender(
    new PatternLayout("%p %t %m%n"),
    ConsoleAppender.SYSTEM_OUT));
  logger.info("hello");
}

// 이제야 제대로 돌아간다.
// 그런데 ConsoleAppender에게 콘솔에 쓰라고 알려야 하다니 뭔가 수상..
// 흥미롭게도 ConsoleAppender.SYSTEM_OUT 인수를 제거 했더니 문제가 없다.
// 하지만, PatternLayout을 제거했더니 또 다시 출력 스트림이 없다는 오류가 뜬다. 아주 수상하다.
```



```java
// 좀 더 구글을 뒤지고, 문서를 읽어보고, 테스트를 돌려보며 log4j가 돌아가는 방식을 상당히 많이 이해
// 여기서 얻은 지식을 간단한 단위 테스트 몇개로 표현
// 이제 모든 지식을 독자적인 로거 클래스로 캡슐화
// 그러면 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 됨

public class LogTest {
    private Logger logger;

    @Before
    public void initialize() {
        logger = Logger.getLogger("logger");
        logger.removeAllAppenders();
        Logger.getRootLogger().removeAllAppenders();
    }

    @Test
    public void basicLogger() {
        BasicConfigurator.configure();
        logger.info("basicLogger");
    }

    @Test
    public void addAppenderWithStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT));
        logger.info("addAppenderWithStream");
    }

    @Test
    public void addAppenderWithoutStream() {
        logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n")));
        logger.info("addAppenderWithoutStream");
    }
}
```



</br></br>

### 학습 테스트는 공짜 이상이다

- 학습 테스트에 드는 비용은 없다. (어쨌든 API를 배워야 하므로) 
  - 오히려 **필요한 지식만 확보하는 손쉬운 방법**이다. 학습 테스트는 이해도를 높여주는 정확한 실험이다.
  - 학습테스트는 공짜 이상이다. 투자하는 노력보다 얻는 성과가 더 크다.
- 학습테스트는 패키지가 예상대로 도는지 검증한다.
  - **패키지 새 버전이 나온다면 학습 테스트를 돌려 차이가 있는지 확인**한다.
  - 이러한 경계 테스트가 있다면 패키지의 **새 버전으로 이전하기 쉬워진다.**
    - 그렇지 않다면, 낡은 버전을 필요 이상으로 오랫동안 사용하려는 유혹에 빠지기 쉽다.

</br></br>

### 아직 존재하지 않는 코드를 사용하기

지금 알지 못하는 코드 (ex. 다른 팀에서 아직 구현이 안됨)를 구현할 때 자체적으로 인터페이스를 정의.

- 필요한 인터페이스를 구현하면 우리가 인터페이스를 전적으로 통제 한다는 장점
- 테스트도 편해짐. 

</br></br>

### 깨끗한 경계

- 경계에 위치하는 코드는 깔끔히 분리
  - 기대치를 정의하는 테스트 케이스도 작성
  - 이쪽 코드에서 외부 패키지를 세세하게 알아야 할 필요가 없다. 
  - 통제 불가능한 외부 패키지에 존재하는 대신 통제 가능한 우리 코드에 의존하는 편이 훨씬 좋다.
- 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리
  - **새로운 클래스로 경계를 감싸**거나 **Adapter 패턴**을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환

</br></br>

#### 

#### TMI

테스트 코드에 대해 공부해 본적이 없어서 인지 아직 완벽히 이해가 되지 않는다.  테스트에 대해 공부해보고 다시 읽어봐야지 ! 