<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>苹果销售助手</title>
  <style>
    /* 背景和容器样式 */
    body {
      font-family: 'Helvetica Neue', sans-serif;
      background: linear-gradient(to bottom right, #dfefff, #f6faff);
      padding: 40px;
      margin: 0;
    }

    .container {
      max-width: 900px;
      margin: 0 auto;
      background: rgba(255, 255, 255, 0.35);
      backdrop-filter: blur(12px);
      border-radius: 20px;
      box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
      padding: 30px;
      border: 1px solid rgba(255, 255, 255, 0.3);
      display: none; /* 初始隐藏，在密码验证后显示 */
    }

    h2 {
      font-weight: 600;
      color: #336699;
      margin-bottom: 10px;
    }

    textarea {
      width: 100%;
      height: 200px;
      padding: 12px;
      border: none;
      border-radius: 12px;
      background: rgba(255, 255, 255, 0.6);
      backdrop-filter: blur(6px);
      font-size: 15px;
      color: #333;
      margin-bottom: 20px;
    }

    button {
      padding: 12px 24px;
      font-size: 16px;
      background: #79aee5;
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
    }

    pre {
      background: rgba(255, 255, 255, 0.5);
      backdrop-filter: blur(6px);
      padding: 15px;
      border-radius: 12px;
      font-size: 14px;
      white-space: pre-wrap;
      line-height: 1.6;
      color: #222;
    }

    /* 登录弹窗 */
    #authOverlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(240, 248, 255, 0.9);
      backdrop-filter: blur(10px);
      z-index: 9999;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
    }

    #authOverlay input {
      padding: 10px;
      font-size: 16px;
      border-radius: 8px;
      border: 1px solid #ccc;
      margin-bottom: 10px;
      width: 200px;
      text-align: center;
    }

    #authOverlay button {
      background: #5a9ddb;
      color: white;
      padding: 8px 18px;
      border-radius: 8px;
      border: none;
      font-size: 15px;
      cursor: pointer;
    }

    #authOverlay button:hover {
      background: #407fc0;
    }

    #authOverlay #authError {
      color: #d8000c;
      margin-top: 10px;
      display: none;
      font-size: 14px;
    }
  </style>
</head>
<body>

<!-- 密码输入弹窗 -->
<div id="authOverlay">
  <div style="font-size:20px; margin-bottom:10px;">🔐 请输入密码以访问系统</div>
  <input id="authInput" type="password" placeholder="密码..." onkeydown="if(event.key==='Enter'){checkPassword()}">
  <button onclick="checkPassword()">进入</button>
  <div id="authError">密码错误，请重试</div>
</div>

<!-- 主功能界面 -->
<div class="container" id="mainApp">
  <h2>输入第一条销售记录</h2>
  <textarea id="inputText" placeholder="请输入销售原始记录...（含日期）"></textarea>
  <button onclick="process()">生成汇总</button>
  <h2>输出汇总信息</h2>
  <pre id="outputText"></pre>
</div>

<!-- 脚本部分 -->
<script>
function checkPassword() {
  const input = document.getElementById("authInput").value;
  const overlay = document.getElementById("authOverlay");
  const error = document.getElementById("authError");
  const mainApp = document.getElementById("mainApp");

  if (input === "1234") {
    overlay.style.display = "none";     // 隐藏密码弹窗
    mainApp.style.display = "block";    // 显示主界面
  } else {
    error.style.display = "block";      // 显示错误提示
  }
}

