Below is your System Design roadmap with Java sample code added wherever practical, while keeping the original structure and short explanations. The code snippets are small examples to help you understand each concept; larger case studies should be implemented as separate projects.

# 1. High-Level Design (HLD)

HLD focuses on overall system architecture, communication, databases, scalability, and reliability. Code is useful for implementing specific mechanisms, while diagrams are more useful for understanding the overall architecture.

## 1.1 Requirements

- Functional Requirements — What the system must do, such as placing orders or logging in.
- Non-Functional Requirements — Performance, scalability, availability, security, and reliability.

Sample example — Java functional requirement:

java
```
public class OrderService {
    public String placeOrder(Long userId) {
        return "Order placed for user: " + userId;
    }
}
```

## 1.2 Capacity Estimation

- Traffic Estimation — Calculate requests per second.
- Storage Estimation — Estimate database and file storage requirements.
- Bandwidth Estimation — Estimate network data transfer.

Sample code — Calculate requests per second:

java
```
public class CapacityEstimator {
    public static void main(String[] args) {
        long dailyRequests = 8_640_000;

        double rps = dailyRequests / 86_400.0;

        System.out.println("Average RPS: " + rps);
    }
}
```

## 1.3 Architecture Fundamentals

- Client-Server — Clients send requests to servers.
- Monolith — One deployable application contains multiple features.
- Microservices — Independent services handle different business capabilities.
- Layered Architecture — Separates presentation, business logic, and data access.

Sample code — Layered architecture:

java
```
// Controller layer
class OrderController {
    private final OrderService service;

    OrderController(OrderService service) {
        this.service = service;
    }

    String createOrder(Long userId) {
        return service.placeOrder(userId);
    }
}

// Service layer
class OrderService {
    String placeOrder(Long userId) {
        return "Order created for user " + userId;
    }
}
```

## 1.4 Load Balancing

- Load Balancer — Distributes traffic across servers.
- Algorithms — Round Robin, Least Connections, and weighted routing.
- Health Checks — Detect unavailable servers.

Sample code — Simple Round Robin:

java
```
import java.util.List;

class LoadBalancer {
    private final List<String> servers;
    private int index = 0;

    LoadBalancer(List<String> servers) {
        this.servers = List.copyOf(servers);
    }

    public synchronized String nextServer() {
        String server = servers.get(index);
        index = (index + 1) % servers.size();
        return server;
    }
}
```

This illustrates server selection, not a production load balancer. Real implementations also need health checks and failure handling.

## 1.5 API Gateway

- Routing — Directs requests to appropriate services.
- Rate Limiting — Restricts excessive requests.
- Authentication — Verifies identity before allowing access.

Sample code — Basic rate limiter:

java
```
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

class RateLimiter {
    private final ConcurrentHashMap<String, AtomicInteger> requests =
            new ConcurrentHashMap<>();

    private final int limit;

    RateLimiter(int limit) {
        this.limit = limit;
    }

    boolean allowRequest(String userId) {
        int count = requests
                .computeIfAbsent(userId, id -> new AtomicInteger())
                .incrementAndGet();

        return count <= limit;
    }
}
```

This is a simplified counter, not a complete time-window rate limiter. A production version needs window expiration and, for distributed deployments, shared state such as Redis.

## 1.6 Databases

- SQL vs NoSQL — Relational databases vs non-relational data models.
- Indexing — Improves query performance.
- Replication — Maintains copies of data.
- Sharding — Splits data across partitions or servers.
- Transactions — Maintain correctness for grouped operations.
- Normalization — Reduces unwanted data duplication.

Sample code — SQL indexing:

sql
```
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL
);

CREATE INDEX idx_users_email ON users(email);

SELECT *
FROM users
WHERE email = 'user@example.com';
```

The index can improve email lookups, depending on the query plan and data distribution.

## 1.7 Caching

- Cache Fundamentals — Store frequently accessed data for faster retrieval.
- Cache-Aside — Load data into the cache when needed.
- Write-Through — Update cache and storage together according to the strategy.
- Eviction Policies — LRU, LFU, and TTL-based expiration.
- Redis — An in-memory data store commonly used for caching.

Sample code — Spring Boot caching:

java
```
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return repository.findById(id)
                .orElseThrow();
    }
}
```

