## Epic
Epic 1: Backend Infrastructure & Authentication

## Type
Setup/Task

## Story Points
5

## Sprint
1

## Branch
feature/setup-spring-boot → Subscription

## User Story
As a developer, I want a properly structured Spring Boot project, so that I have a solid foundation for building the API.

## Acceptance Criteria
- [ ] Spring Boot 3.x project created with Maven build system
- [ ] Dependencies added: Spring Web, Spring Data JPA, PostgreSQL/MySQL driver, Spring Security, Stripe SDK
- [ ] Package structure created: `com.ankurray.{config, controller, service, repository, model, dto, security, exception}`
- [ ] H2 database configured for development/testing
- [ ] Main `Application.java` entry point runs without errors
- [ ] Basic health check endpoint `GET /api/health` returns 200 OK
- [ ] `application.properties` configured for dev environment
- [ ] `.gitignore` includes typical Java/Maven ignores (target/, .classpath, .project, etc.)

## Technical Notes
- Use Spring Boot Starter dependencies
- Configure profiles for dev/prod
- Set up logging with SLF4J
- Add Swagger/OpenAPI dependency for API documentation (optional for later)

## Testing
```bash
mvn spring-boot:run
# Should start on port 8080 without errors
curl http://localhost:8080/api/health
# Should return HTTP 200