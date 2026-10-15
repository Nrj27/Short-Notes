# System Design Roadmap — Complete Tree with Short Explanations

This roadmap covers System Design from fundamentals to advanced concepts, organized like a project folder structure. Each topic includes a short explanation to help you understand what it is and why it matters, especially for Java and Spring Boot development.

## 1. High-Level Design (HLD)

HLD focuses on the overall system architecture: components, communication, databases, scalability, and reliability.

## 1. HLD

1.1 Requirements

- Functional Requirements — What the system must do, such as placing orders or logging in.
- Non-Functional Requirements — Performance, scalability, availability, security, and reliability.

1.2 Capacity Estimation

- Traffic Estimation — Calculate requests per second.
- Storage Estimation — Estimate database and file storage requirements.
- Bandwidth Estimation — Estimate network data transfer.

1.3 Architecture Fundamentals

- Client-Server — Clients send requests to servers.
- Monolith — One deployable application contains multiple features.
- Microservices — Independent services handle different business capabilities.
- Layered Architecture — Separates presentation, business logic, and data access.

1.4 Load Balancing

- Load Balancer — Distributes traffic across servers.
- Algorithms — Round Robin, Least Connections, and weighted routing.
- Health Checks — Detect unavailable servers.

1.5 API Gateway

- Routing — Directs requests to appropriate services.
- Rate Limiting — Restricts excessive requests.
- Authentication — Verifies identity before allowing access.

1.6 Databases

- SQL vs NoSQL — Relational databases vs non-relational data models.
- Indexing — Improves query performance.
- Replication — Maintains copies of data.
- Sharding — Splits data across partitions or servers.
- Transactions — Maintain correctness for grouped operations.
- Normalization — Reduces unwanted data duplication.

1.7 Caching

- Cache Fundamentals — Store frequently accessed data for faster retrieval.
- Cache-Aside — Load data into the cache when needed.
- Write-Through — Update cache and storage together according to the strategy.
- Eviction Policies — LRU, LFU, and TTL-based expiration.
- Redis — An in-memory data store commonly used for caching.

1.8 Messaging & Event-Driven Systems

- Message Queue — Buffer work between producers and consumers.
- Kafka — Distributed event streaming platform.
- RabbitMQ — Message broker supporting queues and routing.
- Async Processing — Execute work without blocking the original request.
- Eventual Consistency — Replicas or services may converge to the same state later.

1.9 Scalability & Reliability

- Vertical Scaling — Increase the capacity of a server.
- Horizontal Scaling — Add more servers.
- High Availability — Minimize service downtime.
- Fault Tolerance — Continue operating despite certain failures.
- Circuit Breaker — Temporarily stop calls to a failing dependency.
- Retry & Timeout — Handle transient failures and prevent indefinite waiting.

1.10 Distributed Systems

- CAP Theorem — Explains consistency and availability trade-offs during network partitions.
- Consistency Models — Define when updates become visible across nodes.
- Distributed Locks — Coordinate access to shared resources.
- Leader Election — Select a coordinator among distributed nodes.
- Idempotency — Repeating an operation does not repeat its intended effect.
- Distributed Transactions — Coordinate changes across multiple data stores or services.

1.11 Networking & Communication

- HTTP/HTTPS — Request-response communication.
- TCP/IP — Network communication protocols.
- DNS — Resolves domain names to network addresses.
- REST & gRPC — Approaches for communication between services.
- WebSockets — Persistent, bidirectional communication.

1.12 Security

- Authentication — Verify who a user is.
- Authorization — Determine what a user can do.
- JWT & OAuth 2.0 — Common token and authorization technologies.
- TLS — Protect data in transit.
- RBAC — Assign permissions based on roles.

1.13 Observability

- Logging — Record application events.
- Metrics — Measure performance and health.
- Distributed Tracing — Follow requests across services.
- Alerting — Notify teams about failures or abnormal behavior.

1.14 Architecture Patterns

- CQRS — Separate read and write models.
- Event Sourcing — Store state changes as a sequence of events.
- Saga — Coordinate multi-step business transactions across services.
- Strangler Pattern — Gradually replace a legacy system.
- API Composition — Combine data from multiple services.

1.15 HLD Case Studies

- URL Shortener — Link generation, redirection, and caching.
- WhatsApp — Messaging, delivery, and online presence.
- YouTube — Video uploads, transcoding, and streaming.
- Uber — Location tracking, matching, and trip management.
- E-commerce — Catalog, cart, orders, payments, and inventory.

## 2. Low-Level Design (LLD)

LLD focuses on the internal design of components: classes, interfaces, objects, methods, and their interactions.

## 2. LLD

2.1 Object-Oriented Programming (OOP)

- Encapsulation — Protect data and control access to it.
- Abstraction — Expose essential behavior while hiding implementation details.
- Inheritance — Extend an existing class.
- Polymorphism — Use a common type for different implementations.

2.2 SOLID Principles

- Single Responsibility — Give a class one clear responsibility.
- Open/Closed — Support extension without unnecessary modification.
- Liskov Substitution — Subtypes should preserve expected parent-type behavior.
- Interface Segregation — Prefer focused interfaces over large ones.
- Dependency Inversion — Depend on abstractions rather than concrete implementations.

2.3 UML Diagrams

- Class Diagram — Shows classes, attributes, methods, and relationships.
- Sequence Diagram — Shows interactions in time order.
- Activity Diagram — Shows workflows and decision paths.
- State Diagram — Shows states and transitions.
- Use Case Diagram — Shows actors and system capabilities.

2.4 Design Patterns

Design patterns are reusable approaches to recurring software design problems.

2.4.1 Creational Patterns

Control how objects are created.

