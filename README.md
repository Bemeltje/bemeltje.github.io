import { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, doc, getDoc, setDoc, updateDoc, onSnapshot, collection, query, where, addDoc, deleteDoc } from 'firebase/firestore';

// Iconen vervangen met inline SVG om de compileerfout op te lossen.
const IconPlus = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>;
const IconMinus = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="5" y1="12" x2="19" y2="12"></line></svg>;
const IconSignOut = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M15 3h4a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-4"></path><polyline points="10 17 15 12 10 7"></polyline><line x1="15" y1="12" x2="3" y2="12"></line></svg>;
const IconUnlock = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>;
const IconCheck = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>;
const IconExclamation = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>;
const IconTrash = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>;
const IconPen = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M17 3a2.828 2.828 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z"></path></svg>;
const IconMoney = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="12" y1="1" x2="12" y2="23"></line><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"></path></svg>;
const IconDownload = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>;
const IconUpload = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>;
const IconReact = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="2"></circle><path d="M16.24 7.76l-2.12 2.12M7.76 16.24l2.12-2.12M7.76 7.76l2.12 2.12M16.24 16.24l-2.12-2.12"></path><path d="M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20z"></path></svg>;
const IconTailwind = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2c-3.14 0-5.91 1.77-7.39 4.39A8.99 8.99 0 0 0 3 13.91C3 18.3 6.69 22 11.08 22c3.41 0 6.32-2.14 7.64-5.11.96-2.11.91-4.7-.22-6.84-1.3-2.48-3.9-4.14-6.88-4.14zM12 4.09c1.99 0 3.73 1.05 4.67 2.62.77 1.25.76 2.82.02 4.05-1.12 2.01-3.26 3.34-5.69 3.34-2.43 0-4.57-1.33-5.69-3.34-.74-1.23-.75-2.8-.02-4.05.94-1.57 2.68-2.62 4.67-2.62zM7.7 13.52c-.62-.97-.56-2.19.14-3.1c1.24-1.63 3.36-2.67 5.75-2.67 2.39 0 4.51 1.04 5.75 2.67.7 1.2.76 2.42.14 3.1-.62-.97-2.31-1.64-4.22-1.64-1.91 0-3.6 0.67-4.22 1.64z"/></svg>;
const IconFirebase = () => <svg xmlns="http://www.w3.org/2000/svg" className="w-4 h-4" viewBox="0 0 24 24" fill="currentColor"><path d="M11.972 5.011C11.972 5.011 11.972 5.011 11.972 5.011L3.924 18.066C3.764 18.341 3.864 18.675 4.144 18.825C4.269 18.895 4.414 18.924 4.558 18.924C4.698 18.924 4.839 18.89 4.965 18.82L13.111 5.378C13.255 5.103 13.155 4.768 12.875 4.618C12.749 4.549 12.604 4.52 12.464 4.52C12.324 4.52 12.183 4.554 12.057 4.624L3.911 18.066C3.764 18.341 3.864 18.675 4.144 18.825C4.269 18.895 4.414 18.924 4.558 18.924C4.698 18.924 4.839 18.89 4.965 18.82L13.111 5.378C13.255 5.103 13.155 4.768 12.875 4.618C12.749 4.549 12.604 4.52 12.464 4.52C12.324 4.52 12.183 4.554 12.057 4.624zM11.972 5.011L11.972 5.011zM14.615 13.064C14.615 13.064 14.615 13.064 14.615 13.064L6.469 22.032C6.319 22.307 6.419 22.642 6.699 22.791C6.825 22.86 6.969 22.889 7.114 22.889C7.254 22.889 7.394 22.855 7.52 22.785L15.666 9.343C15.811 9.068 15.711 8.733 15.431 8.583C15.305 8.513 15.16 8.484 15.02 8.484C14.88 8.484 14.74 8.518 14.615 8.588L6.469 22.032C6.319 22.307 6.419 22.642 6.699 22.791C6.825 22.86 6.969 22.889 7.114 22.889C7.254 22.889 7.394 22.855 7.52 22.785zM22.072 13.627C22.072 13.627 22.072 13.627 22.072 13.627L13.926 22.595C13.776 22.87 13.876 23.205 14.156 23.354C14.281 23.424 14.426 23.453 14.57 23.453C14.71 23.453 14.851 23.419 14.977 23.349L23.123 9.907C23.267 9.632 23.167 9.297 22.887 9.147C22.761 9.077 22.616 9.048 22.476 9.048C22.336 9.048 22.195 9.082 22.069 9.152L13.926 22.595C13.776 22.87 13.876 23.205 14.156 23.354C14.281 23.424 14.426 23.453 14.57 23.453C14.71 23.453 14.851 23.419 14.977 23.349z"/></svg>;


// Constants voor de applicatie
const APP_VERSION = "2025-08-15-react-firestore-v1-bugfix";
const MAX_USERS = 50;
const ADMIN_IDLE_TIMEOUT_MS = 5 * 60 * 1000;
const ADMIN_LOCK_MAX_FAILS = 5;
const ADMIN_LOCK_DURATION_MS = 2 * 60 * 1000;

// Genereer een veilige SHA-256 hash van een string
const sha256Hex = async (str) => {
  const enc = new TextEncoder().encode(str);
  const buf = await crypto.subtle.digest('SHA-256', enc);
  const bytes = new Uint8Array(buf);
  return Array.from(bytes).map(b => b.toString(16).padStart(2, '0')).join('');
};

