# -
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>苹果销售数据助手</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }
    textarea { width: 100%; height: 150px; margin-bottom: 10px; }
    button { padding: 10px 20px; font-size: 16px; }
    pre { background: #f0f0f0; padding: 10px; white-space: pre-wrap; }
  </style>
</head>
<body>
  <h2>输入原始销售记录</h2>
  <textarea id="inputText" placeholder="输入你的第一条销售信息..."></textarea>
  <button onclick="process()">生成第二条信息</button>

  <h2>自动生成的汇总信息</h2>
  <pre id="outputText"></pre>

  <script>
    function process() {
      const input = document.getElementById("inputText").value;
      const lines = input.split('\n').map(l => l.trim()).filter(l => l.length > 0);

      const date = lines[0];
      const phoneModels = [];
      const accessories = {
        '充电头': 0,
        '充电线': 0,
        '充电宝': 0,
        '手表膜': 0,
        '手机膜': 0,
        '手机壳': 0,
        '镜头膜': 0,
        'iPad膜': 0,
        'iPad壳': 0,
        '电脑膜': 0,
        '电脑包': 0,
        '转换线': 0,
        '有线耳机': 0,
        'CNY': 0
      };

      for (let line of lines.slice(1)) {
        const modelMatch = line.match(/^16(pro|max)?[a-z]*\d{0,3}/i);
        if (modelMatch) {
          phoneModels.push(modelMatch[0]);
        }

        // 提取配件关键字
        if (line.includes("充电头")) accessories["充电头"]++;
        if (line.includes("充电线")) accessories["充电线"]++;
        if (line.includes("充电宝")) accessories["充电宝"]++;
        if (line.includes("手机膜")) accessories["手机膜"]++;
        if (line.includes("手机壳")) accessories["手机壳"]++;
        if (line.includes("镜头膜")) accessories["镜头膜"]++;
        if (line.includes("iPad膜")) accessories["iPad膜"]++;
        if (line.includes("iPad壳")) accessories["iPad壳"]++;
        if (line.includes("电脑膜")) accessories["电脑膜"]++;
        if (line.includes("电脑包")) accessories["电脑包"]++;
        if (line.includes("转换线")) accessories["转换线"]++;
        if (line.includes("有线耳机")) accessories["有线耳机"]++;
        if (line.includes("手表膜")) accessories["手表膜"]++;
      }

      const phoneCount = phoneModels.length;
      const accessoriesLines = Object.entries(accessories)
        .filter(([_, v]) => v > 0)
        .map(([k, v]) => `${k}:${v}`).join('\n');

      const output = `${date}唐人街苹果
手机销量:${phoneCount}
明细：
${phoneModels.join('\n')}

手表销量:
明细：
iPad销量: 
明细：
电脑销量: 
明细:
累计销量: 160

融合销量: 
音响: 
键盘: 
鼠标: 
iPad笔: 
AirTag: 
蓝牙耳机:
累计销量: 13

配件销量:${Object.values(accessories).reduce((a,b)=>a+b,0)}
${accessoriesLines}

回收手机:
回收其它机器:
AC+: 
累计销量: 16

售后:
累计销量: 8`;

      document.getElementById("outputText").textContent = output;
    }
  </script>
</body>
</html>
