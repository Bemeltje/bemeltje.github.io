<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Geld Systeem</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background: linear-gradient(135deg, #f0f4f8, #d9e2ec);
        margin: 0;
        padding: 0;
        color: #333;
    }
    header {
        background: #4a90e2;
        color: white;
        padding: 15px;
        text-align: center;
        font-size: 1.4em;
        box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }
    .container {
        max-width: 500px;
        margin: 20px auto;
        background: white;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }
    h2 { margin-top: 0; }
    button {
        padding: 10px 15px;
        border: none;
        background: #4a90e2;
        color: white;
        border-radius: 5px;
        cursor: pointer;
        margin: 5px 0;
        font-size: 1em;
        transition: background 0.3s;
    }
    button:hover { background: #3b7dc4; }
    button.red { background: #e94e4e; }
    button.red:hover { background: #c43d3d; }
    input, select {
        padding: 10px;
        margin: 5px 0;
        width: 100%;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 1em;
    }
    .hidden { display: none; }
    .item {
        border-bottom: 1px solid #eee;
        padding: 8px 0;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .item:last-child { border-bottom: none; }
</style>
</head>
<body>

<header>
    💰 Fictief Geld Systeem
</header>

<!-- Hoofdpagina met accounts -->
<div class="container" id="homeScreen">
    <h2>Kies je account</h2>
    <div id="accountButtons"></div>
    <hr>
    <input type="password" id="adminCode" placeholder="Admin pincode" maxlength="4">
    <button onclick="adminLogin()">Admin inloggen</button>
</div>

<!-- Inlogscherm voor pincode -->
<div class="container hidden" id="pinScreen">
    <h2>Inloggen</h2>
    <p id="selectedUserName"></p>
    <input type="password" id="pincode" placeholder="Voer pincode in" maxlength="4">
    <button onclick="checkLogin()">Inloggen</button>
    <button class="red" onclick="goHome()">Annuleren</button>
</div>

<!-- Gebruiker omgeving -->
<div class="container hidden" id="userScreen">
    <h2 id="welcome"></h2>
    <p>Saldo: €<span id="saldo"></span></p>
    <div id="productList"></div>
    <hr>
    <button class="red" onclick="logout()">Uitloggen</button>
</div>

<!-- Admin omgeving -->
<div class="container hidden" id="adminScreen">
    <h2>Admin Paneel</h2>
    <h3>Accounts</h3>
    <input id="newName" placeholder="Naam">
    <input id="newPin" placeholder="Pincode" maxlength="4">
    <input type="number" id="newSaldo" placeholder="Startsaldo">
    <select id="newType">
        <option value="gast">Gast</option>
        <option value="vast">Vast</option>
    </select>
    <button onclick="addAccount()">Account toevoegen</button>
    <div id="accountList"></div>
    <h3>Producten</h3>
    <input id="prodName" placeholder="Productnaam">
    <input type="number" step="0.01" id="prodPrice" placeholder="Prijs">
    <button onclick="addProduct()">Product toevoegen</button>
    <div id="productAdminList"></div>
    <hr>
    <button class="red" onclick="logout()">Uitloggen</button>
</div>

<script>
let adminPIN = "9999";
let accounts = JSON.parse(localStorage.getItem("accounts")) || [
    {name: "Jan", pin: "1234", saldo: 10.00, type: "vast"},
    {name: "Piet", pin: "5678", saldo: 5.00, type: "gast"}
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
        btn.style.width = "100%";
        btn.textContent = `${acc.name} (€${acc.saldo.toFixed(2)}) [${acc.type}]`;
        if (acc.saldo < 0) {
            btn.style.background = "#e94e4e";
        }
        btn.onclick = () => selectAccount(i);
        container.appendChild(btn);
    });
}
loadAccountButtons();

function selectAccount(index) {
    currentUserIndex = index;
    document.getElementById("pincode").value = "";
    document.getElementById("homeScreen").classList.add("hidden");
    document.getElementById("pinScreen").classList.remove("hidden");
    document.getElementById("selectedUserName").textContent = "Account: " + accounts[index].name;
}

function checkLogin() {
    let pin = document.getElementById("pincode").value;
    if (accounts[currentUserIndex].pin === pin) {
        document.getElementById("pincode").value = "";
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
    products.forEach((p, i) => {
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `<span>${p.name} - €${p.price.toFixed(2)}</span> <button onclick="buyProduct(${i})">Koop</button>`;
        list.appendChild(div);
    });
}

function buyProduct(i) {
    let acc = accounts[currentUserIndex];
    let prijs = products[i].price;

    if (!confirm(`Weet je zeker dat je '${products[i].name}' wilt kopen voor €${prijs.toFixed(2)}?`)) return;

    if (acc.type === "gast" && acc.saldo - prijs < 0) {
        alert("Gast mag niet onder €0 komen!");
        return;
    }
    if (acc.type === "vast" && acc.saldo - prijs < -10) {
        alert("Vast mag niet verder dan -€10 komen!");
        return;
    }

    acc.saldo -= prijs;
    saveData();
    updateUserScreen();
    loadAccountButtons(); 
}

function adminLogin() {
    let pin = document.getElementById("adminCode").value;
    if (pin === adminPIN) {
        document.getElementById("adminCode").value = "";
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
    accounts.forEach((acc, i) => {
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `<span>${acc.name} (€${acc.saldo.toFixed(2)}) [${acc.type}]</span> 
            <span>
                <button onclick="deleteAccount(${i})">X</button> 
                <button onclick="addSaldo(${i})">+€</button>
            </span>`;
        accList.appendChild(div);
    });

    let prodList = document.getElementById("productAdminList");
    prodList.innerHTML = "";
    products.forEach((p, i) => {
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `<span>${p.name} (€${p.price.toFixed(2)})</span> 
            <button onclick="deleteProduct(${i})">X</button>`;
        prodList.appendChild(div);
    });
}

function addAccount() {
    let name = document.getElementById("newName").value;
    let pin = document.getElementById("newPin").value;
    let saldo = parseFloat(document.getElementById("newSaldo").value);
    let type = document.getElementById("newType").value;
    if (!name || !pin || isNaN(saldo)) { alert("Vul alle velden in!"); return; }
    if (pin.length > 4) { alert("Pincode mag maximaal 4 cijfers zijn!"); return; }
    accounts.push({name, pin, saldo, type});
    saveData();
    loadAccountButtons();
    updateAdminScreen();
}

function deleteAccount(i) {
    accounts.splice(i, 1);
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
    products.splice(i, 1);
    saveData();
    updateAdminScreen();
}

function logout() {
    currentUserIndex = null;
    document.getElementById("userScreen").classList.add("hidden");
    document.getElementById("adminScreen").classList.add("hidden");
    document.getElementById("pinScreen").classList.add("hidden");
    document.getElementById("homeScreen").classList.remove("hidden");
}

function goHome() {
    document.getElementById("pincode").value = "";
    document.getElementById("pinScreen").classList.add("hidden");
    document.getElementById("homeScreen").classList.remove("hidden");
}
</script>

</body>
</html>