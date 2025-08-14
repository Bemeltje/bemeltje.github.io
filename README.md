<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Fictief Wallet Systeem</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <meta name="color-scheme" content="light dark">
  <style>
    :root { font-family: Inter, ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji"; }
    .message-box { animation: fadeIn .2s ease-out; }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(-6px) } to { opacity: 1; transform: translateY(0) } }
    .bg-status-red { background-color: #fca5a5; }
    .bg-status-orange { background-color: #fdba74; }
    .modal-overlay{ background: rgba(0,0,0,.5); backdrop-filter: blur(4px); z-index: 50; }
    .modal-content{ animation: pop .18s cubic-bezier(.2,.8,.2,1); z-index: 51; }
    @keyframes pop { from{ transform: scale(.98); opacity: 0 } to{ transform: scale(1); opacity: 1 } }
    button:disabled { opacity:.5; cursor:not-allowed }
  </style>
</head>
<body class="bg-gray-100 min-h-screen p-4 flex items-center justify-center">
  <div id="app" class="w-full max-w-3xl bg-white rounded-2xl shadow-xl p-6 md:p-8">
    <!-- App wordt via JS gerenderd -->
  </div>

  <!-- Edit/Confirm modals -->
  <div id="modal-overlay" class="fixed inset-0 hidden items-center justify-center modal-overlay">
    <div class="modal-content bg-white rounded-2xl shadow-2xl w-full max-w-md p-6">
      <h3 id="modal-title" class="text-2xl font-bold mb-4 text-gray-900">Bewerken</h3>
      <div id="modal-body" class="space-y-4"></div>
    </div>
  </div>
  <div id="confirm-overlay" class="fixed inset-0 hidden items-center justify-center modal-overlay">
    <div class="modal-content bg-white rounded-2xl shadow-2xl w-full max-w-sm p-6">
      <h3 id="confirm-title" class="text-xl font-bold mb-2 text-gray-900">Bevestigen</h3>
      <p id="confirm-msg" class="text-gray-600 mb-6">Weet je het zeker?</p>
      <div class="flex justify-end gap-3">
        <button id="confirm-cancel" class="bg-gray-200 hover:bg-gray-300 text-gray-900 font-medium px-4 py-2 rounded-lg">Annuleren</button>
        <button id="confirm-ok" class="bg-red-600 hover:bg-red-700 text-white font-semibold px-4 py-2 rounded-lg">Bevestigen</button>
      </div>
    </div>
  </div>

  <script>
  // ====== Instellingen ======
  const ADMIN_PIN = '1111';        // Pas aan indien gewenst
  const VASTE_ACCOUNT_LIMIT = -25;  // Kredietlimiet voor vaste accounts
  const GAST_ACCOUNT_LIMIT = 0;     // Min. saldo voor gastaccounts

  // ====== Storage helpers ======
  const LS_KEYS = {
    accounts: 'wallet_accounts_v1',
    products: 'wallet_products_v1',
    tx: 'wallet_transactions_v1'
  };
  const load = (key, fallback) => {
    try { return JSON.parse(localStorage.getItem(key)) ?? fallback; } catch { return fallback; }
  };
  const save = (key, value) => localStorage.setItem(key, JSON.stringify(value));

  // ====== App state ======
  let accounts = load(LS_KEYS.accounts, []);
  let products = load(LS_KEYS.products, [
    { id: 1, name: 'Zakje chips', price: 0.75, stock: 50 },
    { id: 2, name: 'Bierviltje',  price: 0.75, stock: 50 },
    { id: 3, name: 'Flesje cola', price: 1.00, stock: 50 },
  ]);
  let transactions = load(LS_KEYS.tx, []);

  // Maak admin account als die nog niet bestaat
  if (!accounts.find(a => a.pin === ADMIN_PIN)) {
    const nextId = accounts.length ? Math.max(...accounts.map(a => a.id)) + 1 : 1;
    accounts.push({ id: nextId, name: 'Beheerder', pin: ADMIN_PIN, balance: 100.00, type: 'vaste' });
    save(LS_KEYS.accounts, accounts);
  }

  let loggedIn = null;  // {id, name, pin, balance, type}
  let isAdmin = false;
  let qty = {}; // productId -> number

  const app = document.getElementById('app');

  // ====== UI helpers ======
  const toast = (msg, type='info') => {
    const box = document.createElement('div');
    const colors = type==='error' ? 'bg-red-500' : (type==='ok' ? 'bg-emerald-500' : 'bg-indigo-600');
    box.className = `message-box ${colors} text-white rounded-lg px-3 py-2 text-sm font-medium mb-3`;
    box.textContent = msg;
    app.prepend(box);
    setTimeout(()=> box.remove(), 3000);
  };

  const balanceBadgeClass = (balance, type) => {
    const limit = type==='vaste' ? VASTE_ACCOUNT_LIMIT : GAST_ACCOUNT_LIMIT;
    if (balance < limit) return 'bg-status-red text-white';
    if (balance < 5) return 'bg-status-orange text-white';
    return 'bg-gray-200 text-gray-900';
  };

  const fmt = (n) => n.toFixed(2);

  // ====== Views ======
  const viewHome = () => {
    isAdmin = false; loggedIn = null; qty = {};
    const nonAdmin = accounts.filter(a => a.pin !== ADMIN_PIN);
    app.innerHTML = `
      <h1 class="text-3xl font-extrabold text-gray-900 mb-1">Fictief Wallet Systeem</h1>
      <p class="text-gray-600 mb-6">Selecteer een account en log in met je pincode.</p>

      ${nonAdmin.length ? `
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
          ${nonAdmin.map(a => `
            <button data-id="${a.id}" class="account-btn flex items-center justify-between gap-3 p-3 rounded-xl shadow-sm border hover:shadow transition bg-white">
              <span class="font-semibold text-gray-900">${a.name}</span>
              <span class="text-xs px-2 py-1 rounded ${balanceBadgeClass(a.balance, a.type)}">€${fmt(a.balance)}</span>
            </button>
          `).join('')}
        </div>
      ` : `<div class="p-4 rounded-xl bg-gray-50 border">Nog geen accounts. Log in als beheerder om te beginnen.</div>`}

      <div class="mt-6">
        <button id="btn-admin" class="px-4 py-2 rounded-lg bg-gray-200 hover:bg-gray-300 text-gray-900 font-semibold">Admin login</button>
      </div>
    `;

    document.querySelectorAll('.account-btn').forEach(btn => {
      btn.addEventListener('click', (e) => {
        const id = parseInt(e.currentTarget.getAttribute('data-id'));
        loggedIn = accounts.find(a => a.id === id) || null;
        viewPin();
      });
    });
    document.getElementById('btn-admin').addEventListener('click', () => {
      loggedIn = accounts.find(a => a.pin === ADMIN_PIN);
      viewPin();
    });
  };

  const viewPin = () => {
    if (!loggedIn) return viewHome();
    app.innerHTML = `
      <h1 class="text-3xl font-extrabold text-gray-900 mb-1">Fictief Wallet Systeem</h1>
      <p class="text-gray-600 mb-6">Welkom, <span class="font-semibold">${loggedIn.name}</span></p>
      <form id="pin-form" class="space-y-4">
        <input id="pin" type="password" maxlength="4" inputmode="numeric" autocomplete="one-time-code"
               class="w-full p-4 text-center text-2xl tracking-widest rounded-xl border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
               placeholder="Voer je PIN in" />
        <div class="grid grid-cols-2 gap-3">
          <button class="px-4 py-3 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold" type="submit">Inloggen</button>
          <button id="back" class="px-4 py-3 rounded-xl bg-gray-200 hover:bg-gray-300 text-gray-900 font-semibold" type="button">Terug</button>
        </div>
      </form>
    `;
    document.getElementById('pin-form').addEventListener('submit', (e) => {
      e.preventDefault();
      const pin = document.getElementById('pin').value.trim();
      if (pin === loggedIn.pin) {
        isAdmin = pin === ADMIN_PIN;
        toast(`Welkom, ${loggedIn.name}!`, 'ok');
        isAdmin ? viewAdmin() : viewUser();
      } else {
        toast('Ongeldige PIN. Probeer opnieuw.', 'error');
      }
    });
    document.getElementById('back').addEventListener('click', viewHome);
  };

  const viewAdmin = () => {
    // Kleine helper voor options
    const options = (vals, v) => vals.map(o => `<option value="${o}" ${o===v?'selected':''}>${o}</option>`).join('');

    app.innerHTML = `
      <div class="flex items-center justify-between mb-4">
        <div>
          <h1 class="text-3xl font-extrabold text-gray-900">Beheerderspaneel</h1>
          <p class="text-gray-600">Ingelogd als ${loggedIn.name}</p>
        </div>
        <button id="logout" class="px-3 py-2 rounded-lg bg-red-600 hover:bg-red-700 text-white font-semibold">Uitloggen</button>
      </div>

      <!-- Geld toevoegen -->
      <section class="mb-6">
        <h2 class="text-xl font-bold text-gray-900 mb-2">Geld toevoegen</h2>
        <div class="flex gap-2 items-stretch">
          <select id="sel-user" class="flex-1 p-3 rounded-xl border-2">
            <option value="">Kies gebruiker…</option>
            ${accounts.map(a => `<option value="${a.id}">${a.name} (${a.type}) — €${fmt(a.balance)}</option>`).join('')}
          </select>
          <input id="amount" type="number" step="0.01" placeholder="Bedrag" class="w-40 p-3 rounded-xl border-2" />
          <button id="btn-add" class="px-4 py-3 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold">Toevoegen</button>
        </div>
      </section>

      <!-- Productbeheer -->
      <section class="mb-6">
        <h2 class="text-xl font-bold text-gray-900 mb-2">Productbeheer</h2>
        <div class="bg-gray-50 border rounded-xl p-3 max-h-56 overflow-auto" id="admin-products">
          ${products.length ? products.map(p => `
            <div class="flex items-center justify-between gap-3 py-2 border-b last:border-b-0 text-sm">
              <div class="font-medium text-gray-900">${p.name}</div>
              <div class="text-gray-600">€${fmt(p.price)}</div>
              <div class="text-gray-600">Voorraad: ${p.stock}</div>
              <div class="flex gap-2">
                <button data-id="${p.id}" class="btn-edit-product text-indigo-600 hover:underline">Bewerk</button>
                <button data-id="${p.id}" class="btn-del-product text-red-600 hover:underline">Verwijder</button>
              </div>
            </div>
          `).join('') : '<div class="text-gray-500">Geen producten.</div>'}
        </div>
        <form id="form-add-prod" class="grid grid-cols-1 sm:grid-cols-3 gap-2 mt-3">
          <input id="prod-name" class="p-3 rounded-xl border-2" placeholder="Naam" />
          <input id="prod-price" type="number" step="0.01" class="p-3 rounded-xl border-2" placeholder="Prijs" />
          <div class="flex gap-2">
            <input id="prod-stock" type="number" class="p-3 rounded-xl border-2 w-full" placeholder="Voorraad" />
            <button class="px-4 py-3 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold">Toevoegen</button>
          </div>
        </form>
      </section>

      <!-- Accounts overzicht & bewerk -->
      <section class="mb-6">
        <h2 class="text-xl font-bold text-gray-900 mb-2">Accounts</h2>
        <div class="bg-gray-50 border rounded-xl p-3 max-h-56 overflow-auto" id="admin-accounts">
          ${accounts.map(a => `
            <div class="flex items-center justify-between gap-3 py-2 border-b last:border-b-0 text-sm">
              <div class="flex items-center gap-2">
                <div class="font-semibold text-gray-900">${a.name}</div>
                <span class="text-gray-500">(${a.type})</span>
              </div>
              <div class="text-gray-600">€${fmt(a.balance)}</div>
              <button data-id="${a.id}" class="btn-edit-account text-indigo-600 hover:underline">Bewerk</button>
            </div>
          `).join('')}
        </div>
        <form id="form-add-acc" class="grid grid-cols-1 sm:grid-cols-4 gap-2 mt-3">
          <input id="acc-name" class="p-3 rounded-xl border-2" placeholder="Naam" />
          <select id="acc-type" class="p-3 rounded-xl border-2">
            <option value="vaste">Vast account</option>
            <option value="gast">Gast account</option>
          </select>
          <input id="acc-pin" type="password" maxlength="4" class="p-3 rounded-xl border-2" placeholder="PIN (4 cijfers)" />
          <button class="px-4 py-3 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold">Nieuw account</button>
        </form>
      </section>

      <!-- Transacties -->
      <section class="mb-6">
        <h2 class="text-xl font-bold text-gray-900 mb-2">Transactielogboek</h2>
        <div class="bg-gray-50 border rounded-xl p-3 max-h-56 overflow-auto" id="tx-log">
          ${transactions.length ? transactions.slice().reverse().map(t => `
            <div class="py-2 border-b last:border-b-0 text-sm text-gray-800">
              <div class="font-medium">${t.user}: €${fmt(t.amount)} — ${t.items}</div>
              <div class="text-xs text-gray-500">${new Date(t.date).toLocaleString('nl-NL')}</div>
            </div>
          `).join('') : '<div class="text-gray-500">Geen transacties.</div>'}
        </div>
      </section>

      <!-- Backup & reset -->
      <section class="mb-2">
        <div class="flex flex-wrap gap-2">
          <button id="btn-export" class="px-4 py-2 rounded-lg border font-medium">Exporteer data (JSON)</button>
          <label class="px-4 py-2 rounded-lg border font-medium cursor-pointer">
            Importeer data
            <input id="input-import" type="file" accept="application/json" class="hidden" />
          </label>
          <button id="btn-reset" class="px-4 py-2 rounded-lg bg-red-50 text-red-700 border border-red-200 hover:bg-red-100 font-semibold">Alles resetten</button>
        </div>
      </section>
    `;

    // Events
    document.getElementById('logout').addEventListener('click', viewHome);

    // Geld toevoegen
    document.getElementById('btn-add').addEventListener('click', () => {
      const id = parseInt(document.getElementById('sel-user').value);
      const amt = parseFloat(document.getElementById('amount').value);
      const u = accounts.find(a => a.id === id);
      if (!u || isNaN(amt) || amt <= 0) return toast('Kies gebruiker en geldig bedrag.', 'error');
      u.balance = +(u.balance + amt).toFixed(2);
      save(LS_KEYS.accounts, accounts);
      viewAdmin();
      toast(`€${fmt(amt)} toegevoegd aan ${u.name}.`, 'ok');
    });

    // Product toevoegen
    document.getElementById('form-add-prod').addEventListener('submit', (e) => {
      e.preventDefault();
      const name = document.getElementById('prod-name').value.trim();
      const price = parseFloat(document.getElementById('prod-price').value);
      const stock = parseInt(document.getElementById('prod-stock').value, 10);
      if (!name || isNaN(price) || price <= 0 || isNaN(stock) || stock < 0) return toast('Ongeldige productgegevens.', 'error');
      const nextId = products.length ? Math.max(...products.map(p => p.id)) + 1 : 1;
      products.push({ id: nextId, name, price, stock });
      save(LS_KEYS.products, products);
      viewAdmin();
      toast(`Product ‘${name}’ toegevoegd.`, 'ok');
    });

    // Product edit/delete (event delegation)
    document.getElementById('admin-products').addEventListener('click', (e) => {
      const editBtn = e.target.closest('.btn-edit-product');
      const delBtn  = e.target.closest('.btn-del-product');
      if (editBtn) {
        const id = parseInt(editBtn.dataset.id);
        const p = products.find(x => x.id === id);
        showEditModal({
          title: `Bewerk product: ${p.name}`,
          fields: [
            { id:'name', label:'Naam',  type:'text',   value:p.name },
            { id:'price',label:'Prijs', type:'number', value:p.price, step:'0.01' },
            { id:'stock',label:'Voorraad', type:'number', value:p.stock }
          ],
          onSave: (fd) => {
            const name = (fd.name||'').trim();
            const price = parseFloat(fd.price);
            const stock = parseInt(fd.stock,10);
            if (!name || isNaN(price) || price<=0 || isNaN(stock) || stock<0) return toast('Ongeldige invoer.', 'error');
            Object.assign(p, { name, price, stock });
            save(LS_KEYS.products, products);
            viewAdmin();
            toast('Product bijgewerkt.', 'ok');
          }
        });
      }
      if (delBtn) {
        const id = parseInt(delBtn.dataset.id);
        const p = products.find(x => x.id === id);
        showConfirm(`Verwijderen`, `Product ‘${p.name}’ verwijderen?`, () => {
          products = products.filter(x => x.id !== id);
          save(LS_KEYS.products, products);
          viewAdmin();
          toast('Product verwijderd.', 'ok');
        });
      }
    });

    // Account toevoegen
    document.getElementById('form-add-acc').addEventListener('submit', (e) => {
      e.preventDefault();
      const name = document.getElementById('acc-name').value.trim();
      const type = document.getElementById('acc-type').value;
      const pin  = document.getElementById('acc-pin').value.trim();
      if (!name || !pin || pin.length !== 4 || /\D/.test(pin)) return toast('Naam en 4-cijferige PIN vereist.', 'error');
      if (accounts.some(a => a.pin === pin)) return toast('PIN al in gebruik.', 'error');
      const nextId = accounts.length ? Math.max(...accounts.map(a => a.id)) + 1 : 1;
      accounts.push({ id: nextId, name, pin, balance: 0, type });
      save(LS_KEYS.accounts, accounts);
      viewAdmin();
      toast(`Account ‘${name}’ aangemaakt.`, 'ok');
    });

    // Account bewerken
    document.getElementById('admin-accounts').addEventListener('click', (e) => {
      const btn = e.target.closest('.btn-edit-account');
      if (!btn) return;
      const id = parseInt(btn.dataset.id);
      const acc = accounts.find(a => a.id === id);
      showEditModal({
        title: `Bewerk account: ${acc.name}`,
        fields: [
          { id:'name', label:'Naam', type:'text', value: acc.name },
          { id:'pin',  label:'PIN (4 cijfers)', type:'password', value: acc.pin, maxlength:'4' },
          { id:'type', label:'Type', type:'select', value: acc.type, options:['vaste','gast'] }
        ],
        onSave: (fd) => {
          const name = (fd.name||'').trim();
          const pin  = (fd.pin||'').trim();
          const type = fd.type;
          if (!name || !pin || pin.length!==4 || /\D/.test(pin)) return toast('Ongeldige invoer.', 'error');
          if (accounts.some(a => a.pin===pin && a.id!==id)) return toast('PIN al in gebruik.', 'error');
          Object.assign(acc, { name, pin, type });
          save(LS_KEYS.accounts, accounts);
          viewAdmin();
          toast('Account bijgewerkt.', 'ok');
        }
      });
    });

    // Export / Import / Reset
    document.getElementById('btn-export').addEventListener('click', () => {
      const data = { accounts, products, transactions };
      const blob = new Blob([JSON.stringify(data, null, 2)], { type:'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = `wallet-backup-${new Date().toISOString().slice(0,10)}.json`;
      a.click(); URL.revokeObjectURL(url);
    });

    document.getElementById('input-import').addEventListener('change', async (e) => {
      const file = e.target.files?.[0];
      if (!file) return;
      try {
        const text = await file.text();
        const data = JSON.parse(text);
        if (!data.accounts || !data.products || !data.transactions) throw new Error('Ongeldig bestand');
        accounts = data.accounts; products = data.products; transactions = data.transactions;
        save(LS_KEYS.accounts, accounts); save(LS_KEYS.products, products); save(LS_KEYS.tx, transactions);
        viewAdmin();
        toast('Backup geïmporteerd.', 'ok');
      } catch (err) { toast('Import mislukt: ongeldige JSON.', 'error'); }
      finally { e.target.value = ''; }
    });

    document.getElementById('btn-reset').addEventListener('click', () => {
      showConfirm('Alles resetten', 'Alle data (accounts, producten, transacties) wissen? Dit kan niet ongedaan worden gemaakt.', () => {
        localStorage.removeItem(LS_KEYS.accounts);
        localStorage.removeItem(LS_KEYS.products);
        localStorage.removeItem(LS_KEYS.tx);
        accounts = []; products = []; transactions = [];
        // Admin opnieuw aanmaken
        accounts.push({ id: 1, name: 'Beheerder', pin: ADMIN_PIN, balance: 100.00, type: 'vaste' });
        save(LS_KEYS.accounts, accounts);
        viewAdmin();
        toast('Alles gereset.', 'ok');
      });
    });
  };

  const viewUser = () => {
    // Init qty map
    qty = Object.fromEntries(products.map(p => [p.id, 0]));

    const total = () => {
      return Object.entries(qty).reduce((sum, [pid, q]) => {
        const p = products.find(x => x.id === +pid); return sum + (p ? p.price * q : 0);
      }, 0);
    };

    const render = () => {
      app.innerHTML = `
        <div class="flex items-center justify-between mb-4">
          <div>
            <h1 class="text-3xl font-extrabold text-gray-900">Winkel</h1>
            <p class="text-gray-600">Welkom, ${loggedIn.name}</p>
          </div>
          <button id="back" class="px-3 py-2 rounded-lg bg-gray-200 hover:bg-gray-300 text-gray-900 font-semibold">Terug</button>
        </div>

        <div class="text-xl font-bold mb-4 text-indigo-700">Saldo: €<span id="bal">${fmt(loggedIn.balance)}</span></div>

        <div id="plist" class="grid grid-cols-1 sm:grid-cols-2 gap-3 mb-4">
          ${products.map(p => `
            <div class="bg-gray-50 border rounded-xl p-4 flex items-center justify-between gap-3">
              <div>
                <div class="font-semibold text-gray-900">${p.name}</div>
                <div class="text-gray-600 text-sm">€${fmt(p.price)}</div>
                <div class="text-gray-500 text-xs">Op voorraad: ${p.stock}</div>
              </div>
              <input data-id="${p.id}" type="number" min="0" max="${p.stock}" value="${qty[p.id]||0}"
                     class="qty w-20 p-2 text-center rounded-lg border" />
            </div>
          `).join('')}
        </div>

        <div class="text-lg font-bold mb-4">Totaal: €<span id="tot">${fmt(total())}</span></div>
        <button id="buy" class="w-full px-4 py-3 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white font-semibold">Aankopen</button>
      `;

      // Wire up events
      document.getElementById('back').addEventListener('click', viewHome);
      document.querySelectorAll('.qty').forEach(inp => {
        inp.addEventListener('input', (e) => {
          const id = parseInt(e.currentTarget.dataset.id);
          let v = parseInt(e.currentTarget.value, 10);
          if (isNaN(v) || v < 0) v = 0;
          const p = products.find(x => x.id === id);
          if (v > p.stock) v = p.stock; // cap aan voorraad
          qty[id] = v;
          e.currentTarget.value = v;
          document.getElementById('tot').textContent = fmt(total());
        });
      });
      document.getElementById('buy').addEventListener('click', handleBuy);

      // saldo badge kleur
      const bal = document.getElementById('bal');
      const cls = balanceBadgeClass(loggedIn.balance, loggedIn.type);
      bal.parentElement.classList.add(...cls.split(' '), 'inline-flex', 'px-2', 'py-1', 'rounded');
    };

    const handleBuy = () => {
      const amount = +total().toFixed(2);
      if (amount <= 0) return toast('Geen items geselecteerd.', 'error');

      // Check per product voorraad
      for (const [pid, q] of Object.entries(qty)) {
        const p = products.find(x => x.id === +pid);
        if (!p) continue;
        if (q > p.stock) return toast(`Onvoldoende voorraad voor ${p.name}.`, 'error');
      }

      const limit = loggedIn.type === 'vaste' ? VASTE_ACCOUNT_LIMIT : GAST_ACCOUNT_LIMIT;
      const newBal = +(loggedIn.balance - amount).toFixed(2);
      if (newBal < limit) return toast(`Onvoldoende saldo. Minimaal toegestaan: €${fmt(limit)}.`, 'error');

      // Pas voorraad aan
      for (const [pid, q] of Object.entries(qty)) {
        if (q <= 0) continue;
        const p = products.find(x => x.id === +pid);
        p.stock -= q;
      }
      save(LS_KEYS.products, products);

      // Update account saldo
      const acc = accounts.find(a => a.id === loggedIn.id);
      acc.balance = newBal; loggedIn.balance = newBal;
      save(LS_KEYS.accounts, accounts);

      // Log transactie
      const items = Object.entries(qty)
        .filter(([,q]) => q>0)
        .map(([pid,q]) => {
          const p = products.find(x => x.id === +pid); return `${p.name} (${q}x)`;
        }).join(', ');
      const nextId = transactions.length ? Math.max(...transactions.map(t => t.id)) + 1 : 1;
      transactions.push({ id: nextId, user: acc.name, amount, items, date: new Date().toISOString() });
      save(LS_KEYS.tx, transactions);

      // Reset qty inputs
      qty = Object.fromEntries(products.map(p => [p.id, 0]));
      render();
      toast(`Aankoop van €${fmt(amount)} succesvol!`, 'ok');
    };

    render();
  };

  // ====== Modals ======
  const showEditModal = ({ title, fields, onSave }) => {
    const overlay = document.getElementById('modal-overlay');
    const body = document.getElementById('modal-body');
    document.getElementById('modal-title').textContent = title;
    body.innerHTML = `
      <form id="edit-form" class="space-y-4">
        ${fields.map(f => `
          <div>
            <label for="${f.id}" class="block text-sm font-medium text-gray-700">${f.label}</label>
            ${f.type === 'select' ? `
              <select id="${f.id}" class="mt-1 w-full p-2 rounded-lg border">
                ${f.options.map(o => `<option value="${o}" ${o===f.value?'selected':''}>${o}</option>`).join('')}
              </select>
            ` : `
              <input id="${f.id}" type="${f.type}" value="${f.value}"
                     ${f.maxlength?`maxlength="${f.maxlength}"`:''} ${f.step?`step="${f.step}"`:''}
                     class="mt-1 w-full p-2 rounded-lg border" />
            `}
          </div>
        `).join('')}
        <div class="flex justify-end gap-2 pt-2">
          <button type="button" id="edit-cancel" class="px-4 py-2 rounded-lg bg-gray-200 hover:bg-gray-300 font-medium">Annuleren</button>
          <button type="submit" class="px-4 py-2 rounded-lg bg-indigo-600 hover:bg-indigo-700 text-white font-semibold">Opslaan</button>
        </div>
      </form>
    `;
    overlay.classList.remove('hidden'); overlay.classList.add('flex');
    document.getElementById('edit-cancel').onclick = hideEditModal;
    document.getElementById('edit-form').onsubmit = (e) => {
      e.preventDefault();
      const fd = Object.fromEntries(new FormData(e.target).entries());
      onSave(fd); hideEditModal();
    };
  };
  const hideEditModal = () => {
    const overlay = document.getElementById('modal-overlay');
    overlay.classList.add('hidden'); overlay.classList.remove('flex');
  };

  const showConfirm = (title, msg, onOk) => {
    const ov = document.getElementById('confirm-overlay');
    document.getElementById('confirm-title').textContent = title;
    document.getElementById('confirm-msg').textContent = msg;
    ov.classList.remove('hidden'); ov.classList.add('flex');
    const ok = document.getElementById('confirm-ok');
    const cancel = document.getElementById('confirm-cancel');
    ok.onclick = () => { onOk(); hideConfirm(); };
    cancel.onclick = hideConfirm;
  };
  const hideConfirm = () => {
    const ov = document.getElementById('confirm-overlay');
    ov.classList.add('hidden'); ov.classList.remove('flex');
  };

  // Start app
  viewHome();
  </script>
</body>
</html>
