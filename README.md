<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <title>Y&J 训练助手 Ultra Pro</title>
    <style>
        :root {
            --primary: #007aff;
            --y-color: #007aff;
            --j-color: #af52de;
            --success: #34c759;
            --bg-body: #f2f2f7;
            --bg-card: #ffffff;
            --text-main: #1c1c1e;
            --text-sub: #8e8e93;
            --border: #e5e5ea;
            --shadow: 0 4px 12px rgba(0,0,0,0.08);
        }

        @media (prefers-color-scheme: dark) {
            :root {
                --bg-body: #000000;
                --bg-card: #1c1c1e;
                --text-main: #ffffff;
                --border: #38383a;
            }
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; outline: none; }
        body {
            margin: 0; padding: 0;
            font-family: -apple-system, system-ui, sans-serif;
            background: var(--bg-body); color: var(--text-main);
            height: 100vh; display: flex; flex-direction: column; overflow: hidden;
        }

        /* 顶部状态 */
        .top-bar {
            background: var(--bg-card); padding: 10px 16px;
            border-bottom: 1px solid var(--border); z-index: 100;
        }
        .header-row { display: flex; justify-content: space-between; align-items: center; }
        .date-picker { border: none; background: transparent; font-size: 17px; font-weight: 700; color: var(--text-main); }
        
        /* 进度条 */
        .progress-container { height: 6px; background: var(--border); border-radius: 3px; margin-top: 10px; overflow: hidden; }
        .progress-fill { height: 100%; background: linear-gradient(90deg, var(--y-color), var(--j-color)); width: 0%; transition: width 0.5s ease; }

        /* 视图容器 */
        .view-container { flex: 1; overflow-y: auto; padding: 12px 12px 120px 12px; display: none; }
        .view-container.active { display: block; }

        /* 计划选择横轴 */
        .plan-scroll { display: flex; gap: 8px; overflow-x: auto; padding: 4px 0 12px 0; scrollbar-width: none; }
        .plan-scroll::-webkit-scrollbar { display: none; }
        .plan-chip {
            padding: 8px 16px; background: var(--bg-card); border-radius: 12px;
            font-size: 13px; font-weight: 600; color: var(--text-sub);
            white-space: nowrap; border: 1px solid var(--border); transition: all 0.2s;
        }
        .plan-chip.active { background: var(--text-main); color: #fff; border-color: var(--text-main); transform: scale(1.05); }

        /* 动作卡片强化 */
        .exercise-card {
            background: var(--bg-card); border-radius: 18px; margin-bottom: 12px;
            box-shadow: var(--shadow); border: 2px solid transparent; transition: all 0.3s;
        }
        .exercise-card.active-now { border-color: var(--primary); box-shadow: 0 0 15px rgba(0,122,255,0.2); }
        .exercise-card.done { opacity: 0.8; }
        
        .ex-header { padding: 16px; display: flex; justify-content: space-between; align-items: center; }
        .ex-title { font-size: 17px; font-weight: 800; }
        .ex-info { font-size: 12px; color: var(--text-sub); margin-top: 2px; }

        /* 组数行布局 */
        .sets-area { padding: 0 12px 12px 12px; }
        .set-row {
            background: rgba(0,0,0,0.02); border-radius: 12px; padding: 8px;
            margin-bottom: 8px; display: grid; grid-template-columns: 30px 1fr 1fr; gap: 8px; align-items: center;
        }
        .set-num { font-size: 14px; font-weight: 800; color: var(--text-sub); text-align: center; }

        /* 输入控件组 */
        .user-box { display: flex; flex-direction: column; gap: 4px; }
        .input-stepper { display: flex; align-items: center; background: var(--bg-card); border-radius: 8px; border: 1px solid var(--border); overflow: hidden; }
        .step-btn {
            width: 30px; height: 34px; border: none; background: transparent;
            font-size: 18px; font-weight: 600; color: var(--primary); active { background: #eee; }
        }
        .val-display { flex: 1; text-align: center; font-size: 14px; font-weight: 700; border: none; width: 100%; padding: 0; background: transparent; }
        
        .y-theme .step-btn { color: var(--y-color); }
        .j-theme .step-btn { color: var(--j-color); }
        .y-theme { border-left: 3px solid var(--y-color); padding-left: 6px; }
        .j-theme { border-left: 3px solid var(--j-color); padding-left: 6px; }

        .quick-actions { display: flex; gap: 4px; margin-top: 4px; }
        .btn-mini { 
            flex: 1; font-size: 10px; padding: 4px 0; border-radius: 4px; border: 1px solid var(--border);
            background: var(--bg-card); color: var(--text-sub);
        }

        /* 差值显示 */
        .diff-tag { font-size: 10px; text-align: center; color: var(--text-sub); margin-top: 2px; }

        /* 全局计时器 */
        #rest-timer {
            position: fixed; bottom: 85px; left: 50%; transform: translateX(-50%) translateY(150px);
            background: var(--text-main); color: #fff; padding: 12px 24px;
            border-radius: 50px; display: flex; align-items: center; gap: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3); transition: 0.5s cubic-bezier(0.17, 0.89, 0.32, 1.28); z-index: 2000;
        }
        #rest-timer.active { transform: translateX(-50%) translateY(0); }
        .timer-val { font-size: 24px; font-weight: 800; font-family: monospace; }

        /* 底部导航 */
        .tab-bar {
            position: fixed; bottom: 0; width: 100%; background: rgba(255,255,255,0.9);
            backdrop-filter: blur(15px); border-top: 1px solid var(--border);
            display: flex; padding-bottom: env(safe-area-inset-bottom);
        }
        .tab-item { flex: 1; padding: 12px 0; display: flex; flex-direction: column; align-items: center; font-size: 10px; color: var(--text-sub); }
        .tab-item.active { color: var(--primary); }
        .tab-icon { font-size: 20px; margin-bottom: 2px; }

        .toast { position: fixed; top: 20%; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.8); color: white; padding: 8px 20px; border-radius: 20px; font-size: 13px; opacity: 0; transition: 0.3s; z-index: 3000; }
        .toast.show { opacity: 1; }
    </style>
