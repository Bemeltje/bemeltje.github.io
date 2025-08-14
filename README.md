<!-- Alleen de verschillen van vorige versie -->
<script>
let adminPIN = "9999";
let accounts = JSON.parse(localStorage.getItem("accounts")) || [
    {name: "Jan", pin: "1234", saldo: 10.00, type: "vast"},
    {name: "Piet", pin: "5678", saldo: 5.00, type: "gast"}
];
let products = JSON.parse(localStorage.getItem("products")) || [
    {name: "Chips", price: 0.75},
    {name: "Bier", price: 0.75},
    {name: "Cola", price: 1.00}
];
let currentUserIndex = null;

function loadAccountButtons() {
    let container = document.getElementById("accountButtons");
    container.innerHTML = "";
    accounts.forEach((acc, i) => {
        let btn = document.createElement("button");
        btn.style.width = "100%";
        btn.textContent = `${acc.name} (€${acc.saldo.toFixed(2)}) [${acc.type}]`;
        if (acc.saldo < 0) {
            btn.style.background = "#e94e4e";
        }
        btn.onclick = () => selectAccount(i);
        container.appendChild(btn);
    });
}
loadAccountButtons();

function selectAccount(index) {
    currentUserIndex = index;
    document.getElementById("pincode").value = ""; // reset pin
    document.getElementById("homeScreen").classList.add("hidden");
    document.getElementById("pinScreen").classList.remove("hidden");
    document.getElementById("selectedUserName").textContent = "Account: " + accounts[index].name;
}

function checkLogin() {
    let pin = document.getElementById("pincode").value;
    if (accounts[currentUserIndex].pin === pin) {
        document.getElementById("pincode").value = ""; // wis pin
        document.getElementById("pinScreen").classList.add("hidden");
        document.getElementById("userScreen").classList.remove("hidden");
        document.getElementById("welcome").textContent = "Welkom " + accounts[currentUserIndex].name;
        updateUserScreen();
    } else {
        alert("Verkeerde pincode!");
    }
}

function updateUserScreen() {
    let acc = accounts[currentUserIndex];
    document.getElementById("saldo").textContent = acc.saldo.toFixed(2);
    let list = document.getElementById("productList");
    list.innerHTML = "";
    products.forEach((p, i) => {
        let div = document.createElement("div");
        div.classList.add("item");
        div.innerHTML = `<span>${p.name} - €${p.price.toFixed(2)}</span> <button onclick="buyProduct(${i})">Koop</button>`;
        list.appendChild(div);
    });
}

function buyProduct(i) {
    let acc = accounts[currentUserIndex];
    let prijs = products[i].price;

    if (!confirm(`Weet je zeker dat je '${products[i].name}' wilt kopen voor €${prijs.toFixed(2)}?`)) return;

    if (acc.type === "gast" && acc.saldo - prijs < 0) {
        alert("Gast mag niet onder €0 komen!");
        return;
    }
    if (acc.type === "vast" && acc.saldo - prijs < -10) {
        alert("Vast mag niet verder dan -€10 komen!");
        return;
    }

    acc.saldo -= prijs;
    saveData();
    updateUserScreen();
    loadAccountButtons(); 
}

function addAccount() {
    let name = document.getElementById("newName").value;
    let pin = document.getElementById("newPin").value;
    let saldo = parseFloat(document.getElementById("newSaldo").value);
    let type = document.getElementById("newType").value;
    if (!name || !pin || isNaN(saldo)) { alert("Vul alle velden in!"); return; }
    if (pin.length > 4) { alert("Pincode mag maximaal 4 cijfers zijn!"); return; }
    accounts.push({name, pin, saldo, type});
    saveData();
    loadAccountButtons();
    updateAdminScreen();
}
</script>

<!-- Toevoeging in admin UI -->
<select id="newType">
    <option value="gast">Gast</option>
    <option value="vast">Vast</option>
</select>

<!-- PIN input beperken -->
<input type="password" id="pincode" maxlength="4" placeholder="Voer pincode in">
<input id="newPin" maxlength="4" placeholder="Pincode">