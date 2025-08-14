<!DOCTYPE html><html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Wallet Systeem</title>
<style>
  body { font-family: Arial, sans-serif; background: #fff; color: #222; margin: 0; }
  header { background: #333; color: #fff; padding: 12px 16px; text-align: center; }
  .container { padding: 16px; max-width: 900px; margin: 0 auto; }
  button, input, select { font-size: 1rem; padding: 8px 12px; border-radius: 8px; border: 1px solid #ccc; margin: 4px; }
  button { background: #0b66c3; color: #fff; border: none; cursor: pointer; }
  button:hover { background: #084f96; }
  .secondary { background: #f2f2f2; color: #222; border: 1px solid #ddd; }
  .hidden { display: none; }
  .grid { display: grid; gap: 12px; }
  .accounts { grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); }
  .card { border: 1px solid #e5e5e5; border-radius: 12px; padding: 12px; background: #fafafa; }
  .item { border: 1px solid #e5e5e5; border-radius: 12px; padding: 10px; display: flex; justify-content: space-between; align-items: center; }
  .row { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
  .badge { padding: 2px 8px; border-radius: 999px; font-size: 0.8rem; border: 1px solid #ddd; background:#fff }
  .log { max-height: 200px; overflow: auto; border: 1px solid #e5e5e5; border-radius: 8px; padding: 8px; background:#fff }
</style>
</head>
<body>
<header>
  <h1>Fictief Wallet Systeem</h1>
</header>
<div class="container">
  <!-- Home -->
  <section id="homeScreen">
    <h2>Accounts</h2>
    <div id="accountList" class="grid accounts"></div>
    <div class="row" style="margin-top:12px">
      <button id="btnAdmin">🔑 Admin</button>
    </div>
  </section>  <!-- PIN prompt -->  <section id="pinScreen" class="hidden card">
    <h3>Log in als <span id="pinUserName"></span></h3>
    <div class="row">
      <input id="pinInput" type="password" placeholder="Pincode" />
      <button id="btnPinOk">Inloggen</button>
      <button id="btnPinBack" class="secondary">Terug</button>
    </div>
  </section>  <!-- User dashboard -->  <section id="userDashboard" class="hidden">
    <div class="row">
      <h2 id="welcomeUser" style="margin-right:auto"></h2>
      <button id="btnUserBack" class="secondary">⬅ Terug</button>
    </div>
    <div id="balanceWrap" class="row" style="margin:8px 0"></div><div class="card">
  <h3>Producten</h3>
  <div id="productList" class="grid"></div>
</div>

<div class="card" id="cartWrap">
  <h3>Winkelwagen</h3>
  <div id="cartList"></div>
  <div class="row">
    <div id="cartTotal" style="margin-right:auto">Totaal: €0,00</div>
    <button id="btnCheckout">Bevestig aankoop</button>
    <button id="btnEmptyCart" class="secondary">Leegmaken</button>
  </div>
</div>

  </section>  <!-- Admin dashboard -->  <section id="adminDashboard" class="hidden">
    <div class="row">
      <h2 style="margin-right:auto">Admin Beheer</h2>
      <button id="btnAdminBack" class="secondary">⬅ Terug</button>
    </div><div class="card">
  <h3>Accounts</h3>
  <div class="row">
    <input id="accName" placeholder="Naam" />
    <input id="accPin" placeholder="PIN (4 cijfers)" maxlength="4" />
    <select id="accType">
      <option value="vast">Vaste gebruiker (mag tot -10)</option>
      <option value="gast">Gast (niet onder 0)</option>
    </select>
    <input id="accStart" type="number" step="0.01" placeholder="Startsaldo" />
    <button id="btnAddAcc">Account toevoegen</button>
  </div>
  <div id="adminAccList" class="grid"></div>
</div>

<div class="card">
  <h3>Geld toevoegen</h3>
  <div class="row">
    <select id="selFunds"></select>
    <input id="addAmount" type="number" step="0.01" placeholder="Bedrag" />
    <button id="btnAddFunds">Toevoegen</button>
  </div>
</div>

<div class="card">
  <h3>Producten</h3>
  <div class="row">
    <input id="prodName" placeholder="Productnaam" />
    <input id="prodPrice" type="number" step="0.01" placeholder="Prijs" />
    <input id="prodStock" type="number" placeholder="Voorraad" />
    <button id="btnAddProd">Product toevoegen</button>
  </div>
  <div id="adminProdList" class="grid"></div>
</div>

<div class="card">
  <h3>Logboek</h3>
  <div class="row">
    <button id="btnExportCsv" class="secondary">Exporteer CSV</button>
  </div>
  <div id="logView" class="log"></div>
</div>

<div class="card">
  <h3>Admin pincode</h3>
  <div class="row">
    <input id="adminPinInput" placeholder="Nieuwe admin PIN" maxlength="8" />
    <button id="btnSetAdminPin">Wijzigen</button>
  </div>
</div>

  </section>
</div><script>
// ===== Data =====
let users = [];
let products = [];
let logs = []; // {ts, type, user, details, amount}
let cart = []; // {id, name, price}
let currentUser = null; // {name, pin, balance, type}
let selectedForPin = null; // user object to login
let adminPin = '0000';

// ===== Storage helpers =====
const saveAll = () => {
  localStorage.setItem('wallet_users', JSON.stringify(users));
  localStorage.setItem('wallet_products', JSON.stringify(products));
  localStorage.setItem('wallet_logs', JSON.stringify(logs));
  localStorage.setItem('wallet_adminpin', adminPin);
};
const loadAll = () => {
  users = JSON.parse(localStorage.getItem('wallet_users') || '[]');
  products = JSON.parse(localStorage.getItem('wallet_products') || '[]');
  logs = JSON.parse(localStorage.getItem('wallet_logs') || '[]');
  adminPin = localStorage.getItem('wallet_adminpin') || '0000';
  // Seed demo if empty to prevent blank home
  if (!users.length) {
    users = [
      { name:'Jan', pin:'1111', balance:10, type:'vast' },
      { name:'Piet', pin:'2222', balance:5, type:'gast' }
    ];
  }
  if (!products.length) {
    products = [
      { id:1, name:'Chips', price:1.50, stock:10 },
      { id:2, name:'Snoep', price:1.00, stock:15 },
      { id:3, name:'Cola',  price:2.00, stock:8 }
    ];
  }
};

// ===== UI refs =====
const $ = (id) => document.getElementById(id);

// ===== Home render =====
function renderHome() {
  $('#homeScreen').classList.remove('hidden');
  $('#pinScreen').classList.add('hidden');
  $('#userDashboard').classList.add('hidden');
  $('#adminDashboard').classList.add('hidden');

  const box = $('#accountList');
  box.innerHTML = '';
  users.forEach((u, idx) => {
    const el = document.createElement('div');
    el.className = 'card';
    el.innerHTML = `
      <div class="row" style="justify-content:space-between">
        <div>
          <div style="font-weight:700">${u.name}</div>
          <div class="badge">${u.type === 'vast' ? 'Vast (limiet -10)' : 'Gast (min 0)'}</div>
        </div>
        <div class="row">
          <div class="badge">Saldo: €${u.balance.toFixed(2)}</div>
          <button data-idx="${idx}">Inloggen</button>
        </div>
      </div>`;
    el.querySelector('button').onclick = () => showPin(idx);
    box.appendChild(el);
  });
}

function showPin(idx) {
  selectedForPin = users[idx];
  $('#homeScreen').classList.add('hidden');
  $('#pinScreen').classList.remove('hidden');
  $('#pinUserName').textContent = selectedForPin.name;
  $('#pinInput').value = '';
}

// ===== Login flows =====
$('#btnPinBack').onclick = () => { renderHome(); };
$('#btnPinOk').onclick = () => {
  const pin = $('#pinInput').value.trim();
  if (selectedForPin && pin === selectedForPin.pin) {
    currentUser = selectedForPin;
    openUserDashboard();
  } else {
    alert('Ongeldige pincode');
  }
};

$('#btnAdmin').onclick = () => {
  const pin = prompt('Voer admin pincode in');
  if (pin === adminPin) {
    currentUser = { name:'Admin', type:'admin' };
    openAdminDashboard();
  } else { alert('Ongeldige admin pincode'); }
};

// ===== User dashboard =====
function openUserDashboard() {
  $('#pinScreen').classList.add('hidden');
  $('#userDashboard').classList.remove('hidden');
  $('#welcomeUser').textContent = `Welkom, ${currentUser.name}`;
  renderBalance();
  renderProducts();
  renderCart();
}
$('#btnUserBack').onclick = () => { currentUser=null; renderHome(); };

function renderBalance() {
  const wrap = $('#balanceWrap');
  wrap.innerHTML = '';
  if (currentUser && typeof currentUser.balance === 'number') {
    const b = document.createElement('div');
    b.className = 'badge';
    b.textContent = `Saldo: €${currentUser.balance.toFixed(2)}`;
    wrap.appendChild(b);
    const t = document.createElement('div');
    t.className = 'badge';
    t.textContent = currentUser.type === 'vast' ? 'Vast: limiet -10' : 'Gast: min 0';
    wrap.appendChild(t);
  }
}

function renderProducts() {
  const list = $('#productList');
  list.innerHTML = '';
  products.forEach((p, i) => {
    const row = document.createElement('div');
    row.className = 'item';
    row.innerHTML = `<div><strong>${p.name}</strong><br><span>€${p.price.toFixed(2)} • Voorraad: ${p.stock}</span></div>`;
    const btn = document.createElement('button');
    btn.textContent = 'In winkelwagen';
    btn.onclick = () => addToCart(i);
    row.appendChild(btn);
    list.appendChild(row);
  });
}

function addToCart(idx) {
  const p = products[idx];
  if (!p || p.stock <= 0) { alert('Niet op voorraad'); return; }
  cart.push({ id:p.id, name:p.name, price:p.price });
  renderCart();
}

function renderCart() {
  const list = $('#cartList');
  const totalEl = $('#cartTotal');
  if (!cart.length) {
    list.innerHTML = '<em>Leeg</em>';
    totalEl.textContent = 'Totaal: €0,00';
    return;
  }
  list.innerHTML = '';
  cart.forEach((c, i) => {
    const row = document.createElement('div');
    row.className = 'row';
    row.style.justifyContent = 'space-between';
    row.innerHTML = `<div>${c.name} — €${c.price.toFixed(2)}</div>`;
    const rm = document.createElement('button'); rm.className='secondary'; rm.textContent='Verwijderen';
    rm.onclick = () => { cart.splice(i,1); renderCart(); };
    row.appendChild(rm);
    list.appendChild(row);
  });
  const total = cart.reduce((s,x)=>s+x.price,0);
  totalEl.textContent = `Totaal: €${total.toFixed(2)}`;
}

$('#btnEmptyCart').onclick = () => { cart = []; renderCart(); };
$('#btnCheckout').onclick = () => {
  if (!cart.length) return alert('Winkelwagen is leeg');
  const total = +cart.reduce((s,x)=>s+x.price,0).toFixed(2);
  const limit = currentUser.type === 'vast' ? -10 : 0; // vast mag tot -10
  const newBal = +(currentUser.balance - total).toFixed(2);
  if (newBal < limit) return alert('Onvoldoende saldo voor deze aankoop');
  // Update stock per item
  cart.forEach(ci => {
    const p = products.find(pp => pp.id === ci.id);
    if (p && p.stock > 0) p.stock -= 1;
  });
  // Update user balance
  const u = users.find(u=>u.name===currentUser.name && u.pin===currentUser.pin);
  if (u) u.balance = newBal;
  currentUser.balance = newBal;
  // Log entry
  const items = cart.map(c=>`${c.name}`).join(', ');
  logs.push({ ts:new Date().toISOString(), type:'purchase', user:currentUser.name, details:items, amount:total });
  // Save & reset
  cart = [];
  saveAll();
  renderBalance();
  renderProducts();
  renderCart();
  alert('Aankoop bevestigd');
};

// ===== Admin dashboard =====
function openAdminDashboard() {
  $('#homeScreen').classList.add('hidden');
  $('#adminDashboard').classList.remove('hidden');
  renderAdminAccounts();
  renderAdminProducts();
  renderLogView();
}
$('#btnAdminBack').onclick = () => { currentUser=null; renderHome(); };

function renderAdminAccounts() {
  const list = $('#adminAccList');
  const sel = $('#selFunds');
  list.innerHTML = '';
  sel.innerHTML = '';
  users.forEach((u, idx) => {
    const card = document.createElement('div');
    card.className = 'card';
    card.innerHTML = `
      <div class="row" style="justify-content:space-between">
        <div>
          <div style="font-weight:700">${u.name}</div>
          <div class="badge">${u.type==='vast'?'Vast (-10)':'Gast (0)'}</div>
          <div class="badge">Saldo: €${u.balance.toFixed(2)}</div>
        </div>
        <div class="row">
          <button data-i="${idx}">Bewerk</button>
        </div>
      </div>`;
    card.querySelector('button').onclick = () => editAccount(idx);
    list.appendChild(card);

    const opt = document.createElement('option');
    opt.value = idx; opt.textContent = `${u.name} — €${u.balance.toFixed(2)}`;
    sel.appendChild(opt);
  });
}

function editAccount(idx) {
  const u = users[idx];
  const name = prompt('Naam:', u.name);
  if (name===null) return; // cancel
  const pin = prompt('PIN (4 cijfers):', u.pin);
  if (pin===null) return;
  const type = prompt('Type (vast/gast):', u.type);
  if (type===null) return;
  const bal = prompt('Saldo:', u.balance);
  if (bal===null) return;
  u.name = name.trim()||u.name;
  u.pin = pin.trim();
  u.type = (type==='vast'||type==='gast')?type:u.type;
  u.balance = parseFloat(bal)||0;
  saveAll();
  renderAdminAccounts();
  renderHome();
}

$('#btnAddAcc').onclick = () => {
  const name = $('#accName').value.trim();
  const pin  = $('#accPin').value.trim();
  const type = $('#accType').value;
  const start= parseFloat($('#accStart').value||'0');
  if (!name || !pin) return alert('Naam en PIN verplicht');
  users.push({ name, pin, balance: isNaN(start)?0:start, type });
  $('#accName').value=''; $('#accPin').value=''; $('#accStart').value='';
  saveAll();
  renderAdminAccounts();
  renderHome();
};

$('#btnAddFunds').onclick = () => {
  const idx = parseInt($('#selFunds').value,10);
  const amt = parseFloat($('#addAmount').value||'0');
  if (isNaN(idx) || isNaN(amt) || amt<=0) return alert('Kies gebruiker en bedrag');
  users[idx].balance = +(users[idx].balance + amt).toFixed(2);
  logs.push({ ts:new Date().toISOString(), type:'funds', user:users[idx].name, details:'Saldo opgehoogd', amount:amt });
  $('#addAmount').value='';
  saveAll();
  renderAdminAccounts();
  renderHome();
};

function renderAdminProducts() {
  const list = $('#adminProdList');
  list.innerHTML = '';
  products.forEach((p, idx) => {
    const card = document.createElement('div');
    card.className = 'card';
    card.innerHTML = `
      <div class="row" style="justify-content:space-between">
        <div><strong>${p.name}</strong><div class="badge">€${p.price.toFixed(2)} • Voorraad: ${p.stock}</div></div>
        <div class="row">
          <button data-i="${idx}">Bewerk</button>
          <button data-d="${idx}" class="secondary">Verwijder</button>
        </div>
      </div>`;
    card.querySelector('button[data-i]').onclick = () => editProduct(idx);
    card.querySelector('button[data-d]').onclick = () => deleteProduct(idx);
    list.appendChild(card);
  });
}

$('#btnAddProd').onclick = () => {
  const name = $('#prodName').value.trim();
  const price = parseFloat($('#prodPrice').value||'0');
  const stock = parseInt($('#prodStock').value||'0',10);
  if (!name || price<=0 || stock<0) return alert('Onjuiste productgegevens');
  const id = products.length ? Math.max(...products.map(p=>p.id||0))+1 : 1;
  products.push({ id, name, price, stock });
  $('#prodName').value=''; $('#prodPrice').value=''; $('#prodStock').value='';
  saveAll();
  renderAdminProducts();
};

function editProduct(idx) {
  const p = products[idx];
  const name = prompt('Naam:', p.name); if (name===null) return;
  const price = prompt('Prijs:', p.price); if (price===null) return;
  const stock = prompt('Voorraad:', p.stock); if (stock===null) return;
  p.name = name.trim()||p.name;
  p.price = parseFloat(price)||p.price;
  p.stock = parseInt(stock,10); if (isNaN(p.stock)) p.stock = 0;
  saveAll();
  renderAdminProducts();
}
function deleteProduct(idx) {
  if (!confirm('Product verwijderen?')) return;
  products.splice(idx,1);
  saveAll();
  renderAdminProducts();
}

// ===== Logs =====
function renderLogView() {
  const v = $('#logView');
  if (!logs.length) { v.innerHTML = '<em>Geen logregels</em>'; return; }
  v.innerHTML = logs.slice().reverse().map(l => {
    const when = new Date(l.ts).toLocaleString('nl-NL');
    const amt = (typeof l.amount==='number')?` — €${l.amount.toFixed(2)}`:'';
    return `<div>[${when}] ${l.user} • ${l.type} • ${l.details}${amt}</div>`;
  }).join('');
}

$('#btnExportCsv').onclick = () => {
  const header = ['tijdstip','type','gebruiker','details','bedrag'];
  const rows = logs.map(l => [l.ts, l.type, l.user, l.details, (l.amount??'')]);
  const csv = [header, ...rows].map(r => r.map(v => `"${String(v).replaceAll('"','""')}"`).join(',')).join('\n');
  const blob = new Blob([csv], {type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'wallet-log.csv'; a.click(); URL.revokeObjectURL(url);
};

// Admin pin change
$('#btnSetAdminPin').onclick = () => {
  const val = $('#adminPinInput').value.trim();
  if (!val) return alert('Voer een nieuwe PIN in');
  adminPin = val; $('#adminPinInput').value=''; saveAll(); alert('Admin PIN bijgewerkt');
};

// ===== Init =====
loadAll();
saveAll(); // persist seeds if used
renderHome();
</script></body>
</html>
