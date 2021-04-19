
# 1. 프로젝트 생성
- http://start.spring.io 접속
- Group: `org.myproject.hello`
- Artifact: `hello-boot`
- name: `hello-boot`
- Package Name: `org.myproject.hello`
- packaging: jar
- java: 8
- Dependency: Spring Data JPA, Spring Data JDBC, Spring Web, H2 Database, lombok 추가
- Generate 후 다운로드된 zip파일 압축 해제 -> IDE 에서 open

## 1.1 Hello Controller 생성
```java
package org.myproject.hello.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String sayHello(){
        return "Hello~ Developers!";
    }

}
```

## 1.2 Junit Test
### 1.2.1 Springboot 2.1.5 ~ 2.3.x JUnit 사용시  
- SpringTest에서 Junit4는 exclude하여 Junit5와 함께 빌드배포  
    ```xml
    <!-- junit4를 사용할 경우 org.junit.vintage 관련 exclusion 삭제 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
    <!--		<exclusion>-->
    <!--			<groupId>org.junit.vintage</groupId>-->
    <!--			<artifactId>junit-vintage-engine</artifactId>-->
    <!--		</exclusion>-->
        </exclusions>
    </dependency>
    ```
### 1.2.2 Springboot 2.4.x 이상  
- SpringTest에서 Junit5 만 빌드배포  
    ```xml
    <!-- junit4를 사용할 경우 dependency 추가-->
    <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <scope>test</scope>
    </dependency>
    ```
    

# 2. 설정
## 2.1 application.yml
```yml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:hello
    username: sa
    password:
  h2:
    console:
      enabled: true
      path: /h2-console
  jpa:
    database: h2
    generate-ddl: true
    hibernate:
      ddl-auto: update

logging:
  level:
    root: INFO
    org.myproject.hello: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
    org.springframework.jdbc.core.JdbcTemplate: DEBUG
    org.springframework.jdbc.core.StatementCreatorUtils: TRACE
```
## 2.2 schema.sql
```sql
DROP TABLE owners IF EXISTS;

CREATE TABLE IF NOT EXISTS owners (
  id         BIGINT,
  first_name VARCHAR(30),
  last_name  VARCHAR(30),
  address    VARCHAR(255),
  city       VARCHAR(80),
  telephone  VARCHAR(20)
);

INSERT INTO owners VALUES (1, 'George', 'Franklin', '110 W. Liberty St.', 'Madison', '6085551023');
INSERT INTO owners VALUES (2, 'Betty', 'Davis', '638 Cardinal Ave.', 'Sun Prairie', '6085551749');
INSERT INTO owners VALUES (3, 'Eduardo', 'Rodriquez', '2693 Commerce St.', 'McFarland', '6085558763');
INSERT INTO owners VALUES (4, 'Harold', 'Davis', '563 Friendly St.', 'Windsor', '6085553198');
INSERT INTO owners VALUES (5, 'Peter', 'McTavish', '2387 S. Fair Way', 'Madison', '6085552765');
INSERT INTO owners VALUES (6, 'Jean', 'Coleman', '105 N. Lake St.', 'Monona', '6085552654');
INSERT INTO owners VALUES (7, 'Jeff', 'Black', '1450 Oak Blvd.', 'Monona', '6085555387');
INSERT INTO owners VALUES (8, 'Maria', 'Escobito', '345 Maple St.', 'Madison', '6085557683');
INSERT INTO owners VALUES (9, 'David', 'Schroeder', '2749 Blackhawk Trail', 'Madison', '6085559435');
INSERT INTO owners VALUES (10, 'Carlos', 'Estaban', '2335 Independence La.', 'Waunakee', '6085555487');
```  
## 2.3 실행 후 http://localhost:8080/h2-console 접속


# 3. Domain 생성
```java
package org.myproject.hello.domain;

import lombok.Data;

import javax.persistence.*;

@Entity
@Table(name = "owners")
@Data
public class Owner {

    @Id
    protected Long id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "address")
    private String address;

    @Column(name = "city")
    private String city;

    @Column(name = "telephone")
    private String telephone;
}
```


# 4. Repository 생성
## 4.1 Spring-Data-JPA
### 4.1.1 OwnerJpaRpository
```java
package org.myproject.hello.repository;

import org.myproject.hello.domain.Owner;
import org.springframework.data.repository.CrudRepository;

@Repository
public interface OwnerJpaRepository extends CrudRepository<Owner, Long> {
}
```

