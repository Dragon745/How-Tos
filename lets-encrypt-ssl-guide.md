# Complete Let's Encrypt SSL Certificate Guide

This guide will help you obtain free SSL certificates using Let's Encrypt for both Apache and Nginx web servers.

## Prerequisites

- Domain name with proper DNS configuration
- Web server (Apache or Nginx) installed and configured
- Server accessible from the internet (port 80 and 443)
- Root or sudo access
- Domain pointing to your server's IP address

## Example Configuration

This guide uses these example domains:

- **Domain**: example.com
- **Subdomain**: www.example.com
- **Server IP**: 203.0.113.1

Replace these with your actual domain names and server details.

## Step 1: Install Certbot

### For Debian/Ubuntu Systems

```bash
# Update system
sudo apt update
sudo apt upgrade -y

# Install Certbot and web server plugins
sudo apt install certbot python3-certbot-apache python3-certbot-nginx -y
```

### For CentOS/RHEL Systems

```bash
# Install EPEL repository
sudo yum install epel-release -y

# Install Certbot
sudo yum install certbot python3-certbot-apache python3-certbot-nginx -y
```

### For Other Systems

```bash
# Install Certbot using snap (universal method)
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## Step 2: Apache SSL Certificate Setup

### Method 1: Automatic Apache Configuration

```bash
# Get SSL certificate and configure Apache automatically
sudo certbot --apache -d example.com -d www.example.com

# For multiple domains
sudo certbot --apache -d example.com -d www.example.com -d subdomain.example.com
```

### Method 2: Manual Apache Configuration

#### 2.1: Create Apache Virtual Host

```bash
sudo nano /etc/apache2/sites-available/example.com.conf
```

Add this configuration:

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com

    <Directory /var/www/example.com>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined
</VirtualHost>
```

#### 2.2: Enable the Site

```bash
# Create document root
sudo mkdir -p /var/www/example.com
sudo chown -R www-data:www-data /var/www/example.com

# Create a test page
echo "<html><body><h1>Welcome to example.com</h1></body></html>" | sudo tee /var/www/example.com/index.html

# Enable the site
sudo a2ensite example.com.conf
sudo systemctl reload apache2
```

#### 2.3: Get SSL Certificate

```bash
# Get certificate only (without auto-configuration)
sudo certbot certonly --apache -d example.com -d www.example.com
```

#### 2.4: Configure Apache for SSL

```bash
sudo nano /etc/apache2/sites-available/example.com-ssl.conf
```

Add SSL configuration:

```apache
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    <Directory /var/www/example.com>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example.com_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_ssl_access.log combined
</VirtualHost>
```

#### 2.5: Enable SSL Site

```bash
sudo a2enmod ssl
sudo a2enmod rewrite
sudo a2ensite example.com-ssl.conf
sudo systemctl reload apache2
```

## Step 3: Nginx SSL Certificate Setup

### Method 1: Automatic Nginx Configuration

```bash
# Get SSL certificate and configure Nginx automatically
sudo certbot --nginx -d example.com -d www.example.com

# For multiple domains
sudo certbot --nginx -d example.com -d www.example.com -d subdomain.example.com
```

### Method 2: Manual Nginx Configuration

#### 3.1: Create Nginx Server Block

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
}
```

#### 3.2: Enable the Site

```bash
# Create document root
sudo mkdir -p /var/www/example.com
sudo chown -R www-data:www-data /var/www/example.com

# Create a test page
echo "<html><body><h1>Welcome to example.com</h1></body></html>" | sudo tee /var/www/example.com/index.html

# Enable the site
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

#### 3.3: Get SSL Certificate

```bash
# Get certificate only (without auto-configuration)
sudo certbot certonly --nginx -d example.com -d www.example.com
```

#### 3.4: Configure Nginx for SSL

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Replace with SSL configuration:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html index.htm;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
}
```

#### 3.5: Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Step 4: Standalone Certificate (For Non-Web Services)

For services that don't use Apache or Nginx (like mail servers):

```bash
# Stop web server temporarily
sudo systemctl stop apache2
# OR
sudo systemctl stop nginx

# Get certificate
sudo certbot certonly --standalone -d example.com -d mail.example.com

# Restart web server
sudo systemctl start apache2
# OR
sudo systemctl start nginx
```

## Step 5: Wildcard Certificate Setup

For wildcard certificates (e.g., \*.example.com):

```bash
# Install DNS plugin (for manual DNS verification)
sudo apt install python3-certbot-dns-cloudflare -y

