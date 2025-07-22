# Postfix Email Server Setup Guide for Debian

This guide will help you set up a Postfix email server on your Debian system to handle email for your domains:

- psychebot.pro
- syedqutubuddin.in
- website14.com

## Prerequisites

- Debian server with root/sudo access
- Static public IP address (mail.psychebot.pro)
- Port 25 (SMTP) forwarded from router to your server
- DNS MX records pointing to mail.psychebot.pro for all domains

## Step 1: Update System

```bash
sudo apt update
sudo apt upgrade -y
```

## Step 2: Install Postfix

```bash
sudo apt install postfix -y
```

During installation, you'll be prompted to configure Postfix. Choose:

- **General type of mail configuration**: `Internet Site`
- **System mail name**: `mail.psychebot.pro`

## Step 3: Install Additional Dependencies

```bash
sudo apt install mailutils dovecot-core dovecot-imapd dovecot-pop3d sasl2-bin libsasl2-modules -y
```

## Step 4: Configure Postfix Main Configuration

Edit the main Postfix configuration file:

```bash
sudo nano /etc/postfix/main.cf
```

Replace or add the following configuration:

```conf
# Basic Settings
myhostname = mail.psychebot.pro
mydomain = psychebot.pro
myorigin = $mydomain

# Network Settings
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain, psychebot.pro, syedqutubuddin.in, website14.com

# Virtual Domains and Users
virtual_mailbox_domains = psychebot.pro, syedqutubuddin.in, website14.com
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
```

## Step 5: Create Virtual Mailbox Structure

```bash
# Create virtual mailbox directories
sudo mkdir -p /var/mail/vhosts/psychebot.pro
sudo mkdir -p /var/mail/vhosts/syedqutubuddin.in
sudo mkdir -p /var/mail/vhosts/website14.com

# Create virtual group and user
sudo groupadd -g 5000 vmail
sudo useradd -r -u 5000 -g 5000 -d /var/mail/vhosts -s /sbin/nologin vmail
sudo chown -R vmail:vmail /var/mail/vhosts
```

## Step 6: Configure Virtual Mailbox Maps

Create the virtual mailbox mapping file:

```bash
sudo nano /etc/postfix/vmaps
```

Add the following content:

```
contact@psychebot.pro psychebot.pro/contact/
contact@syedqutubuddin.in syedqutubuddin.in/contact/
contact@website14.com website14.com/contact/
```

Create the hash database:

```bash
sudo postmap /etc/postfix/vmaps
```

## Step 7: Create Mailbox Directories

```bash
# Create mailbox directories for each user
sudo mkdir -p /var/mail/vhosts/psychebot.pro/contact
sudo mkdir -p /var/mail/vhosts/syedqutubuddin.in/contact
sudo mkdir -p /var/mail/vhosts/website14.com/contact

# Set proper permissions
sudo chown -R vmail:vmail /var/mail/vhosts
sudo chmod -R 700 /var/mail/vhosts
```

## Step 8: Configure Dovecot for IMAP/POP3

Edit Dovecot configuration:

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Add/modify these lines:

```conf
mail_location = maildir:/var/mail/vhosts/%d/%n
mail_privileged_group = vmail
mail_access_groups = vmail
```

Edit the authentication configuration:

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Add/modify:

```conf
disable_plaintext_auth = no
auth_mechanisms = plain login
passdb {
  driver = pam
}
userdb {
  driver = static
  args = uid=vmail gid=vmail home=/var/mail/vhosts/%d/%n
}
```

## Step 9: Configure Postfix to Use Dovecot for Authentication

Edit the Postfix master configuration:

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

## Step 10: Configure SASL Authentication

Create SASL configuration:

```bash
sudo mkdir -p /etc/postfix/sasl
sudo nano /etc/postfix/sasl/smtpd.conf
```

Add:

```
pwcheck_method: saslauthd
mech_list: plain login
```

Create the SASL configuration directory and file:

```bash
sudo mkdir -p /etc/postfix/sasl
sudo nano /etc/postfix/sasl/smtpd.conf
```

Add the content above, then configure SASL to use PAM:

```bash
sudo nano /etc/default/saslauthd
```

Set these values:

```
START=yes
MECHANISMS=pam
OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"
```

## Step 11: Start and Enable Services

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

## Step 12: Test Configuration

Check Postfix configuration:

```bash
sudo postfix check
sudo postfix reload
```

Test SMTP connection:

```bash
telnet localhost 25
```

## Step 13: Firewall Configuration

Ensure ports are open:

```bash
sudo ufw allow 25/tcp
sudo ufw allow 587/tcp
sudo ufw allow 993/tcp
sudo ufw allow 995/tcp
```

## Step 14: Create Email Users

To add email users, edit the vmaps file and recreate the hash:

```bash
sudo nano /etc/postfix/vmaps
# Add new users in format: email@domain.com domain.com/username/
sudo postmap /etc/postfix/vmaps
sudo postfix reload
```

## Step 15: SSL Certificate (Optional but Recommended)

For production use, replace the self-signed certificate:

```bash
# Install certbot for Let's Encrypt
sudo apt install certbot -y

# Get SSL certificate
sudo certbot certonly --standalone -d mail.psychebot.pro

# Update Postfix configuration to use the new certificate
sudo nano /etc/postfix/main.cf
# Update these lines:
# smtpd_tls_cert_file = /etc/letsencrypt/live/mail.psychebot.pro/fullchain.pem
# smtpd_tls_key_file = /etc/letsencrypt/live/mail.psychebot.pro/privkey.pem
```

## Verification Steps

1. **Check service status:**

   ```bash
   sudo systemctl status postfix
   sudo systemctl status dovecot
   ```

2. **Test local mail delivery:**

   ```bash
   echo "Test message" | mail -s "Test Subject" contact@psychebot.pro
   ```

