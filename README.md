<img width="1021" alt="Screenshot 2025-01-23 at 4 01 49 PM" src="https://github.com/user-attachments/assets/6be0c1ea-3a6b-40d2-ac86-5e83623c583e" /># wordpress-ec2-nginx


# Setting up WordPress with Nginx on Ubuntu

This documentation explains how to set up WordPress with Nginx as the web server on an Ubuntu server, including enabling HTTPS with SSL certificates. Follow the steps below for a successful installation and configuration.

---

## **Prerequisites**
1. An Ubuntu server (e.g., EC2 instance).
2. A domain name or public IP address.
3. SSH access to the server.
4. Nginx installed (can be installed during setup).
5. PHP and MySQL installed on the server.

---

## **Step 1: Update the System**
Start by updating the server’s package index:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## **Step 2: Install Required Software**
Install Nginx, PHP, MySQL, and additional PHP extensions required for WordPress.

### Install Nginx
```bash
sudo apt install nginx -y
```

### Install PHP and Extensions
```bash
sudo apt install php8.3 php8.3-fpm php8.3-mysql -y
```

### As we have RDS MySQL i don't have install Mysql on my EC2 just a Client is enough for connection

### Install MySQL Client
```bash
sudo apt install mysql-client-core-8.8
```

### Connect to RDS Mysql 
```bash
mysql -h <rds-database-endpoint> -P <port-no> -u <user> -p <password>
```

---

Create a database and user for WordPress:

```sql
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## **Step 4: Download and Set Up WordPress**
### Download WordPress
Navigate to the web root directory and download WordPress:

```bash
cd /var/www/html
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo mv wordpress/* .
sudo rm -rf wordpress latest.tar.gz
```

### Making wordpress configuration here i need to create a file name 'wp-config.php'
Here i need to mention Database realted Data
```
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'admin' );

/** Database password */
define( 'DB_PASSWORD', 'admin123' );

/** Database hostname */
define( 'DB_HOST', 'database-1.cxugm6u0ctx0.ap-southeast-2.rds.amazonaws.com' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

```

```
define('AUTH_KEY',         '@VZ<pXEL?vb-kiz(Zfp_R9f9|.+T-O/P$Z9|T-q%~|KX@,/(RZk00K{ybHA=6nT6');
define('SECURE_AUTH_KEY',  '12Ip[Ts<IA>Vc+R#_X>i85OjMMRtks-o^E2(,$P[Q=f~Zt:@FrW1r$,` vqs|%@|');
define('LOGGED_IN_KEY',    'o*_:obJ!+wtc8&]QhK}-xEVv+eVD!hFbBzkxKn@}(gK{-{d|l-?9b)8+)tfx8zjl');
define('NONCE_KEY',        'Ue<fX0Z71vg7Y&F+~CqM-G%N~ozMe%?qrp-@|tTVh??zJ4:~Sm,VhTKBE0C7DY(?');
define('AUTH_SALT',        'v.Z^1,QR66F-CDW=t<daxxk-;|M3cC{XzF`rn#l[U]f-fboHZYY/c8nvYU(uM`a]');
define('SECURE_AUTH_SALT', 'Cz$[Mq>)Hc=BSo,&Q%;,r}Eu7!:>nj4N91WeIx|7jp=fc+S64lMXCNj|h&a9Q5[D');
define('LOGGED_IN_SALT',   'Y{[of`B<!<<Za+|YtiMJkd33@a}+I%-0u}EPmAx?hW~$_(( u iuIJ2UQUJIv7(Q');
define('NONCE_SALT',       'g<!-Nhqy2E{X{87!|x{Amg3v:Z%e8d(z3l9x|g-T uB62u4xuw?uyjS_>1Yl]~Kw');
```

<img width="1032" alt="Screenshot 2025-01-23 at 4 02 21 PM" src="https://github.com/user-attachments/assets/7b825b94-fc84-40c1-aa08-d44d4edab19e" />
<img width="1019" alt="Screenshot 2025-01-23 at 4 02 28 PM" src="https://github.com/user-attachments/assets/9c7be4f5-faee-475b-9d45-0b81ec8607fe" />




### Set Permissions
Set appropriate ownership and permissions for WordPress files:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

---

## **Step 5: Configure Nginx**
Create a new configuration file for WordPress in the `/etc/nginx/conf.d/` directory:

```bash
sudo nano /etc/nginx/conf.d/wordpress.conf
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com; # Replace with your domain or public IP
    root /var/www/html;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Replace `your-domain.com` with your domain name or public IP address.

### Test and Reload Nginx
Test the Nginx configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

---

## **Step 6: Complete the WordPress Installation**
1. Access the WordPress setup wizard in your browser by navigating to:
   ```
   http://your-domain.com/wp-admin
   ```
2. Follow the on-screen instructions to:
   - Select your language.
   - Enter the database details created earlier:
     - Database Name: `wordpress`
     - Username: `wordpressuser`
     - Password: `strongpassword`
3. Complete the installation and set up your admin account.

---

## **Step 7: Enable HTTPS with SSL**
### Install Certbot
Certbot helps obtain and configure SSL certificates from Let's Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Obtain and Configure SSL
Run the following command to obtain an SSL certificate:

```bash
sudo certbot --nginx -d your-domain.com
```

Follow the prompts to complete the setup. Certbot will automatically configure Nginx to redirect HTTP traffic to HTTPS.

### Test Certificate Renewal
To test the automatic renewal of SSL certificates:

```bash
sudo certbot renew --dry-run
```

---

## **Step 8: Verify and Update WordPress Settings**
Update the WordPress site URL to use HTTPS:
1. Log in to WordPress Admin Panel.
2. Navigate to **Settings > General**.
3. Update the **WordPress Address (URL)** and **Site Address (URL)** to `https://your-domain.com`.

<img width="1470" alt="Screenshot 2025-01-23 at 4 07 59 PM" src="https://github.com/user-attachments/assets/26e06689-a103-4d5c-b56c-d8e2075d35be" />


Alternatively, update the database directly:

```sql
UPDATE wp_options SET option_value = 'https://your-domain.com' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = 'https://your-domain.com' WHERE option_name = 'home';
```

---

## **Step 9: Troubleshooting**
1. **PHP-FPM Errors**:
   - Ensure PHP-FPM is running:
     ```bash
     sudo systemctl start php8.3-fpm
     sudo systemctl enable php8.3-fpm
     ```
2. **Directory Permissions**:
   - Ensure the correct ownership and permissions for `/var/www/html`.
3. **Nginx Configuration Errors**:
   - Check the Nginx error log for issues:
     ```bash
     sudo tail -f /var/log/nginx/error.log
     ```

---

## **Conclusion**
Your WordPress site should now be running with Nginx and secured with HTTPS. You can access it via `https://your-domain.com`.

