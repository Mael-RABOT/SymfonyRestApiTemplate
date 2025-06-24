# Symfony REST API Template - Production Deployment Guide

This guide explains how to deploy the Symfony REST API template to production on a Linux VPS.

## Prerequisites

- Linux VPS with root access
- Apache2 web server
- Docker and Docker Compose
- MariaDB/MySQL database
- Domain name pointing to your VPS IP
- SSL certificate (Let's Encrypt recommended)

## Step 1: Clone and Setup the Project

```bash
# Clone the repository to your VPS
git clone <your-repo-url> /var/www/symfony-api
cd /var/www/symfony-api

# Set proper ownership
sudo chown -R $USER:$USER /var/www/symfony-api
```

## Step 2: Generate Production Secrets

```bash
# Make the script executable
chmod +x scripts/generate-production-secrets.sh

# Run the secret generator
./scripts/generate-production-secrets.sh
```

This script will generate:
- `APP_SECRET` for Symfony
- `JWT_PASSPHRASE` for JWT authentication
- `MYSQL_PASSWORD` and `MYSQL_ROOT_PASSWORD` for database
- JWT private and public keys in `config/jwt/`

## Step 3: Configure Environment Variables

```bash
# Copy the production environment template
cp env.production.example .env.production

# Edit the configuration
nvim .env.production
```

Update the following values in `.env.production`:

```env
# Replace with your actual domain
CORS_ALLOW_ORIGIN='^https?://(yourproject\.api\.yourdomain\.com|www\.yourdomain\.com|yourdomain\.com)$'

# Database configuration (use the generated passwords)
DATABASE_URL="mysql://your_db_user:your_generated_password@127.0.0.1:3306/your_db_name?serverVersion=10.11&charset=utf8"
MYSQL_DATABASE=your_db_name
MYSQL_USER=your_db_user
MYSQL_PASSWORD=your_generated_password
MYSQL_ROOT_PASSWORD=your_generated_root_password

# Use the generated secrets
APP_SECRET=your_generated_app_secret
JWT_PASSPHRASE=your_generated_jwt_passphrase
```

## Step 4: Create Symbolic Link for Environment

```bash
# Create symbolic link from .env.production to .env
ln -sf .env.production .env
```

## Step 5: Update Configuration Files with Your Domain

### Update Apache Configuration

```bash
# Edit the Apache configuration file
nvim apache/<project>.api.<domain>.com.conf
```

Replace all instances of `<project>` and `<domain>` with your actual values:

```apache
<VirtualHost *:80>
    ServerName yourproject.api.yourdomain.com
    ServerAdmin webmaster@yourdomain.com

    # Security headers
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "no-referrer-when-downgrade"

    # Proxy configuration - Change port if needed
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/

    # Timeout settings
    ProxyTimeout 300
    ProxyBadHeader Ignore

    # Logging
    ErrorLog ${APACHE_LOG_DIR}/yourproject.api.yourdomain.com_error.log
    CustomLog ${APACHE_LOG_DIR}/yourproject.api.yourdomain.com_access.log combined

    RewriteEngine on
    RewriteCond %{SERVER_NAME} =yourproject.api.yourdomain.com
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

### Update CORS Configuration

```bash
# Edit the CORS configuration file
nvim config/packages/nelmio_cors.yaml
```

Replace all instances of `<project>` and `<domain>` with your actual values:

```yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin:
            - '^https?://(localhost|127\.0\.0\.1|yourproject\.api\.yourdomain\.com)(:\d+)?$'
            - '^https://yourproject\.api\.yourdomain\.com$'
            - '^https://yourproject\.api\.yourdomain\.com$'
            - '^https?://89\.168\.62\.189(:\d+)?$'
        allow_methods: ['GET', 'OPTIONS', 'POST', 'PUT', 'PATCH', 'DELETE']
        allow_headers: ['Content-Type', 'Authorization']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api/':
            origin_regex: true
            allow_origin:
                - '^https?://(localhost|127\.0\.0\.1|yourproject\.api\.yourdomain\.com)(:\d+)?$'
                - '^https://yourproject\.api\.yourdomain\.com$'
                - '^https://yourproject\.api\.yourdomain\.com$'
                - '^https?://89\.168\.62\.189(:\d+)?$'
            allow_methods: ['GET', 'OPTIONS', 'POST', 'PUT', 'PATCH', 'DELETE']
            allow_headers: ['Content-Type', 'Authorization']
            expose_headers: ['Link']
            max_age: 3600
```

**Note:** Replace `yourproject` with your actual project name and `yourdomain.com` with your actual domain name.

## Step 6: Configure Apache Virtual Host

### Copy Apache Configuration

```bash
# Copy the Apache configuration file
sudo cp apache/<project>.api.<domain>.com.conf /etc/apache2/sites-available/

# Rename it to match your domain
sudo mv /etc/apache2/sites-available/<project>.api.<domain>.com.conf /etc/apache2/sites-available/yourproject.api.yourdomain.com.conf
```

### Edit the Apache Configuration

```bash
sudo nvim /etc/apache2/sites-available/yourproject.api.yourdomain.com.conf
```

Update the configuration for your domain:

```apache
<VirtualHost *:80>
    ServerName yourproject.api.yourdomain.com
    ServerAdmin webmaster@yourdomain.com

    # Security headers
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "no-referrer-when-downgrade"

    # Proxy configuration - Change port if needed
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/

    # Timeout settings
    ProxyTimeout 300
    ProxyBadHeader Ignore

    # Logging
    ErrorLog ${APACHE_LOG_DIR}/yourproject.api.yourdomain.com_error.log
    CustomLog ${APACHE_LOG_DIR}/yourproject.api.yourdomain.com_access.log combined

    RewriteEngine on
    RewriteCond %{SERVER_NAME} =yourproject.api.yourdomain.com
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

### Enable Apache Modules and Site

```bash
# Enable required Apache modules
sudo a2enmod proxy proxy_http headers rewrite ssl

# Enable your site
sudo a2ensite yourproject.api.yourdomain.com.conf

# Test Apache configuration
sudo apache2ctl configtest

# Reload Apache
sudo systemctl reload apache2
```

## Step 7: Change Application Port (Optional)

If you want to use a different port than 8080, you need to update both the Docker Compose file and Apache configuration:

### Update Docker Compose

```bash
nvim docker-compose.prod.yml
```

Change the port mapping:
```yaml
ports:
  - "127.0.0.1:YOUR_CUSTOM_PORT:80"  # e.g., 8081, 9000, etc.
```

### Update Apache Configuration

```bash
sudo nvim /etc/apache2/sites-available/yourproject.api.yourdomain.com.conf
```

Update the proxy configuration:
```apache
ProxyPass / http://127.0.0.1:YOUR_CUSTOM_PORT/
ProxyPassReverse / http://127.0.0.1:YOUR_CUSTOM_PORT/
```

## Step 8: Deploy the Application

```bash
# Make the deployment script executable
chmod +x scripts/deploy-production.sh

# Run the deployment
./scripts/deploy-production.sh
```

This script will:
- Build the FrankenPHP container
- Start the application
- Run database migrations
- Clear and warm up the cache
- Set proper permissions
- Test the API endpoint

## Step 9: Set Up SSL Certificate

```bash
# Install Certbot (if not already installed)
sudo apt update
sudo apt install certbot python3-certbot-apache

# Obtain SSL certificate
sudo certbot --apache -d yourproject.api.yourdomain.com

# Test automatic renewal
sudo certbot renew --dry-run
```

## Step 10: Database Setup

If you're using an external MariaDB/MySQL server:

```bash
# Connect to your database server
mysql -u root -p

# Create database and user
CREATE DATABASE <db_name>;
CREATE USER '<db_user>'@'%' IDENTIFIED BY '<db_pssw>';
GRANT ALL PRIVILEGES ON <db_name>.* TO '<db_user>'@'%';
FLUSH PRIVILEGES;
EXIT;
```

## Step 11: Final Configuration

### Set Proper File Permissions

```bash
# Set proper ownership for the application
sudo chown -R www-data:www-data /var/www/symfony-api/var/
sudo chmod -R 755 /var/www/symfony-api/var/

# Set proper permissions for JWT keys
sudo chmod 644 config/jwt/public.pem
sudo chmod 600 config/jwt/private.pem
```

### Create Systemd Service (Optional)

For automatic startup on boot:

```bash
sudo nvim /etc/systemd/system/symfony-api.service
```

Add the following content:
```ini
[Unit]
Description=Symfony API Docker Compose
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/var/www/symfony-api
ExecStart=/usr/local/bin/docker-compose -f docker-compose.prod.yml up -d
ExecStop=/usr/local/bin/docker-compose -f docker-compose.prod.yml down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

Enable the service:
```bash
sudo systemctl enable symfony-api.service
```

## Step 12 (optional): Enable TCP port

```bash
nvim /etc/iptables/rules.v4
```

Add the following line:

```bash
A INPUT -p tcp -m state --state NEW -m tcp --dport <port> -j ACCEPT
```

Replace `<port>` with the port you want to allow.

Reload iptables:

```bash
sudo iptables-restore < /etc/iptables/rules.v4
```

## Step 13: Verify Deployment

### Test the API

```bash
# Test health endpoint
curl -f https://yourproject.api.yourdomain.com/health

# Test API documentation (if using API Platform)
curl -f https://yourproject.api.yourdomain.com/api
```

### Check Logs

```bash
# Check application logs
docker-compose -f docker-compose.prod.yml logs -f frankenphp

# Check Apache logs
sudo tail -f /var/log/apache2/yourproject.api.yourdomain.com_error.log
sudo tail -f /var/log/apache2/yourproject.api.yourdomain.com_access.log
```

## Troubleshooting

### Common Issues

1. **Port already in use**: Change the port in `docker-compose.prod.yml` and Apache configuration
2. **Permission denied**: Ensure proper file ownership and permissions
3. **Database connection failed**: Verify database credentials and network connectivity
4. **JWT keys not found**: Run the secret generation script again

### Useful Commands

```bash
# Restart the application
docker-compose -f docker-compose.prod.yml restart

# View running containers
docker-compose -f docker-compose.prod.yml ps

# Access container shell
docker-compose -f docker-compose.prod.yml exec frankenphp bash

# Clear Symfony cache
docker-compose -f docker-compose.prod.yml exec frankenphp php bin/console cache:clear --env=prod
```

## Security Considerations

- Keep your `.env.production` file secure and never commit it to version control
- Regularly update your secrets using the generation script
- Monitor your logs for suspicious activity
- Keep your system and Docker images updated
- Use strong passwords and consider using a secrets management service
- Enable firewall rules to restrict access to necessary ports only

## Maintenance

### Regular Updates

```bash
# Pull latest changes
git pull origin main

# Rebuild and restart
docker-compose -f docker-compose.prod.yml down
docker-compose -f docker-compose.prod.yml build --no-cache
docker-compose -f docker-compose.prod.yml up -d

# Run migrations
docker-compose -f docker-compose.prod.yml exec frankenphp php bin/console doctrine:migrations:migrate --no-interaction --env=prod
```

### Backup Strategy

```bash
# Backup database
mysqldump -u <db_user> -p <db_name> > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup application files
tar -czf symfony_backup_$(date +%Y%m%d_%H%M%S).tar.gz /var/www/symfony-api
```

Your Symfony REST API is now deployed and ready for production use!