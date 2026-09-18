# receipt-generator<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>小票生成器</title>
<style>
*{box-sizing:border-box;font-family:system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;}
body{background:#f2f2f2;padding:16px;margin:0;}
.tip{background:#fff8cc;color:#856404;padding:10px 12px;border-radius:8px;margin-bottom:16px;font-size:14px;}
.wrap{display:flex;gap:24px;flex-wrap:wrap;justify-content:center;}
.panel{width:340px;background:#fff;padding:16px;border-radius:12px;box-shadow:0 2px 10px #00000014;}
.previewBox{width:340px;background:#fff;}
@media(max-width:720px){
  .panel,.previewBox{width:100%;max-width:400px;}
  body{padding:12px;}
  button{width:100%;margin:4px 0;}
}
/* 小票主体 */
.receipt{
  width:340px;
  max-width:100%;
  background:#ffffff;
  margin:0 auto;
  border-radius:8px;
  overflow:hidden;
}
/* 顶部装饰栏 25px，仅向浅色过渡，杜绝脏深色 */
.decor-top{
  height:25px;
  background:linear-gradient(180deg,#e1d5ff,#ede6ff,#f6f2ff);
  overflow:hidden;
}

/* 底部装饰栏 25px，仅向浅色过渡 */
.decor-bottom{
  height:25px;
  background:linear-gradient(180deg,#f6f2ff,#ede6ff,#e1d5ff);
  overflow:hidden;
}

.receipt-content{padding:18px 22px;}
.title{text-align:center;font-size:32px;color:#8b78b0;margin:6px 0 16px 0;font-weight:500;}
table{
  width:100%;
  border-collapse:collapse;
  margin:10px 0;
  table-layout:fixed;
}
colgroup col{width:25%;}
table th,table td{
  padding:8px 2px;
  font-size:14px;
  overflow:hidden;
  white-space:nowrap;
}
table th:nth-child(1),table td:nth-child(1){text-align:left;}
table th:nth-child(2),table td:nth-child(2){text-align:right;}
table th:nth-child(3),table td:nth-child(3){text-align:right;}
table th:nth-child(4),table td:nth-child(4){text-align:right;}
.line{border-bottom:1px solid #333;margin:16px 0;}
.sum-area{margin:18px 0 22px 0;font-size:14px;line-height:1.6;}

/* 二维码区域 */
.qr-row{
  display:flex;
  justify-content:space-between;
  gap:8px;
  padding:0 10px 20px 10px;
}
.qr-box{text-align:center;flex:1;}
.qr-square{
  width:84px;height:84px;border:1px solid #aaa;border-radius:8px;
  display:flex;align-items:center;justify-content:center;
  margin:0 auto 5px auto;
  background:#fff;
}
.qr-circle{
  width:84px;height:84px;border:1px solid #aaa;border-radius:50%;
  display:flex;align-items:center;justify-content:center;
  margin:0 auto 5px auto;
  background:#fff;
}
.qr-label{font-size:12px;color:#555;}

/* 谢谢惠顾 胶囊样式 */
.thanks{
  text-align:center;
  font-size:17px;
  color:#6b5a9b;
  padding:5px 12px;
  border:none;
  border-radius:999px;
  margin:4px auto 14px auto;
  width:fit-content;
  background:linear-gradient(90deg,#ffe4f1,#f3e8ff,#e8f0ff);
}
.ctrl-line{margin:8px 0;}
input{padding:6px;}
button{padding:7px 10px;cursor:pointer;}
</style>
</head>
<body>
<div class="tip">💡 使用提示：选择上下渐变颜色，填写订单，上传收款码，点击导出小票图片。全部在本地浏览器运行，不会上传你的图片。</div>
<div class="wrap">
  <div class="panel">
    <h3>小票设置</h3>
    <div class="ctrl-line">
      <label>顶部装饰颜色：<input type="color" id="topColor" value="#e1d5ff"></label>
    </div>
    <div class="ctrl-line">
      <label>底部装饰颜色：<input type="color" id="botColor" value="#e1d5ff"></label>
    </div>
    <hr>
    <h4>订单信息</h4>
    <div>名称：<input id="name" style="width:100%"></div>
    <div>日期：<input id="date" style="width:100%"></div>
    <h4>项目添加</h4>
    <div>项目：<input id="pName"></div>
    <div>单价：<input id="pPrice" type="number"></div>
    <div>数量：<input id="pNum" type="number"></div>
    <button onclick="addRow()">添加项目</button>
    <button onclick="calcTotal()">计算金额</button>
    <hr>
    <h4>收款码上传</h4>
    <div>支付宝码：<input type="file" id="imgAlipay" accept="image/*"></div>
    <div>赞赏码(圆形)：<input type="file" id="imgReward" accept="image/*"></div>
    <div>微信码：<input type="file" id="imgWechat" accept="image/*"></div>
    <button onclick="exportImg()">导出小票图片</button>
  </div>

  <div class="previewBox">
    <div class="receipt" id="receiptDom">
      <div class="decor-top" id="topDecor">
      </div>
      <div class="receipt-content">
        <div class="title">订单内容</div>
        <div class="line"></div>
        <table>
          <colgroup><col><col><col><col></colgroup>
          <thead>
            <tr><th>项目</th><th>单价</th><th>数量</th><th>小计</th></tr>
          </thead>
          <tbody id="goodsBody"></tbody>
        </table>
        <div class="line"></div>
        <div class="sum-area" id="sumBox">
          合计：<span id="total">0</span><br>
          折扣：<span id="discount">无</span><br>
          实付：<span id="pay">0</span>
        </div>
      </div>
      <div class="qr-row">
        <div class="qr-box" id="box_alipay">
          <div class="qr-square" id="t_alipay"></div>
          <div class="qr-label">支付宝</div>
        </div>
        <div class="qr-box" id="box_reward">
          <div class="qr-circle" id="t_reward"></div>
          <div class="qr-label">赞赏码</div>
        </div>
        <div class="qr-box" id="box_wechat">
          <div class="qr-square" id="t_wechat"></div>
          <div class="qr-label">微信</div>
        </div>
      </div>
      <div class="thanks">谢谢惠顾</div>
      <div class="decor-bottom" id="botDecor">
      </div>
    </div>
  </div>
</div>

<script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
<script>
let goodsList = [];
const topDecor = document.getElementById("topDecor");
const botDecor = document.getElementById("botDecor");

// 新颜色算法：只提亮，不加深，不会出现脏深色
function shadeLight(hex, percent){
  hex = hex.replace("#","");
  if(hex.length === 3){
    hex = hex.split("").map(c => c + c).join("");
  }
  let r = parseInt(hex.substring(0,2),16);
  let g = parseInt(hex.substring(2,4),16);
  let b = parseInt(hex.substring(4,6),16);
  // 只向白色提亮，最低不低于原色
  r = Math.min(255, r + (255 - r) * percent);
  g = Math.min(255, g + (255 - g) * percent);
  b = Math.min(255, b + (255 - b) * percent);
  return "#" + ((1 << 24) + (Math.round(r) <<16) + (Math.round(g) <<8) + Math.round(b)).toString(16).slice(1);
}

// 顶栏：原色 → 提亮 → 更亮
document.getElementById("topColor").oninput = function(){
  const c = this.value;
  topDecor.style.background = `linear-gradient(180deg, ${c}, ${shadeLight(c,0.25)}, ${shadeLight(c,0.45)})`;
}
// 底栏：更亮 → 提亮 → 原色
document.getElementById("botColor").oninput = function(){
  const c = this.value;
  botDecor.style.background = `linear-gradient(180deg, ${shadeLight(c,0.45)}, ${shadeLight(c,0.25)}, ${c})`;
}

function addRow(){
  const name = document.getElementById("pName").value;
  const price = Number(document.getElementById("pPrice").value||0);
  const num = Number(document.getElementById("pNum").value||0);
  const sub = price*num;
  goodsList.push({name,price,num,sub});
  renderTable();
}
function renderTable(){
  const tbody = document.getElementById("goodsBody");
  tbody.innerHTML = "";
  goodsList.forEach(item=>{
    const tr = document.createElement("tr");
    tr.innerHTML = `<td>${item.name}</td><td>${item.price}</td><td>${item.num}</td><td>${item.sub}</td>`;
    tbody.appendChild(tr);
  })
}
function calcTotal(){
  let sum = goodsList.reduce((a,b)=>a+b.sub,0);
  document.getElementById("total").innerText = sum;
  document.getElementById("pay").innerText = sum;
}

// 图片上传
document.getElementById("imgAlipay").onchange = function(e){
  const fr = new FileReader();
  fr.onload = ev=>{document.getElementById("t_alipay").innerHTML=`<img src="${ev.target.result}" style="width:100%;height:100%;object-fit:cover;border-radius:6px;">`}
  fr.readAsDataURL(e.target.files[0]);
}
document.getElementById("imgReward").onchange = function(e){
  const fr = new FileReader();
  fr.onload = ev=>{document.getElementById("t_reward").innerHTML=`<img src="${ev.target.result}" style="width:100%;height:100%;object-fit:cover;border-radius:50%;">`}
  fr.readAsDataURL(e.target.files[0]);
}
document.getElementById("imgWechat").onchange = function(e){
  const fr = new FileReader();
  fr.onload = ev=>{document.getElementById("t_wechat").innerHTML=`<img src="${ev.target.result}" style="width:100%;height:100%;object-fit:cover;border-radius:6px;">`}
  fr.readAsDataURL(e.target.files[0]);
}

// 导出图片
function exportImg(){
  const dom = document.getElementById("receiptDom");
  html2canvas(dom,{
    scale:3,
    useCORS:true,
    imageTimeout:20000
  }).then(canvas=>{
    const a = document.createElement("a");
    a.href = canvas.toDataURL("image/png");
    a.download = "小票.png";
    a.click();
  })
}
</script>
</body>
</html>
