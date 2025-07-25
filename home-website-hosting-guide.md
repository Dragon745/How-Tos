# Complete Home Website Hosting Guide with Debian

This comprehensive guide will help you set up a complete web hosting environment at home using Debian with LAMP stack (Linux, Apache, MySQL, PHP) and WordPress.

## Prerequisites

- Debian server or desktop with root/sudo access
- Static public IP address (recommended)
- Domain name(s) with DNS configuration
- Router with port forwarding capabilities
- Minimum 2GB RAM, 20GB storage
- Internet connection with reasonable upload speed

## Example Configuration

This guide uses these example domains:

- **Primary domain**: example.com
- **Additional domain**: mywebsite.org
- **Server IP**: 203.0.113.1
- **Local IP**: 192.168.1.100

Replace these with your actual domain names and server details.

## Step 1: System Preparation

### Update System

```bash
# Update package lists and upgrade system
sudo apt update
sudo apt upgrade -y

# Install essential packages
sudo apt install curl wget git unzip software-properties-common -y
```

### Set Hostname and Timezone

```bash
# Set hostname
sudo hostnamectl set-hostname webserver

# Set timezone
sudo timedatectl set-timezone Your/Timezone
# Example: sudo timedatectl set-timezone America/New_York
```

## Step 2: Install LAMP Stack

### Install Apache Web Server

```bash
# Install Apache
sudo apt install apache2 -y

# Enable Apache to start on boot
sudo systemctl enable apache2
sudo systemctl start apache2

# Check Apache status
sudo systemctl status apache2
```

### Install MySQL Database Server

```bash
# Install MySQL Server
sudo apt install mysql-server -y

# Secure MySQL installation
sudo mysql_secure_installation
```

**Follow the prompts:**

- Set root password: `Y` (recommended)
- Remove anonymous users: `Y`
- Disallow root login remotely: `Y`
- Remove test database: `Y`
- Reload privilege tables: `Y`

### Install PHP and Extensions

```bash
# Install PHP and common extensions
sudo apt install php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip php-fpm -y

# Install additional PHP extensions for WordPress
sudo apt install php-imagick php-common php-mysql php-xml php-xmlrpc php-curl php-gd php-imagick php-cli php-dev php-zip php-mbstring -y
```

### Verify LAMP Installation

```bash
# Check Apache version
apache2 -v

# Check MySQL version
mysql --version

# Check PHP version
php -v

# Test PHP with Apache
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Visit `http://your-server-ip/info.php` to verify PHP is working.

## Step 3: Configure PHP

### PHP Configuration for Production

```bash
# Backup original PHP configuration
sudo cp /etc/php/8.1/apache2/php.ini /etc/php/8.1/apache2/php.ini.backup

# Edit PHP configuration
sudo nano /etc/php/8.1/apache2/php.ini
```

**Recommended PHP settings for production:**

```ini
; Memory limit
memory_limit = 256M

; Upload limits
upload_max_filesize = 64M
post_max_size = 64M

; Execution time
max_execution_time = 300
max_input_time = 300

; Error reporting (disable for production)
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log

; Security settings
allow_url_fopen = Off
allow_url_include = Off

; Session security
session.cookie_httponly = 1
session.cookie_secure = 1
session.use_strict_mode = 1

; OPcache for performance
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 4000
opcache.revalidate_freq = 2
opcache.fast_shutdown = 1
```

### Configure PHP-FPM (Optional for Better Performance)

```bash
# Install PHP-FPM
sudo apt install php8.1-fpm -y

# Configure PHP-FPM
sudo nano /etc/php/8.1/fpm/php.ini
```

Add the same settings as above to the FPM configuration.

### Restart Services

```bash
# Restart Apache and PHP
sudo systemctl restart apache2

# If using PHP-FPM
sudo systemctl restart php8.1-fpm
```

## Step 4: Configure Apache for Multiple Domains

### Enable Required Apache Modules

```bash
# Enable required modules
sudo a2enmod rewrite
sudo a2enmod ssl
sudo a2enmod headers
sudo a2enmod expires
sudo a2enmod deflate

# Restart Apache
sudo systemctl restart apache2
```

