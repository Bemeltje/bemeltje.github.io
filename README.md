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

  <h3
