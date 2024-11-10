### **Step 1: Design the Route**
Define a route structure for the purchase action:
- Example route: `/purchase/{product_id}`
- Replace `{product_id}` dynamically with the product’s ID.

---

### **Step 2: Route Handling Logic**
1. **Create a Router Class:**
   A simple router maps URL patterns to specific controller actions.

   ```php
   <?php
   class Router {
       private $routes = [];

       // Register a route
       public function add($method, $pattern, $callback) {
           $this->routes[] = [
               'method' => strtoupper($method),
               'pattern' => $this->convertToRegex($pattern),
               'callback' => $callback
           ];
       }

       // Match the current request to a route
       public function dispatch($method, $uri) {
           foreach ($this->routes as $route) {
               if ($route['method'] === strtoupper($method) && preg_match($route['pattern'], $uri, $matches)) {
                   array_shift($matches); // Remove full match
                   return call_user_func_array($route['callback'], $matches);
               }
           }
           http_response_code(404);
           echo json_encode(['message' => 'Route not found']);
       }

       private function convertToRegex($pattern) {
           return '/^' . preg_replace('/{([\w]+)}/', '(?P<\1>[\w-]+)', str_replace('/', '\/', $pattern)) . '$/';
       }
   }
   ?>
   ```

---

### **Step 3: Define the Purchase Route**
Add the route for the `purchase` action in your entry point (e.g., `index.php`):

```php
<?php
require_once 'Router.php';
require_once 'controllers/ProductsController.php';

$router = new Router();

// Add the purchase route
$router->add('GET', '/purchase/{product_id}', function ($product_id) {
    $controller = new ProductsController();
    $controller->purchaseProduct($product_id);
});

// Dispatch the request
$router->dispatch($_SERVER['REQUEST_METHOD'], parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH));
?>
```

---

### **Step 4: Update the Products Controller**
Add the `purchaseProduct` method to the `ProductsController`:

```php
<?php
class ProductsController {
    private $db;

    public function __construct() {
        $database = new Database();
        $this->db = $database->getConnection();
    }

    public function purchaseProduct($product_id) {
        // Fetch the product
        $query = "SELECT * FROM products WHERE id = :id";
        $stmt = $this->db->prepare($query);
        $stmt->bindParam(':id', $product_id);
        $stmt->execute();

        $product = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$product) {
            http_response_code(404);
            echo json_encode(['message' => 'Product not found']);
            return;
        }

        // Update the product quantity
        if ($product['quantity_available'] > 0) {
            $new_quantity = $product['quantity_available'] - 1;

            $updateQuery = "UPDATE products SET quantity_available = :quantity WHERE id = :id";
            $updateStmt = $this->db->prepare($updateQuery);
            $updateStmt->bindParam(':quantity', $new_quantity);
            $updateStmt->bindParam(':id', $product_id);

            if ($updateStmt->execute()) {
                echo json_encode([
                    'message' => 'Purchase successful',
                    'product' => $product,
                    'remaining_quantity' => $new_quantity
                ]);
            } else {
                http_response_code(500);
                echo json_encode(['message' => 'Failed to update product quantity']);
            }
        } else {
            echo json_encode(['message' => 'Product out of stock']);
        }
    }
}
?>
```

---

### **Step 5: Test the Route**
1. **Example URL:**
   ```
   http://yourdomain.com/purchase/1
   ```
   Replace `1` with a valid product ID.

2. **Expected Response:**
   On success:
   ```json
   {
       "message": "Purchase successful",
       "product": {
           "id": 1,
           "name": "Coke",
           "price": 3.99,
           "quantity_available": 10
       },
       "remaining_quantity": 9
   }
   ```

   If the product is out of stock:
   ```json
   {
       "message": "Product out of stock"
   }
   ```

   If the product ID is invalid:
   ```json
   {
       "message": "Product not found"
   }
   ```

---

### **Step 6: SEO-Friendly URLs**
- Ensure URLs are human-readable, like `/purchase/coke` instead of `/purchase/1`.
- Update the database to include a `slug` column for products.
- Modify the route to use the slug instead of the ID.

```php
$router->add('GET', '/purchase/{slug}', function ($slug) {
    $controller = new ProductsController();
    $controller->purchaseProductBySlug($slug);
});
```

Update the controller method to fetch the product by `slug` instead of `id`.

---

### **Step 7: Enable Pretty URLs**
1. Add an `.htaccess` file in the root directory for Apache:
   ```apache
   RewriteEngine On
   RewriteCond %{REQUEST_FILENAME} !-f
   RewriteCond %{REQUEST_FILENAME} !-d
   RewriteRule ^ index.php [L]
   ```

2. For Nginx, update the server configuration to route all requests to `index.php`.
