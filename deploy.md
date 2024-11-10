### **Step 1: Choose a Hosting Platform**
Select a hosting provider that supports PHP and MySQL. Popular choices include:
- **Shared Hosting:** e.g., Bluehost, HostGator, SiteGround.
- **Cloud Hosting:** e.g., AWS, Google Cloud, DigitalOcean.
- **Dedicated Servers or VPS:** For more control over your environment.

For this guide, let's assume you're deploying to a Linux VPS or shared hosting.

---

### **Step 2: Set Up Your Hosting Environment**
#### **For Shared Hosting:**
1. Access your hosting control panel (e.g., cPanel).
2. Use the **File Manager** to upload your project files.
3. Set up a MySQL database using the control panel.

#### **For VPS/Cloud Hosting:**
1. SSH into your server:
   ```bash
   ssh your_user@your_server_ip
   ```
2. Install PHP and MySQL:
   ```bash
   sudo apt update
   sudo apt install apache2 php libapache2-mod-php mysql-server
   ```
3. Secure MySQL:
   ```bash
   sudo mysql_secure_installation
   ```
4. Create a directory for your project:
   ```bash
   sudo mkdir /var/www/vending_machine
   sudo chown -R $USER:$USER /var/www/vending_machine
   ```

---

### **Step 3: Upload Your Project Files**
#### **For Shared Hosting:**
1. Compress your project folder into a `.zip` file.
2. Use the hosting provider's **File Manager** or FTP to upload and extract the files into the public_html directory.

#### **For VPS:**
1. Use SCP (from your local machine):
   ```bash
   scp -r /path/to/your/project your_user@your_server_ip:/var/www/vending_machine
   ```
2. Ensure proper permissions:
   ```bash
   sudo chmod -R 755 /var/www/vending_machine
   ```

---

### **Step 4: Configure Your Database**
1. **Create a Database:**
   Log into MySQL and create your database:
   ```bash
   mysql -u root -p
   CREATE DATABASE vending_machine;
   CREATE USER 'vending_user'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON vending_machine.* TO 'vending_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

2. **Import Your Database Schema:**
   If you have an `.sql` file:
   ```bash
   mysql -u vending_user -p vending_machine < /path/to/your/schema.sql
   ```

---

### **Step 5: Configure the Project**
1. **Update Database Connection:**
   Edit `Database.php` to reflect your server credentials:
   ```php
   private $host = "localhost";
   private $db_name = "vending_machine";
   private $username = "vending_user";
   private $password = "your_password";
   ```

2. **Set File Paths:**
   Ensure any file paths in your project match the server structure.

3. **Set Proper Permissions:**
   Make `uploads/` or any writable directories accessible:
   ```bash
   sudo chmod -R 777 /var/www/vending_machine/uploads
   ```

---

### **Step 6: Configure Apache**
1. **Create a Virtual Host:**
   Edit Apache’s configuration file:
   ```bash
   sudo nano /etc/apache2/sites-available/vending_machine.conf
   ```
   Add:
   ```apache
   <VirtualHost *:80>
       ServerAdmin admin@your_domain.com
       DocumentRoot /var/www/vending_machine
       ServerName your_domain.com
       ServerAlias www.your_domain.com
       <Directory /var/www/vending_machine>
           Options Indexes FollowSymLinks MultiViews
           AllowOverride All
           Require all granted
       </Directory>
       ErrorLog ${APACHE_LOG_DIR}/vending_machine_error.log
       CustomLog ${APACHE_LOG_DIR}/vending_machine_access.log combined
   </VirtualHost>
   ```
2. **Enable the Site:**
   ```bash
   sudo a2ensite vending_machine.conf
   sudo systemctl reload apache2
   ```

---

### **Step 7: Point Your Domain (Optional)**
If using a domain name:
1. Update your domain's DNS settings to point to your server's IP address.
2. Wait for propagation (can take up to 24 hours).

---

### **Step 8: Secure Your Server**
1. **Install SSL (HTTPS):**
   Use **Let’s Encrypt** for free SSL certificates:
   ```bash
   sudo apt install certbot python3-certbot-apache
   sudo certbot --apache
   ```
2. Follow the prompts to configure SSL.

---

### **Step 9: Test Your Application**
1. Visit your server's IP address or domain:
   ```
   http://your_domain.com
   ```
2. Test all features (CRUD, user authentication, etc.).

---

### **Step 10: Maintenance**
1. Monitor logs for issues:
   ```bash
   tail -f /var/log/apache2/error.log
   ```
2. Regularly back up your database and files.
