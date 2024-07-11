# Introduction

- [Preface](#preface)
- [Communication](#communication)
- [Coordination](#coordination)
- [Scalability](#scalability)
- [Resiliency](#resiliency)
- [Maintainability](#maintainability)
- [Anatomy of a distributed system](#anatomy-of-a-distributed-system)

## Preface
Most materials on distributed systems are either too theoretical or too practical. This book aims at the middle.
If you're already an expert, this book is not for you. It focuses on breadth rather than depth.

Distributed system == group of nodes, communicating over some channel to accomplish a task. Nodes can be a computer, phone, browser, etc.

Why bother building such a system?
 * Some apps are inherently distributed - eg the web.
 * To achieve high availability - if one node fails, the system doesn't crash.
 * To tackle data-intensive workloads which can't fit on a single machine.
 * Performance requirements - eg Netflix streams HQ tv thanks to serving you from a datacenter close to you.

The book focuses on tackling the fundamental problems one faces when designing a distributed system.

## Communication
First challenge is for nodes to communicate with each other over the network:
 * How are request/response messages represented on the wire?
 * What happens when there is a temporary network outage?
 * How to avoid man in the middle attacks?

In practice, a nice network library can abstract away those details, but oftentimes abstractions leak and you need to know how the networks work under the hood.

## Coordination
Some form of coordination is required to make two nodes work together.

Good metaphor - the [two generals problem](https://en.wikipedia.org/wiki/Two_Generals%27_Problem).

Two generals need to agree on a time to make a joint attack on a city. All messages between them can be intercepted & dropped.
How to ensure they attack the city at the same time, despite the faulty communication medium?

## Scalability
How efficiently can the application handle load. Ways to measure it depends on the app's use-case - number of users, number of reads/writes, RPS, etc.

For the type of applications covered in the book, load is measured in:
 * Throughput - number of requests per second (RPS) the application can process.
 * Response time - time elapsed in seconds between sending a request and receiving a response.

When the application reaches a certain threshold in terms of load, throughput starts degrading:
![app-capacity](images/app-capacity.png)

The capacity depends on a system's architecture, implementation, physical limitations (eg CPU, Memory, etc).

An easy way to fix this is to buy more expensive hardware - scaling up. This scaling mechanism has a limit, though, since you can't scale hardware without bounds.

The alternative is to scale horizontally by adding more machines which work together.

## Resiliency
A distributed system is resilient when it doesn't crash even in the face of failures.

Failures which are left unchecked can impact the system's availability - the % of time the system is available for use.
```
availability = uptime / total time.
```

Availability is often expressed in terms of nines:
![availability-nines](images/availability-nines.png)

Three nines is considered acceptable for users, anything above four nines is considered highly available.

There are various techniques one can use to increase availability. All of them try to mitigate failures when they inevitably come up - fault isolation, self-healing mechanisms, redundancy, etc.

## Maintainability
Most of the cost of software is spent maintaining it - fixing bugs, adding new features, operating it.

We should aspire to make our systems easy to maintain - easy to extend, modify and operate.

One critical aspect is for a system to be well-tested - unit, integration & e2e tests.
Another one is for operators to have the necessary tools to monitor a system's health & making quick changes - feature flags, rollback mechanisms, etc.

Historically, developers, testers & operators were different teams, but nowadays, it tends to be the same team.

## Anatomy of a distributed system
Main focus of the book is on backend distributed systems that run on commodity machines.

From a hardware perspective, a distributed system is a set of machines communicating over network links.
From a runtime perspective, it's a set of processes which communicate via inter-process communication (IPC), eg HTTP.
From an implementation perspective, it's a set of loosely-coupled components which communicate via APIs.

All are valid points of view on the system & the book will switch between them based on context.

A service implements one specific part of the overall system's capabilities. At its core sit the business logic.

At the sides, there are interfaces which define functionalities provided by external systems (eg database, another service in the system, etc).
The interfaces are implemented by adapters which implement the technical details of connecting to the external systems.

This architectural pattern is known as [ports and adapters](http://wiki.c2.com/?PortsAndAdaptersArchitecture)

This architectural style accomplishes a setup in which the business logic don't depend on the technical details. Instead, technical details depend on business logic - dependency inversion:
![ports-and-adapters](images/ports-and-adapters.png)


---
You're describing the **Ports and Adapters** architecture, also known as the **Hexagonal Architecture**. This architecture pattern, as you correctly mentioned, aims to decouple the business logic from the technical details, facilitating maintainability, testability, and flexibility. Let's delve deeper into the components and benefits of this architectural style.

### Ports and Adapters Architecture (Hexagonal Architecture)

#### Core Components

1. **Business Logic (Domain Layer)**
   - This is the core part of the application where the business rules and logic reside. It is completely independent of external systems and frameworks.
   - It defines the core entities, services, and operations that drive the business requirements.

2. **Ports**
   - **Definition**: Interfaces that define the entry points (input ports) and exit points (output ports) of the application. These ports represent the boundaries of the business logic.
   - **Function**: They specify what operations can be performed and what data can be accessed but do not include any implementation details.
   - **Examples**:
     - Input Ports: Service interfaces that define the use cases of the application.
     - Output Ports: Repository interfaces for data access, external service interfaces for integrations, etc.

3. **Adapters**
   - **Definition**: Implementations of the ports. Adapters contain the technical details of interacting with external systems, such as databases, web services, or user interfaces.
   - **Function**: They convert the data and calls from the external systems into a format and protocol understood by the business logic and vice versa.
   - **Examples**:
     - Database Adapters: Implement repository interfaces to interact with databases.
     - Web Service Adapters: Implement service interfaces to communicate with external APIs.
     - UI Adapters: Handle user interactions and translate them into service calls.

### Dependency Inversion Principle (DIP)

In the Ports and Adapters architecture, the **Dependency Inversion Principle** is applied to ensure that high-level modules (business logic) do not depend on low-level modules (technical details). Instead, both should depend on abstractions (interfaces).

- **High-level Modules**: The business logic, which is at the core of the application.
- **Low-level Modules**: The technical implementations such as database access, external service communication, and user interfaces.
- **Abstractions**: The interfaces (ports) that define the contract between the business logic and the technical details.

### Benefits of Ports and Adapters Architecture

1. **Separation of Concerns**: Clear separation between business logic and technical implementation details enhances maintainability and understandability.
2. **Testability**: Business logic can be tested independently of external systems by using mock implementations of the ports.
3. **Flexibility and Extensibility**: Adapters can be easily replaced or modified without affecting the core business logic. This makes it easier to switch databases, change external services, or update user interfaces.
4. **Decoupling**: Reduces the coupling between different parts of the system, making the codebase more modular and easier to navigate.

### Example Scenario

#### Business Logic (Domain Layer)
```java
// Input Port
public interface UserService {
    UserDTO createUser(CreateUserRequest request);
    UserDTO getUserById(String userId);
}

// Output Port
public interface UserRepository {
    User save(User user);
    Optional<User> findById(String userId);
}

// Core Business Logic
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDTO createUser(CreateUserRequest request) {
        User user = new User(request.getUsername(), request.getPassword());
        userRepository.save(user);
        return new UserDTO(user.getId(), user.getUsername());
    }

    @Override
    public UserDTO getUserById(String userId) {
        User user = userRepository.findById(userId).orElseThrow(() -> new UserNotFoundException(userId));
        return new UserDTO(user.getId(), user.getUsername());
    }
}
```

#### Adapters
```java
// Database Adapter
public class JpaUserRepository implements UserRepository {
    private final JpaRepository<User, String> repository;

    public JpaUserRepository(JpaRepository<User, String> repository) {
        this.repository = repository;
    }

    @Override
    public User save(User user) {
        return repository.save(user);
    }

    @Override
    public Optional<User> findById(String userId) {
        return repository.findById(userId);
    }
}

// REST API Adapter
@RestController
@RequestMapping("/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@RequestBody CreateUserRequest request) {
        UserDTO user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUserById(@PathVariable String id) {
        UserDTO user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
}
```

### Summary

The Ports and Adapters (Hexagonal) architecture pattern effectively decouples business logic from technical details, adhering to the Dependency Inversion Principle. By defining interfaces (ports) for interaction and implementing these interfaces with adapters, you can create a flexible, maintainable, and testable system. This separation allows the business logic to remain independent of the specific technologies used, facilitating easier updates, testing, and scaling.


Certainly! Let's create a more concrete example of a port in the context of the Ports and Adapters (Hexagonal) architecture. We'll use the example of a simple e-commerce application where we have a service to manage orders. 

### Example Scenario: Order Management Service

In this scenario, we will define an input port for the Order Management Service. This port will be an interface that defines the operations related to order management. The actual business logic will implement this interface.

#### Order Management Service (Port)

The **OrderService** interface represents the port. It defines the operations that can be performed on orders.

```java
// Input Port
public interface OrderService {
    OrderDTO createOrder(CreateOrderRequest request);
    OrderDTO getOrderById(String orderId);
    void updateOrder(String orderId, UpdateOrderRequest request);
    void deleteOrder(String orderId);
}
```

#### CreateOrderRequest and UpdateOrderRequest

These classes represent the data transfer objects (DTOs) used to carry data for creating and updating orders.

```java
public class CreateOrderRequest {
    private String customerId;
    private List<String> productIds;
    private double totalAmount;

    // Constructors, getters, and setters
}

public class UpdateOrderRequest {
    private List<String> productIds;
    private double totalAmount;

    // Constructors, getters, and setters
}
```

#### OrderDTO

This class represents the data transfer object used to transfer order data between layers.

```java
public class OrderDTO {
    private String orderId;
    private String customerId;
    private List<String> productIds;
    private double totalAmount;
    private String status;

    // Constructors, getters, and setters
}
```

#### Business Logic Implementation (Service)

The **OrderServiceImpl** class implements the **OrderService** interface and contains the business logic.

```java
public class OrderServiceImpl implements OrderService {
    private final OrderRepository orderRepository;
    private final CustomerService customerService;
    private final ProductService productService;

    public OrderServiceImpl(OrderRepository orderRepository, CustomerService customerService, ProductService productService) {
        this.orderRepository = orderRepository;
        this.customerService = customerService;
        this.productService = productService;
    }

    @Override
    public OrderDTO createOrder(CreateOrderRequest request) {
        // Validate customer and products
        if (!customerService.exists(request.getCustomerId())) {
            throw new IllegalArgumentException("Invalid customer ID");
        }
        productService.validateProducts(request.getProductIds());

        // Create and save order
        Order order = new Order(request.getCustomerId(), request.getProductIds(), request.getTotalAmount());
        order.setStatus("CREATED");
        orderRepository.save(order);

        return new OrderDTO(order.getId(), order.getCustomerId(), order.getProductIds(), order.getTotalAmount(), order.getStatus());
    }

    @Override
    public OrderDTO getOrderById(String orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow(() -> new OrderNotFoundException(orderId));
        return new OrderDTO(order.getId(), order.getCustomerId(), order.getProductIds(), order.getTotalAmount(), order.getStatus());
    }

    @Override
    public void updateOrder(String orderId, UpdateOrderRequest request) {
        Order order = orderRepository.findById(orderId).orElseThrow(() -> new OrderNotFoundException(orderId));
        order.setProductIds(request.getProductIds());
        order.setTotalAmount(request.getTotalAmount());
        orderRepository.save(order);
    }

    @Override
    public void deleteOrder(String orderId) {
        orderRepository.deleteById(orderId);
    }
}
```

#### Adapters

Now, let's implement adapters that interact with external systems, such as the database and RESTful API.

##### Database Adapter (Repository)

The **OrderRepository** interface and its implementation for database interaction.

```java
// Output Port
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(String orderId);
    void deleteById(String orderId);
}

// Database Adapter
public class JpaOrderRepository implements OrderRepository {
    private final JpaRepository<Order, String> jpaRepository;

    public JpaOrderRepository(JpaRepository<Order, String> jpaRepository) {
        this.jpaRepository = jpaRepository;
    }

    @Override
    public Order save(Order order) {
        return jpaRepository.save(order);
    }

    @Override
    public Optional<Order> findById(String orderId) {
        return jpaRepository.findById(orderId);
    }

    @Override
    public void deleteById(String orderId) {
        jpaRepository.deleteById(orderId);
    }
}
```

##### REST API Adapter

The **OrderController** class acts as the adapter for handling HTTP requests and invoking the business logic.

```java
@RestController
@RequestMapping("/orders")
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<OrderDTO> createOrder(@RequestBody CreateOrderRequest request) {
        OrderDTO order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderDTO> getOrderById(@PathVariable String id) {
        OrderDTO order = orderService.getOrderById(id);
        return ResponseEntity.ok(order);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Void> updateOrder(@PathVariable String id, @RequestBody UpdateOrderRequest request) {
        orderService.updateOrder(id, request);
        return ResponseEntity.noContent().build();
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteOrder(@PathVariable String id) {
        orderService.deleteOrder(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Summary

In this example, the **OrderService** interface acts as an input port, defining the operations related to order management. The **OrderServiceImpl** class contains the business logic and implements the port. The **JpaOrderRepository** class is an adapter that implements the **OrderRepository** interface (an output port) to interact with the database. The **OrderController** class is another adapter that handles HTTP requests and forwards them to the business logic.

This setup ensures that the business logic is decoupled from the technical details of how data is stored and accessed, adhering to the principles of the Ports and Adapters architecture and achieving the benefits of maintainability, testability, and flexibility.
