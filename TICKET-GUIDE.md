# Subscription Feature - Agile Ticket Guide

## Quick Start

**Tickets Location:** `jira-tickets.json` in project root

**How to Import Tickets:**

### Option 1: GitHub Issues
```bash
# Install gh CLI if needed
gh issue create --title "SETUP-1: Spring Boot..." --body "..." --label epic-1

# Or copy-paste each ticket manually from jira-tickets.json
```

### Option 2: Jira
1. Create Jira project
2. Use Jira's JSON import feature
3. Map fields accordingly

### Option 3: Manual Tracking
Copy ticket details to a spreadsheet or project management tool

---

## Execution Plan (8 Weekends Total)

### Weekend 1: Foundation
- **SETUP-1** - Spring Boot project scaffold (2h)
- **SETUP-2** - Database schema & Flyway (3h)

### Weekend 2: Authentication
- **AUTH-1** - User entity & repository (2h)
- **AUTH-2** - JWT configuration (3h)
- **AUTH-3** - Registration & login endpoints (3h)
- **AUTH-4** - Protected endpoints & roles (2h)

### Weekend 3: Content Management
- **CONTENT-1** - Article entity & service (2h)
- **CONTENT-2** - Article API endpoints (2h)
- **CONTENT-3** - Migrate 13 HTML articles to database (4h)

### Weekend 4: Subscriptions
- **SUB-1** - Subscription entity & service (2h)
- **SUB-2** - Premium access control (2h)

### Weekend 5: Billing Setup
- **BILLING-1** - Stripe SDK configuration (1h)
- **BILLING-2** - Payment intent creation (2h)
- **BILLING-3** - Webhook handler for payments (3h)

### Weekend 6: Billing + UI Start
- **BILLING-4** - Subscription plans (2h)
- **UI-1** - Login/registration pages (3h)
- **UI-2** - Article list fetching from API (2h)
- **UI-3** - Premium paywall UI (2h)

### Weekend 7: UI Completion
- **UI-4** - Stripe Elements checkout (4h)
- **UI-5** - Subscriber dashboard (3h)

### Weekend 8: Testing & Deployment
- **TEST-1** - Unit tests 80%+ coverage (4h)
- **TEST-2** - Integration tests (5h)
- **DEP-1** - Deployment & docs (3h)

---

## Git Branching Strategy

```
main (production)
  ↑
  ├─ Subscription (integration branch)
  │  ├─ feature/setup-spring-boot
  │  ├─ feature/database-schema
  │  ├─ feature/user-model
  │  ├─ feature/jwt-security
  │  ├─ feature/subscription-model
  │  ├─ feature/premium-access
  │  ├─ feature/stripe-config
  │  ├─ feature/stripe-payments
  │  ├─ feature/stripe-webhooks
  │  └─ feature/deployment
  │
  ├─ Login (integration branch)
  │  ├─ feature/auth-registration
  │  ├─ feature/auth-endpoints
  │  └─ feature/auth-ui
  │
  └─ FreeContent (integration branch)
     ├─ feature/article-model
     ├─ feature/article-api
     ├─ feature/migrate-articles
     ├─ feature/articles-ui
     └─ feature/paywall-ui
```

**Workflow per Ticket:**
```bash
# 1. Create feature branch from parent
git checkout Subscription  # (or Login/FreeContent)
git pull origin Subscription
git checkout -b feature/setup-spring-boot

# 2. Work on the ticket
# ... implement the ticket ...
# ... write tests ...
# ... commit with descriptive message ...

# 3. Push to origin
git push -u origin feature/setup-spring-boot

# 4. Create PR
gh pr create --base Subscription --title "SETUP-1: Spring Boot project scaffold" --body "..."

# 5. Review, approve, merge
gh pr merge --merge

# 6. Repeat for next ticket
```

---

## Definition of Done (ALL Tickets)

Every ticket must meet these criteria before marking complete:

- [ ] **Code Implemented** - All acceptance criteria met
- [ ] **Tests Written** - Unit/integration tests covering the feature
- [ ] **Tests Passing** - `./gradlew test` passes with no failures
- [ ] **Code Reviewed** - Another review if possible (or self-review against checklist)
- [ ] **Committed & Pushed** - Feature branch has clean commit history
- [ ] **PR Merged** - Feature merged to parent branch (Subscription/Login/FreeContent)
- [ ] **Documentation** - README updated if needed, setup steps clear
- [ ] **Local Verification** - Feature works end-to-end locally

