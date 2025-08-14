<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Geld Systeem</title>
<style>
body {
    font-family: 'Segoe UI', Arial, sans-serif;
    background: linear-gradient(135deg, #f7f9fb, #e4ebf1);
    margin: 0;
    padding: 0;
    color: #333;
}

header {
    background: #2b7cd3;
    color: white;
    padding: 15px;
    text-align: center;
    font-size: 1.6em;
    box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    letter-spacing: 1px;
}

.container {
    max-width: 1200px;
    margin: 20px auto;
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

h2, h3 { margin-top: 0; }

button {
    padding: 8px 14px;
    border: none;
    background: #2b7cd3;
    color: white;
    border-radius: 6px;
    cursor: pointer;
    margin: 3px;
    font-size: 0.95em;
    transition: background 0.3s, transform 0.1s;
}
button:hover { background: #1f65ac; transform: scale(1.03); }
button.red { background: #d64545; }
button.red:hover { background: #b53333; }

input, select {
    padding: 9px;
    margin: 5px 0;
    width: 100%;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 0.95em;
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

.badge {
    background: #4a90e2;
    color: white;
    padding: 2px 6px;
    font-size: 0.8em;
    border-radius: 4px;
    margin-left: 5px;
}
.badge.gast { background: #f39c12; }
.badge.vast { background: #27ae60; }

/* Account grid */
#accountButtons {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 12px;
}
.account-card {
    background: white;
    border-radius: 8px;
    padding: 12px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.05);
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    border-left: 5px solid transparent;
    transition: transform 0.1s;
}
.account-card:hover { transform: translateY(-2px); }
.account-card.red { border-left-color: #d64545; }
.account-card.green { border-left-color: #27ae60; }
.account-card.orange { border-left-color: #f39c12; }
.account-card strong { font-size: 1.1em; }

/* Tabellen netjes */
table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.9em;
    margin-top: 10px;
}
th, td {
    border: 1px solid #ddd;
    padding: 6px;
    text-align: left;
}
th { background: #f4f4f4; }
</style>
</head>
<body>

<header>💰 Fictief Geld Systeem</header>

<div class="container" id="homeScreen">
    <h2>Kies je account</h2>
    <div id="accountButtons"></div>
    <hr>
    <input type="password" id="adminCode" placeholder="Admin pincode" maxlength="4">
    <button onclick="adminLogin()">Admin inloggen</button>
</div>

<div class="container hidden" id="pinScreen">
    <h2>Inloggen</h2>
    <p id="selectedUserName"></p>
    <input type="password" id="pincode" placeholder="Voer pincode in" maxlength="4">
    <button onclick="checkLogin()">Inloggen</button>
    <button class="red" onclick="goHome()">Annuleren</button>
</div>

<div class="container hidden" id="userScreen">
    <h2 id="welcome"></h2>
    <p>Saldo: €<span id="saldo"></span></p>
    <div id="productList"></div>
    <hr>
    <button class="red" onclick="logout()">Uitloggen</button>
</div>

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

    <h3>Logboek</h3>
    <div id="logList"></div>
    <button onclick="clearLogs()" class="red">Logboek wissen</button>

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
let logs = JSON.parse(localStorage.getItem("logs")) || [];
let currentUserIndex = null;
let cart = {};

function saveData() {
    localStorage.setItem("accounts", JSON.stringify(accounts));
    localStorage.setItem("products", JSON.stringify(products));
    localStorage.setItem("logs", JSON.stringify(logs));
}

function loadAccountButtons() {
    let container = document.getElementById("accountButtons");
    container.innerHTML = "";
    accounts.forEach((acc, i) => {
        let card = document.createElement("div");
        card.classList.add("account-card");
        if (acc.saldo >= 0) card.classList.add("green");
        else if (acc.type === "vast" && acc.saldo >= -5) card.classList.add("orange");
        else card.classList.add("red");

        card.innerHTML = `
            <strong>${acc.name}</strong>
            <span>Saldo: €${acc.saldo.toFixed(2)}</span>
            <span class="badge ${acc.type}">${acc.type}</span>
        `;
        card.onclick = () => selectAccount(i);
        container.appendChild(card);
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
    let saldoElement = document.getElementById("saldo");
    saldoElement.textContent = acc.saldo.toFixed(2);
    if (acc.saldo >= 0) saldoElement.style.color = "green";
    else if (acc.type === "vast" && acc.saldo >= -5) saldoElement.style.color = "orange";
    else saldoElement.style.color = "red";

    if (acc.type === "gast" && acc.saldo < 1) alert("Let op! Je saldo is bijna op.");
    if (acc.type === "vast" && acc.saldo < -8) alert("Let op! Je zit bijna aan je limiet van -€10.");

    let list = document.getElementById("productList");
    list.innerHTML = "";
    cart = {};

    products.forEach((p, i) => {
        cart[i] = 0;
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `
            <span>${p.name} - €${p.price.toFixed(2)}</span> 
            <input type="number" min="0" value="0" style="width:60px;" onchange="cart[${i}] = parseInt(this.value) || 0">
        `;
        list.appendChild(div);
    });

    let checkoutBtn = document.createElement("button");
    checkoutBtn.textContent = "🛒 Afrekenen";
    checkoutBtn.onclick = checkoutCart;
    list.appendChild(checkoutBtn);
}

function checkoutCart() {
    let acc = accounts[currentUserIndex];
    let totaal = 0;
    let aankoopDetails = [];

    Object.keys(cart).forEach(i => {
        if (cart[i] > 0) {
            let prijs = products[i].price * cart[i];
            totaal += prijs;
            aankoopDetails.push({product: products[i].name, aantal: cart[i], prijs: products[i].price});
        }
    });

    if (aankoopDetails.length === 0) return alert("Je hebt niets geselecteerd.");
    if (!confirm(`Je wilt ${aankoopDetails.length} producten kopen voor totaal €${totaal.toFixed(2)}. Doorgaan?`)) return;
    if (acc.type === "gast" && acc.saldo - totaal < 0) return alert("Gast mag niet onder €0 komen!");
    if (acc.type === "vast" && acc.saldo - totaal < -10) return alert("Vast mag niet verder dan -€10 komen!");

    acc.saldo -= totaal;
    aankoopDetails.forEach(item => {
        logs.push({
            gebruiker: acc.name,
            product: `${item.product} (x${item.aantal})`,
            prijs: item.prijs * item.aantal,
            tijd: new Date().toLocaleString()
        });
    });

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
    } else alert("Verkeerde admin pincode!");
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

    let logTable = `<table><tr><th>Gebruiker</th><th>Product</th><th>Prijs</th><th>Tijd</th></tr>`;
    logs.forEach(log => {
        logTable += `<tr>
                        <td>${log.gebruiker}</td>
                        <td>${log.product}</td>
                        <td>€${log.prijs.toFixed(2)}</td>
                        <td>${log.tijd}</td>
                     </tr>`;
    });
    logTable += "</table>";
    document.getElementById("logList").innerHTML = logTable + 
        `<br><button onclick="exportLogsToCSV()">📥 Exporteer naar CSV</button>`;
}

function exportLogsToCSV() {
    if (logs.length === 0) return alert("Het logboek is leeg.");
    let csvContent = "data:text/csv;charset=utf-8,"
        + ["Gebruiker,Product,Prijs,Tijd"].join(",") + "\n"
        + logs.map(l => `${l.gebruiker},${l.product},${l.prijs.toFixed(2)},${l.tijd}`).join("\n");
    let encodedUri = encodeURI(csvContent);
    let link = document.createElement("a");
    link.setAttribute("href", encodedUri);
    link.setAttribute("download", "logboek.csv");
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

function addAccount() {
    let name = document.getElementById("newName").value;
    let pin = document.getElementById("newPin").value;
    let saldo = parseFloat(document.getElementById("newSaldo").value);
    let type = document.getElementById("newType").value;
    if (!name || !pin || isNaN(saldo)) return alert("Vul alle velden in!");
    if (pin.length > 4) return alert("Pincode mag maximaal 4 cijfers zijn!");
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
    if (!name || isNaN(price)) return alert("Vul alle velden in!");
    products.push({name, price});
    saveData();
    updateAdminScreen();
}

function deleteProduct(i) {
    products.splice(i, 1);
    saveData();
    updateAdminScreen();
}

function clearLogs() {
    if (confirm("Weet je zeker dat je het logboek wilt wissen?")) {
        logs = [];
        saveData();
        updateAdminScreen();
    }
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