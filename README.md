<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Fictief Geld Systeem</title>
<style>
/* Kleuren en variabelen */
:root {
  --brand-red: #d62828;
  --brand-green: #1c7c54;
  --brand-cream: #ffffff;
  --text: #333;
  --muted: #666;
}

/* Algemene styling */
body {
  font-family: 'Segoe UI', Arial, sans-serif;
  background: linear-gradient(135deg, rgba(28,124,84,.07), rgba(214,40,40,.06));
  margin: 0;
  color: var(--text);
  position: relative;
}
header {
  background: linear-gradient(90deg, var(--brand-green), var(--brand-red));
  color: #fff;
  padding: 14px 18px;
  display: flex;
  align-items: center;
  gap: 12px;
  box-shadow: 0 2px 6px rgba(0,0,0,.2);
}
header img.logo {
  width: 42px;
  height: 42px;
  object-fit: contain;
  filter: drop-shadow(0 1px 2px rgba(0,0,0,.2));
  background: #fff;
  border-radius: 50%;
  padding: 4px;
}
header .title {
  display: flex;
  flex-direction: column;
  line-height: 1.15;
}
header .title strong {
  font-size: 1.15rem;
  letter-spacing: .3px;
}
header .title span {
  font-size: .8rem;
  opacity: .95;
}

/* Container voor breder canvas */
.container {
  max-width: 1600px;
  margin: 20px auto;
  background: #fff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 6px 18px rgba(0,0,0,.08);
  position: relative;
  z-index: 1;
}
@media (min-width: 1900px){
  .container {
    max-width: 1800px;
  }
}

h2, h3 {
  margin-top: 0;
}
button {
  padding: 8px 14px;
  border: 0;
  background: var(--brand-green);
  color: #fff;
  border-radius: 8px;
  cursor: pointer;
  margin: 3px;
  font-size: .95em;
  transition: filter .15s, transform .05s;
}
button:hover {
  filter: brightness(.95);
  transform: translateY(-1px);
}
button.red {
  background: var(--brand-red);
}
button.red:hover {
  filter: brightness(.95);
}
button.ghost {
  background: #eef7f2;
  color: var(--brand-green);
}
button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}
input, select {
  padding: 9px;
  margin: 5px 0;
  width: 100%;
  border: 1px solid #dfe3e8;
  border-radius: 8px;
  font-size: .95em;
}
.hidden {
  display: none;
}
.item {
  border-bottom: 1px solid #f1f3f5;
  padding: 8px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}
.item:last-child {
  border-bottom: none;
}
.badge {
  background: #f39c12;
  color: #fff;
  padding: 2px 6px;
  font-size: .8em;
  border-radius: 6px;
  margin-left: 5px;
}
.badge.admin {
  background: var(--brand-red);
}
.badge.coadmin {
  background: var(--brand-green);
}

/* Meer kolommen op brede schermen */
#accountButtons {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px,1fr));
  gap: 12px;
}
.account-card {
  background: #fff;
  border-radius: 12px;
  padding: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,.05);
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  border-left: 5px solid transparent;
  transition: transform .1s;
  cursor: pointer;
}
.account-card:hover {
  transform: translateY(-2px);
}
.account-card.green {
  border-left-color: var(--brand-green);
}
.account-card.orange {
  border-left-color: #f39c12;
}
.account-card.red {
  border-left-color: var(--brand-red);
}
.account-card strong {
  font-size: 1.1em;
}

/* Tabellen en logboek */
table {
  width: 100%;
  border-collapse: collapse;
  font-size: .9em;
  margin-top: 10px;
}
th, td {
  border: 1px solid #eceff1;
  padding: 6px;
  text-align: left;
}
th {
  background: #f8fafb;
}
.low-stock {
  color: var(--brand-red);
  font-weight: bold;
}
.small {
  font-size: .85em;
  color: var(--muted);
}
.cart-summary {
  margin-top: 10px;
  padding: 10px;
  background: #f7fbff;
  border: 1px solid #dfeaf7;
  border-radius: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
}
.cart-summary strong {
  font-size: 1.05em;
}
.watermark {
  position: fixed;
  inset: 0;
  pointer-events: none;
  /* Je logo gebruikt als watermerk */
  background: url('https://storage.googleapis.com/bacon-prod-uploaded-files/editor-files/6d555c82-e8c0-43ac-91c6-1215b630e2f5/Logo%20rood%20groen.png') no-repeat right 40px top 40px;
  background-size: 320px auto;
  opacity: .06;
  z-index: 0;
}
header, .container {
  position: relative;
  z-index: 1;
}

.pin-wrap {
  position: relative;
  display: flex;
  align-items: center;
  gap: 8px;
}
.pin-toggle {
  background: #eef7f2;
  color: var(--brand-green);
  border: 0;
  border-radius: 8px;
  padding: 8px 10px;
  cursor: pointer;
}
.pin-toggle:active {
  transform: translateY(1px);
}
.pin-toggle:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 8px;
}

/* Modals */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.35);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  transition: opacity 0.2s ease-in-out;
}
.modal {
  background: #fff;
  border-radius: 14px;
  padding: 16px;
  width: min(460px, 92vw);
  box-shadow: 0 10px 30px rgba(0,0,0,.2);
  transform: scale(0.95);
  transition: transform 0.2s ease-in-out;
}
.modal-backdrop.show {
  opacity: 1;
}
.modal.show {
  transform: scale(1);
}
.modal h4 {
  margin: 0 0 8px 0;
}
.modal .row {
  display: flex;
  gap: 8px;
  align-items: center;
}
.modal .actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 12px;
}
.modal input[type="password"],
.modal input[type="text"],
.modal select {
  width: 100%;
}
</style>
</head>
<body>
<div class="watermark"></div>
<header>
  <!-- Het logo dat je hebt geüpload -->
  <img class="logo" src="https://storage.googleapis.com/bacon-prod-uploaded-files/editor-files/6d555c82-e8c0-43ac-91c6-1215b630e2f5/Logo%20rood%20groen.png" alt="Logo Waterscouting" onerror="this.src='https://placehold.co/42x42/333333/ffffff?text=LOGO'">
  <div class="title">
    <strong>Fictief Geld Systeem</strong>
    <span>Waterscouting St. Willibrordus · Gouda</span>
  </div>