Enable caching in the application using `@EnableCaching` and configure a cache provider. Redis can be used as the provider in a distributed application.

## 1.8 Messaging & Event-Driven Systems

- Message Queue — Buffer work between producers and consumers.
- Kafka — Distributed event streaming platform.
- RabbitMQ — Message broker supporting queues and routing.
- Async Processing — Execute work without blocking the original request.
- Eventual Consistency — Replicas or services may converge to the same state later.

Sample code — Asynchronous Java processing:

java
```
import java.util.concurrent.CompletableFuture;

class NotificationService {

    CompletableFuture<Void> sendNotification(String message) {
        return CompletableFuture.runAsync(() ->
                System.out.println("Sending: " + message));
    }
}
```

For production, use a managed executor and appropriate error handling. Kafka and RabbitMQ examples should additionally cover producers, consumers, acknowledgements, and retries.

## 1.9 Scalability & Reliability

- Vertical Scaling — Increase the capacity of a server.
- Horizontal Scaling — Add more servers.
- High Availability — Minimize service downtime.
- Fault Tolerance — Continue operating despite certain failures.
- Circuit Breaker — Temporarily stop calls to a failing dependency.
- Retry & Timeout — Handle transient failures and prevent indefinite waiting.

Sample code — Timeout:

java
```
import java.util.concurrent.*;

class RemoteService {

    String call() throws Exception {
        try (ExecutorService executor =
                     Executors.newSingleThreadExecutor()) {

            Future<String> future = executor.submit(
                    () -> "Response from remote service");

            return future.get(2, TimeUnit.SECONDS);
        }
    }
}
```

This illustrates a time limit on waiting. Production HTTP clients should generally use their built-in connection and request timeouts instead of creating a new executor per call.

## 1.10 Distributed Systems

- CAP Theorem — Explains consistency and availability trade-offs during network partitions.
- Consistency Models — Define when updates become visible across nodes.
- Distributed Locks — Coordinate access to shared resources.
- Leader Election — Select a coordinator among distributed nodes.
- Idempotency — Repeating an operation does not repeat its intended effect.
- Distributed Transactions — Coordinate changes across multiple data stores or services.

Sample code — Idempotent operation in memory:

java
```
import java.util.concurrent.ConcurrentHashMap;

class PaymentService {
    private final ConcurrentHashMap<String, String> results =
            new ConcurrentHashMap<>();

    String processPayment(String idempotencyKey) {
        return results.computeIfAbsent(
                idempotencyKey,
                key -> "Payment processed: " + key
        );
    }
}
```

This is only an in-memory illustration. Real payment systems need durable storage, concurrency-safe transaction boundaries, and safeguards against duplicate external payment requests.

## 1.11 Networking & Communication

- HTTP/HTTPS — Request-response communication.
- TCP/IP — Network communication protocols.
- DNS — Resolves domain names to network addresses.
- REST & gRPC — Approaches for communication between services.
- WebSockets — Persistent, bidirectional communication.

Sample code — Java HTTP client:

java
```
import java.net.URI;
import java.net.http.*;

class ApiClient {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://example.com"))
                .GET()
                .build();

        HttpResponse<String> response = client.send(
                request,
                HttpResponse.BodyHandlers.ofString()
        );

        System.out.println(response.statusCode());
    }
}
```

## 1.12 Security

- Authentication — Verify who a user is.
- Authorization — Determine what a user can do.
- JWT & OAuth 2.0 — Common token and authorization technologies.
- TLS — Protect data in transit.
- RBAC — Assign permissions based on roles.

Sample code — Spring Security authorization:

java
```
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    return http
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/admin/**")
                    .hasRole("ADMIN")
                    .requestMatchers("/public/**")
                    .permitAll()
                    .anyRequest()
                    .authenticated()
            )
            .build();
}
```

This is an authorization example. Authentication configuration, CSRF handling, and JWT or OAuth resource-server configuration depend on the application.

## 1.13 Observability

- Logging — Record application events.
- Metrics — Measure performance and health.
- Distributed Tracing — Follow requests across services.
- Alerting — Notify teams about failures or abnormal behavior.

Sample code — Logging:

