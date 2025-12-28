<!DOCTYPE html>
<html lang="ur">
<head>
<meta charset="UTF-8">
<title>Ultimate Billing & Stock Management App</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body { font-family: Arial, sans-serif; background:#f2f2f2; direction:rtl; margin:0; padding:0;}
.container { max-width:1200px; margin:10px auto; background:white; padding:15px; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.2);}
h2,h3,h4{text-align:center;color:#333;}
input,select,button{padding:5px;margin:5px;width:90%;}
table{width:100%;border-collapse:collapse;margin-top:10px;overflow-x:auto; display:block;}
th,td{border:1px solid #ccc;padding:5px;text-align:center;}
button{cursor:pointer;}
img{width:40px;height:40px;}
@media (max-width:768px){
  input, select, button{width:95%;}
  table{font-size:12px;}
  img{width:30px;height:30px;}
}
</style>
</head>
<body>

<div class="container">
<h2>Ultimate Billing & Stock Management App</h2>

<!-- Mode Selection -->
<div style="text-align:center; margin-bottom:10px;">
<button onclick="showCustomerMode()">کسٹمر موڈ</button>
<button onclick="showSellerMode()">سیلر موڈ</button>
<button onclick="showReports()">رپورٹس</button>
</div>

<!-- Customer Mode -->
<div id="customerMode" style="display:none; margin-top:10px;">
<h3>کسٹمر موڈ</h3>
<input type="text" id="searchItem" placeholder="آئٹم سرچ کریں" oninput="searchItem()">
<div style="overflow-x:auto;">
<table id="stockTable">
<thead>
<tr><th>آئٹم</th><th>تصویر</th><th>موجود مقدار</th><th>فی قیمت</th><th>مقدار</th><th>Action</th></tr>
</thead>
<tbody></tbody>
</table>
</div>

<h3>Bill</h3>
<div style="overflow-x:auto;">
<table id="billTable">
<thead>
<tr><th>آئٹم</th><th>مقدار</th><th>فی قیمت</th><th>کل رقم</th><th>تصویر</th></tr>
</thead>
<tbody></tbody>
</table>
</div>

<p>
Discount (% یا رقم): <input type="number" id="discount" value="0"> 
<button onclick="generateBillPDF()">Bill PDF</button>
</p>
</div>

<!-- Seller Mode -->
<div id="sellerMode" style="display:none; margin-top:10px;">
<h3>سیلر موڈ</h3>
<p>Password: <input type="password" id="sellerPass"> <button onclick="checkPassword()">Login</button></p>
<div id="sellerFunctions" style="display:none;">

<h4>Stock Management</h4>
<input type="text" id="newItemName" placeholder="آئٹم کا نام">
<input type="number" id="newItemQty" placeholder="مقدار">
<input type="number" id="newItemPrice" placeholder="Buying Price">
<input type="number" id="newItemSellPrice" placeholder="Selling Price">
<select id="newItemCategory">
<option value="Steel">Steel</option>
<option value="Plastic">Plastic</option>
<option value="Rubber">Rubber</option>
<option value="Grocery">Grocery</option>
</select>
<input type="file" id="newItemImage" accept="image/*">
<button onclick="addItem()">Add / Update Item</button>

<h4>Stock List</h4>
<div style="overflow-x:auto;">
<table id="sellerStockTable">
<thead><tr><th>آئٹم</th><th>تصویر</th><th>مقدار</th><th>Buying Price</th><th>Selling Price</th><th>Category</th></tr></thead>
<tbody></tbody>
</table>
</div>

<h4>Backup / Restore</h4>
<button onclick="backupData()">Backup JSON</button>
<input type="file" id="restoreFile" onchange="restoreData(event)">
</div>
</div>

<!-- Reports -->
<div id="reportsMode" style="display:none; margin-top:10px;">
<h3>Daily & Monthly Reports</h3>
<button onclick="generateDailyPDF()">Generate Today's PDF</button>
<button onclick="generateMonthlyPDF()">Generate Monthly PDF</button>
<div id="reportsList"></div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script>
// ----------------- Data -----------------
let stock = JSON.parse(localStorage.getItem('stockData')) || [
{ name:"گلاس", qty:12, buy:50, sell:100, category:"Plastic", img:null },
{ name:"پلیٹ", qty:20, buy:20, sell:40, category:"Steel", img:null },
{ name:"چمچ", qty:50, buy:5, sell:10, category:"Steel", img:null }
];
let bill = [];
let sellerPassword = "1234";
let purchaseHistory = JSON.parse(localStorage.getItem('purchaseHistory')) || [];
let salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];

// ----------------- Mode Switch -----------------
function showCustomerMode(){ document.getElementById("customerMode").style.display="block"; document.getElementById("sellerMode").style.display="none"; document.getElementById("reportsMode").style.display="none"; refreshStockTable(); }
function showSellerMode(){ document.getElementById("customerMode").style.display="none"; document.getElementById("sellerMode").style.display="block"; document.getElementById("reportsMode").style.display="none"; }
function showReports(){ document.getElementById("customerMode").style.display="none"; document.getElementById("sellerMode").style.display="none"; document.getElementById("reportsMode").style.display="block"; }

// ----------------- Seller Mode -----------------
function checkPassword(){
  let pass = document.getElementById("sellerPass").value;
  if(pass===sellerPassword){ document.getElementById("sellerFunctions").style.display="block"; refreshSellerStockTable(); alert("Login Successful"); }
  else { alert("Password غلط ہے"); }
}

function addItem(){
  let name=document.getElementById("newItemName").value;
  let qty=parseInt(document.getElementById("newItemQty").value);
  let buy=parseFloat(document.getElementById("newItemPrice").value);
  let sell=parseFloat(document.getElementById("newItemSellPrice").value);
  let category=document.getElementById("newItemCategory").value;
  let file = document.getElementById("newItemImage").files[0];
  let reader = new FileReader();
  reader.onload = function(e){
    let imgData = e.target.result;
    let item=stock.find(i=>i.name===name);
    if(item){ item.qty += qty; item.buy=buy; item.sell=sell; item.category=category; item.img=imgData; }
    else{ stock.push({name:name, qty:qty, buy:buy, sell:sell, category:category, img:imgData}); }
    purchaseHistory.push({date:new Date().toLocaleDateString(), name:name, qty:qty, price:buy, category:category});
    saveStock();
    refreshSellerStockTable();
    alert("Item Added / Updated");
  }
  if(file){ reader.readAsDataURL(file); } else { reader.onload({target:{result:null}}); }
}

function refreshSellerStockTable(){
  let tbody = document.getElementById("sellerStockTable").getElementsByTagName("tbody")[0];
  tbody.innerHTML="";
  stock.forEach(i=>{ tbody.innerHTML += `<tr><td>${i.name}</td><td>${i.img?'<img src="'+i.img+'">':''}</td><td>${i.qty}</td><td>${i.buy}</td><td>${i.sell}</td><td>${i.category}</td></tr>`; });
}
function saveStock(){ localStorage.setItem('stockData', JSON.stringify(stock)); localStorage.setItem('purchaseHistory', JSON.stringify(purchaseHistory)); localStorage.setItem('salesHistory', JSON.stringify(salesHistory)); }

// ----------------- Customer Mode -----------------
function refreshStockTable(){
  let tbody = document.getElementById("stockTable").getElementsByTagName("tbody")[0];
  tbody.innerHTML="";
  stock.forEach(i=>{ tbody.innerHTML += `<tr><td>${i.name}</td><td>${i.img?'<img src="'+i.img+'">':''}</td><td>${i.qty}</td><td>${i.sell}</td><td><input type="number" min="1" max="${i.qty}" value="1" id="qty_${i.name}"></td><td><button onclick="addToBill('${i.name}')">Add</button></td></tr>`; });
}

function searchItem(){
  let val=document.getElementById("searchItem").value.toLowerCase();
  let tbody = document.getElementById("stockTable").getElementsByTagName("tbody")[0];
  tbody.innerHTML="";
  stock.filter(i=>i.name.toLowerCase().includes(val)).forEach(i=>{ tbody.innerHTML += `<tr><td>${i.name}</td><td>${i.img?'<img src="'+i.img+'">':''}</td><td>${i.qty}</td><td>${i.sell}</td><td><input type="number" min="1" max="${i.qty}" value="1" id="qty_${i.name}"></td><td><button onclick="addToBill('${i.name}')">Add</button></td></tr>`; });
}

function addToBill(name){
  let item = stock.find(i=>i.name===name);
  let qty = parseInt(document.getElementById(`qty_${name}`).value);
  if(qty>item.qty){ alert("Stock کم ہے"); return; }
  let billItem = bill.find(b=>b.name===name);
  if(billItem){ billItem.qty+=qty; billItem.total=billItem.qty*item.sell; }
  else{ bill.push({name:name, qty:qty, price:item.sell, total:qty*item.sell, img:item.img}); }
  item.qty -= qty;
  salesHistory.push({date:new Date().toLocaleDateString(), name:item.name, qty:qty, price:item.sell});
  saveStock();
  refreshStockTable(); refreshBillTable();
}

function refreshBillTable(){
  let tbody = document.getElementById("billTable").getElementsByTagName("tbody")[0];
  tbody.innerHTML="";
  bill.forEach(b=>{ tbody.innerHTML += `<tr><td>${b.name}</td><td>${b.qty}</td><td>${b.price}</td><td>${b.total}</td><td>${b.img?'<img src="'+b.img+'">':''}</td></tr>`; });
}

// ----------------- PDF Generation -----------------
function generateBillPDF(){
  const { jsPDF } = window.jspdf;
  let doc = new jsPDF({orientation:"p", unit:"pt", format:"a4"});
  doc.setFontSize(14);
  doc.text("کسٹمر بل", 40, 40);
  let y=60;
  bill.forEach(b=>{ doc.text(`${b.name}   ${b.qty}   ${b.price}   ${b.total}`, 40, y); y+=20; });
  let discount = parseFloat(document.getElementById("discount").value)||0;
  let total = bill.reduce((a,b)=>a+b.total,0);
  let final = total-discount;
  doc.text(`Total: ${total}`, 40, y); y+=20;
  doc.text(`Discount: ${discount}`, 40, y); y+=20;
  doc.text(`Net Payable: ${final}`, 40, y); y+=20;
  doc.save(`Bill_${new Date().toLocaleDateString()}.pdf`);
  bill=[]; refreshBillTable(); refreshStockTable();
}

// ----------------- Reports -----------------
function generateDailyPDF(){
  const { jsPDF } = window.jspdf;
  let doc = new jsPDF();
  doc.setFontSize(14);
  doc.text("Daily Report - "+new Date().toLocaleDateString(), 40, 40);
  let y=60;
  salesHistory.filter(s=>s.date===new Date().toLocaleDateString()).forEach(s=>{
    doc.text(`${s.name}   ${s.qty}   ${s.price}   ${s.qty*s.price}`, 40, y);
    y+=20;
  });
  doc.save(`DailyReport_${new Date().toLocaleDateString()}.pdf`);
  alert("Daily PDF Generated");
}

function generateMonthlyPDF(){
  const { jsPDF } = window.jspdf;
  let doc = new jsPDF();
  doc.setFontSize(14);
  doc.text("Monthly Report - "+new Date().toLocaleDateString(), 40, 40);
  let y=60;
  let month = new Date().getMonth();
  salesHistory.filter(s=>new Date(s.date).getMonth()===month).forEach(s=>{
    doc.text(`${s.date}  ${s.name}  ${s.qty}  ${s.price}  ${s.qty*s.price}`, 40, y);
    y+=20; if(y>800){ doc.addPage(); y=40; }
  });
  doc.save(`MonthlyReport_${new Date().toLocaleDateString()}.pdf`);
  alert("Monthly PDF Generated");
}

// ----------------- Backup / Restore -----------------
function backupData(){
  let data = {stock:stock, salesHistory:salesHistory, purchaseHistory:purchaseHistory};
  let dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(data));
  let dlAnchorElem = document.createElement('a');
  dlAnchorElem.setAttribute("href", dataStr);
  dlAnchorElem.setAttribute("download","BillingAppBackup_"+new Date().toLocaleDateString()+".json");
  dlAnchorElem.click();
}
function restoreData(event){
  let file = event.target.files[0];
  let reader = new FileReader();
  reader.onload = function(e){
    let data = JSON.parse(e.target.result);
    stock = data.stock; salesHistory = data.salesHistory; purchaseHistory = data.purchaseHistory;
    saveStock(); refreshStockTable(); refreshSellerStockTable(); alert("Data Restored");
  }
  reader.readAsText(file);
}
</script>
</body>
</html>
