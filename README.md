<!DOCTYPE html><html lang="nl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Fictief Geld Systeem</title>
<style>
  :root{
    --bg1:#f0f4f8;--bg2:#d9e2ec;--pri:#4a90e2;--pri-700:#3b7dc4;--danger:#e94e4e;--danger-700:#c43d3d;--text:#333;--muted:#667;--card:#fff;--border:#e6ecf3;
  }
  *{box-sizing:border-box}
  body{font-family:Arial,system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;background:linear-gradient(135deg,var(--bg1),var(--bg2));margin:0;color:var(--text)}
  header{background:var(--pri);color:#fff;padding:15px;text-align:center;font-size:1.4em;box-shadow:0 2px 6px rgba(0,0,0,.2)}
  .container{max-width:720px;margin:20px auto;background:var(--card);padding:20px;border-radius:12px;box-shadow:0 4px 12px rgba(0,0,0,.08)}
  h2{margin-top:0}
  .row{display:flex;gap:10px;align-items:center;flex-wrap:wrap}
  .grow{flex:1}
  button{padding:10px 14px;border:0;background:var(--pri);color:#fff;border-radius:8px;cursor:pointer;font-size:1em;transition:.2s background,.2s transform}
  button:hover{background:var(--pri-700)}
  button.ghost{background:#f5f7fa;color:#345;border:1px solid var(--border)}
  button.red{background:var(--danger)}
  button.red:hover{background:var(--danger-700)}
  input,select{padding:10px;width:100%;border:1px solid var(--border);border-radius:8px;font-size:1em}
  .hidden{display:none}
  .item{border-bottom:1px solid var(--border);padding:8px 0;display:flex;justify-content:space-between;align-items:center}
  .item:last-child{border-bottom:none}
  .muted{color:var(--muted)}
  .grid{display:grid;gap:10px;grid-template-columns:repeat(auto-fit,minmax(220px,1fr))}
  .pill{padding:.2rem .6rem;border-radius:999px;border:1px solid var(--border);background:#f5f7fa;color:#345;font-size:.8rem}
  .pill.red{background:#ffe9e9;color:#b40000}
  .table{width:100%;border-collapse:collapse;border:1px solid var(--border);border-radius:8px;overflow:hidden}
  .table th,.table td{padding:8px 10px;border-bottom:1px solid var(--border);text-align:left}
  .table th{background:#f5f7fa}
</style>
</head>
<body>
<header>💰 Fictief Geld Systeem</header><!-- Hoofdpagina --><div class="container" id="homeScreen">
  <div class="row"><h2 class="grow">Kies je account</h2>
    <div class="row" style="gap:6px">
      <input type="password" id="adminCode" placeholder="Admin pincode" maxlength="4" style="width:150px" inputmode="numeric" pattern="[0-9]*" />
      <button onclick="adminLogin()">Admin inloggen</button>
    </div>
  </div>
  <div id="accountButtons" class="grid"></div>
</div><!-- PIN scherm --><div class="container hidden" id="pinScreen">
  <div class="row"><h2 class="grow">Inloggen</h2><button class="red ghost" onclick="goHome()">Annuleren</button></div>
  <p id="selectedUserName" class="muted"></p>
  <input type="password" id="pincode" placeholder="4-cijferige pincode" maxlength="4" inputmode="numeric" pattern="[0-9]*" />
  <div class="row">
    <button onclick="checkLogin()">Inloggen</button>
    <button class="ghost" onclick="clearPin()">Wissen</button>
  </div>
</div><!-- Gebruiker omgeving --><div class="container hidden" id="userScreen">
  <div class="row"><h2 id="welcome" class="grow"></h2><button class="red" onclick="logout()">Uitloggen</button></div>
  <p>Saldo: €<span id="saldo"></span></p>
  <div id="productList"></div>
</div><!-- Admin omgeving --><div class="container hidden" id="adminScreen">
  <div class="row"><h2 class="grow">Admin Paneel</h2><button class="red" onclick="logout()">Uitloggen</button></div>  <h3>Accounts</h3>
  <div class="grid">
    <input id="newName" placeholder="Naam" />
    <input id="newPin" placeholder="Pincode (max 4)" maxlength="4" inputmode="numeric" pattern="[0-9]*" />
    <input type="number" id="newSaldo" placeholder="Startsaldo (bv 10)" />
    <select id="newType"><option value="gast">Gast</option><option value="vast">Vast</option></select>
    <button onclick="addAccount()">Account toevoegen</button>
  </div>
  <div id="accountList"></div>  <h3 style="margin-top:18px">Producten & Voorraad</h3>
  <div class="grid">
    <input id="prodName" placeholder="Productnaam (bv Chips)" />
    <input id="prodPrice" type="number" step="0.01" placeholder="Prijs (bv 0.75)" />
    <input id="prodStock" type="number" step="1" placeholder="Startvoorraad (bv 20)" />
    <button onclick="addProduct()">Product toevoegen</button>
  </div>
  <div id="productAdminList"></div>  <h3 style="margin-top:18px">Logboek</h3>
  <div id="logContainer"></div>
  <div class="row" style="margin-top:8px">
    <button class="ghost" onclick="exportCSV()">Exporteer CSV</button>
    <button class="ghost" onclick="clearLogs()">Log wissen</button>
  </div>
</div><script>
  // ---- Data ----
  let adminPIN = "9999";
  let accounts = JSON.parse(localStorage.getItem("accounts")) || [
    {name:"Jan", pin:"1234", saldo:10.00, type:"vast"},
    {name:"Piet", pin:"5678", saldo:5.00, type:"gast"}
  ];
  let products = JSON.parse(localStorage.getItem("products")) || [
    {name:"Chips", price:0.75, stock:20},
    {name:"Bier", price:0.75, stock:20},
    {name:"Cola", price:1.00, stock:20}
  ];
  let logs = JSON.parse(localStorage.getItem("logs")) || [];
  let currentUserIndex = null;

  function saveData(){
    localStorage.setItem("accounts", JSON.stringify(accounts));
    localStorage.setItem("products", JSON.stringify(products));
    localStorage.setItem("logs", JSON.stringify(logs));
  }

  // ---- Helpers ----
  function fmt2(n){ return (Math.round(n*100)/100).toFixed(2); }
  function ts(){ return new Date().toISOString(); }
  function tsNice(iso){ const d=new Date(iso); return d.toLocaleString('nl-NL'); }

  // ---- Home ----
  function loadAccountButtons(){
    const grid = document.getElementById('accountButtons');
    grid.innerHTML='';
    accounts.forEach((acc,i)=>{
      const card = document.createElement('div');
      card.className='container';
      card.style.padding='14px';
      card.style.margin='0';
      card.style.border='1px solid var(--border)';
      if(acc.saldo<0){ card.style.background='#ffe9e9'; }
      const row = document.createElement('div'); row.className='row';
      const left = document.createElement('div'); left.className='grow'; left.innerHTML = `<strong>${acc.name}</strong><br><span class="muted">Saldo: €${fmt2(acc.saldo)} · <span class="pill ${acc.saldo<0?'red':''}">${acc.type}</span></span>`;
      const btn = document.createElement('button'); btn.textContent='Openen'; btn.onclick=()=>selectAccount(i);
      row.appendChild(left); row.appendChild(btn); card.appendChild(row); grid.appendChild(card);
    });
  }

  function selectAccount(index){
    currentUserIndex=index; clearPin();
    document.getElementById('homeScreen').classList.add('hidden');
    document.getElementById('pinScreen').classList.remove('hidden');
    document.getElementById('selectedUserName').textContent = 'Account: '+accounts[index].name;
  }

  function clearPin(){ document.getElementById('pincode').value=''; }
  function goHome(){ clearPin(); showOnly('homeScreen'); }
  function showOnly(id){ ['homeScreen','pinScreen','userScreen','adminScreen'].forEach(x=>document.getElementById(x).classList.toggle('hidden', x!==id)); }

  function checkLogin(){
    const pin = document.getElementById('pincode').value;
    if(accounts[currentUserIndex].pin===pin){
      clearPin();
      document.getElementById('welcome').textContent = 'Welkom '+accounts[currentUserIndex].name;
      updateUserScreen();
      showOnly('userScreen');
    } else { alert('Verkeerde pincode!'); }
  }

  function logout(){ currentUserIndex=null; showOnly('homeScreen'); }

  // ---- User screen ----
  function updateUserScreen(){
    const acc = accounts[currentUserIndex];
    document.getElementById('saldo').textContent = fmt2(acc.saldo);
    const list = document.getElementById('productList'); list.innerHTML='';
    products.forEach((p,i)=>{
      const div=document.createElement('div'); div.className='item';
      const stockInfo = p.stock>0 ? `<span class="pill">Voorraad: ${p.stock}</span>` : `<span class="pill red">Uitverkocht</span>`;
      const left = document.createElement('div'); left.innerHTML = `${p.name} - €${fmt2(p.price)} ${stockInfo}`;
      const buy = document.createElement('button'); buy.textContent='Koop'; buy.disabled = p.stock<=0; buy.onclick=()=>buyProduct(i);
      div.appendChild(left); div.appendChild(buy); list.appendChild(div);
    });
  }

  function buyProduct(i){
    const acc = accounts[currentUserIndex];
    const p = products[i];
    // Vraag aantal
    let qtyStr = prompt(`Hoeveel '${p.name}' wil je kopen? (voorraad: ${p.stock})`, '1');
    if(qtyStr===null) return; // cancel
    let qty = parseInt(qtyStr,10);
    if(isNaN(qty) || qty<=0){ alert('Ongeldig aantal.'); return; }
    if(qty>p.stock){ alert('Niet genoeg voorraad.'); return; }

    const totaal = p.price*qty;
    // Regels gast/vast
    if(acc.type==='gast' && acc.saldo - totaal < 0){ alert('Gast mag niet onder €0 komen!'); return; }
    if(acc.type==='vast' && acc.saldo - totaal < -10){ alert('Vast mag niet verder dan -€10 komen!'); return; }

    if(!confirm(`Bevestigen?
${p.name} × ${qty} = €${fmt2(totaal)}
Nieuw saldo: €${fmt2(acc.saldo - totaal)}`)) return;

    // Doorvoeren
    acc.saldo -= totaal;
    p.stock -= qty;
    logs.push({t:ts(), user:acc.name, product:p.name, qty, price:p.price, total:totaal, balanceAfter:acc.saldo});
    saveData();

    updateUserScreen();
    loadAccountButtons();
    renderLog();
  }

  // ---- Admin ----
  function adminLogin(){
    const pin = document.getElementById('adminCode').value;
    if(pin===adminPIN){
      document.getElementById('adminCode').value='';
      renderAdmin();
      showOnly('adminScreen');
    } else { alert('Verkeerde admin pincode!'); }
  }

  function renderAdmin(){ updateAdminAccounts(); updateAdminProducts(); renderLog(); }

  function updateAdminAccounts(){
    const box = document.getElementById('accountList'); box.innerHTML='';
    accounts.forEach((acc,i)=>{
      const row=document.createElement('div'); row.className='item';
      row.innerHTML = `<span><strong>${acc.name}</strong> (€${fmt2(acc.saldo)}) [${acc.type}]</span>`;
      const controls=document.createElement('div');
      controls.innerHTML = `
        <button class="ghost" onclick="addSaldo(${i})">+€</button>
        <button class="ghost" onclick="editAccount(${i})">Bewerken</button>
        <button class="red" onclick="deleteAccount(${i})">Verwijderen</button>`;
      row.appendChild(controls); box.appendChild(row);
    });
  }

  function addAccount(){
    const name = document.getElementById('newName').value.trim();
    const pin = document.getElementById('newPin').value.trim();
    const saldo = parseFloat(document.getElementById('newSaldo').value);
    const type = document.getElementById('newType').value;
    if(!name || !pin || isNaN(saldo)){ alert('Vul alle velden in.'); return; }
    if(pin.length>4){ alert('Pincode max 4 cijfers.'); return; }
    accounts.push({name, pin, saldo, type}); saveData();
    ['newName','newPin','newSaldo'].forEach(id=>document.getElementById(id).value='');
    updateAdminAccounts(); loadAccountButtons();
  }

  function deleteAccount(i){ if(confirm('Account verwijderen?')){ accounts.splice(i,1); saveData(); updateAdminAccounts(); loadAccountButtons(); } }
  function addSaldo(i){ const v=parseFloat(prompt('Bedrag toevoegen (bv 5):','5')); if(!isNaN(v)){ accounts[i].saldo += v; saveData(); updateAdminAccounts(); loadAccountButtons(); }}

  function editAccount(i){
    const acc = accounts[i];
    const name = prompt('Nieuwe naam (leeg = ongewijzigd):', acc.name) ?? '';
    const type = prompt("Type: 'gast' of 'vast' (leeg = ongewijzigd):", acc.type) ?? '';
    const pin = prompt('Nieuwe pincode (leeg = ongewijzigd):','') ?? '';
    const saldoStr = prompt('Nieuw saldo (leeg = ongewijzigd):','');
    if(name.trim()) acc.name=name.trim();
    if(type.trim()==='gast' || type.trim()==='vast') acc.type=type.trim();
    if(pin.trim()){ if(pin.trim().length>4){ alert('Pincode max 4.'); } else { acc.pin=pin.trim(); } }
    if(saldoStr && !isNaN(parseFloat(saldoStr))) acc.saldo=parseFloat(saldoStr);
    saveData(); updateAdminAccounts(); loadAccountButtons(); renderLog();
  }

  function updateAdminProducts(){
    const box=document.getElementById('productAdminList'); box.innerHTML='';
    products.forEach((p,i)=>{
      const row=document.createElement('div'); row.className='item';
      row.innerHTML = `<span><strong>${p.name}</strong> · €${fmt2(p.price)} · Voorraad: ${p.stock}</span>`;
      const ctr=document.createElement('div');
      ctr.innerHTML = `
        <button class="ghost" onclick="stockPlus(${i})">+1</button>
        <button class="ghost" onclick="stockMin(${i})">-1</button>
        <button class="ghost" onclick="setStock(${i})">Set</button>
        <button class="ghost" onclick="editProduct(${i})">Bewerken</button>
        <button class="red" onclick="deleteProduct(${i})">Verwijderen</button>`;
      row.appendChild(ctr); box.appendChild(row);
    });
  }

  function addProduct(){
    const name=document.getElementById('prodName').value.trim();
    const price=parseFloat(document.getElementById('prodPrice').value);
    const stock=parseInt(document.getElementById('prodStock').value||'0',10);
    if(!name || isNaN(price)){ alert('Naam en prijs zijn verplicht.'); return; }
    products.push({name, price, stock:isNaN(stock)?0:stock}); saveData();
    ['prodName','prodPrice','prodStock'].forEach(id=>document.getElementById(id).value='');
    updateAdminProducts(); updateUserScreen();
  }
  function editProduct(i){
    const p = products[i];
    const name = prompt('Nieuwe naam (leeg = ongewijzigd):', p.name) ?? '';
    const priceStr = prompt('Nieuwe prijs (leeg = ongewijzigd):', p.price) ?? '';
    const stockStr = prompt('Nieuwe voorraad (leeg = ongewijzigd):', p.stock) ?? '';
    if(name.trim()) p.name=name.trim();
    if(priceStr && !isNaN(parseFloat(priceStr))) p.price=parseFloat(priceStr);
    if(stockStr && !isNaN(parseInt(stockStr,10))) p.stock=parseInt(stockStr,10);
    saveData(); updateAdminProducts(); updateUserScreen();
  }
  function deleteProduct(i){ if(confirm('Product verwijderen?')){ products.splice(i,1); saveData(); updateAdminProducts(); updateUserScreen(); } }
  function stockPlus(i){ products[i].stock=(products[i].stock||0)+1; saveData(); updateAdminProducts(); updateUserScreen(); }
  function stockMin(i){ if((products[i].stock||0)>0){ products[i].stock-=1; saveData(); updateAdminProducts(); updateUserScreen(); } }
  function setStock(i){ const v=parseInt(prompt('Voorraad instellen op:', products[i].stock||0),10); if(!isNaN(v)&&v>=0){ products[i].stock=v; saveData(); updateAdminProducts(); updateUserScreen(); } }

  // ---- Log ----
  function renderLog(){
    const box=document.getElementById('logContainer');
    if(!box) return; box.innerHTML='';
    if(logs.length===0){ box.innerHTML='<p class="muted">Nog geen transacties.</p>'; return; }
    const table=document.createElement('table'); table.className='table';
    table.innerHTML='<thead><tr><th>Datum</th><th>Gebruiker</th><th>Product</th><th>Aantal</th><th>Prijs</th><th>Totaal</th><th>Saldo na</th></tr></thead><tbody></tbody>';
    const tb=table.querySelector('tbody');
    logs.slice().reverse().forEach(l=>{
      const tr=document.createElement('tr');
      tr.innerHTML = `<td>${tsNice(l.t)}</td><td>${l.user}</td><td>${l.product}</td><td>${l.qty}</td><td>€${fmt2(l.price)}</td><td>€${fmt2(l.total)}</td><td>€${fmt2(l.balanceAfter)}</td>`;
      tb.appendChild(tr);
    });
    box.appendChild(table);
  }
  function exportCSV(){
    const rows = [['datum','gebruiker','product','aantal','prijs','totaal','saldo_na']];
    logs.forEach(l=> rows.push([l.t,l.user,l.product,l.qty,fmt2(l.price),fmt2(l.total),fmt2(l.balanceAfter)]));
    const csv = rows.map(r=> r.map(v=> '"'+String(v).replaceAll('"','""')+'"').join(',')).join('\n');
    const blob = new Blob([csv],{type:'text/csv'}); const url=URL.createObjectURL(blob);
    const a=document.createElement('a'); a.href=url; a.download='log.csv'; a.click(); setTimeout(()=>URL.revokeObjectURL(url),1000);
  }
  function clearLogs(){ if(confirm('Hele log wissen?')){ logs=[]; saveData(); renderLog(); } }

  // ---- Init ----
  loadAccountButtons();
</script></body>
</html>
