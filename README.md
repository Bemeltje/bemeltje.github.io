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
input,select { padding:9px; margin:5px 0; width:100%; border:1px solid #ccc; border-radius:5px; font-size:.95em; }
.hidden { display:none; }
.item { border-bottom:1px solid #eee; padding:8px 0; display:flex; justify-content:space-between; align-items:center; }
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
</style>
</head>
<body>
<header>💰 Fictief Geld Systeem</header>

<!-- HOME -->
<div class="container" id="homeScreen">
  <h2>Kies je account</h2>
  <div id="accountButtons"><!-- gevuld door JS --></div>
  <hr>
  <input type="password" id="adminCode" placeholder="Admin/Co-admin pincode" maxlength="4" inputmode="numeric">
  <button id="adminLoginBtn">Inloggen</button>
  <p class="small">Zie je geen accounts? Klik in Admin op <em>Herstel naar standaard</em>, of wis je site-data (lokale opslag).</p>
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
  <hr>
  <button class="red" id="logoutUserBtn">Uitloggen</button>
</div>

<!-- ADMIN -->
<div class="container hidden" id="adminScreen">
  <h2 id="adminTitle">Admin Paneel</h2>
  <div id="adminSections"></div>

  <h3>Logboek</h3>
  <div id="logList"></div>
  <div>
    <button id="exportCsvBtn">📥 Exporteer naar CSV</button>
    <button class="red" id="clearLogsBtn">Logboek wissen</button>
  </div>

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

  <hr>
  <button class="red" id="logoutAdminBtn">Uitloggen</button>
</div>

<script>
document.addEventListener('DOMContentLoaded', () => {
  /* ---- Config ---- */
  const APP_VERSION = "2025-08-15";
  const adminPIN = "9999";
  const coAdminPIN = "8888";
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
  let cart = {};

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
  const logoutUserBtn = document.getElementById('logoutUserBtn');

  const adminTitle = document.getElementById('adminTitle');
  const adminSections = document.getElementById('adminSections');
  const logList = document.getElementById('logList');
  const exportCsvBtn = document.getElementById('exportCsvBtn');
  const clearLogsBtn = document.getElementById('clearLogsBtn');
  const exportJsonBtn = document.getElementById('exportJsonBtn');
  const importJsonBtn = document.getElementById('importJsonBtn');
  const importFile = document.getElementById('importFile');
  const resetBtn = document.getElementById('resetBtn');
  const logoutAdminBtn = document.getElementById('logoutAdminBtn');

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
      updateUserScreen();
    } else {
      alert('Verkeerde pincode!');
    }
  }
  function updateUserScreen(){
    const acc = accounts[currentUserIndex];
    saldoEl.textContent = formatPrice(acc.saldo);
    if (acc.type === 'gast') { saldoEl.style.color = acc.saldo >= 0 ? 'green' : 'red'; }
    else { saldoEl.style.color = (acc.saldo >= 0 ? 'green' : (acc.saldo >= -10 ? 'orange' : 'red')); }

    productList.innerHTML = '';
    cart = {};
    products.forEach((p,i)=>{
      cart[i] = 0;
      const voorraadClass = p.stock <= 5 ? 'low-stock' : '';
      const div = document.createElement('div');
      div.className = 'item';
      div.innerHTML = `
        <span>${p.name} - €${formatPrice(p.price)} (<span class="${voorraadClass}">voorraad: ${p.stock}</span>)</span>
        <input type="number" min="0" max="${p.stock}" value="0" style="width:70px;">
      `;
      const input = div.querySelector('input');
      input.addEventListener('change', () => {
        const val = Math.max(0, Math.min(p.stock, parseInt(input.value)||0));
        cart[i] = val; input.value = val;
      });
      productList.appendChild(div);
    });
    const checkoutBtn = document.createElement('button');
    checkoutBtn.textContent = '🛒 Afrekenen';
    checkoutBtn.onclick = checkoutCart;
    productList.appendChild(checkoutBtn);
  }
  function checkoutCart(){
    const acc = accounts[currentUserIndex];
    let totaal = 0; const items=[];
    Object.keys(cart).forEach(i=>{
      const qty = cart[i]||0;
      if (qty>0){
        if (qty > products[i].stock){ items.length=0; totaal=0; alert(`Niet genoeg voorraad voor ${products[i].name}`); return; }
        totaal += products[i].price * qty;
        items.push({idx:+i, name:products[i].name, qty, price:products[i].price});
      }
    });
    if (items.length===0){ alert('Je hebt niets geselecteerd.'); return; }
    if (!confirm(`Totaal: €${formatPrice(totaal)}. Doorgaan?`)) return;

    if (acc.type==='gast' && acc.saldo - totaal < 0){ alert('Gast mag niet onder €0 komen!'); return; }
    if (acc.type==='vast' && acc.saldo - totaal < -10){ alert('Vast mag niet verder dan -€10 komen!'); return; }

    acc.saldo -= totaal;
    items.forEach(it=>{
      products[it.idx].stock -= it.qty;
      logs.push({gebruiker:acc.name, product:`${it.name} (x${it.qty})`, prijs: it.price*it.qty, tijd:new Date().toLocaleString()});
    });
    saveAll();
    updateUserScreen();
    loadAccountButtons();
  }

  /* ---- Admin login ---- */
  function adminLogin(){
    const pin = adminCode.value;
    if (pin === adminPIN){ isCoAdmin=false; }
    else if (pin === coAdminPIN){ isCoAdmin=true; }
    else { alert('Verkeerde pincode!'); return; }
    adminCode.value = '';
    hide(homeScreen); show(adminScreen);
    adminTitle.textContent = isCoAdmin ? 'Co-Admin Paneel' : 'Admin Paneel';
    updateAdminScreen();
  }

  /* ---- Admin screen ---- */
  function updateAdminScreen(){
    adminSections.innerHTML = '';

    // Accounts sectie
    const accDiv = document.createElement('div');
    accDiv.innerHTML = `
      <h3>Accounts</h3>
      <input id="newName" placeholder="Naam" ${isCoAdmin?'disabled':''}>
      <input id="newPin" placeholder="Pincode" maxlength="4" inputmode="numeric" ${isCoAdmin?'disabled':''}>
      <input type="number" id="newSaldo" placeholder="Startsaldo" ${isCoAdmin?'disabled':''}>
      <select id="newType" ${isCoAdmin?'disabled':''}>
        <option value="gast">Gast</option>
        <option value="vast">Vast</option>
      </select>
      ${isCoAdmin?'':'<button id="addAccountBtn">Account toevoegen</button>'}
      <div id="accountList"></div>
    `;
    adminSections.appendChild(accDiv);

    // Producten sectie
    if (!isCoAdmin){
      const prodDiv = document.createElement('div');
      prodDiv.innerHTML = `
        <h3>Producten</h3>
        <input id="prodName" placeholder="Productnaam">
        <input type="number" step="0.01" id="prodPrice" placeholder="Prijs">
        <input type="number" id="prodStock" placeholder="Voorraad">
        <button id="addProductBtn">Product toevoegen</button>
        <div id="productAdminList"></div>
      `;
      adminSections.appendChild(prodDiv);
    }

    // Accounts lijst
    const accountList = accDiv.querySelector('#accountList');
    accountList.innerHTML = '';
    accounts.forEach((acc,i)=>{
      const row = document.createElement('div');
      row.className = 'item';
      row.innerHTML = `
        <span>${acc.name} (€${formatPrice(acc.saldo)}) ${acc.type==='gast'?'[gast]':''}</span>
        <span>
          <button ${isCoAdmin?'disabled':''} data-del="${i}">X</button>
          <button data-add="${i}">+€</button>
        </span>
      `;
      accountList.appendChild(row);
    });

    // Producten lijst
    if (!isCoAdmin){
      const prodList = adminSections.querySelector('#productAdminList');
      prodList.innerHTML = '';
      products.forEach((p,i)=>{
        const voorraadClass = p.stock <=5 ? 'low-stock' : '';
        const row = document.createElement('div');
        row.className='item';
        row.innerHTML = `
          <span>${p.name} (€${formatPrice(p.price)}) - <span class="${voorraadClass}">Voorraad: ${p.stock}</span></span>
          <span><button data-pdel="${i}">X</button></span>
        `;
        prodList.appendChild(row);
      });
    }

    // Logboek
    let html = '<table><tr><th>Gebruiker</th><th>Product</th><th>Prijs</th><th>Tijd</th></tr>';
    logs.forEach(l=>{ html += `<tr><td>${l.gebruiker}</td><td>${l.product}</td><td>€${formatPrice(l.prijs)}</td><td>${l.tijd}</td></tr>`; });
    html += '</table>';
    logList.innerHTML = html;

    // Pin inputs numeric-only
    const newPin = adminSections.querySelector('#newPin');
    if (newPin) newPin.addEventListener('input', ()=>digitsOnly(newPin));

    // Buttons wiring
    const addAccountBtn = adminSections.querySelector('#addAccountBtn');
    if (addAccountBtn) addAccountBtn.addEventListener('click', addAccount);
    adminSections.querySelectorAll('button[data-del]').forEach(btn=>btn.addEventListener('click',()=>deleteAccount(+btn.dataset.del)));
    adminSections.querySelectorAll('button[data-add]').forEach(btn=>btn.addEventListener('click',()=>addSaldo(+btn.dataset.add)));
    const addProductBtn = adminSections.querySelector('#addProductBtn');
    if (addProductBtn) addProductBtn.addEventListener('click', addProduct);
    adminSections.querySelectorAll('button[data-pdel]').forEach(btn=>btn.addEventListener('click',()=>deleteProduct(+btn.dataset.pdel)));
  }

  /* ---- Admin actions ---- */
  function addAccount(){
    const name = document.getElementById('newName').value.trim();
    const pin = document.getElementById('newPin').value.trim();
    const saldo = parseFloat(document.getElementById('newSaldo').value);
    const type = document.getElementById('newType').value;
    if (!name || !pin || isNaN(saldo)){ alert('Vul alle velden in!'); return; }
    if (pin.length > 4){ alert('Pincode max 4 cijfers!'); return; }
    accounts.push({name, pin, saldo, type});
    saveAll(); loadAccountButtons(); updateAdminScreen();
    document.getElementById('newName').value='';
    document.getElementById('newPin').value='';
    document.getElementById('newSaldo').value='';
    document.getElementById('newType').value='gast';
  }
  function deleteAccount(i){
    if (isCoAdmin) return;
    accounts.splice(i,1);
    saveAll(); loadAccountButtons(); updateAdminScreen();
  }
  function addSaldo(i){
    const bedrag = parseFloat(prompt('Bedrag toevoegen:'));
    if (isNaN(bedrag)) return;
    accounts[i].saldo += bedrag;
    saveAll(); loadAccountButtons(); updateAdminScreen();
  }
  function addProduct(){
    if (isCoAdmin) return;
    const name = document.getElementById('prodName').value.trim();
    const price = parseFloat(document.getElementById('prodPrice').value);
    const stock = parseInt(document.getElementById('prodStock').value);
    if (!name || isNaN(price) || isNaN(stock)){ alert('Vul alle velden in!'); return; }
    products.push({name, price, stock});
    saveAll(); updateAdminScreen();
    document.getElementById('prodName').value='';
    document.getElementById('prodPrice').value='';
    document.getElementById('prodStock').value='';
  }
  function deleteProduct(i){
    if (isCoAdmin) return;
    products.splice(i,1);
    saveAll(); updateAdminScreen();
  }

  /* ---- Logs ---- */
  function exportLogsToCSV(){
    if (logs.length===0){ alert('Het logboek is leeg.'); return; }
    const csv = "Gebruiker,Product,Prijs,Tijd\n" + logs.map(l=>`${l.gebruiker},${l.product},${formatPrice(l.prijs)},${l.tijd}`).join('\n');
    const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href=url; a.download='logboek.csv'; document.body.appendChild(a); a.click();
    document.body.removeChild(a); URL.revokeObjectURL(url);
  }
  function clearLogs(){ if (!confirm('Logboek wissen?')) return; logs=[]; saveAll(); updateAdminScreen(); }

  /* ---- Data import/export/reset ---- */
  function exportAllToJSON(){
    const payload = { version: APP_VERSION, exportedAt: new Date().toISOString(), accounts, products, logs };
    const blob = new Blob([JSON.stringify(payload,null,2)], {type:'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href=url; a.download='fictief-geld-data.json'; document.body.appendChild(a); a.click();
    document.body.removeChild(a); URL.revokeObjectURL(url);
  }
  function importAllFromJSON(file){
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
    if (!confirm('Alle data herstellen naar standaard?')) return;
    accounts = [
      {name:"Jan", pin:"1234", saldo:10.00, type:"vast"},
      {name:"Piet", pin:"5678", saldo:5.00, type:"gast"}
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
