# CODING-STANDARDS.md

# Java 25 & Spring Boot 4 Engineering Standards

## Version

| Component        | Version          |
| ---------------- | ---------------- |
| Java             | 25+              |
| Spring Boot      | 4.x              |
| Spring Framework | 7.x              |
| Jakarta EE       | 11               |
| Gradle           | 9+               |
| JUnit            | 5                |
| MapStruct        | Latest Stable    |
| Testcontainers   | Latest Stable    |
| Lombok           | Restricted Usage |

---

# 1. Purpose

This document defines the mandatory engineering standards for all Java 25 and Spring Boot 4 projects.

The objectives are:

* Maintainability
* Readability
* Reliability
* Security
* Testability
* Performance
* Observability
* Consistency

All code must be written for long-term maintenance rather than short-term delivery.

---

# 2. Engineering Principles

## 2.1 Readability Over Cleverness

Code is read significantly more often than it is written.

Prefer:

```java
if (customer.isEligible()) {
    applyDiscount(customer);
}
```

Over:

```java
Optional.of(customer)
        .filter(Customer::isEligible)
        .ifPresent(this::applyDiscount);
```

unless the latter is clearly more expressive.

---

## 2.2 Simplicity Over Abstraction

Do not introduce abstractions until there is a demonstrated need.

Avoid:

* Premature generic frameworks
* Unnecessary interfaces
* Deep inheritance hierarchies

Prefer straightforward solutions.

---

## 2.3 Explicit Over Implicit

Code behavior should be obvious.

Preferred:

* Explicit configuration
* Explicit mappings
* Explicit dependencies

Avoid:

* Reflection-heavy frameworks
* Hidden side effects
* Magic behavior

---

## 2.4 Immutability By Default

All objects should be immutable unless mutation is required.

Prefer:

```java
public record CustomerResponse(
        UUID id,
        String name
) {}
```

over mutable DTOs.

---

# 3. Project Structure

Projects must be organized by business capability.

Preferred:

```text
customer
├── api
├── application
├── domain
├── infrastructure
└── mapper

order
├── api
├── application
├── domain
├── infrastructure
└── mapper
```

Avoid:

```text
controller
service
repository
entity
dto
mapper
```

for medium and large applications.

---

# 4. Dependency Rules

Dependencies must flow inward.

```text
API
 ↓
Application
 ↓
Domain
 ↓
Infrastructure
```

Rules:

* Controllers depend on services.
* Services depend on domain abstractions.
* Infrastructure implements domain contracts.
* Domain must not depend on Spring.

Forbidden:

* Circular dependencies
* Repository-to-controller dependencies
* Domain-to-framework dependencies

---

# 5. Java Language Standards

## Records

Records are mandatory for:

* API requests
* API responses
* Events
* Value objects
* Query projections

Example:

```java
public record CustomerResponse(
        UUID id,
        String name
) {}
```

---

## Sealed Classes

Use sealed hierarchies when variants are finite.

```java
public sealed interface PaymentResult
        permits Success, Failure {
}
```

---

## Pattern Matching

Prefer pattern matching over casting.

```java
switch (result) {
    case Success s -> process(s);
    case Failure f -> process(f);
}
```

---

## Optional

Use Optional only for return values.

Allowed:

```java
Optional<Customer> findById(UUID id);
```

Forbidden:

```java
Optional<Customer> customer;
void save(Optional<Customer> customer);
List<Optional<Customer>>
```

---

## Streams

Use streams when readability improves.

Avoid deeply nested pipelines.

Prefer loops when they are clearer.

---

# 6. Lombok Policy

Lombok usage is intentionally restricted.

## Allowed

```java
@RequiredArgsConstructor
@Getter
@Builder
@Slf4j
```

## Discouraged

```java
@Data
@Setter
@AllArgsConstructor
@NoArgsConstructor
```

unless specifically required.

---

## Records Preferred

Prefer:

```java
public record CustomerResponse(
        UUID id,
        String name
) {}
```

Over:

```java
@Data
@Builder
public class CustomerResponse {
}
```

---

## Entities

Never use:

```java
@Data
@Entity
```

Use:

```java
@Getter
@NoArgsConstructor
@Entity
```

instead.

---

# 7. Spring Boot Standards

## Constructor Injection

Constructor injection is mandatory.

Preferred:

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerRepository repository;

}
```

Forbidden:

```java
@Autowired
private CustomerRepository repository;
```

---

## Configuration Properties

Use immutable configuration.

Preferred:

```java
@ConfigurationProperties("customer")
public record CustomerProperties(
        Duration timeout
) {}
```

Avoid mutable configuration classes.

---

## Validation

Use Jakarta Validation.

```java
public record CreateCustomerRequest(

        @NotBlank String name,

        @Email String email
) {
}
```

Never manually validate fields that can be validated declaratively.

---

## Transactions

Transactions belong in application services.

Allowed:

```java
@Transactional
public void createOrder() {
}
```

Forbidden:

* Transactional controllers
* Transactional repositories
* Transactional utility classes

---

## HTTP Clients

Preferred:

```java
RestClient
```

Reactive workloads:

```java
WebClient
```

Forbidden:

```java
RestTemplate
```

for new development.

---

# 8. Controller Standards

Controllers must remain thin.

Responsibilities:

* Request mapping
* Validation
* Security context extraction
* Service invocation

Controllers must not contain:

* Business logic
* Persistence logic
* Mapping logic
* Transaction logic

---

# 9. Service Standards

Services contain business logic.

Responsibilities:

* Use case execution
* Validation beyond DTO constraints
* Transaction management
* Domain orchestration

Services should not:

* Build HTTP responses
* Execute SQL
* Parse JSON

---

# 10. Repository Standards

Repositories handle persistence only.

Example:

```java
@Repository
public interface CustomerRepository
        extends JpaRepository<Customer, UUID> {
}
```

Business rules do not belong in repositories.

---

# 11. Mapping Standards

MapStruct is mandatory.

```java
@Mapper(
    componentModel = MappingConstants.ComponentModel.SPRING
)
public interface CustomerMapper {
}
```

Forbidden:

* Reflection mappers
* Repeated manual mapping

---

# 12. Exception Handling

Use centralized exception handling.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
}
```

