# **Java 예외 처리: try-catch-finally와 try-with-resources**

---

- `try-catch-finally`와 `try-with-resources` 문법 이해, 그 차이점과 적절한 활용 방법
- Java 예외 처리 기본 개념 및 자원 관리 패턴

---

### **1. 예외 처리의 기본 개념**

- **예외란 무엇인가?**
    - 프로그램 실행 중 발생하는 예기치 않은 오류 또는 이벤트.
    - 예외를 처리하지 않으면 프로그램이 비정상적으로 종료될 수 있음.
- **예외 처리의 필요성**
    - 안정적인 프로그램 실행 보장.
    - 자원의 누수 방지.
    - 오류 정보를 정확하게 파악하고, 적절한 대응 조치 가능.

### **2. try-catch-finally**

- **try 블록**: 예외가 발생할 수 있는 코드가 들어가는 부분.
- **catch 블록**: 발생한 예외를 잡아내고 처리하는 부분.
- **finally 블록**: 예외 발생 여부와 상관없이 반드시 실행되는 블록 (주로 자원 해제에 사용).

```java
try {
    // 예외가 발생할 수 있는 코드
} catch (Exception e) {
    // 예외 처리
} finally {
    // 자원 해제 등 무조건 실행되는 코드
}
```

- **장점**
    - 예외 발생 시 흐름을 중단하지 않고 처리 가능.
    - 자원 정리 코드를 `finally`에서 처리하여, 예외가 발생하더라도 자원을 반드시 반납할 수 있음.
- **단점**
    - 코드가 장황해질 수 있으며, 자원 관리를 위해 `finally`에서 명시적으로 해제해야 함.
- **try-catch-finally 예시 코드**

```java
FileInputStream inputStream = null;
try {
    inputStream = new FileInputStream("data.txt");
    // 파일 처리 로직
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (inputStream != null) {
        try {
            inputStream.close();  // 자원 해제
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- **설명**
    - 예외 발생 시 `catch`에서 처리.
    - 파일을 닫는 작업은 `finally`에서 명시적으로 처리해야 함.

### **3. try-with-resources**

- **try-with-resources**: `AutoCloseable` 또는 `Closeable`을 구현한 자원을 자동으로 반납하는 구문.
- **구문**: try() 내부에 자원을 선언하면, 블록 종료 시 자동으로 자원을 닫아줌.

```java
try (ResourceType resource = new ResourceType()) {
    // 자원 사용
} catch (ExceptionType e) {
    // 예외 처리
}
```

- **장점**
    - 자원 반납을 자동으로 처리하므로, 코드가 간결해짐.
    - 자원을 명시적으로 닫을 필요가 없어 실수로 자원을 반납하지 않는 상황을 방지할 수 있음.
    - 가독성과 유지보수성 향상.
- **try-with-resources 예시 코드**
    
    ```java
    try (FileInputStream inputStream = new FileInputStream("data.txt")) {
        // 파일 처리 로직
    } catch (IOException e) {
        e.printStackTrace();
    }
    ```
    
- **설명**:
    - `inputStream`은 `AutoCloseable` 인터페이스를 구현했기 때문에, `try-with-resources` 블록이 끝나면 자동으로 `close()` 호출됨.
    - `finally` 블록이 불필요해지고, 코드가 간결해짐.

### **4. 비교**

| 구분 | try-catch-finally | try-with-resources |
| --- | --- | --- |
| **자원 반납** | `finally`에서 명시적으로 자원 반납 | 자원을 자동으로 반납 (`AutoCloseable` 지원 자원) |
| **예외 처리** | 예외 발생 시 `catch` 블록에서 처리 | 동일하지만 자원 관리가 자동화됨 |
| **코드 간결성** | 코드가 길어질 수 있음 | 코드가 간결해지고 실수 방지 가능 |
| **자원 관리 실수 가능성** | 자원 반납을 놓칠 가능성이 있음 | 자원이 자동으로 반납되어 실수 가능성 낮음 |

### **5. 언제 `try-with-resources`를 사용하는가?**

- 자원 반납이 필요할 때 (파일 핸들링, 데이터베이스 연결, 네트워크 소켓 등).
- 자원이 `AutoCloseable` 또는 `Closeable` 인터페이스를 구현했을 때.
- 자원 반납이 자동으로 처리되는 구조가 필요할 때.

### **6. 자주 사용하는 자원**

파일 입출력, 네트워크 통신, 데이터베이스 연결 등의 실제 개발에서 자주 사용하는 자원

- **파일**: `FileInputStream`, `FileOutputStream`, `BufferedReader`
- **네트워크**: `Socket`, `ServerSocket`
- **데이터베이스**: `Connection`, `Statement`, `ResultSet`
- **Java 7 이후**: 많은 표준 자원이 `AutoCloseable`을 지원.

### **7. 요약**

- **try-catch-finally**: 자원 관리를 명시적으로 해야 할 때 사용. 단, 자원 반납의 실수를 방지하려면 주의가 필요.
- **try-with-resources**: 자원 반납이 필요할 때 자동화된 자원 관리를 제공하여 실수를 줄이고 코드 가독성을 높여줌.