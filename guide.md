### Step 1: Setting Up Your Environment
1. **Install PHP and MySQL:**
   Ensure PHP (preferably version 7.4 or later) and MySQL are installed on your system.

2. **Set Up a Local Development Environment:**
   Use tools like XAMPP, WAMP, or MAMP to run PHP and MySQL locally.

3. **Create a Project Directory:**
   Create a directory named `VendingMachine`.

---

### Step 2: Database Design and Setup
1. **Create a Database:**
   Log into MySQL and create a database named `vending_machine`:
   ```sql
   CREATE DATABASE vending_machine;
   ```

2. **Define Tables:**
   Create the following tables:

   - `products`:
     ```sql
     CREATE TABLE products (
         id INT AUTO_INCREMENT PRIMARY KEY,
         name VARCHAR(255) NOT NULL,
         price DECIMAL(10,2) NOT NULL,
         quantity_available INT NOT NULL
     );
     ```

   - `users`:
     ```sql
     CREATE TABLE users (
         id INT AUTO_INCREMENT PRIMARY KEY,
         username VARCHAR(255) UNIQUE NOT NULL,
         password VARCHAR(255) NOT NULL,
         role ENUM('admin', 'user') NOT NULL
     );
     ```

   - `transactions`:
     ```sql
     CREATE TABLE transactions (
         id INT AUTO_INCREMENT PRIMARY KEY,
         product_id INT NOT NULL,
         user_id INT NOT NULL,
         quantity INT NOT NULL,
         total_price DECIMAL(10,2) NOT NULL,
         transaction_date DATETIME DEFAULT CURRENT_TIMESTAMP,
         FOREIGN KEY (product_id) REFERENCES products(id),
         FOREIGN KEY (user_id) REFERENCES users(id)
     );
     ```

---

### Step 3: Database Connection in PHP
1. **Create a Connection Class:**
   In your project, create a file `Database.php`:
   ```php
   <?php
   class Database {
       private $host = "localhost";
       private $db_name = "vending_machine";
       private $username = "root"; // Replace with your MySQL username
       private $password = ""; // Replace with your MySQL password
       public $conn;

       public function getConnection() {
           $this->conn = null;
           try {
               $this->conn = new PDO("mysql:host=" . $this->host . ";dbname=" . $this->db_name, $this->username, $this->password);
               $this->conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
           } catch (PDOException $exception) {
               echo "Connection error: " . $exception->getMessage();
           }
           return $this->conn;
       }
   }
   ?>
   ```

2. **Test the Connection:**
   Create a `test.php` file to verify:
   ```php
   <?php
   include 'Database.php';

   $database = new Database();
   $db = $database->getConnection();

   if ($db) {
       echo "Database connected successfully!";
   }
   ?>
   ```

---

### Step 4: Implement User Authentication
1. **Register Users:**
   Create a script `register.php` for user registration:
   ```php
   <?php
   include 'Database.php';

   $database = new Database();
   $db = $database->getConnection();

   if ($_POST) {
       $username = $_POST['username'];
       $password = password_hash($_POST['password'], PASSWORD_BCRYPT);
       $role = $_POST['role'];

       $query = "INSERT INTO users (username, password, role) VALUES (:username, :password, :role)";
       $stmt = $db->prepare($query);
       $stmt->bindParam(':username', $username);
       $stmt->bindParam(':password', $password);
       $stmt->bindParam(':role', $role);

       if ($stmt->execute()) {
           echo "User registered successfully.";
       } else {
           echo "Registration failed.";
       }
   }
   ?>
   ```

2. **Login Users:**
   Create a `login.php` file to handle authentication:
   ```php
   <?php
   session_start();
   include 'Database.php';

   $database = new Database();
   $db = $database->getConnection();

   if ($_POST) {
       $username = $_POST['username'];
       $password = $_POST['password'];

       $query = "SELECT * FROM users WHERE username = :username";
       $stmt = $db->prepare($query);
       $stmt->bindParam(':username', $username);
       $stmt->execute();

       $user = $stmt->fetch(PDO::FETCH_ASSOC);

       if ($user && password_verify($password, $user['password'])) {
           $_SESSION['user_id'] = $user['id'];
           $_SESSION['role'] = $user['role'];
           echo "Login successful.";
       } else {
           echo "Invalid credentials.";
       }
   }
   ?>
   ```

---

### Step 5: Create Basic Controllers
1. **Products Controller:**
   Create a `ProductsController.php` file to manage products:
   ```php
   <?php
   include 'Database.php';

   class ProductsController {
       private $db;

       public function __construct() {
           $database = new Database();
           $this->db = $database->getConnection();
       }

       public function getProducts() {
           $query = "SELECT * FROM products";
           $stmt = $this->db->prepare($query);
           $stmt->execute();
           return $stmt->fetchAll(PDO::FETCH_ASSOC);
       }
   }
   ?>
   ```

2. **Test the Controller:**
   Create a `test_products.php` file:
   ```php
   <?php
   include 'ProductsController.php';

   $controller = new ProductsController();
   $products = $controller->getProducts();

   print_r($products);
   ?>
   ```
