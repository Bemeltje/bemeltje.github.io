<!DOCTYPE html><html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Wallet Systeem</title>
<style>
    body { font-family: Arial, sans-serif; background-color: #fff; color: #222; margin: 0; padding: 0; }
    header { background-color: #333; color: #fff; padding: 10px; text-align: center; }
    button, input { font-size: 1rem; margin: 5px; padding: 8px; border-radius: 5px; border: 1px solid #ccc; }
    button { background-color: #0066cc; color: white; border: none; cursor: pointer; }
    button:hover { background-color: #004999; }
    .hidden { display: none; }
    .container { padding: 15px; }
    .item { border: 1px solid #ccc; border-radius: 8px; padding: 10px; margin: 10px 0; }
    .admin { background-color: #f8f8f8; padding: 10px; border-radius: 5px; }
</style>
</head>
<body>
<header>
    <h1>Fictief Wallet Systeem</h1>
</header>
<div class="container">
    <div id="homeSection">
        <h2>Selecteer een account</h2>
        <div id="accountList"></div>
        <button onclick="adminLoginPrompt()">Admin</button>
    </div>
    <div id="dashboard" class="hidden">
        <h2 id="welcomeMsg"></h2>
        <div id="balanceSection"></div>
        <div id="productList"></div>
        <div id="adminPanel" class="hidden"></div>
    </div>
</div>
<script>
let users = [];
let products = [];
let currentUser = null;
let adminPin = localStorage.getItem('adminPin') || '0000';function saveData() { localStorage.setItem('users', JSON.stringify(users)); localStorage.setItem('products', JSON.stringify(products)); localStorage.setItem('adminPin', adminPin); }

function loadData() { users = JSON.parse(localStorage.getItem('users')) || []; products = JSON.parse(localStorage.getItem('products')) || []; }

function renderAccountList() { const list = document.getElementById('accountList'); list.innerHTML = ''; users.forEach((u, i) => { const btn = document.createElement('button'); btn.textContent = u.name; btn.onclick = () => accountLogin(i); list.appendChild(btn); }); }

function accountLogin(index) { const pin = prompt('Voer pincode in voor ' + users[index].name); if (pin === users[index].pin) { currentUser = users[index]; document.getElementById('homeSection').classList.add('hidden'); document.getElementById('dashboard').classList.remove('hidden'); document.getElementById('welcomeMsg').innerText = Welkom, ${currentUser.name}; updateDashboard(); } else { alert('Ongeldige pincode'); } }

function adminLoginPrompt() { const pin = prompt('Voer admin pincode in:'); if (pin === adminPin) { currentUser = { name: 'Hoofdadmin', role: 'admin', balance: 0 }; document.getElementById('homeSection').classList.add('hidden'); document.getElementById('dashboard').classList.remove('hidden'); document.getElementById('welcomeMsg').innerText = Welkom, Hoofdadmin; updateDashboard(); } else { alert('Ongeldige admin pincode'); } }

function updateDashboard() { document.getElementById('balanceSection').innerHTML = <p>Saldo: €${(currentUser.balance || 0).toFixed(2)}</p>; renderProducts(); if (currentUser.role === 'admin' || currentUser.role === 'subadmin') { renderAdminPanel(); } }

function renderProducts() { const productList = document.getElementById('productList'); productList.innerHTML = '<h3>Producten</h3>'; products.forEach((p, i) => { productList.innerHTML += <div class="item">${p.name} - €${p.price.toFixed(2)} (${p.stock} op voorraad) <button onclick="buyProduct(${i})">Kopen</button></div>; }); }

function buyProduct(index) { const product = products[index]; if (product.stock > 0 && currentUser.balance >= product.price) { product.stock--; currentUser.balance -= product.price; saveData(); updateDashboard(); } else { alert('Onvoldoende saldo of geen voorraad'); } }

function renderAdminPanel() { const panel = document.getElementById('adminPanel'); panel.classList.remove('hidden'); panel.innerHTML = '<h3>Beheer</h3>'; panel.innerHTML += '<button onclick="addProduct()">Product toevoegen</button>'; panel.innerHTML += '<button onclick="addFunds()">Geld toevoegen aan account</button>'; panel.innerHTML += '<button onclick="addUser()">Account toevoegen</button>'; panel.innerHTML += '<button onclick="changeAdminPin()">Admin pincode wijzigen</button>'; }

function addUser() { const name = prompt('Naam:'); const pin = prompt('Pincode:'); const role = prompt('Rol (admin, subadmin, user):', 'user'); const balance = parseFloat(prompt('Startsaldo:', '0')) || 0; users.push({ name, pin, role, balance }); saveData(); renderAccountList(); }

function addProduct() { const name = prompt('Productnaam:'); const price = parseFloat(prompt('Prijs:')) || 0; const stock = parseInt(prompt('Voorraad:'), 10) || 0; products.push({ name, price, stock }); saveData(); updateDashboard(); }

function addFunds() { const target = prompt('Naam van account:'); const amount = parseFloat(prompt('Bedrag:')) || 0; const user = users.find(u => u.name === target); if (user) { user.balance += amount; saveData(); updateDashboard(); } }

function changeAdminPin() { const newPin = prompt('Nieuwe admin pincode:'); if (newPin) { adminPin = newPin; saveData(); alert('Admin pincode gewijzigd'); } }

loadData(); renderAccountList(); </script>

</body>
</html>