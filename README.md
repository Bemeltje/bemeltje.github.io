<!DOCTYPE html><html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Wallet Systeem</title>
<style>
    body { font-family: Arial, sans-serif; background-color: #fff; color: #222; margin: 0; padding: 0; }
    header { background-color: #333; color: #fff; padding: 10px; text-align: center; }
    button, input, select { font-size: 1rem; margin: 5px; padding: 8px; border-radius: 5px; border: 1px solid #ccc; }
    button { background-color: #0066cc; color: white; border: none; cursor: pointer; }
    button:hover { background-color: #004999; }
    .hidden { display: none; }
    .container { padding: 15px; }
    .item { border: 1px solid #ccc; border-radius: 8px; padding: 10px; margin: 10px 0; cursor: pointer; }
    .admin { background-color: #f8f8f8; padding: 10px; border-radius: 5px; }
    .cart-item { margin: 5px 0; }
    .log { max-height: 150px; overflow-y: scroll; border: 1px solid #ccc; padding: 5px; margin-top: 10px; }
</style>
</head>
<body>
<header>
    <h1>Fictief Wallet Systeem</h1>
</header>
<div class="container">
    <div id="homeScreen"></div>
    <div id="loginSection" class="hidden">
        <h2>Voer pincode in voor <span id="loginUserName"></span></h2>
        <input type="password" id="loginPin" placeholder="Pincode">
        <button onclick="loginUser()">Inloggen</button>
    </div>
    <div id="dashboard" class="hidden">
        <h2 id="welcomeMsg"></h2>
        <div id="balanceSection"></div>
        <div id="cartSection"></div>
        <div id="productList"></div>
        <div id="adminPanel" class="hidden"></div>
        <div class="log" id="logSection"></div>
    </div>
</div>
<script>
let users = [];
let products = [];
let currentUser = null;
let currentLoginUser = null;
let cart = [];
let log = [];
let adminPin = '0000';function saveData() { localStorage.setItem('users', JSON.stringify(users)); localStorage.setItem('products', JSON.stringify(products)); localStorage.setItem('log', JSON.stringify(log)); localStorage.setItem('adminPin', adminPin); }

function loadData() { users = JSON.parse(localStorage.getItem('users')) || []; products = JSON.parse(localStorage.getItem('products')) || []; log = JSON.parse(localStorage.getItem('log')) || []; adminPin = localStorage.getItem('adminPin') || '0000'; renderHomeScreen(); }

function renderHomeScreen() { const home = document.getElementById('homeScreen'); home.innerHTML = '<h2>Selecteer account</h2>'; users.forEach((u, i) => { home.innerHTML += <div class="item" onclick="promptLogin(${i})">${u.name} (${u.role})</div>; }); home.innerHTML += <button onclick=promptAdminLogin()>Admin</button>; }

function promptLogin(index) { currentLoginUser = users[index]; document.getElementById('homeScreen').classList.add('hidden'); document.getElementById('loginSection').classList.remove('hidden'); document.getElementById('loginUserName').innerText = currentLoginUser.name; }

function loginUser() { const pin = document.getElementById('loginPin').value.trim(); if(pin === currentLoginUser.pin) { currentUser = currentLoginUser; document.getElementById('loginSection').classList.add('hidden'); document.getElementById('dashboard').classList.remove('hidden'); document.getElementById('welcomeMsg').innerText = Welkom, ${currentUser.name}; updateDashboard(); } else { alert('Ongeldige pincode'); } }

function promptAdminLogin() { const pin = prompt('Voer admin pincode in:'); if(pin === adminPin) { currentUser = { name: 'Admin', role: 'admin' }; document.getElementById('homeScreen').classList.add('hidden'); document.getElementById('dashboard').classList.remove('hidden'); document.getElementById('welcomeMsg').innerText = 'Welkom Admin'; updateDashboard(); } else { alert('Ongeldige admin pincode'); } }

function updateDashboard() { document.getElementById('balanceSection').innerHTML = <p>Saldo: €${currentUser.balance.toFixed(2)}</p>; renderProducts(); renderCart(); renderLog(); if(currentUser.role === 'admin' || currentUser.role === 'subadmin') { renderAdminPanel(); } }

function renderProducts() { const productList = document.getElementById('productList'); productList.innerHTML = '<h3>Producten</h3>'; products.forEach((p, i) => { productList.innerHTML += <div class="item">${p.name} - €${p.price.toFixed(2)} (${p.stock} op voorraad) <button onclick="addToCart(${i})">Toevoegen aan winkelwagen</button></div>; }); }

function addToCart(index) { cart.push(products[index]); renderCart(); }

function renderCart() { const cartSection = document.getElementById('cartSection'); if(cart.length === 0) { cartSection.innerHTML = ''; return; } cartSection.innerHTML = '<h3>Winkelwagen</h3>'; cart.forEach((p, i) => { cartSection.innerHTML += <div class="cart-item">${p.name} - €${p.price.toFixed(2)} <button onclick="removeFromCart(${i})">Verwijderen</button></div>; }); cartSection.innerHTML += <button onclick="checkout()">Bevestig aankoop</button>; }

function removeFromCart(i) { cart.splice(i, 1); renderCart(); }

function checkout() { let total = cart.reduce((sum, p) => sum + p.price, 0); let allowedNegative = currentUser.role === 'guest' ? 0 : 10; if(currentUser.balance + allowedNegative >= total) { cart.forEach(p => { if(p.stock > 0) { p.stock--; log.push(${currentUser.name} kocht ${p.name} voor €${p.price.toFixed(2)} op ${new Date().toLocaleString()}); } }); currentUser.balance -= total; cart = []; saveData(); updateDashboard(); alert('Aankoop voltooid'); } else { alert('Onvoldoende saldo'); } }

function renderLog() { const logSection = document.getElementById('logSection'); logSection.innerHTML = '<h4>Logboek</h4>' + log.map(e => <div>${e}</div>).join(''); }

function renderAdminPanel() { const panel = document.getElementById('adminPanel'); panel.classList.remove('hidden'); panel.innerHTML = '<h3>Admin Beheer</h3>'; panel.innerHTML += '<button onclick="addProduct()">Product toevoegen</button>'; panel.innerHTML += '<button onclick="addFunds()">Geld toevoegen aan account</button>'; panel.innerHTML += '<button onclick="addUser()">Account toevoegen</button>'; panel.innerHTML += '<button onclick="changeAdminPin()">Wijzig hoofdadmin pincode</button>'; }

function addUser() { const name = prompt('Naam:'); const pin = prompt('Pincode:'); const role = prompt('Rol (admin, subadmin, user, guest, vaste):', 'user'); const balance = parseFloat(prompt('Startsaldo:', '0')) || 0; users.push({ name, pin, role, balance }); saveData(); renderHomeScreen(); }

function addProduct() { const name = prompt('Productnaam:'); const price = parseFloat(prompt('Prijs:')) || 0; const stock = parseInt(prompt('Voorraad:'), 10) || 0; products.push({ name, price, stock }); saveData(); updateDashboard(); }

function addFunds() { const target = prompt('Naam van account:'); const amount = parseFloat(prompt('Bedrag:')) || 0; const user = users.find(u => u.name === target); if(user) { user.balance += amount; saveData(); updateDashboard(); } }

function changeAdminPin() { const newPin = prompt('Nieuwe hoofdadmin pincode:'); if(newPin) { adminPin = newPin; saveData(); alert('Hoofdadmin pincode gewijzigd'); } }

loadData(); </script>

</body>
</html>