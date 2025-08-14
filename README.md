<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fictief Wallet Systeem</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .message-box {
            animation: fadeIn 0.5s;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .bg-status-red { background-color: #fca5a5; }
        .bg-status-orange { background-color: #fdba74; }

        /* Stijlen voor de modal vensters */
        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.5);
            backdrop-filter: blur(5px);
            z-index: 100;
        }
        .modal-content {
            animation: slideIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            z-index: 101;
        }
        @keyframes slideIn {
            from { transform: scale(0.9); opacity: 0; }
            to { transform: scale(1); opacity: 1; }
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div id="app-container" class="bg-white p-8 rounded-xl shadow-lg w-full max-w-xl text-center relative">
        <!-- De inhoud van de applicatie wordt hier met JavaScript geladen -->
    </div>
    
    <!-- Modale vensters voor bewerken en bevestigen -->
    <div id="modal-container" class="fixed inset-0 hidden items-center justify-center modal-overlay">
        <div id="edit-modal" class="bg-white p-8 rounded-xl shadow-2xl w-full max-w-md modal-content">
            <h3 id="modal-title" class="text-2xl font-bold mb-4 text-gray-800">Bewerk</h3>
            <div id="modal-body" class="space-y-4">
                <!-- Formulier voor bewerkingen wordt hier dynamisch geladen -->
            </div>
        </div>
    </div>

    <!-- Confirm modal -->
    <div id="confirm-modal-container" class="fixed inset-0 hidden items-center justify-center modal-overlay">
        <div id="confirm-modal" class="bg-white p-8 rounded-xl shadow-2xl w-full max-w-sm modal-content text-center">
            <h3 id="confirm-title" class="text-xl font-bold mb-4 text-gray-800">Bevestig</h3>
            <p id="confirm-message" class="mb-6 text-gray-600">Weet je het zeker?</p>
            <div class="flex justify-end gap-4">
                <button id="confirm-cancel" class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-lg transition-colors">Annuleren</button>
                <button id="confirm-ok" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg transition-colors">Bevestigen</button>
            </div>
        </div>
    </div>


    <script>
        // Start de applicatie zodra de pagina geladen is
        document.addEventListener('DOMContentLoaded', () => {
            const appContainer = document.getElementById('app-container');
            const ADMIN_PIN = '1111';
            const VASTE_ACCOUNT_LIMIT = -25;
            const GAST_ACCOUNT_LIMIT = 0;

            // Haal de accounts uit localStorage of initialiseer ze
            let accounts = JSON.parse(localStorage.getItem('fictional-accounts')) || [];

            // Haal de producten uit localStorage of initialiseer ze
            let products = JSON.parse(localStorage.getItem('fictional-products')) || [
                { id: 1, name: 'Zakje chips', price: 0.75, stock: 50 },
                { id: 2, name: 'Bierveltje', price: 0.75, stock: 50 },
                { id: 3, name: 'Flesje cola', price: 1.00, stock: 50 }
            ];

            // Haal de transacties uit localStorage of initialiseer ze
            let transactions = JSON.parse(localStorage.getItem('fictional-transactions')) || [];
            
            // Functie om accounts op te slaan in localStorage
            const saveAccounts = () => {
                localStorage.setItem('fictional-accounts', JSON.stringify(accounts));
            };

            // Functie om producten op te slaan in localStorage
            const saveProducts = () => {
                localStorage.setItem('fictional-products', JSON.stringify(products));
            };

            // Functie om transacties op te slaan in localStorage
            const saveTransactions = () => {
                localStorage.setItem('fictional-transactions', JSON.stringify(transactions));
            };

            // Voeg een standaard beheerdersaccount toe als het er nog niet is
            if (!accounts.find(acc => acc.pin === ADMIN_PIN)) {
                accounts.push({ id: accounts.length > 0 ? Math.max(...accounts.map(acc => acc.id)) + 1 : 1, name: 'Sjoerd (Admin)', pin: ADMIN_PIN, balance: 100.00, type: 'vaste' });
                saveAccounts();
            }

            let loggedInUser = null;
            let isAdminMode = false;
            let quantities = {};
            let selectedAdminUser = null;


            // Functie om een bericht weer te geven
            const showMessage = (msg, isError = false) => {
                const messageBox = document.createElement('div');
                messageBox.className = `message-box mb-4 text-sm font-medium text-white p-3 rounded-lg ${isError ? 'bg-red-500' : 'bg-indigo-500'}`;
                messageBox.textContent = msg;
                appContainer.insertBefore(messageBox, appContainer.firstChild);

                setTimeout(() => {
                    messageBox.remove();
                }, 3000);
            };

            // Bepaal de kleurklasse op basis van het saldo en accounttype
            const getBalanceColorClass = (balance, type) => {
                if (balance < (type === 'vaste' ? VASTE_ACCOUNT_LIMIT : GAST_ACCOUNT_LIMIT)) {
                    return 'bg-status-red text-white';
                }
                if (balance < 5) {
                    return 'bg-status-orange text-white';
                }
                return 'bg-gray-200 text-gray-800';
            };

            // Functie voor uitloggen
            const handleLogout = () => {
                loggedInUser = null;
                isAdminMode = false;
                renderHomeView();
                showMessage('U bent uitgelogd.');
            };

            // Functie om de homepagina te renderen (account selectie)
            const renderHomeView = () => {
                appContainer.innerHTML = `
                    <h1 class="text-3xl font-bold mb-6 text-gray-800">Fictief Wallet Systeem</h1>
                    <h2 class="text-2xl font-semibold mb-4 text-gray-700">Selecteer uw account</h2>
                    ${accounts.filter(acc => acc.pin !== ADMIN_PIN).length > 0 ? 
                        `<div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-6">
                            ${accounts.filter(acc => acc.pin !== ADMIN_PIN).map(acc => `
                                <button data-account-id="${acc.id}" class="select-account-button font-bold p-3 rounded-lg transition-colors transform hover:scale-105 ${getBalanceColorClass(acc.balance, acc.type)}">
                                    <span>${acc.name}</span>
                                </button>
                            `).join('')}
                        </div>` :
                        '<p class="text-gray-500">Er zijn nog geen accounts aangemaakt. Log in als beheerder om te beginnen.</p>'
                    }
                    <div class="mt-6">
                        <button type="button" id="admin-login-button" class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold p-3 rounded-lg transition-colors">
                            Admin Login
                        </button>
                    </div>
                `;

                // Event listener voor het selecteren van een account
                document.querySelectorAll('.select-account-button').forEach(button => {
                    button.addEventListener('click', (e) => {
                        const accountId = parseInt(e.target.dataset.accountId);
                        loggedInUser = accounts.find(acc => acc.id === accountId);
                        renderPinView();
                    });
                });

                // Event listener voor de admin login knop
                document.getElementById('admin-login-button').addEventListener('click', () => {
                    loggedInUser = accounts.find(acc => acc.pin === ADMIN_PIN);
                    renderPinView();
                });
            };

            // Functie om de PIN-invoerpagina te renderen
            const renderPinView = () => {
                if (!loggedInUser) {
                    renderHomeView();
                    return;
                }
                appContainer.innerHTML = `
                    <h1 class="text-3xl font-bold mb-6 text-gray-800">Fictief Wallet Systeem</h1>
                    <h2 class="text-2xl font-semibold mb-4 text-gray-700">Welkom, ${loggedInUser.name}!</h2>
                    <form id="pin-form" class="space-y-4">
                        <input
                            type="password"
                            id="pin-input"
                            placeholder="Voer uw PIN in"
                            maxlength="4"
                            class="w-full p-4 text-center text-xl tracking-widest rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                        />
                        <div class="grid grid-cols-2 gap-4 mt-6">
                            <button
                                type="submit"
                                class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold p-3 rounded-lg transition-colors transform hover:scale-105"
                            >
                                Inloggen
                            </button>
                            <button
                                type="button"
                                id="back-button"
                                class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold p-3 rounded-lg transition-colors"
                            >
                                Terug
                            </button>
                        </div>
                    </form>
                `;

                document.getElementById('pin-form').addEventListener('submit', (e) => {
                    e.preventDefault();
                    const pinInput = document.getElementById('pin-input').value;
                    if (pinInput === loggedInUser.pin) {
                        if (loggedInUser.pin === ADMIN_PIN) {
                            isAdminMode = true;
                            renderAdminView();
                        } else {
                            renderUserDashboard();
                        }
                        showMessage(`Welkom, ${loggedInUser.name}!`);
                    } else {
                        showMessage('Ongeldige PIN. Probeer het opnieuw.', true);
                    }
                });

                document.getElementById('back-button').addEventListener('click', () => {
                    loggedInUser = null;
                    renderHomeView();
                });
            };
            
            // Functie om de beheerderspagina te renderen
            const renderAdminView = () => {
                appContainer.innerHTML = `
                    <h1 class="text-3xl font-bold mb-6 text-gray-800">Fictief Wallet Systeem</h1>
                    <h2 class="text-2xl font-semibold mb-4 text-gray-700">Beheerderspaneel</h2>
                    
                    <!-- Sectie: Geld toevoegen aan individuele accounts -->
                    <div class="mb-4 text-left">
                        <label class="block text-gray-700 font-semibold mb-2">
                            Geld toevoegen aan account:
                        </label>
                        <select id="admin-user-select" class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 mb-2">
                            <option value="">Selecteer een gebruiker</option>
                            ${accounts.map(acc => `<option value="${acc.id}">${acc.name} (${acc.type}) - €${acc.balance.toFixed(2)}</option>`).join('')}
                        </select>
                        <div class="flex">
                            <input
                                type="number"
                                id="amount-input-single"
                                placeholder="Bedrag"
                                class="flex-grow p-3 rounded-l-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                            />
                            <button
                                id="add-money-button-single"
                                class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold p-3 rounded-r-lg transition-colors"
                            >
                                Toevoegen
                            </button>
                        </div>
                    </div>
                    
                    <!-- Sectie: Productbeheer -->
                    <div class="mb-6 text-left">
                        <h3 class="text-xl font-semibold mb-2 text-gray-700">Productbeheer</h3>
                        <div id="product-list-admin" class="bg-gray-100 p-4 rounded-lg mb-4 max-h-48 overflow-y-auto">
                            ${products.length > 0 ? products.map(product => `
                                <div class="flex justify-between items-center py-2 border-b last:border-b-0 border-gray-200">
                                    <span class="font-medium text-gray-700">${product.name}</span>
                                    <span class="text-gray-500">€${product.price.toFixed(2)}</span>
                                    <span class="text-gray-500">Voorraad: ${product.stock}</span>
                                    <div class="flex gap-2">
                                        <button data-product-id="${product.id}" class="edit-product-button text-sm text-indigo-600 hover:underline">Bewerk</button>
                                        <button data-product-id="${product.id}" class="delete-product-button text-sm text-red-500 hover:underline">Verwijder</button>
                                    </div>
                                </div>
                            `).join('') : '<p class="text-gray-500">Geen producten beschikbaar.</p>'}
                        </div>
                        <form id="add-product-form" class="space-y-2">
                            <input type="text" id="new-product-name" placeholder="Nieuwe productnaam" class="w-full p-2 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" />
                            <input type="number" id="new-product-price" placeholder="Prijs (bijv. 0.75)" step="0.01" class="w-full p-2 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" />
                            <input type="number" id="new-product-stock" placeholder="Voorraad" class="w-full p-2 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500" />
                            <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold p-2 rounded-lg transition-colors">Product toevoegen</button>
                        </form>
                    </div>

                    <!-- Sectie: Account aanmaken -->
                    <div class="mb-6 text-left">
                        <h3 class="text-xl font-semibold mb-2 text-gray-700">Account aanmaken</h3>
                        <form id="create-account-form">
                            <div class="mb-2">
                                <input
                                    type="text"
                                    id="new-account-name"
                                    placeholder="Accountnaam"
                                    class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                                />
                            </div>
                            <div class="mb-2">
                                <select id="new-account-type" class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                    <option value="vaste">Vast account</option>
                                    <option value="gast">Gast account</option>
                                </select>
                            </div>
                            <div class="mb-2">
                                <input
                                    type="password"
                                    id="new-account-pin"
                                    placeholder="PIN-code (max 4 cijfers)"
                                    maxlength="4"
                                    class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                                />
                            </div>
                            <button
                                type="submit"
                                class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold p-3 rounded-lg transition-colors"
                            >
                                Nieuw account aanmaken
                            </button>
                        </form>
                    </div>

                    <!-- Sectie: Overzicht van accounts -->
                    <div class="mb-6 text-left">
                        <h3 class="text-xl font-semibold mb-2 text-gray-700">Alle accounts</h3>
                        <div id="account-list-admin" class="bg-gray-100 p-4 rounded-lg max-h-48 overflow-y-auto">
                            ${accounts.length > 0 ? accounts.map(acc => `
                                <div class="flex justify-between items-center py-2 border-b last:border-b-0 border-gray-200">
                                    <div class="flex items-center gap-2">
                                        <span class="font-medium ${getBalanceColorClass(acc.balance, acc.type).split(' ').pop()}">${acc.name}</span>
                                        <span class="text-sm text-gray-500">(${acc.type})</span>
                                    </div>
                                    <div class="flex items-center gap-4">
                                        <span class="text-gray-500">€${acc.balance.toFixed(2)}</span>
                                        <button data-account-id="${acc.id}" class="edit-account-button text-sm text-indigo-600 hover:underline">Bewerk</button>
                                    </div>
                                </div>
                            `).join('') : '<p class="text-gray-500">Geen accounts beschikbaar.</p>'}
                        </div>
                    </div>
                    
                    <!-- Sectie: Transactielogboek -->
                    <div class="mb-6 text-left">
                        <h3 class="text-xl font-semibold mb-2 text-gray-700">Transactielogboek</h3>
                        <div id="transaction-log-admin" class="bg-gray-100 p-4 rounded-lg max-h-48 overflow-y-auto">
                            ${transactions.length > 0 ? transactions.map(t => `
                                <div class="py-2 border-b last:border-b-0 border-gray-200 text-sm text-gray-700 text-left">
                                    <p class="font-medium">${t.user}: €${t.amount.toFixed(2)} - ${t.items}</p>
                                    <p class="text-xs text-gray-500">${new Date(t.date).toLocaleString('nl-NL')}</p>
                                </div>
                            `).join('') : '<p class="text-gray-500">Geen transacties beschikbaar.</p>'}
                        </div>
                    </div>

                    <button
                        id="logout-button"
                        class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg transition-colors mt-4"
                    >
                        Uitloggen
                    </button>
                `;
                
                // Event listeners voor het beheerderspaneel
                document.getElementById('admin-user-select').addEventListener('change', (e) => {
                    selectedAdminUser = accounts.find(acc => acc.id === parseInt(e.target.value));
                });

                document.getElementById('add-money-button-single').addEventListener('click', () => {
                    const amountToAdd = parseFloat(document.getElementById('amount-input-single').value);
                    if (!selectedAdminUser || isNaN(amountToAdd) || amountToAdd <= 0) {
                        showMessage('Selecteer een gebruiker en voer een geldig bedrag in.', true);
                        return;
                    }
                    accounts = accounts.map(acc =>
                        acc.id === selectedAdminUser.id
                            ? { ...acc, balance: acc.balance + amountToAdd }
                            : acc
                    );
                    saveAccounts();
                    renderAdminView(); // Render de pagina opnieuw om het saldo bij te werken
                    showMessage(`€${amountToAdd.toFixed(2)} is toegevoegd aan het saldo van ${selectedAdminUser.name}.`);
                });

                document.getElementById('create-account-form').addEventListener('submit', (e) => {
                    e.preventDefault();
                    const newName = document.getElementById('new-account-name').value;
                    const newPin = document.getElementById('new-account-pin').value;
                    const newType = document.getElementById('new-account-type').value;

                    if (!newName || !newPin) {
                        showMessage('Naam, type en PIN zijn verplicht.', true);
                        return;
                    }
                    // Controleer of de PIN al bestaat
                    if (accounts.some(acc => acc.pin === newPin)) {
                        showMessage('Deze PIN is al in gebruik. Kies een andere.', true);
                        return;
                    }
                    const newAccount = {
                        id: accounts.length > 0 ? Math.max(...accounts.map(acc => acc.id)) + 1 : 1,
                        name: newName,
                        pin: newPin,
                        balance: 0.00,
                        type: newType
                    };
                    accounts.push(newAccount);
                    saveAccounts();
                    renderAdminView();
                    showMessage(`Account '${newName}' succesvol aangemaakt.`);
                });

                // Event listeners voor productbeheer
                document.getElementById('add-product-form').addEventListener('submit', (e) => {
                    e.preventDefault();
                    const newName = document.getElementById('new-product-name').value;
                    const newPrice = parseFloat(document.getElementById('new-product-price').value);
                    const newStock = parseInt(document.getElementById('new-product-stock').value, 10);
                    if (!newName || isNaN(newPrice) || newPrice <= 0 || isNaN(newStock) || newStock <= 0) {
                        showMessage('Ongeldige naam, prijs of voorraad. Probeer het opnieuw.', true);
                        return;
                    }
                    const newProduct = {
                        id: products.length > 0 ? Math.max(...products.map(p => p.id)) + 1 : 1,
                        name: newName,
                        price: newPrice,
                        stock: newStock
                    };
                    products.push(newProduct);
                    saveProducts();
                    renderAdminView();
                    showMessage(`Product '${newName}' is toegevoegd.`);
                });
                
                document.getElementById('product-list-admin').addEventListener('click', (e) => {
                    if (e.target.classList.contains('edit-product-button')) {
                        const productId = parseInt(e.target.dataset.productId);
                        const product = products.find(p => p.id === productId);
                        showEditModal({
                            title: `Bewerk product: ${product.name}`,
                            fields: [
                                { id: 'edit-name', label: 'Naam', value: product.name, type: 'text' },
                                { id: 'edit-price', label: 'Prijs', value: product.price.toFixed(2), type: 'number', step: '0.01' },
                                { id: 'edit-stock', label: 'Voorraad', value: product.stock, type: 'number' }
                            ],
                            onSave: (formData) => {
                                const newName = formData['edit-name'];
                                const newPrice = parseFloat(formData['edit-price']);
                                const newStock = parseInt(formData['edit-stock'], 10);
                                if (!newName || isNaN(newPrice) || newPrice <= 0 || isNaN(newStock) || newStock < 0) {
                                    showMessage('Ongeldige invoer. Product niet bijgewerkt.', true);
                                    return;
                                }
                                products = products.map(p => p.id === productId ? { ...p, name: newName, price: newPrice, stock: newStock } : p);
                                saveProducts();
                                renderAdminView();
                                showMessage(`Product '${newName}' is bijgewerkt.`);
                            }
                        });
                    } else if (e.target.classList.contains('delete-product-button')) {
                        const productId = parseInt(e.target.dataset.productId);
                        const product = products.find(p => p.id === productId);
                        showConfirmModal(`Product '${product.name}' verwijderen`, `Weet je zeker dat je het product '${product.name}' wilt verwijderen?`, () => {
                            products = products.filter(p => p.id !== productId);
                            saveProducts();
                            renderAdminView();
                            showMessage('Product succesvol verwijderd.');
                        });
                    }
                });

                document.getElementById('account-list-admin').addEventListener('click', (e) => {
                    if (e.target.classList.contains('edit-account-button')) {
                        const accountId = parseInt(e.target.dataset.accountId);
                        const account = accounts.find(a => a.id === accountId);
                        showEditModal({
                            title: `Bewerk account: ${account.name}`,
                            fields: [
                                { id: 'edit-name', label: 'Naam', value: account.name, type: 'text' },
                                { id: 'edit-pin', label: 'PIN', value: account.pin, type: 'password', maxlength: '4' },
                                { id: 'edit-type', label: 'Type', value: account.type, type: 'select', options: ['vaste', 'gast'] }
                            ],
                            onSave: (formData) => {
                                const newName = formData['edit-name'];
                                const newPin = formData['edit-pin'];
                                const newType = formData['edit-type'];
                                if (!newName || !newPin || newPin.length !== 4) {
                                    showMessage('Ongeldige invoer. Account niet bijgewerkt.', true);
                                    return;
                                }
                                if (accounts.some(acc => acc.pin === newPin && acc.id !== accountId)) {
                                    showMessage('Deze PIN is al in gebruik. Pincode niet aangepast.', true);
                                    return;
                                }
                                accounts = accounts.map(a => a.id === accountId ? { ...a, name: newName, pin: newPin, type: newType } : a);
                                saveAccounts();
                                renderAdminView();
                                showMessage(`Account van ${newName} is bijgewerkt.`);
                            }
                        });
                    }
                });

                document.getElementById('logout-button').addEventListener('click', handleLogout);
            };

            // Functie om het gebruikersdashboard te renderen
            const renderUserDashboard = () => {
                appContainer.innerHTML = `
                    <h1 class="text-3xl font-bold mb-6 text-gray-800">Fictief Wallet Systeem</h1>
                    <h2 class="text-2xl font-semibold mb-4 text-gray-700">Welkom, ${loggedInUser.name}!</h2>
                    <div class="text-2xl font-bold mb-6 text-indigo-600">
                        Saldo: €<span id="user-balance">${loggedInUser.balance.toFixed(2)}</span>
                    </div>

                    <div id="product-list" class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6 text-left">
                        ${products.map(product => `
                            <div class="flex items-center justify-between bg-gray-100 p-4 rounded-lg shadow-sm">
                                <div class="flex-grow">
                                    <h3 class="font-bold text-gray-700">${product.name}</h3>
                                    <p class="text-gray-500">€${product.price.toFixed(2)}</p>
                                    <p class="text-sm text-gray-500">Op voorraad: ${product.stock}</p>
                                </div>
                                <input
                                    type="number"
                                    data-product-id="${product.id}"
                                    value="${quantities[product.id] || 0}"
                                    min="0"
                                    max="${product.stock}"
                                    class="quantity-input w-16 p-2 rounded-lg border text-center"
                                />
                            </div>
                        `).join('')}
                    </div>

                    <div class="mb-4">
                        <p class="text-xl font-bold text-gray-800">
                            Totaal: €<span id="total-cost">0.00</span>
                        </p>
                    </div>

                    <button
                        id="purchase-button"
                        class="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-4 rounded-lg transition-colors transform hover:scale-105"
                    >
                        Aankopen
                    </button>
                    <button
                        id="logout-button"
                        class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-lg transition-colors mt-4"
                    >
                        Terug
                    </button>
                `;

                // Reset de hoeveelheden en de totale kosten
                quantities = products.reduce((acc, product) => {
                    acc[product.id] = quantities[product.id] || 0;
                    return acc;
                }, {});

                // Functie om de totalen bij te werken
                const updateTotals = () => {
                    const totalCost = Object.keys(quantities).reduce((total, productId) => {
                        const product = products.find(p => p.id === parseInt(productId));
                        return total + (product ? product.price * quantities[productId] : 0);
                    }, 0);
                    document.getElementById('total-cost').textContent = totalCost.toFixed(2);
                };

                document.querySelectorAll('.quantity-input').forEach(input => {
                    input.addEventListener('change', (e) => {
                        const productId = parseInt(e.target.dataset.productId);
                        const value = Math.max(0, parseInt(e.target.value, 10) || 0);
                        quantities[productId] = value;
                        updateTotals();
                    });
                });
                
                document.getElementById('user-balance').classList = getBalanceColorClass(loggedInUser.balance, loggedInUser.type).split(' ').pop();
                
                document.getElementById('purchase-button').addEventListener('click', () => {
                    const totalCost = parseFloat(document.getElementById('total-cost').textContent);
                    const newBalance = loggedInUser.balance - totalCost;

                    let isPurchaseAllowed = false;
                    const creditLimit = loggedInUser.type === 'vaste' ? VASTE_ACCOUNT_LIMIT : GAST_ACCOUNT_LIMIT;

                    if (newBalance >= creditLimit) {
                        isPurchaseAllowed = true;
                    }

                    if (isPurchaseAllowed) {
                        // Update stock
                        Object.keys(quantities).forEach(productId => {
                            const quantity = quantities[productId];
                            if (quantity > 0) {
                                products = products.map(p => 
                                    p.id === parseInt(productId) ? { ...p, stock: p.stock - quantity } : p
                                );
                            }
                        });
                        saveProducts();

                        loggedInUser.balance = newBalance;
                        accounts = accounts.map(acc => acc.id === loggedInUser.id ? loggedInUser : acc);
                        saveAccounts();

                        // Log de transactie
                        const transactionItems = Object.keys(quantities).map(productId => {
                            const product = products.find(p => p.id === parseInt(productId));
                            return `${product.name} (${quantities[productId]}x)`;
                        }).filter(item => !item.includes('(0x)')).join(', ');

                        transactions.push({
                            id: transactions.length > 0 ? Math.max(...transactions.map(t => t.id)) + 1 : 1,
                            user: loggedInUser.name,
                            amount: totalCost,
                            items: transactionItems,
                            date: new Date().toISOString()
                        });
                        saveTransactions();

                        document.getElementById('user-balance').textContent = loggedInUser.balance.toFixed(2);
                        document.getElementById('user-balance').className = getBalanceColorClass(loggedInUser.balance, loggedInUser.type).split(' ').pop();
                        showMessage(`Aankoop van €${totalCost.toFixed(2)} succesvol! Uw nieuwe saldo is €${loggedInUser.balance.toFixed(2)}.`);
                        
                        // Reset de hoeveelheden op de UI
                        document.querySelectorAll('.quantity-input').forEach(input => input.value = 0);
                        quantities = products.reduce((acc, product) => {
                            acc[product.id] = 0;
                            return acc;
                        }, {});
                        updateTotals();
                    } else {
                        showMessage(`Onvoldoende saldo. Totale kosten: €${totalCost.toFixed(2)}, uw saldo: €${loggedInUser.balance.toFixed(2)}. Het minimaal toegestane saldo is €${creditLimit.toFixed(2)}.`, true);
                    }
                });

                document.getElementById('logout-button').addEventListener('click', handleLogout);
                updateTotals(); // Initieel de totalen updaten
            };

            // Functie voor het tonen van een bewerkingsmodal
            const showEditModal = ({ title, fields, onSave }) => {
                const modalContainer = document.getElementById('modal-container');
                const modalTitle = document.getElementById('modal-title');
                const modalBody = document.getElementById('modal-body');

                modalTitle.textContent = title;
                modalBody.innerHTML = `
                    <form id="edit-form" class="space-y-4">
                        ${fields.map(field => `
                            <div>
                                <label for="${field.id}" class="block text-sm font-medium text-gray-700">${field.label}</label>
                                ${field.type === 'select' ? `
                                    <select id="${field.id}" class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                                        ${field.options.map(option => `<option value="${option}" ${option === field.value ? 'selected' : ''}>${option}</option>`).join('')}
                                    </select>
                                ` : `
                                    <input
                                        type="${field.type}"
                                        id="${field.id}"
                                        value="${field.value}"
                                        ${field.maxlength ? `maxlength="${field.maxlength}"` : ''}
                                        ${field.step ? `step="${field.step}"` : ''}
                                        class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
                                    />
                                `}
                            </div>
                        `).join('')}
                        <div class="flex justify-end gap-4 mt-6">
                            <button type="button" id="modal-cancel-button" class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-lg transition-colors">Annuleren</button>
                            <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg transition-colors">Opslaan</button>
                        </div>
                    </form>
                `;
                
                const form = document.getElementById('edit-form');
                form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    const formData = Object.fromEntries(new FormData(e.target).entries());
                    onSave(formData);
                    hideEditModal();
                });
                document.getElementById('modal-cancel-button').addEventListener('click', hideEditModal);
                modalContainer.classList.remove('hidden');
                modalContainer.classList.add('flex');
            };

            // Functie voor het verbergen van de bewerkingsmodal
            const hideEditModal = () => {
                const modalContainer = document.getElementById('modal-container');
                modalContainer.classList.add('hidden');
                modalContainer.classList.remove('flex');
            };

            // Functie voor het tonen van een bevestigingsmodal
            const showConfirmModal = (title, message, onConfirm) => {
                const confirmModalContainer = document.getElementById('confirm-modal-container');
                document.getElementById('confirm-title').textContent = title;
                document.getElementById('confirm-message').textContent = message;

                const confirmOkButton = document.getElementById('confirm-ok');
                const confirmCancelButton = document.getElementById('confirm-cancel');
                
                confirmOkButton.onclick = () => {
                    onConfirm();
                    hideConfirmModal();
                };
                confirmCancelButton.onclick = hideConfirmModal;
                
                confirmModalContainer.classList.remove('hidden');
                confirmModalContainer.classList.add('flex');
            };

            // Functie voor het verbergen van de bevestigingsmodal
            const hideConfirmModal = () => {
                const confirmModalContainer = document.getElementById('confirm-modal-container');
                confirmModalContainer.classList.add('hidden');
                confirmModalContainer.classList.remove('flex');
            };

            renderHomeView(); // Start met het inlogscherm
        });
    </script>
</body>
</html>