---

## Key File Locations

### Backend Structure
```
ankurray-backend/
├── build.gradle                    (dependencies)
├── src/main/java/com/ankurray/
│   ├── Application.java            (main entry point)
│   ├── config/                     (configuration classes)
│   ├── controller/                 (REST endpoints)
│   ├── service/                    (business logic)
│   ├── repository/                 (database queries)
│   ├── model/                      (JPA entities)
│   ├── dto/                        (request/response objects)
│   ├── security/                   (JWT, auth filters)
│   └── exception/                  (custom exceptions)
├── src/main/resources/
│   ├── application.properties      (config)
│   └── db/migration/               (Flyway SQL migrations)
└── src/test/java/com/ankurray/    (tests - mirror src/main structure)
```

### Frontend Additions
```
index.html                          (update navbar with login link)
posts/index.html                   (update to fetch from API)
posts/article-detail.html          (new - display full article)
auth/
├── login.html                     (new)
├── register.html                  (new)
└── login.js                       (JWT handling)
checkout/
├── checkout.html                  (new)
└── checkout.js                    (Stripe Elements)
dashboard/
├── dashboard.html                 (new)
└── dashboard.js                   (user profile + subscription)
assets/
└── paywall.css                    (new - premium content modal)
```

---

## Database Schema Overview

```sql
-- Users: Store user accounts
users (id, email, password_hash, role, created_at)

-- Articles: Store blog posts
articles (id, title, slug, content, is_premium, created_at)

-- Subscriptions: Track active subscriptions per user
subscriptions (id, user_id, status, tier, end_date, stripe_id, created_at)

-- Payments: Track payment transactions
payments (id, user_id, amount, status, stripe_id, created_at)
```

---

## Environment Variables Needed

Create `.env` file in backend root:

```bash
# Database
DATABASE_URL=jdbc:postgresql://localhost:5432/ankurray
DB_USER=postgres
DB_PASSWORD=your_password

# Stripe (get from https://dashboard.stripe.com/apikeys)
STRIPE_API_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...

# JWT
JWT_SECRET=your_super_secret_key_min_32_chars
JWT_EXPIRATION=86400000  # 24 hours in milliseconds

# Server
SERVER_PORT=8080
SPRING_PROFILES_ACTIVE=dev
```

---

## Testing Commands

```bash
# Run all tests
./gradlew test

# Run with coverage report
./gradlew test jacocoTestReport

# Run specific test class
./gradlew test --tests UserServiceTest

# Run integration tests
./gradlew integrationTest
```

---

## Deployment Checklist (Before DEP-1)

- [ ] All tests passing (80%+ coverage)
- [ ] No hardcoded secrets (use env vars)
- [ ] Error logging configured
- [ ] CORS properly configured (frontend domain)
- [ ] Stripe webhook secret configured
- [ ] Database migrations tested
- [ ] Health check endpoint working
- [ ] Dockerfile builds successfully
- [ ] Documentation complete
- [ ] Staging environment ready

---

## Success Metrics

After completing all 18 tickets:

✅ Users can register and login to ankurray.com
✅ Authentication via JWT tokens works
✅ 13 articles migrated from HTML to database
✅ Free articles visible to all users
✅ Premium articles require active subscription
✅ Stripe payment processing complete
✅ Subscriptions activate after successful payment
✅ Subscriber dashboard shows subscription status
✅ 80%+ test coverage on backend
✅ Deployment-ready Spring Boot application

---

## Learning Outcomes

By completing all tickets, you'll have learned:

1. **Spring Boot** - Project setup, dependencies, application configuration
2. **Spring Data JPA** - Entity relationships, repositories, queries
3. **Spring Security** - JWT authentication, role-based access control
4. **PostgreSQL** - Schema design, migrations, relationships
5. **REST API Design** - Endpoint standards, error handling, pagination
6. **Payment Processing** - Stripe integration, webhooks, subscription management
7. **Testing** - Unit tests, integration tests, mocking
8. **Frontend Integration** - API calls from JavaScript, localStorage, fetch/AJAX

Each ticket is a learning opportunity. After each ticket, document what you learned and consider writing it as premium content for your blog!

---

## Questions?

Each ticket has:
- **User Story**: Why this feature matters
- **Acceptance Criteria**: How to know it's done
- **Dependencies**: What must complete first
- **Estimated Hours**: Rough time estimate

Follow acceptance criteria strictly. If something seems unclear, ask before starting.

Good luck! 🚀