java
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class OrderService {
    private static final Logger log =
            LoggerFactory.getLogger(OrderService.class);

    void placeOrder(Long orderId) {
        log.info("Processing orderId={}", orderId);
    }
}
```

## 1.14 Architecture Patterns

- CQRS — Separate read and write models.
- Event Sourcing — Store state changes as a sequence of events.
- Saga — Coordinate multi-step business transactions across services.
- Strangler Pattern — Gradually replace a legacy system.
- API Composition — Combine data from multiple services.

Sample code — Strategy for selecting a read model:

java
```
interface OrderReader {
    Object findOrder(Long id);
}

class OrderQueryService {
    private final OrderReader reader;

    OrderQueryService(OrderReader reader) {
        this.reader = reader;
    }

    Object getOrder(Long id) {
        return reader.findOrder(id);
    }
}
```

This demonstrates dependency inversion, not a complete CQRS implementation. Actual CQRS separates command handling from query handling.

## 1.15 HLD Case Studies

These are best learned by drawing the architecture first and implementing selected components afterward.

- URL Shortener — Link generation, redirection, and caching.
- WhatsApp — Messaging, delivery, and online presence.
- YouTube — Video uploads, transcoding, and streaming.
- Uber — Location tracking, matching, and trip management.
- E-commerce — Catalog, cart, orders, payments, and inventory.

Sample code — URL shortener core logic:

java
```
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

class UrlShortener {
    private final AtomicLong counter = new AtomicLong();
    private final ConcurrentHashMap<String, String> urls =
            new ConcurrentHashMap<>();

    String shorten(String originalUrl) {
        String code = Long.toString(counter.incrementAndGet(), 36);
        urls.put(code, originalUrl);
        return code;
    }

    String resolve(String code) {
        return urls.get(code);
    }
}
```

This is a local demonstration. A production system needs persistent storage, collision-safe identifiers, validation, and redirect handling.

# 2. Low-Level Design (LLD)

LLD focuses on classes, interfaces, objects, methods, relationships, and interactions. Sample Java code is especially important in this section.

## 2.1 Object-Oriented Programming (OOP)

- Encapsulation — Protect data and control access to it.
- Abstraction — Expose essential behavior while hiding implementation details.
- Inheritance — Extend an existing class.
- Polymorphism — Use a common type for different implementations.

java
```
interface Payment {
    void pay(double amount);
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Card payment: " + amount);
    }
}

class Checkout {
    void process(Payment payment, double amount) {
        payment.pay(amount);
    }
}
```

## 2.2 SOLID Principles

- Single Responsibility — Give a class one clear responsibility.
- Open/Closed — Support extension without unnecessary modification.
- Liskov Substitution — Subtypes should preserve expected parent-type behavior.
- Interface Segregation — Prefer focused interfaces over large ones.
- Dependency Inversion — Depend on abstractions rather than concrete implementations.

java
```
interface PaymentGateway {
    void pay(double amount);
}

class UpiPaymentGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Paid using UPI");
    }
}

class PaymentService {
    private final PaymentGateway gateway;

    PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    void processPayment(double amount) {
        gateway.pay(amount);
    }
}
```

This demonstrates dependency inversion: `PaymentService` depends on the `PaymentGateway` interface rather than a specific payment implementation.

## 2.3 UML Diagrams

- Class Diagram — Shows classes, attributes, methods, and relationships.
- Sequence Diagram — Shows interactions in time order.
- Activity Diagram — Shows workflows and decision paths.
- State Diagram — Shows states and transitions.
- Use Case Diagram — Shows actors and system capabilities.

Sample code — Classes for a class diagram:

java
```
class Customer {
    private Long id;
    private String name;
}

