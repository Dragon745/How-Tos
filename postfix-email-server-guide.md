# Complete Postfix Email Server Setup Guide for Debian

This comprehensive guide will help you set up a complete email server with Postfix, Dovecot, and Outlook integration.

## Prerequisites

- Debian server with root/sudo access
- Static public IP address
- Port forwarding configured on your router (25, 587, 993, 995)
- DNS MX records pointing to your mail server
- Domain names configured with proper DNS records

## Example Configuration

This guide uses these example domains and emails:

- **Domains**: example.com, testdomain.net, mywebsite.org
- **Email addresses**: admin@example.com, contact@testdomain.net, info@mywebsite.org
- **Mail server hostname**: mail.example.com

Replace these with your actual domains and email addresses.

## Step 1: System Preparation

```bash
# Update system
sudo apt update
sudo apt upgrade -y

# Install required packages
sudo apt install postfix mailutils dovecot-core dovecot-imapd dovecot-pop3d sasl2-bin libsasl2-modules -y
```

## Step 2: Configure Postfix

During Postfix installation, choose:

- **General type of mail configuration**: `Internet Site`
- **System mail name**: `mail.example.com` (replace with your mail server hostname)

### Edit Postfix Main Configuration

```bash
sudo nano /etc/postfix/main.cf
```

Replace the entire content with:

```conf
# Basic Settings
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain

# Network Settings
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost

# Virtual Domains and Users
virtual_mailbox_domains = example.com, testdomain.net, mywebsite.org
virtual_mailbox_base = /var/mail/vhosts
virtual_mailbox_maps = hash:/etc/postfix/vmaps
virtual_minimum_uid = 100
virtual_uid_maps = static:5000
virtual_gid_maps = static:5000

# Security Settings
smtpd_tls_cert_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file = /etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls = yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# SMTP Authentication
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $myhostname
smtpd_recipient_restrictions = permit_sasl_authenticated, permit_mynetworks, reject_unauth_destination

# Message Size Limits
message_size_limit = 10485760
mailbox_size_limit = 1073741824

# SendGrid SMTP Relay Configuration (Optional - for ISPs blocking port 25)
# relayhost = [smtp.sendgrid.net]:587
# smtp_sasl_auth_enable = yes
# smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# smtp_sasl_security_options = noanonymous
# smtp_tls_security_level = encrypt
```

## Step 3: Create Virtual Mailbox Structure

```bash
# Create virtual mailbox directories
sudo mkdir -p /var/mail/vhosts/example.com
sudo mkdir -p /var/mail/vhosts/testdomain.net
sudo mkdir -p /var/mail/vhosts/mywebsite.org

# Create virtual group and user
sudo groupadd -g 5000 vmail
sudo useradd -r -u 5000 -g 5000 -d /var/mail/vhosts -s /sbin/nologin vmail
sudo chown -R vmail:vmail /var/mail/vhosts
```

## Step 4: Configure Virtual Mailbox Maps

```bash
sudo nano /etc/postfix/vmaps
```

Add the following content:

```
admin@example.com example.com/admin/
contact@testdomain.net testdomain.net/contact/
info@mywebsite.org mywebsite.org/info/
```

Create the hash database:

```bash
sudo postmap /etc/postfix/vmaps
```

## Step 5: Create Mailbox Directories

```bash
# Create mailbox directories for each user
sudo mkdir -p /var/mail/vhosts/example.com/admin
sudo mkdir -p /var/mail/vhosts/testdomain.net/contact
sudo mkdir -p /var/mail/vhosts/mywebsite.org/info

# Set proper permissions
sudo chown -R vmail:vmail /var/mail/vhosts
sudo chmod -R 700 /var/mail/vhosts
```

## Step 6: Configure Email Authentication

Create password file for email accounts:

```bash
sudo nano /etc/postfix/virtual_passwords
```

Add your email accounts with passwords:

```
admin@example.com:your_secure_password_here
contact@testdomain.net:your_secure_password_here
info@mywebsite.org:your_secure_password_here
```

