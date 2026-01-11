# drink_diary
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🥤 飲料日記 App</title>
<style>
body {
    background:#111;
    color:#eee;
    font-family:Arial,sans-serif;
    padding:10px;
    margin:0;
}
h1 { text-align:center; margin-bottom:15px; }
.form { background:#1c1c1c; padding:15px; border-radius:12px; margin-bottom:15px; }
label { display:block; margin-top:10px; color:#ccc; font-size:16px; }
input, select, button { width:100%; padding:12px; margin-top:5px; background:#2a2a2a; color:#fff; border:1px solid #444; border-radius:8px; font-size:16px; }
button { margin-top:12px; background:#444; cursor:pointer; }
button:hover { background:#666; }
.cards { display:flex; flex-direction:column; gap:10px; margin-bottom:20px; }
.card {
    background:#1c1c1c;
    padding:12px;
    border-radius:10px;
    border-left:4px solid #0af;
}
.card p { margin:4px 0; font-size:14px; }
.delete-btn { background:#800; color:#fff; border:none; padding:5px 8px; border-radius:4px; cursor:pointer; font-size:12px; float:right; }
.delete-btn:hover { background:#f00; }
.stat-box { background:#1c1c1c; padding:12px; border-radius:10px; margin-bottom:15px; }
.stat-box h2 { margin-top:0; color:#0af; text-align:center; }
@media(max-width:480px){ label{font-size:14px;} input,select,button{font-size:14px;padding:10px;} .card p{font-size:13px;} }
</style>
</head>
<body>

<h1>🥤 飲料日記 App</h1>

<div class="form">
<label>日期 <input type="date" id="date"></label>
<label>店名 <input type="text" id="store" placeholder="例如：CoCo"></label>
<label>飲料品項 <input type="text" id="drink" placeholder="例如：珍珠奶茶"></label>
<label>冰塊 
    <select id="ice">
        <option>正常冰</option>
        <option>少冰</option>
        <option>微冰</option>
        <option>去冰</option>
        <option>溫</option>
    </select>
</label>
<label>甜度
    <select id="sugar">
        <option>正常糖</option>
        <option>七分糖</option>
        <option>五分糖</option>
        <option>三分糖</option>
        <option>一分糖</option>
        <option>無糖</option>
    </select>
</label>
<label>價錢 <input type="number" id="price" placeholder="例如：55"></label>
<button onclick="addRecord()">新增紀錄</button>
<button onclick="exportCSV()">匯出 CSV</button>
</div>

<div class="stat-box">
<h2>統計資訊</h2>
<p id="monthly">每月花費：0</p>
<p id="favDrink">最常買飲料：-</p>
<p id="favSugar">甜度偏好：-</p>
</div>

<div class="cards" id="records"></div>

<script>
const recordsEl=document.getElementById("records");
const monthlyEl=document.getElementById("monthly");
const favDrinkEl=document.getElementById("favDrink");
const favSugarEl=document.getElementById("favSugar");

function loadRecords(){
    let data=[];
    try{ data=JSON.parse(localStorage.getItem("drinkRecords"))||[]; }catch(e){ data=[]; }
    recordsEl.innerHTML="";
    data.forEach((r,i)=>addCard(r,i));
    updateStats(data);
}

function addCard(r,i){
    const card=document.createElement("div");
    card.className="card";
    card.innerHTML=`<p><strong>${r.date}</strong></p>
                    <p>店名：${r.store}</p>
                    <p>飲料：${r.drink}</p>
                    <p>冰塊：${r.ice} / 甜度：${r.sugar}</p>
                    <p>價錢：$${r.price} <button class="delete-btn" onclick="deleteRecord(${i})">刪除</button></p>`;
    recordsEl.appendChild(card);
}

function addRecord(){
    const record={date:date.value, store:store.value, drink:drink.value, ice:ice.value, sugar:sugar.value, price:price.value};
    if(!record.date||!record.store||!record.drink||!record.price){ alert("請完整填寫日期、店名、飲料品項和價錢"); return; }
    let data=JSON.parse(localStorage.getItem("drinkRecords")||"[]");
    data.push(record);
    localStorage.setItem("drinkRecords", JSON.stringify(data));
    addCard(record,data.length-1);
    document.querySelector(".form").reset();
    updateStats(data);
}

function deleteRecord(index){
    let data=JSON.parse(localStorage.getItem("drinkRecords")||"[]");
    data.splice(index,1);
    localStorage.setItem("drinkRecords", JSON.stringify(data));
    loadRecords();
}

// 統計功能
function updateStats(data){
    // 每月花費
    const monthMap={};
    data.forEach(r=>{
        if(!r.date) return;
        const m=r.date.slice(0,7);
        monthMap[m]=(monthMap[m]||0)+parseFloat(r.price);
    });
    let monthlyText=Object.entries(monthMap).map(([m,v])=>`${m}：$${v}`).join("，");
    monthlyEl.textContent=monthlyText||"每月花費：0";

    // 最常買飲料
    const drinkMap={};
    data.forEach(r=>{ drinkMap[r.drink]=(drinkMap[r.drink]||0)+1; });
    const maxDrink=Object.keys(drinkMap).reduce((a,b)=>drinkMap[a]>=drinkMap[b]?a:b,"-");
    favDrinkEl.textContent=maxDrink?`最常買飲料：${maxDrink}`:"最常買飲料：-";

    // 甜度偏好
    const sugarMap={};
    data.forEach(r=>{ sugarMap[r.sugar]=(sugarMap[r.sugar]||0)+1; });
    const maxSugar=Object.keys(sugarMap).reduce((a,b)=>sugarMap[a]>=sugarMap[b]?a:b,"-");
    favSugarEl.textContent=maxSugar?`甜度偏好：${maxSugar}`:"甜度偏好：-";
}

// 匯出 CSV (含統計)
function exportCSV(){
    let data=JSON.parse(localStorage.getItem("drinkRecords")||"[]");
    if(data.length===0){ alert("沒有紀錄可以匯出"); return; }

    let csv="日期,店名,飲料品項,冰塊,甜度,價錢\n";
    data.forEach(r=>{csv+=`${r.date},${r.store},${r.drink},${r.ice},${r.sugar},${r.price}\n`;});

    csv+="\n統計資訊\n";
    csv+=monthlyEl.textContent+"\n";
    csv+=favDrinkEl.textContent+"\n";
    csv+=favSugarEl.textContent+"\n";

    const blob=new Blob([csv],{type:"text/csv"});
    const url=URL.createObjectURL(blob);
    const a=document.createElement("a");
    a.href=url;
    a.download="飲料日記.csv";
    a.click();
    URL.revokeObjectURL(url);
}

loadRecords();
</script>

</body>
</html>
