# **상품 - 주문 API 개발로 알아보는 TDD**

---

- 출처 : [`inflearn 강의`](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%8B%A4%EC%A0%84-%EC%83%81%ED%92%88%EC%A3%BC%EB%AC%B8-tdd/dashboard "실전! 스프링부트 상품-주문 API 개발로 알아보는 TDD")
- Test 코드 작성 후 클래스를 상위 클래스로 올린 후 Spring Boot Bean으로 확인 하는 순차 Test 코드 작성

---

## **Project 생성**

![tdd1.png](..%2Fimg%2Ftdd1.png)
- dependency - jpa, h2, lombok, web
- ```
   implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
   implementation 'org.springframework.boot:spring-boot-starter-web'
   compileOnly 'org.projectlombok:lombok'
   runtimeOnly 'com.h2database:h2'
   annotationProcessor 'org.projectlombok:lombok'
   testImplementation 'org.springframework.boot:spring-boot-starter-test'
   testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
  ```

## **1. Test 코드 작성**

### **1-1. Boot Project 로드 테스트**
- contextLoads 메소드 좌측의 실행 버튼을 이용하여 실행 후 테스트 결과 확인


![tdd2.png](..%2Fimg%2Ftdd2.png)

### **1-2. TDD 코드 작성 1차**
- product 패키지 생성
- ProductServiceTest 클래스 생성

```java
package com.example.tdd.product;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.util.Assert;

public class ProductServiceTest {

  private ProductService productService;

  @BeforeEach
  void setUp() {
    productService = new ProductService();
  }

  @Test
  void 상품등록() {
    final String name = "상품명";
    final int price = 1000;
    final DiscountPolicy discountPolicy = DiscountPolicy.NONE;

    final AddProductRequest request = new AddProductRequest(name, price, discountPolicy);
    productService.addProduct(request);
  }

  private class ProductService {
    public void addProduct(AddProductRequest request) {
    }
  }

  private record AddProductRequest(String name, int price, DiscountPolicy discountPolicy) {

    private AddProductRequest(final String name, final int price, final DiscountPolicy discountPolicy) {
      this.name = name;
      this.price = price;
      this.discountPolicy = discountPolicy;
      Assert.hasText(name, "상품명은 필수입니다.");
      Assert.isTrue(price > 0, "상품 가격은 0보다 커야합니다.");
      Assert.notNull(discountPolicy, "할인 정책은 필수입니다.");
    }
  }

  private enum DiscountPolicy {
    NONE
  }

}
```

### **1-3. TDD 코드 작성 2차**
- @Test 확인

```java
package com.example.tdd.product;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.util.Assert;

import java.util.HashMap;
import java.util.Map;

public class ProductServiceTest {

    private ProductService productService;
    private ProductPort productPort;
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository = new ProductRepository();
        productPort = new ProductAdapter(productRepository);
        productService = new ProductService(productPort);
    }

    @Test
    void 상품등록() {
        final String name = "상품명";
        final int price = 1000;
        final DiscountPolicy discountPolicy = DiscountPolicy.NONE;

        final AddProductRequest request = new AddProductRequest(name, price, discountPolicy);
        productService.addProduct(request);
    }

    private class ProductService {
        private final ProductPort productPort;

        private ProductService(ProductPort productPort) {
            this.productPort = productPort;
        }

        public void addProduct(AddProductRequest request) {
            final Product product = new Product(request.name(), request.price(), request.discountPolicy());

            productPort.save(product);
        }
    }

    private record AddProductRequest(String name, int price, DiscountPolicy discountPolicy) {

            private AddProductRequest(final String name, final int price, final DiscountPolicy discountPolicy) {
                this.name = name;
                this.price = price;
                this.discountPolicy = discountPolicy;
                Assert.hasText(name, "상품명은 필수입니다.");
                Assert.isTrue(price > 0, "상품 가격은 0보다 커야합니다.");
                Assert.notNull(discountPolicy, "할인 정책은 필수입니다.");
            }
        }

    private enum DiscountPolicy {
        NONE
    }

    private class Product {
        private Long id;
        private final String name;
        private final int price;
        private final DiscountPolicy discountPolicy;

        public Product(String name, int price, DiscountPolicy discountPolicy) {
            Assert.hasText(name, "상품명은 필수입니다.");
            Assert.isTrue(price > 0, "상품 가격은 0보다 커야합니다.");
            Assert.notNull(discountPolicy, "할인 정책은 필수입니다.");
            this.name = name;
            this.price = price;
            this.discountPolicy = discountPolicy;
        }

        public void assignId(Long id) {
            this.id = id;
        }

        public Long getId() {
            return id;
        }
    }

    private interface ProductPort {
        void save(Product product);
    }

    private class ProductAdapter implements ProductPort {
        private final ProductRepository productRepository;

        private ProductAdapter(ProductRepository productRepository) {
            this.productRepository = productRepository;
        }

        @Override
        public void save(Product product) {
            productRepository.save(product);
        }
    }

    private class ProductRepository {
        private Map<Long, Product> persistence = new HashMap<>();
        private Long sequence = 0L;

        public void save(Product product) {
            product.assignId(++sequence);
            persistence.put(product.getId(), product);
        }
    }
}
```

### **1-4. 내부클래스를 상위 클래스로 이동 / 단축키: F6**
- Test 클래스 제외 모두 상위로 이동
![tdd3.png](..%2Fimg%2Ftdd3.png)

### **1-5. Test 확인**

### **1-6. test 폴더의 클래스들을 main으로 이동**
![tdd4.png](..%2Fimg%2Ftdd4.png)
![tdd5.png](..%2Fimg%2Ftdd5.png)




## **2. Spring Boot Bean으로 테스트 전환하기**