Create the hash database:

```bash
sudo postmap /etc/postfix/virtual_passwords
```

## Step 7: Configure Dovecot

### Edit Dovecot Mail Configuration

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Add/modify these lines:

```conf
mail_location = maildir:/var/mail/vhosts/%d/%n
mail_privileged_group = vmail
mail_access_groups = vmail
```

### Edit Dovecot Authentication

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Replace the content with:

```conf
disable_plaintext_auth = no
auth_mechanisms = plain login

passdb {
  driver = passwd-file
  args = scheme=SHA512-CRYPT username_format=%u /etc/postfix/virtual_passwords
}

userdb {
  driver = static
  args = uid=vmail gid=vmail home=/var/mail/vhosts/%d/%n
}
```

### Configure Dovecot SSL

```bash
sudo nano /etc/dovecot/conf.d/10-ssl.conf
```

Add/modify:

```conf
ssl = yes
ssl_cert = </etc/ssl/certs/ssl-cert-snakeoil.pem
ssl_key = </etc/ssl/private/ssl-cert-snakeoil.key
```

### Configure Dovecot LMTP

```bash
sudo nano /etc/dovecot/conf.d/20-lmtp.conf
```

Add:

```conf
protocol lmtp {
  mail_plugins = $mail_plugins sieve
}
```

## Step 8: Configure Postfix Master Configuration

```bash
sudo nano /etc/postfix/master.cf
```

Add these lines at the end:

```
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

## Step 9: Configure SASL Authentication

```bash
sudo mkdir -p /etc/postfix/sasl
sudo nano /etc/postfix/sasl/smtpd.conf
```

Add:

```
pwcheck_method: saslauthd
mech_list: plain login
```

Configure SASL:

```bash
sudo nano /etc/default/saslauthd
```

Set these values:

```
START=yes
MECHANISMS=pam
OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"
```

## Step 10: Start and Enable Services

```bash
# Start and enable Postfix
sudo systemctl start postfix
sudo systemctl enable postfix

# Start and enable Dovecot
sudo systemctl start dovecot
sudo systemctl enable dovecot

# Start and enable SASL
sudo systemctl start saslauthd
sudo systemctl enable saslauthd

# Create the SASL socket directory for Postfix
sudo mkdir -p /var/spool/postfix/var/run/saslauthd
sudo chown postfix:sasl /var/spool/postfix/var/run/saslauthd
sudo chmod 710 /var/spool/postfix/var/run/saslauthd
```

## Step 11: Firewall Configuration

```bash
sudo ufw allow 25/tcp
sudo ufw allow 587/tcp
sudo ufw allow 993/tcp
sudo ufw allow 995/tcp
```

## Step 12: SSL Certificate (Production)

For production use, get a proper SSL certificate:

```bash
# Install certbot
sudo apt install certbot -y

# Get SSL certificate
sudo certbot certonly --standalone -d mail.example.com

# Update Postfix configuration
sudo nano /etc/postfix/main.cf
# Update these lines:
# smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example.com/fullchain.pem
# smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem

# Update Dovecot configuration
sudo nano /etc/dovecot/conf.d/10-ssl.conf
# Update these lines:
# ssl_cert = </etc/letsencrypt/live/mail.example.com/fullchain.pem
# ssl_key = </etc/letsencrypt/live/mail.example.com/privkey.pem
```

## Step 13: Test Your Email Server

### Test Local Mail Delivery

```bash
echo "Test message" | mail -s "Test Subject" admin@example.com
```

### Check Mail Logs

```bash
sudo tail -f /var/log/mail.log
```

### Test IMAP Connection

```bash
telnet localhost 993
```

### Test SMTP Connection

```bash
telnet localhost 587
```

## Step 14: Configure Outlook

### Connection Information

For each email account, use these settings:

- **Email Address**: admin@example.com, contact@testdomain.net, or info@mywebsite.org
- **IMAP Server**: mail.example.com
- **IMAP Port**: 993 (SSL/TLS)
- **SMTP Server**: mail.example.com
- **SMTP Port**: 587 (STARTTLS)
- **Username**: Full email address (e.g., admin@example.com)
- **Password**: The password you set in virtual_passwords

### Outlook 2016/2019/365 Configuration

1. **Open Outlook**
2. **Go to File → Add Account**
3. **Enter your email address** (e.g., admin@example.com)
4. **Click "Connect"**
5. **Choose "IMAP"** when prompted
6. **Enter the server settings as listed above**

### Manual Configuration (Outlook 2013 and earlier)

1. **Go to File → Account Settings**
2. **Click "New"**
3. **Select "Manual setup or additional server types"**
4. **Choose "POP or IMAP"**
5. **Fill in the server settings as listed above**

## Step 15: Troubleshooting

### Common Issues and Solutions

#### 1. "Cannot connect to server"

```bash
# Check if Dovecot is running
sudo systemctl status dovecot

