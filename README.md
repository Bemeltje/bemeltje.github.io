<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Wallet Systeem</title>
<style>
    body { font-family: Arial, sans-serif; background-color: #fff; color: #222; margin: 0; padding: 0; }
    header { background-color: #333; color: #fff; padding: 10px; text-align: center; }
    button { font-size: 1rem; margin: 5px; padding: 8px 12px; border-radius: 5px; border: none; background-color: #0066cc; color: white; cursor: pointer; }
    button:hover { background-color: #004999; }
    .hidden { display: none; }
    .container { padding: 15px; }
    .item { border: 1px solid #ccc; border-radius: 8px; padding: 10px; margin: 10px 0; }
</style>
</head>
<body>
<header>
    <h1>Fictief Wallet Systeem</h1>
</header>
<div class="container">
    <!-- Hoofdscherm -->
    <div id="homeScreen">
        <h2>Kies een account</h2>
        <div id="accountList"></div>
        <hr>
        <button onclick="loginAdmin()">🔑 Admin</button>
    </div>

    <!-- Dashboard gebruiker -->
    <div id="userDashboard" class="hidden">
        <h2 id="welcomeUser"></h2>
        <div id="balanceSection"></div>
        <div id="productList"></div>
        <button onclick="logout()">⬅ Terug</button>
    </div>

    <!-- Admin dashboard -->
    <div id="adminDashboard" class="hidden">
        <h2>Admin Beheer</h2>
        <button onclick="addUser()">Account toevoegen</button>
        <button onclick="addFunds()">Geld toevoegen</button>
        <button onclick="addProduct()">Product toevoegen</button>
        <button onclick="changeAdminPin()">Admin pincode wijzigen</button>
        <div id="adminProducts"></div>
        <button onclick="logout()">⬅ Terug</button>
    </div>
</div>

<script>
let users = [];
let products = [];
let currentUser = null;
let adminPin = localStorage.getItem('adminPin') || "0000";

function saveData() {
    localStorage.setItem('users', JSON.stringify(users));
    localStorage.setItem('products', JSON.stringify(products));
    localStorage.setItem('adminPin', adminPin);
}

function loadData() {
    users = JSON.parse(localStorage.getItem('users')) || [
        { name: "Jan", pin: "1111", balance: 10, role: "user" },
        { name: "Piet", pin: "2222", balance: 5, role: "user" }
    ];
    products = JSON.parse(localStorage.getItem('products')) || [
        { name: "Chips", price: 1.5, stock: 5 },
        { name: "Snoep", price: 1.0, stock: 10 }
    ];
}

function renderHomeScreen() {
    const list = document.getElementById('accountList');
    list.innerHTML = "";
    users.forEach((u, i) => {
        const btn = document.createElement('button');
        btn.textContent = u.name;
        btn.onclick = () => loginUser(i);
        list.appendChild(btn);
    });
}

function loginUser(index) {
    const pin = prompt(`Pincode voor ${users[index].name}:`);
    if (pin === users[index].pin) {
        currentUser = users[index];
        document.getElementById('homeScreen').classList.add('hidden');
        document.getElementById('userDashboard').classList.remove('hidden');
        document.getElementById('welcomeUser').innerText = `Welkom, ${currentUser.name}`;
        updateUserDashboard();
    } else {
        alert("Foutieve pincode");
    }
}

function loginAdmin() {
    const pin = prompt("Voer admin pincode in:");
    if (pin === adminPin) {
        document.getElementById('homeScreen').classList.add('hidden');
        document.getElementById('adminDashboard').classList.remove('hidden');
        renderAdminProducts();
    } else {
        alert("Foutieve admin pincode");
    }
}

function updateUserDashboard() {
    document.getElementById('balanceSection').innerHTML = `<p>Saldo: €${currentUser.balance.toFixed(2)}</p>`;
    const productList = document.getElementById('productList');
    productList.innerHTML = "";
    products.forEach((p, i) => {
        const div = document.createElement('div');
        div.className = "item";
        div.innerHTML = `${p.name} - €${p.price.toFixed(2)} (${p.stock} op voorraad) 
            <button onclick="buyProduct(${i})">Kopen</button>`;
        productList.appendChild(div);
    });
}

function buyProduct(index) {
    const product = products[index];
    if (product.stock > 0 && currentUser.balance >= product.price) {
        product.stock--;
        currentUser.balance -= product.price;
        saveData();
        updateUserDashboard();
    } else {
        alert("Onvoldoende saldo of geen voorraad");
    }
}

function renderAdminProducts() {
    const adminProducts = document.getElementById('adminProducts');
    adminProducts.innerHTML = "<h3>Producten</h3>";
    products.forEach((p) => {
        adminProducts.innerHTML += `<div>${p.name} - €${p.price.toFixed(2)} (${p.stock} stuks)</div>`;
    });
}

function addUser() {
    const name = prompt("Naam:");
    const pin = prompt("Pincode:");
    const balance = parseFloat(prompt("Startsaldo:")) || 0;
    users.push({ name, pin, balance, role: "user" });
    saveData();
    renderHomeScreen();
}

function addFunds() {
    const target = prompt("Naam van account:");
    const amount = parseFloat(prompt("Bedrag:")) || 0;
    const user = users.find(u => u.name === target);
    if (user) {
        user.balance += amount;
        saveData();
    }
}

function addProduct() {
    const name = prompt("Productnaam:");
    const price = parseFloat(prompt("Prijs:")) || 0;
    const stock = parseInt(prompt("Voorraad:"), 10) || 0;
    products.push({ name, price, stock });
    saveData();
    renderAdminProducts();
}

function changeAdminPin() {
    const newPin = prompt("Nieuwe admin pincode:");
    if (newPin) {
        adminPin = newPin;
        saveData();
        alert("Admin pincode gewijzigd");
    }
}

function logout() {
    currentUser = null;
    document.getElementById('userDashboard').classList.add('hidden');
    document.getElementById('adminDashboard').classList.add('hidden');
    document.getElementById('homeScreen').classList.remove('hidden');
    renderHomeScreen();
}

loadData();
renderHomeScreen();
</script>
</body>
</html>