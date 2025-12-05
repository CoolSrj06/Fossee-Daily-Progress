For setting up Roundcube we are going to move in stages.

- **Postfix** (SMTP + Submission + Maildir delivery)
- **Dovecot** (IMAP + SASL backend)
- **Maildir** (local user mailbox storage)
- **Roundcube** (webmail UI)
- **SQLite** (for Roundcube only)
- **SELinux configuration**
- **Firewalld rules**
- **Host-only IP environment**

# **1. Overview**

| Component              | Role                                               |
| ---------------------- | -------------------------------------------------- |
| **Postfix**            | SMTP server — sends and receives email             |
| **Dovecot**            | IMAP server — users read mail + provides SMTP AUTH |
| **Maildir**            | Storage format for each user’s mailbox             |
| **Roundcube**          | Webmail client                                     |
| **SQLite**             | Storage for Roundcube’s preferences and contacts   |
| **Linux system users** | Email accounts (no MySQL, no virtual users)        |

This setup works entirely offline using **your VM’s host-only adapter IP**, e.g.:

```
192.168.56.xx
```


#  **2. System Requirements**

- AlmaLinux 10 VM
- Host-only network enabled
- Root/sudo access

Update system first:

```bash
sudo dnf update -y
```

---

#  **3. Create Linux Users (Email Accounts)**

Email accounts = Linux system users.

Example:

```bash
sudo useradd alice
sudo passwd alice

sudo useradd bob
sudo passwd bob
```

Repeat for any number of accounts.

---

#  **4. Install Postfix + Dovecot**

```bash
sudo dnf install -y postfix dovecot
```

Enable services:

```bash
sudo systemctl enable --now postfix
sudo systemctl enable --now dovecot
```

---

#  **5. Configure Postfix: Maildir + Submission + SASL**

Edit:

```bash
sudo nano /etc/postfix/main.cf
```

Add/ensure these values exist:

```bash
myhostname = mail.dev.local
myorigin = $myhostname
mydestination = $myhostname, localhost.localdomain, localhost
inet_interfaces = all
inet_protocols = all

home_mailbox = Maildir/

smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth

smtpd_tls_security_level = may
smtpd_recipient_restrictions = permit_sasl_authenticated,reject
```

---

#  **6. Configure Postfix Submission Service (Port 587)**

This is required for sending mail through Roundcube.

Edit:

```bash
sudo nano /etc/postfix/master.cf
```

Find the submission section and replace with:

```
submission inet n       -       n       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=may
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o chroot=no
```

Restart Postfix completely:

```bash
sudo systemctl stop postfix
sudo systemctl start postfix
```

Verify port 587:

```bash
ss -lntp | grep 587
```

---

#  **7. Configure Dovecot (IMAP + SASL)**

Edit:

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Set:

```
mail_location = maildir:~/Maildir
```

Configure system-user authentication:

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Ensure:

```
disable_plaintext_auth = no
auth_mechanisms = plain login
```

Enable PAM (system users):

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Uncomment:

```
!include auth-system.conf.ext
```

Enable the Dovecot socket for Postfix SASL:

```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

Find:

```
unix_listener /var/spool/postfix/private/auth {
```

Ensure:

```
  mode = 0660
  user = postfix
  group = postfix
```

Restart Dovecot:

```bash
sudo systemctl restart dovecot
```

---

#  **8. Test Mail Delivery (Optional)**

Send a test email as a local user:

```bash
sudo -u alice bash -c 'echo "Test" | mail -s "Hello Bob" bob'
```

Check Bob’s mailbox:

```bash
sudo -u bob ls -l ~/Maildir/new
```

---

#  **9. Install Roundcube + Dependencies**

Install Apache + PHP:

```bash
sudo dnf install -y httpd php php-fpm php-mbstring php-intl php-xml \
php-json php-pdo php-pdo_sqlite php-sqlite3 php-gd php-zip php-curl
```

Start services:

```bash
sudo systemctl enable --now httpd php-fpm
```

Allow Apache network connections:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

---

# **10. Download & Install Roundcube**

```bash
cd /usr/local/src
sudo curl -LO https://github.com/roundcube/roundcubemail/releases/download/1.6.7/roundcubemail-1.6.7-complete.tar.gz
sudo tar xzf roundcubemail-1.6.7-complete.tar.gz
sudo mv roundcubemail-1.6.7 /var/www/roundcube
sudo chown -R apache:apache /var/www/roundcube
```

---

#  **11. Create SQLite Database for Roundcube**

Install SQLite CLI:

```bash
sudo dnf install -y sqlite
```

Initialize DB:

```bash
cd /var/www/roundcube
sudo -u apache mkdir db
sudo -u apache sqlite3 db/sqlite.db < SQL/sqlite.initial.sql
sudo chmod 660 db/sqlite.db
sudo chown apache:apache db/sqlite.db
```

---

#  **12. Configure Roundcube**

Create configuration:

```bash
sudo -u apache cp /var/www/roundcube/config/config.inc.php.sample \
/var/www/roundcube/config/config.inc.php
```

Edit:

```bash
sudo nano /var/www/roundcube/config/config.inc.php
```

Set:

```php
$config['db_dsnw'] = 'sqlite:////var/www/roundcube/db/sqlite.db';

$config['default_host'] = '127.0.0.1';
$config['default_port'] = 143;

$config['smtp_host'] = '127.0.0.1';
$config['smtp_port'] = 587;
$config['smtp_user'] = '%u';
$config['smtp_pass'] = '%p';

$config['des_key'] = 'generate_a_random_32_char_key';
```

---

#  **13. Configure Apache for Roundcube**

Create:

```bash
sudo nano /etc/httpd/conf.d/roundcube.conf
```

Add:

```apache
Alias /roundcube /var/www/roundcube

<Directory /var/www/roundcube/>
    Options -Indexes
    AllowOverride All
    Require all granted
</Directory>
```

Restart Apache:

```bash
sudo systemctl restart httpd
```

---

# **14. Firewall Rules**

Open HTTP:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

---

#  **15. Fix SELinux for Roundcube (CRITICAL)**

Fix file labels:

```bash
sudo restorecon -RFv /var/www/roundcube
```

Give write access:

```bash
sudo chcon -Rt httpd_sys_rw_content_t /var/www/roundcube/temp
sudo chcon -Rt httpd_sys_rw_content_t /var/www/roundcube/logs
sudo chcon -Rt httpd_sys_rw_content_t /var/www/roundcube/db
```

Allow Apache network:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

If enforcing mode blocked login, turn it on again:

```bash
sudo setenforce 1
```

---

#  **16. Access Roundcube**

From your browser:

```
http://192.168.56.11/roundcube
```

Login using Linux system accounts:

```
username: alice
password: test@123
```

---

# **17. Send Test Email from Roundcube**

Send to:

```
bob@localhost
```

On server:

```bash
sudo -u bob ls -l ~/Maildir/new
```

Message should be there.

---