### Create Directory Structure

```bash
# Create web directories
sudo mkdir -p /var/www/example.com/public_html
sudo mkdir -p /var/www/mywebsite.org/public_html

# Set proper permissions
sudo chown -R www-data:www-data /var/www/example.com
sudo chown -R www-data:www-data /var/www/mywebsite.org
sudo chmod -R 755 /var/www/example.com
sudo chmod -R 755 /var/www/mywebsite.org
```

### Configure Virtual Hosts

#### Create First Domain Configuration

```bash
sudo nano /etc/apache2/sites-available/example.com.conf
```

Add this configuration:

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    ServerAdmin webmaster@example.com

    # Logging
    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined

    # Directory configuration
    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Security headers
    Header always set X-Content-Type-Options nosniff
    Header always set X-Frame-Options DENY
    Header always set X-XSS-Protection "1; mode=block"

    # Compression
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/plain
        AddOutputFilterByType DEFLATE text/html
        AddOutputFilterByType DEFLATE text/xml
        AddOutputFilterByType DEFLATE text/css
        AddOutputFilterByType DEFLATE application/xml
        AddOutputFilterByType DEFLATE application/xhtml+xml
        AddOutputFilterByType DEFLATE application/rss+xml
        AddOutputFilterByType DEFLATE application/javascript
        AddOutputFilterByType DEFLATE application/x-javascript
    </IfModule>

    # Caching
    <IfModule mod_expires.c>
        ExpiresActive On
        ExpiresByType image/jpg "access plus 1 month"
        ExpiresByType image/jpeg "access plus 1 month"
        ExpiresByType image/gif "access plus 1 month"
        ExpiresByType image/png "access plus 1 month"
        ExpiresByType text/css "access plus 1 month"
        ExpiresByType application/pdf "access plus 1 month"
        ExpiresByType text/javascript "access plus 1 month"
        ExpiresByType application/javascript "access plus 1 month"
        ExpiresByType application/x-javascript "access plus 1 month"
        ExpiresByType application/x-shockwave-flash "access plus 1 month"
        ExpiresByType image/x-icon "access plus 1 year"
        ExpiresDefault "access plus 2 days"
    </IfModule>
</VirtualHost>
```

#### Create Second Domain Configuration

```bash
sudo nano /etc/apache2/sites-available/mywebsite.org.conf
```

Add similar configuration for the second domain:

```apache
<VirtualHost *:80>
    ServerName mywebsite.org
    ServerAlias www.mywebsite.org
    DocumentRoot /var/www/mywebsite.org/public_html
    ServerAdmin webmaster@mywebsite.org

    # Logging
    ErrorLog ${APACHE_LOG_DIR}/mywebsite.org_error.log
    CustomLog ${APACHE_LOG_DIR}/mywebsite.org_access.log combined

    # Directory configuration
    <Directory /var/www/mywebsite.org/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Security headers
    Header always set X-Content-Type-Options nosniff
    Header always set X-Frame-Options DENY
    Header always set X-XSS-Protection "1; mode=block"

    # Compression and caching (same as above)
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/plain
        AddOutputFilterByType DEFLATE text/html
        AddOutputFilterByType DEFLATE text/xml
        AddOutputFilterByType DEFLATE text/css
        AddOutputFilterByType DEFLATE application/xml
        AddOutputFilterByType DEFLATE application/xhtml+xml
        AddOutputFilterByType DEFLATE application/rss+xml
        AddOutputFilterByType DEFLATE application/javascript
        AddOutputFilterByType DEFLATE application/x-javascript
    </IfModule>

    <IfModule mod_expires.c>
        ExpiresActive On
        ExpiresByType image/jpg "access plus 1 month"
        ExpiresByType image/jpeg "access plus 1 month"
        ExpiresByType image/gif "access plus 1 month"
        ExpiresByType image/png "access plus 1 month"
        ExpiresByType text/css "access plus 1 month"
        ExpiresByType application/pdf "access plus 1 month"
        ExpiresByType text/javascript "access plus 1 month"
        ExpiresByType application/javascript "access plus 1 month"
        ExpiresByType application/x-javascript "access plus 1 month"
        ExpiresByType application/x-shockwave-flash "access plus 1 month"
        ExpiresByType image/x-icon "access plus 1 year"
        ExpiresDefault "access plus 2 days"
    </IfModule>
