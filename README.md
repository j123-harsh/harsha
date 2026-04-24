# harsha
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const app = express();
app.use(express.json());
app.use(cors());

// Connect DB
mongoose.connect("mongodb://127.0.0.1:27017/shop");

// Models
const User = mongoose.model("User", {
  username: String,
  password: String
});

const Product = mongoose.model("Product", {
  name: String,
  price: Number,
  image: String
});

// Register
app.post("/register", async (req, res) => {
  const bcrypt = require("bcryptjs");
  const hash = await bcrypt.hash(req.body.password, 10);
  const user = new User({ username: req.body.username, password: hash });
  await user.save();
  res.send("User created");
});

// Login
app.post("/login", async (req, res) => {
  const bcrypt = require("bcryptjs");
  const user = await User.findOne({ username: req.body.username });

  if (!user) return res.status(400).send("User not found");

  const valid = await bcrypt.compare(req.body.password, user.password);
  if (!valid) return res.status(400).send("Wrong password");

  res.send({ message: "Login success" });
});

// Products
app.get("/products", async (req, res) => {
  const products = await Product.find();
  res.send(products);
});

// Seed products (run once)
app.get("/seed", async (req, res) => {
  await Product.insertMany([
    { name: "Shoes", price: 2000, image: "https://via.placeholder.com/150" },
    { name: "Watch", price: 3000, image: "https://via.placeholder.com/150" },
    { name: "Bag", price: 1500, image: "https://via.placeholder.com/150" }
  ]);
  res.send("Products added");
});

app.listen(5000, () => console.log("Server running on 5000"));

<!DOCTYPE html>
<html>
<head>
<title>Shop</title>
<link rel="stylesheet" href="style.css">
</head>
<body>

<div id="login">
  <h2>Login</h2>
  <input id="user" placeholder="Username">
  <input id="pass" type="password" placeholder="Password">
  <button onclick="login()">Login</button>
  <button onclick="register()">Register</button>
</div>

<div id="shop" style="display:none;">
  <h2>Products</h2>
  <input id="search" placeholder="Search..." onkeyup="searchProduct()">

  <div id="products"></div>

  <h3>Cart</h3>
  <ul id="cart"></ul>
  <h4>Total: ₹<span id="total">0</span></h4>

  <button onclick="checkout()">Pay Now</button>
  <button onclick="logout()">Logout</button>
</div>

<script src="app.js"></script>
</body>
</html>
body {
  font-family: Arial;
  padding: 20px;
}

#products {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
}

.product {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
}

img {
  width: 100%;
}

let products = [];
let cart = [];

// Login
async function login() {
  const res = await fetch("http://localhost:5000/login", {
    method: "POST",
    headers: {"Content-Type":"application/json"},
    body: JSON.stringify({
      username: user.value,
      password: pass.value
    })
  });

  if (res.ok) {
    loginDiv.style.display = "none";
    shop.style.display = "block";
    loadProducts();
  } else {
    alert("Login failed");
  }
}

// Register
async function register() {
  await fetch("http://localhost:5000/register", {
    method: "POST",
    headers: {"Content-Type":"application/json"},
    body: JSON.stringify({
      username: user.value,
      password: pass.value
    })
  });
  alert("Registered!");
}

// Load products
async function loadProducts() {
  const res = await fetch("http://localhost:5000/products");
  products = await res.json();
  display(products);
}

// Display
function display(list) {
  productsDiv.innerHTML = "";
  list.forEach(p => {
    productsDiv.innerHTML += `
      <div class="product">
        <img src="${p.image}">
        <h4>${p.name}</h4>
        <p>₹${p.price}</p>
        <button onclick="add(${p.price}, '${p.name}')">Add</button>
      </div>
    `;
  });
}

// Add to cart
function add(price, name) {
  cart.push({price, name});
  renderCart();
}

// Render cart
function renderCart() {
  cartDiv.innerHTML = "";
  let total = 0;

  cart.forEach(i => {
    total += i.price;
    cartDiv.innerHTML += `<li>${i.name} - ₹${i.price}</li>`;
  });

  totalSpan.innerText = total;
}

// Search
function searchProduct() {
  let q = search.value.toLowerCase();
  let filtered = products.filter(p => p.name.toLowerCase().includes(q));
  display(filtered);
}

// Logout
function logout() {
  location.reload();
}

// Payment (Razorpay demo)
function checkout() {
  alert("Integrate Razorpay here");
}