</head>
<body>

    <div class="top-bar">
        <div class="header-row">
            <input type="date" id="date-input" class="date-picker">
            <div id="wake-status" style="font-size: 11px; color: var(--success);">● 屏幕常亮</div>
        </div>
        <div class="progress-container"><div class="progress-fill" id="daily-progress"></div></div>
    </div>

    <!-- 训练视图 -->
    <div id="view-workout" class="view-container active">
        <div class="plan-scroll" id="plan-selector"></div>
        <div id="exercise-list"></div>
    </div>

    <!-- 历史视图 -->
    <div id="view-history" class="view-container">
        <h3>📅 训练记录</h3>
        <div id="history-list"></div>
    </div>

    <!-- 饮食视图 -->
    <div id="view-diet" class="view-container">
        <h3>🥗 营养补给</h3>
        <div id="diet-content"></div>
    </div>

    <!-- 计时器浮窗 -->
    <div id="rest-timer">
        <span style="font-size: 20px;">☕</span>
        <span class="timer-val" id="timer-display">00:00</span>
        <div style="display: flex; gap: 8px;">
            <button class="btn-mini" onclick="adjustTimer(30)" style="background:rgba(255,255,255,0.2); color:white; border:none; padding: 5px 10px;">+30s</button>
            <button class="btn-mini" onclick="closeTimer()" style="background:rgba(255,255,255,0.2); color:white; border:none; padding: 5px 10px;">跳过</button>
        </div>
    </div>

    <!-- 底部导航 -->
    <div class="tab-bar">
        <div class="tab-item active" onclick="switchTab('workout')">
            <div class="tab-icon">💪</div><div>训练</div>
        </div>
        <div class="tab-item" onclick="switchTab('history')">
            <div class="tab-icon">📊</div><div>数据</div>
        </div>
        <div class="tab-item" onclick="switchTab('diet')">
            <div class="tab-icon">🍚</div><div>饮食</div>
        </div>
    </div>

    <div id="toast" class="toast"></div>

<script>
// ==================== 配置与数据 ====================
const PLANS = {
    pushA: { name: "推A(上胸)", rest: 90, exercises: [{n:"上斜卧推", s:4, t:"6-10", rest:120}, {n:"哑铃卧推", s:4, t:"8-12"}, {n:"侧平举", s:4, t:"12-15", rest:60}, {n:"三头下压", s:3, t:"12-15"}] },
    pullA: { name: "拉A(宽度)", rest: 90, exercises: [{n:"高位下拉", s:4, t:"8-12"}, {n:"窄距下拉", s:4, t:"10-12"}, {n:"直臂下压", s:3, t:"12-15"}, {n:"弯举", s:4, t:"12"}] },
    legA: { name: "腿A(股四)", rest: 120, exercises: [{n:"杠铃深蹲", s:4, t:"8-10", rest:180}, {n:"腿屈伸", s:4, t:"12-15"}, {n:"RDL", s:4, t:"10-12"}] },
    // 可继续添加其他计划...
};