</header>

<!-- HOME -->
<div class="container" id="homeScreen" autocomplete="off">
  <h2>Kies je account</h2>
  <div id="accountButtons"></div>
  <hr>
  <div class="item">
    <span>Beheerder inloggen</span>
    <span style="display:flex; gap:6px; align-items:center;">
      <select id="adminAccountSelect" style="min-width:220px" autocomplete="off"></select>
      <input
        type="password"
        id="adminCode"
        placeholder="Pincode (4 cijfers)"
        maxlength="4"
        inputmode="numeric"
        style="width:140px;"
        autocomplete="new-password"
        autocapitalize="off"
        spellcheck="false"
        disabled
      >
      <button id="adminLoginBtn">Inloggen</button>
    </span>
  </div>
  <p class="small">Tip: Alleen accounts met rol <strong>Admin</strong> of <strong>Co-admin</strong> kunnen inloggen op het beheer.</p>
</div>

<!-- PIN -->
<div class="container hidden" id="pinScreen" autocomplete="off">
  <h2>Inloggen</h2>
  <p id="selectedUserName"></p>
  <input type="password" id="pincode" placeholder="Pincode (4 cijfers)" maxlength="4" inputmode="numeric" autocomplete="new-password" autocapitalize="off" spellcheck="false">
  <button id="userLoginBtn">Inloggen</button>
  <button class="red" id="cancelPinBtn">Annuleren</button>
</div>

<!-- USER -->
<div class="container hidden" id="userScreen">
  <h2 id="welcome"></h2>
  <p>Saldo: €<span id="saldo"></span></p>
  <div id="productList"></div>
  <div class="cart-summary">
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

<div id="modalContainer"></div>

