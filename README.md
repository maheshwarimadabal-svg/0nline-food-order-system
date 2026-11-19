# 0nline-food-order-system
The Online Food Ordering System is a web-based application that allows customers to order food from restaurants through the internet. Customers can view food items, check prices, select their favorite dishes, add items to the shopping cart, and place orders online.  The system makes food ordering easier, faster, and more convenien
1. Introduction
The Online Food Ordering System is a web-based application that allows customers to order food from restaurants through the internet. Customers can view food items, check prices, select their favorite dishes, add items to the shopping cart, and place orders online.

The system makes food ordering easier, faster, and more convenient. It reduces the time spent visiting restaurants and allows customers to order food from anywhere.
online-food-order/
│
├── index.html
├── style.css
├── script.js
├── db.php
├── order.php
└── database.sql

<!DOCTYPE html>
<html>
<head>
    <title>Online Food Ordering System</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

<header>
    <h1>🍔 Foodie Restaurant</h1>
    <p>Order Your Favourite Food Online</p>
</header>

<div class="container">
    <h2>Our Food Menu</h2>

    <div class="menu" id="food-menu"></div>

    <h2>Shopping Cart</h2>
    <div id="cart"></div>

    <h3 id="total">Total: ₹0</h3>

    <input type="text" id="customer"
           placeholder="Enter your name">

    <button onclick="placeOrder()">Place Order</button>
</div>

<script src="script.js"></script>
</body>
</html>

const foods = [
    {id: 1, name: "Burger", price: 120},
    {id: 2, name: "Pizza", price: 250},
    {id: 3, name: "Biryani", price: 180},
    {id: 4, name: "Dosa", price: 60},
    {id: 5, name: "Fried Rice", price: 140},
    {id: 6, name: "Sandwich", price: 90}
];

let cart = [];

function displayFood() {
    const menu = document.getElementById("food-menu");

    menu.innerHTML = foods.map(food => `
        <div class="food">
            <h3>${food.name}</h3>
            <p>Price: ₹${food.price}</p>
            <button onclick="addToCart(${food.id})">
                Add to Cart
            </button>
        </div>
    `).join("");
}

function addToCart(id) {
    const food = foods.find(item => item.id === id);
    const existing = cart.find(item => item.id === id);

    if (existing) {
        existing.quantity++;
    } else {
        cart.push({...food, quantity: 1});
    }

    displayCart();
}

function displayCart() {
    const cartDiv = document.getElementById("cart");
    let total = 0;

    cartDiv.innerHTML = cart.map(item => {
        total += item.price * item.quantity;

        return `
            <p>
                ${item.name} - ₹${item.price}
                x ${item.quantity}
                <button onclick="removeItem(${item.id})">
                    Remove
                </button>
            </p>
        `;
    }).join("");

    document.getElementById("total").innerText =
        "Total: ₹" + total;
}

function removeItem(id) {
    cart = cart.filter(item => item.id !== id);
    displayCart();
}

function placeOrder() {
    const name = document.getElementById("customer").value;

    if (!name.trim() || cart.length === 0) {
        alert("Enter your name and add food to cart!");
        return;
    }

    fetch("order.php", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            customer: name,
            cart: cart
        })
    })
    .then(response => response.json())
    .then(data => {
        alert(data.message);

        if (data.success) {
            cart = [];
            displayCart();
            document.getElementById("customer").value = "";
        }
    })
    .catch(() => alert("Unable to place order."));
}

displayFood();
sql

CREATE DATABASE food_order;

USE food_order;

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer VARCHAR(100),
    food_items TEXT,
    total DECIMAL(10,2),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);php

<?php
$conn = new mysqli(
    "localhost",
    "root",
    "",
    "food_order"
);

if ($conn->connect_error) {
    die("Database connection failed");
}
<?php
header("Content-Type: application/json");

require "db.php";

$data = json_decode(file_get_contents("php://input"), true);

