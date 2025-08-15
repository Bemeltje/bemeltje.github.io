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
.badge.admin { background:#2b7cd3; }
.badge.coadmin { background:#27ae60; }
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
  <div class="item">
    <span>Beheerder inloggen</span>
    <span style="display:flex; gap:6px; align-items:center;">
      <select id="adminAccountSelect" style="min-width:220px"></select>
      <input type="password" id="adminCode" placeholder="Pincode" maxlength="4" inputmode="numeric" style="width:120px;">
      <button id="adminLoginBtn">Inloggen</button>
    </span>
  </div>
  <p class="small">Tip: Rollen worden toegekend per account. Alleen accounts met rol <strong>Admin</strong> of <strong>Co‑admin</strong> kunnen inloggen op het beheer.</p>
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
  let currentManager = null; // {index, role: 'admin'|'coadmin'}

  /* ---- State ---- */
  let accounts = safeGet('accounts', [
    {name:"Jan", pin:"1234", saldo:40.00, type:"vast", role:"user"},
    {name:"Piet", pin:"5678", saldo:5.00, type:"gast", role:"user"},
    {name:"Beheer", pin:"9999", saldo:0.00, type:"vast", role:"admin"}
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
  const adminAccountSelect = document.getElementById('adminAccountSelect');
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
  function now(){ return new Date().toLocaleString(); }
  function actorName(){ return currentManager ? accounts[currentManager.index].name : 'SYSTEEM'; }
  function logAction(text, bedrag=0){
    logs.push({gebruiker: actorName(), product: `ACTIE: ${text}`, prijs: bedrag, tijd: now()});
  }

  /* ---- Home: accounts grid + beheer select ---- */
  function loadAccountButtons(){
    accountButtons.innerHTML = '';
    accounts.forEach((acc, i) => {
      const card = document.createElement('div');
      card.className = 'account-card ' + classifyCard(acc);
      let roleBadge = '';
      if (acc.role === 'admin') roleBadge = ' <span class="badge admin">admin</span>';
      else if (acc.role === 'coadmin') roleBadge = ' <span class="badge coadmin">co-admin</span>';
      card.innerHTML = `
        <strong>${acc.name}${roleBadge}</strong>
        <span>Saldo: €${formatPrice(acc.saldo)} ${acc.type === 'gast' ? '<span class="badge">gast</span>' : ''}</span>
      `;
      card.onclick = () => selectAccount(i);
      accountButtons.appendChild(card);
    });

    // Beheerder-select vullen met alleen admin/coadmin
    adminAccountSelect.innerHTML = '';
    const staff = accounts.map((a,idx)=>({idx, a})).filter(x=>x.a.role==='admin' || x.a.role==='coadmin');
    if (staff.length===0){
      const opt = document.createElement('option');
      opt.text = '— geen beheerders —'; opt.value=''; adminAccountSelect.appendChild(opt);
    } else {
      staff.forEach(s=>{
        const opt = document.createElement('option');
        opt.value = s.idx; opt.text = `${s.a.name} (${s.a.role})`; adminAccountSelect.appendChild(opt);
      });
    }
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
    currentManager = null;
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
    for (const i of Object.keys(cart)){
      const qty = cart[i]||0;
      if (qty > products[i].stock){
        alert(`Niet genoeg voorraad voor ${products[i].name}`);
        return;
      }
    }

    // Bevestiging met totaal
    if (!confirm(`Je staat op het punt te kopen voor €${formatPrice(total)} (${items} item(s)). Doorgaan?`)) return;

    // Afrekenen
    acc.saldo -= total;
    Object.keys(cart).forEach(i=>{
      const qty = cart[i]||0;
      if (qty>0){
        products[i].stock -= qty;
        logs.push({gebruiker:acc.name, product:`${products[i].name} (x${qty})`, prijs: products[i].price*qty, tijd: now()});
        cart[i]=0;
      }
    });

    saveAll();
    updateUserScreen();
    loadAccountButtons();
    alert('Aankoop voltooid.');
  }

  /* ---- Admin login via account met rol ---- */
  function adminLogin(){
    const sel = adminAccountSelect.value;
    if (sel === ''){ alert('Kies een beheerder-account.'); return; }
    const idx = parseInt(sel);
    const acc = accounts[idx];
    if (!acc || (acc.role!=='admin' && acc.role!=='coadmin')){ alert('Geen beheerdersrol.'); return; }
    if (adminCode.value !== acc.pin){ alert('Verkeerde pincode!'); return; }
    currentManager = { index: idx, role: acc.role };
    adminCode.value = '';
    hide(homeScreen); show(adminScreen);
    adminTitle.textContent = acc.role === 'coadmin' ? 'Co-Admin Paneel' : 'Admin Paneel';
    updateAdminScreen();
    applyPermissions();
  }
  function isAdmin(){ return currentManager && currentManager.role==='admin'; }
  function isCoAdmin(){ return currentManager && currentManager.role==='coadmin'; }
  function applyPermissions(){
    // Data beheer alleen voor admin
    if (isAdmin()){
      show(dataBeheer);
    } else {
      hide(dataBeheer);
    }
    // Log-acties: co-admin mag exporteren, NIET wissen
    exportCsvBtn.disabled = false; // beide mogen exporteren
    if (isAdmin()){
      clearLogsBtn.disabled = false; clearLogsBtn.classList.remove('hidden');
    } else {
      clearLogsBtn.disabled = true; clearLogsBtn.classList.add('hidden');
    }
  }

  /* ---- Admin screen ---- */
  function updateAdminScreen(){
    adminSections.innerHTML = '';

    // Accounts sectie
    const accDiv = document.createElement('div');
    accDiv.innerHTML = `
      <h3>Accounts</h3>
      <div class="item">
        <div style="flex:1; min-width:240px;">
          <input id="newName" placeholder="Naam" ${!(isAdmin()||isCoAdmin())?'disabled':''}>
          <input id="newPin" placeholder="Pincode (4 cijfers)" maxlength="4" inputmode="numeric" ${!(isAdmin()||isCoAdmin())?'disabled':''}>
          <input type="number" id="newSaldo" placeholder="Startsaldo" ${!(isAdmin()||isCoAdmin())?'disabled':''}>
          <select id="newType" ${!(isAdmin()||isCoAdmin())?'disabled':''}>
            <option value="gast">Gast</option>
            <option value="vast">Vast</option>
          </select>
          <select id="newRole" ${!isAdmin()?'disabled':''}>
            <option value="user">Rol: Gebruiker</option>
            <option value="coadmin">Rol: Co-admin</option>
            <option value="admin">Rol: Admin</option>
          </select>
        </div>
        <div>
          ${(isAdmin()||isCoAdmin())?'<button id="addAccountBtn">Account toevoegen</button>':''}
        </div>
      </div>
      <div id="accountList"></div>
    `;
    adminSections.appendChild(accDiv);

    // Producten sectie
    const prodDiv = document.createElement('div');
    prodDiv.innerHTML = `
      <h3>Producten</h3>
      ${isAdmin() ? `
      <div class="item">
        <div style="flex:1; min-width:220px;">
          <input id="prodName" placeholder="Productnaam">
          <input type="number" step="0.01" id="prodPrice" placeholder="Prijs">
          <input type="number" id="prodStock" placeholder="Voorraad">
        </div>
        <div>
          <button id="addProductBtn">Product toevoegen</button>
        </div>
      </div>`: ''}
      <div id="productAdminList"></div>
    `;
    adminSections.appendChild(prodDiv);

    // Accounts lijst
    const accountList = accDiv.querySelector('#accountList');
    accountList.innerHTML = '';
    accounts.forEach((acc,i)=>{
      const row = document.createElement('div');
      row.className = 'item';
      row.innerHTML = `
        <span>
          ${acc.name}
          ${acc.type==='gast' ? '<span class="badge">gast</span>' : ''}
          ${acc.role==='admin' ? '<span class="badge admin">admin</span>' : acc.role==='coadmin' ? '<span class="badge coadmin">co-admin</span>' : ''}
          (Saldo €${formatPrice(acc.saldo)})
        </span>
        <span>
          ${isAdmin()?`<button data-role="${i}">Rol wijzigen</button>`:''}
          ${isAdmin()?`<button data-pin="${i}">PIN wijzigen</button>`:''}
          <button data-add="${i}">+€</button>
          ${isAdmin()?`<button class="red" data-del="${i}">X</button>`:''}
        </span>
      `;
      accountList.appendChild(row);
    });

    // Producten lijst
    const prodList = prodDiv.querySelector('#productAdminList');
    prodList.innerHTML = '';
    products.forEach((p,i)=>{
      const voorraadClass = p.stock <=5 ? 'low-stock' : '';
      const row = document.createElement('div');
      row.className='item';
      row.innerHTML = `
        <span>${p.name} (€${formatPrice(p.price)}) - <span class="${voorraadClass}">Voorraad: ${p.stock}</span></span>
        <span>
          ${isAdmin()?`<button data-price="${i}">Prijs wijzigen</button>`:''}
          ${isAdmin()?`<button data-restock="${i}">Voorraad bijvullen</button>`:''}
          ${isAdmin()?`<button class="red" data-pdel="${i}">X</button>`:''}
        </span>
      `;
      prodList.appendChild(row);
    });

    // Logboek
    let html = '<table><tr><th>Gebruiker</th><th>Product/Actie</th><th>Prijs/Bedrag</th><th>Tijd</th></tr>';
    logs.forEach(l=>{ html += `<tr><td>${l.gebruiker}</td><td>${l.product}</td><td>€${formatPrice(l.prijs)}</td><td>${l.tijd}</td></tr>`; });
    html += '</table>';
    logList.innerHTML = html;

    // wire inputs
    const newPin = adminSections.querySelector('#newPin');
    if (newPin) newPin.addEventListener('input', ()=>digitsOnly(newPin));

    // Buttons wiring
    const addAccountBtn = adminSections.querySelector('#addAccountBtn');
    if (addAccountBtn) addAccountBtn.addEventListener('click', addAccount);
    adminSections.querySelectorAll('button[data-del]').forEach(btn=>btn.addEventListener('click',()=>deleteAccount(+btn.dataset.del)));
    adminSections.querySelectorAll('button[data-add]').forEach(btn=>btn.addEventListener('click',()=>addSaldo(+btn.dataset.add)));
    adminSections.querySelectorAll('button[data-pin]').forEach(btn=>btn.addEventListener('click',()=>changePin(+btn.dataset.pin)));
    adminSections.querySelectorAll('button[data-role]').forEach(btn=>btn.addEventListener('click',()=>changeRole(+btn.dataset.role)));
    const addProductBtn = adminSections.querySelector('#addProductBtn');
    if (addProductBtn) addProductBtn.addEventListener('click', addProduct);
    adminSections.querySelectorAll('button[data-pdel]').forEach(btn=>btn.addEventListener('click',()=>deleteProduct(+btn.dataset.pdel)));
    adminSections.querySelectorAll('button[data-restock]').forEach(btn=>btn.addEventListener('click',()=>restockProduct(+btn.dataset.restock)));
    adminSections.querySelectorAll('button[data-price]').forEach(btn=>btn.addEventListener('click',()=>changePrice(+btn.dataset.price)));
  }

  /* ---- Admin actions ---- */
  function addAccount(){
    if (!(isAdmin()||isCoAdmin())) return;
    const name = document.getElementById('newName').value.trim();
    const pin = document.getElementById('newPin').value.trim();
    const saldo = parseFloat(document.getElementById('newSaldo').value);
    const type = document.getElementById('newType').value;
    const roleSelect = document.getElementById('newRole');
    const role = isAdmin() ? roleSelect.value : 'user'; // co-admin kan geen rol kiezen
    if (!name || !pin || isNaN(saldo)){ alert('Vul alle velden in!'); return; }
    if (!/^\d{1,4}$/.test(pin)){ alert('Pincode moet 1–4 cijfers zijn.'); return; }
    accounts.push({name, pin, saldo, type, role});
    logAction(`Account aangemaakt: ${name} (rol: ${role})`);
    saveAll(); loadAccountButtons(); updateAdminScreen();
    document.getElementById('newName').value='';
    document.getElementById('newPin').value='';
    document.getElementById('newSaldo').value='';
    document.getElementById('newType').value='gast';
    if (isAdmin()) document.getElementById('newRole').value='user';
  }
  function deleteAccount(i){
    if (!isAdmin()) return;
    if (!confirm(`Account "${accounts[i].name}" verwijderen?`)) return;
    logAction(`Account verwijderd: ${accounts[i].name}`);
    accounts.splice(i,1);
    saveAll(); loadAccountButtons(); updateAdminScreen();
  }
  function addSaldo(i){
    const bedrag = parseFloat(prompt('Bedrag toevoegen:'));
    if (isNaN(bedrag)) return;
    accounts[i].saldo += bedrag;
    logAction(`Saldo +€${formatPrice(bedrag)} voor ${accounts[i].name}`, bedrag);
    saveAll(); loadAccountButtons(); updateAdminScreen();
  }
  function changePin(i){
    if (!isAdmin()) return;
    const nieuw = prompt(`Nieuwe pincode voor ${accounts[i].name} (max 4 cijfers):`, "");
    if (nieuw===null) return;
    if (!/^\d{1,4}$/.test(nieuw)){ alert('Pincode moet 1–4 cijfers zijn.'); return; }
    accounts[i].pin = nieuw;
    logAction(`PIN gewijzigd voor ${accounts[i].name}`);
    saveAll(); updateAdminScreen();
    alert('Pincode bijgewerkt.');
  }
  function changeRole(i){
    if (!isAdmin()) return;
    const huidige = accounts[i].role || 'user';
    const nieuw = prompt(`Rol voor ${accounts[i].name} (user / coadmin / admin):`, huidige);
    if (nieuw===null) return;
    if (!['user','coadmin','admin'].includes(nieuw)){ alert('Ongeldige rol.'); return; }
    accounts[i].role = nieuw;
    logAction(`Rol gewijzigd: ${accounts[i].name} → ${nieuw}`);
    saveAll(); loadAccountButtons(); updateAdminScreen();
  }
  function addProduct(){
    if (!isAdmin()) return;
    const name = document.getElementById('prodName').value.trim();
    const price = parseFloat(document.getElementById('prodPrice').value);
    const stock = parseInt(document.getElementById('prodStock').value);
    if (!name || isNaN(price) || isNaN(stock)){ alert('Vul alle velden in!'); return; }
    products.push({name, price, stock});
    logAction(`Product toegevoegd: ${name} (€${formatPrice(price)})`);
    saveAll(); updateAdminScreen();
    document.getElementById('prodName').value='';
    document.getElementById('prodPrice').value='';
    document.getElementById('prodStock').value='';
  }
  function deleteProduct(i){
    if (!isAdmin()) return;
    if (!confirm(`Product "${products[i].name}" verwijderen?`)) return;
    logAction(`Product verwijderd: ${products[i].name}`);
    products.splice(i,1);
    saveAll(); updateAdminScreen();
  }
  function restockProduct(i){
    if (!isAdmin()) return;
    const add = parseInt(prompt(`Aantal bijvullen voor "${products[i].name}" (huidig: ${products[i].stock})`, "0"));
    if (isNaN(add)) return;
    products[i].stock = Math.max(0, products[i].stock + add);
    logAction(`Voorraad +${add} voor ${products[i].name}`);
    saveAll(); updateAdminScreen();
  }
  function changePrice(i){
    if (!isAdmin()) return;
    const nieuw = parseFloat(prompt(`Nieuwe prijs voor "${products[i].name}" (huidig: €${formatPrice(products[i].price)})`, products[i].price));
    if (isNaN(nieuw)) return;
    const oud = products[i].price;
    products[i].price = nieuw;
    logAction(`Prijs gewijzigd: ${products[i].name} €${formatPrice(oud)} → €${formatPrice(nieuw)}`);
    saveAll(); updateAdminScreen();
  }

  /* ---- Logs ---- */
  function exportLogsToCSV(){
    // Admin én Co-admin mogen exporteren
    if (!(isAdmin()||isCoAdmin())) return;
    if (logs.length===0){ alert('Het logboek is leeg.'); return; }
    const csv = "Gebruiker,Product,Prijs,Tijd\n" + logs.map(l=>`${l.gebruiker},${l.product},${formatPrice(l.prijs)},${l.tijd}`).join('\n');
    const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href=url; a.download='logboek.csv'; document.body.appendChild(a); a.click();
    document.body.removeChild(a); URL.revokeObjectURL(url);
  }
  function clearLogs(){
    // Alleen Admin mag wissen
    if (!isAdmin()) return;
    if (!confirm('Logboek wissen?')) return;
    logs=[]; saveAll(); updateAdminScreen();
  }

  /* ---- Data import/export/reset (alleen admin) ---- */
  function exportAllToJSON(){
    if (!isAdmin()) return;
    const payload = { version: APP_VERSION, exportedAt: new Date().toISOString(), accounts, products, logs };
    const blob = new Blob([JSON.stringify(payload,null,2)], {type:'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href=url; a.download='fictief-geld-data.json'; document.body.appendChild(a); a.click();
    document.body.removeChild(a); URL.revokeObjectURL(url);
  }
  function importAllFromJSON(file){
    if (!isAdmin()) return;
    if (!file) return;
    const r = new FileReader();
    r.onload = e=>{
      try{
        const data = JSON.parse(e.target.result);
        if (!data || !Array.isArray(data.accounts) || !Array.isArray(data.products) || !Array.isArray(data.logs)){ alert('Onjuist JSON-formaat.'); return; }
        if (!confirm('Huidige data overschrijven?')) return;
        accounts = data.accounts; products = data.products; logs = data.logs;
        saveAll(); loadAccountButtons(); updateAdminScreen(); alert('Data geïmporteerd.');
      }catch(err){ alert('Kon JSON niet lezen: '+err.message); }
      importFile.value='';
    };
    r.readAsText(file);
  }
  function resetAllData(){
    if (!isAdmin()) return;
    if (!confirm('Alle data herstellen naar standaard?')) return;
    accounts = [
      {name:"Jan", pin:"1234", saldo:40.00, type:"vast", role:"user"},
      {name:"Piet", pin:"5678", saldo:5.00, type:"gast", role:"user"},
      {name:"Beheer", pin:"9999", saldo:0.00, type:"vast", role:"admin"}
    ];
    products = [
      {name:"Chips", price:0.75, stock:20},
      {name:"Bier",  price:0.75, stock:30},
      {name:"Cola",  price:1.00, stock:15}
    ];
    logs = [];
    saveAll(); loadAccountButtons(); updateAdminScreen(); alert('Hersteld.');
  }

  /* ---- Wire up ---- */
  adminLoginBtn.addEventListener('click', adminLogin);
  userLoginBtn.addEventListener('click', checkLogin);
  cancelPinBtn.addEventListener('click', goHome);
  logoutUserBtn.addEventListener('click', goHome);
  logoutAdminBtn.addEventListener('click', goHome);

  adminCode.addEventListener('input', ()=>digitsOnly(adminCode));
  pincode.addEventListener('input', ()=>digitsOnly(pincode));

  // User cart buttons
  checkoutBtn.addEventListener('click', checkoutCart);
  clearCartBtn.addEventListener('click', () => { initCart(); updateUserScreen(); });

  // Admin: logs + data beheer
  exportCsvBtn.addEventListener('click', exportLogsToCSV);
  clearLogsBtn.addEventListener('click', clearLogs);
  exportJsonBtn.addEventListener('click', exportAllToJSON);
  importJsonBtn.addEventListener('click', ()=>importFile.click());
  importFile.addEventListener('change', ()=>importAllFromJSON(importFile.files[0]));
  resetBtn.addEventListener('click', resetAllData);

  // Start
  loadAccountButtons();
});
</script>
</body>
</html>
