//7.1 00:21 更正
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>苹果销售助手</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }
    textarea { width: 100%; height: 200px; margin-bottom: 10px; }
    button { padding: 10px 20px; font-size: 16px; }
    pre { background: #f0f0f0; padding: 10px; white-space: pre-wrap; }
  </style>
</head>
<body>
  <h2>输入第一条销售记录</h2>
  <textarea id="inputText" placeholder="请输入销售原始记录...（含日期）"></textarea>
  <button onclick="process()">生成汇总</button>
  <h2>输出汇总信息</h2>
  <pre id="outputText"></pre>

  <script>
    function process() {
      const input = document.getElementById("inputText").value.trim();
      const lines = input.split('\n').map(line => line.trim()).filter(line => line);

      // 1. 取第一行作为日期（格式如：6.30）
      const date = lines[0];

      // 2. 定义各类统计容器
      const phones = [], watches = [], ipads = [], computers = [];
      const fusion = { '音响': 0, '键盘': 0, '鼠标': 0, 'iPad笔': 0, 'AirTag': 0, '蓝牙耳机': 0 };
      const accessories = {
        '充电头': 0, '充电线': 0, '充电宝': 0, '手表膜': 0, '手机膜': 0, '手机壳': 0,
        '镜头膜': 0, 'iPad膜': 0, 'iPad壳': 0, '电脑膜': 0, '电脑包': 0,
        '转换线': 0, '有线耳机': 0, 'CNY': 0
      };
      const recyclePhones = [], recycleOthers = [], services = [];

      // 3. 关键词正则
      const phoneRegex = /^(16(?:pro|max|promax)?\S*\d{3,4})/i; // 手机识别
      const watchRegex = /(s\d{1,2}|watch)/i; // 手表识别
      const ipadRegex = /ipad/i; // iPad识别
      const macRegex = /macbook/i; // Mac识别

      let skipNext = false; // 跳过标记：用于回收/售后场景中跳过附属信息行

      // 4. 遍历每行，按类型识别
      for (let i = 1; i < lines.length; i++) {
        const line = lines[i];
        const lower = line.toLowerCase();

        if (skipNext) {
          skipNext = false; // 当前行跳过处理，恢复标志
          continue;
        }

        const isRecycle = line.includes("以旧换新") || line.includes("仅回收");
        const isService = line.includes("售后") || line.includes("保内") || line.includes("保外");
        const next = lines[i + 1] || '';

        // 回收识别（包括下一行机器信息）
        if (isRecycle) {
          if (next.toLowerCase().includes("ipad")) recycleOthers.push(next);
          else recyclePhones.push(next);
          skipNext = true;
          continue;
        }

        // 售后识别（包括下一行机器信息）
        if (isService) {
          services.push(line + (next ? `\n${next}` : ''));
          skipNext = true;
          continue;
        }

        // 销售识别逻辑（只识别未被 skip 掉的销售部分）
        if (phoneRegex.test(line)) {
          phones.push(line.match(phoneRegex)[0]);
        } else if (watchRegex.test(lower)) {
          watches.push(line);
        } else if (ipadRegex.test(lower)) {
          ipads.push(line);
        } else if (macRegex.test(lower)) {
          computers.push(line);
        }

        // 融合商品识别
        if (line.includes("音响")) fusion['音响']++;
        if (line.includes("键盘")) fusion['键盘']++;
        if (line.includes("鼠标")) fusion['鼠标']++;
        if (lower.includes("pencil")) fusion['iPad笔']++;
        if (lower.includes("airtag")) fusion['AirTag']++;
        if (lower.includes("airpods")) fusion['蓝牙耳机']++;

        // 配件识别（关键词映射）
        if (line.includes("充电头")) accessories['充电头']++;
        if (line.includes("充电线")) accessories['充电线']++;
        if (line.includes("充电宝") || line.includes("大麦")) accessories['充电宝']++;
        if (line.includes("有线耳机")) accessories['有线耳机']++;
      }

      // 5. 输出格式拼接函数
      function arrText(arr) { return arr.length ? arr.join('\n') : ''; }

      // 6. 构造输出内容
      let output = `${date}唐人街苹果\n`;
      output += `手机销量: ${phones.length}\n明细：${arrText(phones)}\n`;
      output += `手表销量: ${watches.length}\n明细：${arrText(watches)}\n`;
      output += `iPad销量: ${ipads.length}\n明细：${arrText(ipads)}\n`;
      output += `电脑销量: ${computers.length}\n明细: ${arrText(computers)}\n`;

      // 融合类
      const fusionTotal = Object.values(fusion).reduce((a,b) => a+b, 0);
      output += `累计销量: \n\n融合销量: ${fusionTotal}\n`;
      for (const k in fusion) {
        if (fusion[k] > 0) output += `${k}: ${fusion[k]}\n`;
      }
      output += `累计销量: \n\n`;

      // 配件类
      const accessoryTotal = Object.values(accessories).reduce((a,b)=>a+b, 0);
      output += `配件销量:${accessoryTotal}\n`;
      for (const k in accessories) {
        if (accessories[k] > 0) output += `${k}: ${accessories[k]}\n`;
      }
      output += `CNY: \n累计销量: \n\n`;

      // 回收类
      output += `回收手机: ${arrText(recyclePhones)}\n`;
      output += `回收其它机器: ${arrText(recycleOthers)}\n`;
      output += `AC+: \n累计销量: \n\n`;

      // 售后类
      output += `售后: ${arrText(services)}\n累计销量:`;

      // 7. 展示
      document.getElementById("outputText").textContent = output;
    }
  </script>
</body>
</html>
