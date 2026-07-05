# Subscription Feature - Jira Tickets

**Project:** ankurray.com Subscription Feature
**Stack:** Java/Spring Boot 3.x + PostgreSQL + Stripe
**Timeline:** 8 Weekends
**Total Tickets:** 18

---

## Epic 1: Backend Infrastructure & Authentication (Weekends 1-2)

### SETUP-1: Spring Boot Project Scaffold
**Type:** Setup/Task
**Story Points:** 5
**Sprint:** 1
**Branch:** `feature/setup-spring-boot` → `Subscription`
**Effort:** 4-6 hours

**Description:**
Create a new Spring Boot 3.x project with complete initial configuration.

**User Story:**
As a developer, I want a properly structured Spring Boot project, so that I have a solid foundation for building the API.

**Acceptance Criteria:**
- [ ] Spring Boot 3.x project created with Maven build system
- [ ] Dependencies added: Spring Web, Spring Data JPA, PostgreSQL/MySQL driver, Spring Security, Stripe SDK
- [ ] Package structure created: `com.ankurray.{config, controller, service, repository, model, dto, security, exception}`
- [ ] H2 database configured for development/testing
- [ ] Main `Application.java` entry point runs without errors
- [ ] Basic health check endpoint `GET /api/health` returns 200 OK
- [ ] `application.properties` configured for dev environment
- [ ] `.gitignore` includes typical Java/Maven ignores (target/, .classpath, .project, etc.)

**Technical Notes:**
- Use Spring Boot Starter dependencies
- Configure profiles for dev/prod
- Set up logging with SLF4J
- Add Swagger/OpenAPI dependency for API documentation (optional for later)

**Testing:**
```bash
mvn spring-boot:run
# Should start on port 8080 without errors
curl http://localhost:8080/api/health
# Should return HTTP 200
```

---

### SETUP-2: Database Schema & Flyway Migrations
**Type:** Task
**Story Points:** 4
**Sprint:** 1
**Branch:** `feature/database-schema` → `Subscription`
**Effort:** 3-4 hours
**Depends on:** SETUP-1

**Description:**
Design and create database migrations for all required tables using Flyway.

**User Story:**
As a developer, I want a properly structured database schema, so that data integrity is maintained across the application.

**Acceptance Criteria:**
- [ ] Flyway configured in Spring Boot (dependency + configuration)
- [ ] Database migrations created:
  - `V1__create_users_table.sql` - users with id, email, password_hash, first_name, last_name, role, is_active, timestamps
  - `V2__create_articles_table.sql` - articles with id, title, slug, content, category, author_id FK, is_premium, timestamps
  - `V3__create_subscriptions_table.sql` - subscriptions with id, user_id FK, status, tier, dates, stripe_id, timestamps
  - `V4__create_payments_table.sql` - payments with id, user_id FK, subscription_id FK, amount, currency, status, stripe_id, timestamp
- [ ] Foreign keys properly configured with ON DELETE CASCADE where appropriate
- [ ] Unique constraints on email (users), slug (articles)
- [ ] Indexes created on frequently queried columns: users.email, articles.slug, subscriptions.user_id, payments.stripe_payment_id
- [ ] All tables have created_at and updated_at timestamps
- [ ] Migrations can be run and rolled back cleanly
- [ ] PostgreSQL driver configured in `application.properties`

**Technical Notes:**
- Migrations go in `src/main/resources/db/migration/`
- Follow Flyway naming: `V{version}__{description}.sql`
- Use lowercase table names with underscores (snake_case)
- Test migrations run automatically on app startup

**Testing:**
```bash
# Create test database locally
psql -c "CREATE DATABASE ankurray_test;"

# Update application.properties for test DB
# Run migrations
mvn spring-boot:run

# Verify tables with psql
psql -d ankurray_test -c "\dt"
# Should show all 4 tables
```

---

### AUTH-1: User Entity, Repository & Password Security
**Type:** Task
**Story Points:** 4
**Sprint:** 2
**Branch:** `feature/user-model` → `Subscription`
**Effort:** 3-4 hours
**Depends on:** SETUP-2

**Description:**
Create User JPA entity, UserRepository, and password hashing with BCryptPasswordEncoder.

**User Story:**
As a developer, I want a secure User model with password hashing, so that user passwords are never stored in plain text.

**Acceptance Criteria:**
- [ ] `User` JPA entity created:
  - All fields: id, email, firstName, lastName, passwordHash, role, isActive, createdAt, updatedAt
  - Properly annotated with `@Entity`, `@Table`, `@Column`, `@Temporal` as appropriate
  - Role enum: USER, ADMIN
  - isActive boolean for soft-delete capability
- [ ] `UserRepository` interface extends `JpaRepository<User, Integer>`
  - Custom methods: `findByEmail(String email)`, `findByEmailAndIsActive(String email, boolean isActive)`
- [ ] `BCryptPasswordEncoder` bean configured in `@Configuration` class
- [ ] Password encoding utility method (not storing raw passwords)
- [ ] Unit tests pass:
  - Test password encoding creates different hash each time (BCrypt adds salt)
  - Test password verification works correctly
  - Test duplicate email detection

**Technical Notes:**
- User passwords are hashed on registration only, never compared raw
- Store `passwordHash` field, not `password`
- `@Transient` annotation for temporary password field in registration DTOs
- Do NOT expose passwordHash in responses

**Testing:**
```java
// Example test structure
@Test
void testPasswordEncoding() {
  String rawPassword = "TestPassword123";
  String hash1 = passwordEncoder.encode(rawPassword);
  String hash2 = passwordEncoder.encode(rawPassword);

  assertNotEquals(hash1, hash2); // Different salts
  assertTrue(passwordEncoder.matches(rawPassword, hash1));
}
```

---

### AUTH-2: JWT Token Generation & Spring Security Configuration
**Type:** Task
**Story Points:** 6
**Sprint:** 2
**Branch:** `feature/jwt-security` → `Subscription`
**Effort:** 5-6 hours
**Depends on:** AUTH-1

**Description:**
Set up JWT token provider, configure Spring Security, and create JWT authentication filter.

**User Story:**
As an API user, I want stateless authentication with JWT tokens, so that I don't need server-side sessions.

**Acceptance Criteria:**
- [ ] `JwtTokenProvider` class created:
  - `generateToken(UserDetails user)` - creates JWT with user claims (id, email, role)
  - `validateToken(String token)` - validates signature and expiration
  - `getClaimsFromToken(String token)` - extracts user info
  - Token expiration: 24 hours access token
- [ ] JWT configuration in `application.properties`:
  - `jwt.secret` - strong secret key (at least 256 bit)
  - `jwt.expiration` - token expiry time
  - Externalized, not hardcoded