### **2-1. 어노테이션 추가**
- @Component
  - ProductService.java
  - ProductAdapter.java
- @Repository
  - ProductRepository.java

```java
package com.example.tdd.product;

import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

@Component
class ProductService {
    private final ProductPort productPort;

    ProductService(ProductPort productPort) {
        this.productPort = productPort;
    }

    @Transactional
    public void addProduct(AddProductRequest request) {
        final Product product = new Product(request.name(), request.price(), request.discountPolicy());

        productPort.save(product);
    }
}
```

```java
package com.example.tdd.product;

import org.springframework.stereotype.Component;

@Component
class ProductAdapter implements ProductPort {
    private final ProductRepository productRepository;

    ProductAdapter(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public void save(Product product) {
        productRepository.save(product);
    }
}
```

```java
package com.example.tdd.product;

import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;

@Repository
class ProductRepository {
    private Map<Long, Product> persistence = new HashMap<>();
    private Long sequence = 0L;

    public void save(Product product) {
        product.assignId(++sequence);
        persistence.put(product.getId(), product);
    }
}
```

### **2-2. ProductServiceTest 클래스 @Autowired로 변환**
```java
package com.example.tdd.product;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class ProductServiceTest {

    @Autowired
    private ProductService productService;

    @Test
    void 상품등록() {
        final AddProductRequest request = 상품등록요청_생성();

        productService.addProduct(request);
    }

    private static AddProductRequest 상품등록요청_생성() {
        final String name = "상품명";
        final int price = 1000;
        final DiscountPolicy discountPolicy = DiscountPolicy.NONE;

        return new AddProductRequest(name, price, discountPolicy);
    }

}
```

## **3. API 테스트로 전환**

### **3-1. gradle 의존성 추가**
```shell
testImplementation group: 'io.rest-assured', name: 'rest-assured', version: '4.4.0'
```
### **3-2. ApiTest 클래스 추가**
```java
package com.example.tdd;

import io.restassured.RestAssured;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ApiTest {

  @LocalServerPort
  private int port;

  @BeforeEach
  void setUp() {
    RestAssured.port = port;
  }
}
```

### **3-3. ProductServiceTest -> ProductApiTest로 Rename 및 API 요청**
```java
package com.example.tdd.product;

import com.example.tdd.ApiTest;
import io.restassured.RestAssured;
import io.restassured.response.ExtractableResponse;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;

import static org.assertj.core.api.Assertions.assertThat;

public class ProductApiTest extends ApiTest {

  @Test
  void 상품등록() {
    final AddProductRequest request = 상품등록요청_생성();

    final ExtractableResponse<Response> response = 
            RestAssured.given().log().all()         // 요청 로그를 남김
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .body(request)
            .when()
            .post("/products")
            .then()
            .log().all().extract();

    assertThat(response.statusCode()).isEqualTo(HttpStatus.CREATED.value());
  }

  private static AddProductRequest 상품등록요청_생성() {
    final String name = "상품명";
    final int price = 1000;
    final DiscountPolicy discountPolicy = DiscountPolicy.NONE;

    return new AddProductRequest(name, price, discountPolicy);
  }

}
```

### **3-3. ProductService Controller 매핑**

```java
package com.example.tdd.product;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/products")
class ProductService {
    private final ProductPort productPort;

    ProductService(ProductPort productPort) {
        this.productPort = productPort;
    }

    @PostMapping
    public ResponseEntity addProduct(@RequestBody final AddProductRequest request) {
        final Product product = new Product(request.name(), request.price(), request.discountPolicy());

        productPort.save(product);

        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

### **3-4. Test 상품등록 실행**
```shell
Request method:	POST
Request URI:	http://localhost:55287/products
Proxy:			<none>
Request params:	<none>
Query params:	<none>
Form params:	<none>
Path params:	<none>
Headers:		Accept=*/*
				Content-Type=application/json
Cookies:		<none>
Multiparts:		<none>
Body:
{
    "name": "상품명",
    "price": 1000,
    "discountPolicy": "NONE"
}
2024-11-03 00:13:10.703  INFO 22664 --- [o-auto-1-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2024-11-03 00:13:10.704  INFO 22664 --- [o-auto-1-exec-2] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2024-11-03 00:13:10.705  INFO 22664 --- [o-auto-1-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
HTTP/1.1 201 
Content-Length: 0
Date: Sat, 02 Nov 2024 15:13:10 GMT
Keep-Alive: timeout=60
Connection: keep-alive

```

### **3-5. Test 성공 후 리팩토링**
```java
package com.example.tdd.product;

import com.example.tdd.ApiTest;
import io.restassured.RestAssured;
import io.restassured.response.ExtractableResponse;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;

import static org.assertj.core.api.Assertions.assertThat;

public class ProductApiTest extends ApiTest {

//    @Autowired
//    private ProductService productService;

    @Test
    void 상품등록() {
        final var request = 상품등록요청_생성();
//        productService.addProduct(request);

        final var response = 상품등록요청(request);

        assertThat(response.statusCode()).isEqualTo(HttpStatus.CREATED.value());
    }

    private static ExtractableResponse<Response> 상품등록요청(final AddProductRequest request) {
        return RestAssured.given().log().all()         // 요청 로그를 남김
                .contentType(MediaType.APPLICATION_JSON_VALUE)
                .body(request)
                .when()
                .post("/products")
                .then()
                .log().all().extract();
    }

    private static AddProductRequest 상품등록요청_생성() {
        final String name = "상품명";
        final int price = 1000;
        final DiscountPolicy discountPolicy = DiscountPolicy.NONE;

        return new AddProductRequest(name, price, discountPolicy);
    }

}
```

## **3. JPA 사용으로 전환**