# Check if port 993 is open
sudo netstat -tlnp | grep 993

# Test IMAP connection
telnet mail.example.com 993
```

#### 2. "Authentication failed"

```bash
# Check virtual passwords
sudo cat /etc/postfix/virtual_passwords

# Check Dovecot logs
sudo tail -f /var/log/mail.log | grep dovecot
```

#### 3. "SSL/TLS error"

```bash
# Test SSL certificate
openssl s_client -connect mail.example.com:993

# Check certificate files
sudo ls -la /etc/ssl/certs/ssl-cert-snakeoil.pem
sudo ls -la /etc/ssl/private/ssl-cert-snakeoil.key
```

#### 4. "Cannot send email"

```bash
# Check SMTP port
telnet mail.example.com 587

# Check Postfix status
sudo systemctl status postfix

# Check mail queue
sudo postqueue -p
```

### Debug Commands

```bash
# Check service status
sudo systemctl status postfix
sudo systemctl status dovecot

# Check mail logs
sudo tail -f /var/log/mail.log

# Test configuration
sudo postfix check
sudo doveconf -c

# Check mailbox directories
ls -la /var/mail/vhosts/
```

## Step 16: Security Recommendations

### 1. Use Strong Passwords

- Use complex passwords for all email accounts
- Consider using a password manager

### 2. Implement Email Security Records

Add these DNS records for your domains:

- **SPF Record**: `v=spf1 mx a ip4:YOUR_SERVER_IP ~all`
- **DKIM Record**: Configure DKIM signing
- **DMARC Record**: `v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com`

### 3. Regular Maintenance

```bash
# Update SSL certificates
sudo certbot renew

# Monitor mail logs
sudo tail -f /var/log/mail.log

# Check disk space
df -h /var/mail/vhosts/
```

### 4. Additional Security

```bash
# Install fail2ban for additional protection
sudo apt install fail2ban -y

# Configure fail2ban for mail services
sudo nano /etc/fail2ban/jail.local
```

## Step 17: Optional: SendGrid Relay Setup

If your ISP blocks port 25, configure SendGrid relay:

### 1. Create SendGrid Credentials

```bash
sudo nano /etc/postfix/sasl_passwd
```

Add:

```
[smtp.sendgrid.net]:587 apikey:YOUR_SENDGRID_API_KEY
```

### 2. Create Hash Database

```bash
sudo postmap /etc/postfix/sasl_passwd
sudo chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo chmod 600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

### 3. Update Postfix Configuration

```bash
sudo nano /etc/postfix/main.cf
```

Uncomment and modify the SendGrid section:

```conf
relayhost = [smtp.sendgrid.net]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt
```

### 4. Reload Postfix

```bash
sudo postfix reload
```

## Verification Checklist

- [ ] Postfix is running and configured
- [ ] Dovecot is running and configured
- [ ] Virtual mailboxes are created
- [ ] Email passwords are set
- [ ] SSL certificates are configured
- [ ] Firewall ports are open
- [ ] DNS records are configured
- [ ] Outlook can connect via IMAP
- [ ] Emails can be sent and received
- [ ] Mail logs are being written

Your email server is now fully configured and ready for production use!