- [ ] `JwtAuthenticationFilter` extends `OncePerRequestFilter`:
  - Extracts token from `Authorization: Bearer <token>` header
  - Validates token
  - Sets Spring Security context on valid token
- [ ] `SecurityConfig` class configures Spring Security:
  - Disable CSRF (for stateless REST API)
  - Configure HttpSecurity: permitAll on `/api/auth/**`, require auth on other endpoints
  - Add JwtAuthenticationFilter to filter chain
  - Configure authentication manager
- [ ] `CustomUserDetailsService` implements `UserDetailsService`:
  - Load user by email for authentication
- [ ] Test units pass:
  - Token generated and can be validated
  - Expired token rejected
  - Invalid signature rejected
  - User details loaded correctly

**Technical Notes:**
- Use `io.jsonwebtoken:jjwt` library or Spring Security's built-in JWT support
- Store JWT secret in environment variable, NOT in code
- Filter runs on every request, check for Authorization header
- Handle missing/invalid tokens gracefully (401 response)

**Testing:**
```bash
# Manual test
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Test123!"}'

# Should eventually return token (after endpoint creation)
```

---

### AUTH-3: User Registration & Login Endpoints
**Type:** Feature
**Story Points:** 5
**Sprint:** 2
**Branch:** `feature/auth-registration` → `Login`
**Effort:** 4-5 hours
**Depends on:** AUTH-2

**Description:**
Create REST endpoints for user registration and login with validation.

**User Story:**
As a new user, I want to register with email/password and login, so that I can access the platform.

**Acceptance Criteria:**
- [ ] `POST /api/auth/register` endpoint:
  - Request DTO: `{ email, password, firstName, lastName }`
  - Email validation: valid format, unique in database
  - Password validation: min 8 chars, at least 1 uppercase, 1 lowercase, 1 number
  - Input sanitization (no SQL injection attempts)
  - Response: 201 Created with `{ id, email, firstName, lastName, createdAt }`
  - Error responses:
    - 400 Bad Request for invalid input
    - 409 Conflict if email already exists
  - Password hashed before storage
- [ ] `POST /api/auth/login` endpoint:
  - Request DTO: `{ email, password }`
  - Verify email exists and is active
  - Verify password matches hash
  - Response: 200 OK with `{ accessToken, user: { id, email, role } }`
  - Error responses:
    - 401 Unauthorized for invalid credentials
    - Do not reveal whether email exists or password wrong (generic message)
- [ ] `AuthController` handles both endpoints
- [ ] `UserService.registerUser()` - creates and saves user
- [ ] `UserService.authenticate()` - validates credentials
- [ ] Integration tests pass:
  - Register new user successfully
  - Register with duplicate email rejected
  - Register with weak password rejected
  - Login with correct credentials returns token
  - Login with wrong password returns 401

**Technical Notes:**
- Use `@RequestBody` for JSON input
- Validate on both frontend and backend
- Never log passwords or tokens
- Return consistent error messages for security

**API Examples:**
```bash
# Register
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "SecurePass123",
    "firstName": "John",
    "lastName": "Doe"
  }'

# Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "SecurePass123"
  }'
```

---

### AUTH-4: Protected Endpoints & Role-Based Access Control
**Type:** Task
**Story Points:** 4
**Sprint:** 2
**Branch:** `feature/secured-endpoints` → `Login`
**Effort:** 3-4 hours
**Depends on:** AUTH-3

**Description:**
Add secured endpoints that extract user context from JWT and implement role-based access.

**User Story:**
As the API, I want to protect sensitive endpoints, so that only authenticated users can access them.

**Acceptance Criteria:**
- [ ] `GET /api/auth/me` endpoint (requires valid JWT):
  - Returns current user info: `{ id, email, firstName, lastName, role, createdAt }`
  - Returns 401 if no valid token provided
- [ ] `POST /api/auth/logout` endpoint (requires valid JWT):
  - Invalidates token (optional: could add to blacklist)
  - Returns 200 OK
- [ ] `@PreAuthorize` annotations used for role checks:
  - `@PreAuthorize("hasRole('ADMIN')")` for admin endpoints
  - `@PreAuthorize("isAuthenticated()")` for authenticated endpoints
- [ ] User extraction in controller:
  - Use `@AuthenticationPrincipal` or `SecurityContextHolder` to get current user
  - Available throughout controller methods
- [ ] Test cases pass:
  - Authenticated request to `/api/auth/me` returns user info
  - Unauthenticated request to protected endpoint returns 401
  - Request with invalid token returns 401
  - Request with expired token returns 401
  - Admin-only endpoint returns 403 for non-admin user
  - Admin-only endpoint returns 200 for admin user

**Technical Notes:**
- JWT is extracted from every request by JwtAuthenticationFilter (created in AUTH-2)
- Spring Security automatically sets SecurityContext
- Use `@AuthenticationPrincipal` for clean controller code
- Refresh token logic can be added later (optional for Phase 1)

**Testing:**
```bash
# Get token first (from login)
TOKEN="eyJhbGc..."

# Access protected endpoint
curl -X GET http://localhost:8080/api/auth/me \
  -H "Authorization: Bearer $TOKEN"
# Returns user info

# Without token
curl -X GET http://localhost:8080/api/auth/me
# Returns 401 Unauthorized
```

---

## Epic 2: Free Content Management (Weekend 3)

### CONTENT-1: Article Entity, Repository & Service
**Type:** Task
**Story Points:** 4
**Sprint:** 3
**Branch:** `feature/article-model` → `FreeContent`
**Effort:** 3-4 hours
**Depends on:** SETUP-2

**Description:**
Create Article JPA entity, ArticleRepository, and ArticleService for content management.

**User Story:**
As a content manager, I want articles stored in a database, so that they can be easily queried and filtered.

**Acceptance Criteria:**
- [ ] `Article` JPA entity created:
  - Fields: id, title, slug, content (TEXT), category, authorId (FK to User), isPremium, createdAt, updatedAt
  - Slug auto-generated from title (or validated unique on save)
  - Category: TECH, BOOK_ANALYSIS, PERSONAL (enum or string)
  - isPremium boolean controls visibility
- [ ] `ArticleRepository` interface:
  - `findBySlug(String slug)` - retrieve single article
  - `findByCategoryAndIsPremiumFalse(String category)` - free articles by category
  - `findByIsPremiumFalse(Pageable)` - paginated free articles
  - `findAll(Pageable)` - all articles for admin
- [ ] `ArticleService`:
  - `getArticleBySlug(String slug, User user)` - returns article if accessible by user
  - `getFreeArticles(String category, Page page)` - returns paginated free articles
  - `createArticle(ArticleRequest)` - create new article (admin only)
  - Handle LazyLoadingException for large content fields
- [ ] Unit tests pass:
  - Article can be persisted and retrieved
  - Slug generated correctly
  - Free/premium filter works
  - Pagination works

