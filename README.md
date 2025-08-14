<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Geld Systeem</title>
<style>
    body { font-family: Arial, sans-serif; background: #f7f7f7; margin: 0; padding: 0; }
    header { background: #333; color: white; padding: 10px; text-align: center; }
    .container { max-width: 500px; margin: 20px auto; background: white; padding: 20px; border-radius: 10px; }
    button { padding: 8px 12px; border: none; background: #28a745; color: white; border-radius: 5px; cursor: pointer; }
    button:hover { background: #218838; }
    input { padding: 8px; margin: 5px 0; width: 100%; }
    .hidden { display: none; }
    .item { border-bottom: 1px solid #ccc; padding: 5px 0; }
</style>
</head>
<body>

<header>
    <h1>Fictief Geld Systeem</h1>
</header>

<!-- Hoofdpagina met accounts -->
<div class="container" id="homeScreen">
    <h2>Kies je account</h2>
    <div id="accountButtons"></div>
    <hr>
    <input type="password" id="adminCode" placeholder="Admin pincode">
    <button onclick="adminLogin()">Admin inloggen</button>
</div>

<!-- Inlogscherm voor pincode -->
<div class="container hidden" id="pinScreen">
    <h2>Inloggen</h2>
    <p id="selectedUserName"></p>
    <input type="password" id="pincode" placeholder="Voer pincode in">
    <button onclick="checkLogin()">Inloggen</button>
    <button onclick="goHome()">Annuleren</button>
</div>

<!-- Gebruiker omgeving -->
<div class="container hidden" id="userScreen">
    <h2 id="welcome"></h2>
    <p>Saldo: €<span id="saldo"></span></p>
    <div id="productList"></div>
    <button onclick="logout()">Uitloggen</button>
</div>

<!-- Admin omgeving -->
<div class="container hidden" id="adminScreen">
    <h2>Admin Paneel</h2>
    <h3>Accounts</h3>
    <input id="newName" placeholder="Naam">
    <input id="newPin" placeholder="Pincode">
    <input type="number" id="newSaldo" placeholder="Startsaldo">
    <button onclick="addAccount()">Account toevoegen</button>
    <div id="accountList"></div>
    <h3>Producten</h3>
    <input id="prodName" placeholder="Productnaam">
    <input type="number" step="0.01" id="prodPrice" placeholder="Prijs">
    <button onclick="addProduct()">Product toevoegen</button>
    <div id="productAdminList"></div>
    <button onclick="logout()">Uitloggen</button>
</div>

<script>
let adminPIN = "9999";
let accounts = JSON.parse(localStorage.getItem("accounts")) || [
    {name: "Jan", pin: "1234", saldo: 10.00},
    {name: "Piet", pin: "5678", saldo: 5.00}
];
let products = JSON.parse(localStorage.getItem("products")) || [
    {name: "Chips", price: 0.75},
    {name: "Bier", price: 0.75},
    {name: "Cola", price: 1.00}
];
let currentUserIndex = null;

function saveData() {
    localStorage.setItem("accounts", JSON.stringify(accounts));
    localStorage.setItem("products", JSON.stringify(products));
}

function loadAccountButtons() {
    let container = document.getElementById("accountButtons");
    container.innerHTML = "";
    accounts.forEach((acc, i) => {
        let btn = document.createElement("button");
        btn.textContent = `${acc.name} (€${acc.saldo.toFixed(2)})`;
        btn.onclick = () => selectAccount(i);
        container.appendChild(btn);
        container.appendChild(document.createElement("br"));
    });
}
loadAccountButtons();

function selectAccount(index) {
    currentUserIndex = index;
    document.getElementById("homeScreen").classList.add("hidden");
    document.getElementById("pinScreen").classList.remove("hidden");
    document.getElementById("selectedUserName").textContent = "Account: " + accounts[index].name;
}

function checkLogin() {
    let pin = document.getElementById("pincode").value;
    if (accounts[currentUserIndex].pin === pin) {
        document.getElementById("pinScreen").classList.add("hidden");
        document.getElementById("userScreen").classList.remove("hidden");
        document.getElementById("welcome").textContent = "Welkom " + accounts[currentUserIndex].name;
        updateUserScreen();
    } else {
        alert("Verkeerde pincode!");
    }
}

function updateUserScreen() {
    let acc = accounts[currentUserIndex];
    document.getElementById("saldo").textContent = acc.saldo.toFixed(2);
    let list = document.getElementById("productList");
    list.innerHTML = "";
    products.forEach((p,i) => {
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `${p.name} - €${p.price.toFixed(2)} <button onclick="buyProduct(${i})">Koop</button>`;
        list.appendChild(div);
    });
}

function buyProduct(i) {
    let acc = accounts[currentUserIndex];
    if (acc.saldo >= products[i].price) {
        acc.saldo -= products[i].price;
        saveData();
        updateUserScreen();
        loadAccountButtons(); // update saldo op home
    } else {
        alert("Niet genoeg saldo!");
    }
}

function adminLogin() {
    let pin = document.getElementById("adminCode").value;
    if (pin === adminPIN) {
        document.getElementById("homeScreen").classList.add("hidden");
        document.getElementById("adminScreen").classList.remove("hidden");
        updateAdminScreen();
    } else {
        alert("Verkeerde admin pincode!");
    }
}

function updateAdminScreen() {
    let accList = document.getElementById("accountList");
    accList.innerHTML = "";
    accounts.forEach((acc,i) => {
        let div = document.createElement("div");
        div.innerHTML = `${acc.name} (€${acc.saldo.toFixed(2)}) 
            <button onclick="deleteAccount(${i})">X</button> 
            <button onclick="addSaldo(${i})">+€</button>`;
        accList.appendChild(div);
    });

    let prodList = document.getElementById("productAdminList");
    prodList.innerHTML = "";
    products.forEach((p,i) => {
        let div = document.createElement("div");
        div.innerHTML = `${p.name} (€${p.price.toFixed(2)}) <button onclick="deleteProduct(${i})">X</button>`;
        prodList.appendChild(div);
    });
}

function addAccount() {
    let name = document.getElementById("newName").value;
    let pin = document.getElementById("newPin").value;
    let saldo = parseFloat(document.getElementById("newSaldo").value);
    if (!name || !pin || isNaN(saldo)) { alert("Vul alle velden in!"); return; }
    accounts.push({name, pin, saldo});
    saveData();
    loadAccountButtons();
    updateAdminScreen();
}

function deleteAccount(i) {
    accounts.splice(i,1);
    saveData();
    loadAccountButtons();
    updateAdminScreen();
}

function addSaldo(i) {
    let bedrag = parseFloat(prompt("Bedrag toevoegen:"));
    if (!isNaN(bedrag)) {
        accounts[i].saldo += bedrag;
        saveData();
        updateAdminScreen();
        loadAccountButtons();
    }
}

function addProduct() {
    let name = document.getElementById("prodName").value;
    let price = parseFloat(document.getElementById("prodPrice").value);
    if (!name || isNaN(price)) { alert("Vul alle velden in!"); return; }
    products.push({name, price});
    saveData();
    updateAdminScreen();
}

function deleteProduct(i) {
    products.splice(i,1);
    saveData();
    updateAdminScreen();
}

function logout() {
    currentUserIndex = null;
    document.getElementById("userScreen").classList.add("hidden");
    document.getElementById("adminScreen").classList.add("hidden");
    document.getElementById("homeScreen").classList.remove("hidden");
}

function goHome() {
    document.getElementById("pinScreen").classList.add("hidden");
    document.getElementById("homeScreen").classList.remove("hidden");
}
</script>

</body>
</html>