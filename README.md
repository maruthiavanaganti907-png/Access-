<!DOCTYPE html>
<html>
<head>
<title>Farmer Market Pro</title>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage.js"></script>

<style>
body { font-family: Arial; background:#eef; }
.card { background:white; padding:15px; margin:10px; }
button { padding:8px; background:green; color:white; border:none; }
</style>
</head>

<body>

<div class="card">
<h2>Login / Register</h2>
<input id="email" placeholder="Email">
<input id="password" type="password" placeholder="Password">
<select id="role">
<option value="farmer">Farmer</option>
<option value="buyer">Buyer</option>
</select>
<button onclick="register()">Register</button>
<button onclick="login()">Login</button>
</div>

<div id="farmerUI" style="display:none" class="card">
<h2>Farmer Dashboard</h2>
<input id="pname" placeholder="Product">
<input id="pprice" placeholder="Price ₹">
<input type="file" id="file">
<button onclick="uploadProduct()">Upload</button>
<div id="farmerProducts"></div>
<div id="orders"></div>
</div>

<div id="buyerUI" style="display:none" class="card">
<h2>Buyer Dashboard</h2>
<div id="products"></div>
<div id="cart"></div>
<button onclick="checkout()">Pay (UPI)</button>
</div>

<script>
// 🔴 PASTE YOUR FIREBASE CONFIG HERE
const firebaseConfig = {
  apiKey: "YOUR_KEY",
};

firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();
const storage = firebase.storage();

let currentUserRole = "";
let cart = [];

// REGISTER
function register() {
    auth.createUserWithEmailAndPassword(email.value, password.value)
    .then(user=>{
        db.collection("users").doc(user.user.uid).set({
            role: role.value
        });
        alert("Registered!");
    });
}

// LOGIN
function login() {
    auth.signInWithEmailAndPassword(email.value, password.value)
    .then(res=>{
        db.collection("users").doc(res.user.uid).get().then(doc=>{
            currentUserRole = doc.data().role;

            if(currentUserRole==="farmer"){
                farmerUI.style.display="block";
                loadFarmerProducts();
                loadOrders();
            } else {
                buyerUI.style.display="block";
                loadProducts();
            }
        });
    });
}

// UPLOAD PRODUCT
function uploadProduct() {
    let file = document.getElementById("file").files[0];
    let ref = storage.ref("products/"+file.name);

    ref.put(file).then(()=>{
        ref.getDownloadURL().then(url=>{
            db.collection("products").add({
                name:pname.value,
                price:pprice.value,
                image:url
            });
            loadFarmerProducts();
        });
    });
}

// LOAD PRODUCTS
function loadProducts() {
    db.collection("products").onSnapshot(snapshot=>{
        let html="";
        snapshot.forEach(doc=>{
            let p=doc.data();
            html+=`
            <div>
                <img src="${p.image}" width="80">
                ${p.name} - ₹${p.price}
                <button onclick='addCart("${p.name}",${p.price})'>Add</button>
            </div>`;
        });
        products.innerHTML=html;
    });
}

// CART
function addCart(name,price){
    cart.push({name,price});
    showCart();
}

function showCart(){
    let html="";
    cart.forEach((c,i)=>{
        html+=`${c.name} - ₹${c.price} <button onclick="removeCart(${i})">❌</button><br>`;
    });
    cartDiv=document.getElementById("cart");
    cartDiv.innerHTML=html;
}

function removeCart(i){
    cart.splice(i,1);
    showCart();
}

// CHECKOUT
function checkout(){
    cart.forEach(c=>{
        db.collection("orders").add(c);
    });
    alert("UPI Payment Success");
    cart=[];
    showCart();
}

// FARMER VIEW
function loadFarmerProducts(){
    db.collection("products").onSnapshot(snapshot=>{
        let html="";
        snapshot.forEach(doc=>{
            let p=doc.data();
            html+=`${p.name} - ₹${p.price}<br>`;
        });
        farmerProducts.innerHTML=html;
    });
}

// ORDERS
function loadOrders(){
    db.collection("orders").onSnapshot(snapshot=>{
        let html="";
        snapshot.forEach((doc,i)=>{
            let o=doc.data();
            html+=`${o.name} - ₹${o.price} 
            <button onclick="acceptOrder('${doc.id}')">Accept</button><br>`;
        });
        orders.innerHTML=html;
    });
}

function acceptOrder(id){
    db.collection("orders").doc(id).delete();
    alert("Order Accepted");
}

</script>

</body>
</html>