// Functie om de Firebase en Firestore instanties te initialiseren en te returnen
const getFirebaseInstances = () => {
  try {
    const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
    if (!firebaseConfig || Object.keys(firebaseConfig).length === 0) {
      console.error("Firebase config is niet beschikbaar.");
      return null;
    }
    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);
    return { app, auth, db };
  } catch (e) {
    console.error("Fout bij het initialiseren van Firebase:", e);
    return null;
  }
};

const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// Hoofdcomponent van de applicatie
const App = () => {
  const [screen, setScreen] = useState('home');
  const [users, setUsers] = useState([]);
  const [products, setProducts] = useState([]);
  const [logs, setLogs] = useState([]);
  const [cart, setCart] = useState({});
  const [currentUser, setCurrentUser] = useState(null);
  const [currentAdmin, setCurrentAdmin] = useState(null);
  const [isAuthReady, setIsAuthReady] = useState(false);
  const [db, setDb] = useState(null);
  const [auth, setAuth] = useState(null);
  const [lastAdminActionAt, setLastAdminActionAt] = useState(Date.now());
  const [adminLockState, setAdminLockState] = useState({});
  const [adminPin, setAdminPin] = useState('');

  // Initialisatie van Firebase en authenticatie bij het laden van de app
  useEffect(() => {
    const firebaseInstances = getFirebaseInstances();
    if (firebaseInstances) {
      const { auth, db } = firebaseInstances;
      setAuth(auth);
      setDb(db);
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          console.log("Anoniem aanmelden...");
          const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
          try {
            if (token) {
              await signInWithCustomToken(auth, token);
            } else {
              await signInAnonymously(auth);
            }
          } catch (error) {
            console.error("Fout bij aanmelden:", error);
          }
        }
        setIsAuthReady(true);
      });
    }
  }, []);

  // Real-time data ophalen van Firestore met onSnapshot
  useEffect(() => {
    if (!db || !isAuthReady) return;

    // Accounts
    const usersRef = collection(db, `artifacts/${appId}/public/data/users`);
    const usersUnsub = onSnapshot(usersRef, (snapshot) => {
      const fetchedUsers = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setUsers(fetchedUsers);
    }, (error) => console.error("Fout bij ophalen accounts:", error));

    // Producten
    const productsRef = collection(db, `artifacts/${appId}/public/data/products`);
    const productsUnsub = onSnapshot(productsRef, (snapshot) => {
      const fetchedProducts = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setProducts(fetchedProducts);
    }, (error) => console.error("Fout bij ophalen producten:", error));

    // Logs
    const logsRef = collection(db, `artifacts/${appId}/public/data/logs`);
    const logsUnsub = onSnapshot(logsRef, (snapshot) => {
      const fetchedLogs = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setLogs(fetchedLogs.sort((a, b) => b.timestamp.toMillis() - a.timestamp.toMillis()));
    }, (error) => console.error("Fout bij ophalen logs:", error));

    // Admin Lock
    const lockDocRef = doc(db, `artifacts/${appId}/public/data/adminLock/state`);
    const lockUnsub = onSnapshot(lockDocRef, (snapshot) => {
      setAdminLockState(snapshot.data() || { fails: 0, until: 0 });
    }, (error) => console.error("Fout bij ophalen admin lock:", error));

    return () => {
      usersUnsub();
      productsUnsub();
      logsUnsub();
      lockUnsub();
    };
  }, [db, isAuthReady]);

  // Automatische uitloggen admin na inactiviteit
  useEffect(() => {
    const checkIdle = () => {
      if (currentAdmin && Date.now() - lastAdminActionAt > ADMIN_IDLE_TIMEOUT_MS) {
        logAction('Admin auto-uitlog (inactiviteit)');
        setCurrentAdmin(null);
        setScreen('home');
      }
    };
    const interval = setInterval(checkIdle, 15000);
    return () => clearInterval(interval);
  }, [currentAdmin, lastAdminActionAt]);

  // Utility functies
  const formatPrice = (n) => Number(n).toFixed(2);
  const isOnlyAdmin = (userId) => users.filter(u => u.role === 'admin' && u.id !== userId).length === 0;
  const isLastAdmin = (userId) => users.filter(u => u.role === 'admin').length === 1 && users.find(u => u.id === userId);
  const isAdmin = () => currentAdmin?.role === 'admin';
  const isCoAdmin = () => currentAdmin?.role === 'coadmin' || currentAdmin?.role === 'admin';
  const touchAdminActivity = () => setLastAdminActionAt(Date.now());
  const logAction = async (text, bedrag = 0) => {
    if (!db) return;
    const logsRef = collection(db, `artifacts/${appId}/public/data/logs`);
    const actor = currentAdmin ? users.find(u => u.id === currentAdmin.id)?.name || 'ONBEKEND' : 'SYSTEEM';
    await addDoc(logsRef, {
      gebruiker: actor,
      product: `ACTIE: ${text}`,
      prijs: bedrag,
      timestamp: new Date()
    });
  };

  // Navigatie
  const goHome = () => {
    setScreen('home');
    setCurrentUser(null);
    setCurrentAdmin(null);
    setAdminPin('');
  };

  // Functies voor schermlogica
  const selectUser = (user) => {
    setCurrentUser(user);
    setScreen('pin');
  };

  const loginUser = async (pin) => {
    const inputHash = await sha256Hex(pin);
    if (currentUser.pinHash === inputHash) {
      setScreen('user');
      const initialCart = {};
      products.forEach(p => initialCart[p.id] = 0);
      setCart(initialCart);
    } else {
      alert('Verkeerde pincode!');
    }
  };

  const loginAdmin = async () => {
    const nowTs = Date.now();
    if (adminLockState.until > nowTs) {
      const sec = Math.ceil((adminLockState.until - nowTs) / 1000);
      alert(`Te veel mislukte pogingen. Probeer over ${sec}s opnieuw.`);
      return;
    }
    const selectedAdmin = users.find(u => u.id === adminPin.split(';')[0]);
    const pin = adminPin.split(';')[1];
    if (!selectedAdmin || !pin) {
      alert('Kies een beheerder en voer een pincode in.');
      return;
    }
    const inputHash = await sha256Hex(pin);
    if (selectedAdmin.pinHash === inputHash) {
      await setDoc(doc(db, `artifacts/${appId}/public/data/adminLock/state`), { fails: 0, until: 0 });
      setCurrentAdmin(selectedAdmin);
      setScreen('admin');
      logAction(`Beheerlogin als ${selectedAdmin.role} (${selectedAdmin.name})`);
    } else {
      alert('Verkeerde pincode!');
      const nextFails = (adminLockState.fails || 0) + 1;
      await setDoc(doc(db, `artifacts/${appId}/public/data/adminLock/state`), {
        fails: nextFails,
        until: nextFails >= ADMIN_LOCK_MAX_FAILS ? nowTs + ADMIN_LOCK_DURATION_MS : 0
      });
      logAction(`MISLUKTE beheerlogin voor ${selectedAdmin.name}`);
    }
  };

  // Functies voor winkelwagen
  const updateCart = (productId, quantity) => {
    setCart(prev => ({ ...prev, [productId]: quantity }));
  };

  const clearCart = () => {
    const emptyCart = {};
    products.forEach(p => emptyCart[p.id] = 0);
    setCart(emptyCart);
  };

  const checkoutCart = async () => {
    if (!currentUser) return;
    const total = Object.keys(cart).reduce((sum, id) => sum + (cart[id] || 0) * (products.find(p => p.id === id)?.price || 0), 0);
    const items = Object.values(cart).reduce((sum, qty) => sum + qty, 0);

    const userRef = doc(db, `artifacts/${appId}/public/data/users`, currentUser.id);

    if (currentUser.type === 'gast' && currentUser.saldo - total < 0) {
      alert('Gast mag niet onder €0 komen!');
      return;
    }
    if (currentUser.type === 'vast' && currentUser.saldo - total < -10) {
      alert('Vast mag niet verder dan -€10 komen!');
      return;
    }

    const confirmed = window.confirm(`Bevestig aankoop van €${formatPrice(total)} (${items} item(s)). Doorgaan?`);
    if (!confirmed) return;

    try {
      // Begin transactie
      await updateDoc(userRef, { saldo: currentUser.saldo - total });

      for (const prodId in cart) {
        const qty = cart[prodId];
        if (qty > 0) {
          const productRef = doc(db, `artifacts/${appId}/public/data/products`, prodId);
          const product = products.find(p => p.id === prodId);
          if (product) {
            await updateDoc(productRef, { stock: Math.max(0, product.stock - qty) });
            await logAction(`Aankoop: ${product.name} (x${qty}) door ${currentUser.name}`, product.price * qty);
          }
        }
      }
      clearCart();
      alert('Aankoop voltooid!');
      goHome();
    } catch (e) {
      console.error("Transactie mislukt:", e);
      alert('Aankoop mislukt, probeer opnieuw.');
    }
  };

  // Admin functies
  const handleAdminAction = async (action, ...args) => {
    if (!isCoAdmin()) {
      alert('Onvoldoende rechten.');
      return;
    }
    touchAdminActivity();
    try {
      if (action === 'addAccount') {
        await addAccount(...args);
      } else if (action === 'deleteAccount') {
        await deleteAccount(...args);
      } else if (action === 'addSaldo') {
        await addSaldo(...args);
      } else if (action === 'changePin') {
        await changePin(...args);
      } else if (action === 'changeRole') {
        await changeRole(...args);
      } else if (action === 'renameAccount') {
        await renameAccount(...args);
      } else if (action === 'addProduct') {
        await addProduct(...args);
      } else if (action === 'deleteProduct') {
        await deleteProduct(...args);
      } else if (action === 'restockProduct') {
        await restockProduct(...args);
      } else if (action === 'changePrice') {
        await changePrice(...args);
      } else if (action === 'clearLogs') {
        await clearLogs();
      } else if (action === 'resetData') {
        await resetData();
      }
    } catch (e) {
      console.error(`Fout bij actie ${action}:`, e);
      alert(`Actie mislukt: ${e.message}`);
    }
  };

  const addAccount = async (name, pin, startSaldo, isGuest, role) => {
    if (users.length >= MAX_USERS) throw new Error(`Maximum aantal accounts (${MAX_USERS}) bereikt.`);
    if (!name || !pin || isNaN(startSaldo) || !/^\d{4}$/.test(pin)) throw new Error('Vul alle velden correct in.');

    const pinHash = await sha256Hex(pin);
    const newDocRef = doc(collection(db, `artifacts/${appId}/public/data/users`));
    await setDoc(newDocRef, {
      name,
      pinHash,
      saldo: Number(startSaldo),
      type: isGuest ? 'gast' : 'vast',
      role: isCoAdmin() ? role : 'user'
    });
    logAction(`Account aangemaakt: ${name} (rol: ${role}, type: ${isGuest ? 'gast' : 'vast'})`);
  };

  const deleteAccount = async (id) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    const userToDelete = users.find(u => u.id === id);
    if (!userToDelete) return;
    if (isLastAdmin(id)) {
      throw new Error('Kan de laatste admin niet verwijderen. Wijs eerst een andere admin aan.');
    }
    const confirmed = window.confirm(`Account "${userToDelete.name}" verwijderen?`);
    if (confirmed) {
      await deleteDoc(doc(db, `artifacts/${appId}/public/data/users`, id));
      logAction(`Account verwijderd: ${userToDelete.name}`);
    }
  };

  const addSaldo = async (id, amount) => {
    if (isNaN(amount) || amount <= 0) throw new Error('Voer een positief getal in.');
    const user = users.find(u => u.id === id);
    if (!user) return;
    const userRef = doc(db, `artifacts/${appId}/public/data/users`, id);
    await updateDoc(userRef, { saldo: user.saldo + Number(amount) });
    logAction(`Saldo +€${formatPrice(amount)} voor ${user.name}`, amount);
  };

  const changePin = async (id, newPin) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    if (!/^\d{4}$/.test(newPin)) throw new Error('Pincode moet precies 4 cijfers zijn.');
    const pinHash = await sha256Hex(newPin);
    const userRef = doc(db, `artifacts/${appId}/public/data/users`, id);
    await updateDoc(userRef, { pinHash });
    const user = users.find(u => u.id === id);
    logAction(`PIN gewijzigd voor ${user.name}`);
  };

  const changeRole = async (id, newRole) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    const user = users.find(u => u.id === id);
    if (!user) return;
    if (isLastAdmin(id) && newRole !== 'admin') {
      throw new Error('Je kunt de laatste admin niet degraderen. Wijs eerst iemand anders als admin aan.');
    }
    const userRef = doc(db, `artifacts/${appId}/public/data/users`, id);
    await updateDoc(userRef, { role: newRole });
    logAction(`Rol gewijzigd: ${user.name} -> ${newRole}`);
  };

  const renameAccount = async (id, newName) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    if (!newName.trim()) throw new Error('Naam mag niet leeg zijn.');
    const user = users.find(u => u.id === id);
    if (!user) return;
    const userRef = doc(db, `artifacts/${appId}/public/data/users`, id);
    await updateDoc(userRef, { name: newName.trim() });
    logAction(`Naam gewijzigd: ${user.name} -> ${newName}`);
  };

  const addProduct = async (name, price, stock) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    if (!name || isNaN(price) || price < 0 || isNaN(stock) || stock < 0) throw new Error('Vul alle velden correct in.');
    const newDocRef = doc(collection(db, `artifacts/${appId}/public/data/products`));
    await setDoc(newDocRef, {
      name,
      price: Number(price),
      stock: Number(stock)
    });
    logAction(`Product toegevoegd: ${name} (€${price})`);
  };

  const deleteProduct = async (id) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    const product = products.find(p => p.id === id);
    if (!product) return;
    const confirmed = window.confirm(`Product "${product.name}" verwijderen?`);
    if (confirmed) {
      await deleteDoc(doc(db, `artifacts/${appId}/public/data/products`, id));
      logAction(`Product verwijderd: ${product.name}`);
    }
  };

  const restockProduct = async (id, amount) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    if (isNaN(amount) || amount <= 0) throw new Error('Voer een positief geheel getal in.');
    const product = products.find(p => p.id === id);
    if (!product) return;
    const productRef = doc(db, `artifacts/${appId}/public/data/products`, id);
    await updateDoc(productRef, { stock: Math.max(0, product.stock + Number(amount)) });
    logAction(`Voorraad +${amount} voor ${product.name}`);
  };

  const changePrice = async (id, newPrice) => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    if (isNaN(newPrice) || newPrice < 0) throw new Error('Prijs moet ≥ 0 zijn.');
    const product = products.find(p => p.id === id);
    if (!product) return;
    const productRef = doc(db, `artifacts/${appId}/public/data/products`, id);
    await updateDoc(productRef, { price: Number(newPrice) });
    logAction(`Prijs gewijzigd: ${product.name} €${formatPrice(product.price)} → €${formatPrice(newPrice)}`);
  };

  const clearLogs = async () => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    const confirmed = window.confirm('Weet je zeker dat je het logboek wilt wissen? Dit kan niet ongedaan worden gemaakt.');
    if (confirmed) {
      const logsRef = collection(db, `artifacts/${appId}/public/data/logs`);
      const q = query(logsRef);
      const snapshot = await getDocs(q);
      const batch = db.batch();
      snapshot.docs.forEach(d => batch.delete(d.ref));
      await batch.commit();
      logAction('Logboek gewist');
    }
  };

  const resetData = async () => {
    if (!isAdmin()) throw new Error('Onvoldoende rechten.');
    const confirmed = window.confirm('Weet je zeker dat je alle data wilt herstellen naar de standaardwaarden? Dit kan niet ongedaan worden gemaakt.');
    if (confirmed) {
      // Gebruik Promise.all voor parallelle operaties
      await Promise.all([
        clearCollection('users'),
        clearCollection('products'),
        clearCollection('logs')
      ]);

      const initialUsers = [
        { name: "Jan", pinHash: await sha256Hex("1234"), saldo: 40.00, type: "vast", role: "user" },
        { name: "Piet", pinHash: await sha256Hex("5678"), saldo: 5.00, type: "gast", role: "user" },
        { name: "Beheer", pinHash: await sha256Hex("9999"), saldo: 0.00, type: "vast", role: "admin" }
      ];
      const initialProducts = [
        { name: "Chips", price: 0.75, stock: 20 },
        { name: "Bier", price: 0.75, stock: 30 },
        { name: "Cola", price: 1.00, stock: 15 }
      ];
      const usersRef = collection(db, `artifacts/${appId}/public/data/users`);
      const productsRef = collection(db, `artifacts/${appId}/public/data/products`);
      for (const user of initialUsers) {
        await addDoc(usersRef, user);
      }
      for (const product of initialProducts) {
        await addDoc(productsRef, product);
      }
      logAction('Data reset naar standaardwaarden');
      alert('Alle data is hersteld naar de standaardwaarden.');
    }
  };

  const clearCollection = async (collectionName) => {
    const colRef = collection(db, `artifacts/${appId}/public/data/${collectionName}`);
    const q = query(colRef);
    const snapshot = await getDocs(q);
    const batch = db.batch();
    snapshot.docs.forEach(d => batch.delete(d.ref));
    await batch.commit();
  };

  // Functies voor data import/export
  const exportData = () => {
    if (!isAdmin()) {
      alert('Onvoldoende rechten.');
      return;
    }
    const data = {
      version: APP_VERSION,
      exportedAt: new Date().toISOString(),
      users: users.map(({ id, ...rest }) => rest), // Verwijder de id
      products: products.map(({ id, ...rest }) => rest),
      logs: logs.map(l => ({ ...l, timestamp: l.timestamp.toDate() })) // Converteer Firestore timestamp
    };
    const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'fictief-geld-data.json';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    logAction('Data geëxporteerd (JSON)');
  };

  const importData = async (event) => {
    if (!isAdmin()) {
      alert('Onvoldoende rechten.');
      return;
    }
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = async (e) => {
      try {
        const data = JSON.parse(e.target.result);
        if (!data.users || !data.products || !data.logs) {
          throw new Error('Onjuist JSON-formaat.');
        }
        const confirmed = window.confirm('Huidige data overschrijven? Dit kan niet ongedaan worden gemaakt.');
        if (!confirmed) return;

        await clearCollection('users');
        await clearCollection('products');
        await clearCollection('logs');

        const usersRef = collection(db, `artifacts/${appId}/public/data/users`);
        const productsRef = collection(db, `artifacts/${appId}/public/data/products`);
        const logsRef = collection(db, `artifacts/${appId}/public/data/logs`);

        // Importeer data
        for (const user of data.users) await addDoc(usersRef, user);
        for (const product of data.products) await addDoc(productsRef, product);
        for (const log of data.logs) await addDoc(logsRef, { ...log, timestamp: new Date(log.timestamp) });

        logAction('Data geïmporteerd (JSON)');
        alert('Data geïmporteerd.');
      } catch (err) {
        console.error("Importfout:", err);
        alert('Fout bij importeren van data: ' + err.message);
      }
    };
    reader.readAsText(file);
  };

  const getCardColor = (user) => {
    if (user.type === 'gast') {
      return user.saldo >= 0 ? 'border-green-500' : 'border-red-500';
    }
    if (user.saldo >= 0) return 'border-green-500';
    if (user.saldo >= -10) return 'border-orange-500';
    return 'border-red-500';
  };

  // Render logica
  const renderHome = () => (
    <div className="container max-w-7xl mx-auto p-4 md:p-8 bg-white rounded-xl shadow-lg mt-8">
      <h2 className="text-2xl font-semibold text-gray-800 mb-6">Kies je account</h2>
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 mb-8">
        {users.map((user) => (
          <button
            key={user.id}
            onClick={() => selectUser(user)}
            className={`flex flex-col items-start p-4 bg-gray-50 rounded-lg shadow-md hover:shadow-lg transition-all transform hover:-translate-y-1 ${getCardColor(user)} border-l-4`}
          >
            <strong className="text-lg font-bold text-gray-900">{user.name}</strong>
            <span className="text-sm text-gray-600">
              Saldo: €{formatPrice(user.saldo)}
              {user.role !== 'user' && <span className={`ml-2 px-2 py-1 text-xs font-semibold rounded-full ${user.role === 'admin' ? 'bg-red-500 text-white' : 'bg-green-500 text-white'}`}>{user.role}</span>}
              {user.type === 'gast' && <span className="ml-2 px-2 py-1 text-xs font-semibold rounded-full bg-yellow-500 text-white">gast</span>}
            </span>
          </button>
        ))}
      </div>
      <hr className="my-6 border-gray-200" />
      <div className="flex flex-col md:flex-row md:items-center justify-between p-4 bg-gray-100 rounded-lg">
        <span className="text-gray-700 text-lg font-semibold mb-2 md:mb-0">Beheerder inloggen</span>
        <div className="flex flex-col md:flex-row items-stretch md:items-center gap-2">
          <select
            className="flex-1 min-w-[150px] p-2 rounded-lg border border-gray-300 bg-white text-gray-800 focus:outline-none focus:ring-2 focus:ring-green-500"
            onChange={(e) => setAdminPin(e.target.value)}
          >
            <option value="">-- Kies een beheerder --</option>
            {users.filter(u => u.role === 'admin' || u.role === 'coadmin').map(u => (
              <option key={u.id} value={`${u.id};${u.pin}`}>{u.name} ({u.role})</option>
            ))}
          </select>
          <input
            type="password"
            className="flex-1 p-2 rounded-lg border border-gray-300 bg-white text-gray-800 focus:outline-none focus:ring-2 focus:ring-green-500"
            placeholder="Pincode (1234)"
            value={adminPin.split(';')[1] || ''}
            onChange={(e) => setAdminPin(`${adminPin.split(';')[0]};${e.target.value}`)}
            maxLength="4"
            inputMode="numeric"
            disabled={!adminPin.split(';')[0]}
          />
          <button
            onClick={loginAdmin}
            className="bg-green-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-green-700 transition disabled:opacity-50 disabled:cursor-not-allowed"
            disabled={!adminPin.split(';')[1] || adminPin.split(';')[1].length !== 4 || adminLockState.until > Date.now()}
          >
            Inloggen
          </button>
        </div>
      </div>
      <p className="text-sm text-gray-500 mt-2">
        Tip: Dit is een demo. De pincodes zijn in het echt 1234, 5678 en 9999.
      </p>
    </div>
  );

  const renderPinScreen = () => (
    <div className="container max-w-xl mx-auto p-8 bg-white rounded-xl shadow-lg mt-8">
      <h2 className="text-2xl font-semibold text-gray-800 mb-2">Inloggen</h2>
      <p className="text-lg text-gray-600 mb-6">Account: {currentUser?.name}</p>
      <input
        type="password"
        className="w-full p-3 mb-4 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-green-500 text-center text-2xl font-mono"
        placeholder="Pincode (4 cijfers)"
        maxLength="4"
        inputMode="numeric"
        onChange={(e) => {
          const val = e.target.value;
          if (val.length === 4) loginUser(val);
        }}
      />
      <div className="flex gap-4">
        <button
          onClick={() => goHome()}
          className="flex-1 py-3 px-6 rounded-lg bg-gray-200 text-gray-700 font-bold hover:bg-gray-300 transition"
        >
          Annuleren
        </button>
      </div>
    </div>
  );

  const renderUserScreen = () => {
    const total = Object.keys(cart).reduce((sum, id) => sum + (cart[id] || 0) * (products.find(p => p.id === id)?.price || 0), 0);
    const items = Object.values(cart).reduce((sum, qty) => sum + qty, 0);
    const saldoColor = currentUser?.saldo >= 0 ? 'text-green-600' : currentUser?.saldo >= -10 ? 'text-orange-600' : 'text-red-600';
    return (
      <div className="container max-w-7xl mx-auto p-4 md:p-8 bg-white rounded-xl shadow-lg mt-8">
        <h2 className="text-2xl font-semibold text-gray-800 mb-2">Welkom {currentUser?.name}</h2>
        <p className={`text-xl font-bold mb-6 ${saldoColor}`}>Saldo: €{formatPrice(currentUser?.saldo)}</p>
        <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
          {products.map((product) => (
            <div key={product.id} className="flex flex-col p-4 bg-gray-50 rounded-lg shadow-inner">
              <div className="flex justify-between items-center mb-2">
                <span className="text-lg font-medium">{product.name}</span>
                <span className="text-green-600 font-semibold">€{formatPrice(product.price)}</span>
              </div>
              <p className="text-sm text-gray-500 mb-4">Voorraad: <span className={product.stock <= 5 ? 'font-bold text-red-500' : 'text-gray-500'}>{product.stock}</span></p>
              <div className="flex items-center justify-between">
                <div className="flex items-center space-x-2">
                  <button
                    onClick={() => updateCart(product.id, Math.max(0, (cart[product.id] || 0) - 1))}
                    className="bg-red-500 text-white p-2 rounded-full hover:bg-red-600 transition disabled:opacity-50"
                    disabled={!cart[product.id]}
                  >
                    <IconMinus />
                  </button>
                  <span className="text-xl font-bold w-8 text-center">{cart[product.id] || 0}</span>
                  <button
                    onClick={() => updateCart(product.id, Math.min(product.stock, (cart[product.id] || 0) + 1))}
                    className="bg-green-500 text-white p-2 rounded-full hover:bg-green-600 transition disabled:opacity-50"
                    disabled={cart[product.id] >= product.stock}
                  >
                    <IconPlus />
                  </button>
                </div>
                <span className="text-gray-700 font-medium">Totaal: €{formatPrice((cart[product.id] || 0) * product.price)}</span>
              </div>
            </div>
          ))}
        </div>
        <div className="flex flex-col md:flex-row items-center justify-between p-4 bg-green-100 rounded-lg mt-8 shadow-inner">
          <span className="text-lg font-bold text-green-800 mb-2 md:mb-0">Totaal: €{formatPrice(total)} — {items} item(s)</span>
          <div className="flex items-center gap-4">
            <button
              onClick={checkoutCart}
              className="bg-green-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-green-700 transition disabled:opacity-50"
              disabled={items === 0}
            >
              <IconCheck /> Afrekenen
            </button>
            <button
              onClick={clearCart}
              className="bg-gray-200 text-gray-700 py-2 px-4 rounded-lg font-bold hover:bg-gray-300 transition"
            >
              <IconTrash /> Leegmaken
            </button>
          </div>
        </div>
        <hr className="my-6 border-gray-200" />
        <button
          onClick={goHome}
          className="bg-red-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-red-700 transition"
        >
          <IconSignOut /> Uitloggen
        </button>
      </div>
    );
  };

  const renderAdminScreen = () => {
    return (
      <div className="container max-w-7xl mx-auto p-4 md:p-8 bg-white rounded-xl shadow-lg mt-8" onClick={touchAdminActivity}>
        <h2 className="text-2xl font-semibold text-gray-800 mb-2">{currentAdmin?.role === 'admin' ? 'Admin Paneel' : 'Co-Admin Paneel'}</h2>
        <p className="text-sm text-gray-500 mb-6">Ingelogd als {currentAdmin?.name}</p>

        {/* Accounts Beheer */}
        <h3 className="text-xl font-semibold text-gray-800 mb-4">Accounts</h3>
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
          {users.map((user) => (
            <div key={user.id} className="flex flex-col p-4 bg-gray-50 rounded-lg shadow-inner">
              <div className="flex items-center justify-between mb-2">
                <span className="text-lg font-medium">{user.name}</span>
                <div className="flex items-center gap-2">
                  {user.role === 'admin' && <span className="px-2 py-1 text-xs font-semibold rounded-full bg-red-500 text-white">Admin</span>}
                  {user.role === 'coadmin' && <span className="px-2 py-1 text-xs font-semibold rounded-full bg-green-500 text-white">Co-Admin</span>}
                  {user.type === 'gast' && <span className="px-2 py-1 text-xs font-semibold rounded-full bg-yellow-500 text-white">Gast</span>}
                </div>
              </div>
              <span className="text-md text-gray-700 mb-4">Saldo: €{formatPrice(user.saldo)}</span>
              <div className="flex flex-wrap gap-2">
                <button onClick={() => handleAdminAction('addSaldo', user.id, prompt(`Bedrag toevoegen aan ${user.name}:`, "0"))} className="bg-green-500 text-white p-2 rounded-lg text-sm"><IconMoney /></button>
                {isAdmin() && <button onClick={() => handleAdminAction('changePin', user.id, prompt(`Nieuwe PIN voor ${user.name}:`))} className="bg-blue-500 text-white p-2 rounded-lg text-sm"><IconUnlock /></button>}
                {isAdmin() && <button onClick={() => handleAdminAction('changeRole', user.id, prompt(`Nieuwe rol voor ${user.name} (user, coadmin, admin):`, user.role))} className="bg-purple-500 text-white p-2 rounded-lg text-sm"><IconPen /></button>}
                {isAdmin() && <button onClick={() => handleAdminAction('deleteAccount', user.id)} className="bg-red-500 text-white p-2 rounded-lg text-sm" disabled={isLastAdmin(user.id)}><IconTrash /></button>}
              </div>
            </div>
          ))}
          <div className="flex flex-col p-4 bg-gray-200 rounded-lg shadow-inner">
            <h4 className="text-md font-bold mb-2">Nieuw account toevoegen</h4>
            <input id="newName" type="text" placeholder="Naam" className="p-2 rounded-lg mb-2" />
            <input id="newPin" type="text" placeholder="Pincode (4 cijfers)" className="p-2 rounded-lg mb-2" maxLength="4" inputMode="numeric" />
            <input id="newSaldo" type="number" placeholder="Startsaldo" className="p-2 rounded-lg mb-2" step="0.01" />
            <select id="newRole" className="p-2 rounded-lg mb-2">
              <option value="user">Gebruiker</option>
              <option value="coadmin">Co-admin</option>
              {isAdmin() && <option value="admin">Admin</option>}
            </select>
            <div className="flex items-center gap-2 mb-2">
              <input id="newIsGuest" type="checkbox" className="accent-green-500" />
              <label htmlFor="newIsGuest" className="text-sm">Gastaccount (saldo &gt; €0)</label>
            </div>
            <button
              onClick={() => handleAdminAction('addAccount', document.getElementById('newName').value, document.getElementById('newPin').value, document.getElementById('newSaldo').value, document.getElementById('newIsGuest').checked, document.getElementById('newRole').value)}
              className="bg-green-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-green-700 transition"
            >
              Toevoegen
            </button>
          </div>
        </div>

        {/* Producten Beheer */}
        <h3 className="text-xl font-semibold text-gray-800 mb-4">Producten</h3>
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
          {products.map((product) => (
            <div key={product.id} className="flex flex-col p-4 bg-gray-50 rounded-lg shadow-inner">
              <div className="flex items-center justify-between mb-2">
                <span className="text-lg font-medium">{product.name}</span>
                <span className="text-md font-semibold text-green-600">€{formatPrice(product.price)}</span>
              </div>
              <span className="text-sm text-gray-500 mb-4">Voorraad: <span className={product.stock <= 5 ? 'font-bold text-red-500' : 'text-gray-500'}>{product.stock}</span></span>
              <div className="flex flex-wrap gap-2">
                {isAdmin() && <button onClick={() => handleAdminAction('changePrice', product.id, prompt(`Nieuwe prijs voor ${product.name}:`))} className="bg-blue-500 text-white p-2 rounded-lg text-sm"><IconPen /></button>}
                {isAdmin() && <button onClick={() => handleAdminAction('restockProduct', product.id, prompt(`Aantal bijvullen voor ${product.name}:`))} className="bg-green-500 text-white p-2 rounded-lg text-sm"><IconPlus /></button>}
                {isAdmin() && <button onClick={() => handleAdminAction('deleteProduct', product.id)} className="bg-red-500 text-white p-2 rounded-lg text-sm"><IconTrash /></button>}
              </div>
            </div>
          ))}
          {isAdmin() && (
            <div className="flex flex-col p-4 bg-gray-200 rounded-lg shadow-inner">
              <h4 className="text-md font-bold mb-2">Nieuw product toevoegen</h4>
              <input id="prodName" type="text" placeholder="Productnaam" className="p-2 rounded-lg mb-2" />
              <input id="prodPrice" type="number" placeholder="Prijs" className="p-2 rounded-lg mb-2" step="0.01" />
              <input id="prodStock" type="number" placeholder="Voorraad" className="p-2 rounded-lg mb-2" />
              <button
                onClick={() => handleAdminAction('addProduct', document.getElementById('prodName').value, document.getElementById('prodPrice').value, document.getElementById('prodStock').value)}
                className="bg-green-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-green-700 transition"
              >
                Toevoegen
              </button>
            </div>
          )}
        </div>

        {/* Logboek */}
        <h3 className="text-xl font-semibold text-gray-800 mb-4">Logboek</h3>
        <div className="overflow-x-auto mb-6 bg-gray-50 rounded-lg shadow-inner">
          <table className="w-full text-sm text-left text-gray-500 rounded-lg">
            <thead className="text-xs text-gray-700 uppercase bg-gray-200">
              <tr>
                <th scope="col" className="px-4 py-2">Gebruiker</th>
                <th scope="col" className="px-4 py-2">Product/Actie</th>
                <th scope="col" className="px-4 py-2">Prijs/Bedrag</th>
                <th scope="col" className="px-4 py-2">Tijd</th>
              </tr>
            </thead>
            <tbody>
              {logs.map((log) => (
                <tr key={log.id} className="bg-white border-b hover:bg-gray-100">
                  <td className="px-4 py-2">{log.gebruiker}</td>
                  <td className="px-4 py-2">{log.product}</td>
                  <td className="px-4 py-2">€{formatPrice(log.prijs)}</td>
                  <td className="px-4 py-2">{log.timestamp?.toDate().toLocaleString()}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
        <div className="flex flex-wrap items-center justify-between gap-4 p-4 bg-gray-100 rounded-lg">
          <span className="text-sm text-gray-600">Logboek acties</span>
          <div className="flex flex-wrap gap-2">
            <button onClick={() => handleAdminAction('exportCsv')} className="bg-blue-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-blue-700 transition"><IconDownload /> CSV Exporteren</button>
            {isAdmin() && <button onClick={() => handleAdminAction('clearLogs')} className="bg-red-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-red-700 transition"><IconTrash /> Logboek Wissen</button>}
          </div>
        </div>

        {/* Data Beheer */}
        {isAdmin() && (
          <div className="p-4 mt-6 bg-red-100 rounded-lg border border-red-300">
            <h3 className="text-xl font-semibold text-red-800 mb-4">Data Beheer</h3>
            <div className="flex flex-wrap items-center justify-between gap-4">
              <span className="text-sm text-red-700">Volledige data (accounts, producten, logs)</span>
              <div className="flex flex-wrap gap-2">
                <button onClick={exportData} className="bg-gray-700 text-white py-2 px-4 rounded-lg font-bold hover:bg-gray-800 transition"><IconDownload /> JSON Exporteren</button>
                <label className="cursor-pointer">
                  <input type="file" onChange={importData} className="hidden" accept=".json" />
                  <span className="bg-gray-700 text-white py-2 px-4 rounded-lg font-bold hover:bg-gray-800 transition"><IconUpload /> JSON Importeren</span>
                </label>
                <button onClick={() => handleAdminAction('resetData')} className="bg-red-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-red-700 transition"><IconExclamation /> Alles Herstellen</button>
              </div>
            </div>
          </div>
        )}

        <hr className="my-6 border-gray-200" />
        <button
          onClick={goHome}
          className="bg-red-600 text-white py-2 px-4 rounded-lg font-bold hover:bg-red-700 transition"
        >
          <IconSignOut /> Uitloggen
        </button>
      </div>
    );
  };

  // Hoofd render
  return (
    <div className="min-h-screen font-sans antialiased text-gray-900 bg-gray-100 flex flex-col items-center">
      <div className="fixed inset-0 bg-gradient-to-br from-green-50 to-red-50 z-0"></div>
      <header className="relative w-full bg-gradient-to-r from-green-700 to-red-700 text-white p-4 shadow-lg z-10 flex items-center gap-4">
        <img className="w-12 h-12 rounded-full p-1 bg-white" src="https://storage.googleapis.com/bacon-prod-uploaded-files/editor-files/6d555c82-e8c0-43ac-91c6-1215b630e2f5/Logo%20rood%20groen.png" alt="Logo" />
        <div className="flex flex-col">
          <strong className="text-xl">Fictief Geld Systeem</strong>
          <span className="text-sm opacity-80">Waterscouting St. Willibrordus · Gouda</span>
        </div>
      </header>

      {/* Render het juiste scherm op basis van de state */}
      <main className="relative z-10 w-full p-4">
        {screen === 'home' && renderHome()}
        {screen === 'pin' && renderPinScreen()}
        {screen === 'user' && renderUserScreen()}
        {screen === 'admin' && renderAdminScreen()}
      </main>

      <footer className="relative z-10 w-full text-center text-gray-500 text-sm mt-8 pb-4">
        <div className="flex justify-center items-center gap-2">
          <IconReact /> Gebouwd met React &amp; <IconTailwind /> Tailwind CSS
          <IconFirebase /> Firestore Enabled
        </div>
        Versie: {APP_VERSION}
      </footer>
    </div>
  );
};

export default App;
