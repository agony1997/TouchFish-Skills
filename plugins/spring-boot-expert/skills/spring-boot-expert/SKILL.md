---
name: spring-boot-expert
description: "Spring Boot 2.x/3.x 全方位開發專家。專精於 Spring Boot 應用開發、自動配置、Starter、Actuator、Profile、Properties 管理、Spring Boot 測試、最佳實踐。關鍵字: spring boot, spring, java, springboot, spring framework, 微服務, rest api, spring boot 3"
---

# Spring Boot Expert

You are a Spring Boot Expert specializing in Spring Boot 2.x and 3.x application development, configuration, best practices, and production deployment.

## Overview

As a Spring Boot Expert, you provide:
- Spring Boot application architecture and design
- Auto-configuration and custom starters
- RESTful API development
- Spring Boot Actuator monitoring
- Configuration management (Properties, YAML, Profiles)
- Spring Boot testing strategies
- Production-ready best practices
- Migration guidance (Boot 2.x → 3.x)

## When to use this skill

Activate this skill when users:
- Develop Spring Boot applications (關鍵字: "spring boot", "springboot", "spring應用")
- Ask about Spring configuration (關鍵字: "spring配置", "application.yml", "properties")
- Need REST API development (關鍵字: "rest api", "controller", "restcontroller")
- Want monitoring and health checks (關鍵字: "actuator", "health check", "監控")
- Ask about Spring Boot best practices

## Core Knowledge Areas

### 1. Spring Boot Fundamentals

**Project Structure:**
```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example.myapp/
│   │   │       ├── MyApplication.java          # Main application class
│   │   │       ├── config/                     # Configuration classes
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── DatabaseConfig.java
│   │   │       ├── controller/                 # REST controllers
│   │   │       │   └── UserController.java
│   │   │       ├── service/                    # Business logic
│   │   │       │   ├── UserService.java
│   │   │       │   └── impl/
│   │   │       │       └── UserServiceImpl.java
│   │   │       ├── repository/                 # Data access
│   │   │       │   └── UserRepository.java
│   │   │       ├── model/                      # Domain models
│   │   │       │   ├── entity/
│   │   │       │   │   └── User.java
│   │   │       │   └── dto/
│   │   │       │       ├── UserDTO.java
│   │   │       │       └── UserCreateRequest.java
│   │   │       ├── exception/                  # Exception handling
│   │   │       │   ├── GlobalExceptionHandler.java
│   │   │       │   └── ResourceNotFoundException.java
│   │   │       └── util/                       # Utilities
│   │   └── resources/
│   │       ├── application.yml                 # Main configuration
│   │       ├── application-dev.yml             # Dev profile
│   │       ├── application-prod.yml            # Production profile
│   │       ├── static/                         # Static resources
│   │       ├── templates/                      # Templates (if using Thymeleaf)
│   │       └── db/
│   │           └── migration/                  # Flyway/Liquibase migrations
│   └── test/
│       └── java/
│           └── com.example.myapp/
│               ├── controller/                  # Controller tests
│               ├── service/                     # Service tests
│               └── repository/                  # Repository tests
├── pom.xml                                     # Maven dependencies
└── README.md
```

**Main Application Class:**
```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // Combines @Configuration, @EnableAutoConfiguration, @ComponentScan
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**Maven Dependencies (pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.1</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-app</artifactId>
    <version>1.0.0</version>
    <name>My Spring Boot App</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Web Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Lombok (Optional but recommended) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2. RESTful API Development

**Controller Example:**
```java
package com.example.myapp.controller;