let state = {
    date: new Date().toISOString().split('T')[0],
    planKey: 'pushA',
    data: JSON.parse(localStorage.getItem('yj_fitness_ultra_v1') || '{}'),
    timer: { interval: null, remaining: 0 }
};

// ==================== 初始化 ====================
function init() {
    document.getElementById('date-input').value = state.date;
    document.getElementById('date-input').onchange = (e) => { state.date = e.target.value; renderWorkout(); };
    
    // 自动屏幕锁定
    if ('wakeLock' in navigator) {
        navigator.wakeLock.request('screen').catch(()=>{});
    }

    renderPlanSelector();
    renderWorkout();
}

// ==================== 渲染核心 ====================
function renderPlanSelector() {
    const container = document.getElementById('plan-selector');
    container.innerHTML = Object.keys(PLANS).map(k => `
        <div class="plan-chip ${k === state.planKey ? 'active' : ''}" onclick="switchPlan('${k}')">
            ${PLANS[k].name}
        </div>
    `).join('');
}

function switchPlan(k) { state.planKey = k; renderPlanSelector(); renderWorkout(); }

function renderWorkout() {
    const list = document.getElementById('exercise-list');
    const plan = PLANS[state.planKey];
    
    list.innerHTML = plan.exercises.map((ex, idx) => {
        const isCurrent = checkIsCurrent(ex.n);
        const isDone = checkIsDone(ex.n, ex.s);
        
        return `
        <div class="exercise-card ${isCurrent ? 'active-now' : ''} ${isDone ? 'done' : ''}" id="ex-${idx}">
            <div class="ex-header" onclick="toggleExpand(${idx})">
                <div>
                    <div class="ex-title">${ex.n}</div>
                    <div class="ex-info">${ex.s}组 · 目标 ${ex.t} · ☕${ex.rest || plan.rest}s</div>
                </div>
                <div style="color:${isDone ? 'var(--success)' : 'var(--border)'}">●</div>
            </div>
            <div class="sets-area">
                ${renderSets(ex)}
            </div>
        </div>`;
    }).join('');
    updateProgress();
}

function renderSets(ex) {
    let html = '';
    for(let i=1; i<=ex.s; i++) {
        const dY = getRecord(ex.n, i, 'Y');
        const dJ = getRecord(ex.n, i, 'J');
        const diff = (dY.w && dJ.w) ? (dJ.w - dY.w) : null;
        
        html += `
        <div class="set-row">
            <div class="set-num">${i}</div>
            <!-- Y 的输入 -->
            <div class="user-box y-theme">
                ${renderStepper(ex.n, i, 'Y', 'w', dY.w, 'kg')}
                ${renderStepper(ex.n, i, 'Y', 'r', dY.r, '次')}
                <div class="quick-actions">
                    <button class="btn-mini" onclick="copyPrev('${ex.n}', ${i}, 'Y')">同上</button>
                </div>
            </div>
            <!-- J 的输入 -->
            <div class="user-box j-theme">
                ${renderStepper(ex.n, i, 'J', 'w', dJ.w, 'kg')}
                ${renderStepper(ex.n, i, 'J', 'r', dJ.r, '次')}
                <div class="quick-actions">
                    <button class="btn-mini" onclick="copyPrev('${ex.n}', ${i}, 'J')">同上</button>
                </div>
                ${diff !== null ? `<div class="diff-tag">${diff >=0 ? '需加' : '需减'} ${Math.abs(diff)}kg</div>` : ''}
            </div>
        </div>`;
    }
    return html;
}

function renderStepper(ex, set, user, field, val, unit) {
    const step = field === 'w' ? 2.5 : 1;
    return `
    <div class="input-stepper">
        <button class="step-btn" onclick="changeVal('${ex}',${set},'${user}','${field}',-${step})">−</button>
        <input type="number" class="val-display" value="${val || ''}" placeholder="${unit}" 
               onchange="updateData('${ex}',${set},'${user}','${field}',this.value)">
        <button class="step-btn" onclick="changeVal('${ex}',${set},'${user}','${field}',${step})">+</button>
    </div>`;
}

// ==================== 操作逻辑 ====================

function changeVal(ex, set, user, field, delta) {
    let current = parseFloat(getRecord(ex, set, user)[field] || 0);
    if (field === 'r' && current === 0 && delta > 0) current = 8; // 次数默认从8开始
    if (field === 'w' && current === 0 && delta > 0) current = 20; // 重量默认从20开始
    
    const newVal = Math.max(0, current + delta);
    updateData(ex, set, user, field, newVal);
    renderWorkout();
}

