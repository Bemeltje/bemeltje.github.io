<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fictief Geld Systeem</title>
<style>
    body { font-family: Arial, sans-serif; background: #f7f7f7; margin: 0; padding: 0; }
    header { background: #333; color: white; padding: 10px; text-align: center; }
    .container { max-width: 500px; margin: 20px auto; background: white; padding: 20px; border-radius: 10px; }
    button { padding: 8px 12px; border: none; background: #28a745; color: white; border-radius: 5px; cursor: pointer; }
    button:hover { background: #218838; }
    input, select { padding: 8px; margin: 5px 0; width: 100%; }
    .hidden { display: none; }
    .item { border-bottom: 1px solid #ccc; padding: 5px 0; }
</style>
</head>
<body>

<header>
    <h1>Fictief Geld Systeem</h1>
</header>

<div class="container" id="loginScreen">
    <h2>Kies account</h2>
    <select id="accountSelect"></select>
    <input type="password" id="pincode" placeholder="Voer pincode in">
    <button onclick="login()">Inloggen</button>
    <hr>
    <input type="password" id="adminCode" placeholder="Admin pincode">
    <button onclick="adminLogin()">Admin inloggen</button>
</div>

<div class="container hidden" id="userScreen">
    <h2 id="welcome"></h2>
    <p>Saldo: €<span id="saldo"></span></p>
    <div id="productList"></div>
    <button onclick="logout()">Uitloggen</button>
</div>

<div class="container hidden" id="adminScreen">
    <h2>Admin Paneel</h2>
    <h3>Accounts</h3>
    <input id="newName" placeholder="Naam">
    <input id="newPin" placeholder="Pincode">
    <input type="number" id="newSaldo" placeholder="Startsaldo">
    <button onclick="addAccount()">Account toevoegen</button>
    <div id="account