</VirtualHost>
```

### Enable Sites and Test Configuration

```bash
# Enable sites
sudo a2ensite example.com.conf
sudo a2ensite mywebsite.org.conf

# Disable default site
sudo a2dissite 000-default.conf

# Test Apache configuration
sudo apache2ctl configtest

# Restart Apache
sudo systemctl restart apache2
```

### Create Test Pages

```bash
# Create test page for first domain
echo "<html><body><h1>Welcome to example.com</h1><p>Your website is working!</p></body></html>" | sudo tee /var/www/example.com/public_html/index.html

# Create test page for second domain
echo "<html><body><h1>Welcome to mywebsite.org</h1><p>Your second website is working!</p></body></html>" | sudo tee /var/www/mywebsite.org/public_html/index.html
```

## Step 5: Configure Database

### Create Database and User for WordPress

```bash
# Access MySQL as root
sudo mysql -u root -p
```

In MySQL, create databases and users:

```sql
-- Create database for first domain
CREATE DATABASE example_com_db;
CREATE USER 'example_user'@'localhost' IDENTIFIED BY 'your_secure_password_here';
GRANT ALL PRIVILEGES ON example_com_db.* TO 'example_user'@'localhost';
FLUSH PRIVILEGES;

-- Create database for second domain
CREATE DATABASE mywebsite_org_db;
CREATE USER 'mywebsite_user'@'localhost' IDENTIFIED BY 'your_secure_password_here';
GRANT ALL PRIVILEGES ON mywebsite_org_db.* TO 'mywebsite_user'@'localhost';
FLUSH PRIVILEGES;

-- Exit MySQL
EXIT;
```

### Optimize MySQL Configuration

```bash
# Backup original MySQL configuration
sudo cp /etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf.backup

# Edit MySQL configuration
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Add these optimizations under `[mysqld]`:

```ini
[mysqld]
# Basic settings
default-storage-engine = InnoDB
innodb_buffer_pool_size = 256M
innodb_log_file_size = 64M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT

# Connection settings
max_connections = 100
max_connect_errors = 10
table_open_cache = 2000
max_allowed_packet = 16M

# Query cache (if available)
query_cache_type = 1
query_cache_size = 32M
query_cache_limit = 2M

# Logging
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

### Restart MySQL

```bash
sudo systemctl restart mysql
```

## Step 6: Install WordPress

### Download and Install WordPress for First Domain

```bash
# Navigate to web directory
cd /var/www/example.com/public_html

# Download WordPress
sudo wget https://wordpress.org/latest.tar.gz

# Extract WordPress
sudo tar -xzvf latest.tar.gz

# Move WordPress files to public_html
sudo mv wordpress/* .
sudo rmdir wordpress

# Set proper permissions
sudo chown -R www-data:www-data /var/www/example.com/public_html
sudo chmod -R 755 /var/www/example.com/public_html
sudo chmod -R 644 /var/www/example.com/public_html/wp-config.php

# Clean up
sudo rm latest.tar.gz
```

### Configure WordPress

```bash
# Copy WordPress configuration file
sudo cp wp-config-sample.php wp-config.php

# Edit WordPress configuration
sudo nano wp-config.php
```

Update the database settings:

```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'example_com_db' );

/** MySQL database username */
define( 'DB_USER', 'example_user' );