- Singleton — Provides one shared instance where appropriate.
- Factory Method — Delegates object creation to factory methods.
- Abstract Factory — Creates families of related objects.
- Builder — Constructs complex objects step by step.
- Prototype — Creates objects by cloning existing instances.

2.4.2 Structural Patterns

Organize classes and objects into flexible structures.

- Adapter — Makes incompatible interfaces work together.
- Decorator — Adds behavior without changing the original class.
- Facade — Provides a simplified interface to a complex subsystem.
- Proxy — Controls access to another object.
- Composite — Treats individual objects and groups uniformly in a tree.
- Bridge — Separates abstraction from implementation.
- Flyweight — Shares common state to reduce memory usage.

2.4.3 Behavioral Patterns

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

2.5 Class Relationships

- Association — Two classes are connected.
- Aggregation — A whole contains parts that can exist independently.
- Composition — A whole strongly owns its parts.
- Dependency — One class temporarily uses another.
- Multiplicity — Defines how many objects participate in a relationship.

2.6 Composition vs Inheritance

- Inheritance — Model an “is-a” relationship, such as `Car extends Vehicle`.
- Composition — Model a “has-a” relationship, such as `Car` containing an `Engine`.
- Prefer Composition — Often makes behavior easier to change and test.

2.7 Dependency Injection & IoC

- Inversion of Control — Transfers object lifecycle control to a framework or container.
- Dependency Injection — Supplies required dependencies from outside an object.
- Constructor Injection — Makes required dependencies explicit and testable.

2.8 Exception Handling & Validation

- Input Validation — Rejects invalid data.
- Custom Exceptions — Represent meaningful application errors.
- Global Exception Handling — Produces consistent error responses.
- Error Handling Strategy — Defines which failures to retry, report, or propagate.

2.9 Concurrency & Thread Safety

- Threads — Execute tasks concurrently.
- Synchronization — Coordinates access to shared state.
- Locks — Protect critical sections.
- ExecutorService — Manages task execution with thread pools.
- Race Conditions — Occur when timing affects correctness.
- Concurrent Collections — Provide thread-safe collection operations.

2.10 Refactoring & Clean Code

- Refactoring — Improves code structure without changing external behavior.
- DRY — Avoid unnecessary duplication.
- KISS — Keep solutions as simple as practical.
- Code Smells — Identify warning signs of poor design.
- Separation of Concerns — Keep unrelated responsibilities separate.

2.11 LLD Case Studies

- Parking Lot — Parking allocation, pricing, and availability.
- Elevator System — Scheduling, states, and request handling.
- Splitwise — Expense splitting and balance settlement.
- Library Management — Books, members, lending, and fines.
- Tic-Tac-Toe — Board modeling and game rules.
- Snake and Ladder — Board representation and turn management.
- Chess — Pieces, legal moves, and game state.
- Payment System — Payment lifecycle, retries, and idempotency.

## 3. Java Foundations for System Design

These concepts help you implement LLD solutions and understand real-world backend projects.

## 3. Java

- 3.1 Collections Framework — Use lists, sets, maps, queues, and appropriate data structures.
- 3.2 Generics — Write reusable, type-safe classes and methods.
- 3.3 Java 8+ Features — Understand lambdas, streams, and modern Java APIs.
- 3.4 Streams & Lambdas — Process collections declaratively.
- 3.5 Multithreading & Concurrency — Execute and coordinate concurrent tasks.
- 3.6 Executors, Locks & Synchronization — Manage threads and shared resources safely.
- 3.7 JVM & Garbage Collection — Understand memory management and object cleanup.
- 3.8 Interfaces, Abstract Classes & Enums — Define contracts, reusable behavior, and fixed values.
- 3.9 Spring Boot, REST APIs & JPA — Build backend services and persist data.
- 3.10 JUnit & Mockito — Test application logic and mock dependencies.

## 4. Supporting Concepts

These fundamentals support both HLD and LLD.

## 4. Supporting Concepts

- 4.1 DBMS & SQL — Learn joins, transactions, normalization, indexing, and query optimization.
- 4.2 Operating Systems — Understand processes, threads, memory, and scheduling.
- 4.3 Computer Networks — Learn HTTP, TCP/IP, DNS, TLS, and network communication.
- 4.4 Data Structures & Algorithms — Understand trees, graphs, hashing, complexity, and common algorithms.
- 4.5 Distributed Systems — Study coordination, replication, failures, and consistency across machines.
- 4.6 Cloud & Docker — Package, deploy, and scale applications.
- 4.7 Testing & CI/CD — Automate tests, builds, and software releases.

## 5. How HLD, LLD, and Java connect in a real project

Consider an e-commerce application.

Client / Frontend

User places an order

HLD: System Architecture

Load balancer → API → Services → Database

LLD: Component Design

OrderController → OrderService → OrderRepository

Java Implementation

Classes, interfaces, Spring DI, JPA, and tests

- HLD question: How would you design the application to handle millions of users?
- LLD question: How would you design `OrderService`, `PaymentService`, and `OrderRepository` and define their interactions?
- Java question: How would you implement these classes using interfaces, dependency injection, and concurrency features?

## 6. Recommended learning order

1. Java OOP and Collections.
2. SOLID principles, UML, and class relationships.
3. Creational, Structural, and Behavioral design patterns.
4. LLD case studies, starting with Parking Lot.
5. HLD fundamentals: requirements, APIs, databases, and caching.
6. Distributed systems: messaging, consistency, scaling, and fault tolerance.
7. Complete HLD case studies, such as URL Shortener and WhatsApp.

Key distinction: HLD defines the overall system and its components. LLD defines the internal design of those components. In real projects, both are refined iteratively as requirements and constraints become clearer.
