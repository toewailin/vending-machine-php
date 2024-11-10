### **1. Install Laravel**
If Laravel is not yet installed, install it using Composer:

```bash
composer create-project --prefer-dist laravel/laravel my-project
cd my-project
```

---

### **2. Configure Database Connection**
1. Open the `.env` file in your Laravel project root.
2. Update the database connection details:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=your_database_name
   DB_USERNAME=your_database_user
   DB_PASSWORD=your_database_password
   ```

   Replace the placeholders (`your_database_name`, etc.) with your actual database details.

3. Ensure the corresponding database driver is installed:
   - For MySQL:
     ```bash
     sudo apt install php-mysql
     ```
   - For SQLite, PostgreSQL, or SQL Server, install the relevant driver.

4. Test the connection by running a migration:
   ```bash
   php artisan migrate
   ```
   If successful, Laravel will create the default tables in the specified database.

---

### **3. Use PDO Directly in Laravel**
While Laravel primarily uses Eloquent or the query builder, you can directly access PDO using the `DB` facade.

#### Example 1: Raw PDO Connection
```php
use Illuminate\Support\Facades\DB;

Route::get('/pdo-connection', function () {
    $pdo = DB::connection()->getPdo();
    
    if ($pdo) {
        return "Connected to database: " . DB::connection()->getDatabaseName();
    } else {
        return "Failed to connect to database.";
    }
});
```

This fetches the raw PDO instance used by Laravel.

---

### **4. Execute Raw Queries Using PDO**
You can run raw SQL queries through PDO via the `DB` facade.

#### Example 2: Fetch Data
```php
Route::get('/fetch-products', function () {
    $products = DB::select('SELECT * FROM products WHERE quantity_available > ?', [0]);

    return response()->json($products);
});
```

#### Example 3: Insert Data
```php
Route::get('/add-product', function () {
    $result = DB::insert('INSERT INTO products (name, price, quantity_available) VALUES (?, ?, ?)', [
        'New Product', 9.99, 100
    ]);

    return $result ? "Product added successfully!" : "Failed to add product.";
});
```

---

### **5. Use Laravel’s Query Builder**
Laravel’s query builder uses PDO internally and offers a fluent syntax.

#### Example 4: Fetch Data
```php
use Illuminate\Support\Facades\DB;

Route::get('/products', function () {
    $products = DB::table('products')->where('quantity_available', '>', 0)->get();

    return response()->json($products);
});
```

#### Example 5: Insert Data
```php
Route::get('/add-product', function () {
    $id = DB::table('products')->insertGetId([
        'name' => 'Another Product',
        'price' => 5.99,
        'quantity_available' => 50
    ]);

    return "Inserted product with ID: " . $id;
});
```

---

### **6. Use Eloquent ORM (Recommended)**
Eloquent ORM is Laravel’s abstraction layer that builds on PDO. Define a model for your database table.

#### Step 1: Create a Model
Run the artisan command:
```bash
php artisan make:model Product
```

#### Step 2: Define the Model
Edit the `Product` model (`app/Models/Product.php`):
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'price', 'quantity_available'];
}
```

#### Step 3: Use the Model
Fetch and insert data using Eloquent.

**Fetch Data:**
```php
use App\Models\Product;

Route::get('/products', function () {
    $products = Product::where('quantity_available', '>', 0)->get();

    return response()->json($products);
});
```

**Insert Data:**
```php
Route::get('/add-product', function () {
    $product = Product::create([
        'name' => 'Product Eloquent',
        'price' => 7.99,
        'quantity_available' => 30
    ]);

    return response()->json($product);
});
```

---

### **7. Test the Connection**
Run the application:
```bash
php artisan serve
```
Visit the routes (e.g., `/products` or `/add-product`) in your browser or via Postman to test.

---

### **8. Debugging Database Issues**
- Check the `.env` file for typos or misconfigurations.
- Ensure the database is running and accessible from your Laravel project.
- Check Laravel logs (`storage/logs/laravel.log`) for detailed error messages.
