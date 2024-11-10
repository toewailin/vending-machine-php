### **1. Refactor Controller for Dependency Injection**
Adjust the `ProductsController` to accept dependencies (e.g., the database connection) via the constructor. This makes it easier to replace the dependencies during testing.

#### Updated `ProductsController`:
```php
<?php
class ProductsController {
    private $db;

    public function __construct($db) {
        $this->db = $db;
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
        return $stmt->execute();
    }
}
?>
```

---

### **2. Install PHPUnit**
Make sure PHPUnit is installed. If not, you can install it via Composer:
```bash
composer require --dev phpunit/phpunit
```

---

### **3. Write Tests Using Mocking**
You will use PHPUnit to create mock objects for the database connection.

#### Test Setup:
```php
<?php
use PHPUnit\Framework\TestCase;

class ProductsControllerTest extends TestCase {
    private $dbMock;
    private $controller;

    protected function setUp(): void {
        // Create a mock for the PDO class
        $this->dbMock = $this->createMock(PDO::class);

        // Inject the mock into the controller
        $this->controller = new ProductsController($this->dbMock);
    }
}
?>
```

---

### **4. Test `getProducts` Method**
Mock the database interaction and test the `getProducts` method.

```php
public function testGetProducts() {
    // Create a mock for the PDOStatement
    $stmtMock = $this->createMock(PDOStatement::class);

    // Define the expected behavior of the statement
    $stmtMock->expects($this->once())
        ->method('execute')
        ->willReturn(true);

    $stmtMock->expects($this->once())
        ->method('fetchAll')
        ->willReturn([
            ['id' => 1, 'name' => 'Coke', 'price' => 3.99, 'quantity_available' => 10],
        ]);

    // Mock the prepare method on the PDO object
    $this->dbMock->expects($this->once())
        ->method('prepare')
        ->with('SELECT * FROM products')
        ->willReturn($stmtMock);

    // Call the method to test
    $result = $this->controller->getProducts();

    // Assert the expected result
    $this->assertEquals([
        ['id' => 1, 'name' => 'Coke', 'price' => 3.99, 'quantity_available' => 10],
    ], $result);
}
```

---

### **5. Test `createProduct` Method**
Mock the database interaction for the `createProduct` method.

```php
public function testCreateProduct() {
    // Create a mock for the PDOStatement
    $stmtMock = $this->createMock(PDOStatement::class);

    // Define the expected behavior of the statement
    $stmtMock->expects($this->once())
        ->method('execute')
        ->willReturn(true);

    // Mock the prepare method on the PDO object
    $this->dbMock->expects($this->once())
        ->method('prepare')
        ->with('INSERT INTO products (name, price, quantity_available) VALUES (:name, :price, :quantity)')
        ->willReturn($stmtMock);

    // Call the method to test
    $data = ['name' => 'Pepsi', 'price' => 6.885, 'quantity' => 5];
    $result = $this->controller->createProduct($data);

    // Assert the expected result
    $this->assertTrue($result);
}
```

---

### **6. Run PHPUnit Tests**
Execute the tests with the following command:
```bash
vendor/bin/phpunit --testdox
```

---

### **7. Example Output**
When all tests pass, you will see an output like this:
```
ProductsControllerTest
 ✔ Get products returns expected data
 ✔ Create product executes successfully
```
