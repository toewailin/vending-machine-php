### **Step 16: Create a RESTful API**
1. **Set Up Directory Structure:**
   - Organize your API-related files in a dedicated folder (e.g., `api/`).

2. **Set Up Routing Logic:**
   Create a file `api/index.php` to handle incoming requests:
   ```php
   <?php
   header("Content-Type: application/json");

   require_once '../Database.php';
   require_once 'controllers/ProductsController.php';
   require_once 'auth/JWTHandler.php';

   $method = $_SERVER['REQUEST_METHOD'];
   $uri = $_SERVER['REQUEST_URI'];

   // Example routing logic
   $path = explode('/', trim($uri, '/'));
   if ($path[0] === 'api' && isset($path[1])) {
       $resource = $path[1];
       if ($resource === 'products') {
           $controller = new ProductsController();
           if ($method === 'GET') {
               echo json_encode($controller->getProducts());
           } elseif ($method === 'POST') {
               $input = json_decode(file_get_contents("php://input"), true);
               echo json_encode($controller->createProduct($input));
           }
       }
   } else {
       http_response_code(404);
       echo json_encode(["message" => "Resource not found"]);
   }
   ?>
   ```

3. **Create the Products Controller:**
   Add logic for CRUD operations:
   ```php
   <?php
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

       public function createProduct($data) {
           $query = "INSERT INTO products (name, price, quantity_available) VALUES (:name, :price, :quantity)";
           $stmt = $this->db->prepare($query);
           $stmt->bindParam(':name', $data['name']);
           $stmt->bindParam(':price', $data['price']);
           $stmt->bindParam(':quantity', $data['quantity']);
           if ($stmt->execute()) {
               return ["message" => "Product created successfully"];
           } else {
               return ["message" => "Failed to create product"];
           }
       }
   }
   ?>
   ```

---

### **Step 17: Implement JWT Authentication**
1. **Install JWT Library:**
   Use a PHP JWT library like [Firebase JWT](https://github.com/firebase/php-jwt).
   ```bash
   composer require firebase/php-jwt
   ```

2. **Create a JWT Handler:**
   Add `auth/JWTHandler.php` to generate and verify tokens:
   ```php
   <?php
   use Firebase\JWT\JWT;
   use Firebase\JWT\Key;

   class JWTHandler {
       private static $secret_key = "your_secret_key"; // Replace with a strong secret key
       private static $issuer = "your_domain.com"; // Replace with your domain

       public static function generateToken($user_id) {
           $payload = [
               "iss" => self::$issuer,
               "iat" => time(),
               "exp" => time() + (60 * 60), // 1-hour expiration
               "data" => ["user_id" => $user_id]
           ];
           return JWT::encode($payload, self::$secret_key, 'HS256');
       }

       public static function validateToken($token) {
           try {
               $decoded = JWT::decode($token, new Key(self::$secret_key, 'HS256'));
               return (array) $decoded;
           } catch (Exception $e) {
               return null;
           }
       }
   }
   ?>
   ```

3. **Set Up User Authentication:**
   Create an `auth/login.php` to issue tokens:
   ```php
   <?php
   include '../Database.php';
   include 'JWTHandler.php';

   header("Content-Type: application/json");

   if ($_SERVER['REQUEST_METHOD'] === 'POST') {
       $input = json_decode(file_get_contents("php://input"), true);

       $database = new Database();
       $db = $database->getConnection();

       $query = "SELECT * FROM users WHERE username = :username";
       $stmt = $db->prepare($query);
       $stmt->bindParam(':username', $input['username']);
       $stmt->execute();

       $user = $stmt->fetch(PDO::FETCH_ASSOC);

       if ($user && password_verify($input['password'], $user['password'])) {
           $token = JWTHandler::generateToken($user['id']);
           echo json_encode(["token" => $token]);
       } else {
           http_response_code(401);
           echo json_encode(["message" => "Invalid credentials"]);
       }
   }
   ?>
   ```

4. **Secure API Routes:**
   Update `api/index.php` to check for a valid token before allowing access:
   ```php
   <?php
   $headers = getallheaders();
   if (!isset($headers['Authorization'])) {
       http_response_code(401);
       echo json_encode(["message" => "Unauthorized"]);
       exit;
   }

   $token = str_replace("Bearer ", "", $headers['Authorization']);
   $decoded = JWTHandler::validateToken($token);

   if (!$decoded) {
       http_response_code(401);
       echo json_encode(["message" => "Invalid or expired token"]);
       exit;
   }
   ?>
   ```

---

### **Test Your API**
1. **Login to Get a Token:**
   - Make a POST request to `auth/login.php` with a username and password.
   - Copy the token from the response.

2. **Access Secure Routes:**
   - Include the token in the `Authorization` header for secure API endpoints:
     ```
     Authorization: Bearer <your_token>
     ```

3. **Test with Tools:**
   - Use tools like **Postman** or **cURL** to test your API endpoints.

---

### Example cURL Commands
- **Login:**
  ```bash
  curl -X POST -H "Content-Type: application/json" -d '{"username":"admin", "password":"password"}' http://yourdomain.com/auth/login.php
  ```
- **Get Products (Authenticated):**
  ```bash
  curl -X GET -H "Authorization: Bearer <your_token>" http://yourdomain.com/api/products
  ```
