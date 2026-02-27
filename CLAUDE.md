# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
./mvnw clean install              # Full build (compiles, generates code, runs tests)
./mvnw spring-boot:run            # Run app locally (http://localhost:9966/petclinic/)
./mvnw test                       # Run all tests
./mvnw test -Dtest=OwnerRestControllerTests          # Run a single test class
./mvnw test -Dtest=OwnerRestControllerTests#testGetOwnerSuccess  # Run a single test method
./mvnw compile                    # Compile + generate OpenAPI DTOs and MapStruct mappers
```

The app runs on port 9966 with context path `/petclinic/`. Swagger UI: `/petclinic/swagger-ui.html`.

## Architecture

This is a Spring Boot 4.0 REST API for a veterinary clinic, using an **API-first** approach with code generation.

### Code Generation Pipeline (must run `mvn compile` before IDE use)
- **OpenAPI Generator** reads `src/main/resources/openapi.yml` → generates DTO classes (`rest.dto` package) and API interfaces (`rest.api` package) into `target/generated-sources/openapi/`
- **MapStruct** generates mapper implementations from interfaces in `mapper/` package
- Controllers implement the generated API interfaces; DTOs and API interfaces should NOT be hand-edited

### Layered Architecture
- **Controllers** (`rest.controller`) — implement generated API interfaces, use `@PreAuthorize` for role-based security, return `ResponseEntity<T>`
- **Service** (`service`) — `ClinicService` facade interface with `ClinicServiceImpl` aggregating all repository operations
- **Repositories** (`repository`) — three interchangeable implementations: `jpa/`, `jdbc/`, `springdatajpa/`, selected via Spring profiles
- **Model** (`model`) — JPA entities with inheritance: `BaseEntity` → `NamedEntity` → `Person` → `Owner`/`Vet`
- **Mappers** (`mapper`) — MapStruct interfaces for bidirectional entity↔DTO conversion (Spring component model)
- **Security** (`security`) — conditional via `petclinic.security.enable` property; disabled by default. Uses HTTP Basic + JDBC auth with roles: `OWNER_ADMIN`, `VET_ADMIN`, `ADMIN`

### Profiles
Active profiles are set as `{database},{repository-impl}`:
- Database: `h2` (default), `mysql`, `postgres`, `hsqldb`
- Repository: `spring-data-jpa` (default), `jpa`, `jdbc`

## Testing Patterns

- **Controller tests** use `MockMvc` with `@MockitoBean` for `ClinicService`; built via `MockMvcBuilders.standaloneSetup()` with `ExceptionControllerAdvice`
- **Service tests** use abstract base classes (`AbstractClinicServiceTests`, `AbstractUserServiceTests`) with concrete subclasses per repository implementation
- **JaCoCo** enforces minimum 85% line coverage and 66% branch coverage (excludes generated `rest.dto.*` and `rest.api.*`)

## Key Conventions

- Global exception handling via `ExceptionControllerAdvice` returning RFC 7807 `ProblemDetail`
- All controllers annotated with `@CrossOrigin(exposedHeaders = "errors, content-type")`
- Java 17, indent 4 spaces, UTF-8, LF line endings (see `.editorconfig`)
- To modify API contracts, edit `src/main/resources/openapi.yml` then rebuild
