### **Step 1: Access cPanel**
1. Log in to your hosting account's cPanel dashboard.
2. Locate your **cPanel URL** (e.g., `https://yourdomain.com:2083`) or access it from your hosting provider's portal.

---

### **Step 2: Set Up the Database**
1. **Create a Database:**
   - In cPanel, go to **MySQL Databases**.
   - Create a new database (e.g., `vending_machine`).
   - Note the database name.

2. **Create a Database User:**
   - In the same **MySQL Databases** section, create a new user (e.g., `vending_user`) and set a strong password.
   - Note the username and password.

3. **Grant Privileges:**
   - Assign the user to the database you created.
   - Select **All Privileges** and save changes.

4. **Import Your Schema:**
   - Go to **phpMyAdmin** in cPanel.
   - Select the `vending_machine` database.
   - Use the **Import** tab to upload your `.sql` file (database schema).

---

### **Step 3: Upload Project Files**
1. **Compress Your Project:**
   - On your local machine, compress your project folder into a `.zip` file.

2. **Upload Files:**
   - In cPanel, go to **File Manager**.
   - Navigate to the `public_html` directory (or a subdirectory if you want to deploy under a subdomain).
   - Click **Upload** and select your `.zip` file.

3. **Extract Files:**
   - After uploading, right-click the `.zip` file in **File Manager** and choose **Extract**.
   - Ensure all files are in the appropriate folder (e.g., `public_html` for root deployment).

4. **Adjust File Paths:**
   - Verify the `Database.php` file and update the connection details with your cPanel database credentials:
     ```php
     private $host = "localhost";
     private $db_name = "your_cpanel_username_vending_machine";
     private $username = "your_cpanel_username_vending_user";
     private $password = "your_password";
     ```

---

### **Step 4: Set Permissions**
1. Ensure writable directories (like `uploads/`) have appropriate permissions:
   - Right-click the directory in **File Manager**.
   - Set **Permissions** to `755` or `777` for directories needing write access.

---

### **Step 5: Configure the Application**
1. If your project has a `.htaccess` file, ensure it’s uploaded. A common `.htaccess` for PHP projects includes:
   ```apache
   Options -Indexes
   RewriteEngine On
   RewriteBase /
   RewriteCond %{REQUEST_FILENAME} !-f
   RewriteCond %{REQUEST_FILENAME} !-d
   RewriteRule ^ index.php [QSA,L]
   ```

2. Test the application by visiting your domain or subdomain.

---

### **Step 6: Configure a Domain or Subdomain**
1. **For Main Domain:**
   - Files should be in the `public_html` folder.
   - Access the application via `http://yourdomain.com`.

2. **For Subdomain:**
   - In cPanel, go to **Subdomains**.
   - Create a subdomain (e.g., `vending.yourdomain.com`).
   - Point it to a folder (e.g., `public_html/vending_machine`).

---

### **Step 7: Set Up SSL (Optional but Recommended)**
1. **Enable SSL:**
   - In cPanel, go to **SSL/TLS** or **AutoSSL**.
   - Install an SSL certificate for your domain.

2. **Force HTTPS:**
   - Edit the `.htaccess` file to redirect HTTP to HTTPS:
     ```apache
     RewriteCond %{HTTPS} !=on
     RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
     ```

---

### **Step 8: Test Your Application**
1. Visit your domain or subdomain (e.g., `https://yourdomain.com`).
2. Test all functionalities (CRUD operations, login/logout, transactions, etc.).

---

### **Step 9: Maintain Your Deployment**
1. **Backup Regularly:**
   - Use cPanel’s **Backup** tool to download backups of files and the database.
2. **Monitor Logs:**
   - Check **Error Logs** in cPanel for any issues.
3. **Update PHP Version:**
   - If needed, update your PHP version in cPanel under **Select PHP Version**.