class Order {
    private Long id;
    private Customer customer;
}
```

The `Order` class references a `Customer`. Draw this association in a UML class diagram and decide the intended multiplicity.

## 2.4 Design Patterns

Design patterns are reusable approaches to recurring software design problems.

### 2.4.1 Creational Patterns

Control how objects are created.

- Singleton — Provides one shared instance where appropriate.
- Factory Method — Delegates object creation to factory methods.
- Abstract Factory — Creates families of related objects.
- Builder — Constructs complex objects step by step.
- Prototype — Creates objects by cloning existing instances.

Sample code — Factory Method-style example:

java
```
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsNotification implements Notification {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class NotificationFactory {
    static Notification create(String type) {
        return switch (type.toLowerCase()) {
            case "email" -> new EmailNotification();
            case "sms" -> new SmsNotification();
            default -> throw new IllegalArgumentException(type);
        };
    }
}
```

### 2.4.2 Structural Patterns

Organize classes and objects into flexible structures.

- Adapter — Makes incompatible interfaces work together.
- Decorator — Adds behavior without changing the original class.
- Facade — Provides a simplified interface to a complex subsystem.
- Proxy — Controls access to another object.
- Composite — Treats individual objects and groups uniformly in a tree.
- Bridge — Separates abstraction from implementation.
- Flyweight — Shares common state to reduce memory usage.

Sample code — Adapter:

java
```
interface PaymentProcessor {
    void pay(double amount);
}

class LegacyGateway {
    void makePayment(double amount) {
        System.out.println("Legacy payment: " + amount);
    }
}

class PaymentAdapter implements PaymentProcessor {
    private final LegacyGateway gateway;

    PaymentAdapter(LegacyGateway gateway) {
        this.gateway = gateway;
    }

    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

### 2.4.3 Behavioral Patterns

Manage communication and behavior between objects.

- Strategy — Makes algorithms interchangeable.
- Observer — Notifies subscribers when a subject changes.
- State — Changes behavior according to the current state.
- Command — Represents a request as an object.
- Chain of Responsibility — Passes a request through a sequence of handlers.
- Template Method — Defines an algorithm structure with customizable steps.
- Iterator — Traverses a collection without exposing its internal structure.
- Mediator — Centralizes communication between objects.
- Memento — Saves and restores an object's state.
- Visitor — Adds operations to an object structure without modifying its element classes.
- Interpreter — Represents and evaluates expressions in a grammar.

Sample code — Strategy:

java
```
interface DiscountStrategy {
    double apply(double price);
}

class NoDiscount implements DiscountStrategy {
    public double apply(double price) {
        return price;
    }
}

class TenPercentDiscount implements DiscountStrategy {
    public double apply(double price) {
        return price * 0.90;
    }
}

class PriceCalculator {
    double calculate(
            double price, DiscountStrategy strategy) {
        return strategy.apply(price);
    }
}
```

## 2.5 Class Relationships

- Association — Two classes are connected.
- Aggregation — A whole contains parts that can exist independently.
- Composition — A whole strongly owns its parts.
- Dependency — One class temporarily uses another.
- Multiplicity — Defines how many objects participate in a relationship.

java
```
import java.util.ArrayList;
import java.util.List;

class OrderItem {
    private final String productName;

    OrderItem(String productName) {
        this.productName = productName;
    }
}

class Order {
    private final List<OrderItem> items = new ArrayList<>();

    void addItem(OrderItem item) {
        items.add(item);
    }
}
```

This shows an `Order` holding multiple `OrderItem` objects. Whether the relationship is composition depends on ownership and lifecycle rules, not merely on the field declaration.

## 2.6 Composition vs Inheritance

- Inheritance — Model an “is-a” relationship, such as `Car extends Vehicle`.
- Composition — Model a “has-a” relationship, such as `Car` containing an `Engine`.
- Prefer Composition — Often makes behavior easier to change and test.

java
```
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }

    void start() {
        engine.start();
    }
}
```

## 2.7 Dependency Injection & IoC

- Inversion of Control — Transfers object lifecycle control to a framework or container.
- Dependency Injection — Supplies required dependencies from outside an object.
- Constructor Injection — Makes required dependencies explicit and testable.

java
```
@Service
class OrderService {
    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

Spring manages the service and injects the repository when both are registered as beans.

## 2.8 Exception Handling & Validation

- Input Validation — Rejects invalid data.
- Custom Exceptions — Represent meaningful application errors.
- Global Exception Handling — Produces consistent error responses.
- Error Handling Strategy — Defines which failures to retry, report, or propagate.

java
```
class InsufficientBalanceException extends RuntimeException {
    InsufficientBalanceException() {
        super("Insufficient balance");
    }
}

class BankAccount {
    private double balance = 100;

    void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException(
                    "Amount must be positive");
        }

        if (amount > balance) {
            throw new InsufficientBalanceException();
        }

        balance -= amount;
    }
}
```

## 2.9 Concurrency & Thread Safety

- Threads — Execute tasks concurrently.
- Synchronization — Coordinates access to shared state.
- Locks — Protect critical sections.
- ExecutorService — Manages task execution with thread pools.
- Race Conditions — Occur when timing affects correctness.
- Concurrent Collections — Provide thread-safe collection operations.

java
```
import java.util.concurrent.atomic.AtomicInteger;