<script>
document.addEventListener('DOMContentLoaded', () => {
  /* ---- Config ---- */
  const APP_VERSION = "2025-08-15-wide-rename-pin4-v5";
  const MAX_USERS = 50;
  const ADMIN_IDLE_TIMEOUT_MS = 5 * 60 * 1000;
  const ADMIN_LOCK_MAX_FAILS = 5;
  const ADMIN_LOCK_DURATION_MS = 2 * 60 * 1000;

  let currentManager = null; // {index, role:'admin'|'coadmin'}
  let lastAdminActionAt = Date.now();

  /* ---- State ---- */
  let accounts = safeGet('accounts', [
    { name: "Jan", pin: "1234", saldo: 40.00, type: "vast", role: "user" },
    { name: "Piet", pin: "5678", saldo: 5.00, type: "gast", role: "user" },
    { name: "Beheer", pin: "9999", saldo: 0.00, type: "vast", role: "admin" }
  ]);
  let products = safeGet('products', [
    { name: "Chips", price: 0.75, stock: 20 },
    { name: "Bier", price: 0.75, stock: 30 },
    { name: "Cola", price: 1.00, stock: 15 }
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

  const modalContainer = document.getElementById('modalContainer');

  /* ---- Utils ---- */
  function saveAll() {
    localStorage.setItem('accounts', JSON.stringify(accounts));
    localStorage.setItem('products', JSON.stringify(products));
    localStorage.setItem('logs', JSON.stringify(logs));
    localStorage.setItem('appVersion', APP_VERSION);
  }

  function safeGet(key, fallback) {
    try {
      const raw = localStorage.getItem(key);
      if (!raw) return fallback;
      const val = JSON.parse(raw);
      if (key === 'accounts' && !Array.isArray(val)) return fallback;
      if (key === 'products' && !Array.isArray(val)) return fallback;
      if (key === 'logs' && !Array.isArray(val)) return fallback;
      return val;
    } catch (e) {
      console.error('Error getting from localStorage:', e);
      return fallback;
    }
  }

  function formatPrice(n) {
    return Number(n).toFixed(2);
  }

  function digitsOnly(el) {
    el.value = el.value.replace(/\D+/g, '').slice(0, 4);
  }

  function show(el) {
    el.classList.remove('hidden');
  }

  function hide(el) {
    el.classList.add('hidden');
  }

  function now() {
    return new Date().toLocaleString();
  }

  function actorName() {
    return currentManager ? accounts[currentManager.index].name : 'SYSTEEM';
  }

  function logAction(text, bedrag = 0) {
    logs.push({ gebruiker: actorName(), product: `ACTIE: ${text}`, prijs: bedrag, tijd: now() });
  }

  function getAdminCount() {
    return accounts.filter(a => a.role === 'admin').length;
  }

  function isOnlyAdmin(index) {
    return accounts[index]?.role === 'admin' && getAdminCount() === 1;
  }

  // SHA-256 hashing
  async function sha256Hex(str) {
    const enc = new TextEncoder().encode(str);
    const buf = await crypto.subtle.digest('SHA-256', enc);
    const bytes = new Uint8Array(buf);
    return Array.from(bytes).map(b => b.toString(16).padStart(2, '0')).join('');
  }

  /**
   * Generieke modale dialoog voor waarschuwingen en bevestigingen.
   * @param {object} options
   * @param {string} options.title - Titel van de dialoog.
   * @param {string} options.message - Bericht dat wordt weergegeven.
   * @param {boolean} [options.isConfirm=false] - Of het een bevestigingsdialoog is (met OK/Annuleren).
   * @param {string} [options.okText="OK"] - Tekst voor de OK-knop.
   * @param {string} [options.cancelText="Annuleren"] - Tekst voor de Annuleer-knop.
   * @returns {Promise<boolean>} Resolves met true voor OK, false voor Annuleren/sluiten.
   */
  function showModal({ title, message, isConfirm = false, okText = "OK", cancelText = "Annuleren" }) {
    return new Promise(resolve => {
      const backdrop = document.createElement('div');
      backdrop.className = 'modal-backdrop';
      const modal = document.createElement('div');
      modal.className = 'modal';
      modal.innerHTML = `
        <h4>${title}</h4>
        <p>${message}</p>
        <div class="actions">
          ${isConfirm ? `<button id="cancelBtn" class="ghost">${cancelText}</button>` : ''}
          <button id="okBtn">${okText}</button>
        </div>
      `;
      modalContainer.appendChild(backdrop);
      backdrop.appendChild(modal);

      // Timeout om CSS-transities te laten werken
      setTimeout(() => {
        backdrop.classList.add('show');
        modal.classList.add('show');
      }, 10);

      const okBtn = modal.querySelector('#okBtn');
      const cancelBtn = modal.querySelector('#cancelBtn');

      okBtn.addEventListener('click', () => {
        backdrop.classList.remove('show');
        modal.classList.remove('show');
        setTimeout(() => {
          modalContainer.removeChild(backdrop);
          resolve(true);
        }, 200);
      });

      if (cancelBtn) {
        cancelBtn.addEventListener('click', () => {
          backdrop.classList.remove('show');
          modal.classList.remove('show');
          setTimeout(() => {
            modalContainer.removeChild(backdrop);
            resolve(false);
          }, 200);
        });
      }
    });
  }

  // PIN modal (precies 4)
  function securePinModal({ title = "Nieuwe pincode", okText = "Opslaan" }) {
    return new Promise(resolve => {
      const backdrop = document.createElement('div');
      backdrop.className = 'modal-backdrop';
      const modal = document.createElement('div');
      modal.className = 'modal';
      modal.innerHTML = `
        <h4>${title}</h4>
        <div class="row">
          <input id="pin1" type="password" placeholder="Pincode (exact 4 cijfers)" maxlength="4" inputmode="numeric" autocomplete="new-password" autocapitalize="off" spellcheck="false" style="flex:1;">
          <button id="toggle1" class="pin-toggle" aria-label="Toon/verberg">👁️</button>
        </div>
        <div class="row" style="margin-top:6px;">
          <input id="pin2" type="password" placeholder="Bevestig pincode (4 cijfers)" maxlength="4" inputmode="numeric" autocomplete="new-password" autocapitalize="off" spellcheck="false" style="flex:1;">
          <button id="toggle2" class="pin-toggle" aria-label="Toon/verberg">👁️</button>
        </div>
        <div class="actions">
          <button id="cancel" class="ghost">Annuleren</button>
          <button id="ok">${okText}</button>
        </div>
      `;
      modalContainer.appendChild(backdrop);
      backdrop.appendChild(modal);

      setTimeout(() => {
        backdrop.classList.add('show');
        modal.classList.add('show');
      }, 10);

      const pin1 = modal.querySelector('#pin1');
      const pin2 = modal.querySelector('#pin2');
      const ok = modal.querySelector('#ok');
      const cancel = modal.querySelector('#cancel');
      const t1 = modal.querySelector('#toggle1');
      const t2 = modal.querySelector('#toggle2');

      const enforceDigits = (el) => el.addEventListener('input', () => { el.value = el.value.replace(/\D+/g, '').slice(0, 4); });
      enforceDigits(pin1);
      enforceDigits(pin2);
      const toggle = (btn, el) => btn.addEventListener('click', () => { el.type = el.type === 'password' ? 'text' : 'password'; });
      toggle(t1, pin1);
      toggle(t2, pin2);

      function close(val) {
        backdrop.classList.remove('show');
        modal.classList.remove('show');
        setTimeout(() => {
          modalContainer.removeChild(backdrop);
          resolve(val);
        }, 200);
      }
      cancel.addEventListener('click', () => close(null));
      ok.addEventListener('click', async () => {
        if (!/^\d{4}$/.test(pin1.value)) {
          await showModal({ title: 'Fout', message: 'Pincode moet precies 4 cijfers zijn.' });
          return;
        }
        if (pin1.value !== pin2.value) {
          await showModal({ title: 'Fout', message: 'Pincodes komen niet overeen.' });
          return;
        }
        close(pin1.value);
      });
      backdrop.addEventListener('click', (e) => {
        if (e.target === backdrop) close(null);
      });
      pin1.focus();
    });
  }

  // Role modal (dropdown)
  function roleModal({ title = "Rol wijzigen", current = "user" }) {
    return new Promise(resolve => {
      const roles = ["user", "coadmin", "admin"];
      const backdrop = document.createElement('div');
      backdrop.className = 'modal-backdrop';
      const modal = document.createElement('div');
      modal.className = 'modal';
      modal.innerHTML = `
        <h4>${title}</h4>
        <div class="row">
          <select id="roleSelect">
            ${roles.map(r => `<option value="${r}" ${r === current ? 'selected' : ''}>${r}</option>`).join('')}
          </select>
        </div>
        <div class="actions">
          <button id="cancel" class="ghost">Annuleren</button>
          <button id="ok">Wijzigen</button>
        </div>
      `;
      modalContainer.appendChild(backdrop);
      backdrop.appendChild(modal);

      setTimeout(() => {
        backdrop.classList.add('show');
        modal.classList.add('show');
      }, 10);

      const ok = modal.querySelector('#ok');
      const cancel = modal.querySelector('#cancel');
      const sel = modal.querySelector('#roleSelect');
      function close(val) {
        backdrop.classList.remove('show');
        modal.classList.remove('show');
        setTimeout(() => {
          modalContainer.removeChild(backdrop);
          resolve(val);
        }, 200);
      }
      cancel.addEventListener('click', () => close(null));
      ok.addEventListener('click', () => close(sel.value));
      backdrop.addEventListener('click', (e) => {
        if (e.target === backdrop) close(null);
      });
      sel.focus();
    });
  }

  // Login lock helpers
  function getLockState() {
    try {
      return JSON.parse(localStorage.getItem('adminLock') || '{}');
    } catch {
      return {};
    }
  }

  function setLockState(s) {
    localStorage.setItem('adminLock', JSON.stringify(s));
  }

  // Migreer plaintext -> hash
  async function migratePinsIfNeeded() {
    let changed = false;
    for (const acc of accounts) {
      if (acc.pinHash && !acc.pin) continue;
      if (typeof acc.pin === 'string' && /^\d{1,4}$/.test(acc.pin)) {
        acc.pinHash = await sha256Hex(acc.pin);
        delete acc.pin;
        changed = true;
      } else if (acc.pin) {
        acc.pinHash = await sha256Hex(String(acc.pin));
        delete acc.pin;
        changed = true;
      }
    }
    if (changed) {
      logAction('PINs gemigreerd naar hashes');
      saveAll();
    }
  }

  /* ---- UI build ---- */
  function loadAccountButtons() {
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

    // Beheerder-select vullen met admin/coadmin
    adminAccountSelect.innerHTML = '';
    const staff = accounts.map((a, idx) => ({ idx, a })).filter(x => x.a.role === 'admin' || x.a.role === 'coadmin');
    if (staff.length === 0) {
      const opt = document.createElement('option');
      opt.text = '— geen beheerders —';
      opt.value = '';
      adminAccountSelect.appendChild(opt);
      adminLoginBtn.disabled = true;
    } else {
      adminLoginBtn.disabled = false;
      staff.forEach(s => {
        const opt = document.createElement('option');
        opt.value = s.idx;
        opt.text = `${s.a.name} (${s.a.role})`;
        adminAccountSelect.appendChild(opt);
      });
      adminCode.removeAttribute('disabled');
      adminCode.setAttribute('name', 'pin_' + Math.random().toString(36).slice(2, 8));
    }
  }

  function classifyCard(acc) {
    if (acc.type === 'gast') return acc.saldo >= 0 ? 'green' : 'red';
    if (acc.saldo >= 0) return 'green';
    if (acc.saldo >= -10) return 'orange';
    return 'red';
  }

  function goHome() {
    hide(pinScreen);
    hide(userScreen);
    hide(adminScreen);
    show(homeScreen);
    adminCode.value = '';
    adminCode.disabled = true;
    currentManager = null;
    loadAccountButtons();
  }

  /* ---- User login ---- */
  function selectAccount(index) {
    currentUserIndex = index;
    selectedUserName.textContent = 'Account: ' + accounts[index].name;
    pincode.value = '';
    hide(homeScreen);
    show(pinScreen);
  }

  async function checkLogin() {
    if (!/^\d{4}$/.test(pincode.value || '')) {
      await showModal({ title: 'Fout', message: 'Pincode moet precies 4 cijfers zijn.' });
      return;
    }
    const acc = accounts[currentUserIndex];
    const inputHash = await sha256Hex(pincode.value);
    if (acc.pinHash === inputHash) {
      pincode.value = '';
      hide(pinScreen);
      show(userScreen);
      welcome.textContent = 'Welkom ' + acc.name;
      initCart();
      updateUserScreen();
    } else {
      await showModal({ title: 'Fout', message: 'Verkeerde pincode!' });
    }
  }

  /* ---- Cart helpers ---- */
  function initCart() {
    cart = {};
    products.forEach((_, i) => cart[i] = 0);
  }

  function computeCart() {
    let total = 0,
      items = 0;
    Object.keys(cart).forEach(i => {
      const qty = cart[i] || 0;
      items += qty;
      total += qty * (products[i]?.price || 0);
    });
    return { total, items };
  }

  function renderCartSummary() {
    const { total, items } = computeCart();
    cartTotalEl.textContent = formatPrice(total);
    cartItemsEl.textContent = items;
    checkoutBtn.disabled = items === 0;
  }

  /* ---- User screen / products ---- */
  function updateUserScreen() {
    const acc = accounts[currentUserIndex];
    saldoEl.textContent = formatPrice(acc.saldo);
    if (acc.type === 'gast') {
      saldoEl.style.color = acc.saldo >= 0 ? 'green' : 'red';
    } else {
      saldoEl.style.color = (acc.saldo >= 0 ? 'green' : (acc.saldo >= -10 ? 'orange' : 'red'));
    }

    productList.innerHTML = '';
    products.forEach((p, i) => {
      const voorraadClass = p.stock <= 5 ? 'low-stock' : '';
      const row = document.createElement('div');
      row.className = 'item';
      row.innerHTML = `
        <span>${p.name} - €${formatPrice(p.price)} (<span class="${voorraadClass}">voorraad: ${p.stock}</span>)</span>
        <span>
          <input type="number" step="1" inputmode="numeric" min="0" max="${p.stock}" value="${cart[i] || 0}" style="width:80px;">
        </span>
      `;
      const input = row.querySelector('input');
      input.addEventListener('input', () => {
        const val = Math.max(0, Math.min(p.stock, parseInt(input.value) || 0));
        cart[i] = val;
        input.value = val;
        renderCartSummary();
      });
      productList.appendChild(row);
    });
    renderCartSummary();
  }

  async function checkoutCart() {
    const acc = accounts[currentUserIndex];
    const { total, items } = computeCart();
    if (items === 0) {
      await showModal({ title: 'Fout', message: 'Je hebt niets geselecteerd.' });
      return;
    }

    if (acc.type === 'gast' && acc.saldo - total < 0) {
      await showModal({ title: 'Fout', message: 'Gast mag niet onder €0 komen!' });
      return;
    }
    if (acc.type === 'vast' && acc.saldo - total < -10) {
      await showModal({ title: 'Fout', message: 'Vast mag niet verder dan -€10 komen!' });
      return;
    }

    for (const i of Object.keys(cart)) {
      const qty = cart[i] || 0;
      if (qty > products[i].stock) {
        await showModal({ title: 'Fout', message: `Niet genoeg voorraad voor ${products[i].name}` });
        return;
      }
    }

    const confirmed = await showModal({
      title: 'Aankoop Bevestigen',
      message: `Je staat op het punt te kopen voor €${formatPrice(total)} (${items} item(s)). Doorgaan?`,
      isConfirm: true
    });
    if (!confirmed) return;

    acc.saldo -= total;
    Object.keys(cart).forEach(i => {
      const qty = cart[i] || 0;
      if (qty > 0) {
        products[i].stock -= qty;
        logs.push({ gebruiker: acc.name, product: `${products[i].name} (x${qty})`, prijs: products[i].price * qty, tijd: now() });
        cart[i] = 0;
      }
    });

    saveAll();
    updateUserScreen();
    loadAccountButtons();
    await showModal({ title: 'Succes', message: 'Aankoop voltooid.' });
    // Automatisch uitloggen en terug naar homescherm
    goHome();
  }

  /* ---- Admin login met cooldown + idle timeout ---- */
  async function adminLogin() {
    const sel = adminAccountSelect.value;
    if (sel === '') {
      await showModal({ title: 'Fout', message: 'Kies een beheerder-account.' });
      return;
    }
    const idx = parseInt(sel);
    const acc = accounts[idx];
    if (!acc || (acc.role !== 'admin' && acc.role !== 'coadmin')) {
      await showModal({ title: 'Fout', message: 'Geen beheerdersrol.' });
      return;
    }

    const lock = getLockState();
    const nowTs = Date.now();
    if (lock.until && nowTs < lock.until) {
      const sec = Math.ceil((lock.until - nowTs) / 1000);
      await showModal({ title: 'Te veel mislukte pogingen', message: `Probeer over ${sec}s opnieuw.` });
      return;
    }

    if (!/^\d{4}$/.test(adminCode.value || '')) {
      await showModal({ title: 'Fout', message: 'Pincode moet precies 4 cijfers zijn.' });
      return;
    }
    const inputHash = await sha256Hex(adminCode.value);
    
    // Wis de pincode onmiddellijk na invoer voor extra veiligheid
    adminCode.value = '';

    if (acc.pinHash !== inputHash) {
      await showModal({ title: 'Fout', message: 'Verkeerde pincode!' });
      const nextFails = (lock.fails || 0) + 1;
      if (nextFails >= ADMIN_LOCK_MAX_FAILS) {
        setLockState({ fails: 0, until: nowTs + ADMIN_LOCK_DURATION_MS });
        await showModal({ title: 'Geblokkeerd', message: 'Account tijdelijk geblokkeerd voor beheerlogin (2 minuten).' });
      } else {
        setLockState({ fails: nextFails, until: 0 });
      }
      logAction(`MISLUKTE beheerlogin voor ${acc.name}`);
      saveAll();
      return;
    }

    setLockState({ fails: 0, until: 0 });
    currentManager = { index: idx, role: acc.role };
    hide(homeScreen);
    show(adminScreen);
    adminTitle.textContent = acc.role === 'coadmin' ? 'Co-Admin Paneel' : 'Admin Paneel';
    logAction(`Beheerlogin als ${acc.role} (${acc.name})`);
    saveAll();
    updateAdminScreen();
    applyPermissions();
    touchAdminActivity();
  }

  function isAdmin() {
    return currentManager && currentManager.role === 'admin';
  }

  function isCoAdmin() {
    return currentManager && currentManager.role === 'coadmin';
  }

  function applyPermissions() {
    if (isAdmin()) {
      show(dataBeheer);
    } else {
      hide(dataBeheer);
    }
    if (exportCsvBtn) exportCsvBtn.disabled = false;
    if (isAdmin()) {
      clearLogsBtn.disabled = false;
      clearLogsBtn.classList.remove('hidden');
    } else {
      clearLogsBtn.disabled = true;
      clearLogsBtn.classList.add('hidden');
    }
  }

  function touchAdminActivity() {
    lastAdminActionAt = Date.now();
  }
  setInterval(async () => {
    if (!currentManager) return;
    if (Date.now() - lastAdminActionAt > ADMIN_IDLE_TIMEOUT_MS) {
      await showModal({ title: 'Sessie verlopen', message: 'Vanwege inactiviteit ben je uitgelogd uit het adminpaneel.' });
      logAction('Beheer auto-uitlog (inactiviteit)');
      saveAll();
      goHome();
    }
  }, 15 * 1000);
  document.addEventListener('click', () => {
    if (!adminScreen.classList.contains('hidden')) touchAdminActivity();
  });
  document.addEventListener('keydown', () => {
    if (!adminScreen.classList.contains('hidden')) touchAdminActivity();
  });

  /* ---- Admin screen ---- */
  function updateAdminScreen() {
    adminSections.innerHTML = '';

    // Accounts
    const accDiv = document.createElement('div');
    accDiv.innerHTML = `
      <h3>Accounts</h3>
      <div class="item">
        <div id="newAccountForm" style="flex:1; min-width:260px; display:flex; flex-direction:column;">
          <input id="newName" placeholder="Naam" ${!(isAdmin() || isCoAdmin()) ? 'disabled' : ''}>
          <div class="pin-wrap">
            <input id="newPin" type="password" placeholder="Pincode (exact 4 cijfers)" maxlength="4" inputmode="numeric" ${!(isAdmin() || isCoAdmin()) ? 'disabled' : ''} autocomplete="new-password" autocapitalize="off" spellcheck="false">
            <button class="pin-toggle" id="toggleNewPin" ${!(isAdmin() || isCoAdmin()) ? 'disabled' : ''}>👁️</button>
          </div>
          <input type="number" id="newSaldo" placeholder="Startsaldo" value="0" ${!(isAdmin() || isCoAdmin()) ? 'disabled' : ''}>
          <label class="small"><input type="checkbox" id="newIsGuest" ${!(isAdmin() || isCoAdmin()) ? 'disabled' : ''}> Dit is een gastaccount (gastaccounts mogen niet onder &euro;0 komen)</label>
          <select id="newRole" ${!isAdmin() ? 'disabled' : ''}>
            <option value="user">Rol: Gebruiker</option>
            <option value="coadmin">Rol: Co-admin</option>
            <option value="admin">Rol: Admin</option>
          </select>
          ${(isAdmin() || isCoAdmin()) ? `
          <div class="form-actions">
            <button id="addAccountBtn">Account toevoegen</button>
          </div>` : ''}
        </div>
      </div>
      <div id="accountList"></div>
    `;
    adminSections.appendChild(accDiv);

    // Producten
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
      </div>` : ''}
      <div id="productAdminList"></div>
    `;
    adminSections.appendChild(prodDiv);

    // Accounts lijst
    const accountList = accDiv.querySelector('#accountList');
    accountList.innerHTML = '';
    accounts.forEach((acc, i) => {
      const row = document.createElement('div');
      row.className = 'item';
      row.innerHTML = `
        <span>
          ${acc.name}
          ${acc.type === 'gast' ? '<span class="badge">gast</span>' : ''}
          ${acc.role === 'admin' ? '<span class="badge admin">admin</span>' : acc.role === 'coadmin' ? '<span class="badge coadmin">co-admin</span>' : ''}
          (Saldo €${formatPrice(acc.saldo)})
        </span>
        <span style="display:flex; align-items:center; gap:8px;">
          <label class="small" title="Schakel gaststatus">
            <input type="checkbox" data-type="${i}" ${acc.type === 'gast' ? 'checked' : ''} ${!(isAdmin() || isCoAdmin()) ? 'disabled' : ''}>
            <strong>Gast</strong>
          </label>
          ${isAdmin() ? `<button data-rename="${i}">Naam wijzigen</button>` : ''}
          ${isAdmin() ? `<button data-role="${i}">Rol wijzigen</button>` : ''}
          ${isAdmin() ? `<button data-pin="${i}">PIN wijzigen</button>` : ''}
          <button data-add="${i}">+€</button>
          ${isAdmin() ? `<button class="red" data-del="${i}">X</button>` : ''}
        </span>
      `;
      accountList.appendChild(row);
    });

    // Producten lijst
    const prodList = prodDiv.querySelector('#productAdminList');
    prodList.innerHTML = '';
    products.forEach((p, i) => {
      const voorraadClass = p.stock <= 5 ? 'low-stock' : '';
      const row = document.createElement('div');
      row.className = 'item';
      row.innerHTML = `
        <span>${p.name} (€${formatPrice(p.price)}) - <span class="${voorraadClass}">Voorraad: ${p.stock}</span></span>
        <span>
          ${isAdmin() ? `<button data-price="${i}">Prijs wijzigen</button>` : ''}
          ${isAdmin() ? `<button data-restock="${i}">Voorraad bijvullen</button>` : ''}
          ${isAdmin() ? `<button class="red" data-pdel="${i}">X</button>` : ''}
        </span>
      `;
      prodList.appendChild(row);
    });

    // Logboek
    let html = '<table><tr><th>Gebruiker</th><th>Product/Actie</th><th>Prijs/Bedrag</th><th>Tijd</th></tr>';
    logs.forEach(l => {
      html += `<tr><td>${l.gebruiker}</td><td>${l.product}</td><td>€${formatPrice(l.prijs)}</td><td>${l.tijd}</td></tr>`;
    });
    html += '</table>';
    logList.innerHTML = html;

    // wire inputs
    const newPin = adminSections.querySelector('#newPin');
    if (newPin) newPin.addEventListener('input', () => digitsOnly(newPin));
    const toggleNewPin = adminSections.querySelector('#toggleNewPin');
    if (toggleNewPin && newPin) {
      toggleNewPin.addEventListener('click', () => {
        newPin.type = newPin.type === 'password' ? 'text' : 'password';
      });
    }

    // Enter-toevoegen in nieuw account formulier
    const newForm = adminSections.querySelector('#newAccountForm');
    if (newForm) {
      newForm.addEventListener('keydown', (e) => {
        if (e.key === 'Enter') {
          e.preventDefault();
          const btn = adminSections.querySelector('#addAccountBtn');
          if (btn && !btn.disabled) btn.click();
        }
      });
    }

    // Buttons wiring
    const addAccountBtn = adminSections.querySelector('#addAccountBtn');
    if (addAccountBtn) addAccountBtn.addEventListener('click', () => addAccount());
    adminSections.querySelectorAll('button[data-del]').forEach(btn => btn.addEventListener('click', () => deleteAccount(+btn.dataset.del)));
    adminSections.querySelectorAll('button[data-add]').forEach(btn => btn.addEventListener('click', () => addSaldo(+btn.dataset.add)));
    adminSections.querySelectorAll('button[data-pin]').forEach(btn => btn.addEventListener('click', () => changePin(+btn.dataset.pin)));
    adminSections.querySelectorAll('button[data-role]').forEach(btn => btn.addEventListener('click', () => changeRole(+btn.dataset.role)));
    adminSections.querySelectorAll('button[data-rename]').forEach(btn => btn.addEventListener('click', () => renameAccount(+btn.dataset.rename)));
    const addProductBtn = adminSections.querySelector('#addProductBtn');
    if (addProductBtn) addProductBtn.addEventListener('click', addProduct);
    adminSections.querySelectorAll('button[data-pdel]').forEach(btn => btn.addEventListener('click', () => deleteProduct(+btn.dataset.pdel)));
    adminSections.querySelectorAll('button[data-restock]').forEach(btn => btn.addEventListener('click', () => restockProduct(+btn.dataset.restock)));
    adminSections.querySelectorAll('button[data-price]').forEach(btn => btn.addEventListener('click', () => changePrice(+btn.dataset.price)));

    // Toggle gast/vast checkbox
    adminSections.querySelectorAll('input[data-type]').forEach(chk => {
      chk.addEventListener('change', () => {
        const idx = +chk.getAttribute('data-type');
        const oud = accounts[idx].type;
        const nieuw = chk.checked ? 'gast' : 'vast';
        if (oud === nieuw) return;
        accounts[idx].type = nieuw;
        logAction(`Type gewijzigd: ${accounts[idx].name} → ${nieuw}`);
        saveAll();
        loadAccountButtons();
        updateAdminScreen();
      });
    });
  }

  /* ---- Admin actions ---- */
  async function addAccount() {
    if (!(isAdmin() || isCoAdmin())) return;
    if (accounts.length >= MAX_USERS) {
      await showModal({ title: 'Fout', message: `Maximum aantal accounts (${MAX_USERS}) bereikt. Verwijder eerst een account.` });
      return;
    }
    const name = (document.getElementById('newName').value || '').trim();
    const pin = (document.getElementById('newPin').value || '').trim();
    const saldo = parseFloat(document.getElementById('newSaldo').value);
    const isGuest = !!document.getElementById('newIsGuest').checked;
    const roleSelect = document.getElementById('newRole');
    const role = isAdmin() ? roleSelect.value : 'user';

    if (!name || !pin || isNaN(saldo)) {
      await showModal({ title: 'Fout', message: 'Vul alle velden in!' });
      return;
    }
    if (!/^\d{4}$/.test(pin)) {
      await showModal({ title: 'Fout', message: 'Pincode moet precies 4 cijfers zijn.' });
      return;
    }

    const pinHash = await sha256Hex(pin);
    const type = isGuest ? 'gast' : 'vast';
    accounts.push({ name, pinHash, saldo: Number(saldo), type, role });
    logAction(`Account aangemaakt: ${name} (rol: ${role}, type: ${type})`);
    saveAll();
    loadAccountButtons();
    updateAdminScreen();

    // reset formulier
    document.getElementById('newName').value = '';
    document.getElementById('newPin').value = '';
    document.getElementById('newSaldo').value = '0';
    document.getElementById('newIsGuest').checked = false;
    if (isAdmin()) document.getElementById('newRole').value = 'user';
  }

  async function deleteAccount(i) {
    if (!isAdmin()) return;
    if (isOnlyAdmin(i)) {
      await showModal({ title: 'Fout', message: 'Je kunt de laatste admin niet verwijderen. Wijs eerst een andere admin toe.' });
      return;
    }
    const confirmed = await showModal({
      title: 'Account Verwijderen',
      message: `Account "${accounts[i].name}" verwijderen?`,
      isConfirm: true
    });
    if (!confirmed) return;
    logAction(`Account verwijderd: ${accounts[i].name}`);
    accounts.splice(i, 1);
    saveAll();
    loadAccountButtons();
    updateAdminScreen();
  }

  async function addSaldo(i) {
    const invoer = prompt('Bedrag toevoegen (positief getal):');
    const bedrag = parseFloat(invoer);
    if (!isFinite(bedrag) || bedrag <= 0) {
      await showModal({ title: 'Fout', message: 'Voer een positief getal in.' });
      return;
    }
    accounts[i].saldo += Number(bedrag);
    logAction(`Saldo +€${formatPrice(bedrag)} voor ${accounts[i].name}`, bedrag);
    saveAll();
    loadAccountButtons();
    updateAdminScreen();
  }

  async function changePin(i) {
    if (!isAdmin()) return;
    const val = await securePinModal({ title: `Nieuwe pincode voor ${accounts[i].name}` });
    if (val === null) return;
    accounts[i].pinHash = await sha256Hex(val);
    if ('pin' in accounts[i]) delete accounts[i].pin;
    logAction(`PIN gewijzigd voor ${accounts[i].name}`);
    saveAll();
    updateAdminScreen();
    await showModal({ title: 'Succes', message: 'Pincode bijgewerkt.' });
  }

  async function changeRole(i) {
    if (!isAdmin()) return;
    if (isOnlyAdmin(i)) {
      await showModal({ title: 'Fout', message: 'Deze gebruiker is de laatste admin. Je kunt de laatste admin niet degraderen.' });
      return;
    }
    const huidige = accounts[i].role || 'user';
    const nieuw = await roleModal({ title: `Rol wijzigen voor ${accounts[i].name}`, current: huidige });
    if (nieuw === null) return;
    if (!['user', 'coadmin', 'admin'].includes(nieuw)) {
      await showModal({ title: 'Fout', message: 'Ongeldige rol.' });
      return;
    }

    if (accounts[i].role === 'admin' && nieuw !== 'admin' && getAdminCount() <= 1) {
      await showModal({ title: 'Fout', message: 'Er moet altijd minstens één admin blijven. Maak eerst een andere admin aan.' });
      return;
    }
    if (currentManager && currentManager.index === i && accounts[i].role === 'admin' && nieuw !== 'admin' && getAdminCount() <= 1) {
      await showModal({ title: 'Fout', message: 'Je bent de laatste admin en kunt jezelf niet degraderen. Wijs eerst iemand anders als admin aan.' });
      return;
    }
    accounts[i].role = nieuw;
    logAction(`Rol gewijzigd: ${accounts[i].name} → ${nieuw}`);
    saveAll();
    loadAccountButtons();
    updateAdminScreen();
  }

  async function renameAccount(i) {
    if (!isAdmin()) return;
    const oud = accounts[i].name;
    const nieuw = prompt(`Nieuwe naam voor "${oud}":`, oud);
    if (nieuw === null) return;
    const clean = (nieuw || '').trim();
    if (!clean) {
      await showModal({ title: 'Fout', message: 'Naam mag niet leeg zijn.' });
      return;
    }
    accounts[i].name = clean;
    logAction(`Naam gewijzigd: ${oud} → ${clean}`);
    saveAll();
    loadAccountButtons();
    updateAdminScreen();
  }

  async function addProduct() {
    if (!isAdmin()) return;
    const name = (document.getElementById('prodName').value || '').trim();
    const price = parseFloat(document.getElementById('prodPrice').value);
    const stock = parseInt(document.getElementById('prodStock').value);
    if (!name) {
      await showModal({ title: 'Fout', message: 'Productnaam mag niet leeg zijn.' });
      return;
    }
    if (!isFinite(price) || price < 0) {
      await showModal({ title: 'Fout', message: 'Prijs moet ≥ 0 zijn.' });
      return;
    }
    if (!Number.isInteger(stock) || stock < 0) {
      await showModal({ title: 'Fout', message: 'Voorraad moet een geheel getal ≥ 0 zijn.' });
      return;
    }
    products.push({ name, price: Number(price), stock: Number(stock) });
    logAction(`Product toegevoegd: ${name} (€${formatPrice(price)})`);
    saveAll();
    updateAdminScreen();
    document.getElementById('prodName').value = '';
    document.getElementById('prodPrice').value = '';
    document.getElementById('prodStock').value = '';
  }

  async function deleteProduct(i) {
    if (!isAdmin()) return;
    const confirmed = await showModal({
      title: 'Product Verwijderen',
      message: `Product "${products[i].name}" verwijderen?`,
      isConfirm: true
    });
    if (!confirmed) return;
    logAction(`Product verwijderd: ${products[i].name}`);
    products.splice(i, 1);
    saveAll();
    updateAdminScreen();
  }

  async function restockProduct(i) {
    if (!isAdmin()) return;
    const add = parseInt(prompt(`Aantal bijvullen voor "${products[i].name}" (huidig: ${products[i].stock})`, "0"));
    if (!Number.isInteger(add) || add <= 0) {
      await showModal({ title: 'Fout', message: 'Voer een positief geheel getal in.' });
      return;
    }
    products[i].stock = Math.max(0, products[i].stock + add);
    logAction(`Voorraad +${add} voor ${products[i].name}`);
    saveAll();
    updateAdminScreen();
  }

  async function changePrice(i) {
    if (!isAdmin()) return;
    const nieuw = parseFloat(prompt(`Nieuwe prijs voor "${products[i].name}" (huidig: €${formatPrice(products[i].price)})`, products[i].price));
    if (!isFinite(nieuw) || nieuw < 0) {
      await showModal({ title: 'Fout', message: 'Prijs moet ≥ 0 zijn.' });
      return;
    }
    const oud = products[i].price;
    products[i].price = Number(nieuw);
    logAction(`Prijs gewijzigd: ${products[i].name} €${formatPrice(oud)} → €${formatPrice(nieuw)}`);
    saveAll();
    updateAdminScreen();
  }

  /* ---- Logs ---- */
  async function exportLogsToCSV() {
    if (!(isAdmin() || isCoAdmin())) return;
    if (logs.length === 0) {
      await showModal({ title: 'Fout', message: 'Het logboek is leeg.' });
      return;
    }

    const header = ["Gebruiker", "Product", "Prijs", "Tijd"];
    const esc = v => `"${String(v).replace(/"/g, '""')}"`;
    const rows = logs.map(l => [l.gebruiker, l.product, formatPrice(l.prijs), l.tijd].map(esc).join(','));
    const csv = header.map(esc).join(',') + '\n' + rows.join('\n');

    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'logboek.csv';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);

    logAction('Logboek geëxporteerd (CSV)');
    saveAll();
  }

  async function clearLogs() {
    if (!isAdmin()) return;
    const confirmed = await showModal({ title: 'Logboek Wissen', message: 'Logboek wissen?', isConfirm: true });
    if (!confirmed) return;
    logs = [];
    saveAll();
    updateAdminScreen();
  }

  /* ---- Data import/export/reset ---- */
  function exportAllToJSON() {
    if (!isAdmin()) return;
    const payload = { version: APP_VERSION, exportedAt: new Date().toISOString(), accounts, products, logs };
    const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'fictief-geld-data.json';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    logAction('Data geëxporteerd (JSON)');
    saveAll();
  }

  async function importAllFromJSON(file) {
    if (!isAdmin()) return;
    if (!file) return;
    const r = new FileReader();
    r.onload = async e => {
      try {
        const data = JSON.parse(e.target.result);
        if (!data || !Array.isArray(data.accounts) || !Array.isArray(data.products) || !Array.isArray(data.logs)) {
          await showModal({ title: 'Fout', message: 'Onjuist JSON-formaat.' });
          return;
        }
        const confirmed = await showModal({ title: 'Data Importeren', message: 'Huidige data overschrijven?', isConfirm: true });
        if (!confirmed) return;
        accounts = data.accounts;
        products = data.products;
        logs = data.logs;
        await migratePinsIfNeeded();
        logAction('Data geïmporteerd (JSON)');
        saveAll();
        loadAccountButtons();
        updateAdminScreen();
        await showModal({ title: 'Succes', message: 'Data geïmporteerd.' });
      } catch (err) {
        await showModal({ title: 'Fout', message: 'Kon JSON niet lezen: ' + err.message });
      }
      importFile.value = '';
    };
    r.readAsText(file);
  }

  async function resetAllData() {
    if (!isAdmin()) return;
    const confirmed = await showModal({
      title: 'Data Herstellen',
      message: 'Alle data herstellen naar standaard?',
      isConfirm: true
    });
    if (!confirmed) return;
    logAction('Data reset naar standaard');
    accounts = [
      { name: "Jan", pinHash: "", saldo: 40.00, type: "vast", role: "user" },
      { name: "Piet", pinHash: "", saldo: 5.00, type: "gast", role: "user" },
      { name: "Beheer", pinHash: "", saldo: 0.00, type: "vast", role: "admin" }
    ];
    products = [
      { name: "Chips", price: 0.75, stock: 20 },
      { name: "Bier", price: 0.75, stock: 30 },
      { name: "Cola", price: 1.00, stock: 15 }
    ];
    logs = [];
    (async () => {
      accounts[0].pinHash = await sha256Hex("1234");
      accounts[1].pinHash = await sha256Hex("5678");
      accounts[2].pinHash = await sha256Hex("9999");
      saveAll();
      loadAccountButtons();
      updateAdminScreen();
      await showModal({ title: 'Hersteld', message: 'Alle data is hersteld naar de standaardwaarden.' });
    })();
  }

  /* ---- Wire up ---- */
  adminLoginBtn.addEventListener('click', () => adminLogin());
  userLoginBtn.addEventListener('click', () => checkLogin());
  cancelPinBtn.addEventListener('click', goHome);
  logoutUserBtn.addEventListener('click', goHome);

  logoutAdminBtn.addEventListener('click', () => {
    logAction('Beheeruitlog');
    saveAll();
    goHome();
  });

  adminCode.addEventListener('input', () => digitsOnly(adminCode));
  pincode.addEventListener('input', () => digitsOnly(pincode));

  checkoutBtn.addEventListener('click', checkoutCart);
  clearCartBtn.addEventListener('click', () => {
    initCart();
    updateUserScreen();
  });

  exportCsvBtn.addEventListener('click', exportLogsToCSV);
  clearLogsBtn.addEventListener('click', clearLogs);
  exportJsonBtn.addEventListener('click', exportAllToJSON);
  importJsonBtn.addEventListener('click', () => importFile.click());
  importFile.addEventListener('change', () => importAllFromJSON(importFile.files[0]));
  resetBtn.addEventListener('click', resetAllData);

  const logoImg = document.querySelector('img.logo');
  const watermark = document.querySelector('.watermark');
  logoImg.addEventListener('error', () => {
    logoImg.src = 'https://placehold.co/42x42/333333/ffffff?text=LOGO';
    if (watermark) watermark.style.display = 'none';
  });

  // Init
  (async () => {
    await migratePinsIfNeeded();
    if (accounts.some(a => !a.pinHash)) {
      for (const a of accounts) {
        if (!a.pinHash) {
          a.pinHash = await sha256Hex("0000");
        }
      }
      saveAll();
    }
    loadAccountButtons();
  })();
});
</script>
</body>
</html>
