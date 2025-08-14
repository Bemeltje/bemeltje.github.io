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
    background: #f39c12;
    color: white;
    padding: 2px 6px;
    font-size: 0.8em;
    border-radius: 4px;
    margin-left: 5px;
}
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
.low-stock { color: red; font-weight: bold; }
</style>
</head>
<body>

<header>💰 Fictief Geld Systeem</header>

<!-- Home / accountkeuze + admin/co-admin login -->
<div class="container" id="homeScreen">
    <h2>Kies je account</h2>
    <div id="accountButtons"></div>
    <hr>
    <input type="password" id="adminCode" placeholder="Admin/Co-admin pincode" maxlength="4" inputmode="numeric">
    <button onclick="adminLogin()">Inloggen</button>
</div>

<!-- Pincode voor gebruiker -->
<div class="container hidden" id="pinScreen">
    <h2>Inloggen</h2>
    <p id="selectedUserName"></p>
    <input type="password" id="pincode" placeholder="Voer pincode in" maxlength="4" inputmode="numeric">
    <button onclick="checkLogin()">Inloggen</button>
    <button class="red" onclick="goHome()">Annuleren</button>
</div>

<!-- Gebruikersscherm -->
<div class="container hidden" id="userScreen">
    <h2 id="welcome"></h2>
    <p>Saldo: €<span id="saldo"></span></p>
    <div id="productList"></div>
    <hr>
    <button class="red" onclick="logout()">Uitloggen</button>
</div>

<!-- Admin/Co-admin -->
<div class="container hidden" id="adminScreen">
    <h2 id="adminTitle">Admin Paneel</h2>
    <div id="adminSections"></div>

    <h3>Logboek</h3>
    <div id="logList"></div>
    <div>
        <button onclick="exportLogsToCSV()">📥 Exporteer naar CSV</button>
        <button onclick="clearLogs()" class="red">Logboek wissen</button>
    </div>

    <hr>
    <button class="red" onclick="logout()">Uitloggen</button>
</div>

<script>
/* ---- Config ---- */
let adminPIN = "9999";
let coAdminPIN = "8888";
let isCoAdmin = false;

/* ---- Data ---- */
let accounts = JSON.parse(localStorage.getItem("accounts")) || [
    {name: "Jan", pin: "1234", saldo: 10.00, type: "vast"},
    {name: "Piet", pin: "5678", saldo: 5.00, type: "gast"}
];
let products = JSON.parse(localStorage.getItem("products")) || [
    {name: "Chips", price: 0.75, stock: 20},
    {name: "Bier", price: 0.75, stock: 30},
    {name: "Cola", price: 1.00, stock: 15}
];
let logs = JSON.parse(localStorage.getItem("logs")) || [];

let currentUserIndex = null;
let cart = {};

/* ---- Helpers ---- */
function saveData() {
    localStorage.setItem("accounts", JSON.stringify(accounts));
    localStorage.setItem("products", JSON.stringify(products));
    localStorage.setItem("logs", JSON.stringify(logs));
}

function formatPrice(n) {
    return Number(n).toFixed(2);
}