class SafeCounter {
    private final AtomicInteger count = new AtomicInteger();

    void increment() {
        count.incrementAndGet();
    }

    int getCount() {
        return count.get();
    }
}
```

## 2.10 Refactoring & Clean Code

- Refactoring — Improves code structure without changing external behavior.
- DRY — Avoid unnecessary duplication.
- KISS — Keep solutions as simple as practical.
- Code Smells — Identify warning signs of poor design.
- Separation of Concerns — Keep unrelated responsibilities separate.

Before:

java
```
if (userType.equals("ADMIN")) {
    discount = price * 0.20;
} else if (userType.equals("CUSTOMER")) {
    discount = price * 0.10;
}
```

After — use a strategy or dedicated discount policy when the rules are expected to grow:

java
```
interface DiscountPolicy {
    double discount(double price);
}

class AdminDiscount implements DiscountPolicy {
    public double discount(double price) {
        return price * 0.20;
    }
}
```

## 2.11 LLD Case Studies

Each case study should include a class diagram, Java implementation, important design patterns, and unit tests.

- Parking Lot — Parking allocation, pricing, and availability.
- Elevator System — Scheduling, states, and request handling.
- Splitwise — Expense splitting and balance settlement.
- Library Management — Books, members, lending, and fines.
- Tic-Tac-Toe — Board modeling and game rules.
- Snake and Ladder — Board representation and turn management.
- Chess — Pieces, legal moves, and game state.
- Payment System — Payment lifecycle, retries, and idempotency.

Sample code — Parking Lot model:

java
```
enum VehicleType {
    CAR, BIKE, TRUCK
}

class Vehicle {
    private final String registrationNumber;
    private final VehicleType type;

    Vehicle(String registrationNumber, VehicleType type) {
        this.registrationNumber = registrationNumber;
        this.type = type;
    }

    VehicleType getType() {
        return type;
    }
}

class ParkingSpot {
    private final VehicleType supportedType;
    private Vehicle parkedVehicle;

    ParkingSpot(VehicleType supportedType) {
        this.supportedType = supportedType;
    }

    boolean park(Vehicle vehicle) {
        if (parkedVehicle != null
                || vehicle.getType() != supportedType) {
            return false;
        }

        parkedVehicle = vehicle;
        return true;
    }
}
```

This is a starting model, not a complete parking system. A full implementation should add spot allocation, tickets, pricing, and concurrency-safe booking.

# 3. Java Foundations for System Design

These concepts help you implement LLD solutions and understand real-world backend projects.

## 3.1 Collections Framework

- Explanation — Use lists, sets, maps, queues, and appropriate data structures.

java
```
Map<Long, String> users = new HashMap<>();
users.put(1L, "Amit");

System.out.println(users.get(1L));
```

## 3.2 Generics

- Explanation — Write reusable, type-safe classes and methods.

java
```
class Box<T> {
    private final T value;

    Box(T value) {
        this.value = value;
    }

    T getValue() {
        return value;
    }
}
```

## 3.3 Java 8+ Features

- Explanation — Understand lambdas, streams, and modern Java APIs.

java
```
List<String> names = List.of("Amit", "Riya");

names.forEach(name -> System.out.println(name));
```

## 3.4 Streams & Lambdas

- Explanation — Process collections declaratively.

java
```
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

List<Integer> evenNumbers = numbers.stream()
        .filter(n -> n % 2 == 0)
        .toList();
```

## 3.5 Multithreading & Concurrency

- Explanation — Execute and coordinate concurrent tasks.

java
```
Thread thread = new Thread(
        () -> System.out.println("Running concurrently"));

thread.start();
```

## 3.6 Executors, Locks & Synchronization

- Explanation — Manage threads and shared resources safely.

java
```
ExecutorService executor =
        Executors.newFixedThreadPool(2);

