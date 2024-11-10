### **1. ProductsController Setup**
We assume the `ProductsController` has two methods:
- `getProducts`: Retrieves all products from the database.
- `createProduct`: Creates a new product in the database.

---

### **2. PHPUnit Test File**
Create a test file `ProductsControllerTest.php`.

#### **Test File Structure**
```php
<?php
use PHPUnit\Framework\TestCase;

class ProductsControllerTest extends TestCase {
    private $dbMock;
    private $controller;

    protected function setUp(): void {
        // Mock the PDO class
        $this->dbMock = $this->createMock(PDO::class);

        // Inject the mock into the ProductsController
        $this->controller = new ProductsController($this->dbMock);
    }
}
?>
```

---

### **3. Test `getProducts`**
#### **Normal Scenario**
- Test that `getProducts` returns a list of products.
```php
public function testGetProducts() {
    // Mock the PDOStatement
    $stmtMock = $this->createMock(PDOStatement::class);

    // Define the behavior of the fetchAll method
    $stmtMock->expects($this->once())
        ->method('fetchAll')
        ->willReturn([
            ['id' => 1, 'name' => 'Coke', 'price' => 3.99, 'quantity_available' => 10],
            ['id' => 2, 'name' => 'Pepsi', 'price' => 6.885, 'quantity_available' => 5],
        ]);

    // Mock the prepare method on the PDO object
    $this->dbMock->expects($this->once())
        ->method('prepare')
        ->with('SELECT * FROM products')
        ->willReturn($stmtMock);

    // Execute the test
    $result = $this->controller->getProducts();

    // Assert the result
    $this->assertEquals([
        ['id' => 1, 'name' => 'Coke', 'price' => 3.99, 'quantity_available' => 10],
        ['id' => 2, 'name' => 'Pepsi', 'price' => 6.885, 'quantity_available' => 5],
    ], $result);
}
```

#### **Edge Case: Empty Products Table**
- Test that `getProducts` handles an empty table gracefully.
```php
public function testGetProductsEmpty() {
    // Mock the PDOStatement
    $stmtMock = $this->createMock(PDOStatement::class);

    // Define the behavior of the fetchAll method
    $stmtMock->expects($this->once())
        ->method('fetchAll')
        ->willReturn([]);

    // Mock the prepare method on the PDO object
    $this->dbMock->expects($this->once())
        ->method('prepare')
        ->with('SELECT * FROM products')
        ->willReturn($stmtMock);

    // Execute the test
    $result = $this->controller->getProducts();

    // Assert the result is an empty array
    $this->assertEquals([], $result);
}
```

---

### **4. Test `createProduct`**
#### **Normal Scenario**
- Test that `createProduct` inserts a product successfully.
```php
public function testCreateProduct() {
    // Mock the PDOStatement
    $stmtMock = $this->createMock(PDOStatement::class);

    // Define the behavior of the execute method
    $stmtMock->expects($this->once())
        ->method('execute')
        ->willReturn(true);

    // Mock the prepare method on the PDO object
    $this->dbMock->expects($this->once())
        ->method('prepare')
        ->with('INSERT INTO products (name, price, quantity_available) VALUES (:name, :price, :quantity)')
        ->willReturn($stmtMock);

    // Execute the test
    $data = ['name' => 'Water', 'price' => 0.5, 'quantity' => 100];
    $result = $this->controller->createProduct($data);

    // Assert the result is true
    $this->assertTrue($result);
}
```

#### **Edge Case: Missing Required Fields**
- Test that `createProduct` fails when required fields are missing.
```php
public function testCreateProductMissingFields() {
    $this->expectException(InvalidArgumentException::class);

    // Execute the test with missing fields
    $data = ['name' => 'Water', 'price' => 0.5]; // Missing quantity
    $this->controller->createProduct($data);
}
```

#### **Edge Case: Database Error**
- Test that `createProduct` handles database errors gracefully.
```php
public function testCreateProductDatabaseError() {
    // Mock the PDOStatement
    $stmtMock = $this->createMock(PDOStatement::class);

    // Define the behavior of the execute method
    $stmtMock->expects($this->once())
        ->method('execute')
        ->willReturn(false);

    // Mock the prepare method on the PDO object
    $this->dbMock->expects($this->once())
        ->method('prepare')
        ->with('INSERT INTO products (name, price, quantity_available) VALUES (:name, :price, :quantity)')
        ->willReturn($stmtMock);

    // Execute the test
    $data = ['name' => 'Juice', 'price' => 2.5, 'quantity' => 50];
    $result = $this->controller->createProduct($data);

    // Assert the result is false
    $this->assertFalse($result);
}
```

---

### **5. Running the Tests**
Run your tests with PHPUnit:
```bash
vendor/bin/phpunit ProductsControllerTest.php --testdox
```

---

### **6. Example Output**
```
ProductsControllerTest
 ✔ Get products returns expected data
 ✔ Get products handles empty table
 ✔ Create product inserts successfully
 ✔ Create product fails with missing fields
 ✔ Create product handles database error gracefully
```