/** MySQL database password */
define( 'DB_PASSWORD', 'your_secure_password_here' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

// Security keys (generate new ones)
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

// WordPress Database Table prefix
$table_prefix = 'wp_';

// For developers: WordPress debugging mode
define( 'WP_DEBUG', false );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

### Install WordPress for Second Domain

```bash
# Navigate to second domain directory
cd /var/www/mywebsite.org/public_html

# Download WordPress
sudo wget https://wordpress.org/latest.tar.gz

# Extract WordPress
sudo tar -xzvf latest.tar.gz

# Move WordPress files to public_html
sudo mv wordpress/* .
sudo rmdir wordpress

# Set proper permissions
sudo chown -R www-data:www-data /var/www/mywebsite.org/public_html
sudo chmod -R 755 /var/www/mywebsite.org/public_html
sudo chmod -R 644 /var/www/mywebsite.org/public_html/wp-config.php

# Clean up
sudo rm latest.tar.gz
```

Configure the second WordPress installation:

```bash
sudo nano wp-config.php
```

Update with second domain's database settings:

```php
define( 'DB_NAME', 'mywebsite_org_db' );
define( 'DB_USER', 'mywebsite_user' );
define( 'DB_PASSWORD', 'your_secure_password_here' );
```

## Step 7: Configure Firewall and Port Forwarding

### Configure UFW Firewall

```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow ssh

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Check UFW status
sudo ufw status
```

### Router Port Forwarding

Configure your router to forward these ports to your server:

- **Port 80** (HTTP) → Your server's local IP
- **Port 443** (HTTPS) → Your server's local IP
- **Port 22** (SSH) → Your server's local IP (optional)

## Step 8: DNS Configuration

### Understanding DNS Records

DNS (Domain Name System) translates domain names to IP addresses. You'll need to configure these records in your domain registrar's DNS settings.

### Required DNS Records

#### A Records (Address Records)

Point your domain to your server's public IP address:

**For example.com:**

```
Type: A
Name: @ (or leave blank)
Value: 203.0.113.1 (your public IP)
TTL: 3600 (or default)

Type: A
Name: www
Value: 203.0.113.1 (your public IP)
TTL: 3600 (or default)
```

**For mywebsite.org:**

```
Type: A
Name: @ (or leave blank)
Value: 203.0.113.1 (your public IP)
TTL: 3600 (or default)

Type: A
Name: www
Value: 203.0.113.1 (your public IP)
TTL: 3600 (or default)
```

### Optional DNS Records

#### CNAME Records (Canonical Name)

For subdomains that point to your main domain:

```
Type: CNAME
Name: mail
Value: example.com
TTL: 3600

Type: CNAME
Name: ftp
Value: example.com
TTL: 3600
```

#### MX Records (Mail Exchange)

If you plan to host email on the same server:

```
Type: MX
Name: @ (or leave blank)
Value: mail.example.com
Priority: 10
TTL: 3600
```

#### TXT Records

For email authentication and security:

**SPF Record (Sender Policy Framework):**

```
Type: TXT
Name: @ (or leave blank)
Value: v=spf1 mx a ip4:203.0.113.1 ~all
TTL: 3600
```

**DMARC Record (Domain-based Message Authentication):**

```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com
TTL: 3600
```

### DNS Configuration Steps

#### Step 1: Find Your Public IP

```bash
# Get your public IP address
curl ifconfig.me
# OR
curl ipinfo.io/ip
# OR
wget -qO- http://ipecho.net/plain
```

#### Step 2: Access Your Domain Registrar

1. Log into your domain registrar's website
2. Navigate to DNS management or DNS settings
3. Look for "DNS Records", "Zone File", or "DNS Configuration"

#### Step 3: Add DNS Records

**Common Domain Registrars:**

**GoDaddy:**

1. Go to "My Products" → "DNS"
2. Click "Manage" next to your domain
3. Scroll to "DNS Records" section
4. Click "Add" to add each record

**Namecheap:**

1. Go to "Domain List" → "Manage"
2. Click "Advanced DNS"
3. Add records in "Host Records" section

**Cloudflare:**

1. Go to your domain dashboard
2. Click "DNS" tab
3. Add records in "DNS Records" section

**Google Domains:**

1. Go to "My domains" → "Manage"
2. Click "DNS" tab
3. Add records in "DNS records" section

#### Step 4: Verify DNS Propagation

```bash
# Check if DNS is propagating
nslookup example.com
dig example.com
dig www.example.com

# Check from different locations
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
```

### DNS Troubleshooting

#### Common Issues:

**1. DNS Not Propagating**

- Wait 24-48 hours for full propagation
- Check with different DNS servers
- Verify records are saved correctly

**2. Wrong IP Address**

```bash
# Verify your current public IP
curl ifconfig.me
# Update DNS records if IP changed
```

**3. Subdomain Not Working**

```bash
# Test subdomain resolution
nslookup www.example.com
dig www.example.com
```

**4. Email Not Working**

- Ensure MX records are configured
- Check SPF and DMARC records
- Verify mail server configuration

### DNS Security Best Practices

#### 1. Use Strong TTL Values

```
TTL: 3600 (1 hour) - Good for most sites
TTL: 86400 (24 hours) - For stable configurations
TTL: 300 (5 minutes) - For frequent changes
```

#### 2. Implement DNS Security

**DNSSEC (if supported by your registrar):**

- Enable DNSSEC signing
- Add DS (Delegation Signer) records
- Monitor DNSSEC validation

#### 3. Monitor DNS Health

```bash
# Create DNS monitoring script
sudo nano /usr/local/bin/dns-monitor.sh
```

Add this content:

```bash
#!/bin/bash

DOMAINS=("example.com" "www.example.com" "mywebsite.org" "www.mywebsite.org")
EXPECTED_IP="203.0.113.1"

for domain in "${DOMAINS[@]}"; do
    RESOLVED_IP=$(dig +short $domain)
    if [ "$RESOLVED_IP" = "$EXPECTED_IP" ]; then
        echo "✓ $domain resolves correctly to $RESOLVED_IP"
    else
        echo "✗ $domain resolves to $RESOLVED_IP (expected $EXPECTED_IP)"
    fi
done
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/dns-monitor.sh
```

### DNS Record Examples for Different Scenarios

#### Basic Website Only

```
A Record: @ → 203.0.113.1
A Record: www → 203.0.113.1
```

#### Website with Email

```
A Record: @ → 203.0.113.1
A Record: www → 203.0.113.1
A Record: mail → 203.0.113.1
MX Record: @ → mail.example.com (Priority: 10)
TXT Record: @ → v=spf1 mx a ip4:203.0.113.1 ~all
```

#### Multiple Services

```
A Record: @ → 203.0.113.1
A Record: www → 203.0.113.1
A Record: mail → 203.0.113.1
A Record: ftp → 203.0.113.1
A Record: api → 203.0.113.1
MX Record: @ → mail.example.com (Priority: 10)
TXT Record: @ → v=spf1 mx a ip4:203.0.113.1 ~all
TXT Record: _dmarc → v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com
```

## Step 9: Install SSL Certificates

**Refer to our SSL guide for complete instructions:**

For Let's Encrypt SSL certificates, follow the guide: `lets-encrypt-ssl-guide.md`

Quick commands for your domains:

```bash
# Install SSL for first domain
sudo certbot --apache -d example.com -d www.example.com

# Install SSL for second domain
sudo certbot --apache -d mywebsite.org -d www.mywebsite.org
```

## Step 10: WordPress Setup

### Complete WordPress Installation

1. **Visit your domain**: `http://example.com`
2. **Choose language** and click "Continue"
3. **Fill in site information**:
   - Site Title: Your site name
   - Username: Choose a secure username
   - Password: Use a strong password
   - Email: Your email address
4. **Click "Install WordPress"**

### WordPress Security Recommendations

```bash
# Install security plugin
# In WordPress admin: Plugins → Add New → Search "Wordfence Security"

# Additional security measures
sudo nano /var/www/example.com/public_html/.htaccess
```

Add security rules to `.htaccess`:

```apache
# Security headers
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options nosniff
    Header always set X-Frame-Options DENY
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>

# Protect wp-config.php
<Files wp-config.php>
    Order allow,deny
    Deny from all
</Files>

# Disable directory browsing
Options -Indexes

# Protect against common attacks
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{QUERY_STRING} (\<|%3C).*script.*(\>|%3E) [NC,OR]
    RewriteCond %{QUERY_STRING} GLOBALS(=|\[|\%[0-9A-Z]{0,2}) [OR]
    RewriteCond %{QUERY_STRING} _REQUEST(=|\[|\%[0-9A-Z]{0,2}) [OR]
    RewriteCond %{QUERY_STRING} proc/self/environ [OR]
    RewriteCond %{QUERY_STRING} mosConfig [OR]
    RewriteCond %{QUERY_STRING} base64_(en|de)code[^(]*\([^)]*\) [OR]
    RewriteCond %{QUERY_STRING} (<|%3C)([^s]*s)+cript.*(>|%3E) [NC,OR]
    RewriteCond %{QUERY_STRING} (\<|%3C).*iframe.*(\>|%3E) [NC]
    RewriteRule .* - [F]
</IfModule>
```

## Step 11: Performance Optimization

### Install Redis for Caching

```bash
# Install Redis
sudo apt install redis-server -y

# Configure Redis
sudo nano /etc/redis/redis.conf
```

Add these settings:

```conf
maxmemory 256mb
maxmemory-policy allkeys-lru
```

### Install OPcache

```bash
# Install OPcache
sudo apt install php8.1-opcache -y

# Configure OPcache
sudo nano /etc/php/8.1/apache2/conf.d/10-opcache.ini
```

Add these settings:

```ini
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
opcache.enable_cli=1
```

### Restart Services

```bash
sudo systemctl restart apache2
sudo systemctl restart redis-server
```

## Step 12: Backup Configuration

### Create Backup Script

```bash
sudo nano /usr/local/bin/backup-websites.sh
```

Add this content:

```bash
#!/bin/bash

# Backup script for websites
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/websites"
MYSQL_USER="root"
MYSQL_PASS="your_mysql_root_password"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup websites
tar -czf $BACKUP_DIR/example_com_$DATE.tar.gz /var/www/example.com/public_html
tar -czf $BACKUP_DIR/mywebsite_org_$DATE.tar.gz /var/www/mywebsite.org/public_html

# Backup databases
mysqldump -u $MYSQL_USER -p$MYSQL_PASS example_com_db > $BACKUP_DIR/example_com_db_$DATE.sql
mysqldump -u $MYSQL_USER -p$MYSQL_PASS mywebsite_org_db > $BACKUP_DIR/mywebsite_org_db_$DATE.sql

# Keep only last 7 days of backups
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete

echo "Backup completed: $DATE"
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/backup-websites.sh
```

### Set Up Automated Backups

```bash
# Add to crontab
sudo crontab -e
```

Add this line for daily backups at 2 AM:

```
0 2 * * * /usr/local/bin/backup-websites.sh
```

## Step 13: Monitoring and Maintenance

### Install Monitoring Tools

```bash
# Install monitoring packages
sudo apt install htop iotop nethogs -y

# Install log monitoring
sudo apt install logwatch -y
```

### Create System Monitoring Script

```bash
sudo nano /usr/local/bin/system-monitor.sh
```

Add this content:

```bash
#!/bin/bash

# System monitoring script
echo "=== System Status ==="
echo "Date: $(date)"
echo "Uptime: $(uptime)"
echo "Disk Usage:"
df -h
echo "Memory Usage:"
free -h
echo "Apache Status:"
sudo systemctl status apache2 --no-pager -l
echo "MySQL Status:"
sudo systemctl status mysql --no-pager -l
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/system-monitor.sh
```

## Verification Checklist

- [ ] LAMP stack installed and configured
- [ ] PHP optimized for production
- [ ] Apache configured for multiple domains
- [ ] MySQL databases created and secured
- [ ] WordPress installed on all domains
- [ ] SSL certificates installed (refer to SSL guide)
- [ ] Firewall configured
- [ ] Port forwarding set up
- [ ] DNS records configured
- [ ] Backup system in place
- [ ] Monitoring tools installed
- [ ] Security measures implemented

## Important Notes

1. **Security**: Always use strong passwords and keep systems updated
2. **Backups**: Regular backups are essential for data protection
3. **Updates**: Keep WordPress, themes, and plugins updated
4. **Monitoring**: Monitor server resources and logs regularly
5. **SSL**: Use SSL certificates for all domains (see SSL guide)
6. **Performance**: Optimize images and use caching plugins
7. **Maintenance**: Regular maintenance prevents issues

Your home web hosting server is now ready for production use!