API errors must use Problem Details (RFC 9457).

Example:

```json
{
  "type": "customer-not-found",
  "title": "Customer not found",
  "status": 404
}
```

---

# 13. Database Standards

## Migrations

All schema changes must be versioned.

Required:

* Flyway

or

* Liquibase

Manual schema changes are prohibited.

---

## Naming

Tables:

```sql
customer
customer_order
payment_transaction
```

Columns:

```sql
created_at
updated_at
customer_id
```

---

## Auditing

Entities should contain:

```java
Instant createdAt;
Instant updatedAt;
```

where appropriate.

---

# 14. Security Standards

## Secrets

Secrets must never be stored in source control.

Use:

* Vault
* AWS Secrets Manager
* Azure Key Vault
* GCP Secret Manager

---

## Validation

All external input must be validated.

Includes:

* REST requests
* Kafka messages
* File uploads
* External APIs

---

## Least Privilege

Applications must operate with the minimum permissions required.

---

# 15. Observability Standards

Observability is mandatory.

Every service must provide:

* Metrics
* Logs
* Health checks
* Traces

---

## Logging

Use structured logging.

```java
log.info(
    "Order processed. orderId={} customerId={}",
    orderId,
    customerId
);
```

Never log:

* Passwords
* Tokens
* Secrets
* Personal information

---

## Metrics

Expose metrics for critical business operations.

Examples:

* Orders created
* Payments processed
* Login failures

---

## Tracing

Distributed tracing is required for:

* HTTP calls
* Messaging
* Service-to-service communication

---

# 16. Virtual Threads

Virtual threads are enabled by default.

Configuration:

```properties
spring.threads.virtual.enabled=true
```

Use platform threads only when justified by profiling.

Never create threads manually.

Forbidden:

```java
new Thread(...)
```

---

# 17. Testing Standards

## Testing Pyramid

Target distribution:

```text
70% Unit Tests
20% Integration Tests
10% End-to-End Tests
```

---

## Unit Tests

Required for business logic.

Preferred:

```java
@ExtendWith(MockitoExtension.class)
class CustomerServiceTest {
}
```

---

## Integration Tests

Use:

```java
@SpringBootTest
```

for integration testing.

---

## Testcontainers

Required for:

* PostgreSQL
* Kafka
* Redis
* Elasticsearch

Avoid embedded alternatives.

---

## Deterministic Tests

Do not depend on:

* Current time
* Random values
* Shared state

Inject:

```java
Clock
```

instead of using:

```java
Instant.now()
```

directly.

---

# 18. Performance Standards

Performance optimization requires evidence.

Use:

* Java Flight Recorder
* Async Profiler
* JMH

before making optimization decisions.

---

## Database Performance

Prevent N+1 queries.

Review generated SQL.

Use:

* Fetch joins
* Entity graphs
* Projections

when appropriate.

---

## Caching

Caching requires:

* Documented TTL
* Documented invalidation strategy
* Measured benefit

---

# 19. CI/CD Standards

Every pull request must:

* Compile successfully
* Pass all tests
* Pass static analysis
* Pass security scanning
* Pass coverage requirements

---

## Branch Protection

Protected branches:

```text
main
release/*
```

Require:

* Approved review
* Successful CI

---

# 20. Tooling Standards

## Build

Preferred:

```text
Gradle 9+
```

---

## Formatting

Required:

```text
Spotless
Google Java Format
```

Formatting is enforced automatically.

---

## Static Analysis

Required:

```text
Checkstyle
SpotBugs
PMD
Error Prone
```

Build failures are mandatory for critical violations.

---

## Dependency Management

Required:

```text
Dependabot
Renovate
OWASP Dependency Check
```

---

## API Documentation

Required:

```text
springdoc-openapi
```

OpenAPI specifications must be generated automatically.

---

# 21. Documentation Standards

Every service must provide:

* README
* OpenAPI specification
* Architecture overview
* Deployment instructions
* Environment variables

---

## Architecture Decision Records

Significant architectural decisions require ADRs.

Format:

```text
Context
Decision
Alternatives
Consequences
```

---

# 22. Non-Negotiable Rules

The following rules are mandatory:

1. No field injection.
2. No `@Data` on entities.
3. No business logic in controllers.
4. No direct database schema modifications.
5. No secrets in source control.
6. No circular dependencies.
7. No untested business logic.
8. No manual thread creation.
9. No logging of sensitive information.
10. No bypassing CI quality gates.
11. No merging without code review.
12. No disabling security controls without documented approval.
13. No reflection-based object mapping.
14. No mutable configuration properties.
15. No framework dependencies inside the domain layer.

---

# 23. Golden Rules

When faced with multiple implementation choices, prefer:

1. Simplicity over cleverness.
2. Readability over brevity.
3. Immutability over mutation.
4. Explicitness over magic.
5. Records over Lombok DTOs.
6. Constructor injection over field injection.
7. MapStruct over manual repetitive mapping.
8. Virtual threads over manual concurrency.
9. Measured optimization over assumptions.
10. Long-term maintainability over short-term convenience.

> Write code as if the next engineer maintaining it is highly skilled, unfamiliar with the project, and responsible for operating it in production.