/* ---- Home/accounts ---- */
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
            <span>Saldo: €${formatPrice(acc.saldo)}</span>
            ${acc.type === "gast" ? `<span class="badge">gast</span>` : ""}
        `;
        card.onclick = () => selectAccount(i);
        container.appendChild(card);
    });
}
loadAccountButtons();

/* ---- Gebruiker login ---- */
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

/* ---- Gebruiker scherm / winkelwagen ---- */
function updateUserScreen() {
    let acc = accounts[currentUserIndex];
    let saldoElement = document.getElementById("saldo");
    saldoElement.textContent = formatPrice(acc.saldo);
    if (acc.saldo >= 0) saldoElement.style.color = "green";
    else if (acc.type === "vast" && acc.saldo >= -5) saldoElement.style.color = "orange";
    else saldoElement.style.color = "red";

    let list = document.getElementById("productList");
    list.innerHTML = "";
    cart = {};

    products.forEach((p, i) => {
        cart[i] = 0;
        let voorraadClass = p.stock <= 5 ? "low-stock" : "";
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `
            <span>${p.name} - €${formatPrice(p.price)} (<span class="${voorraadClass}">voorraad: ${p.stock}</span>)</span>
            <input type="number" min="0" max="${p.stock}" value="0" style="width:70px;" 
                   onchange="cart[${i}] = Math.max(0, Math.min(${p.stock}, parseInt(this.value)||0))">
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
        let qty = cart[i] || 0;
        if (qty > 0) {
            if (qty > products[i].stock) {
                alert(`Niet genoeg voorraad voor ${products[i].name}`);
                return;
            }
            let prijs = products[i].price * qty;
            totaal += prijs;
            aankoopDetails.push({idx: Number(i), product: products[i].name, aantal: qty, prijs: products[i].price});
        }
    });

    if (aankoopDetails.length === 0) { alert("Je hebt niets geselecteerd."); return; }
    if (!confirm(`Totaal: €${formatPrice(totaal)}. Doorgaan?`)) return;

    // Limieten
    if (acc.type === "gast" && acc.saldo - totaal < 0) { alert("Gast mag niet onder €0 komen!"); return; }
    if (acc.type === "vast" && acc.saldo - totaal < -10) { alert("Vast mag niet verder dan -€10 komen!"); return; }

    // Afrekenen
    acc.saldo -= totaal;
    aankoopDetails.forEach(item => {
        products[item.idx].stock -= item.aantal;
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

/* ---- Admin / Co-admin login ---- */
function adminLogin() {
    let pin = document.getElementById("adminCode").value;
    if (pin === adminPIN) {
        isCoAdmin = false;
    } else if (pin === coAdminPIN) {
        isCoAdmin = true;
    } else {
        alert("Verkeerde pincode!");
        return;
    }
    document.getElementById("adminCode").value = "";
    document.getElementById("homeScreen").classList.add("hidden");
    document.getElementById("adminScreen").classList.remove("hidden");
    document.getElementById("adminTitle").textContent = isCoAdmin ? "Co-Admin Paneel" : "Admin Paneel";
    updateAdminScreen();
}

/* ---- Admin / Co-admin scherm ---- */
function updateAdminScreen() {
    let sec = document.getElementById("adminSections");
    sec.innerHTML = "";

    // Accounts (altijd zichtbaar; toevoegen alleen voor admin)
    let accDiv = document.createElement("div");
    accDiv.innerHTML = `
        <h3>Accounts</h3>
        <input id="newName" placeholder="Naam" ${isCoAdmin ? 'disabled' : ''}>
        <input id="newPin" placeholder="Pincode" maxlength="4" inputmode="numeric" ${isCoAdmin ? 'disabled' : ''}>
        <input type="number" id="newSaldo" placeholder="Startsaldo" ${isCoAdmin ? 'disabled' : ''}>
        <select id="newType" ${isCoAdmin ? 'disabled' : ''}>
            <option value="gast">Gast</option>
            <option value="vast">Vast</option>
        </select>
        ${isCoAdmin ? '' : '<button onclick="addAccount()">Account toevoegen</button>'}
        <div id="accountList"></div>`;
    sec.appendChild(accDiv);

    // Productbeheer (alleen admin)
    if (!isCoAdmin) {
        let prodDiv = document.createElement("div");
        prodDiv.innerHTML = `
            <h3>Producten</h3>
            <input id="prodName" placeholder="Productnaam">
            <input type="number" step="0.01" id="prodPrice" placeholder="Prijs">
            <input type="number" id="prodStock" placeholder="Voorraad">
            <button onclick="addProduct()">Product toevoegen</button>
            <div id="productAdminList"></div>`;
        sec.appendChild(prodDiv);
    }

    // Accounts tabel
    let accList = document.getElementById("accountList");
    accList.innerHTML = "";
    accounts.forEach((acc, i) => {
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `
            <span>${acc.name} (€${formatPrice(acc.saldo)}) ${acc.type === "gast" ? "[gast]" : ""}</span>
            <span>
                <button onclick="deleteAccount(${i})" ${isCoAdmin ? 'disabled' : ''}>X</button>
                <button onclick="addSaldo(${i})">+€</button>
            </span>`;
        accList.appendChild(div);
    });

    // Producten tabel (alleen admin)
    if (!isCoAdmin) {
        let prodList = document.getElementById("productAdminList");
        prodList.innerHTML = "";
        products.forEach((p, i) => {
            let voorraadClass = p.stock <= 5 ? "low-stock" : "";
            let div = document.createElement("div");
            div.classList.add("item");
            div.innerHTML = `
                <span>${p.name} (€${formatPrice(p.price)}) - <span class="${voorraadClass}">Voorraad: ${p.stock}</span></span>
                <span>
                    <button onclick="deleteProduct(${i})">X</button>
                </span>`;
            prodList.appendChild(div);
        });
    }

    // Logboek tabel + knoppen (knoppen staan al buiten sec)
    let logTable = `<table><tr><th>Gebruiker</th><th>Product</th><th>Prijs</th><th>Tijd</th></tr>`;
    logs.forEach(log => {
        logTable += `<tr><td>${log.gebruiker}</td><td>${log.product}</td><td>€${formatPrice(log.prijs)}</td><td>${log.tijd}</td></tr>`;
    });
    logTable += "</table>";
    document.getElementById("logList").innerHTML = logTable;
}

/* ---- Admin acties ---- */
function addAccount() {
    let name = document.getElementById("newName").value.trim();
    let pin = document.getElementById("newPin").value.trim();
    let saldo = parseFloat(document.getElementById("newSaldo").value);
    let type = document.getElementById("newType").value;
    if (!name || !pin || isNaN(saldo)) { alert("Vul alle velden in!"); return; }
    if (pin.length > 4) { alert("Pincode max 4 cijfers!"); return; }
    accounts.push({name, pin, saldo, type});
    saveData();
    loadAccountButtons();
    updateAdminScreen();
    // velden leegmaken
    document.getElementById("newName").value = "";
    document.getElementById("newPin").value = "";
    document.getElementById("newSaldo").value = "";
    document.getElementById("newType").value = "gast";
}

function deleteAccount(i) {
    if (isCoAdmin) return; // safety
    accounts.splice(i, 1);
    saveData();
    loadAccountButtons();
    updateAdminScreen();
}

function addSaldo(i) {
    let bedrag = parseFloat(prompt("Bedrag toevoegen:"));
    if (isNaN(bedrag)) return;
    accounts[i].saldo += bedrag;
    saveData();
    updateAdminScreen();
    loadAccountButtons();
}

function addProduct() {
    if (isCoAdmin) return; // safety
    let name = document.getElementById("prodName").value.trim();
    let price = parseFloat(document.getElementById("prodPrice").value);
    let stock = parseInt(document.getElementById("prodStock").value);
    if (!name || isNaN(price) || isNaN(stock)) { alert("Vul alle velden in!"); return; }
    products.push({name, price, stock});
    saveData();
    updateAdminScreen();
    document.getElementById("prodName").value = "";
    document.getElementById("prodPrice").value = "";
    document.getElementById("prodStock").value = "";
}

function deleteProduct(i) {
    if (isCoAdmin) return; // safety
    products.splice(i, 1);
    saveData();
    updateAdminScreen();
}

/* ---- Logboek acties ---- */
function exportLogsToCSV() {
    if (logs.length === 0) { alert("Het logboek is leeg."); return; }
    let csvContent = "data:text/csv;charset=utf-8,"
        + ["Gebruiker,Product,Prijs,Tijd"].join(",") + "\n"
        + logs.map(l => `${l.gebruiker},${l.product},${formatPrice(l.prijs)},${l.tijd}`).join("\n");
    let encodedUri = encodeURI(csvContent);
    let link = document.createElement("a");
    link.setAttribute("href", encodedUri);
    link.setAttribute("download", "logboek.csv");
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

function clearLogs() {
    if (!confirm("Weet je zeker dat je het logboek wilt wissen?")) return;
    logs = [];
    saveData();
    updateAdminScreen();
}

/* ---- Navigatie ---- */
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