function process() {
  const input = document.getElementById("inputText").value.trim();
  const lines = input.split('\n').map(l => l.trim()).filter(Boolean);
  const date = lines[0];

  const phoneRegex = /^(16(?:pro|max|promax)?\S*\d{3,4})/i;
  const watchRegex = /(s\d{1,2}|watch)/i;
  const ipadRegex = /ipad/i;
  const macRegex = /macbook/i;

  const fusion = { '音响': 0, '键盘': 0, '鼠标': 0, 'iPad笔': 0, 'AirTag': 0, '蓝牙耳机': 0 };
  const accessories = {
    '充电头': 0, '充电线': 0, '充电宝': 0,
    '手表膜': 0, '手机膜': 0, '手机壳': 0,
    '镜头膜': 0, 'iPad膜': 0, 'iPad壳': 0,
    '电脑膜': 0, '电脑包': 0, '转换线': 0,
    '有线耳机': 0, 'CNY': 0
  };

  const phones = [], watches = [], ipads = [], computers = [];
  const recyclePhones = [], recycleOthers = [], services = [];

  let skip = false;
  for (let i = 1; i < lines.length; i++) {
    const line = lines[i];
    const lower = line.toLowerCase();
    const next = lines[i + 1] || "";
    const isRecycle = line.includes("以旧换新") || line.includes("仅回收");
    const isService = line.includes("售后") || line.includes("保内") || line.includes("保外");

    if (skip) { skip = false; continue; }

    if (isRecycle) {
      if (next.toLowerCase().includes("ipad")) recycleOthers.push(next);
      else recyclePhones.push(next);
      skip = true;
      continue;
    }
    if (isService) {
      services.push(line + (next ? `\n${next}` : ""));
      skip = true;
      continue;
    }

    if (phoneRegex.test(line)) phones.push(line.match(phoneRegex)[0]);
    else if (watchRegex.test(lower)) watches.push(line);
    else if (ipadRegex.test(lower)) ipads.push(line);
    else if (macRegex.test(lower)) computers.push(line);

    if (line.includes("音响")) fusion['音响']++;
    if (line.includes("键盘")) fusion['键盘']++;
    if (line.includes("鼠标")) fusion['鼠标']++;
    if (lower.includes("pencil")) fusion['iPad笔']++;
    if (lower.includes("airtag")) fusion['AirTag']++;
    if (lower.includes("airpods")) fusion['蓝牙耳机']++;

    if (line.includes("充电头")|| line.includes("充电器")|| line.includes("mophie")) accessories['充电头']++;
    if (line.includes("充电线") || line.includes("数据线")) accessories['充电线']++;
    if (line.includes("充电宝") || line.includes("大麦")) accessories['充电宝']++;
    if (line.includes("手机壳")) accessories['手机壳']++;
    if (line.includes("有线耳机")) accessories['有线耳机']++;
  }

  function writeList(title, items) {
    return items.length ? `${title}: ${items.length}\n明细：\n${items.join('\n')}\n` : `${title}:\n明细：\n`;
  }

  let output = `${date}唐人街苹果\n`;
  output += writeList("手机销量", phones);
  output += writeList("手表销量", watches);
  output += writeList("iPad销量", ipads);
  output += writeList("电脑销量", computers);
  output += `累计销量: \n\n`;

  const fusionTotal = Object.values(fusion).reduce((a,b) => a+b, 0);
  output += `融合销量: ${fusionTotal ? fusionTotal : ''}\n`;
  for (const k in fusion) output += `${k}: ${fusion[k] ? fusion[k] : ''}\n`;
  output += `累计销量: \n\n`;

  const accessoriesTotal = Object.values(accessories).reduce((a,b)=>a+b, 0);
  output += `配件销量:${accessoriesTotal ? accessoriesTotal : ''}\n`;
  for (const k in accessories) output += `${k}: ${accessories[k] ? accessories[k] : ''}\n`;
  output += `累计销量: \n\n`;

  output += `回收手机:${recyclePhones.length ? '\n' + recyclePhones.join('\n') : ''}\n`;
  output += `回收其它机器:${recycleOthers.length ? '\n' + recycleOthers.join('\n') : ''}\n`;
  output += `AC+: \n累计销量: \n\n`;
  output += `售后:${services.length ? '\n' + services.join('\n\n') : ''}\n累计销量:`;

  document.getElementById("outputText").textContent = output;
}
</script>

</body>
</html>
