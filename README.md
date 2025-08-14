<!DOCTYPE html><html lang="nl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Fictief Geld Systeem</title>
<style>
  :root{
    --bg1:#f5f7fb;--bg2:#e8eef6;--pri:#2457d6;--pri-700:#1744b2;--accent:#00a389;--danger:#e5484d;--danger-700:#c03439;--text:#1e293b;--muted:#5b6b83;--card:#ffffff;--border:#e5e9f2;
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{font-family:Inter,ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;background:linear-gradient(135deg,var(--bg1),var(--bg2));margin:0;color:var(--text)}
  header{background:var(--card);border-bottom:1px solid var(--border);position:sticky;top:0;z-index:5}
  .wrap{max-width:1060px;margin:0 auto;padding:10px 16px;display:flex;align-items:center;gap:12px}
  .brand{display:flex;align-items:center;gap:10px}
  .brand img{height:42px;width:42px;object-fit:contain;border-radius:8px;border:1px solid var(--border);background:#fff}
  .brand h1{font-size:1.1rem;margin:0}
  .badge{font-size:.75rem;color:var(--muted);}
  .spacer{flex:1}
  .navbtn{padding:8px 12px;border:1px solid var(--border);background:#f7f9fd;border-radius:10px;cursor:pointer}.container{max-width:1060px;margin:18px auto;background:var(--card);padding:18px;border-radius:14px;box-shadow:0 8px 30px rgba(15,23,42,.06);border:1px solid var(--border)} h2{margin:0 0 8px} .row{display:flex;gap:10px;align-items:center;flex-wrap:wrap} .grow{flex:1} button{padding:10px 14px;border:0;background:var(--pri);color:#fff;border-radius:10px;cursor:pointer;font-weight:600;transition:.2s background,.2s transform} button:hover{background:var(--pri-700)} button.ghost{background:#f5f7fa;color:#213;border:1px solid var(--border)} button.red{background:var(--danger)} button.red:hover{background:var(--danger-700)} input,select{padding:10px;width:100%;border:1px solid var(--border);border-radius:10px;font-size:1rem} .hidden{display:none} .muted{color:var(--muted)} .grid{display:grid;gap:12px;grid-template-columns:repeat(auto-fit,minmax(240px,1fr))} .pill{padding:.2rem .55rem;border-radius:999px;border:1px solid var(--border);background:#f5f7fa;color:#334155;font-size:.8rem} .pill.red{background:#ffe9e9;color:#b40000} .item{border-bottom:1px solid var(--border);padding:10px 0;display:flex;justify-content:space-between;align-items:center} .item:last-child{border-bottom:none} .table{width:100%;border-collapse:collapse;border:1px solid var(--border);border-radius:12px;overflow:hidden} .table th,.table td{padding:9px 10px;border-bottom:1px solid var(--border);text-align:left} .table th{background:#f5f7fa} .card{border:1px solid var(--border);border-radius:14px;padding:14px;background:var(--card)} .qty{display:inline-flex;align-items:center;border:1px solid var(--border);border-radius:10px;overflow:hidden} .qty button{background:#f7f9fd;color:#111;border-radius:0;border-right:1px solid var(--border)} .qty input{width:56px;text-align:center;border:0} .total{font-weight:800} .accent{color:var(--accent)} </style>

</head>
<body>
<header>
  <div class="wrap">
    <div class="brand">
      <img id="brandLogo" alt="logo" />
      <div>
        <h1 id="brandTitle">Fictief Geld Systeem</h1>
        <div class="badge">Client‑side demo (localStorage)</div>
      </div>
    </div>
    <div class="spacer"></div>
    <button class="navbtn" onclick="showOnly('homeScreen')">Home</button>
    <button class="navbtn" onclick="adminLogin()">Admin</button>
  </div>
</header><!-- Hoofdpagina --><div class="container" id="homeScreen">
  <div class="row"><h2 class="grow">Kies je account</h2>
    <div class="row" style="gap:6px">
      <input type="password" id="adminCode" placeholder="Admin pincode" maxlength="4" style="width:160px" inputmode="numeric" pattern="[0-9]*" />
      <button onclick="adminLogin()">Inloggen</button>
    </div>
  </div>
  <div id="accountButtons" class="grid"></div>
</div><!-- PIN scherm --><div class="container hidden" id="pinScreen">
  <div class="row"><h2 class="grow">Inloggen</h2><button class="red ghost" onclick="goHome()">Annuleren</button></div>
  <p id="selectedUserName" class="muted"></p>
  <input type="password" id="pincode" placeholder="4-cijferige pincode" maxlength="4" inputmode="numeric" pattern="[0-9]*" />
  <div class="row"><button onclick="checkLogin()">Verder</button><button class="ghost" onclick="clearPin()">Wissen</button></div>
</div><!-- Gebruiker omgeving --><div class="container hidden" id="userScreen">
  <div class="row"><h2 id="welcome" class="grow"></h2><button class="red" onclick="logout()">Uitloggen</button></div>
  <p>Saldo: €<span id="saldo"></span></p>  <div class="grid">
    <div class="card">
      <h3>Producten</h3>
      <div id="productList"></div>
    </div>
    <div class="card">
      <h3>Winkelwagen</h3>
      <div id="cartList" class="item muted">Nog leeg…</div>
      <div class="row" style="margin-top:8px">
        <div class="grow"><span class="muted">Totaal:</span> <span class="total" id="cartTotal">€0,00</span></div>
        <button class="ghost" onclick="clearCart()">Leeg</button>
        <button onclick="checkout()">Afrekenen</button>
      </div>
      <div id="cartNote" class="muted" style="margin-top:6px"></div>
    </div>
  </div>
</div><!-- Admin omgeving --><div class="container hidden" id="adminScreen">
  <div class="row"><h2 class="grow">Admin Paneel</h2><button class="red" onclick="logout()">Uitloggen</button></div>  <div class="grid">
    <div class="card">
      <h3>Accounts</h3>
      <div class="grid">
        <input id="newName" placeholder="Naam" />
        <input id="newPin" placeholder="Pincode (max 4)" maxlength="4" inputmode="numeric" pattern="[0-9]*" />
        <input type="number" id="newSaldo" placeholder="Startsaldo (bv 10)" />
        <select id="newType"><option value="gast">Gast</option><option value="vast">Vast</option></select>
        <button onclick="addAccount()">Account toevoegen</button>
      </div>
      <div id="accountList" style="margin-top:8px"></div>
    </div><div class="card">
  <h3>Producten & Voorraad</h3>
  <div class="grid">
    <input id="prodName" placeholder="Productnaam (bv Chips)" />
    <input id="prodPrice" type="number" step="0.01" placeholder="Prijs (bv 0.75)" />
    <input id="prodStock" type="number" step="1" placeholder="Voorraad (bv 20)" />
    <button onclick="addProduct()">Product toevoegen</button>
  </div>
  <div id="productAdminList" style="margin-top:8px"></div>
</div>

<div class="card">
  <h3>Logboek</h3>
  <div id="logContainer"></div>
  <div class="row" style="margin-top:8px">
    <button class="ghost" onclick="exportCSV()">Exporteer CSV</button>
    <button class="ghost" onclick="clearLogs()">Log wissen</button>
  </div>
</div>

<div class="card">
  <h3>Huisstijl</h3>
  <div class="grid">
    <div>
      <label class="muted">Logo upload</label>
      <input type="file" accept="image/*" id="logoInput" />
    </div>
    <div>
      <label class="muted">Titel</label>
      <input id="brandInput" placeholder="Bijv. Waterscouting St Willibrordus Gouda" />
    </div>
    <button class="ghost" onclick="saveBranding()">Opslaan</button>
  </div>
  <p class="muted" style="margin-top:6px">Logo en titel worden lokaal bewaard en verschijnen bovenin.</p>
</div>

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
  let branding = JSON.parse(localStorage.getItem("branding")) || {title:"Fictief Geld Systeem", logo:null};

  let currentUserIndex = null;
  const cart = new Map(); // productIndex -> qty

  function saveData(){
    localStorage.setItem("accounts", JSON.stringify(accounts));
    localStorage.setItem("products", JSON.stringify(products));
    localStorage.setItem("logs", JSON.stringify(logs));
    localStorage.setItem("branding", JSON.stringify(branding));
  }

  // ---- Helpers ----
  function fmt2(n){ return (Math.round(n*100)/100).toFixed(2); }
  function ts(){ return new Date().toISOString(); }
  function tsNice(iso){ const d=new Date(iso); return d.toLocaleString('nl-NL'); }

  // ---- Branding ----
  function applyBranding(){
    document.getElementById('brandTitle').textContent = branding.title || 'Fictief Geld Systeem';
    const img = document.getElementById('brandLogo');
    if(branding.logo){ img.src = branding.logo; img.style.display='block'; } else { img.style.display='none'; }
    const inp = document.getElementById('brandInput'); if(inp) inp.value = branding.title || '';
  }
  const logoInput = document.getElementById('logoInput');
  if(logoInput){ logoInput.addEventListener('change', async (e)=>{
    const f=e.target.files[0]; if(!f) return; const r=new FileReader(); r.onload=()=>{ branding.logo=r.result; saveData(); applyBranding(); }; r.readAsDataURL(f);
  }); }
  function saveBranding(){ const t=document.getElementById('brandInput').value.trim(); if(t){ branding.title=t; saveData(); applyBranding(); } }

  // ---- Home ----
  function loadAccountButtons(){
    const grid = document.getElementById('accountButtons');
    grid.innerHTML='';
    accounts.forEach((acc,i)=>{
      const card = document.createElement('div');
      card.className='card';
      if(acc.saldo<0){ card.style.background='#fff1f1'; }
      const row = document.createElement('div'); row.className='row';
      const left = document.createElement('div'); left.className='grow'; left.innerHTML = `<strong>${acc.name}</strong><br><span class=\"muted\">Saldo: €${fmt2(acc.saldo)} · <span class=\"pill ${acc.saldo<0?'red':''}\">${acc.type}</span></span>`;
      const btn = document.createElement('button'); btn.textContent='Openen'; btn.onclick=()=>selectAccount(i);
      row.appendChild(left); row.appendChild(btn); card.appendChild(row); grid.appendChild(card);
    });
  }

  function selectAccount(index){
    currentUserIndex=index; clearPin();
    showOnly('pinScreen');
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
      renderProducts(); renderCart();
      showOnly('userScreen');
    } else { alert('Verkeerde pincode!'); }
  }

  function logout(){ currentUserIndex=null; cart.clear(); renderCart(); showOnly('homeScreen'); }

  // ---- Products + Cart (User) ----
  function renderProducts(){
    const list = document.getElementById('productList'); list.innerHTML='';
    products.forEach((p,i)=>{
      const row = document.createElement('div'); row.className='item';
      const stockInfo = p.stock>0 ? `<span class=\"pill\">Voorraad: ${p.stock}</span>` : `<span class=\"pill red\">Uitverkocht</span>`;
      const left = document.createElement('div'); left.innerHTML = `<strong>${p.name}</strong> · €${fmt2(p.price)} ${stockInfo}`;

      const qty = document.createElement('div'); qty.className='qty';
      const minus = document.createElement('button'); minus.textContent='−'; minus.onclick=()=>{ const v=Number(inp.value||1); inp.value=Math.max(1,v-1); };
      const inp = document.createElement('input'); inp.type='number'; inp.step='1'; inp.min='1'; inp.value='1';
      const plus = document.createElement('button'); plus.textContent='+'; plus.onclick=()=>{ const v=Number(inp.value||1); inp.value=v+1; };
      qty.appendChild(minus); qty.appendChild(inp); qty.appendChild(plus);

      const add = document.createElement('button'); add.textContent='In wagen'; add.disabled = p.stock<=0; add.onclick=()=>{
        const q = Math.max(1, parseInt(inp.value||'1',10));
        if(q>p.stock){ alert('Niet genoeg voorraad.'); return; }
        cart.set(i, (cart.get(i)||0)+q); renderCart();
      };

      const right = document.createElement('div'); right.className='row'; right.appendChild(qty); right.appendChild(add);
      row.appendChild(left); row.appendChild(right); list.appendChild(row);
    });
  }

  function cartTotal(){ let t=0; for(const [i,q] of cart){ const p=products[i]; t += p.price*q; } return t; }

  function renderCart(){
    const box = document.getElementById('cartList'); box.innerHTML='';
    if(cart.size===0){ box.textContent='Nog leeg…'; }
    else{
      const tbl=document.createElement('table'); tbl.className='table';
      tbl.innerHTML='<thead><tr><th>Product</th><th>Aantal</th><th>Prijs</th><th>Subtotaal</th><th></th></tr></thead><tbody></tbody>';
      const tb=tbl.querySelector('tbody');
      for(const [i,q] of cart){ const p=products[i]; const tr=document.createElement('tr');
        tr.innerHTML = `<td>${p.name}</td><td>${q}</td><td>€${fmt2(p.price)}</td><td>€${fmt2(p.price*q)}</td>`;
        const tdDel=document.createElement('td'); const del=document.createElement('button'); del.className='ghost'; del.textContent='Verwijder'; del.onclick=()=>{ cart.delete(i); renderCart(); }; tdDel.appendChild(del); tr.appendChild(tdDel);
        tb.appendChild(tr);
      }
      box.appendChild(tbl);
    }
    document.getElementById('cartTotal').textContent = '€'+fmt2(cartTotal());
    updateCartNote();
  }

  function updateCartNote(){
    const note = document.getElementById('cartNote');
    const acc = accounts[currentUserIndex]; if(!acc){ note.textContent=''; return; }
    const total = cartTotal(); const newBal = acc.saldo - total;
    if(acc.type==='gast' && newBal<0){ note.textContent = 'Gast mag niet onder €0 komen.'; }
    else if(acc.type==='vast' && newBal<-10){ note.textContent = 'Vast mag niet verder dan −€10.'; }
    else note.textContent='';
  }

  function clearCart(){ cart.clear(); renderCart(); }

  function checkout(){
    if(cart.size===0) return;
    const acc = accounts[currentUserIndex];
    // Controleer voorraad + regels
    let lines=[]; let total=0; for(const [i,q] of cart){ const p=products[i]; if(q>p.stock){ alert('Onvoldoende voorraad voor '+p.name); return; } total += p.price*q; lines.push(`${p.name}×${q}`); }
    const newBal = acc.saldo - total;
    if(acc.type==='gast' && newBal<0){ alert('Gast mag niet onder €0 komen.'); return; }
    if(acc.type==='vast' && newBal<-10){ alert('Vast mag niet verder dan −€10.'); return; }

    if(!confirm(`Bevestigen?
${lines.join(', ')}
Totaal: €${fmt2(total)}
Nieuw saldo: €${fmt2(newBal)}`)) return;

    // Doorvoeren
    acc.saldo = newBal;
    for(const [i,q] of cart){ products[i].stock -= q; }
    logs.push({t:ts(), user:acc.name, items:Array.from(cart,([i,q])=>({name:products[i].name, qty:q, price:products[i].price})), total, balanceAfter:acc.saldo});
    saveData();

    cart.clear(); renderProducts(); renderCart(); loadAccountButtons(); renderLog();
  }

  // ---- Admin ----
  function adminLogin(){
    const pin = document.getElementById('adminCode').value;
    if(pin===adminPIN){ document.getElementById('adminCode').value=''; renderAdmin(); showOnly('adminScreen'); }
    else { alert('Verkeerde admin pincode!'); }
  }

  function renderAdmin(){ updateAdminAccounts(); updateAdminProducts(); renderLog(); applyBranding(); }

  function updateAdminAccounts(){
    const box = document.getElementById('accountList'); box.innerHTML='';
    accounts.forEach((acc,i)=>{
      const row=document.createElement('div'); row.className='item';
      row.innerHTML = `<span><strong>${acc.name}</strong> (€${fmt2(acc.saldo)}) [${acc.type}]</span>`;
      const controls=document.createElement('div');
      controls.innerHTML = `
        <button class=\"ghost\" onclick=\"addSaldo(${i})\">+€</button>
        <button class=\"ghost\" onclick=\"editAccount(${i})\">Bewerken</button>
        <button class=\"red\" onclick=\"deleteAccount(${i})\">Verwijderen</button>`;
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
        <button class=\"ghost\" onclick=\"stockPlus(${i})\">+1</button>
        <button class=\"ghost\" onclick=\"stockMin(${i})\">-1</button>
        <button class=\"ghost\" onclick=\"setStock(${i})\">Set</button>
        <button class=\"ghost\" onclick=\"editProduct(${i})\">Bewerken</button>
        <button class=\"red\" onclick=\"deleteProduct(${i})\">Verwijderen</button>`;
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
    updateAdminProducts(); renderProducts();
  }
  function editProduct(i){
    const p = products[i];
    const name = prompt('Nieuwe naam (leeg = ongewijzigd):', p.name) ?? '';
    const priceStr = prompt('Nieuwe prijs (leeg = ongewijzigd):', p.price) ?? '';
    const stockStr = prompt('Nieuwe voorraad (leeg = ongewijzigd):', p.stock) ?? '';
    if(name.trim()) p.name=name.trim();
    if(priceStr && !isNaN(parseFloat(priceStr))) p.price=parseFloat(priceStr);
    if(stockStr && !isNaN(parseInt(stockStr,10))) p.stock=parseInt(stockStr,10);
    saveData(); updateAdminProducts(); renderProducts();
  }
  function deleteProduct(i){ if(confirm('Product verwijderen?')){ products.splice(i,1); saveData(); updateAdminProducts(); renderProducts(); } }
  function stockPlus(i){ products[i].stock=(products[i].stock||0)+1; saveData(); updateAdminProducts(); renderProducts(); }
  function stockMin(i){ if((products[i].stock||0)>0){ products[i].stock-=1; saveData(); updateAdminProducts(); renderProducts(); } }
  function setStock(i){ const v=parseInt(prompt('Voorraad instellen op:', products[i].stock||0),10); if(!isNaN(v)&&v>=0){ products[i].stock=v; saveData(); updateAdminProducts(); renderProducts(); } }

  // ---- Log ----
  function renderLog(){
    const box=document.getElementById('logContainer'); if(!box) return; box.innerHTML='';
    if(logs.length===0){ box.innerHTML='<p class="muted">Nog geen transacties.</p>'; return; }
    const table=document.createElement('table'); table.className='table';
    table.innerHTML='<thead><tr><th>Datum</th><th>Gebruiker</th><th>Items</th><th>Totaal</th><th>Saldo na</th></tr></thead><tbody></tbody>';
    const tb=table.querySelector('tbody');
    logs.slice().reverse().forEach(l=>{
      const items = (l.items? l.items.map(it=>`${it.name}×${it.qty}`).join(', '): (l.product? `${l.product}×${l.qty}`:''));
      const tr=document.createElement('tr');
      tr.innerHTML = `<td>${tsNice(l.t)}</td><td>${l.user||''}</td><td>${items}</td><td>€${fmt2(l.total||0)}</td><td>€${fmt2(l.balanceAfter||0)}</td>`;
      tb.appendChild(tr);
    });
    box.appendChild(table);
  }
  function exportCSV(){
    const rows = [['datum','gebruiker','items','totaal','saldo_na']];
    logs.forEach(l=> rows.push([l.t, l.user||'', (l.items? l.items.map(it=>`${it.name}x${it.qty}`).join('|') : ''), fmt2(l.total||0), fmt2(l.balanceAfter||0)]));
    const csv = rows.map(r=> r.map(v=> '"'+String(v).replaceAll('"','""')+'"').join(',')).join('
');
    const blob = new Blob([csv],{type:'text/csv'}); const url=URL.createObjectURL(blob);
    const a=document.createElement('a'); a.href=url; a.download='log.csv'; a.click(); setTimeout(()=>URL.revokeObjectURL(url),1000);
  }
  function clearLogs(){ if(confirm('Hele log wissen?')){ logs=[]; saveData(); renderLog(); } }

  // ---- Init ----
  loadAccountButtons(); applyBranding();
</script></body>
</html>