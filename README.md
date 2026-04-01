<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>DLS26 后卫练卡推演 (唯抢断胜率版)</title>
    <style>
        :root {
            --bg-main: #ffffff;
            --bg-alt: #f8f9fa;
            --text-main: #111111;
            --text-muted: #666666;
            --border-light: #e0e0e0;
            --accent-target: #2e7d32;
            --accent-warn: #d32f2f;
            --accent-gold: #b8860b;
            --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, sans-serif;
            --font-mono: "SF Mono", Menlo, Consolas, monospace;
        }

        body { font-family: var(--font-sans); background-color: var(--bg-main); color: var(--text-main); margin: 0; padding: 40px 20px; display: flex; justify-content: center; line-height: 1.5; }
        .container { max-width: 650px; width: 100%; display: flex; flex-direction: column; gap: 30px; }

        .header { text-align: center; border-bottom: 2px solid var(--text-main); padding-bottom: 15px; }
        .header h1 { font-size: 22px; font-weight: 700; letter-spacing: 1px; margin: 0; }

        .matrix-wrapper { background-color: var(--bg-alt); border: 1px solid var(--border-light); border-radius: 8px; padding: 25px 20px; }
        .data-matrix { display: grid; grid-template-columns: 70px repeat(4, 1fr); gap: 20px 10px; align-items: center; }
        @media (max-width: 500px) { .data-matrix { grid-template-columns: 55px repeat(4, 1fr); gap: 20px 5px; } }

        .col-header { text-align: center; font-size: 13px; font-weight: 600; color: var(--text-main); padding-bottom: 10px; border-bottom: 1px solid var(--border-light); }
        .col-header.target { color: var(--accent-target); }

        .row-label { font-size: 12px; font-weight: 600; color: var(--text-muted); text-align: right; padding-right: 10px; line-height: 1.2; }

        input[type=number] { width: 100%; background: transparent; border: none; border-bottom: 1px solid var(--border-light); color: var(--text-main); text-align: center; font-family: var(--font-mono); border-radius: 0; padding: 5px 0; transition: all 0.2s; -moz-appearance: textfield; }
        input[type=number]::-webkit-inner-spin-button, input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }
        input[type=number]:focus { outline: none; border-bottom: 2px solid var(--text-main); }

        .input-curr { font-size: 24px; font-weight: 700; color: var(--text-main); border-bottom: 2px solid var(--border-light); }
        .input-curr:focus { border-bottom-color: var(--text-main); }
        .input-sub { font-size: 15px; color: var(--text-muted); }

        .dashboard { display: flex; justify-content: center; border-top: 1px solid var(--border-light); border-bottom: 1px solid var(--border-light); padding: 20px 0; }
        .dash-item { display: flex; flex-direction: column; align-items: center; text-align: center; }
        .dash-label { font-size: 12px; font-weight: 600; color: var(--text-muted); margin-bottom: 8px; }
        .dash-val { font-family: var(--font-mono); font-size: 36px; font-weight: 700; line-height: 1; }

        .btn-execute { width: 100%; background-color: var(--text-main); color: var(--bg-main); border: none; padding: 16px; font-size: 15px; font-weight: 600; letter-spacing: 1px; cursor: pointer; border-radius: 6px; transition: 0.2s; }
        .btn-execute:hover { background-color: #333333; }

        .result-container { display: none; flex-direction: column; gap: 20px; animation: fadeIn 0.4s ease; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .hero-rec { background-color: #fafafa; border: 1px solid var(--border-light); border-left: 4px solid var(--text-main); padding: 20px; border-radius: 4px; }
        .hero-title { font-size: 16px; font-weight: 700; margin-bottom: 10px; display: flex; align-items: center; gap: 8px; color: var(--text-main); }
        .hero-desc { font-size: 14px; line-height: 1.6; color: #444; margin-bottom: 15px; }

        .status-badge { display: inline-block; padding: 6px 12px; border-radius: 4px; font-size: 13px; font-weight: 600; margin-bottom: 15px; }
        .status-neutral { background: #f0f0f0; color: #555; border: 1px solid #ddd; }
        .status-good { background: #e8f5e9; color: #2e7d32; border: 1px solid #c8e6c9; }
        .status-mid { background: #e6f7ff; color: #0077b6; border: 1px solid #bae0ff; }
        .status-warn { background: #fff8e1; color: #f57f17; border: 1px solid #ffecb3; }
        .status-bad { background: #ffebee; color: #c62828; border: 1px solid #ffcdd2; }
        
        .score-highlight { font-family: var(--font-mono); font-size: 15px; font-weight: 800; margin-right: 6px; }

        .trend-table-wrapper { margin-top: 15px; border-top: 1px solid var(--border-light); padding-top: 15px; }
        .trend-table-title { font-size: 13px; font-weight: 600; color: var(--text-muted); margin-bottom: 10px; }
        .trend-table { width: 100%; border-collapse: collapse; text-align: center; font-size: 13px; }
        .trend-table th, .trend-table td { border: 1px solid var(--border-light); padding: 10px 5px; }
        .trend-table th { background-color: #f8f9fa; color: #555; font-weight: 600; width: 20%; }
        .trend-table td { font-family: var(--font-mono); font-weight: 600; color: var(--text-main); }

    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <p style="margin: 0 0 5px 0; font-size: 13px; color: var(--text-main); font-weight: bold;">微信公众号：DLS26China</p>
        <h1>后卫练卡推演</h1>
    </div>

    <div class="matrix-wrapper">
        <div id="data-matrix" class="data-matrix">
            <div></div>
            <div class="col-header">控制</div>
            <div class="col-header">传球</div>
            <div class="col-header">射门</div>
            <div class="col-header target">抢断 <small>(目标)</small></div>

            <div class="row-label">当前<br>进度</div>
            <input type="number" class="input-curr" id="curr_ctrl" placeholder="-" oninput="updateDashboard()">
            <input type="number" class="input-curr" id="curr_pass" placeholder="-" oninput="updateDashboard()">
            <input type="number" class="input-curr" id="curr_shoot" placeholder="-" oninput="updateDashboard()">
            <input type="number" class="input-curr" id="curr_tkl" placeholder="-" oninput="updateDashboard()">

            <div class="row-label">满级<br>模板</div>
            <input type="number" class="input-sub" id="target_ctrl" placeholder="-" oninput="updateDashboard()">
            <input type="number" class="input-sub" id="target_pass" placeholder="-" oninput="updateDashboard()">
            <input type="number" class="input-sub" id="target_shoot" placeholder="-" oninput="updateDashboard()">
            <input type="number" class="input-sub" id="target_tkl" placeholder="-" value="100" readonly style="color:var(--accent-target); font-weight:bold;">
        </div>
    </div>

    <div class="dashboard">
        <div class="dash-item">
            <span class="dash-label">剩余安全容量 (点)</span>
            <span id="remain-pool" class="dash-val" style="color: var(--accent-target);">0</span>
        </div>
    </div>

    <button id="btn-sdp" class="btn-execute" onclick="runSDP()">开始模拟</button>
    
    <div id="result" class="result-container"></div>
</div>

<script>
    const techStats = [
        { id: 'ctrl', name: '控制', weight: 2 },
        { id: 'pass', name: '传球', weight: 1 },
        { id: 'shoot', name: '射门', weight: 1 },
        { id: 'tkl', name: '抢断', weight: 1, isTarget: true }
    ];

    let simulationCount = 0;

    const textPersonas = [
        { 
            gold: "既然只搏 100 满分，就不留退路了。用金教练强杀废属性，只要这把能洗掉杂质，后面才有直通 100 的可能。别心疼底子，干就完了。",
            blue: "这回合用蓝教练。金教练爆仓风险太大，直接葬送满分希望；白教练太拖节奏。蓝教练是唯一能让你保住那一丝 100 分概率的独木桥。",
            white: "容量已经极度危险。现在任何高级教练都会直接引爆平分惩罚，彻底断送 100 分的可能。切白教练，死抠这最后的胜率！",
            luck99: "盘面极佳：这张卡就是为 100 抢断生的。只要你不手贱乱点，稳稳的满分防卫者。",
            luck97: "走势良好：目前保留了冲击 100 分的优良火种。按策略执行，我们就是在买最大的赢面。",
            luck95: "容错极低：因为只搏 100 分，你现在的处境非常被动。接下来的每一步都必须精确到小数点，没有退路。",
            luck0:  "濒临死局：老实说，这卡冲击 100 的希望极其渺茫了。但既然要赌命，我们就用策略去抢那哪怕 1% 的奇迹。"
        },
        { 
            gold: "听我的，要 100 满分就得敢下重注。金教练直接上，要么死要么神，把废属性全部顶爆，给抢断腾路！",
            blue: "上蓝教练。纯赌狗玩法也得讲究基本法，蓝教练能让废属性快速退场，还不至于一把把家底全败光，这是搏 100 的最佳跳板。",
            white: "没筹码了兄弟，赶紧切白教练。你的点数已经不允许任何激进了，用白底慢慢磨，强行吊住这口 100 分的气。",
            luck99: "完美开局：系统发牌员已经在对你笑了，这把不出 100 我倒立洗头。",
            luck97: "稳中向好：目前冲击 100 的希望非常大。别浪，严守纪律。",
            luck95: "赌徒狂欢：这卡的 100 分概率已经被你挥霍得差不多了，接下来就是真正的极限微操，生死有命。",
            luck0:  "命悬一线：这卡基本算是废了，满分希望微乎其微。但只要概率不是 0，我们就继续按最优解去赌。"
        },
        { 
            gold: "最高优先级指令：传奇教练。为了达到 100 满分，我们需要激进地清除概率池。这是数学上唯一能拉高极限胜率的动作。",
            blue: "中期关键手：稀有教练。在淘汰废属性和保留点数之间寻找唯一的生机。这是通往 100 分的最优贝叶斯路径。",
            white: "系统红线！严禁使用高级教练。任何溢出消耗都会将达到 100 的概率直接归零。强制执行普通教练单抽。",
            luck99: "绝对优势：冲刺 100 的路线图极其清晰。保持执行力。",
            luck97: "均值博弈：目前属于正常发育，达成 100 分的概率依然掌握在策略手中。",
            luck95: "高危预警：冗余点数消耗过度。冲击 100 的数学概率正在断崖式下跌，必须执行极端止损。",
            luck0:  "战术灾难：前期的无效消耗已经摧毁了基盘。现在我们只是在数学废墟里寻找奇迹。"
        },
        { 
            gold: "既然立下了 100 分的军令状，开局就得下猛药。金教练安排上，把那些边角料属性赶紧喂饱，别让它们后期出来作妖。",
            blue: "现在的局势很微妙，蓝教练是最好的破局点。温水煮青蛙，一点点剥离杂质，把 100 分的火种护住。",
            white: "停下！别冲动。你的点数如果再用大教练，连 98 都保不住，更别提 100 了。现在只能靠白教练一刀一刀去抠那个奇迹。",
            luck99: "极品天胡：这张卡底子太棒了。只要咱们不犯错，100 抢断就是你的囊中之物。",
            luck97: "发育优良：冲击 100 的概率非常可观。稳住心态，别因为一次歪属性就乱了阵脚。",
            luck95: "步步惊心：说实话，这卡想冲 100 已经有点吃力了。但咱们既然定了目标，就咬牙按推荐走，拼一把。",
            luck0:  "绝地求生：这卡之前的走势确实糟糕，100 分的希望很渺茫。但练卡就是不到最后一刻不放弃，咱们继续干！"
        }
    ];

    function getVal(id) { return parseInt(document.getElementById(id).value) || 0; }

    window.updateDashboard = function() {
        let remain = 0;
        let isReady = true;

        techStats.forEach(stat => {
            const target = getVal(`target_${stat.id}`);
            let curr = getVal(`curr_${stat.id}`);
            if (target === 0 || curr === 0) isReady = false;
            
            remain += (target - curr) * stat.weight;
            
            const inputCurr = document.getElementById(`curr_${stat.id}`);
            if (curr >= 100) {
                inputCurr.style.color = "var(--accent-target)";
                inputCurr.style.borderColor = "var(--accent-target)";
            } else {
                inputCurr.style.color = "var(--text-main)";
                inputCurr.style.borderColor = "var(--border-light)";
            }
        });

        const remainEl = document.getElementById('remain-pool');
        remainEl.innerText = remain;
        
        if (remain <= 0 && isReady) {
            remainEl.style.color = "var(--accent-warn)";
            document.getElementById('btn-sdp').innerText = "潜能已满 (容量耗尽)";
        } else {
            remainEl.style.color = remain >= 12 ? "var(--accent-target)" : (remain >= 6 ? "var(--accent-gold)" : "var(--accent-warn)");
            document.getElementById('btn-sdp').innerText = "开始模拟";
        }
    }

    function getCombinations(arr, k) {
        if (k >= arr.length) return [arr]; 
        let result = [];
        function run(level, start, currentCombo) {
            if (level === k) { result.push(currentCombo); return; }
            for (let i = start; i < arr.length; i++) { run(level + 1, i + 1, currentCombo.concat([arr[i]])); }
        }
        run(0, 0, []);
        return result;
    }

    const memo = new Map();

    // 【算法核心重构】：不再计算平均期望，只计算达到 100 的概率
    function dp_prob(cap, ctrl, pass, shoot, tkl) {
        if (tkl >= 100) return 1.0;
        if (cap <= 0) return 0.0;
        const key = `${cap}_${ctrl}_${pass}_${shoot}_${tkl}`;
        if (memo.has(key)) return memo.get(key);

        let pool = [];
        if (ctrl < 100) pool.push({id: 'ctrl', w: 2});
        if (pass < 100) pool.push({id: 'pass', w: 1});
        if (shoot < 100) pool.push({id: 'shoot', w: 1});
        if (tkl < 100) pool.push({id: 'tkl', w: 1});
        if (pool.length === 0) return tkl >= 100 ? 1.0 : 0.0;

        let maxProb = 0;
        for (let action of [1, 2, 3]) {
            let numPicks = Math.min(action, pool.length);
            let combos = getCombinations(pool, numPicks);
            let probForAction = 0;

            for (let combo of combos) {
                let cost = 0, hasTkl = false;
                for (let s of combo) { cost += s.w * action; if (s.id === 'tkl') hasTkl = true; }

                if (cost > cap) {
                    let pointsPerItem = cap / combo.length;
                    let gainedTkl = hasTkl ? pointsPerItem : 0;
                    probForAction += (tkl + gainedTkl >= 100) ? 1.0 : 0.0;
                } else {
                    let n_ctrl = Math.min(100, ctrl + (combo.some(s=>s.id==='ctrl') ? action : 0));
                    let n_pass = Math.min(100, pass + (combo.some(s=>s.id==='pass') ? action : 0));
                    let n_shoot = Math.min(100, shoot + (combo.some(s=>s.id==='shoot') ? action : 0));
                    let n_tkl = Math.min(100, tkl + (combo.some(s=>s.id==='tkl') ? action : 0));
                    probForAction += dp_prob(cap - cost, n_ctrl, n_pass, n_shoot, n_tkl);
                }
            }
            probForAction /= combos.length; 
            if (probForAction > maxProb) maxProb = probForAction;
        }
        memo.set(key, maxProb);
        return maxProb;
    }

    function evaluateRootActions(cap, ctrl, pass, shoot, tkl) {
        memo.clear(); 
        let pool = [];
        if (ctrl < 100) pool.push({id: 'ctrl', w: 2});
        if (pass < 100) pool.push({id: 'pass', w: 1});
        if (shoot < 100) pool.push({id: 'shoot', w: 1});
        if (tkl < 100) pool.push({id: 'tkl', w: 1});

        let results = [];
        for (let action of [1, 2, 3]) {
            let numPicks = Math.min(action, pool.length);
            let combos = getCombinations(pool, numPicks);
            let probForAction = 0;

            for (let combo of combos) {
                let cost = 0, hasTkl = false;
                for (let s of combo) { cost += s.w * action; if (s.id === 'tkl') hasTkl = true; }

                if (cost > cap) {
                    let pointsPerItem = cap / combo.length;
                    let gainedTkl = hasTkl ? pointsPerItem : 0;
                    probForAction += (tkl + gainedTkl >= 100) ? 1.0 : 0.0;
                } else {
                    let n_ctrl = Math.min(100, ctrl + (combo.some(s=>s.id==='ctrl') ? action : 0));
                    let n_pass = Math.min(100, pass + (combo.some(s=>s.id==='pass') ? action : 0));
                    let n_shoot = Math.min(100, shoot + (combo.some(s=>s.id==='shoot') ? action : 0));
                    let n_tkl = Math.min(100, tkl + (combo.some(s=>s.id==='tkl') ? action : 0));
                    probForAction += dp_prob(cap - cost, n_ctrl, n_pass, n_shoot, n_tkl);
                }
            }
            probForAction /= combos.length;
            results.push({ action: action, expectedProb: probForAction });
        }
        return results;
    }

    function runSDP() {
        updateDashboard(); 
        const cap = parseInt(document.getElementById('remain-pool').innerText);
        const resultDiv = document.getElementById('result');

        let allValid = true;
        techStats.forEach(stat => {
            if (getVal(`curr_${stat.id}`) === 0 || getVal(`target_${stat.id}`) === 0) allValid = false;
        });

        if (!allValid) { alert("请把当前的进度和满级模板填满，不然算不出准确容量。"); return; }
        if (getVal('curr_tkl') >= 100) { alert("抢断已经练满 100 了，无需再测。"); return; }
        if (cap <= 0) { alert("安全点数不够了，没法继续推演了。"); return; }

        simulationCount++;

        const ctrl = getVal('curr_ctrl');
        const pass = getVal('curr_pass');
        const shoot = getVal('curr_shoot');
        const tkl = getVal('curr_tkl');

        const sdpResults = evaluateRootActions(cap, ctrl, pass, shoot, tkl);

        let results = [
            { id: 1, name: '普通教练 (白)', val: sdpResults.find(r=>r.action===1).expectedProb },
            { id: 2, name: '稀有教练 (蓝)', val: sdpResults.find(r=>r.action===2).expectedProb },
            { id: 3, name: '传奇教练 (金)', val: sdpResults.find(r=>r.action===3).expectedProb }
        ];

        // 唯抢断100概率排序
        results.sort((a, b) => b.val - a.val);
        let best = results[0];

        const personaIdx = Math.floor(Math.random() * textPersonas.length);
        const p = textPersonas[personaIdx];

        let heroText = "";
        if (best.id === 3) heroText = p.gold;
        else if (best.id === 1) heroText = p.white;
        else heroText = p.blue;

        // 计算 1-10 运气打分（现在基于 100 分的胜率）
        let maxProb = best.val;
        let score = 1;
        if (maxProb >= 0.8) score = 10;
        else if (maxProb >= 0.6) score = 9;
        else if (maxProb >= 0.4) score = 8;
        else if (maxProb >= 0.2) score = 7;
        else if (maxProb >= 0.1) score = 6;
        else if (maxProb >= 0.05) score = 5;
        else if (maxProb >= 0.02) score = 4;
        else if (maxProb >= 0.01) score = 3;
        else if (maxProb > 0) score = 2;
        else score = 1;

        let luckHtml = "";
        if (simulationCount === 1) {
            luckHtml = `<div class="status-badge status-neutral">📌 初始盘面已确认。按下方策略执行你的第一手训练。</div>`;
        } else {
            let scoreStr = `<span class="score-highlight">[运气评分: ${score}/10]</span>`;
            if (maxProb >= 0.5) {
                luckHtml = `<div class="status-badge status-good">🟢 ${scoreStr} ${p.luck99}</div>`;
            } else if (maxProb >= 0.2) {
                luckHtml = `<div class="status-badge status-mid">🔵 ${scoreStr} ${p.luck97}</div>`;
            } else if (maxProb >= 0.05) {
                luckHtml = `<div class="status-badge status-warn">🟠 ${scoreStr} ${p.luck95}</div>`;
            } else {
                luckHtml = `<div class="status-badge status-bad">🔴 ${scoreStr} ${p.luck0}</div>`;
            }
        }

        // 修改表格：显示满分胜率的理论走势（假设未来也做最优选择）
        let stepCap = Math.max(1, Math.floor(cap / 4));
        let capStages = [cap, Math.max(0, cap - stepCap), Math.max(0, cap - 2*stepCap), Math.max(0, cap - 3*stepCap), 0];
        capStages = [...new Set(capStages)].sort((a,b) => b-a);
        
        let tableHeaders = "";
        let tableData = "";
        
        capStages.forEach((c, index) => {
            let label = index === 0 ? "当前" : (c === 0 ? "满级极限" : `剩 ${c} 点`);
            tableHeaders += `<th>${label}</th>`;
            
            // 用当前最高胜率展示，模拟如果一切顺利的存活率演变
            let progressRatio = (cap - c) / cap;
            // 越往后，如果要赢，胜率必然向 100% 收敛
            let projectedProb = maxProb + (1.0 - maxProb) * Math.pow(progressRatio, 2);
            tableData += `<td>${(projectedProb * 100).toFixed(1)}%</td>`;
        });

        let html = `
            <div class="hero-rec">
                ${luckHtml}
                <div class="hero-title">📌 最终决策：这把死磕【${best.name}】</div>
                <div class="hero-desc">${heroText}</div>
                
                <div style="font-size:14px; margin: 15px 0; font-family:var(--font-mono); font-weight:bold; color:var(--accent-target);">
                    🎯 满分(100)极限胜率：${(maxProb * 100).toFixed(2)}%
                </div>

                <div class="trend-table-wrapper">
                    <div class="trend-table-title">不成功便成仁：满分胜率走势预期</div>
                    <table class="trend-table">
                        <tr>${tableHeaders}</tr>
                        <tr>${tableData}</tr>
                    </table>
                </div>
            </div>
        `;

        resultDiv.style.display = 'flex';
        resultDiv.innerHTML = html;
    }

    updateDashboard();
</script>

</body>
</html>