import com.example.myapp.model.dto.UserDTO;
import com.example.myapp.model.dto.UserCreateRequest;
import com.example.myapp.service.UserService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<Page<UserDTO>> getAllUsers(Pageable pageable) {
        Page<UserDTO> users = userService.findAll(pageable);
        return ResponseEntity.ok(users);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUserById(@PathVariable Long id) {
        UserDTO user = userService.findById(id);
        return ResponseEntity.ok(user);
    }

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserCreateRequest request) {
        UserDTO user = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserCreateRequest request) {
        UserDTO user = userService.update(id, request);
        return ResponseEntity.ok(user);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/search")
    public ResponseEntity<Page<UserDTO>> searchUsers(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String email,
            Pageable pageable) {
        Page<UserDTO> users = userService.search(name, email, pageable);
        return ResponseEntity.ok(users);
    }
}
```

**Service Layer:**
```java
package com.example.myapp.service.impl;

import com.example.myapp.exception.ResourceNotFoundException;
import com.example.myapp.model.dto.UserDTO;
import com.example.myapp.model.dto.UserCreateRequest;
import com.example.myapp.model.entity.User;
import com.example.myapp.repository.UserRepository;
import com.example.myapp.service.UserService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    @Override
    public Page<UserDTO> findAll(Pageable pageable) {
        log.debug("Finding all users with pagination: {}", pageable);
        return userRepository.findAll(pageable)
                .map(this::convertToDTO);
    }

    @Override
    public UserDTO findById(Long id) {
        log.debug("Finding user by id: {}", id);
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        return convertToDTO(user);
    }

    @Override
    @Transactional
    public UserDTO create(UserCreateRequest request) {
        log.info("Creating new user: {}", request.getEmail());

        // Check if email already exists
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new IllegalArgumentException("Email already exists: " + request.getEmail());
        }

        User user = User.builder()
                .name(request.getName())
                .email(request.getEmail())
                .build();

        User savedUser = userRepository.save(user);
        log.info("User created successfully with id: {}", savedUser.getId());

        return convertToDTO(savedUser);
    }

    @Override
    @Transactional
    public UserDTO update(Long id, UserCreateRequest request) {
        log.info("Updating user with id: {}", id);

        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));

        user.setName(request.getName());
        user.setEmail(request.getEmail());

        User updatedUser = userRepository.save(user);
        log.info("User updated successfully: {}", id);

        return convertToDTO(updatedUser);
    }

    @Override
    @Transactional
    public void delete(Long id) {
        log.info("Deleting user with id: {}", id);

        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User not found with id: " + id);
        }

        userRepository.deleteById(id);
        log.info("User deleted successfully: {}", id);
    }

    private UserDTO convertToDTO(User user) {
        return UserDTO.builder()
                .id(user.getId())
                .name(user.getName())
                .email(user.getEmail())
                .createdAt(user.getCreatedAt())
                .build();
    }
}
```

**Entity:**
```java
package com.example.myapp.model.entity;

import jakarta.persistence.*;
import lombok.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true, length = 100)
    private String email;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;
}
```

**DTO:**
```java
package com.example.myapp.model.dto;

import lombok.*;

import java.time.LocalDateTime;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    private LocalDateTime createdAt;
}
```

**Request DTO:**
```java
package com.example.myapp.model.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserCreateRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    @Size(max = 100, message = "Email must not exceed 100 characters")
    private String email;
}
```

**Repository:**
```java
package com.example.myapp.repository;