### 4.1.2 OwnerJpaRpositoryTest
```java
package org.myproject.hello.repository;

import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.myproject.hello.domain.Owner;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

import static org.junit.Assert.*;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
//@DataJpaTest
public class OwnerJpaRepositoryTest {

    @Resource
    OwnerJpaRepository ownerJpaRepository;

    @Test
    public void findById(){

        Owner owner = ownerJpaRepository.findById(1l).orElse(null);
        log.debug(owner.toString());

        assertEquals("George", owner.getFirstName());
        assertEquals("Franklin", owner.getLastName());
    }
}
```

## 4.2 Spring-JDBC
### 4.2.1 OwnerJdbcRepository
```java
package org.myproject.hello.repository;

import org.myproject.hello.domain.Owner;

public interface OwnerJdbcRepository {
    Owner findById(Long id);
}

```

### 4.2.2 OwnerJdbcRepositoryImpl
```java
package org.myproject.hello.repository;

import org.myproject.hello.domain.Owner;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import javax.annotation.Resource;

@Repository
public class OwnerJdbcRepositoryImpl implements OwnerJdbcRepository {

    @Resource
    JdbcTemplate jdbcTemplate;

    @Override
    public Owner findById(Long id) {
        try{
            return jdbcTemplate.queryForObject("SELECT id, first_name, last_name, address, city, telephone FROM owners WHERE id= ?",
                    BeanPropertyRowMapper.newInstance(Owner.class), id);
        } catch (EmptyResultDataAccessException ex) {
            return null;
        }
    }
}
```

### 4.2.3 OwnerJdbcRepositoryTest
```java
package org.myproject.hello.repository;

import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.myproject.hello.domain.Owner;
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

import static org.junit.Assert.*;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
//@JdbcTest
//@Import({OwnerJdbcRepositoryImpl.class})
public class OwnerJdbcRepositoryTest {

    @Resource
    OwnerJdbcRepository ownerJdbcRepository;

    @Test
    public void findById() {
        Owner owner = ownerJdbcRepository.findById(1l);

        log.debug(owner.toString());
        assertEquals("George", owner.getFirstName());
        assertEquals("Franklin", owner.getLastName());
    }
}
```


# 5 Service 생성
## 5.1 OwnerService 인터페이스
```java
package org.myproject.hello.service;

import org.myproject.hello.domain.Owner;

public interface OwnerService {

    Owner findById(Long id);
}

```

## 5.2 OwnerServiceImpl 구현클래스
```java
package org.myproject.hello.service;

import org.myproject.hello.domain.Owner;
import org.myproject.hello.repository.OwnerJdbcRepository;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class OwnerServiceImpl implements OwnerService{

    @Resource
    private OwnerJdbcRepository ownerJdbcRepository;

    @Override
    public Owner findById(Long id) {
        return ownerJdbcRepository.findById(id);
    }
}
```

## 5.3 OwnerServiceTest
```java
package org.myproject.hello.service;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.myproject.hello.domain.Owner;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
public class OwnerServiceTest {

    @Resource
    OwnerService ownerService;

    @Test
    public void findById(){
        Owner owner = ownerService.findById(1l);

        assertEquals("George", owner.getFirstName());
        assertEquals("Franklin", owner.getLastName());
    }
}
```

## 5.4 Mock을 활용한 OwnerServiceMockTest
```java
package org.myproject.hello.service;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.myproject.hello.domain.Owner;
import org.myproject.hello.repository.OwnerJdbcRepository;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

@RunWith(SpringRunner.class)
@SpringBootTest
public class OwnerServiceTest {

    @Resource
    OwnerService ownerService;

    @MockBean
    OwnerJdbcRepository ownerJdbcRepository;

    @Test
    public void findById(){
        Owner expectedOwner = new Owner();
        expectedOwner.setId(11l);
        expectedOwner.setFirstName("Doni");
        expectedOwner.setLastName("Lee");
        expectedOwner.setAddress("Rodeo Street");
        expectedOwner.setCity("Seoul");
        expectedOwner.setTelephone("01012345678");

        when(ownerJdbcRepository.findById(11l)).thenReturn(expectedOwner);

        Owner owner = ownerService.findById(11l);

        assertEquals(expectedOwner.getFirstName(), owner.getFirstName());
        assertEquals(expectedOwner.getLastName(), owner.getLastName());

        verify(ownerJdbcRepository, times(1)).findById(11l);
    }
}
```


# 6. Controller 생성
## 6.1 OwnerController
```java
package org.myproject.hello.controller;

import lombok.extern.slf4j.Slf4j;
import org.myproject.hello.domain.Owner;
import org.myproject.hello.service.OwnerService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@Slf4j
@RestController
@RequestMapping("/owners")
public class OwnerController {

    @Resource
    private OwnerService ownerService;

    @GetMapping("/{ownerId}")
    public ResponseEntity<Owner> findById(@PathVariable("ownerId") Long ownerId){
        log.debug("findById: {}", ownerId);
        Owner owner = ownerService.findById(ownerId);
        if(owner == null){
            return ResponseEntity.notFound().build();
        }else{
            return ResponseEntity.ok(owner);
        }
    }
}

```