**Technical Notes:**
- Slug should be URL-friendly (lowercase, no spaces, hyphens instead)
- Use `@Lob` for large TEXT content to avoid truncation
- Consider pagination for large article lists
- Premium articles should not appear in free queries without special handling

**Repository Example:**
```java
public interface ArticleRepository extends JpaRepository<Article, Integer> {
    Optional<Article> findBySlug(String slug);
    Page<Article> findByIsPremiumFalse(Pageable pageable);
    Page<Article> findByCategoryAndIsPremiumFalse(String category, Pageable pageable);
}
```

---

### CONTENT-2: Article API Endpoints
**Type:** Feature
**Story Points:** 5
**Sprint:** 3
**Branch:** `feature/article-api` → `FreeContent`
**Effort:** 4-5 hours
**Depends on:** CONTENT-1

**Description:**
Create REST endpoints for listing and retrieving articles with proper access control.

**User Story:**
As a frontend, I want to fetch articles via API, so that I can display content without reloading pages.

**Acceptance Criteria:**
- [ ] `GET /api/articles` (no auth required):
  - Query params: `page=0, size=10, category=TECH, sort=createdAt,desc`
  - Returns only free articles
  - Paginated response: `{ content: [...], totalElements, currentPage, totalPages }`
  - HTTP 200
- [ ] `GET /api/articles?category={category}` (no auth):
  - Filter by category (defaults to all if not specified)
  - Returns only free articles in that category
- [ ] `GET /api/articles/{slug}` (no auth for free articles):
  - Returns full article if free
  - Returns 403 Forbidden if premium article (not authenticated)
  - Returns 403 Forbidden if premium and user not subscribed
  - Returns full article if premium and user subscribed
  - HTTP 200 for accessible articles, 403 for premium, 404 if not found
- [ ] `GET /api/articles/admin/all` (requires ADMIN role):
  - Returns all articles (free + premium)
  - Protected with `@PreAuthorize("hasRole('ADMIN')")`
  - HTTP 403 if not admin
- [ ] Response DTOs:
  - ArticleListResponse: `{ id, title, slug, excerpt (first 200 chars), category, createdAt, isPremium }`
  - ArticleDetailResponse: `{ id, title, slug, content, category, authorName, createdAt, updatedAt, isPremium }`
- [ ] Error handling:
  - 400 for invalid pagination params
  - 404 for non-existent article
  - 403 for premium article without subscription
- [ ] Integration tests pass:
  - Get free articles returns 200
  - Get premium article without auth returns 403
  - Get premium article as subscribed user returns 200
  - Invalid slug returns 404
  - Pagination works correctly

**Technical Notes:**
- Use Spring Data's `Pageable` for pagination
- Response DTOs separate from entities (never expose sensitive data)
- Excerpt: truncate content to 200 chars + "..."
- Consider caching frequently accessed articles (optional for later)

**API Examples:**
```bash
# Get paginated free articles
curl http://localhost:8080/api/articles?page=0&size=10&sort=createdAt,desc

# Get articles in TECH category
curl http://localhost:8080/api/articles?category=TECH

# Get single article
curl http://localhost:8080/api/articles/aws-exam-prep

# Get all articles (admin only)
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:8080/api/articles/admin/all
```

---

### CONTENT-3: Migrate Existing HTML Articles to Database
**Type:** Task
**Story Points:** 5
**Sprint:** 3
**Branch:** `feature/migrate-articles` → `FreeContent`
**Effort:** 4-6 hours
**Depends on:** CONTENT-2

**Description:**
Parse 13 existing HTML article files and insert into database with proper metadata.

**User Story:**
As a site owner, I want existing articles in the database, so that I can manage them via API.

**Acceptance Criteria:**
- [ ] Create migration script or utility class:
  - Parse HTML from `/posts/*.html` files
  - Extract title, content (HTML preserved), category
  - Generate slug from title
  - Mark as free or premium based on content or manual mapping
  - Insert all 13 articles into articles table
- [ ] Article metadata correct:
  - All articles marked as free for now (isPremium = false)
  - Categories assigned: TECH, BOOK_ANALYSIS, PERSONAL
  - Slugs unique and URL-friendly
  - HTML content preserved (may contain `<p>`, `<h2>`, `<figure>`, etc.)
- [ ] Verify via API:
  - `GET /api/articles` returns all 13 articles
  - Each article accessible via `GET /api/articles/{slug}`
  - Correct categories in responses
  - Pagination works with 13 articles
