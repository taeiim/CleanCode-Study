# Clean Code 7장. 오류 처리

> 오류 처리는 프로그램에 반드시 필요한 요소 중 하나이다.

깨끗하고 튼튼한 코드에 한걸음 더 다가가는 단계로 우아하고 고상하게 오류를 처리하는 기법과 고려사항 몇가지를 소개

</br>

### 오류 코드보다 예외를 사용하라

```java
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // 디바이스 상태를 점검
    if (handle != DeviceHandle.INVALID) {
      // 레코드 필드에 디바이스 상태 저장
      retrieveDeviceRecord(handle);
      //디바이스가 일지정지 상태가 아니라면 종료
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
  ...
}
```

- 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법
- 위와 같은 방법을 사용하면 호출자 코드가 복잡해짐
  - 함수를 호출한 즉시 오류를 확인해야 하기 때문

</br>

```java
// Good
public class DeviceController {
  ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }

  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle);
    clearDeviceWorkQueue(handle);
    closeDevice(handle);
  }

  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }
  ...
}
```

- 논리가 오류 처리 코드와 뒤섞이지 않으니까 호출자 코드가 더 깔끔해짐.
  - 앞서 뒤섞였던 개념인 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리
  - 각 개념을 독립적으로 살펴보고 이해할 수 있음

</br></br>

### Try-Catch-Finally 문부터 작성하라

- 예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다.
  - try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.
  - try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

</br></br>

### 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기 쉬워진다.

실패한 연산 이름과 실패 유형도 언급한다.



</br></br>

### 호출자를 고려해 예외 클래스를 정의하라

```java
//Bad
ACMEPort port = new ACMEPort(12);
try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device response exception");
} finally {
  ...
}
```

- 오류를 형편없이 분류한 사례
  - 중복이 심함, 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일
  - 외부 라이브러리를 호출하는 try-catch-finally 문을 포함한 코드로, 외부 라이브러리가 던질 예외를 모두 잡아낸다.

</br>

```java
// Good : 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환

LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}


public class LocalPort {
  private ACMEPort innerPort;
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }

  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  ...
}
```

- LocalPort 클래스는 단순히 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 wrapper 클래스일 뿐
- 외부 API를 사용할 때는 감싸기 기법이 최선
  - LocalPort 클래스는 ACMEPort 감싸는 클래스
  - 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어듬
    - 나중에 다른 라이브러리로 갈아타도 비용이 적다.
  - 감싸기 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 테스트하기도 쉬워짐



</br></br>

### 정상 흐름을 정의하라

```java
// Bad : 예외가 논리를 따라가기 어렵게 만듬
// 특수 상황을 처리할 필요가 없다면 코드가 훨씬 간결해질것
try { 
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal(); 
} catch(MealExpensesNotFound e) {
   m_total += getMealPerDiem(); 
}


// 식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다.
// 식비를 비용으로 청구하지 않았다면 일일 기본 식비를 총계에 더한다.
```

</br>

```java
// ExpenseReportDAO를 고쳐 언제나 MealExpense 객체를 반환
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

=====
  
public class PerDiemMealExpenses implements MealExpenses {
  public int getTotal() {
    // 기본값으로 일일 기본 식비를 반환
  }
}
```

이를 `특수 사례 패턴`이라 부른다.  

`특수 사례 패턴` :  **클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식**이다. 
그러면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다. 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하므로.



</br></br>

### null을 반환하지 마라

null을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 떠넘긴다. 

(누구 하나라도 null 확인을 빼먹는다면 애플리케이션이 통제 불능에 빠질지도 모름)

```java
// Bad : 한 줄 건너 하나씩 null을 확인하는 코드
public void registerItem(Item item) { 
  if (item != null) {
    ItemRegistry registry = peristentStore.getItemRegistry();
    if (registry != null) {
      Item existing = registry.getItem(item.getID());
      if (existing.getBillingPeriod().hasRetailOwner()) {
        existing.register(item);
      }
    }
  }
}
```

</br>

null 을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 **특수 사례 객체**를 반환한다.

```java
// Bad : getEmployees() 가 null도 반환

List<Employee> employees = getEmployees();
if (employees != null) {
  for(Employee e : employees) {
    totalPay += e.getPay();
  }
}
```

```java
// Good : getEmployees를 변경해 빈 리스트를 반환
// 코드도 깔끔해지고 NullPointerException이 발생할 가능성도 줄어듬

List<Employee> employees = getEmployees();
for(Employee e : employees) {
	totalPay += e.getPay();
}

====

public List<Employee> getEmployees() {
	if( .. 직원이 없다면 .. )
		return Collections.emptyList();
	}
}
```



</br></br>

### null을 전달하지 마라

- 메서드에서 null을 반환하는 방식도 나쁘지만 메서드로 null을 전달하는 방식은 더 나쁘다. 
  - 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다.
- 대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다.
  - 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.

```java
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    return (p2.x – p1.x) * 1.5; 
  } 
}

// calculator.xProjection(null, new Point(1, 2)); 
// 누군가 인수로 null을 전달하면 바로 NullPointerException 발생
```

</br></br>

### 결론

- 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다.
- 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지 보수성도 크게 높아진다.

#### TMI

null을 반환/전달하지 말자 ! 

특수 상황 패턴을 사용해 예외사항을 처리하여 코드를 간결하게 하자 ! 



