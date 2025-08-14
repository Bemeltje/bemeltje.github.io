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

<div class="container" id="homeScreen">
    <h2>Kies je account</h2>
    <div id="accountButtons"></div>
    <hr>
    <input type="password" id="adminCode" placeholder="Admin/Co-admin pincode" maxlength="4">
    <button onclick="adminLogin()">Inloggen</button>
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
    <h2 id="adminTitle">Admin Paneel</h2>
    <div id="adminSections"></div>
    <h3>Logboek</h3>
    <div id="logList"></div>
    <button onclick="clearLogs()" class="red">Logboek wissen</button>
    <hr>
    <button class="red" onclick="logout()">Uitloggen</button>
</div>

<script>
let adminPIN = "9999";
let coAdminPIN = "8888";
let isCoAdmin = false;

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

        card.innerHTML = `<strong>${acc.name}</strong>
            <span>Saldo: €${acc.saldo.toFixed(2)}</span>
            ${acc.type === "gast" ? `<span class="badge">gast</span>` : ""}`;
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

    let list = document.getElementById("productList");
    list.innerHTML = "";
    cart = {};

    products.forEach((p, i) => {
        cart[i] = 0;
        let voorraadClass = p.stock <= 5 ? "low-stock" : "";
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `${p.name} - €${p.price.toFixed(2)} (<span class="${voorraadClass}">voorraad: ${p.stock}</span>)
            <input type="number" min="0" max="${p.stock}" value="0" style="width:60px;" onchange="cart[${i}] = parseInt(this.value) || 0">`;
        list.appendChild(div);
    });

    let checkoutBtn = document.createElement("button");
    checkoutBtn.textContent = "🛒 Afrekenen";
    checkoutBtn.onclick = checkoutCart;
    list.appendChild(checkoutBtn);
}

// ... de rest van de functies blijven exact zoals in de vorige versie ...
</script>
</body>
</html>