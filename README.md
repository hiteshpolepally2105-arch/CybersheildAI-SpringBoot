# CyberShield AI — Spring Boot application

This is the additive Java implementation of CyberShield AI. The existing Flask
prototype in the project root remains unchanged.

## Implemented phases

- **Phase 1:** Java 17, Spring Boot 3.5.7, Maven Wrapper, Thymeleaf, JPA, MySQL
  configuration, and the initial package structure.
- **Phase 2:** session-based login/logout, BCrypt password hashes, four server-
  enforced roles, Super Admin account management, self-service password change,
  Flyway-backed user/audit tables, and login/account audit events.
- **Phase 3:** Flyway-backed security-event and incident records, database-derived
  operational metrics, locally served Chart.js charts, and role-protected,
  filterable event and incident listings.
- **Not yet implemented:** event ingestion, threat detection, and incident
  investigation workflows.

Password reset is deliberately not simulated. Add a verified reset channel in
a later phase before exposing a public forgot-password workflow. An optional
administrator-set password reset is available only under the explicitly
activated `dev` profile; it is not available in the default or `prod` profile.

## Prerequisites

- JDK 17.
- The included Maven Wrapper downloads Maven 3.9.9 on first use.
- MySQL 8.x for the default application profile.

Confirm the selected Java runtime with `java -version`. Set `JAVA_HOME` to a
JDK 17 installation before building.

## Build and tests

From this directory, run:

```bash
./mvnw test
```

Tests use isolated in-memory H2 and the `test` plus `dev` profiles to cover
migrations, the Super Admin bootstrap, login/logout, CSRF, password
hashing/change, account administration, disabled accounts, role boundaries,
auditor read-only access, database-backed dashboard metrics, event/incident
filters, and local chart assets. H2 is test-only.

## Configure MySQL locally

Create a dedicated development database and a least-privilege application
account in MySQL. Do not use a production account or commit credentials.

```sql
CREATE DATABASE cybershield CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'cybershield_app'@'localhost' IDENTIFIED BY 'choose-a-local-secret';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX
    ON cybershield.* TO 'cybershield_app'@'localhost';
```

Set the connection values in the shell that will run the app:

```bash
export DB_URL='jdbc:mysql://localhost:3306/cybershield?serverTimezone=UTC'
export DB_USERNAME='cybershield_app'
export DB_PASSWORD='your-local-secret'
export BOOTSTRAP_SUPER_ADMIN_USERNAME='your-admin-name'
export BOOTSTRAP_SUPER_ADMIN_PASSWORD='choose-a-unique-password-of-at-least-12-characters'
./mvnw spring-boot:run
```

The bootstrap variables must either both be set or both be omitted. On first
startup, the configured account is created as an enabled `SUPER_ADMIN` with a
BCrypt hash. The same account is not overwritten on later starts. There is no
default username or password. Keep the bootstrap password private and unset the
bootstrap variables after creating the account. Flyway applies versioned schema
migrations; Hibernate never creates or deletes tables. If port 8080 is already
in use, also set `SERVER_PORT` to an available port.

To enable the development-only, Super-Admin-restricted password reset control:

```bash
./mvnw -Dspring-boot.run.profiles=dev spring-boot:run
```

This still requires database credentials and bootstrap configuration as above.
Do not activate the `dev` profile in a public deployment. For production
session cookies, activate the `prod` profile and terminate HTTPS at the
application or a trusted reverse proxy.

## Configuration

- `DB_URL`: JDBC URL; defaults to `jdbc:mysql://localhost:3306/cybershield?serverTimezone=UTC`.
- `DB_USERNAME`: required database username for the default profile.
- `DB_PASSWORD`: required database password for the default profile.
- `BOOTSTRAP_SUPER_ADMIN_USERNAME`: optional initial Super Admin name.
- `BOOTSTRAP_SUPER_ADMIN_PASSWORD`: optional initial Super Admin password
  (minimum 12 characters; never set this in source control).
- `SERVER_PORT`: HTTP port; defaults to `8080`.

All browser form changes, including login and logout, use Spring Security CSRF
protection. Sessions are HTTP-only, SameSite=Lax, and expire after 30 minutes.
Role boundaries are enforced server-side: Super Admin manages accounts;
Security Admin has analyst access; Regular User has only their account/dashboard
access; Auditor has read-only audit, event, and incident access.