if (
    !$data ||
    empty(trim($data["customer"] ?? "")) ||
    empty($data["cart"]) ||
    !is_array($data["cart"])
) {
    echo json_encode([
        "success" => false,
        "message" => "Invalid order details"
    ]);
    exit;
}

$prices = [
    1 => ["name" => "Burger", "price" => 120],
    2 => ["name" => "Pizza", "price" => 250],
    3 => ["name" => "Biryani", "price" => 180],
    4 => ["name" => "Dosa", "price" => 60],
    5 => ["name" => "Fried Rice", "price" => 140],
    6 => ["name" => "Sandwich", "price" => 90]
];

$items = [];
$total = 0;

foreach ($data["cart"] as $item) {
    $id = filter_var($item["id"] ?? null, FILTER_VALIDATE_INT);
    $qty = filter_var($item["quantity"] ?? null, FILTER_VALIDATE_INT);

    if (!$id || !$qty || $qty < 1 || $qty > 50 || !isset($prices[$id])) {
        echo json_encode([
            "success" => false,
            "message" => "Invalid food item or quantity"
        ]);
        exit;
    }

    $food = $prices[$id];
    $total += $food["price"] * $qty;

    $items[] = [
        "name" => $food["name"],
        "quantity" => $qty,
        "price" => $food["price"]
    ];
}

$customer = trim($data["customer"]);

$stmt = $conn->prepare(
    "INSERT INTO orders (customer, food_items, total)
     VALUES (?, ?, ?)"
);

$foodItems = json_encode($items);

$stmt->bind_param("ssd", $customer, $foodItems, $total);

if ($stmt->execute()) {
    echo json_encode([
        "success" => true,
        "message" => "Order placed successfully!"
    ]);
} else {
    echo json_encode([
        "success" => false,
        "message" => "Order failed"
    ]);
}

$stmt->close();
$conn->close();
?>

<?php
header("Content-Type: application/json");

require "db.php";

$data = json_decode(file_get_contents("php://input"), true);

if (
    !$data ||
    empty(trim($data["customer"] ?? "")) ||
    empty($data["cart"]) ||
    !is_array($data["cart"])
) {
    echo json_encode([
        "success" => false,
        "message" => "Invalid order details"
    ]);
    exit;
}

$prices = [
    1 => ["name" => "Burger", "price" => 120],
    2 => ["name" => "Pizza", "price" => 250],
    3 => ["name" => "Biryani", "price" => 180],
    4 => ["name" => "Dosa", "price" => 60],
    5 => ["name" => "Fried Rice", "price" => 140],
    6 => ["name" => "Sandwich", "price" => 90]
];

$items = [];
$total = 0;

foreach ($data["cart"] as $item) {
    $id = filter_var($item["id"] ?? null, FILTER_VALIDATE_INT);
    $qty = filter_var($item["quantity"] ?? null, FILTER_VALIDATE_INT);

    if (!$id || !$qty || $qty < 1 || $qty > 50 || !isset($prices[$id])) {
        echo json_encode([
            "success" => false,
            "message" => "Invalid food item or quantity"
        ]);
        exit;
    }

    $food = $prices[$id];
    $total += $food["price"] * $qty;

    $items[] = [
        "name" => $food["name"],
        "quantity" => $qty,
        "price" => $food["price"]
    ];
}

$customer = trim($data["customer"]);

$stmt = $conn->prepare(
    "INSERT INTO orders (customer, food_items, total)
     VALUES (?, ?, ?)"
);

$foodItems = json_encode($items);

$stmt->bind_param("ssd", $customer, $foodItems, $total);

if ($stmt->execute()) {
    echo json_encode([
        "success" => true,
        "message" => "Order placed successfully!"
    ]);
} else {
    echo json_encode([
        "success" => false,
        "message" => "Order failed"
    ]);
}

$stmt->close();
$conn->close();
?>