try {
    executor.submit(() -> System.out.println("Task running"));
} finally {
    executor.shutdown();
}
```

## 3.7 JVM & Garbage Collection

- Explanation — Understand memory management and object cleanup.

java
```
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}

User user = new User("Amit");
user = null; // Eligible for garbage collection if unreachable.
```

## 3.8 Interfaces, Abstract Classes & Enums

- Explanation — Define contracts, reusable behavior, and fixed values.

java
```
enum OrderStatus {
    CREATED, PAID, SHIPPED, DELIVERED, CANCELLED
}
```

## 3.9 Spring Boot, REST APIs & JPA

- Explanation — Build backend services and persist data.

java
```
@RestController
@RequestMapping("/users")
class UserController {

    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id) {
        return "User ID: " + id;
    }
}
```

## 3.10 JUnit & Mockito

\- Explanation — Test application logic and mock dependencies.

java
```
import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;

class CalculatorTest {

    @Test
    void addsTwoNumbers() {
        assertEquals(5, 2 + 3);
    }
}
```

# 4\. Supporting Concepts

## 4.1 DBMS & SQL

\- Explanation — Learn joins, transactions, normalization, indexing, and query optimization.

sql
```
SELECT o.id, u.name
FROM orders o
JOIN users u ON o.user_id = u.id;
```

## 4.2 Operating Systems

\- Explanation — Understand processes, threads, memory, and scheduling.

java
```
Thread first = new Thread(() -> System.out.println("Task A"));
Thread second = new Thread(() -> System.out.println("Task B"));

first.start();
second.start();
```

## 4.3 Computer Networks

\- Explanation — Learn HTTP, TCP/IP, DNS, TLS, and network communication.

Use the Java `HttpClient` example from section 1.11 to send a real HTTP request and inspect the response status.

## 4.4 Data Structures & Algorithms

\- Explanation — Understand trees, graphs, hashing, complexity, and common algorithms.

java
```
Map<String, Integer> frequency = new HashMap<>();

for (String word : List.of("java", "sql", "java")) {
    frequency.merge(word, 1, Integer::sum);
}
```

## 4.5 Distributed Systems

\- Explanation — Study coordination, replication, failures, and consistency across machines.

java
```
String requestId = "payment-123";

String result = paymentService.processPayment(requestId);
```

Use the same request ID for retries and enforce uniqueness durably in the payment service. Merely passing an ID does not guarantee idempotency.

## 4.6 Cloud & Docker

\- Explanation — Package, deploy, and scale applications.

Sample `Dockerfile` for a Spring Boot JAR:

dockerfile
```
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## 4.7 Testing & CI/CD

\- Explanation — Automate tests, builds, and software releases.

Sample Maven command:

bash
```
mvn clean verify
```

This runs the Maven build and configured verification lifecycle, including tests and other checks configured in the project.

# 5\. How HLD, LLD, and Java connect in a real project

Consider an e-commerce application.

text
```
Client / Frontend
       |
       v
HLD: System Architecture
Load Balancer -> API Gateway -> Services -> Database
                              |
                              v
                         Message Queue
       |
       v
LLD: Component Design
OrderController
       |
       v
OrderService
       |
       v
OrderRepository
       |
       v
Java Implementation
Classes, interfaces, Spring DI, JPA, and tests
```

- HLD question: How would you design the application to handle millions of users?
- LLD question: How would you design `OrderService`, `PaymentService`, and `OrderRepository` and define their interactions?
- Java question: How would you implement these classes using interfaces, dependency injection, and concurrency features?

# 6\. Recommended Learning Order

1. Java OOP and Collections.
2. SOLID principles, UML, and class relationships.
3. Creational, Structural, and Behavioral design patterns.
4. LLD case studies, starting with Parking Lot.
5. HLD fundamentals: requirements, APIs, databases, and caching.
6. Distributed systems: messaging, consistency, scaling, and fault tolerance.
7. Complete HLD case studies, such as URL Shortener and WhatsApp.

Important: These are learning snippets, not production-ready implementations. For every major concept, understand the problem first, draw the design, implement the Java version, and test edge cases before applying it to a complete project.