function updateData(ex, set, user, field, val) {
    if(!state.data[state.date]) state.data[state.date] = {};
    if(!state.data[state.date][state.planKey]) state.data[state.date][state.planKey] = {};
    const key = `${ex}_s${set}`;
    if(!state.data[state.date][state.planKey][key]) state.data[state.date][state.planKey][key] = {};
    if(!state.data[state.date][state.planKey][key][user]) state.data[state.date][state.planKey][key][user] = {};
    
    state.data[state.date][state.planKey][key][user][field] = parseFloat(val);
    saveData();
    
    // 检查是否两人都完成此组 -> 触发计时器
    checkAndStartTimer(ex, set);
}

function copyPrev(ex, set, user) {
    if (set > 1) {
        const prev = getRecord(ex, set - 1, user);
        updateData(ex, set, user, 'w', prev.w);
        updateData(ex, set, user, 'r', prev.r);
    } else {
        // 如果是第一组，尝试寻找历史最后一次
        const last = findHistoryLast(ex, user);
        if (last) {
            updateData(ex, set, user, 'w', last.w);
            updateData(ex, set, user, 'r', last.r);
            showToast("已同步上次训练数据");
        }
    }
    renderWorkout();
}

function checkAndStartTimer(ex, set) {
    const row = state.data[state.date][state.planKey][`${ex}_s${set}`];
    if (row.Y?.r && row.J?.r) {
        const exConfig = PLANS[state.planKey].exercises.find(e => e.n === ex);
        const restTime = exConfig.rest || PLANS[state.planKey].rest;
        startRestTimer(restTime);
    }
}

// ==================== 计时器逻辑 ====================
function startRestTimer(sec) {
    clearInterval(state.timer.interval);
    state.timer.remaining = sec;
    document.getElementById('rest-timer').classList.add('active');
    
    state.timer.interval = setInterval(() => {
        state.timer.remaining--;
        const m = Math.floor(state.timer.remaining / 60);
        const s = state.timer.remaining % 60;
        document.getElementById('timer-display').textContent = `${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
        
        if(state.timer.remaining <= 0) {
            clearInterval(state.timer.interval);
            if(window.navigator.vibrate) window.navigator.vibrate([300, 100, 300]);
            showToast("休息结束！开始下一组");
            setTimeout(closeTimer, 2000);
        }
    }, 1000);
}

function adjustTimer(s) { state.timer.remaining += s; }
function closeTimer() { 
    document.getElementById('rest-timer').classList.remove('active'); 
    clearInterval(state.timer.interval);
}

// ==================== 工具函数 ====================
function getRecord(ex, set, user) {
    return state.data[state.date]?.[state.planKey]?.[`${ex}_s${set}`]?.[user] || {};
}

function saveData() {
    localStorage.setItem('yj_fitness_ultra_v1', JSON.stringify(state.data));
    updateProgress();
}

function updateProgress() {
    const plan = PLANS[state.planKey];
    let total = 0, done = 0;
    plan.exercises.forEach(ex => {
        for(let i=1; i<=ex.s; i++) {
            total++;
            if(getRecord(ex.n, i, 'Y').r || getRecord(ex.n, i, 'J').r) done++;
        }
    });
    document.getElementById('daily-progress').style.width = `${(done/total)*100}%`;
}

function checkIsCurrent(exName) {
    const plan = state.data[state.date]?.[state.planKey];
    if (!plan) return false;
    // 如果该动作有记录但还没满，或者前一个动作刚满，它就是current (简化逻辑)
    return false; 
}

function checkIsDone(exName, totalSets) {
    const plan = state.data[state.date]?.[state.planKey];
    if (!plan) return false;
    let setsDone = 0;
    for(let i=1; i<=totalSets; i++) {
        if(plan[`${exName}_s${i}`]?.Y?.r || plan[`${exName}_s${i}`]?.J?.r) setsDone++;
    }
    return setsDone === totalSets;
}

function findHistoryLast(ex, user) {
    const dates = Object.keys(state.data).sort().reverse();
    for(let d of dates) {
        if(d === state.date) continue;
        for(let pk in state.data[d]) {
            const rec = state.data[d][pk][`${ex}_s1`]?.[user]; // 拿第一组参考
            if(rec && rec.w) return rec;
        }
    }
    return null;
}

function showToast(msg) {
    const t = document.getElementById('toast');
    t.textContent = msg; t.classList.add('show');
    setTimeout(() => t.classList.remove('show'), 2000);
}

function switchTab(t) {
    document.querySelectorAll('.view-container, .tab-item').forEach(el => el.classList.remove('active'));
    document.getElementById(`view-${t}`).classList.add('active');
    event.currentTarget.classList.add('active');
}

init();
</script>
</body>
</html>
