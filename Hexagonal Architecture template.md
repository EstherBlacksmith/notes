
# Hexagonal Architecture Template for CRUD with Database

## Overview

Hexagonal Architecture (Ports and Adapters) organizes applications into distinct layers with clear boundaries. The core principle is separating **domain logic** from **infrastructure concerns**.

---

## Architecture Diagram

 
```  
┌─────────────────────────────────────────────────────────────────────────────┐
│                           EXTERNAL WORLD                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   REST API  │  │  Web UI     │  │  Mobile App │  │  External Services  │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
└─────────┼────────────────┼────────────────┼────────────────────┼────────────┘
          │                │                │                    │
          └────────────────┴───────┌────────┴────────────────────┘
                                   │
┌──────────────────────────────────┼─────────────────────────────────────────┐
│                    ADAPTERS LAYER (OUTBOUND/Inbound)                       │
│  ┌──────────────────────────┐  ┌─────────────────────────────────────────┐ │
│  │   INBOUND ADAPTER        │  │           OUTBOUND ADAPTERS             │ │
│  │   (Driving Adapter)      │  │          (Driven Adapters)              │ │
│  │  ┌────────────────────┐  │  │  ┌─────────────┐  ┌─────────────────┐   │ │
│  │  │  REST Controller   │◄─┼──┼─►│  Database   │  │   Message Queue │   │ │
│  │  └────────────────────┘  │  │  │  Repository │  │    Publisher    │   │ │
│  └──────────────────────────┘  │  └─────────────┘  └─────────────────┘   │ │
└────────────────────────────────┼───────────────────────────────────────────┘
                                 │
                        ┌────────▼────────┐
                        │      PORTS      │
                        │   (Interfaces)  │
                        │  ┌───────────┐  │
                        │  │  UserPort │  │  ◄── Inbound interface
                        │  └───────────┘  │
                        │  ┌───────────┐  │
                        │  │ Repository│  │  ◄── Outbound interface
                        │  │  Port     │  │
                        │  └───────────┘  │
                        └────────┬────────┘
                                 │
┌────────────────────────────────┼─────────────────────────────────────────┐
│                      APPLICATION CORE (Domain)                           │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                      APPLICATION SERVICE                           │  │
│  │  ┌─────────────────────────────────────────────────────────────┐   │  │
│  │  │                   UserService                               │   │  │
│  │  │  - createUser(), getUserById(), updateUser(), deleteUser()  │   │  │
│  │  └─────────────────────────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                         DOMAIN MODEL                                │ │
│  │  ┌───────────────────┐  ┌───────────────────┐  ┌─────────────────┐  │ │
│  │  │      User         │  │    UserRepository │  │  Domain Events  │  │ │
│  │  │    (Entity)       │  │   (Interface)     │  │   (Optional)    │  │ │
│  │  │  - id: Long       │  │  + save(user)     │  │                 │  │ │
│  │  │  - name: String   │  │  + findById(id)   │  │                 │  │ │
│  │  │  - email: String  │  │  + findAll()      │  │                 │  │ │
│  │  └───────────────────┘  └───────────────────┘  └─────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────┘

``` 
  
## File Structure

```
src/main/java/com/example/
├── domain/                    # Domain Layer (Pure Business Logic)
│   ├── model/
│   │   └── User.java          # Entity
│   ├── repository/
│   │   └── UserRepository.java # Repository interface (port)
│   └── event/
│       └── UserCreatedEvent.java
├── application/               # Application Layer (Use Cases)
│   ├── port/
│   │   ├── inbound/
│   │   │   └── UserUseCasePort.java  # Inbound port (interface)
│   │   └── outbound/
│   │       └── UserRepositoryPort.java # Outbound port (interface)
│   └── service/
│       └── UserService.java   # Use case implementation
└── adapter/                   # Adapters Layer (Implementations)
    ├── inbound/
    │   └── rest/
    │       ├── UserController.java
    │       └── UserDto.java
    └── outbound/
        ├── persistence/
        │   ├── UserRepositoryAdapter.java
        │   └── UserJpaRepository.java
        └── messaging/
            └── UserEventPublisher.java
```

---

## 1. Domain Layer

### Example: User Entity

```java
package com.example.domain.model;

import java.time.LocalDateTime;

public class User {
    private final Long id;
    private final String name;
    private final String email;
    private final LocalDateTime createdAt;
    
    public User(Long id, String name, String email) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        this.id = id;
        this.name = name;
        this.email = email;
        this.createdAt = LocalDateTime.now();
    }
    
    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public LocalDateTime getCreatedAt() { return createdAt; }
}
```

