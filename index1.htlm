<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>ULTRA POS SYSTEM</title>
<style>
body { margin:0; font-family:Arial; background:#0a0a0a; color:white; }

header {
  background:linear-gradient(90deg,#6a00ff,#b400ff);
  padding:12px;
  display:flex;
  justify-content:space-between;
}

nav button {
  margin:3px;
  padding:10px;
  cursor:pointer;
  border:none;
  background:#222;
  color:white;
  border-radius:6px;
}

.container { display:none; padding:10px; }
.active { display:block; }

.grid { display:grid; grid-template-columns:1fr 1fr 1fr; gap:10px; }

.box {
  background:#1c1c1c;
  padding:10px;
  border-radius:10px;
}

.item {
  background:#333;
  padding:8px;
  margin:5px 0;
  cursor:pointer;
  border-radius:5px;
}

.good { color:lime; }

.badge {
  background:purple;
  padding:3px 6px;
  border-radius:5px;
  font-size:12px;
}

input, select {
  width:100%;
  padding:8px;
  margin:4px 0;
  border-radius:5px;
  border:none;
}

.ticket {
  background:white;
  color:black;
  padding:10px;
}
</style>
</head>

<body>

<header>
<div>ULTRA POS SYSTEM</div>
<div id="userBox"></div>
</header>

<!-- LOGIN -->
<div id="login">
  <h2>Connexion PIN</h2>
  <input id="pin" placeholder="1234 admin / 1111 manager / 2222 vendeur">
  <button onclick="login()">Connexion</button>
</div>

<!-- APP -->
<div id="app" style="display:none">

<nav>
<button onclick="show('cash')">🧾 Caisse</button>
<button onclick="show('reception')">🎟️ Accueil</button>
<button onclick="show('kitchen')">🍔 Cuisine</button>
<button onclick="show('stats')">📊 Stats</button>
<button onclick="show('admin')">⚙️ Admin</button>
</nav>

<!-- CAISSE -->
<div id="cash" class="container active">
<h2>Caisse tactile</h2>

<div class="grid">

<div class="box">
<h3>Produits</h3>
<div id="products"></div>
</div>

<div class="box">
<h3>Panier</h3>
<div id="cart"></div>
<p>Total: <span id="total">0</span>€</p>
<button onclick="pay()">Encaisser</button>
<button onclick="printTicket()">Ticket</button>
</div>

<div class="box">
<h3>Modes paiement</h3>
<button onclick="setPay('CB')">CB</button>
<button onclick="setPay('Cash')">Cash</button>
<button onclick="setPay('QR')">QR</button>
<p id="payMode">CB</p>
</div>

</div>
</div>

<!-- ACCUEIL -->
<div id="reception" class="container">
<h2>Accueil / Activités</h2>

<div class="box">
<h3>Créer réservation</h3>
<input id="client" placeholder="Nom client">

<select id="activity"></select>
<input id="time" placeholder="Créneau">

<button onclick="addBooking()">Réserver</button>
</div>

<div class="box">
<h3>Réservations</h3>
<div id="bookings"></div>
</div>
</div>

<!-- CUISINE -->
<div id="kitchen" class="container">
<h2>Écran cuisine</h2>
<div id="orders"></div>
</div>

<!-- STATS -->
<div id="stats" class="container">
<h2>Statistiques</h2>
<div class="box">
<p>CA total: <span id="ca">0</span>€</p>
<p>Ventes: <span id="salesCount">0</span></p>
<p>Réservations: <span id="bookCount">0</span></p>
</div>
</div>

<!-- ADMIN -->
<div id="admin" class="container">
<h2>Admin panel</h2>

<div class="box">
<h3>Produits</h3>
<input id="pname" placeholder="Nom">
<input id="pprice" placeholder="Prix">
<button onclick="addProduct()">Ajouter</button>
</div>

<div class="box">
<h3>Activités</h3>
<input id="aname" placeholder="Nom activité">
<button onclick="addActivity()">Ajouter</button>
</div>

</div>

</div>

<script>

let user=null;
let cart=[];
let payMode="CB";

const channel = new BroadcastChannel("pos_sync");

let data = JSON.parse(localStorage.getItem("ultra")) || {
  products:[
    {name:"Boisson",price:2},
    {name:"Snack",price:3}
  ],
  activities:["Parc","Kid Park"],
  bookings:[],
  sales:[],
  orders:[]
};

const users={
  "1234":"admin",
  "1111":"manager",
  "2222":"vendeur"
};

function save(){
  localStorage.setItem("ultra",JSON.stringify(data));
  channel.postMessage(data);
}

channel.onmessage = (e)=>{
  data = e.data;
  render();
};

function login(){
  let pin=pin.value;
  if(users[pin]){
    user=users[pin];
    login.style.display="none";
    app.style.display="block";
    userBox.innerHTML=`${user}`;
    render();
  }
}

// NAV
function show(id){
  document.querySelectorAll(".container").forEach(c=>c.classList.remove("active"));
  document.getElementById(id).classList.add("active");
}

// CART
function addCart(n,p){
  cart.push({n,p});
  render();
}

// PAYMENT
function setPay(m){
  payMode=m;
  payModeEl.innerText=m;
}

function pay(){
  if(cart.length===0)return;

  let total=cart.reduce((a,b)=>a+b.p,0);

  data.sales.push({
    total,
    mode:payMode,
    date:new Date().toLocaleString()
  });

  data.orders.push({items:cart,status:"ready"});

  cart=[];
  save();
  render();
}

// TICKET
function printTicket(){
  let w=window.open("","ticket");
  let total=cart.reduce((a,b)=>a+b.p,0);

  w.document.write(`
  <div class='ticket'>
  <h2>REÇU ULTRA POS</h2>
  <hr>
  ${cart.map(i=>i.n+" "+i.p+"€").join("<br>")}
  <hr>
  TOTAL: ${total.toFixed(2)}€<br>
  Paiement: ${payMode}
  </div>`);
  w.print();
}

// BOOKINGS
function addBooking(){
  data.bookings.push({
    name:client.value,
    activity:activity.value,
    time:time.value,
    status:"check-in"
  });
  save(); render();
}

// ADMIN
function addProduct(){
  data.products.push({name:pname.value,price:+pprice.value});
  save(); render();
}

function addActivity(){
  data.activities.push(aname.value);
  save(); render();
}

// RENDER
function render(){

  products.innerHTML=data.products.map((p,i)=>
    `<div class='item' onclick="addCart('${p.name}',${p.price})">
    ${p.name} - ${p.price}€
    </div>`
  ).join("");

  activity.innerHTML=data.activities.map(a=>`<option>${a}</option>`).join("");

  bookings.innerHTML=data.bookings.map(b=>
    `<div class='item'>
    ${b.name} | ${b.activity} | ${b.time} | ${b.status}
    </div>`
  ).join("");

  orders.innerHTML=data.orders.map(o=>
    `<div class='item'>Commande ${o.items.length} items</div>`
  ).join("");

  ca.innerText=data.sales.reduce((a,b)=>a+b.total,0);
  salesCount.innerText=data.sales.length;
  bookCount.innerText=data.bookings.length;

  cartDiv();
  save();
}

function cartDiv(){
  let c=0;
  cartEl.innerHTML=cart.map(i=>{
    c+=i.p;
    return `<div>${i.n} ${i.p}€</div>`;
  }).join("");
  total.innerText=c.toFixed(2);
}

// refs
let products=document.getElementById("products");
let cartEl=document.getElementById("cart");
let total=document.getElementById("total");
let bookings=document.getElementById("bookings");
let orders=document.getElementById("orders");
let activity=document.getElementById("activity");
let payModeEl=document.getElementById("payMode");
let userBox=document.getElementById("userBox");

</script>

</body>
</html>
