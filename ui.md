### **Step 1: Choose a Framework**
Use a front-end framework like **Bootstrap** for responsive and consistent styling.

1. **Download Bootstrap:**
   Visit [Bootstrap's website](https://getbootstrap.com/) and download the framework or include it via a CDN:
   ```html
   <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
   <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
   ```

---

### **Step 2: Create a Base Template**
Create a `base.php` file for a common layout structure:
```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vending Machine</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container-fluid">
            <a class="navbar-brand" href="#">Vending Machine</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav ms-auto">
                    <li class="nav-item"><a class="nav-link" href="products.php">Products</a></li>
                    <li class="nav-item"><a class="nav-link" href="cart.php">Cart</a></li>
                    <li class="nav-item"><a class="nav-link" href="logout.php">Logout</a></li>
                </ul>
            </div>
        </div>
    </nav>
    <div class="container mt-4">
        <?php include($view); ?>
    </div>
</body>
</html>
```

---

### **Step 3: Create a Product List View**
Design a `products.php` file for displaying available products:
```php
<div class="row">
    <?php foreach ($products as $product): ?>
        <div class="col-md-4">
            <div class="card mb-4">
                <img src="product_images/<?php echo $product['id']; ?>.jpg" class="card-img-top" alt="<?php echo $product['name']; ?>">
                <div class="card-body">
                    <h5 class="card-title"><?php echo $product['name']; ?></h5>
                    <p class="card-text">$<?php echo number_format($product['price'], 2); ?></p>
                    <p class="card-text">Available: <?php echo $product['quantity_available']; ?></p>
                    <a href="purchase.php?id=<?php echo $product['id']; ?>" class="btn btn-primary">Buy Now</a>
                </div>
            </div>
        </div>
    <?php endforeach; ?>
</div>
```

---

### **Step 4: Create Admin Product Management UI**
Create a `manage_products.php` file for CRUD operations:
```php
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1>Manage Products</h1>
    <a href="add_product.php" class="btn btn-success">Add New Product</a>
</div>

<table class="table table-bordered">
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Price</th>
            <th>Quantity</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        <?php foreach ($products as $product): ?>
        <tr>
            <td><?php echo $product['id']; ?></td>
            <td><?php echo $product['name']; ?></td>
            <td>$<?php echo number_format($product['price'], 2); ?></td>
            <td><?php echo $product['quantity_available']; ?></td>
            <td>
                <a href="edit_product.php?id=<?php echo $product['id']; ?>" class="btn btn-warning btn-sm">Edit</a>
                <a href="delete_product.php?id=<?php echo $product['id']; ?>" class="btn btn-danger btn-sm">Delete</a>
            </td>
        </tr>
        <?php endforeach; ?>
    </tbody>
</table>
```

---

### **Step 5: Create User Registration and Login Forms**
#### Registration Form (`register.php`):
```php
<div class="row justify-content-center">
    <div class="col-md-6">
        <h1>Register</h1>
        <form method="POST">
            <div class="mb-3">
                <label for="username" class="form-label">Username</label>
                <input type="text" class="form-control" id="username" name="username" required>
            </div>
            <div class="mb-3">
                <label for="password" class="form-label">Password</label>
                <input type="password" class="form-control" id="password" name="password" required>
            </div>
            <div class="mb-3">
                <label for="role" class="form-label">Role</label>
                <select class="form-select" id="role" name="role">
                    <option value="user">User</option>
                    <option value="admin">Admin</option>
                </select>
            </div>
            <button type="submit" class="btn btn-primary">Register</button>
        </form>
    </div>
</div>
```

#### Login Form (`login.php`):
```php
<div class="row justify-content-center">
    <div class="col-md-6">
        <h1>Login</h1>
        <form method="POST">
            <div class="mb-3">
                <label for="username" class="form-label">Username</label>
                <input type="text" class="form-control" id="username" name="username" required>
            </div>
            <div class="mb-3">
                <label for="password" class="form-label">Password</label>
                <input type="password" class="form-control" id="password" name="password" required>
            </div>
            <button type="submit" class="btn btn-primary">Login</button>
        </form>
    </div>
</div>
```

---

### **Step 6: Add Custom Styling**
Create a `styles.css` file in your project for additional styling:
```css
body {
    background-color: #f8f9fa;
}

.navbar-brand {
    font-size: 1.5rem;
    font-weight: bold;
}

.card img {
    height: 200px;
    object-fit: cover;
}

.card-title {
    font-size: 1.2rem;
    font-weight: bold;
}
```
Include it in your `base.php` file:
```html
<link rel="stylesheet" href="styles.css">
```