3. **Check mail logs:**
   ```bash
   sudo tail -f /var/log/mail.log
   ```

## Troubleshooting

- Check logs: `/var/log/mail.log`
- Test configuration: `sudo postfix check`
- Verify DNS: `nslookup -type=mx psychebot.pro`
- Test SMTP: `telnet mail.psychebot.pro 25`

## Important Notes

- Ensure your router forwards port 25 to your server
- DNS MX records should point to mail.psychebot.pro
- Consider implementing SPF, DKIM, and DMARC records
- Monitor mail logs for delivery issues
- Regularly update SSL certificates

Your Postfix server is now configured to handle email for all three domains with the specified email addresses.

## Setting Up Outlook to Connect to Your Email Server

### Prerequisites

- Your Postfix/Dovecot server is running and accessible
- Port 993 (IMAP SSL) and 587 (SMTP) are open and forwarded
- SSL certificate is properly configured

### Step 1: Gather Connection Information

For each email account, you'll need:

- **Email Address**: contact@psychebot.pro, contact@syedqutubuddin.in, or contact@website14.com
- **IMAP Server**: mail.psychebot.pro
- **IMAP Port**: 993 (SSL/TLS)
- **SMTP Server**: mail.psychebot.pro
- **SMTP Port**: 587 (STARTTLS)
- **Username**: Your email address (full email address)
- **Password**: You'll need to set this up (see authentication setup below)

### Step 2: Set Up Email Authentication

Since you're using virtual mailboxes, you need to set up authentication. Create a password file:

```bash
sudo nano /etc/postfix/virtual_passwords
```

Add your email accounts with passwords:

```
contact@psychebot.pro:your_password_here
contact@syedqutubuddin.in:your_password_here
contact@website14.com:your_password_here
```

Create the hash database:

```bash
sudo postmap /etc/postfix/virtual_passwords
```

### Step 3: Configure Postfix for Authentication

Edit Postfix main configuration:

```bash
sudo nano /etc/postfix/main.cf
```

Add these lines to the authentication section:

```conf
# Virtual Authentication
virtual_transport = lmtp:unix:private/dovecot-lmtp
virtual_mailbox_domains = psychebot.pro, syedqutubuddin.in, website14.com
virtual_mailbox_maps = hash:/etc/postfix/vmaps
virtual_alias_maps = hash:/etc/postfix/vmaps
virtual_mailbox_base = /var/mail/vhosts
virtual_minimum_uid = 100
virtual_uid_maps = static:5000
virtual_gid_maps = static:5000
```

### Step 4: Configure Dovecot for Authentication

Edit Dovecot authentication:

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

### Step 5: Configure Dovecot for LMTP

Edit Dovecot LMTP configuration:

```bash
sudo nano /etc/dovecot/conf.d/20-lmtp.conf
```

Add:

```conf
protocol lmtp {
  mail_plugins = $mail_plugins sieve
}
```

### Step 6: Restart Services

```bash
sudo systemctl restart postfix
sudo systemctl restart dovecot
```

### Step 7: Outlook Configuration

#### For Outlook 2016/2019/365:

1. **Open Outlook**
2. **Go to File → Add Account**
3. **Enter your email address** (e.g., contact@psychebot.pro)
4. **Click "Connect"**
5. **Choose "IMAP"** when prompted
6. **Enter the following settings:**

**Incoming Mail (IMAP):**

- Server: mail.psychebot.pro
- Port: 993
- Encryption: SSL/TLS
- Username: contact@psychebot.pro (full email address)
- Password: (the password you set in virtual_passwords)

**Outgoing Mail (SMTP):**

- Server: mail.psychebot.pro
- Port: 587
- Encryption: STARTTLS
- Username: contact@psychebot.pro (full email address)
- Password: (same password)

#### For Outlook 2013 and earlier:

1. **Go to File → Account Settings**
2. **Click "New"**
3. **Select "Manual setup or additional server types"**
4. **Choose "POP or IMAP"**
5. **Fill in the settings as above**

### Step 8: Test Connection

1. **Send a test email** to yourself
2. **Check if you receive it**
3. **Check mail logs** if there are issues:
   ```bash
   sudo tail -f /var/log/mail.log
   ```

### Troubleshooting Outlook Connection Issues

#### Common Issues:

1. **"Cannot connect to server"**

   - Check if port 993 is open: `telnet mail.psychebot.pro 993`
   - Verify firewall settings
   - Check if Dovecot is running: `sudo systemctl status dovecot`

2. **"Authentication failed"**

   - Verify password in `/etc/postfix/virtual_passwords`
   - Check Dovecot logs: `sudo tail -f /var/log/mail.log`
   - Ensure username is full email address

3. **"SSL/TLS error"**

   - Check SSL certificate: `openssl s_client -connect mail.psychebot.pro:993`
   - Verify certificate path in Dovecot config

4. **"Cannot send email"**
   - Check SMTP port 587: `telnet mail.psychebot.pro 587`
   - Verify SMTP authentication settings
   - Check Postfix logs: `sudo tail -f /var/log/mail.log`

#### Debug Commands:

```bash
# Test IMAP connection
telnet mail.psychebot.pro 993

# Test SMTP connection
telnet mail.psychebot.pro 587

# Check Dovecot status
sudo systemctl status dovecot

# Check Postfix status
sudo systemctl status postfix

# View mail logs
sudo tail -f /var/log/mail.log
```

### Security Notes:

- Use strong passwords for email accounts
- Consider implementing SPF, DKIM, and DMARC records
- Regularly update SSL certificates
- Monitor mail logs for suspicious activity
- Consider using fail2ban for additional security

Your Outlook should now be able to connect to your Postfix/Dovecot email server!