### Example: UserRepository Interface (Outbound Port)

```java
package com.example.domain.repository;

import com.example.domain.model.User;
import java.util.List;
import java.util.Optional;

public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
    List<User> findAll();
    void delete(Long id);
    boolean existsByEmail(String email);
}
```

---

## 2. Application Layer

### Example: UserUseCasePort (Inbound Port)

```java
package com.example.application.port.inbound;

import com.example.domain.model.User;
import java.util.List;
import java.util.Optional;

public interface UserUseCasePort {
    User createUser(String name, String email);
    Optional<User> getUserById(Long id);
    List<User> getAllUsers();
    User updateUser(Long id, String name, String email);
    void deleteUser(Long id);
}
```

### Example: UserService (Application Service)

```java
package com.example.application.service;

import com.example.application.port.inbound.UserUseCasePort;
import com.example.application.port.outbound.UserRepositoryPort;
import com.example.domain.model.User;
import java.util.List;
import java.util.Optional;

public class UserService implements UserUseCasePort {
    
    private final UserRepositoryPort userRepository;
    
    public UserService(UserRepositoryPort userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public User createUser(String name, String email) {
        if (userRepository.existsByEmail(email)) {
            throw new IllegalStateException("Email already exists");
        }
        return userRepository.save(new User(null, name, email));
    }
    
    @Override
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    @Override
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    @Override
    public User updateUser(Long id, String name, String email) {
        User existing = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
        return userRepository.save(new User(id, name, email));
    }
    
    @Override
    public void deleteUser(Long id) {
        userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("User not found"));
        userRepository.delete(id);
    }
}
```

---

## 3. Adapter Layer

### Example: UserController (Inbound Adapter)

```java
package com.example.adapter.inbound.rest;

import com.example.application.port.inbound.UserUseCasePort;
import com.example.domain.model.User;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserUseCasePort userUseCase;
    
    public UserController(UserUseCasePort userUseCase) {
        this.userUseCase = userUseCase;
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody CreateUserRequest req) {
        User user = userUseCase.createUser(req.name(), req.email());
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userUseCase.getUserById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userUseCase.getAllUsers());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userUseCase.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    record CreateUserRequest(String name, String email) {}
}
```

### Example: UserRepositoryAdapter (Outbound Adapter)

```java
package com.example.adapter.outbound.persistence;

import com.example.application.port.outbound.UserRepositoryPort;
import com.example.domain.model.User;
import com.example.domain.repository.UserRepository;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Optional;

@Component
public class UserRepositoryAdapter implements UserRepositoryPort {
    
    private final UserRepository jpaRepository;
    
    public UserRepositoryAdapter(UserRepository jpaRepository) {
        this.jpaRepository = jpaRepository;
    }
    
    @Override
    public User save(User user) {
        return toDomain(jpaRepository.save(toEntity(user)));
    }
    
    @Override
    public Optional<User> findById(Long id) {
        return jpaRepository.findById(id).map(this::toDomain);
    }
    
    @Override
    public List<User> findAll() {
        return jpaRepository.findAll().stream().map(this::toDomain).toList();
    }
    
    @Override
    public void delete(Long id) {
        jpaRepository.deleteById(id);
    }
    
    @Override
    public boolean existsByEmail(String email) {
        return jpaRepository.existsByEmail(email);
    }
    
    private UserEntity toEntity(User user) {
        return new UserEntity(user.getName(), user.getEmail());
    }
    
    private User toDomain(UserEntity entity) {
        return new User(entity.getId(), entity.getName(), entity.getEmail());
    }
}
```

---

## Benefits of Hexagonal Architecture

| Benefit | Description |
|---------|-------------|
| **Testability** | Core logic can be tested without infrastructure |
| **Flexibility** | Easy to swap out adapters (e.g., change database) |
| **Separation of Concerns** | Each layer has a single responsibility |
| **Dependency Inversion** | High-level modules do not depend on low-level modules |
| **Maintainability** | Clear structure makes code easier to understand |

---

## SOLID Principles Applied

1. **S**ingle Responsibility: Each layer has one clear purpose
2. **O**pen/Closed: Extend through new adapters, not modifying core
3. **L**iskov Substitution: Any adapter implementing a port works the same
4. **I**nterface Segregation: Ports are focused, not monolithic
5. **D**ependency Inversion: Core depends on abstractions (ports), not implementations