# Get wildcard certificate
sudo certbot certonly --manual --preferred-challenges=dns -d example.com -d *.example.com
```

Follow the DNS challenge instructions provided by Certbot.

## Step 6: Certificate Renewal

### Automatic Renewal Setup

```bash
# Test automatic renewal
sudo certbot renew --dry-run

# Check renewal status
sudo certbot certificates
```

### Manual Renewal

```bash
# Renew all certificates
sudo certbot renew

# Renew specific certificate
sudo certbot renew --cert-name example.com
```

### Cron Job for Automatic Renewal

```bash
# Add to crontab
sudo crontab -e
```

Add this line:

```
0 12 * * * /usr/bin/certbot renew --quiet
```

## Step 7: Certificate Management

### List All Certificates

```bash
sudo certbot certificates
```

### Delete Certificate

```bash
sudo certbot delete --cert-name example.com
```

### Revoke Certificate

```bash
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

## Step 8: Troubleshooting

### Common Issues and Solutions

#### 1. "Challenge failed" Error

```bash
# Check if port 80 is open
sudo netstat -tlnp | grep :80

# Check firewall
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

#### 2. "DNS challenge failed"

```bash
# Verify DNS records
nslookup example.com
dig example.com

# Check DNS propagation
dig +trace example.com
```

#### 3. "Certificate not found"

```bash
# Check certificate location
sudo ls -la /etc/letsencrypt/live/example.com/

# Verify certificate files
sudo openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -text -noout
```

#### 4. "Web server not responding"

```bash
# Check web server status
sudo systemctl status apache2
sudo systemctl status nginx

# Check web server configuration
sudo apache2ctl configtest
sudo nginx -t
```

### Debug Commands

```bash
# Check Certbot logs
sudo tail -f /var/log/letsencrypt/letsencrypt.log

# Test SSL certificate
openssl s_client -connect example.com:443 -servername example.com

# Check certificate expiration
openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -noout -dates
```

## Step 9: Security Best Practices

### 1. SSL Configuration

#### For Apache (mod_ssl)

```apache
# Add to SSL VirtualHost
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder on
SSLCompression off
SSLSessionTickets off
```

#### For Nginx

```nginx
# Enhanced SSL configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

### 2. Security Headers

#### For Apache

```apache
# Add to VirtualHost
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
Header always set X-XSS-Protection "1; mode=block"
```

#### For Nginx

```nginx
# Add to server block
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

### 3. Certificate Monitoring

```bash
# Create monitoring script
sudo nano /usr/local/bin/ssl-monitor.sh
```

Add this content:

```bash
#!/bin/bash
CERT="/etc/letsencrypt/live/example.com/cert.pem"
DAYS=30

if [ -f "$CERT" ]; then
    EXPIRY=$(openssl x509 -enddate -noout -in "$CERT" | cut -d= -f2)
    EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
    CURRENT_EPOCH=$(date +%s)
    DAYS_LEFT=$(( ($EXPIRY_EPOCH - $CURRENT_EPOCH) / 86400 ))

    if [ $DAYS_LEFT -lt $DAYS ]; then
        echo "SSL certificate for example.com expires in $DAYS_LEFT days"
        # Add notification logic here
    fi
fi
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/ssl-monitor.sh
```

## Step 10: Advanced Configuration

### Multiple Domains on Single Certificate

```bash
# Get certificate for multiple domains
sudo certbot --apache -d example.com -d www.example.com -d api.example.com -d admin.example.com
```

### Separate Certificates for Different Domains

```bash
# Get separate certificates
sudo certbot --apache -d example.com
sudo certbot --apache -d anotherdomain.com
```

### Certificate for Subdomains

```bash
# Get certificate for specific subdomains
sudo certbot --apache -d example.com -d mail.example.com -d ftp.example.com
```

## Verification Checklist

- [ ] Certbot installed successfully
- [ ] Domain DNS configured correctly
- [ ] Web server running and accessible
- [ ] SSL certificate obtained
- [ ] Web server configured for SSL
- [ ] Automatic renewal configured
- [ ] Security headers implemented
- [ ] SSL configuration optimized
- [ ] Certificate monitoring set up
- [ ] Backup procedures in place

## Important Notes

1. **Rate Limits**: Let's Encrypt has rate limits (50 certificates per domain per week)
2. **Certificate Validity**: Certificates are valid for 90 days
3. **Renewal**: Set up automatic renewal to avoid expiration
4. **Backup**: Keep backups of your certificates and configurations
5. **Monitoring**: Monitor certificate expiration dates
6. **Security**: Use strong SSL configurations and security headers

Your SSL certificates are now properly configured and secured!