import com.example.myapp.model.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

    boolean existsByEmail(String email);
}
```

### 3. Exception Handling

**Global Exception Handler:**
```java
package com.example.myapp.exception;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        log.error("Resource not found: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.NOT_FOUND.value())
                .error("Not Found")
                .message(ex.getMessage())
                .build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        log.error("Illegal argument: {}", ex.getMessage());

        ErrorResponse error = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Bad Request")
                .message(ex.getMessage())
                .build();

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException ex) {
        log.error("Validation error: {}", ex.getMessage());

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        ErrorResponse error = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Validation Failed")
                .message("Input validation failed")
                .validationErrors(errors)
                .build();

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex) {
        log.error("Unexpected error: ", ex);

        ErrorResponse error = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .error("Internal Server Error")
                .message("An unexpected error occurred")
                .build();

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

**Error Response:**
```java
package com.example.myapp.exception;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.*;

import java.time.LocalDateTime;
import java.util.Map;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private Map<String, String> validationErrors;
}
```

### 4. Configuration Management

**application.yml:**
```yaml
spring:
  application:
    name: my-spring-boot-app

  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:password}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQLDialect

  flyway:
    enabled: true
    baseline-on-migrate: true

server:
  port: 8080
  servlet:
    context-path: /
  error:
    include-message: always
    include-binding-errors: always

logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
    org.hibernate.SQL: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
    max-size: 10MB
    max-history: 30

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: true
```

**application-dev.yml:**
```yaml
spring:
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update

logging:
  level:
    root: DEBUG
    com.example.myapp: TRACE

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

**application-prod.yml:**
```yaml
spring:
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    root: WARN
    com.example.myapp: INFO

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

**Configuration Class:**
```java
package com.example.myapp.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing
public class JpaConfig {
    // JPA Auditing configuration
}
```

### 5. Spring Boot Actuator

**Actuator Endpoints:**
```
/actuator/health        - Health status
/actuator/info          - Application info
/actuator/metrics       - Application metrics
/actuator/prometheus    - Prometheus metrics
/actuator/env           - Environment properties
/actuator/loggers       - Logging configuration
/actuator/beans         - Spring beans
/actuator/mappings      - Request mappings
```

**Custom Health Indicator:**
```java
package com.example.myapp.config;

import lombok.RequiredArgsConstructor;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

import javax.sql.DataSource;
import java.sql.Connection;

@Component
@RequiredArgsConstructor
public class DatabaseHealthIndicator implements HealthIndicator {

    private final DataSource dataSource;

    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(1)) {
                return Health.up()
                        .withDetail("database", "MySQL")
                        .withDetail("status", "Connected")
                        .build();
            } else {
                return Health.down()
                        .withDetail("database", "MySQL")
                        .withDetail("status", "Connection invalid")
                        .build();
            }
        } catch (Exception e) {
            return Health.down()
                    .withDetail("database", "MySQL")
                    .withDetail("error", e.getMessage())
                    .build();
        }
    }
}
```

**Application Info:**
```yaml
# application.yml
info:
  app:
    name: My Spring Boot Application
    description: Enterprise REST API
    version: 1.0.0
  build:
    artifact: ${project.artifactId}
    version: ${project.version}
```

### 6. Testing

**Controller Test:**
```java
package com.example.myapp.controller;

import com.example.myapp.model.dto.UserDTO;
import com.example.myapp.model.dto.UserCreateRequest;
import com.example.myapp.service.UserService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void shouldCreateUser() throws Exception {
        UserCreateRequest request = UserCreateRequest.builder()
                .name("John Doe")
                .email("john@example.com")
                .build();

        UserDTO response = UserDTO.builder()
                .id(1L)
                .name("John Doe")
                .email("john@example.com")
                .build();

        when(userService.create(any(UserCreateRequest.class))).thenReturn(response);

        mockMvc.perform(post("/api/v1/users")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

**Service Test:**
```java
package com.example.myapp.service;

import com.example.myapp.exception.ResourceNotFoundException;
import com.example.myapp.model.dto.UserCreateRequest;
import com.example.myapp.model.dto.UserDTO;
import com.example.myapp.model.entity.User;
import com.example.myapp.repository.UserRepository;
import com.example.myapp.service.impl.UserServiceImpl;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceImplTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserServiceImpl userService;

    @Test
    void shouldCreateUser() {
        UserCreateRequest request = UserCreateRequest.builder()
                .name("John Doe")
                .email("john@example.com")
                .build();

        User savedUser = User.builder()
                .id(1L)
                .name("John Doe")
                .email("john@example.com")
                .build();

        when(userRepository.existsByEmail(anyString())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(savedUser);

        UserDTO result = userService.create(request);

        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("John Doe", result.getName());
        assertEquals("john@example.com", result.getEmail());

        verify(userRepository).save(any(User.class));
    }

    @Test
    void shouldThrowExceptionWhenUserNotFound() {
        when(userRepository.findById(anyLong())).thenReturn(Optional.empty());

        assertThrows(ResourceNotFoundException.class, () -> userService.findById(1L));
    }
}
```

**Integration Test:**
```java
package com.example.myapp;

import com.example.myapp.model.dto.UserCreateRequest;
import com.example.myapp.model.entity.User;
import com.example.myapp.repository.UserRepository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class UserIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateAndRetrieveUser() throws Exception {
        UserCreateRequest request = UserCreateRequest.builder()
                .name("John Doe")
                .email("john@example.com")
                .build();

        String responseContent = mockMvc.perform(post("/api/v1/users")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andReturn().getResponse().getContentAsString();

        // Extract ID from response (simplified)
        Long userId = 1L;

        mockMvc.perform(get("/api/v1/users/" + userId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

## Best Practices

### 1. Layered Architecture

```
Controller Layer (REST API) → Service Layer (Business Logic) → Repository Layer (Data Access)
```

- **Controllers**: Handle HTTP requests, validation, response formatting
- **Services**: Business logic, transaction management
- **Repositories**: Database operations

### 2. Constructor Injection

**Prefer:**
```java
@Service
@RequiredArgsConstructor  // Lombok generates constructor
public class UserService {
    private final UserRepository userRepository;
}
```

**Avoid:**
```java
@Service
public class UserService {
    @Autowired  // Field injection - harder to test
    private UserRepository userRepository;
}
```

### 3. Use DTOs

- Never expose entities directly in REST APIs
- Use DTOs for request/response
- Validate DTOs with Bean Validation

### 4. Proper Exception Handling

- Use `@RestControllerAdvice` for global exception handling
- Create custom exceptions for business logic
- Return meaningful error messages

### 5. Transaction Management

```java
@Service
@Transactional(readOnly = true)  // Default for all methods
public class UserService {

    @Transactional  // Override for write operations
    public User create(UserCreateRequest request) {
        // Write operation
    }
}
```

### 6. Logging

```java
@Service
@Slf4j  // Lombok generates logger
public class UserService {

    public User findById(Long id) {
        log.debug("Finding user by id: {}", id);
        // ...
        log.info("User found: {}", user.getEmail());
    }
}
```

### 7. Configuration Profiles

- Use profiles for different environments (dev, test, prod)
- Externalize configuration
- Use environment variables for sensitive data

## Common Patterns

### Pattern 1: Repository Pattern

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameContaining(String name);
    Optional<User> findByEmail(String email);

    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findRecentUsers(@Param("date") LocalDateTime date);
}
```

### Pattern 2: Service Interface Pattern

```java
public interface UserService {
    UserDTO findById(Long id);
    Page<UserDTO> findAll(Pageable pageable);
    UserDTO create(UserCreateRequest request);
    UserDTO update(Long id, UserCreateRequest request);
    void delete(Long id);
}
```

### Pattern 3: Builder Pattern (Lombok)

```java
User user = User.builder()
    .name("John")
    .email("john@example.com")
    .build();
```

## Quick Reference

### Common Annotations

```java
@SpringBootApplication  // Main application class
@RestController        // REST controller
@Service              // Service layer
@Repository           // Data access layer
@Configuration        // Configuration class
@Component            // Generic Spring component

@GetMapping           // HTTP GET
@PostMapping          // HTTP POST
@PutMapping           // HTTP PUT
@DeleteMapping        // HTTP DELETE
@RequestMapping       // Base URL mapping

@PathVariable         // URL path parameter
@RequestParam         // Query parameter
@RequestBody          // Request body
@Valid                // Trigger validation

@Transactional        // Transaction management
@Autowired            // Dependency injection (avoid)
@RequiredArgsConstructor  // Lombok constructor injection
```

### Maven Commands

```bash
mvn clean install         # Build project
mvn spring-boot:run      # Run application
mvn test                 # Run tests
mvn package              # Create JAR
```

---

**Remember:** Spring Boot favors convention over configuration. Follow Spring Boot conventions and best practices for maintainable applications.
