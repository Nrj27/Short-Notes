## Proposed cheat sheet structure
It will cover the complete system design landscape using Java 21+, Spring Boot, PostgreSQL (SQL), MongoDB, Redis, and Kafka.

a compact engineering reference you can use for interview preparation, quick revision, and real-world architecture decisions.


## System Design Master Cheat Sheet

Compact Edition · Interview + Production Engineering · Java & Spring Boot

### 10 pages

Core reference edition

### Java stack

Spring Boot + SQL + NoSQL

Contents: architecture, requirements, capacity estimates, databases, caching, distributed systems, Kafka, reliability, security, case studies, and interview answers.

# Page 1 — System design fundamentals

## 1. What is system design?

System design defines how software components, data stores, APIs, infrastructure, and communication mechanisms work together to meet business and technical requirements.

- High-level design (HLD): services, databases, queues, caches, communication, and deployment.
- Low-level design (LLD): classes, interfaces, design patterns, schemas, and implementation details.
- Functional requirements: what the system must do.
- Non-functional requirements (NFRs): performance, scalability, availability, security, durability, and cost.

## 2. The standard architecture

mermaid
```
flowchart TD
    U[Client / Mobile / Web] --> CDN[CDN / WAF]
    CDN --> LB[Load Balancer]
    LB --> API[Spring Boot API]
    API --> SQL[(PostgreSQL)]
    API --> DOC[(MongoDB)]
    API --> CACHE[(Redis)]
    API --> Q[Kafka]
    Q --> W[Background Workers]
    W --> SQL
    W --> DOC
    API --> OBS[Logs / Metrics / Traces]
```

Component responsibilities

| Component | Primary purpose |
| --- | --- |
| CDN | Cache content close to users |
| WAF | Filter malicious web traffic |
| Load balancer | Distribute requests |
| API gateway | Routing, authentication integration, quotas |
| Spring Boot service | Business logic |
| PostgreSQL | Relational and transactional data |
| MongoDB | Document-oriented data |
| Redis | Low-latency caching and selected ephemeral state |
| Kafka | Durable event streaming and asynchronous processing |
| Workers | Background processing |
| Observability stack | Understand performance and failures |

## 3. Core design principles

- Start simple; add distributed complexity only when justified.
- Prefer stateless application instances for horizontal scaling.
- Keep business logic separate from transport and persistence layers.
- Design for failure, not just the happy path.
- Make retryable operations idempotent.
- Define measurable SLOs before choosing infrastructure.
- Choose technology based on access patterns and requirements—not popularity.
- Treat security, monitoring, and cost as design requirements from day one.

Remember: a microservices architecture is not automatically more scalable or reliable than a well-designed modular monolith.

# Page 2 — Requirements and capacity estimation

## 1. Clarify requirements first

Ask these questions before drawing architecture:

- Who uses the system, and what are the primary user journeys?
- What are the read/write operations?
- What is the expected user count and peak traffic?
- What are the latency and availability targets?
- What data must never be lost?
- Is strong consistency required?
- What are the retention, security, compliance, and budget constraints?

Separate must-have requirements from optional features.

## 2. Capacity estimation formulas

| Metric | Formula |
| --- | --- |
| Average RPS | Daily requests ÷ 86,400 |
| Peak RPS | Average RPS × peak multiplier |
| Write RPS | Total RPS × write fraction |
| Storage | Records/day × average record size × retention days |
| Bandwidth | RPS × average response size |
| Concurrency | Throughput × average response time |
| Availability | Uptime ÷ total time |