- [ ] Backup:
  - Original HTML files kept in `/posts/` (don't delete)
  - Commit migration script to git

**Technical Notes:**
- Article list (from exploration): AWS Exam Prep, Book analyses (6), Java Fundamentals, Markdown Guide, MongoDB/AI, Snowflake, Software Engineer Journey
- Create `MigrationScript.java` or SQL seed file
- Test in dev database first before production
- Could use Jsoup library for HTML parsing if needed
- Consider author_id: all articles set to admin user (id=1)

**Checklist:**
- [ ] 13 articles inserted with correct content
- [ ] Slugs are unique
- [ ] Categories assigned correctly
- [ ] All articles accessible via `/api/articles/{slug}`
- [ ] Original HTML files untouched
- [ ] Migration script committed to git

---

## Epic 3: Subscription Model (Weekend 4)

### SUB-1: Subscription Entity & Repository
**Type:** Task
**Story Points:** 4
**Sprint:** 4
**Branch:** `feature/subscription-model` → `Subscription`
**Effort:** 3-4 hours
**Depends on:** SETUP-2

**Description:**
Create Subscription JPA entity and repository for tracking user subscriptions.

**User Story:**
As the system, I want to track user subscription status, so that premium content can be gated properly.

**Acceptance Criteria:**
- [ ] `Subscription` JPA entity created:
  - Fields: id, userId (FK), status (ACTIVE/CANCELLED/EXPIRED), tier (PREMIUM/LIFETIME), startDate, endDate, stripeSubscriptionId, createdAt, updatedAt
  - Status enum or string: ACTIVE, CANCELLED, EXPIRED
  - Tier enum: PREMIUM (monthly/annual), LIFETIME
  - Foreign key relationship to User
- [ ] `SubscriptionRepository`:
  - `findByUserIdAndStatus(Integer userId, String status)` - get active subscription
  - `findActiveByUserId(Integer userId)` - active subscription or empty
  - `findExpiredSubscriptions()` - for cleanup job
- [ ] `SubscriptionService`:
  - `getUserSubscription(Integer userId)` → Optional<Subscription>
  - `createSubscription(Integer userId, String tier)` → Subscription
  - `updateSubscriptionStatus(Integer subscriptionId, String status)` → Subscription
  - `hasActiveSubscription(Integer userId)` → boolean (convenience method)
  - `expireSubscription(Integer subscriptionId)` → void (mark as EXPIRED when end date passes)
- [ ] Cleanup logic:
  - Optional: scheduled task `@Scheduled` to expire subscriptions with end_date < now
- [ ] Unit tests pass:
  - Subscription created and retrieved
  - Status transitions valid
  - `hasActiveSubscription()` returns true for active, false for others
  - Expired subscriptions detected properly

**Technical Notes:**
- Subscription lifecycle: ACTIVE → (CANCELLED or EXPIRED)
- endDate null for LIFETIME tier
- startDate = current date on creation
- stripeSubscriptionId links to Stripe for management

**Service Example:**
```java
public Optional<Subscription> getActiveByUserId(Integer userId) {
    return subscriptionRepository.findByUserIdAndStatus(
        userId,
        SubscriptionStatus.ACTIVE.name()
    );
}
```

---

### SUB-2: Premium Content Access Control
**Type:** Feature
**Story Points:** 5
**Sprint:** 4
**Branch:** `feature/premium-access` → `Subscription`
**Effort:** 4-5 hours
**Depends on:** SUB-1, CONTENT-2

**Description:**
Implement access control for premium articles based on subscription status.

**User Story:**
As the API, I want to block non-subscribers from premium content, so that premium articles generate revenue.

**Acceptance Criteria:**
- [ ] Update `ArticleController.getArticleBySlug()`:
  - Check if article is premium
  - If premium and user not authenticated: return 401 Unauthorized
  - If premium and user authenticated but not subscribed: return 403 Forbidden with message "Subscribe to read full article"
  - If premium and user has active subscription: return 200 with full content
  - If free: always return 200 regardless of auth
- [ ] Add `GET /api/articles/{slug}/preview` (no auth):
  - Return preview of ANY article (free or premium)
  - Preview = first 200 characters + "..."
  - Include `{ id, title, slug, excerpt, isPremium, category }`
  - Full content only if subscribed
- [ ] Create `SubscriptionInterceptor` or `@Aspect` to enrich requests:
  - Extract user from JWT (if present)
  - Load user's subscription status
  - Add to request context for controller access
- [ ] Error messages:
  - 401: "Please log in to read premium content"
  - 403: "You must have an active subscription to read this article. Click here to subscribe."
- [ ] Integration tests pass:
  - Subscriber can read full premium article
  - Non-subscriber gets 403 for premium article
  - Unauthenticated user gets 401 for premium article
  - Free articles accessible to everyone
  - Preview endpoint works for all articles

**Technical Notes:**
- Reuse article fetching logic from CONTENT-2
- May need custom `AuthenticationProvider` or interceptor
- Include subscription status in user context (avoid N+1 queries)
- Error responses should include action items (link to pricing page)

**Testing:**
```bash
# Unauthenticated (no token)
curl http://localhost:8080/api/articles/premium-article
# Returns 401

# With token (subscribed user)
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:8080/api/articles/premium-article
# Returns 200 with full content

# Preview (no auth)
curl http://localhost:8080/api/articles/premium-article/preview
# Returns excerpt only
```

---

## Epic 4: Stripe Integration (Weekends 5-6)

### BILLING-1: Stripe SDK Configuration
**Type:** Task
**Story Points:** 4
**Sprint:** 5
**Branch:** `feature/stripe-integration` → `Subscription`
**Effort:** 3-4 hours
**Depends on:** SETUP-1

**Description:**
Set up Stripe Java SDK and configure payment initialization.

**User Story:**
As the backend, I want Stripe SDK initialized, so that I can process payments securely.

**Acceptance Criteria:**
- [ ] Stripe Java SDK dependency added: `com.stripe:stripe-java`
- [ ] `StripeConfig.java` created:
  - Initialize Stripe with API key from environment variable
  - Never hardcode Stripe key
  - Set API version
- [ ] `application.properties` configuration:
  - `stripe.api.key` - read from environment `STRIPE_API_KEY`
  - `stripe.webhook.secret` - read from environment (for later webhook validation)
  - Different keys for dev/prod profiles
- [ ] `PaymentService` interface created (abstraction):
  - `createPaymentIntent(...)` - method signature
  - `confirmPayment(...)`
  - `createCustomer(...)`
- [ ] `StripePaymentService` implements `PaymentService`:
  - Can initialize and instantiate without errors
  - Error handling for Stripe API exceptions
- [ ] Logging:
  - Log payment operations (without exposing sensitive data)
- [ ] Environment setup:
  - Create `.env.example` file showing required variables
  - Document where to get Stripe keys (test mode first)
- [ ] Test cases:
  - Configuration loads from environment
  - Stripe client initialized successfully
  - Exception handling tested

**Technical Notes:**
- Use Stripe test API key initially (starts with pk_test_)
- Secret key should start with sk_test_
- Store in environment variables, not `.properties` file
- Never commit real API keys
- Stripe Java library throws `com.stripe.exception.StripeException`

**Configuration Example:**
```java
@Configuration
public class StripeConfig {
    @Value("${stripe.api.key}")
    private String stripeApiKey;

    @PostConstruct
    public void init() {
        Stripe.apiKey = stripeApiKey;
    }
}
```

---

### BILLING-2: Payment Intent Creation Endpoint
**Type:** Feature
**Story Points:** 6
**Sprint:** 5-6
**Branch:** `feature/payment-endpoint` → `Subscription`
**Effort:** 5-6 hours
**Depends on:** BILLING-1

**Description:**
Create endpoint to initiate subscription payment and return Stripe clientSecret.

**User Story:**
As a frontend, I want to create a payment intent, so that I can securely collect card information via Stripe Elements.

**Acceptance Criteria:**
- [ ] `POST /api/payments/create-intent` endpoint (requires auth):
  - Request: `{ planId: "monthly" | "annual" | "lifetime", months?: number }`
  - Calculate price based on plan:
    - MONTHLY: $9.99
    - ANNUAL: $99.99 (or $X monthly * 12)
    - LIFETIME: $299.99
  - Create Stripe PaymentIntent with Stripe SDK:
    - Amount in cents
    - Currency: USD
    - Metadata: userId, planId
    - Confirmation method: automatic
  - Response: 200 OK with `{ clientSecret, intentId, amount }`
  - Error: 400 for invalid plan, 500 for Stripe errors
- [ ] `PaymentService.createPaymentIntent()` method:
  - Accepts userId, planId
  - Returns PaymentIntent object
  - Handles StripeException gracefully
- [ ] Database storage (optional for now):
  - May store intent in payments table tracking intent creation
  - Or handled in webhook (next ticket)
- [ ] Security:
  - Only authenticated users can create intents for their own account
  - Cannot create payment for another user
- [ ] Error handling:
  - Stripe API errors logged and returned as 500
  - Invalid plan returns 400
- [ ] Integration tests pass:
  - Authenticated user can create payment intent
  - Unauthenticated user gets 401
  - Invalid plan returns 400
  - Response contains clientSecret and other required fields

**Technical Notes:**
- `PaymentIntent` object holds intent state
- clientSecret shared with frontend to complete payment
- Stripe Java SDK: `com.stripe.model.PaymentIntent`
- Test with Stripe test cards (4242 4242 4242 4242)

**API Example:**
```bash
curl -X POST http://localhost:8080/api/payments/create-intent \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "planId": "monthly" }'

# Response
{
  "clientSecret": "pi_123_secret_456",
  "intentId": "pi_123",
  "amount": 999  // in cents
}
```

---

### BILLING-3: Webhook Handler for Payment Success
**Type:** Feature
**Story Points:** 6
**Sprint:** 6
**Branch:** `feature/payment-webhook` → `Subscription`
**Effort:** 5-6 hours
**Depends on:** BILLING-2

**Description:**
Create webhook endpoint to handle Stripe payment success and activate subscription.

**User Story:**
As Stripe, I want to notify the backend of successful payments, so that subscriptions are automatically activated.

**Acceptance Criteria:**
- [ ] `POST /api/payments/webhook` endpoint (public, no auth):
  - Stripe sends POST request with payment event
  - Endpoint accessible without authentication
  - Handle `payment_intent.succeeded` event
  - Other events ignored
- [ ] Webhook signature verification:
  - Extract signature from `Stripe-Signature` header
  - Verify signature using Stripe secret
  - Return 403 if signature invalid (prevents spoofing)
  - Stripe Java SDK: `Webhook.constructEvent()`
- [ ] On `payment_intent.succeeded`:
  - Extract userId and planId from PaymentIntent metadata
  - Create Subscription record:
    - status = ACTIVE
    - tier = based on planId
    - startDate = today
    - endDate = based on plan duration (or null for LIFETIME)
    - stripeSubscriptionId = payment intent ID
  - Store Payment record:
    - Link user → subscription → payment
    - Amount, status (completed), stripe ID
  - Return 200 OK to Stripe
- [ ] Idempotent:
  - If webhook received twice, don't create duplicate subscription
  - Check if subscription already exists for this payment intent
- [ ] Error handling:
  - Log errors for debugging
  - Still return 200 to Stripe (else Stripe retries forever)
  - Send alert to admin if payment processing fails
- [ ] Testing with Stripe CLI (optional):
  - Test webhook locally with Stripe CLI forwarding
  - Send test event: `stripe trigger payment_intent.succeeded`
- [ ] Integration tests pass:
  - Webhook with valid signature processes payment
  - Webhook with invalid signature returns 403
  - Subscription created and activated
  - Payment logged in database
  - Duplicate payment intent doesn't create duplicate subscription

**Technical Notes:**
- Stripe retries failed webhooks, so make it idempotent
- Use database transaction to ensure atomicity
- Webhook should be fast (use async processing if needed later)
- Log all webhook events for audit trail
- Stripe webhook secret (signing secret, not API key)

**Implementation Example:**
```java
@PostMapping("/webhook")
public ResponseEntity<String> handleWebhook(
    @RequestBody String payload,
    @RequestHeader("Stripe-Signature") String sigHeader
) {
    Event event = Webhook.constructEvent(
        payload, sigHeader, webhookSecret
    );

    if ("payment_intent.succeeded".equals(event.getType())) {
        PaymentIntent intent = (PaymentIntent) event.getDataObjectDeserializer()
            .getObject()
            .orElse(null);

        // Process payment, create subscription
    }

    return ResponseEntity.ok("{}");
}
```

---

### BILLING-4: Subscription Plans Management
**Type:** Task
**Story Points:** 4
**Sprint:** 6
**Branch:** `feature/subscription-plans` → `Subscription`
**Effort:** 3-4 hours
**Depends on:** SETUP-1

**Description:**
Define subscription plans and create endpoint to list available plans.

**User Story:**
As a frontend, I want to display subscription plans, so that users know what they're purchasing.

**Acceptance Criteria:**
- [ ] Subscription plans defined:
  - PREMIUM_MONTHLY: $9.99, 1 month duration
  - PREMIUM_ANNUAL: $99.99, 12 month duration
  - PREMIUM_LIFETIME: $299.99, forever
- [ ] `SubscriptionPlan` class or enum:
  - Fields: id, name, price (in cents), durationMonths, description
- [ ] `SubscriptionPlanService`:
  - `getAvailablePlans()` → List<SubscriptionPlan>
  - `getPlanById(String planId)` → SubscriptionPlan
  - Pricing calculation logic
- [ ] `GET /api/plans` endpoint (public):
  - No authentication required
  - Returns list: `[{ id: "monthly", name: "Premium Monthly", price: 999, duration: 1 }, ...]`
  - HTTP 200
- [ ] Plans stored in:
  - Configuration file (`application.properties` or `plans.yml`)
  - Or database table (optional for later)
- [ ] Integration tests:
  - Fetch plans successfully
  - Plans have correct pricing
  - Plans retrievable by ID

**Technical Notes:**
- Keep plans externalized (config file), not hardcoded
- Prices in cents for Stripe compatibility (9.99 USD = 999 cents)
- Easy to add more plans later
- Consider promoting annual plan (discount vs monthly * 12)

**API Example:**
```bash
curl http://localhost:8080/api/plans

# Response
[
  {
    "id": "monthly",
    "name": "Premium Monthly",
    "price": 999,
    "currency": "USD",
    "durationMonths": 1
  },
  {
    "id": "annual",
    "name": "Premium Annual",
    "price": 99999,
    "currency": "USD",
    "durationMonths": 12
  },
  {
    "id": "lifetime",
    "name": "Lifetime Access",
    "price": 299999,
    "currency": "USD",
    "durationMonths": -1
  }
]
```

---

## Epic 5: Frontend Integration (Weekends 6-7)

### UI-1: Login/Registration Pages
**Type:** Feature
**Story Points:** 5
**Sprint:** 6
**Branch:** `feature/auth-ui` → `Login`
**Effort:** 4-5 hours
**Depends on:** AUTH-3

**Description:**
Create HTML/CSS/JavaScript login and registration forms integrated with backend API.

**User Story:**
As a visitor, I want to login or register, so that I can access member content.

**Acceptance Criteria:**
- [ ] `/auth/login.html` page created:
  - Form fields: email, password
  - Login button
  - "Don't have account?" link to register
  - Form validation (real-time feedback)
  - Error message display on failed login
  - Loading state during request
  - Redirect to dashboard on successful login
- [ ] `/auth/register.html` page created:
  - Form fields: email, first name, last name, password, confirm password
  - Real-time password strength indicator (green/yellow/red)
  - Password match validation
  - Agree to terms checkbox
  - Submit button
  - Error message display
  - Redirect to login on successful registration
- [ ] `assets/js/auth.js` - authentication logic:
  - `login(email, password)` - call `/api/auth/login`, store JWT
  - `register(email, firstName, lastName, password)` - call `/api/auth/register`
  - `logout()` - clear JWT from localStorage
  - `getToken()` - retrieve stored JWT
  - `isAuthenticated()` - check if valid JWT exists
- [ ] JWT token storage:
  - Store in `localStorage` with key `authToken`
  - Include in all API requests: `Authorization: Bearer <token>`
- [ ] Navigation updates:
  - Show "Login/Register" links for anonymous users
  - Show user email + "Logout" link for authenticated users
  - Update navbar dynamically based on auth status
- [ ] Styling:
  - Consistent with existing site design
  - Responsive (mobile-friendly)
  - Clear form styling with labels, placeholders
- [ ] Error handling:
  - Display server error messages
  - Show validation errors inline
  - Prevent double-submit (disable button during request)
- [ ] Testing manual checklist:
  - Register new account successfully
  - Login with registered account successfully
  - JWT token stored in localStorage
  - Failed login shows error message
  - Logout clears token
  - Protected pages redirect to login if not authenticated

**Technical Notes:**
- Use vanilla JavaScript (no framework) to keep simple
- Form validation on frontend AND backend
- Never log passwords or tokens to console in production
- Handle CORS if backend on different domain
- Graceful fallback if JavaScript disabled

**File Structure:**
```
auth/
├── login.html
├── register.html
└── ../assets/js/auth.js
```

---

### UI-2: Article List & Free Content Display
**Type:** Feature
**Story Points:** 5
**Sprint:** 6-7
**Branch:** `feature/articles-ui` → `FreeContent`
**Effort:** 4-5 hours
**Depends on:** CONTENT-2

**Description:**
Update article listing to fetch from API instead of static HTML.

**User Story:**
As a reader, I want to browse all articles dynamically, so that new articles appear without site rebuild.

**Acceptance Criteria:**
- [ ] Update `/posts/index.html`:
  - Fetch articles from `GET /api/articles` instead of static list
  - Display in categories: TECH, BOOK_ANALYSIS, PERSONAL
  - Show article title, excerpt, category, date
  - Link to article detail page
  - Pagination controls (if > 10 articles)
- [ ] Update `/index.html`:
  - Show featured free articles on homepage
  - Link to "View All Articles" page
- [ ] `assets/js/articles.js`:
  - `loadArticles(page, category)` - fetch from API
  - `displayArticles(articles)` - render on page
  - Error handling (show error message if API fails)
- [ ] Article detail page (new):
  - Route: `/articles/{slug}.html` or `/article.html?slug=aws-exam-prep`
  - Fetch article via `GET /api/articles/{slug}`
  - Display full content if free
  - Show paywall if premium (see UI-3)
  - Responsive article styling
- [ ] Styling:
  - Consistent with existing site design
  - Article cards with title, excerpt, category badge
  - Hover effects
- [ ] Error handling:
  - Show "No articles found" if API returns empty
  - Show error banner if API fails
  - Fallback to previous page on navigation error
- [ ] Manual testing:
  - Homepage loads free articles
  - Articles page displays all free articles
  - Click article opens detail page
  - Article content displays correctly
  - Pagination works

**Technical Notes:**
- Use `fetch()` API for HTTP requests
- Include JWT token in Authorization header if user logged in
- Handle 401/403 responses appropriately
- Display loading spinner while fetching
- Consider lazy-loading images (optional)

---

### UI-3: Premium Paywall UI
**Type:** Feature
**Story Points:** 5
**Sprint:** 7
**Branch:** `feature/paywall-ui` → `FreeContent`
**Effort:** 4-5 hours
**Depends on:** SUB-2, UI-2

**Description:**
Display paywall for premium articles and call-to-action to subscribe.

**User Story:**
As a non-subscriber, I want to see premium content preview with subscribe button, so that I'm motivated to purchase.

**Acceptance Criteria:**
- [ ] Premium article page behavior:
  - Show article title and first 200 characters (excerpt)
  - Display "This is a premium article" banner
  - Show "Subscribe to read full article" CTA button
  - Subscription pricing options
  - Link to pricing/checkout page
- [ ] For authenticated subscribers:
  - Show full article content
  - No paywall shown
- [ ] For unauthenticated users on premium article:
  - Show paywall: "Log in to read this article"
  - Link to login page
  - OR show registration CTA
- [ ] Paywall styling:
  - Persistent header with CTA
  - Or inline in article (after excerpt)
  - High contrast, clear call-to-action
  - Button styled attractively
- [ ] JavaScript logic:
  - Check `isPremium` field from article response
  - Check user authentication status
  - Check subscription status (403 error = not subscribed)
  - Show/hide article content accordingly
  - Handle 401/403 errors from API
- [ ] Manual testing:
  - Non-subscriber sees paywall on premium article
  - Subscriber sees full premium article
  - Anonymous user sees login CTA
  - CTA buttons click to correct pages

**Technical Notes:**
- Paywall can be shown client-side or server-side
- Client-side more efficient (show/hide based on response)
- Error messages: 401 (login), 403 (subscribe)
- Consider upsell: show pricing page when clicking CTA

---

### UI-4: Checkout Flow with Stripe Elements
**Type:** Feature
**Story Points:** 6
**Sprint:** 7
**Branch:** `feature/checkout-ui` → `Subscription`
**Effort:** 5-6 hours
**Depends on:** BILLING-2, BILLING-4

**Description:**
Create checkout page with Stripe Elements card form and payment processing.

**User Story:**
As a customer, I want to securely enter my card and complete subscription purchase, so that I gain access to premium content.

**Acceptance Criteria:**
- [ ] `/pricing.html` or `/checkout.html` page created:
  - Display subscription plans with pricing
  - Plan comparison (monthly, annual, lifetime)
  - Select button for each plan
  - Annual plan highlighted as popular/best value
- [ ] Checkout form:
  - Stripe Elements card input (secure)
  - Card holder name field
  - Email field (pre-filled if logged in)
  - Agree to terms checkbox
  - "Subscribe" button (disabled during processing)
- [ ] JavaScript checkout logic (`assets/js/checkout.js`):
  - Initialize Stripe: `new Stripe(publishableKey)`
  - On plan select, create payment intent: `POST /api/payments/create-intent`
  - Initialize Stripe Elements with clientSecret
  - Handle card input
  - On subscribe click, confirm payment: `stripe.confirmCardPayment(clientSecret, card)`
  - Handle success/error responses
- [ ] Success flow:
  - After payment confirmed, show "Success!" message
  - Auto-redirect to dashboard after 2 seconds
  - Newsletter signup prompt (optional)
- [ ] Error handling:
  - Display error message from Stripe
  - Retry button for failed payments
  - Show helpful error messages (card declined, etc.)
- [ ] Security:
  - Use Stripe test key in dev
  - Never send card details to backend (Stripe handles it)
  - Validate form on client and server
- [ ] Manual testing with Stripe test cards:
  - Card success: 4242 4242 4242 4242
  - Card decline: 4000 0000 0000 0002
  - 3D Secure: 4000 0025 0000 3155
- [ ] HTML structure:
  - Plan selection
  - Stripe Elements container
  - Error message container
  - Success message container

**Technical Notes:**
- Include Stripe JS library: `<script src="https://js.stripe.com/v3/"></script>`
- Use publishable key (public, safe to expose)
- Test mode keys for development
- Handle Stripe response states: requires_action, succeeded, etc.
- Store subscription status after success (might need page refresh or token update)

---

### UI-5: Subscriber Dashboard
**Type:** Feature
**Story Points:** 5
**Sprint:** 7
**Branch:** `feature/dashboard-ui` → `Subscription`
**Effort:** 4-5 hours
**Depends on:** AUTH-4, SUB-2

**Description:**
Create member dashboard showing subscription status and account information.

**User Story:**
As a subscriber, I want a dashboard showing my subscription details, so that I can manage my account.

**Acceptance Criteria:**
- [ ] `/dashboard.html` or `/account.html` page:
  - Accessible only to authenticated users (redirect to login if not)
  - Display user info: name, email, join date
  - Subscription status section:
    - Plan name (Premium Monthly, etc.)
    - Current period dates
    - Renews on (date)
    - Status badge (ACTIVE, EXPIRED, CANCELLED)
  - Action buttons:
    - "Upgrade Plan" (if on monthly, offer annual)
    - "Manage Subscription" (link to Stripe portal or custom)
    - "Cancel Subscription" (with confirmation)
  - Recent activity: (optional) login history, payment history
- [ ] JavaScript logic (`assets/js/dashboard.js`):
  - Load user profile: `GET /api/auth/me`
  - Load subscription: `GET /api/subscriptions/me` (new endpoint needed in backend)
  - Display subscription details
  - Handle errors (401 redirect to login)
- [ ] Styling:
  - Clear information hierarchy
  - Status badges with colors (green=active, red=expired)
  - Action buttons prominently displayed
- [ ] Navigation:
  - Add "Dashboard" link to navbar for authenticated users
  - Link to dashboard from success page after payment
- [ ] Manual testing:
  - Authenticated user sees their subscription
  - Subscription dates display correctly
  - Buttons are functional (links work)
  - Unauthenticated user redirected to login

**Technical Notes:**
- Subscription endpoint (`GET /api/subscriptions/me`): needed in backend (future ticket)
- Dashboard refreshes automatically on page load
- Could add future features: download invoices, payment history, referalls

**Endpoint Needed (backend):**
```
GET /api/subscriptions/me (requires auth)
Response: {
  id, userId, status, tier, startDate, endDate,
  stripeId, createdAt, updatedAt
}
```

---

## Epic 6: Testing & Deployment (Weekend 8)

### TEST-1: Backend Unit & Integration Tests
**Type:** Task
**Story Points:** 5
**Sprint:** 8
**Branch:** `feature/backend-tests` → `Subscription`
**Effort:** 4-5 hours
**Depends on:** All backend tickets

**Description:**
Write comprehensive unit and integration tests for backend services and API.

**User Story:**
As a developer, I want automated tests, so that regressions are caught before production.

**Acceptance Criteria:**
- [ ] Unit tests (services):
  - `UserServiceTest`: registration, authentication, not found errors
  - `ArticleServiceTest`: free/premium filtering, slug generation
  - `SubscriptionServiceTest`: status checking, active subscription queries
  - `StripePaymentServiceTest`: payment intent creation (mocked)
  - Coverage: 80%+ of service methods
- [ ] Integration tests (API endpoints):
  - `AuthenticationIntegrationTest`: register, login, retrieve profile
  - `ArticleApiIntegrationTest`: list free articles, access premium (403), subscribe then access
  - `PaymentIntegrationTest`: create payment intent, webhook handling
  - Create test database (H2 or TestContainers with PostgreSQL)
  - Use `@SpringBootTest` for full context
- [ ] Test structure:
  - Arrange-Act-Assert pattern
  - Clear test names describing scenario
  - Isolated tests (no shared state)
  - Mock external calls (Stripe SDK)
  - Use `@MockBean` for Stripe service
- [ ] Test data:
  - Fixtures/builders for User, Article, Subscription models
  - Test article factory with free/premium variants
- [ ] Coverage report:
  - Generate coverage report
  - Verify 80%+ line coverage
  - Identify low-coverage areas and test them
- [ ] CI integration (optional):
  - Configure to run on git push
  - Fail build if coverage < 80%
- [ ] Test execution:
  - All tests pass locally
  - No flaky tests
  - Fast execution (<10 seconds for unit, <30 for integration)

**Testing Tools:**
- JUnit 5
- Mockito for mocking
- assertJ for fluent assertions
- TestContainers for database testing (optional but recommended)

**Example Test:**
```java
@SpringBootTest
class UserServiceIntegrationTest {
    @Autowired
    private UserService userService;

    @Test
    void testRegisterNewUser() {
        UserRequest request = new UserRequest("test@mail.com", "Pass123", "John", "Doe");
        User user = userService.registerUser(request);

        assertThat(user).isNotNull();
        assertThat(user.getEmail()).isEqualTo("test@mail.com");
        assertThat(user.getId()).isNotNull();
    }
}
```

---

### TEST-2: E2E Testing & Security Review
**Type:** Task
**Story Points:** 5
**Sprint:** 8
**Branch:** `feature/e2e-security` → `Subscription`
**Effort:** 4-5 hours
**Depends on:** All tickets

**Description:**
End-to-end testing of full user journey and security checklist.

**User Story:**
As the team, I want to ensure the feature works end-to-end and is secure, so that we can deploy confidently.

**Acceptance Criteria:**
- [ ] E2E test scenarios:
  1. Anonymous user sees free articles
  2. Anonymous user tries to access premium article → 401 redirect to login
  3. New user registers → receives confirmation
  4. Registered user logs in → redirected to dashboard
  5. Logged-in user clicks premium article → sees paywall with pricing
  6. User subscribes via payment flow → payment succeeds
  7. Subscription activated → user can now access premium content
  8. User logs out → session cleared
  9. Unsubscribed user cannot access premium content
  10. Admin can see all articles (free + premium)
- [ ] E2E execution method:
  - Manual testing checklist (browser-based)
  - OR automated with Selenium/Playwright (optional)
  - Test on dev and staging environments
- [ ] Security checklist (MUST PASS):
  - [ ] SQL Injection prevention: all queries use parameterized queries (Spring Data JPA)
  - [ ] XSS prevention: HTML content escaped in responses, Thymeleaf auto-escapes
  - [ ] CSRF protection: disabled for stateless API (ok for REST)
  - [ ] Password hashing: BCrypt with salt, never plain text
  - [ ] JWT validation: tokens signed and verified
  - [ ] Authentication: all protected endpoints require valid JWT
  - [ ] Authorization: role-based access enforced (@PreAuthorize)
  - [ ] Webhook signature verification: Stripe signatures validated
  - [ ] Rate limiting: (optional) added to auth endpoints to prevent brute force
  - [ ] Error messages: don't leak sensitive data
  - [ ] HTTPS: enforced in production (config)
  - [ ] Environment variables: secrets not in code
  - [ ] Dependency scan: no known vulnerabilities in dependencies
- [ ] Manual security testing:
  - Try to access endpoints without token → 401
  - Try to access admin endpoints as user → 403
  - Try invalid JSON in requests → 400
  - Try SQL injection in search (if added) → safely handled
  - Inspect API responses → no sensitive data exposed
  - Check localStorage → only token stored, no passwords
- [ ] Test with Stripe test cards:
  - Successful charge
  - Declined card
  - 3D Secure flow
  - Refund scenario (future)
- [ ] Documentation:
  - Record test results
  - Screenshot/video of payment flow
  - List of security verifications passed

**Security Scan Tools (Optional):**
- `dependency-check` Maven plugin for CVE scanning
- OWASP ZAP for automated security testing

**Manual Checklist Format:**
```
[ ] E2E Test 1: Anonymous user sees free articles
  Status: PASS/FAIL
  Notes: ...

[ ] E2E Test 2: User registration
  Status: PASS/FAIL
  Notes: ...

[ ] Security Test: JWT validation
  Status: PASS/FAIL
  Notes: ...
```

---

### DEP-1: Deployment Configuration & Documentation
**Type:** Task
**Story Points:** 4
**Sprint:** 8
**Branch:** `feature/deployment` → `Subscription`
**Effort:** 3-4 hours
**Depends on:** All tickets

**Description:**
Configure production deployment and create deployment documentation.

**User Story:**
As the team, I want to deploy to production safely, so that users can access premium features.

**Acceptance Criteria:**
- [ ] Production configuration:
  - `application-prod.properties` with production settings
  - Database connection pool tuned for production
  - Stripe production keys (not test keys)
  - JWT secret strong and different from dev
  - HTTPS forced (redirect HTTP → HTTPS)
  - CORS configured for your domain
  - Logging level appropriate for production
- [ ] Deployment target (choose one):
  - Heroku: Create Procfile, add-ons for PostgreSQL
  - Railway: Create railway.json, connect PostgreSQL
  - AWS: EC2 with security groups, RDS for database
  - Do tell me your choice for specific instructions
- [ ] Database migration:
  - Flyway automatically runs migrations on startup
  - Verify on production database
  - Backup strategy documented
- [ ] Environment variables:
  - `.env.example` file checked in (no secrets)
  - Document all required variables:
    - `STRIPE_API_KEY` - Stripe secret key
    - `STRIPE_WEBHOOK_SECRET` - Webhook signing secret
    - `JWT_SECRET` - JWT signing key
    - `DATABASE_URL` - PostgreSQL connection
    - `SPRING_PROFILES_ACTIVE=prod`
  - Document how to set on production platform
- [ ] Deployment checklist:
  - [ ] All tests pass
  - [ ] Security checklist complete
  - [ ] Dependencies updated and no vulnerabilities
  - [ ] Database backed up
  - [ ] Deployment tested in staging first
  - [ ] Rollback plan documented
  - [ ] Monitoring/logging configured
- [ ] Documentation:
  - `DEPLOYMENT.md` with step-by-step instructions
  - How to set environment variables on platform
  - How to run database migrations
  - How to check health after deployment
  - Rollback instructions
  - Monitoring/alerting setup
  - Contact info for on-call support
- [ ] Post-deployment:
  - Verify health check endpoint `GET /api/health` returns 200
  - Test login flow in production
  - Test article access (free + premium)
  - Test payment webhook (Stripe can test via dashboard)
  - Check logs for errors
  - Monitor error rates

**Deployment Script Example (Heroku):**
```bash
# First time
heroku create ankurray-subscription
heroku addons:create heroku-postgresql:standard-0 -a ankurray-subscription
heroku config:set STRIPE_API_KEY=sk_live_... -a ankurray-subscription
git push heroku main

# Verify
heroku logs -t -a ankurray-subscription
curl https://ankurray-subscription.herokuapp.com/api/health
```

**Documentation Template:** See `DEPLOYMENT.md` template elsewhere

---

## Summary Table

| Epic | Tickets | Weekends | Total SP | Branch |
|------|---------|----------|----------|--------|
| 1: Infrastructure | SETUP-1, SETUP-2, AUTH-1, AUTH-2, AUTH-3, AUTH-4 | 1-2 | 25 | Subscription, Login |
| 2: Free Content | CONTENT-1, CONTENT-2, CONTENT-3 | 3 | 14 | FreeContent |
| 3: Subscription | SUB-1, SUB-2 | 4 | 9 | Subscription |
| 4: Stripe | BILLING-1, BILLING-2, BILLING-3, BILLING-4 | 5-6 | 20 | Subscription |
| 5: Frontend | UI-1, UI-2, UI-3, UI-4, UI-5 | 6-7 | 26 | Login, FreeContent, Subscription |
| 6: Testing | TEST-1, TEST-2, DEP-1 | 8 | 14 | All branches |
| **TOTAL** | **18 tickets** | **8 weekends** | **108 SP** | |

---

## How to Use These Tickets

1. **Import to GitHub Issues:**
   - Create issue for each ticket
   - Use labels: Epic, Sprint, Type (Task/Feature)
   - Assign to yourself
   - Add to project board

2. **Or Use as Kanban Board:**
   - Create columns: Backlog, In Progress, In Review, Done
   - Move tickets as you progress
   - Reference ticket number in commits: "SETUP-1: Spring Boot scaffold"

3. **Track Progress:**
   - Check off acceptance criteria as you complete
   - Update branch names consistently
   - PR title should reference ticket: "SETUP-1: Spring Boot Project Setup"

4. **Customize as Needed:**
   - Adjust story points based on your estimation
   - Reorder if you prefer different sequence
   - Combine/split tickets based on work style

---

## Ready to Start?

Begin with **SETUP-1: Spring Boot Project Scaffold** on your next weekend!

Have questions about a specific ticket? Let me know and I'll clarify the requirements.

Good luck! 🚀
