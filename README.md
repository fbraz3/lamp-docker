[![Build Base Images](https://github.com/fbraz3/lamp-docker/actions/workflows/base-images.yml/badge.svg)](https://github.com/fbraz3/lamp-docker/actions/workflows/base-images.yml)
[![Build Phalcon Images](https://github.com/fbraz3/lamp-docker/actions/workflows/phalcon-images.yml/badge.svg)](https://github.com/fbraz3/lamp-docker/actions/workflows/phalcon-images.yml)
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/fbraz3/lamp-docker)

# Braz LAMP Docker Image

This repository provides Docker images for LAMP stack (`L`inux, `A`pache, `M`ariaDB, and `P`HP-FPM).

It is designed to simplify the deployment of PHP applications with a robust and modern environment, offering both **Development** and **Production** versions to meet different deployment needs.

The images are built on top of other modular images from the `fbraz3` ecosystem, ensuring flexibility, maintainability, and ease of use.

We also provide an AI generated [DeepWiki Page](https://deepwiki.com/fbraz3/lamp-docker) with more technical information.

💡 For a complete list of available images, please visit the [PHP System Docs](https://github.com/fbraz3/php-system-docs) page.

## Table of Contents

- [Braz LAMP Docker Image](#braz-lamp-docker-image)
  - [Image Versions](#image-versions)
  - [Tags](#tags)
  - [Flavors](#flavors)
  - [How to Use](#how-to-use)
    - [Development Version](#development-version)
    - [Production Version](#production-version)
  - [Custom SQL Scripts](#custom-sql-scripts)
  - [Security Considerations](#security-considerations)
  - [Environment Variables](#environment-variables)
  - [Manage PHP Directives via Environment Variables](#manage-php-directives-via-environment-variables)
  - [Cronjobs](#cronjobs)
  - [Sending Mails](#sending-mails)
  - [Contribution](#contribution)
  - [Donation](#donation)
  - [License](#license)

## Image Versions

This project provides two distinct versions for different use cases:

### Development Version (`-dev` suffix)
- **Purpose**: Designed for local development and testing
- **Features**:
  - MariaDB with **no root password** (empty password)
  - phpMyAdmin accessible at `/pma/` endpoint
  - Relaxed security settings for ease of development
  - Direct database access without authentication

### Production Version (no suffix)
- **Purpose**: Optimized for production environments
- **Features**:
  - MariaDB with **mandatory root password** configuration
  - **Enforced password security**: Container will fail to start if using default password
  - **Custom SQL scripts support**: Execute custom SQL files at startup
  - **No phpMyAdmin** included for security
  - Secure database configuration
  - Root password configurable via environment variables
  - Anonymous users and test databases removed
  - Enhanced security settings

## Tags

The image follows a consistent tagging scheme:

### Base Images (Vanilla LAMP Stack)
- **Production**: `fbraz3/lamp:{php_version}` (e.g., `8.2`, `8.3`, `8.4`)
- **Development**: `fbraz3/lamp:{php_version}-dev` (e.g., `8.2-dev`, `8.3-dev`)

### Phalcon Images (With Phalcon Framework)
- **Production**: `fbraz3/lamp:{php_version}-phalcon` (e.g., `8.2-phalcon`, `8.3-phalcon`)
- **Development**: `fbraz3/lamp:{php_version}-phalcon-dev` (e.g., `8.2-phalcon-dev`)

### Latest Tags
- `fbraz3/lamp:latest` - Latest production version
- `fbraz3/lamp:latest-dev` - Latest development version  
- `fbraz3/lamp:latest-phalcon` - Latest Phalcon production version
- `fbraz3/lamp:latest-phalcon-dev` - Latest Phalcon development version

### Architecture Support
- All images support both `amd64` and `arm64` architectures
- LAMP variants are also available as `fbraz3/lamp:{tag}`

## Flavors

This image supports multiple flavors to cater to different use cases:

- `Vanilla`: A standard LAMP stack with no additional frameworks.
- `Phalcon`: Includes the Phalcon PHP framework pre-installed.

## How to Use

### Development Version

Perfect for local development with easy database access and debugging tools.

```yaml
# docker-compose.yml
services:
  web:
    image: fbraz3/lamp:8.4-dev  # or fbraz3/lamp:8.4-phalcon-dev
    volumes:
      - ./:/app/public/
    ports:
      - "127.0.0.1:80:80"
```

The development version includes:
- **phpMyAdmin** accessible at `http://localhost/pma/`
- **MariaDB** with no root password (empty password)
- Direct database access for development

### Production Version

Optimized for production environments with enhanced security.

```yaml
# docker-compose.yml
services:
  web:
    image: fbraz3/lamp:8.4  # or fbraz3/lamp:8.4-phalcon
    volumes:
      - ./:/app/public/
      - ./sql-scripts:/sql-scripts  # Optional: Custom SQL scripts
    ports:
      - "127.0.0.1:80:80"
    environment:
      - MYSQL_ROOT_PASSWORD=your_secure_password_here
      - MYSQL_DATABASE=your_app_db
      - MYSQL_USER=app_user
      - MYSQL_PASSWORD=app_user_password
```

Run the following command to start the container:

```bash
docker-compose up -d
```

Your application will be accessible at `http://localhost/`.

## Custom SQL Scripts

The production version of this image supports the execution of custom SQL scripts at startup. 

To use this feature, place your SQL files in a directory (e.g., `./sql-scripts`) and mount it as a volume to the container:

```yaml
services:
  web:
    image: fbraz3/lamp:8.4  # Production version
    volumes:
      - ./:/app/public/
      - ./sql-scripts:/sql-scripts  # Custom SQL scripts directory
    ports:
      - "127.0.0.1:80:80"
    environment:
      - MYSQL_ROOT_PASSWORD=your_secure_password_here
```

SQL scripts in the mounted directory will be executed in alphabetical order during container startup.

## Security Considerations

- **Development Version**: This version has relaxed security settings for ease of development. It is not recommended for production use.
- **Production Version**: This version enforces strict security measures. Ensure that you configure the root password and remove any default or test databases.

## Environment Variables

You can configure various aspects of the container using environment variables. 

Commonly used variables include:

- `MYSQL_ROOT_PASSWORD`: Set the root password for MariaDB.
- `MYSQL_DATABASE`: Create a default database.
- `MYSQL_USER` and `MYSQL_PASSWORD`: Create a new user with access to the default database.

Refer to the [official MariaDB Docker image documentation](https://hub.docker.com/_/mariadb) for a complete list of environment variables.

## Manage PHP Directives via Environment Variables
This image allows you to customize PHP directives using environment variables. 

For detailed instructions, refer to the [php-fpm-docker documentation](https://github.com/fbraz3/php-fpm-docker#manage-php-directives-via-environment-variables).

## Cronjobs
Cronjobs can be configured by binding a file to `/cronfile` in the container. The system will automatically install and execute the jobs.

For more details, see the [php-fpm-docker documentation](https://github.com/fbraz3/php-fpm-docker#cronjobs).

## Sending Mails
To enable email sending, this image relies on the configuration provided in the `php-base-docker` project.

Follow the instructions in the [php-base-docker documentation](https://github.com/fbraz3/php-base-docker#sending-mails) to set up email functionality.

## Contribution
Contributions are welcome! Feel free to open issues or submit pull requests to improve the project.

Please visit the [CONTRIBUTING.md](CONTRIBUTING.md) file for guidelines on how to contribute to this project.

#### Useful links
- [How to create a pull request](https://docs.github.com/pt/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)
- [How to report an issue](https://docs.github.com/pt/issues/tracking-your-work-with-issues/creating-an-issue)

## Donation
I spend a lot of time and effort maintaining this project. If you find it useful, consider supporting me with a donation:
- [GitHub Sponsor](https://github.com/sponsors/fbraz3)
- [Patreon](https://www.patreon.com/fbraz3)

## License

This project is licensed under the [Apache License 2.0](LICENSE), so you can use it for personal and commercial projects. However, please note that the images are provided "as is" without any warranty or guarantee of any kind. Use them at your own risk.