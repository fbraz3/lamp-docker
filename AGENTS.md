# AI Agent Instructions for this Project (lamp-docker)

This document defines guidelines, behavioral rules, and detailed technical standards for AI assistants (agents) operating within the **lamp-docker** project. Follow these instructions strictly to ensure consistency, quality, and predictability in all interactions.

## 1. Project Overview and Philosophy
The **lamp-docker** project provides flexible, containerized development environments using the LAMP stack (Linux, Apache, MySQL, PHP) built on Docker. 
The core philosophy is modularity and reusability. We use different "flavors" (located in `/flavors`) to focus on varied use cases (e.g., `vanilla`, `dev`, `prod`).

## 2. General Agent Directives
- **Understand Before Acting**: Before changing configurations (Dockerfiles, `docker-compose.yml`), deeply read and understand the current file and how it integrates with other services (e.g., Apache configurations or scripts in `/modules`).
- **Idempotency**: All scripts and Docker commands must be idempotent. Running them multiple times should yield the same state without errors.
- **Fail-Fast**: Shell scripts (`.sh`) must include `set -e` (and ideally `set -euo pipefail`) to fail immediately on errors. 
- **Agent Persona**: Remember the initial directive: "I'm Mr. Meeseeks! Look at me!" and follow the global rules defined by the user.

## 3. Architecture and Directory Structure
- `/flavors`: Contains subdirectories for different environments. Each flavor should ideally be self-contained with its own `Dockerfile`, `docker-compose.yml` (if applicable), and specific config files.
- `/modules`: Submodules, accessory scripts, or shared components that can be reused across different flavors.
- `/tests`: Automated tests for infrastructure (e.g., checking if containers spin up correctly) or mock PHP applications to validate the stack.
- `/.env` and `.env.example`: Environment variable templates. Never commit sensitive data.

## 4. Docker and Container Best Practices
### 4.1. Dockerfiles
- **Base Images**: Always use the latest, most specific official images (e.g., `php:8.2-apache` instead of just `php:latest`). Prefer Debian/Ubuntu bases for broad compatibility, or Alpine if specifically requested for size optimization.
- **Layer Optimization**: Combine `RUN` commands where logical to reduce image layers. 
- **Package Management**:
  - Use the `--no-install-recommends` flag with `apt-get install` to minimize bloat.
  - ALWAYS clear the apt cache in the same layer: `rm -rf /var/lib/apt/lists/*`.
- **Permissions**: Avoid running services as `root`. If necessary, create a dedicated user/group and use the `USER` directive. Handle file permissions carefully when mounting volumes.
- **Extensions (PHP)**: Use standard helper scripts provided by the official PHP images (like `docker-php-ext-install`, `docker-php-ext-enable`) rather than manual compilation unless necessary.

### 4.2. Docker Compose
- **Service Naming**: Container, network, and volume names should be explicitly defined and prefixed relative to the project or flavor (e.g., `lamp-dev-mysql`, `lamp-dev-network`).
- **Networks**: Do not rely on the default bridge network. Explicitly define custom bridge networks for secure service-to-service communication.
- **Volumes**: Use named volumes for persistent data (like MySQL databases) and bind mounts for source code.
- **Healthchecks**: Always define a `healthcheck` block for critical services (especially MySQL) so dependent services can use `depends_on: ... condition: service_healthy`.

## 5. Configuration Patterns
- **Apache**: Custom vhost files should be mapped to `/etc/apache2/sites-available/` and enabled via `a2ensite`. Document any specific `.htaccess` requirements.
- **MySQL**: Initialization scripts (SQL or shell) should be placed in `/docker-entrypoint-initdb.d/`. Ensure they check if the database/tables already exist to remain idempotent.
- **PHP**: Custom `php.ini` settings should be provided via localized `.ini` files mapped to `/usr/local/etc/php/conf.d/` rather than overwriting the main `php.ini`.

## 6. Testing and Validation
- **Infrastructure Tests**: Any new flavor or significant change must have corresponding tests. Tests should ideally spin up the environment and perform `curl` requests or database queries to verify functionality.
- **Debugging**: When a container fails, prioritize checking `docker logs <container_name>` and inspect the container state. Ensure networking isn't the root cause before changing service configurations.

## 7. Communication, Workflow, and Documentation
- **Planning**: Architectural changes (new services, base image swaps, major version bumps) require user validation and a clear plan. Do not execute destructive changes without approval.
- **Documentation**: Non-trivial changes must be summarized clearly. Use Markdown, code blocks, and explain the *why*, not just the *what*.
- **Language**: Comments in code, Dockerfiles, and compose files must be in English to maintain project consistency.