## 6.2 OwnerControllerTest
```java
package org.myproject.hello.controller;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import javax.annotation.Resource;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class OwnerControllerTest {

    @Resource
    MockMvc mockMvc;

    @Test
    public void findById() throws Exception {
        mockMvc.perform(get("/owners/{ownerId}", 1))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.firstName").value("George"))
                .andExpect(jsonPath("$.firstName").value("Franklin"))
                ;
    }
}
```

## 6.2 Mock을 활용한 OwnerControllerMockTest
```java
package org.myproject.hello.controller;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.myproject.hello.domain.Owner;
import org.myproject.hello.service.OwnerService;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import javax.annotation.Resource;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class OwnerControllerMockTest {

    @Resource
    MockMvc mockMvc;

    @MockBean
    OwnerService ownerService;

    @Test
    public void findById() throws Exception {
        Owner expectedOwner = new Owner();
        expectedOwner.setId(11l);
        expectedOwner.setFirstName("Doni");
        expectedOwner.setLastName("Lee");
        expectedOwner.setAddress("Rodeo Street");
        expectedOwner.setCity("Seoul");
        expectedOwner.setTelephone("01012345678");

        when(ownerService.findById(11l)).thenReturn(expectedOwner);

        mockMvc.perform(get("/owners/{ownerId}", 11))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.firstName").value(expectedOwner.getFirstName()))
                .andExpect(jsonPath("$.lastName").value(expectedOwner.getLastName()))
                ;
    }
}
```

## 6.2 Test를 위한 DB생성, API Test 
- test/resources 폴더 생성
- 우클릭 -> mark directory as -> Test Resources Root
- 
    ```sql
    # test_schema.sql
    DROP TABLE owners IF EXISTS;

    CREATE TABLE IF NOT EXISTS owners (
    id         BIGINT,
    first_name VARCHAR(30),
    last_name  VARCHAR(30),
    address    VARCHAR(255),
    city       VARCHAR(80),
    telephone  VARCHAR(20)
    );

    INSERT INTO owners VALUES (1, 'Doni', 'Lee', 'Rodeo Street', 'Seoul', '01012345678');
    ```
- 
    ```java
    package org.myproject.hello.controller;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.TestPropertySource;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;

    import javax.annotation.Resource;

    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

    @RunWith(SpringRunner.class)
    @SpringBootTest
    @AutoConfigureMockMvc
    @TestPropertySource(properties = {
            "spring.datasource.schema=classpath:test_schema.sql",
            "spring.datasource.url=jdbc:h2:mem:"
    })
    public class OwnerControllerTestWithTestDB {

        @Resource
        MockMvc mockMvc;

        @Test
        public void findById() throws Exception {
            mockMvc.perform(get("/owners/{ownerId}", 1))
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.firstName").value("Doni"))
                    .andExpect(jsonPath("$.lastName").value("Lee"))
                    ;
        }
    }
    ```

    
# 7 Swagger Document 설정 (택1)
## 7.1 Springfox (https://github.com/springfox/springfox)  
- 
    ```xml
    <!-- pom.xml -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>
    ```
- 
    ```java
    package org.myproject.hello.config;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import springfox.documentation.builders.PathSelectors;
    import springfox.documentation.builders.RequestHandlerSelectors;
    import springfox.documentation.spi.DocumentationType;
    import springfox.documentation.spring.web.plugins.Docket;

    @Configuration
    public class SwaggerConfig {
    
        @Bean
        public Docket api() {
            return new Docket(DocumentationType.SWAGGER_2)
                    .select()
                    .apis(RequestHandlerSelectors.basePackage("org.myproject.hello.controller"))
                    .paths(PathSelectors.any())
                    .build();
        }
    }
    ```
- http://localhost:8080/swagger-ui/index.html 접속

## 7.2 Springdoc-openapi (https://github.com/springdoc/springdoc-openapi)  
- 
    ```xml
    <!-- pom.xml -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-ui</artifactId>
        <version>1.5.7</version>
    </dependency>
    ```
- 
    ```yml
    # application.yml

    springdoc:
      swagger-ui:
        path: /swagger-ui.html
      api-docs:
        path: /api-docs
      packages-to-scan: com.sds.cleancode.hello.controller
    ```
- http://localhost:8080/swagger-ui.html 접속
