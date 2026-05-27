# XYZ E-Commerce Platform — Spring Boot Microservices

## Project Structure

```
ecommerce-platform/                  ← Parent Maven project
├── common-library/                  ← Shared: AppLogger, ApiResponse, Exceptions
├── user-service/          :8081     ← User CRUD, Feign + RestTemplate inter-service calls
├── card-validation-service/:8082    ← Luhn validation, card masking
├── order-service/         :8083     ← Multi-threading + Observer pattern
├── payment-service/       :8084     ← Payment processing, refunds
└── product-catalog-service/:8085   ← Product CRUD, stock management
```

---

## Prerequisites

| Tool       | Version  |
|------------|----------|
| Java       | 17+      |
| Maven      | 3.8+     |

---

## Build & Run

### Step 1 — Install common-library to local .m2

```bash
cd ecommerce-platform
mvn install -pl common-library
```

### Step 2 — Build entire platform (skip tests for speed)

```bash
mvn clean install -DskipTests
```

### Step 3 — Run each service (separate terminals)

```bash
# Terminal 1
cd user-service && mvn spring-boot:run

# Terminal 2
cd card-validation-service && mvn spring-boot:run

# Terminal 3
cd order-service && mvn spring-boot:run

# Terminal 4
cd payment-service && mvn spring-boot:run

# Terminal 5
cd product-catalog-service && mvn spring-boot:run
```

### Step 4 — Run all tests with coverage report

```bash
# From root — runs all tests + JaCoCo 80% coverage check
mvn clean test

# Per-service coverage report (opens in browser)
open user-service/target/site/jacoco/index.html
```

---

## H2 Console (in-memory DB browser)

| Service                | URL                                       | JDBC URL                    |
|------------------------|-------------------------------------------|-----------------------------|
| user-service           | http://localhost:8081/h2-console          | jdbc:h2:mem:userdb          |
| card-validation-service| http://localhost:8082/h2-console          | jdbc:h2:mem:carddb          |
| order-service          | http://localhost:8083/h2-console          | jdbc:h2:mem:orderdb         |
| payment-service        | http://localhost:8084/h2-console          | jdbc:h2:mem:paymentdb       |
| product-catalog-service| http://localhost:8085/h2-console          | jdbc:h2:mem:productdb       |

Username: `sa` | Password: `password`

---

## Sample API Calls

### User Service (port 8081)

```bash
# Create user
curl -X POST http://localhost:8081/api/v1/users \
  -H "Content-Type: application/json" \
  -d '{"firstName":"John","lastName":"Doe","email":"john@example.com","password":"secret123"}'

# Get user by ID
curl http://localhost:8081/api/v1/users/1

# Get all users
curl http://localhost:8081/api/v1/users

# Update user
curl -X PUT http://localhost:8081/api/v1/users/1 \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Johnny","lastName":"Doe","email":"john@example.com","password":"secret123"}'

# Delete user
curl -X DELETE http://localhost:8081/api/v1/users/1
```

### Card Validation Service (port 8082)

```bash
# Add a card
curl -X POST http://localhost:8082/api/v1/cards \
  -H "Content-Type: application/json" \
  -d '{"cardNumber":"4111111111111111","cardHolderName":"John Doe","expiryMonth":12,"expiryYear":2028,"userId":1}'

# Get card details (masked)
curl http://localhost:8082/api/v1/cards/4111111111111111

# Validate card (Luhn + expiry check)
curl -X POST http://localhost:8082/api/v1/cards/validate/4111111111111111
```

### Product Catalog Service (port 8085)

```bash
# Create product
curl -X POST http://localhost:8085/api/v1/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Laptop","description":"Gaming Laptop","price":999.99,"stockQuantity":10,"category":"Electronics"}'

# Get all products
curl http://localhost:8085/api/v1/products

# Search products
curl "http://localhost:8085/api/v1/products/search?name=lap"
```

### Order Service (port 8083)

```bash
# Create order (triggers main thread + child thread for notification)
curl -X POST http://localhost:8083/api/v1/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":1,"productId":1,"quantity":2,"totalAmount":199.98,"cardNumber":"4111111111111111"}'

# Get order
curl http://localhost:8083/api/v1/orders/1

# Get orders by user
curl http://localhost:8083/api/v1/orders/user/1

# Update status
curl -X PATCH "http://localhost:8083/api/v1/orders/1/status?status=SHIPPED"
```

### Payment Service (port 8084)

```bash
# Process payment
curl -X POST http://localhost:8084/api/v1/payments \
  -H "Content-Type: application/json" \
  -d '{"orderId":1,"userId":1,"amount":199.98,"cardNumber":"4111111111111111"}'

# Get payment
curl http://localhost:8084/api/v1/payments/1

# Refund
curl -X POST http://localhost:8084/api/v1/payments/1/refund
```

---

## Concepts Covered

| Concept                   | Where                                           |
|---------------------------|-------------------------------------------------|
| Builder Pattern           | `UserDTO`, `ApiResponse`, all entities          |
| DAO / Repository Pattern  | All `*Repository` interfaces                    |
| Observer Pattern          | `PaymentObserver` → `CustomerNotificationObserver` |
| MVC Pattern               | All Controller → Service → DAO layers           |
| IoC / DI                  | `@RequiredArgsConstructor` everywhere           |
| Multi-threading           | `OrderService` + `PaymentStatusCheckerThread`   |
| Feign Client              | `CardValidationFeignClient` in user-service     |
| RestTemplate              | `CardIntegrationService` in user-service        |
| @Value Config Management  | `UserService`, `ProductService`                 |
| JPA One-to-Many           | `User` ↔ `Address`                             |
| Transaction Management    | `@Transactional` on all service methods         |
| Java Reflection           | Tests for `isValidEmail`, `validateLuhn`, `isExpired`, `detectCardType`, `simulatePaymentCheck`, `processWithGateway` |
| JUnit 5 + Mockito         | All test packages (model, dao, service, controller) |
| JaCoCo Coverage ≥ 80%     | Parent POM build plugin                         |
| Common Library reuse      | `AppLogger`, `ApiResponse` shared via .m2       |
