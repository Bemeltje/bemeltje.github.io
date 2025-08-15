<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Fictief Geld Systeem</title>
<style>
body { font-family: 'Segoe UI', Arial, sans-serif; background: linear-gradient(135deg,#f7f9fb,#e4ebf1); margin:0; color:#333; }
header { background:#2b7cd3; color:#fff; padding:15px; text-align:center; font-size:1.6em; box-shadow:0 2px 6px rgba(0,0,0,.2); letter-spacing:1px; }
.container { max-width:1200px; margin:20px auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 4px 12px rgba(0,0,0,.08); }
h2,h3 { margin-top:0; }
button { padding:8px 14px; border:0; background:#2b7cd3; color:#fff; border-radius:6px; cursor:pointer; margin:3px; font-size:.95em; transition:background .3s, transform .1s; }
button:hover { background:#1f65ac; transform:scale(1.03); }
button.red { background:#d64545; } button.red:hover{ background:#b53333; }
button.ghost { background:#eef4fb; color:#1f65ac; }
input,select { padding:9px; margin:5px 0; width:100%; border:1px solid #ccc; border-radius:5px; font-size:.95em; }
.hidden { display:none; }
.item { border-bottom:1px solid #eee; padding:8px 0; display:flex; justify-content:space-between; align-items:center; gap:10px; flex-wrap:wrap; }
.item:last-child{ border-bottom:none; }
.badge { background:#f39c12; color:#fff; padding:2px 6px; font-size:.8em; border-radius:4px; margin-left:5px; }
#accountButtons { display:grid; grid-template-columns:repeat(auto-fill, minmax(250px,1fr)); gap:12px; }
.account-card { background:#fff; border-radius:8px; padding:12px; box-shadow:0 2px 8px rgba(0,0,0,.05); display:flex; flex-direction:column; align-items:flex-start; border-left:5px solid transparent; transition:transform .1s; }
.account-card:hover{ transform:translateY(-2px); }
.account-card.green{ border-left-color:#27ae60; }
.account-card.orange{ border-left-color:#f39c12; }
.account-card.red{ border-left-color:#d64545; }
.account-card strong{ font-size:1.1em; }
table { width:100%; border-collapse:collapse; font-size:.9em; margin-top:10px; }
th,td { border:1px solid #ddd; padding:6px; text-align:left; }
th{ background:#f4f4f4; }
.low-stock{ color:red; font-weight:bold; }
.small{ font-size:.85em; color:#666; }
.cart-summary { margin-top:10px; padding:10px; background:#f7fbff; border:1px solid #dfeaf7; border-radius:8px; display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:8px; }
.cart-summary strong { font-size:1.05em; }
</style>
</head>
<body>
<header>💰 Fictief Geld Systeem</header>

<!-- HOME -->
<div class="container" id="homeScreen">
  <h2>Kies je account</h2>
  <div id="accountButtons"></div>
  <hr>
  <input type="password" id="adminCode" placeholder="Admin/Co-admin pincode" maxlength="4" inputmode="numeric">
  <button id="adminLoginBtn">Inloggen</button>
  <p class="small">Zie je geen accounts? Log in als admin en kies "Herstel naar standaard".</p>
</div>

<!-- PIN -->
<div class="container hidden" id="pinScreen">
  <h2>Inloggen</h2>
  <p id="selectedUserName"></p>
  <input type="password" id="pincode" placeholder="Voer pincode in" maxlength="4" inputmode="numeric">
  <button id="userLoginBtn">Inloggen</button>
  <button class="red" id="cancelPinBtn">Annuleren</button>
</div>

<!-- USER -->
<div class="container hidden" id="userScreen">
  <h2 id="welcome"></h2>
  <p>Saldo: €<span id="saldo"></span></p>
  <div id="productList"></div>
  <div id="cartSummary" class="cart-summary">
    <span><strong>Totaal:</strong> €<span id="cartTotal">0.00</span> — <span id="cartItems">0</span> item(s)</span>
    <span>
      <button id="checkoutBtn" disabled>🛒 Afrekenen</button>
      <button class="ghost" id="clearCartBtn">Winkelwagen leegmaken</button>
    </span>
  </div>
  <hr>
  <button class="red" id="logoutUserBtn">Uitloggen</button>
</div>

<!-- ADMIN -->
<div class="container hidden" id="adminScreen">
  <h2 id="adminTitle">Admin Paneel</h2>
  <div id="adminSections"></div>

  <h3>Logboek</h3>
  <div id="logList"></div>
  <div id="logActions" class="item">
    <span class="small">Logboek acties</span>
    <span>
      <button id="exportCsvBtn">📥 Exporteer naar CSV</button>
      <button class="red" id="clearLogsBtn">Logboek wissen</button>
    </span>
  </div>

  <div id="dataBeheer" class="hidden">
    <h3>Data beheer</h3>
    <div class="item">
      <span>Volledige data (accounts, producten, logs)</span>
      <span>
        <button id="exportJsonBtn">⬇️ Exporteer JSON</button>
        <label style="margin-left:6px;">
          <input type="file" id="importFile" accept=".json" class="hidden">
          <button id="importJsonBtn">⬆️ Importeer JSON</button>
        </label>
        <button class="red" id="resetBtn">Herstel naar standaard</button>
      </span>
    </div>
  </div>

  <hr>
  <button class="red" id="logoutAdminBtn">Uitloggen</button>
</div>

<script>
document.addEventListener('DOMContentLoaded', () => {
  /* ---- Config ---- */
  const APP_VERSION = "2025-08-15";
  const ADMIN_PIN = "9999";
  const COADMIN_PIN = "8888";
  let isCoAdmin = false;

  /* ---- State ---- */
  let accounts = safeGet('accounts', [
    {name:"Jan", pin:"1234", saldo:10.00, type:"vast"},
    {name:"Piet", pin:"5678", saldo:5.00, type:"gast"}
  ]);
  let products = safeGet('products', [
    {name:"Chips", price:0.75, stock:20},
    {name:"Bier",  price:0.75, stock:30},
    {name:"Cola",  price:1.00, stock:15}
  ]);
  let logs = safeGet('logs', []);
  let currentUserIndex = null;
  let cart = {}; // {productIndex: qty}

  /* ---- Elements ---- */
  const homeScreen = document.getElementById('homeScreen');
  const pinScreen = document.getElementById('pinScreen');
  const userScreen = document.getElementById('userScreen');
  const adminScreen = document.getElementById('adminScreen');

  const accountButtons = document.getElementById('accountButtons');
  const adminCode = document.getElementById('adminCode');
  const adminLoginBtn = document.getElementById('adminLoginBtn');

  const selectedUserName = document.getElementById('selectedUserName');
  const pincode = document.getElementById('pincode');
  const userLoginBtn = document.getElementById('userLoginBtn');
  const cancelPinBtn = document.getElementById('cancelPinBtn');

  const welcome = document.getElementById('welcome');
  const saldoEl = document.getElementById('saldo');
  const productList = document.getElementById('productList');
  const checkoutBtn = document.getElementById('checkoutBtn');
  const clearCartBtn = document.getElementById('clearCartBtn');
  const cartTotalEl = document.getElementById('cartTotal');
  const cartItemsEl = document.getElementById('cartItems');
  const logoutUserBtn = document.getElementById('logoutUserBtn');

  const adminTitle = document.getElementById('adminTitle');
  const adminSections = document.getElementById('adminSections');
  const logList = document.getElementById('logList');
  const exportCsvBtn = document.getElementById('exportCsvBtn');
  const clearLogsBtn = document.getElementById('clearLogsBtn');
  const dataBeheer = document.getElementById('dataBeheer');
  const exportJsonBtn = document.getElementById('exportJsonBtn');
  const importJsonBtn = document.getElementById('importJsonBtn');
  const importFile = document.getElementById('importFile');
  const resetBtn = document.getElementById('resetBtn');
  const logoutAdminBtn = document.getElementById('logoutAdminBtn');
  const logActions = document.getElementById('logActions');

  /* ---- Utils ---- */
  function saveAll() {
    localStorage.setItem('accounts', JSON.stringify(accounts));
    localStorage.setItem('products', JSON.stringify(products));
    localStorage.setItem('logs', JSON.stringify(logs));
    localStorage.setItem('appVersion', APP_VERSION);
  }
  function safeGet(key, fallback){
    try {
      const raw = localStorage.getItem(key);
      if(!raw) return fallback;
      const val = JSON.parse(raw);
      if (key==='accounts' && !Array.isArray(val)) return fallback;
      if (key==='products' && !Array.isArray(val)) return fallback;
      if (key==='logs' && !Array.isArray(val)) return fallback;
      return val;
    } catch(e){ return fallback; }
  }
  function formatPrice(n){ return Number(n).toFixed(2); }
  function digitsOnly(el){ el.value = el.value.replace(/\D+/g,'').slice(0,4); }
  function show(el){ el.classList.remove('hidden'); }
  function hide(el){ el.classList.add('hidden'); }

  /* ---- Home: accounts grid ---- */
  function loadAccountButtons(){
    accountButtons.innerHTML = '';
    if (!accounts || accounts.length === 0){
      const info = document.createElement('p');
      info.className = 'small';
      info.textContent = 'Geen accounts gevonden. Log in als admin en kies "Herstel naar standaard", of voeg accounts toe.';
      accountButtons.appendChild(info);
      return;
    }
    accounts.forEach((acc, i) => {
      const card = document.createElement('div');
      card.className = 'account-card ' + classifyCard(acc);
      card.innerHTML = `
        <strong>${acc.name}</strong>
        <span>Saldo: €${formatPrice(acc.saldo)}</span>
        ${acc.type === 'gast' ? '<span class="badge">gast</span>' : ''}
      `;
      card.onclick = () => selectAccount(i);
      accountButtons.appendChild(card);
    });
  }
  function classifyCard(acc){
    if (acc.type === 'gast') return acc.saldo >= 0 ? 'green' : 'red';
    if (acc.saldo >= 0) return 'green';
    if (acc.saldo >= -10) return 'orange';
    return 'red';
  }

  /* ---- Nav ---- */
  function goHome(){
    hide(pinScreen); hide(userScreen); hide(adminScreen);
    show(homeScreen);
    adminCode.value = '';
    loadAccountButtons();
  }

  /* ---- User login ---- */
  function selectAccount(index){
    currentUserIndex = index;
    selectedUserName.textContent = 'Account: ' + accounts[index].name;
    pincode.value = '';
    hide(homeScreen); show(pinScreen);
  }
  function checkLogin(){
    if (!pincode.value){ alert('Voer pincode in'); return; }
    if (accounts[currentUserIndex].pin === pincode.value){
      pincode.value = '';
      hide(pinScreen); show(userScreen);
      welcome.textContent = 'Welkom ' + accounts[currentUserIndex].name;
      initCart();
      updateUserScreen();
    } else {
      alert('Verkeerde pincode!');
    }
  }

  /* ---- Cart helpers ---- */
  function initCart(){ cart = {}; products.forEach((_,i)=>cart[i]=0); }
  function computeCart(){
    let total = 0, items = 0;
    Object.keys(cart).forEach(i=>{
      const qty = cart[i]||0;
      items += qty;
      total += qty * (products[i]?.price||0);
    });
    return {total, items};
  }
  function renderCartSummary(){
    const {total, items} = computeCart();
    cartTotalEl.textContent = formatPrice(total);
    cartItemsEl.textContent = items;
    checkoutBtn.disabled = items === 0;
  }

  /* ---- User screen / products ---- */
  function updateUserScreen(){
    const acc = accounts[currentUserIndex];
    saldoEl.textContent = formatPrice(acc.saldo);
    if (acc.type === 'gast') { saldoEl.style.color = acc.saldo >= 0 ? 'green' : 'red'; }
    else { saldoEl.style.color = (acc.saldo >= 0 ? 'green' : (acc.saldo >= -10 ? 'orange' : 'red')); }

    productList.innerHTML = '';
    products.forEach((p,i)=>{
      const voorraadClass = p.stock <= 5 ? 'low-stock' : '';
      const row = document.createElement('div');
      row.className = 'item';
      row.innerHTML = `
        <span>${p.name} - €${formatPrice(p.price)} (<span class="${voorraadClass}">voorraad: ${p.stock}</span>)</span>
        <span>
          <input type="number" min="0" max="${p.stock}" value="${cart[i]||0}" style="width:80px;">
        </span>
      `;
      const input = row.querySelector('input');
      input.addEventListener('input', () => {
        const val = Math.max(0, Math.min(p.stock, parseInt(input.value)||0));
        cart[i] = val; input.value = val;
        renderCartSummary();
      });
      productList.appendChild(row);
    });
    renderCartSummary();
  }

  function checkoutCart(){
    const acc = accounts[currentUserIndex];
    const {total, items} = computeCart();
    if (items === 0){ alert('Je hebt niets geselecteerd.'); return; }

    // Limieten check
    if (acc.type==='gast' && acc.saldo - total < 0){ alert('Gast mag niet onder €0 komen!'); return; }
    if (acc.type==='vast' && acc.saldo - total < -10){ alert('Vast mag niet verder dan -€10 komen!'); return; }

    // Voorraad check
    fo
