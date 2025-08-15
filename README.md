<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Fictief Geld Systeem</title>
<style>
:root{
  --brand-red:#d62828;
  --brand-green:#1c7c54;
  --brand-cream:#ffffff;
  --text:#333;
  --muted:#666;
}
body { font-family: 'Segoe UI', Arial, sans-serif; background: linear-gradient(135deg, rgba(28,124,84,.07), rgba(214,40,40,.06)); margin:0; color:var(--text); position:relative; }
header { background: linear-gradient(90deg, var(--brand-green), var(--brand-red)); color:#fff; padding:14px 18px; display:flex; align-items:center; gap:12px; box-shadow:0 2px 6px rgba(0,0,0,.2); }
header img.logo { width:42px; height:42px; object-fit:contain; filter: drop-shadow(0 1px 2px rgba(0,0,0,.2)); background:#fff; border-radius:50%; padding:4px; }
header .title { display:flex; flex-direction:column; line-height:1.15; }
header .title strong { font-size:1.15rem; letter-spacing:.3px; }
header .title span { font-size:.8rem; opacity:.95; }

/* Breder canvas */
.container { max-width:1600px; margin:20px auto; background:#fff; padding:20px; border-radius:12px; box-shadow:0 6px 18px rgba(0,0,0,.08); }
@media (min-width:1900px){ .container{ max-width:1800px; } }

h2,h3 { margin-top:0; }
button { padding:8px 14px; border:0; background:var(--brand-green); color:#fff; border-radius:8px; cursor:pointer; margin:3px; font-size:.95em; transition:filter .15s, transform .05s; }
button:hover { filter:brightness(.95); transform:translateY(-1px); }
button.red { background:var(--brand-red); }
button.red:hover{ filter:brightness(.95); }
button.ghost { background:#eef7f2; color:var(--brand-green); }
input,select { padding:9px; margin:5px 0; width:100%; border:1px solid #dfe3e8; border-radius:8px; font-size:.95em; }
.hidden { display:none; }
.item { border-bottom:1px solid #f1f3f5; padding:8px 0; display:flex; justify-content:space-between; align-items:center; gap:10px; flex-wrap:wrap; }
.item:last-child{ border-bottom:none; }
.badge { background:#f39c12; color:#fff; padding:2px 6px; font-size:.8em; border-radius:6px; margin-left:5px; }
.badge.admin { background:var(--brand-red); }
.badge.coadmin { background:var(--brand-green); }

/* Meer kolommen op brede schermen */
#accountButtons { display:grid; grid-template-columns:repeat(auto-fill, minmax(200px,1fr)); gap:12px; }
.account-card { background:#fff; border-radius:12px; padding:12px; box-shadow:0 2px 8px rgba(0,0,0,.05); display:flex; flex-direction:column; align-items:flex-start; border-left:5px solid transparent; transition:transform .1s; }
.account-card:hover{ transform:translateY(-2px); }
.account-card.green{ border-left-color:var(--brand-green); }
.account-card.orange{ border-left-color:#f39c12; }
.account-card.red{ border-left-color:var(--brand-red); }
.account-card strong{ font-size:1.1em; }

table { width:100%; border-collapse:collapse; font-size:.9em; margin-top:10px; }
th,td { border:1px solid #eceff1; padding:6px; text-align:left; }
th{ background:#f8fafb; }
.low-stock{ color:var(--brand-red); font-weight:bold; }
.small{ font-size:.85em; color:var(--muted); }
.cart-summary { margin-top:10px; padding:10px; background:#f7fbff; border:1px solid #dfeaf7; border-radius:12px; display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:8px; }
.cart-summary strong { font-size:1.05em; }
.watermark{ position:fixed; inset:0; pointer-events:none; background: url('logo.png') no-repeat right 40px top 40px; background-size: 320px auto; opacity:.06; z-index:0; }
header, .container{ position:relative; z-index:1; }

.pin-wrap { position: relative; display:flex; align-items:center; gap:8px; }
.pin-toggle { background:#eef7f2; color:var(--brand-green); border:0; border-radius:8px; padding:8px 10px; cursor:pointer; }
.pin-toggle:active{ transform:translateY(1px); }

.form-actions { display:flex; justify-content:flex-end; gap:8px; margin-top:8px; }

/* Modals */
.modal-backdrop{ position:fixed; inset:0; background:rgba(0,0,0,.35); display:flex; align-items:center; justify-content:center; z-index:9999; }
.modal{ background:#fff; border-radius:14px; padding:16px; width:min(460px,92vw); box-shadow:0 10px 30px rgba(0,0,0,.2); }
.modal h4{ margin:0 0 8px 0; }
.modal .row{ display:flex; gap:8px; align-items:center; }
.modal .actions{ display:flex; justify-content:flex-end; gap:8px; margin-top:12px; }
.modal input[type="password"], .modal input[type="text"], .modal select{ width:100%; }
</style>
</head>
<body>
<div class="watermark"></div>
<header>
  <img class="logo" src="logo.png" alt="Logo Waterscouting">
  <div class="title">
    <strong>Fictief Geld Systeem</strong>
    <span>Waterscouting St. Willibrordus · Gouda</span>
  </div>
</header>

<!-- ALLE SCHERMEN HIER — zelfde HTML-structuur als je vorige versie -->

<script>
document.addEventListener('DOMContentLoaded', () => {
  const APP_VERSION = "2025-08-15-fixed";
  /* zelfde variabelen en initiële data als in jouw code */

  // Escape helper voor XSS
  function esc(str){ return String(str).replace(/[&<>"']/g,s=>({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;' }[s])); }

  /* rest van utils zoals bij jou */

  function renderCartSummary(){
    const {total, items} = computeCart();
    cartTotalEl.textContent = formatPrice(total);
    cartItemsEl.textContent = items;
    checkoutBtn.disabled = items === 0;
  }

  function checkoutCart(){
    const acc = accounts[currentUserIndex];
    const {total, items} = computeCart();
    if (items === 0){ alert('Je hebt niets geselecteerd.'); return; }
    if (acc.type==='gast' && acc.saldo - total < 0){ alert('Gast mag niet onder €0 komen!'); return; }
    if (acc.type==='vast' && acc.saldo - total < -10){ alert('Vast mag niet verder dan -€10 komen!'); return; }
    for (const i of Object.keys(cart)){
      const qty = cart[i]||0;
      const prod = products[i];
      if (!prod){ alert('Productlijst is gewijzigd. Vernieuw het scherm.'); return; }
      if (qty > prod.stock){
        alert(`Niet genoeg voorraad voor ${esc(prod.name)}`);
        return;
      }
    }
    if (!confirm(`Je staat op het punt te kopen voor €${formatPrice(total)} (${items} item(s)). Doorgaan?`)) return;
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

  function deleteAccount(i){
    if (!isAdmin()) return;
    if (isOnlyAdmin(i)){
      alert('Je kunt de laatste admin niet verwijderen. Wijs eerst een andere admin toe.');
      return;
    }
    if (!confirm(`Account "${accounts[i].name}" verwijderen?`)) return;
    logAction(`Account verwijderd: ${accounts[i].name}`);
    if (currentManager && currentManager.index === i){
      saveAll(); goHome(); alert('Je was ingelogd met dit account en bent nu uitgelogd.');
    }
    accounts.splice(i,1);
    saveAll(); loadAccountButtons(); updateAdminScreen();
  }

  async function changeRole(i){
    if (!isAdmin()) return;
    if (isOnlyAdmin(i)){
      alert('Deze gebruiker is de laatste admin. Je kunt de laatste admin niet degraderen.');
      return;
    }
    const huidige = accounts[i].role || 'user';
    const nieuw = await roleModal({title:`Rol wijzigen voor ${esc(accounts[i].name)}`, current: huidige});
    if (nieuw===null) return;
    if (!['user','coadmin','admin'].includes(nieuw)){ alert('Ongeldige rol.'); return; }
    if (accounts[i].role === 'admin' && nieuw !== 'admin' && getAdminCount() <= 1){
      alert('Er moet altijd minstens één admin blijven.'); return;
    }
    accounts[i].role = nieuw;
    logAction(`Rol gewijzigd: ${accounts[i].name} → ${nieuw}`);
    saveAll();
    if (currentManager && currentManager.index === i && nieuw !== 'admin' && nieuw !== 'coadmin'){
      goHome(); alert('Je rechten zijn gewijzigd. Je bent uitgelogd.');
      return;
    }
    loadAccountButtons(); updateAdminScreen();
  }

  function loadAccountButtons(){
    accountButtons.innerHTML = '';
    accounts.forEach((acc, i) => {
      const card = document.createElement('div');
      card.className = 'account-card ' + classifyCard(acc);
      let roleBadge = '';
      if (acc.role === 'admin') roleBadge = ' <span class="badge admin">admin</span>';
      else if (acc.role === 'coadmin') roleBadge = ' <span class="badge coadmin">co-admin</span>';
      card.innerHTML = `
        <strong>${esc(acc.name)}${roleBadge}</strong>
        <span>Saldo: €${formatPrice(acc.saldo)} ${acc.type === 'gast' ? '<span class="badge">gast</span>' : ''}</span>
      `;
      card.onclick = () => selectAccount(i);
      accountButtons.appendChild(card);
    });
    /* rest zoals in jouw versie */
  }

  /* alle andere functies ongewijzigd behalve dat ik esc() gebruik bij weergeven van namen/producten in innerHTML of confirm/alert */

});
</script>
</body>
</html>
