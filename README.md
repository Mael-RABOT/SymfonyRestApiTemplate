# Symfony REST API Template

A production-ready template for creating REST APIs with Symfony, featuring JWT authentication, API Platform integration, and Docker deployment.

## Overview

This template provides a solid foundation for building REST APIs with Symfony. It's designed for personal projects and small-scale applications, offering a clean, secure, and maintainable codebase that can be quickly deployed to production.

## Features

- **Symfony 7** with API Platform
- **JWT Authentication** with LexikJWTAuthenticationBundle
- **Docker** support with FrankenPHP for production
- **MariaDB/MySQL** database integration
- **CORS** configuration for cross-origin requests
- **Health check** endpoint
- **User management** with secure password hashing
- **Production-ready** deployment scripts
- **Apache** reverse proxy configuration

## Quick Start

### Development

```bash
# Clone the repository
git clone <your-repo-url>
cd SymfonyRestApiTemplate

# Start development environment
docker-compose up -d

# Install dependencies
docker-compose exec php composer install

# Run database migrations
docker-compose exec php php bin/console doctrine:migrations:migrate

# Create a user (optional)
docker-compose exec php php bin/console app:create-user
```

### Production Deployment

For detailed production deployment instructions, see [DEPLOYMENT.md](DEPLOYMENT.md).

## Production Architecture

### Database Configuration

In production mode, this template is configured to use a **database running on the host** rather than inside Docker containers. This design choice makes it easier to manage multiple small APIs that can share the same database server.

**Benefits:**
- Centralized database management
- Easier backup and maintenance
- Reduced resource usage
- Simplified monitoring

**Alternative Setup:**
If you prefer to run the database inside Docker (as in development mode), you can modify `docker-compose.prod.yml` by copying the MariaDB service from the development `docker-compose.yml` file. The database will be automatically created within the Docker environment.

## Security Features

### Registration Endpoint

The `/auth/register` route is **disabled by default** for security reasons. This template is designed for personal endpoints rather than large-scale services where user registration might not be necessary.

**To enable registration:**
Edit `src/Controller/AuthController.php` and remove the early 403 response in the register method:

```php
// Remove or comment out those line:
return $this->json([
    'message' => 'Registration is disabled, please contact the administrator.',
    'status' => 'error',
    'code' => 403,
    'timestamp' => (new \DateTime())->format('c'),
]);
```

### JWT Authentication

- Secure JWT token generation and validation
- Configurable token expiration
- Private/public key pair authentication

### CORS Configuration

Pre-configured CORS settings for secure cross-origin requests.

## API Endpoints

- `GET /health` - Health check endpoint
- `POST /auth/login` - User authentication
- `POST /auth/register` - User registration (disabled by default)
- `GET /api` - API Platform documentation (if enabled)

## Development Commands

```bash
# Create a new user
php bin/console app:create-user

# List all users
php bin/console app:list-users

# Delete a user
php bin/console app:delete-user

# Clear cache
php bin/console cache:clear
```

## Configuration

### Environment Variables

- `APP_SECRET` - Symfony application secret
- `JWT_PASSPHRASE` - JWT key passphrase
- `DATABASE_URL` - Database connection string
- `CORS_ALLOW_ORIGIN` - Allowed CORS origins

### Database

The template supports both MariaDB and MySQL. Update the `DATABASE_URL` in your environment file to match your database configuration.

## License

This project is licensed under the **GNU General Public License v3.0 (GPLv3)**. This means:

- You are free to use, modify, and distribute this template
- Any derivative works must also be licensed under GPLv3
- Source code must be made available when distributing

## Contributing

I welcome contributions and improvements to this template! Feel free to:

- Submit bug reports
- Suggest new features
- Create pull requests
- Improve documentation

### How to Contribute

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Some examples of usage

*a.k.a the shameless plug section*

These two project are powered by this template:
- My portfolio website:
  - [maelrabot.com](https://maelrabot.com)
  - [github repository](https://github.com/Mael-RABOT/portfolio)
- My personal cloud:
  - [cloud.maelrabot.com](https://cloud.maelrabot.com)
  - [github repository](https://github.com/Mael-RABOT/CloudInterface)

## Support

This template is designed to be self-contained and well-documented. For deployment assistance, refer to the [DEPLOYMENT.md](DEPLOYMENT.md) file.

## Acknowledgments

Built with Symfony, API Platform, and other open-source components. Special thanks to the Symfony community for their excellent documentation and tools.
