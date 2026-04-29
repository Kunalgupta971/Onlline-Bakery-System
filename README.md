# Onlline-Bakery-System
An Online Bakery System is a web-based platform where customers can browse bakery items, add them to a cart, place orders, make online payments, and track deliveries. It includes features like user accounts, product listings, and order management, helping bakeries provide convenient, efficient, and modern digital services to customers.
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ultimate Bakery App</title>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">
<style>
:root{--primary:#e23744;--bg:#fafafa}
body{margin:0;font-family:'Poppins',sans-serif;background:var(--bg)}
header{background:linear-gradient(90deg,#ff7a7a,#e23744);color:#fff;padding:15px;display:flex;gap:10px;align-items:center}
header input,select{padding:8px;border:none;border-radius:8px}
.container{display:flex}
.products{flex:3;display:grid;grid-template-columns:repeat(auto-fill,minmax(240px,1fr));gap:15px;padding:10px}
.card{background:#fff;border-radius:12px;overflow:hidden;box-shadow:0 4px 12px rgba(0,0,0,.1);transition:.3s}
.card:hover{transform:translateY(-5px)}
.card img{width:100%;height:160px;object-fit:cover}
.card-body{padding:10px}
.price{color:var(--primary)}
button{background:var(--primary);color:#fff;border:none;padding:6px;border-radius:6px;cursor:pointer}
#cart{flex:1;background:#fff;padding:10px}
.modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,.6);justify-content:center;align-items:center}
.modal-content{background:#fff;padding:20px;border-radius:10px;text-align:center}
.qr{width:140px;margin:10px auto}
</style>
</head>
<body>
<header>
<h2>🍰 Bakery</h2>
<input id="search" placeholder="Search..." onkeyup="searchProducts()">
<select onchange="filterCategory(this.value)"><option value="all">All</option><option value="cake">Cakes</option><option value="pastry">Pastries</option></select>
<select onchange="sortPrice(this.value)"><option value="default">Sort</option><option value="low">Low→High</option><option value="high">High→Low</option></select>
</header>
<div class="container">
<div class="products" id="productList"></div>
<div id="cart">
<h3>Cart</h3>
<ul id="cartItems"></ul>
<p>Total: ₹<span id="total">0</span></p>
<button onclick="openPayment()">Pay Now</button>
</div>
</div>

<div class="modal" id="paymentModal">
<div class="modal-content">
<h3>Select Payment</h3>
<button onclick="pay('GPay')">GPay</button>
<button onclick="pay('PhonePe')">PhonePe</button>
<button onclick="pay('Paytm')">Paytm</button>
<p>Scan QR</p>
<img class="qr" src="https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=BakeryPayment">
<br><button onclick="closePayment()">Cancel</button>
</div>
</div>

<script>
let products=[
{id:1,name:"Chocolate Cake",price:500,category:"cake",img:"https://images.unsplash.com/photo-1578985545062-69928b1d9587?w=400"},
{id:2,name:"Black Forest Cake",price:600,category:"cake",img:"https://images.unsplash.com/photo-1605475128023-5c0cbe4c6c66?w=400"},
{id:3,name:"Red Velvet Cake",price:700,category:"cake",img:"https://images.unsplash.com/photo-1606313564200-e75d5e30476c?w=400"},
{id:4,name:"Cheesecake",price:650,category:"cake",img:"https://images.unsplash.com/photo-1565958011703-44f9829ba187?w=400"},
{id:5,name:"Pineapple Cake",price:550,category:"cake",img:"https://images.unsplash.com/photo-1621303837063-9c8b6e3f0b12?w=400"},
{id:6,name:"Strawberry Cupcake",price:120,category:"pastry",img:"https://images.unsplash.com/photo-1587668178277-295251f900ce?w=400"},
{id:7,name:"Blueberry Muffin",price:180,category:"pastry",img:"https://images.unsplash.com/photo-1604908554027-56a0d3d8249e?w=400"},
{id:8,name:"Croissant",price:220,category:"pastry",img:"https://images.unsplash.com/photo-1555507036-ab1f4038808a?w=400"},
{id:9,name:"Brownie",price:250,category:"pastry",img:"https://images.unsplash.com/photo-1606312619070-d48b4c652a52?w=400"},
{id:10,name:"Chocolate Eclair",price:200,category:"pastry",img:"https://images.unsplash.com/photo-1623334044303-241021148842?w=400"},
{id:11,name:"Fruit Tart",price:300,category:"pastry",img:"https://images.unsplash.com/photo-1464306076886-da185f6a9d05?w=400"},
{id:12,name:"Macarons",price:400,category:"pastry",img:"https://images.unsplash.com/photo-1559622214-7a6b3c4a2c8d?w=400"},
{id:13,name:"Banana Bread",price:280,category:"pastry",img:"https://images.unsplash.com/photo-1608198093002-ad4e005484ec?w=400"}
];

let cart=JSON.parse(localStorage.getItem("cart"))||[];
let orders=JSON.parse(localStorage.getItem("orders"))||[];

function displayProducts(list=products){
let c=document.getElementById("productList");c.innerHTML="";
list.forEach(p=>{
c.innerHTML+=`<div class='card'>
<img src='${p.img}'>
<div class='card-body'>
<h4>${p.name}</h4>
<p class='price'>₹${p.price}</p>
<button onclick='addToCart(${p.id})'>Add</button>
</div></div>`});}

function addToCart(id){let item=cart.find(i=>i.id===id);item?item.qty++:cart.push({...products.find(p=>p.id===id),qty:1});saveCart();}

function saveCart(){localStorage.setItem("cart",JSON.stringify(cart));renderCart();}

function renderCart(){let list=document.getElementById("cartItems");list.innerHTML="";let total=0;
cart.forEach(i=>{total+=i.price*i.qty;
list.innerHTML+=`<li>${i.name} x ${i.qty} ₹${i.price*i.qty}
<button onclick='changeQty(${i.id},1)'>+</button>
<button onclick='changeQty(${i.id},-1)'>-</button></li>`});
document.getElementById("total").innerText=total;}

function changeQty(id,d){let i=cart.find(x=>x.id===id);i.qty+=d;if(i.qty<=0)cart=cart.filter(x=>x.id!==id);saveCart();}

function searchProducts(){let q=search.value.toLowerCase();displayProducts(products.filter(p=>p.name.toLowerCase().includes(q)));}

function filterCategory(c){displayProducts(c==='all'?products:products.filter(p=>p.category===c));}

function sortPrice(t){let s=[...products];if(t==='low')s.sort((a,b)=>a.price-b.price);if(t==='high')s.sort((a,b)=>b.price-a.price);displayProducts(s);} 

function openPayment(){if(!cart.length)return alert("Cart empty");paymentModal.style.display="flex"}
function closePayment(){paymentModal.style.display="none"}

function pay(method){
let order={items:[...cart],total:cart.reduce((a,b)=>a+b.price*b.qty,0),method,status:"Preparing"};
orders.push(order);
localStorage.setItem("orders",JSON.stringify(orders));
alert(method+" Payment Successful ✅");
trackOrder(order);
cart=[];saveCart();closePayment();
}

function trackOrder(order){
setTimeout(()=>{order.status="Out for Delivery";alert("Order is Out for Delivery 🚚")},3000);
setTimeout(()=>{order.status="Delivered";alert("Order Delivered 🎉")},6000);
}

displayProducts();renderCart();
</script>
</body>
</html>