RPS means requests per second. Concurrency is the average number of requests in flight under steady-state assumptions (Little's Law).

### Worked example

Assume a service handles 10 million requests/day, with a peak multiplier of 5.

- Average RPS = 10,000,000 ÷ 86,400 ≈ 116
- Peak RPS ≈ 116 × 5 = 580
- At 20% writes, peak write RPS ≈ 116
- At 50 KB per response, peak payload bandwidth ≈ 580 × 50 KB = 29 MB/s

This bandwidth estimate excludes protocol overhead, request bodies, and retransmissions.

For 2 million new records/day at 1 KB each:

- Raw storage/day ≈ 2 GB
- Raw storage/year ≈ 730 GB

Indexes, replicas, metadata, backups, and storage overhead increase the actual requirement.

## 3. Availability cheat sheet

| Availability | Approximate downtime/year |
| --- | --- |
| 99% | 3.65 days |
| 99.9% | 8.76 hours |
| 99.95% | 4.38 hours |
| 99.99% | 52.6 minutes |
| 99.999% | 5.26 minutes |

These are theoretical annual equivalents; actual SLOs depend on the measurement window and exclusions.

## 4. Latency budget

A user request may spend time in:

- DNS, connection setup, and TLS
- Load balancing and gateway processing
- Application execution
- Database and cache access
- Downstream service calls
- Serialization and network transfer

Rule: optimize the measured bottleneck. Average latency alone is insufficient; inspect p95 and p99 latency to understand slow requests.

## 5. Scaling options

- Vertical scaling: increase CPU, memory, or machine capacity.
- Horizontal scaling: add instances.
- Partitioning: divide data into logical or physical parts.
- Replication: maintain copies for availability or read scaling.
- Caching: avoid repeated expensive work.
- Asynchronous processing: remove noncritical work from request paths.

# Page 3 — Architecture, APIs, and service boundaries

## 1. Choose an architecture

| Architecture | Good fit | Main risk |
| --- | --- | --- |
| Modular monolith | Small teams, clear domain boundaries | Modules can become tightly coupled |
| Microservices | Independent scaling and ownership needs | Network failures and operational overhead |
| Event-driven | Asynchronous workflows and decoupling | Eventual consistency and debugging complexity |
| Serverless | Event-triggered, variable workloads | Runtime constraints and platform coupling |
| Layered architecture | Clear separation of concerns | Poorly enforced layers can become coupled |

Decision rule: start with a modular monolith unless independent deployment, scaling, ownership, or fault isolation justify separate services.

## 2. Example Spring Boot request flow

text
```
HTTP Request
    ↓
Controller → Validate input
    ↓
Service → Business rules / transaction
    ↓
Repository → PostgreSQL or MongoDB
    ↓
Response DTO → HTTP Response
```

Keep controllers thin. Do not put business rules in controllers or expose database entities directly as public API contracts.

## 3. API design checklist

- Use nouns for resource-oriented REST endpoints.
- Use HTTP methods consistently.
- Validate input at the boundary.
- Return meaningful status codes and stable error formats.
- Add pagination, filtering, and sorting for collections.
- Use API versioning when compatibility requires it.
- Apply authentication, authorization, and rate limits.
- Set timeouts and request-size limits.
- Use idempotency keys for operations where duplicate execution is dangerous.

Example:

http
```
POST /api/v1/orders
Idempotency-Key: 9d8f...
Content-Type: application/json
```

json
```
{
  "customerId": "c-101",
  "items": [
    {"productId": "p-42", "quantity": 2}
  ]
}
```

Possible responses:

- `201 Created`: order created.
- `400 Bad Request`: invalid input.
- `401 Unauthorized`: missing or invalid authentication.
- `403 Forbidden`: caller lacks permission.
- `409 Conflict`: conflicting business state.
- `429 Too Many Requests`: rate limit exceeded.
- `503 Service Unavailable`: temporary service unavailability.

## 4. Service communication

| REST/HTTP | Kafka events |
| --- | --- |
| Immediate response | Asynchronous processing |
| Caller depends on service availability | Producer and consumer are more decoupled |
| Useful for queries and immediate commands | Useful for notifications, analytics, and workflows |
| Latency includes downstream calls | Processing can lag behind production |

Use synchronous calls when the result is needed immediately. Use events when work can happen asynchronously.

## 5. Spring Boot layering

Typical responsibilities:

- `@RestController`: HTTP interface.
- `@Service`: application and business logic.
- `@Repository`: persistence abstraction.
- DTO/record: API request and response models.
- `@Configuration`: explicit configuration.
- `@Transactional`: transaction boundaries where supported by the selected transaction manager.

Be careful: Spring proxy-based transaction behavior can be affected by self-invocation, transaction manager selection, and whether work spans multiple resources.

# Page 4 — SQL, PostgreSQL, and MongoDB

## 1. Database selection cheat sheet

| Requirement | Typical starting choice |
| --- | --- |
| Orders, payments, inventory | PostgreSQL |
| Complex joins and relational reporting | PostgreSQL |
| Strong relational constraints and transactions | PostgreSQL |
| Flexible, nested document structures | MongoDB |
| Document-centric reads and writes | MongoDB |
| High-volume key-based lookups | Either, depending on access patterns |
| Very low-latency cache lookups | Redis |
| Durable event streaming | Kafka |

MongoDB supports transactions and schema validation; SQL databases can also handle JSON data. Choose based on data relationships, query patterns, consistency requirements, and operational needs.

## 2. SQL fundamentals

Know these concepts for interviews and production:

- Primary key: uniquely identifies a row.
- Foreign key: enforces a relationship between tables.
- Index: accelerates selected queries at storage and write costs.
- Normalization: reduces duplication and update anomalies.
- Denormalization: duplicates data deliberately for access efficiency.
- ACID: atomicity, consistency, isolation, durability.
- Transaction isolation: determines how concurrent transactions interact.
- Connection pool: reuses database connections; size it against database capacity.

### PostgreSQL example

sql
```
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status VARCHAR(30) NOT NULL,
    total_amount NUMERIC(12, 2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_orders_customer_created
    ON orders (customer_id, created_at DESC);
```

This index is suitable for queries filtering by `customer_id` and ordering by `created_at DESC`. Verify the actual plan with `EXPLAIN (ANALYZE, BUFFERS)` on representative data.

Index rules:

- Index frequently filtered, joined, or ordered columns when the workload justifies it.
- Match composite-index column order to common predicates and sort patterns.
- Avoid indexing every column: indexes consume space and increase write cost.
- Measure selectivity, query plans, and real workload behavior.

## 3. MongoDB fundamentals

MongoDB stores BSON documents inside collections.

json
```
{
  "_id": "order-1001",
  "customerId": "customer-101",
  "status": "PLACED",
  "items": [
    {"productId": "p-42", "quantity": 2, "unitPrice": 25.00}
  ],
  "totalAmount": 50.00
}
```

Example index:

javascript
```
db.orders.createIndex({
  customerId: 1,
  createdAt: -1
});
```

Useful MongoDB concepts:

- Embedding: store related data inside a document; useful for bounded, frequently read-together data.
- Referencing: store identifiers for separately managed or independently growing data.
- Aggregation pipeline: transform and group data through stages such as `$match`, `$group`, and `$project`.
- Schema validation: enforce document structure and field constraints.
- Sharding: distribute a collection across multiple shards using a shard key.
- Replication: maintain copies for resilience and eligible read scaling.

## 4. SQL vs. MongoDB decision checklist

Choose PostgreSQL when:

- Relationships and joins are central.
- Business invariants need relational constraints.
- Multi-row transactional updates are common.
- Reporting and flexible SQL queries are important.

Choose MongoDB when:

- The data naturally forms aggregate documents.
- Related fields are usually read together.
- Document structures need controlled flexibility.
- Queries align well with document and index design.

Use both only when the workload justifies the additional consistency, deployment, backup, monitoring, and operational complexity.

Critical warning: a database transaction does not automatically make a simultaneous update in PostgreSQL and MongoDB atomic. Cross-database workflows need explicit coordination, such as an outbox, saga, or reconciliation process.

# Page 5 — Caching, Redis, and performance

## 1. Why cache?

Caching reduces repeated work, database load, and response latency. It introduces freshness, invalidation, and consistency challenges.

### Cache-aside pattern

mermaid
```
sequenceDiagram
    participant C as Client
    participant A as Spring Boot
    participant R as Redis
    participant D as PostgreSQL
    C->>A: GET /products/42
    A->>R: GET product:42
    alt Cache hit
        R-->>A: Cached product
    else Cache miss
        R-->>A: Not found
        A->>D: SELECT product
        D-->>A: Product row
        A->>R: SET product:42 with TTL
    end
    A-->>C: Product response
```

## 2. Common cache strategies

| Strategy | Behavior | Consideration |
| --- | --- | --- |
| Cache-aside | Application loads and fills cache | Common default; misses hit database |
| Read-through | Cache layer loads missing data | Requires supporting cache abstraction |
| Write-through | Writes update cache and backing store through the strategy | Adds write-path coordination |
| Write-behind | Cache delays persistence | Risk of loss or delayed durability |
| Refresh-ahead | Refresh before expiry | Requires good refresh policy |

## 3. Cache problems

- Cache stampede: many requests reload the same expired key.
- Cache penetration: repeated misses for nonexistent keys.
- Cache avalanche: many keys expire or become unavailable together.
- Hot key: one key receives disproportionate traffic.
- Stale data: cached value no longer matches authoritative state.
- Memory pressure: eviction removes useful entries.

Mitigations include TTL jitter, request coalescing, bounded negative caching, hot-key replication or local caching where appropriate, and controlled fallback behavior.

## 4. Redis checklist

- Set explicit TTLs for expiring data.
- Define behavior when Redis is unavailable.
- Avoid caching sensitive data without appropriate protections.
- Monitor hit rate, memory, evictions, latency, and connection usage.
- Prevent unbounded key growth.
- Use atomic Redis operations for counters and token-bucket logic.
- Do not treat an ordinary cache as the sole source of truth for critical durable business data.

## 5. Spring Boot example

Using Spring's cache abstraction:

java
```
@Service
public class ProductService {
    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    @Cacheable(cacheNames = "products", key = "#id")
    public ProductDto findById(Long id) {
        return repository.findById(id)
            .map(ProductDto::from)
            .orElseThrow(() ->
                new NoSuchElementException("Product not found"));
    }

    @CacheEvict(cacheNames = "products", key = "#id")
    public void evictProduct(Long id) {
        // Invoke after a successful update when appropriate.
    }
}
```

This illustrates the annotations, not a complete application. Configure Redis, cache TTLs, serialization, and cache eviction carefully. Invalidation must be connected to the actual update workflow.

## 6. Performance checklist

- Measure p50, p95, and p99 latency.
- Inspect database query plans and connection pools.
- Use pagination rather than unbounded result sets.
- Avoid N+1 queries.
- Bound queues, request sizes, and concurrent work.
- Profile before optimizing.
- Load-test expected peak traffic and failure scenarios.

# Page 6 — Distributed systems, Kafka, and resilience

## 1. Distributed systems fundamentals

A distributed system consists of components communicating over a network. Networks can delay, drop, duplicate, or reorder messages, and components can fail independently.

Important concepts:

- Timeout: bounds how long a caller waits.
- Retry: repeats an operation after a failure.
- Idempotency: repeating an operation has the same intended effect as performing it once.
- Eventual consistency: replicas may temporarily differ but converge under stated conditions.
- Consensus: nodes agree on a value or decision under a specified failure model.
- Backpressure: slows or rejects incoming work when downstream capacity is insufficient.
- Bulkhead: isolates resources so one failure does not exhaust everything.
- Circuit breaker: temporarily stops calls to an unhealthy dependency.

### Retry rule

Use bounded retries with exponential backoff and jitter. Retry only failures likely to be transient. Retrying non-idempotent operations without deduplication can create duplicate orders or payments.

## 2. Kafka essentials

| Concept | Meaning |
| --- | --- |
| Topic | Named event stream |
| Partition | Ordered log within a topic |
| Offset | Position in a partition |
| Consumer group | Consumers sharing work |
| Retention | How long records remain available |
| Consumer lag | Difference between produced and consumed progress |
| Dead-letter topic | Destination for messages that cannot be processed normally |

Kafka preserves order within a partition, not automatically across all partitions. A key such as `orderId` can keep related events on the same partition.

### Event-driven architecture

mermaid
```
flowchart LR
    A[Order Service] --> DB[(PostgreSQL)]
    A --> O[Outbox Table]
    O --> P[Outbox Publisher]
    P --> K[Kafka Topic]
    K --> N[Notification Consumer]
    K --> I[Inventory Consumer]
    K --> X[Analytics Consumer]
```

## 3. The dual-write problem

Suppose an application commits an order to PostgreSQL and separately publishes an event to Kafka. Either operation can succeed while the other fails.

Transactional outbox pattern:

1. Save the business change and an outbox record in the same database transaction.
2. A publisher or CDC mechanism forwards the outbox event to Kafka.
3. Consumers deduplicate or idempotently process events.
4. Track failures and retry safely.

The publisher may send the same event more than once. Consumers must tolerate duplicates.

## 4. Delivery guarantees

- At-most-once: a message may be lost, but is not redelivered under the stated delivery semantics.
- At-least-once: messages can be redelivered; duplicates must be handled.
- Exactly-once: always qualify the scope. Kafka can support exactly-once processing within supported Kafka workflows, but arbitrary external database side effects require additional coordination.

## 5. Failure-handling checklist

- Set deadlines and timeouts.
- Bound retries and use jitter.
- Limit concurrency and queue depth.
- Use dead-letter handling with replay procedures.
- Make consumers idempotent.
- Monitor consumer lag and processing failures.
- Design graceful degradation for noncritical dependencies.
- Test dependency outages and recovery.

# Page 7 — Load balancing, scaling, and availability

## 1. Load balancing

A load balancer distributes traffic among service instances.

- Layer 4: routes using transport-level information.
- Layer 7: routes using HTTP-level information, such as host, path, or headers.
- Round-robin: distributes requests in rotation.
- Least-connections: favors instances with fewer active connections.
- Health checks: stop routing to instances considered unhealthy.
- Sticky sessions: keep a client associated with an instance; avoid requiring them when stateless design is practical.

## 2. Horizontal scaling checklist

Before adding more instances, check:

- Is the application stateless?
- Are sessions stored in shared storage if required?
- Can the database support additional traffic?
- Are there connection-pool limits?
- Is there a shared bottleneck such as Redis or Kafka?
- Does autoscaling respond quickly enough to demand?
- Can the system handle partial capacity during failures?

Adding application servers does not help if the database is already saturated.

## 3. Replication vs. sharding

| Replication | Sharding |
| --- | --- |
| Copies data across nodes | Splits data across nodes |
| Supports resilience and potentially more reads | Distributes storage and workload |
| Can introduce replication lag | Introduces routing and rebalancing complexity |
| Requires failover and consistency planning | Requires shard-key and hot-partition planning |

A system can use both.

## 4. Availability and dependency chains

For independent serial dependencies, all of which must succeed, approximate availability is the product of their availabilities.

Example: three mandatory dependencies each have 99.9% availability.

$$
0.999^3 \approx 0.997
$$

Combined availability is approximately 99.70%, assuming independence and identical measurement conditions.

Real failures are often correlated, so this is a simplified model.

## 5. Multi-region design

Choose based on recovery and business requirements:

- Single region, multiple zones: a common starting point for resilience.
- Active-passive regions: one primary region with a recovery region.
- Active-active regions: multiple regions serve traffic; data consistency and conflict resolution become more complex.

Define:

- RTO: target time to restore service.
- RPO: target maximum tolerable data loss measured in time.
- Failover triggers and ownership.
- Replication lag limits.
- Data residency requirements.
- Recovery testing frequency.

Production rule: backups are not a disaster-recovery strategy until restoration has been tested.

# Page 8 — Security, observability, deployment, and cost

## 1. Security checklist

- Authenticate users and services.
- Authorize each protected operation.
- Apply least privilege to applications and database accounts.
- Use TLS for network communication.
- Encrypt sensitive data at rest where required.
- Store secrets in an appropriate secret-management system.
- Validate input and encode output appropriately.
- Apply rate limits and request-size limits.
- Protect against injection, broken access control, and credential abuse.
- Audit sensitive actions.
- Define data retention and deletion policies.
- Never log passwords, access tokens, or unnecessary personal data.

### Authentication vs. authorization

- Authentication: Who are you?
- Authorization: What are you allowed to do?

OAuth 2.0 is an authorization framework. OpenID Connect adds an identity layer. Use well-supported libraries and correctly validate tokens, issuers, audiences, expiry, and permissions.

## 2. Observability

Three core pillars:

- Logs: event details and errors.
- Metrics: numeric measurements over time.
- Traces: request flow across services.

Monitor the RED signals for request-driven services:

- Rate: requests per second.
- Errors: failed requests or operations.
- Duration: request latency distribution.

For infrastructure, USE is useful:

- Utilization: how busy a resource is.
- Saturation: queued or waiting work.
- Errors: failed operations.

Add business metrics such as checkout success rate and order-processing delay.

### Essential dashboards

- API throughput and p95/p99 latency.
- Error rate and timeout rate.
- JVM heap, garbage collection, and thread pools.
- Database connections, query latency, locks, and replication lag.
- Redis hit rate, memory, evictions, and latency.
- Kafka consumer lag and failed processing.
- Queue depth, retries, and dead-letter volume.

Use correlation IDs and distributed traces to connect related operations. Avoid high-cardinality metric labels such as unique user IDs.

## 3. Deployment strategy

| Strategy | Main benefit | Main concern |
| --- | --- | --- |
| Rolling deployment | Gradual replacement | Mixed versions coexist |
| Blue-green | Fast traffic switch and rollback | Extra capacity required |
| Canary | Limits exposure to new release | Needs metrics and traffic control |

Deployment checklist:

- Health and readiness checks.
- Resource requests and limits.
- Externalized configuration.
- Safe database migrations.
- Backward-compatible APIs and event schemas.
- Automated tests and security scanning.
- Rollback procedure.
- Alerts and post-deployment validation.

## 4. Cost optimization

- Right-size compute based on observed utilization.
- Reduce unnecessary data transfer and duplicate storage.
- Tune database queries before scaling hardware.
- Set retention and lifecycle policies.
- Choose managed services when their operational benefits justify the cost.
- Evaluate observability cost and metric cardinality.
- Load-test cost at expected and peak demand.

Key principle: optimize cost per successful business operation, not just the monthly infrastructure bill.

# Page 9 — Twelve system design case studies

Use this page as a quick architecture map. For any case study, expand the requirements, capacity estimates, API, data model, failure modes, and trade-offs before committing to a design.

Design a URL Shortener - System Design Interview

1. URL shortener

PostgreSQL for link ownership and metadata; Redis for hot redirects; unique ID generation; cache invalidation; abuse protection; click analytics through Kafka.

Rate Limiter System Design (Token Bucket Explained) · InterviewCrafted

2. Rate limiter

Token bucket or sliding window; atomic Redis operations; keys by user/API key; distributed enforcement; fail-open vs. fail-closed policy; `429` responses.

I Failed Four System Design Rounds Before I Understood What They Were Actually Asking. | by Let's Talk Basics | Apr, 2026 | Medium

3. Notification service

Kafka or queues; channel workers; templates; user preferences; provider retries; deduplication; delivery status and dead-letter handling.

Top 10 WebSocket Use Cases in System Design

4. Chat application

WebSockets; persistent message storage; per-conversation ordering; online presence; offline delivery; reconnects; unread counts and fan-out.

How I Designed a Scalable Twitter Feed | by Gyanaa Vaibhav | Medium

5. News feed

Fan-out on write for many ordinary users; fan-out on read for high-follower accounts; ranking; timeline caches; pagination and feed freshness.

Design YouTube - System Design Interview

6. Video streaming

Object storage; asynchronous transcoding; multiple bitrates; manifests; CDN delivery; signed URLs; metadata and content lifecycle.

The diagram shows the core architecture of a distributed file storage system like Google Drive. Clients connect through a load balancer to frontend APIs, which route metadata requests to a sharded… | Samriddha Chattopadhyay

7. Cloud file storage

Object storage for file contents; SQL metadata and permissions; multipart uploads; versioning; synchronization; deduplication where worthwhile.

Amazon-Style E-Commerce System Design: Cart, Orders, Payments, Scalability | by DevLogic - Engineering Thinking | Medium

8. E-commerce

Product catalog; search; cart; inventory reservations; order lifecycle; payment integration; idempotent checkout; transactional workflows and reconciliation.

Uber's Real-Time Dispatch Architecture — Architecture Diagram | Diagrams.so

9. Ride-hailing / delivery

Geospatial indexes; location updates; nearby-driver matching; dispatch; trip lifecycle; pricing; real-time messaging and regional partitioning.

PPT - Search Engines PowerPoint Presentation, free download - ID:351956

10. Search engine

Crawlers; ingestion queues; inverted indexes; ranking; query serving; index freshness; sharding; caching and relevance measurement.

Design Instagram - System Design Interview

11. Photo/video social feed

Upload pipeline; media transformations; object storage; metadata; moderation; feeds; CDN; asynchronous processing and fan-out.

Architecting Multi-Region SaaS Solutions on AWS | AWS Partner Network (APN) Blog

12. Multi-region SaaS

Tenant isolation; regional routing; quotas; regional data placement; failover; replication; data residency and recovery objectives.

# Page 10 — Interview cheat sheet and Spring Boot implementation

## 1. The 45–60 minute interview framework

1. Requirements — 5 minutes: clarify users, features, traffic, latency, consistency, and scope.
2. Estimation — 5 minutes: calculate RPS, storage, bandwidth, and peak demand.
3. API and data model — 5–10 minutes: define key endpoints, entities, relationships, and access patterns.
4. High-level architecture — 10 minutes: draw services, databases, cache, queues, and request flows.
5. Deep dive — 15–20 minutes: investigate bottlenecks, consistency, scaling, and failure recovery.
6. Trade-offs and summary — 5 minutes: explain alternatives, risks, security, and next steps.

## 2. Questions you must be able to answer

| Interview question | What a strong answer covers |
| --- | --- |
| Why microservices? | Independent ownership, deployment, scaling, and the operational costs |
| SQL or MongoDB? | Relationships, transactions, access patterns, schema, and consistency |
| Why Redis? | Repeated reads, latency reduction, TTLs, invalidation, failure policy |
| How do you scale reads? | Indexes, caching, replicas, partitioning, and query optimization |
| How do you scale writes? | Write-path optimization, partitioning, queues, batching, contention |
| What happens if Kafka is down? | Persistence, producer behavior, retries, buffering limits, recovery |
| How do you prevent duplicate payments? | Idempotency, durable operation records, provider semantics, reconciliation |
| How do you prevent data loss? | Durability settings, backups, replication, tested recovery |
| How do you debug high latency? | Traces, p95/p99, database plans, saturation, downstream calls |
| How do you handle traffic spikes? | Capacity headroom, autoscaling, caching, load shedding, queues |

## 3. Spring Boot project skeleton

A reasonable starting structure for the Shelf marketplace:

text
```
shelf/
├── controller/
├── service/
├── repository/
├── dto/
├── domain/
├── config/
├── exception/
├── messaging/
├── security/
└── observability/
```

Example REST controller:

java
```
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public ProductDto getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }
}
```

The controller delegates business logic to the service. The service handles rules, the repository accesses the database, and the DTO defines the API response.

For a production application, add validation, centralized exception handling, authorization, observability, tests, and appropriate API documentation.

## 4. Common architecture mistakes

- Starting with microservices before identifying a real need.
- Choosing MongoDB because it is NoSQL, without analyzing queries and relationships.
- Adding Redis without defining TTL and invalidation behavior.
- Assuming Kafka automatically guarantees end-to-end exactly-once business processing.
- Retrying requests without considering idempotency.
- Using synchronous calls across too many services.
- Ignoring database connection limits and slow queries.
- Treating replicas as backups.
- Claiming high availability without defining failure scenarios.
- Using a distributed transaction when a simpler workflow would suffice.
- Optimizing average latency while ignoring p99.
- Adding infrastructure without measuring the bottleneck.

## 5. Final architecture review checklist

Before approving a design, verify:

- [ ]Functional and non-functional requirements are measurable.
- [ ]Traffic, storage, and capacity estimates have explicit assumptions.
- [ ]APIs and data models support the main access patterns.
- [ ]Database choices and indexes are justified.
- [ ]Authentication and authorization are defined.
- [ ]Timeouts, retries, idempotency, and failure handling are covered.
- [ ]Queues have capacity limits and recovery procedures.
- [ ]Logs, metrics, traces, and alerts are defined.
- [ ]Backups and disaster recovery are tested.
- [ ]Scaling limits and operational costs are understood.
- [ ]Architectural trade-offs are documented.

## 6. Last-minute revision formulas

| Concept | Quick reference |
| --- | --- |
| Average RPS | Daily requests ÷ 86,400 |
| Concurrency | Throughput × average latency in seconds |
| Storage | Record count × average record size |
| Bandwidth | Requests/second × bytes/response |
| Availability of serial dependencies | Multiply availabilities, under independence assumptions |
| Cache hit ratio | Cache hits ÷ total cache lookups |
| Error budget | Allowed failure fraction × eligible request volume or time |
| Little's Law | $L = \lambda W$ |

## 7. Recommended official references

Use these to deepen your understanding and verify implementation details:

- Spring Boot — [Official documentation](<https://docs.spring.io/spring-boot/index.html>)
- PostgreSQL — [Official documentation](<https://www.postgresql.org/docs/>)
- MongoDB — [Official documentation](<https://www.mongodb.com/docs/>)
- Redis — [Official documentation](<https://redis.io/docs/>)
- Apache Kafka — [Official documentation](<https://kafka.apache.org/documentation/>)
- OWASP — [OWASP Top 10](<https://owasp.org/www-project-top-ten/>)
- Google SRE — [Books and resources](<https://sre.google/books/>)

## How to use this cheat sheet

- Before an interview: revise Pages 2, 4, 6, 7, and 10.
- While designing a system: use Pages 1–8 as architecture review references.
- For implementation: use Java/Spring Boot with the appropriate database and messaging patterns.
- For deeper study: expand each of the 12 case studies into a complete design exercise.

This is the compact, 10-section edition. It covers the main concepts, but it is a revision aid rather than a substitute for the deeper implementation labs and detailed case-study solutions.

