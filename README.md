<!DOCTYPE html>

<html lang="ko">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>EXERCISE RPG</title>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose/pose.js" crossorigin="anonymous"></script>
<style>
*, *::before, *::after { box-sizing:border-box; margin:0; padding:0; }
body {
  min-height:100vh; background:#08080f; color:#e8e0d0;
  font-family:'Courier New',monospace;
  display:flex; flex-direction:column; align-items:center;
  padding:14px; overflow-x:hidden;
}
body::before {
  content:''; position:fixed; inset:0;
  background-image:
    linear-gradient(rgba(255,200,80,.03) 1px,transparent 1px),
    linear-gradient(90deg,rgba(255,200,80,.03) 1px,transparent 1px);
  background-size:40px 40px; pointer-events:none;
}

/* ── 헤더 ── */
header { text-align:center; margin:10px 0 14px; }
h1 {
font-size:clamp(22px,5vw,44px); font-weight:900; letter-spacing:-2px;
color:#f5c842; text-transform:uppercase;
text-shadow:0 0 40px rgba(245,200,66,.4);
}
.subtitle { font-size:9px; letter-spacing:4px; color:rgba(245,200,66,.35); margin-top:4px; text-transform:uppercase; }

/* ── RPG 레벨/EXP 바 ── */
.rpg-wrap {
width:100%; max-width:720px;
background:#111118; border:1px solid rgba(245,200,66,.2);
border-radius:4px; padding:12px 16px; margin-bottom:12px;
}
.rpg-top { display:flex; justify-content:space-between; align-items:center; margin-bottom:8px; }
.level-badge {
font-size:12px; font-weight:700; letter-spacing:2px;
background:rgba(245,200,66,.12); border:1px solid #f5c842;
color:#f5c842; padding:4px 14px; border-radius:2px;
transition:all .3s;
}
.level-badge.up { background:rgba(245,200,66,.35); box-shadow:0 0 20px rgba(245,200,66,.5); }
.rpg-right { text-align:right; }
.exp-cur { font-size:16px; font-weight:700; color:#f5c842; font-variant-numeric:tabular-nums; }
.exp-need { font-size:10px; color:#555; letter-spacing:1px; }
.exp-bar-bg { height:6px; background:#1a1a28; border-radius:3px; overflow:hidden; margin-bottom:6px; }
.exp-bar-fill {
height:100%; border-radius:3px;
background:linear-gradient(90deg,#f5a822,#f5c842,#ffe066);
transition:width .5s cubic-bezier(.4,0,.2,1);
box-shadow:0 0 10px rgba(245,200,66,.4);
}
.rpg-bottom { display:flex; justify-content:space-between; }
.total-exp-txt { font-size:9px; color:#555; letter-spacing:1px; }

/* ── 운동 탭 ── */
.ex-tabs { display:flex; gap:8px; margin-bottom:12px; justify-content:center; }
.ex-tab {
padding:7px 14px; font-family:inherit; font-size:10px; font-weight:700;
letter-spacing:2px; text-transform:uppercase; background:transparent;
border:1px solid rgba(245,200,66,.2); color:rgba(245,200,66,.35);
border-radius:2px; cursor:pointer; transition:all .2s;
}
.ex-tab.active { background:rgba(245,200,66,.12); border-color:#f5c842; color:#f5c842; }

/* ── 레이아웃 ── */
.layout {
display:flex; gap:12px; align-items:flex-start;
flex-wrap:wrap; justify-content:center;
width:100%; max-width:980px;
}

/* ── 비디오 ── */
.video-panel {
position:relative; border:1px solid rgba(245,200,66,.25);
border-radius:4px; overflow:hidden; background:#111118; flex-shrink:0;
}
#video {
display:block; width:clamp(240px,44vw,500px);
aspect-ratio:4/3; object-fit:cover; transform:scaleX(-1);
}
#canvas {
position:absolute; top:0; left:0;
width:100%; height:100%; transform:scaleX(-1); pointer-events:none;
}
.live-badge {
position:absolute; top:8px; left:8px;
font-size:9px; letter-spacing:2px; padding:3px 8px; border-radius:2px;
text-transform:uppercase; font-weight:700;
background:rgba(200,50,50,.2); color:#dc5050; border:1px solid #dc5050; transition:all .3s;
}
.live-badge.on { background:rgba(50,220,100,.2); color:#50dc64; border-color:#50dc64; }
.ex-label-ov {
position:absolute; top:8px; right:8px;
font-size:9px; letter-spacing:2px; padding:3px 8px; border-radius:2px;
font-weight:700; background:rgba(10,10,20,.7); color:#f5c842; border:1px solid rgba(245,200,66,.3);
}

/* EXP 팝업 */
.exp-popup {
position:absolute; pointer-events:none;
font-size:20px; font-weight:900; color:#f5c842;
text-shadow:0 0 16px rgba(245,200,66,.9);
animation:float-up .9s ease-out forwards; z-index:10;
white-space:nowrap;
}
@keyframes float-up {
0%   { opacity:1; transform:translateY(0) scale(1); }
60%  { opacity:1; transform:translateY(-40px) scale(1.2); }
100% { opacity:0; transform:translateY(-80px) scale(.9); }
}
.levelup-overlay {
position:absolute; inset:0; display:flex; align-items:center; justify-content:center;
background:rgba(245,200,66,.12); pointer-events:none;
animation:levelup-flash .8s ease-out forwards; z-index:9;
}
@keyframes levelup-flash {
0%  { opacity:1; }
100%{ opacity:0; }
}
.levelup-text {
font-size:28px; font-weight:900; color:#f5c842; letter-spacing:4px;
text-shadow:0 0 30px rgba(245,200,66,.9);
animation:levelup-scale .8s ease-out forwards;
}
@keyframes levelup-scale {
0%  { transform:scale(.5); opacity:0; }
50% { transform:scale(1.2); opacity:1; }
100%{ transform:scale(1);   opacity:0; }
}

/* ── 사이드 패널 ── */
.side { display:flex; flex-direction:column; gap:10px; min-width:190px; flex:1; max-width:260px; }
.card {
background:#111118; border:1px solid rgba(245,200,66,.13);
border-radius:4px; padding:14px;
}
.card-label { font-size:9px; letter-spacing:3px; color:rgba(245,200,66,.4); text-transform:uppercase; margin-bottom:7px; }

/* 카운터 */
#count-display {
font-size:clamp(48px,8vw,82px); font-weight:900; color:#f5c842; line-height:1;
letter-spacing:-4px; text-shadow:0 0 50px rgba(245,200,66,.45); transition:transform .1s;
}
#count-display.bump { transform:scale(1.15); }
#phase-badge {
display:inline-block; font-size:10px; font-weight:700; letter-spacing:2px;
padding:4px 10px; border-radius:2px; text-transform:uppercase; margin-top:7px;
background:rgba(200,200,200,.07); color:#777; border:1px solid #2a2a2a; transition:all .2s;
}
#phase-badge.down { background:rgba(245,100,66,.2); color:#f56442; border-color:#f56442; }
#phase-badge.up   { background:rgba(66,200,245,.2);  color:#42c8f5; border-color:#42c8f5; }

/* 콤보 */
.combo-row { display:flex; justify-content:space-between; align-items:center; margin-bottom:4px; }
.combo-num {
font-size:32px; font-weight:900; color:#ff8c42; line-height:1;
letter-spacing:-2px; text-shadow:0 0 20px rgba(255,140,66,.4);
transition:transform .1s;
}
.combo-num.bump { transform:scale(1.2); }
.combo-bonus {
font-size:11px; font-weight:700; color:#ff8c42; letter-spacing:1px;
padding:3px 8px; border-radius:2px;
background:rgba(255,140,66,.1); border:1px solid rgba(255,140,66,.3);
}
.combo-bonus.hidden { opacity:0; }
.combo-milestones { display:flex; gap:6px; margin-top:8px; flex-wrap:wrap; }
.milestone {
font-size:9px; letter-spacing:1px; padding:2px 7px; border-radius:2px;
border:1px solid #222; color:#444; text-transform:uppercase; transition:all .3s;
}
.milestone.reached { border-color:rgba(255,140,66,.5); color:#ff8c42; background:rgba(255,140,66,.08); }

/* 세션 EXP */
.session-list { display:flex; flex-direction:column; gap:5px; }
.session-row { display:flex; justify-content:space-between; align-items:center; }
.session-name { font-size:10px; color:#666; letter-spacing:1px; }
.session-val  { font-size:13px; font-weight:700; color:#f5c842; font-variant-numeric:tabular-nums; }

/* 버튼 */
button.action {
width:100%; padding:10px; background:transparent;
border:1px solid rgba(245,200,66,.35); color:#f5c842;
font-size:9px; letter-spacing:3px; text-transform:uppercase;
font-family:inherit; font-weight:700; cursor:pointer;
border-radius:2px; transition:background .2s; margin-bottom:6px;
}
button.action:hover { background:rgba(245,200,66,.07); }
button.action:disabled { opacity:.35; cursor:not-allowed; }
button.danger { border-color:rgba(255,80,80,.3); color:#f56060; }
button.danger:hover { background:rgba(255,80,80,.07); }

/* 각도 */
.meter-row { display:flex; justify-content:space-between; align-items:center; margin-bottom:3px; }
.meter-label { font-size:9px; color:#666; letter-spacing:1px; }
.meter-val { font-size:14px; font-weight:700; color:#f5c842; font-variant-numeric:tabular-nums; }
.bar { height:3px; background:#1a1a28; border-radius:2px; margin-bottom:9px; overflow:hidden; }
.bar-fill { height:100%; border-radius:2px; transition:width .1s, background .2s; }

#error-msg {
display:none; background:rgba(255,80,80,.1); border:1px solid rgba(255,80,80,.4);
color:#f56060; padding:10px 14px; border-radius:4px;
font-size:10px; letter-spacing:1px; margin-bottom:12px; max-width:560px; text-align:center;
}

@keyframes flash-anim { 0%{opacity:.5} 100%{opacity:0} }
.flash-overlay {
position:absolute; inset:0; background:rgba(245,200,66,.15);
pointer-events:none; animation:flash-anim .4s ease-out forwards;
}
/* 치팅 경고 */
.cheat-warn {
position:absolute; bottom:14px; left:50%; transform:translateX(-50%);
font-size:12px; font-weight:700; letter-spacing:2px;
padding:6px 16px; border-radius:2px; text-transform:uppercase;
background:rgba(255,60,60,.25); color:#ff5050; border:1px solid #ff5050;
animation:flash-anim 1.2s ease-out forwards; pointer-events:none; white-space:nowrap;
}
</style>

</head>
<body>

<header>
  <h1>EXERCISE RPG</h1>
  <p class="subtitle">MediaPipe Pose · EXP · Level · Combo</p>
</header>

<!-- ── RPG EXP 바 ── -->

<div class="rpg-wrap">
  <div class="rpg-top">
    <div class="level-badge" id="level-badge">LV. 1</div>
    <div class="rpg-right">
      <div class="exp-cur"><span id="exp-cur">0</span> EXP</div>
      <div class="exp-need">NEXT: <span id="exp-need">200</span></div>
    </div>
  </div>
  <div class="exp-bar-bg"><div class="exp-bar-fill" id="exp-bar" style="width:0%"></div></div>
  <div class="rpg-bottom">
    <div class="total-exp-txt">TOTAL EXP: <span id="total-exp">0</span></div>
    <div class="total-exp-txt">SESSION: <span id="session-exp-top">+0</span></div>
  </div>
</div>

<div id="error-msg"></div>

<!-- 운동 탭 -->

<div class="ex-tabs">
  <button class="ex-tab active" data-ex="squat">🦵 SQUAT</button>
  <button class="ex-tab" data-ex="pushup">💪 PUSH-UP</button>
  <button class="ex-tab" data-ex="pullup">🏋️ PULL-UP</button>
</div>

<div class="layout">
  <!-- 비디오 -->
  <div class="video-panel" id="video-panel">
    <video id="video" autoplay muted playsinline></video>
    <canvas id="canvas"></canvas>
    <span class="live-badge" id="live-badge">○ OFF</span>
    <span class="ex-label-ov" id="ex-label">SQUAT</span>
  </div>

  <!-- 사이드 -->

  <div class="side">

```
<!-- 카운터 -->
<div class="card">
  <div class="card-label">REP COUNT</div>
  <div id="count-display">00</div>
  <div id="phase-badge">IDLE</div>
</div>

<!-- 콤보 -->
<div class="card">
  <div class="card-label">COMBO</div>
  <div class="combo-row">
    <div class="combo-num" id="combo-num">0</div>
    <div class="combo-bonus hidden" id="combo-bonus">+0 BONUS</div>
  </div>
  <div class="combo-milestones">
    <div class="milestone" id="ms10">×10 +20</div>
    <div class="milestone" id="ms20">×20 +50</div>
    <div class="milestone" id="ms30">×30 +100</div>
  </div>
</div>

<!-- 세션 획득 EXP -->
<div class="card">
  <div class="card-label">SESSION EXP</div>
  <div class="session-list">
    <div class="session-row">
      <span class="session-name">SQUAT</span>
      <span class="session-val" id="s-squat">+0</span>
    </div>
    <div class="session-row">
      <span class="session-name">PUSH-UP</span>
      <span class="session-val" id="s-pushup">+0</span>
    </div>
    <div class="session-row">
      <span class="session-name">PULL-UP</span>
      <span class="session-val" id="s-pullup">+0</span>
    </div>
  </div>
</div>

<!-- 각도 -->
<div class="card">
  <div class="card-label">JOINT ANGLES</div>
  <div class="meter-row"><span class="meter-label">LEFT</span><span class="meter-val" id="m1">—</span></div>
  <div class="bar"><div class="bar-fill" id="b1" style="width:0%"></div></div>
  <div class="meter-row"><span class="meter-label">RIGHT</span><span class="meter-val" id="m2">—</span></div>
  <div class="bar"><div class="bar-fill" id="b2" style="width:0%"></div></div>
</div>

<div>
  <button class="action" id="btn-start">▶ START CAMERA</button>
  <button class="action danger" id="btn-reset">↺ RESET SESSION</button>
</div>
```

  </div>
</div>

<script>
// ═══════════════════════════════════════════════
//  EXP 시스템
// ═══════════════════════════════════════════════
const EXP_PER_REP = { squat:10, pushup:12, pullup:20 };

const COMBO_BONUS = [
  { at:10, bonus:20 },
  { at:20, bonus:50 },
  { at:30, bonus:100 },
];

// 레벨별 필요 EXP 계산
// 1-5: 200, 6-10: 400, 11-20: 600, 21-30: 800, 이후 +200씩
function expToNextLevel(level) {
  if (level <= 5)  return 200;
  if (level <= 10) return 400;
  if (level <= 20) return 600;
  if (level <= 30) return 800;
  // 31레벨 이상: 800 + (구간-3)*200
  const tier = Math.floor((level - 31) / 10) + 4;
  return 800 + tier * 200;
}

// RPG 상태
let totalExp   = 0;
let currentLevel = 1;
let expInLevel = 0;  // 현재 레벨에서 쌓은 EXP
let sessionExp = { squat:0, pushup:0, pullup:0 };
let combo = 0;       // 현재 세션 총 반복 (콤보용)

function addExp(type, repCount) {
  const base = EXP_PER_REP[type];
  let gained = base;

  // 콤보 보너스 체크 (정확히 milestone 도달 시)
  const milestone = COMBO_BONUS.find(m => m.at === combo);
  let bonusExp = 0;
  if (milestone) {
    bonusExp = milestone.bonus;
    gained += bonusExp;
  }

  sessionExp[type] += gained;
  totalExp += gained;
  expInLevel += gained;

  // 레벨업 체크
  let leveledUp = false;
  while (true) {
    const need = expToNextLevel(currentLevel);
    if (expInLevel >= need) {
      expInLevel -= need;
      currentLevel++;
      leveledUp = true;
    } else break;
  }

  updateRPGUI(leveledUp);
  showExpPopup(gained, bonusExp > 0);
  updateSessionUI();
  return gained;
}

// ═══════════════════════════════════════════════
//  RPG UI 업데이트
// ═══════════════════════════════════════════════
function updateRPGUI(leveledUp) {
  const need = expToNextLevel(currentLevel);
  const pct  = Math.min(100, (expInLevel / need) * 100);

  document.getElementById('level-badge').textContent = 'LV. ' + currentLevel;
  document.getElementById('exp-cur').textContent = expInLevel;
  document.getElementById('exp-need').textContent = need;
  document.getElementById('exp-bar').style.width = pct + '%';
  document.getElementById('total-exp').textContent = totalExp;
  document.getElementById('session-exp-top').textContent =
    '+' + (sessionExp.squat + sessionExp.pushup + sessionExp.pullup);

  if (leveledUp) {
    const badge = document.getElementById('level-badge');
    badge.classList.add('up');
    setTimeout(() => badge.classList.remove('up'), 800);
    showLevelUp();
  }
}

function updateSessionUI() {
  document.getElementById('s-squat').textContent  = '+' + sessionExp.squat;
  document.getElementById('s-pushup').textContent = '+' + sessionExp.pushup;
  document.getElementById('s-pullup').textContent = '+' + sessionExp.pullup;
}

function updateComboUI() {
  const el = document.getElementById('combo-num');
  el.textContent = combo;
  el.classList.add('bump');
  setTimeout(() => el.classList.remove('bump'), 150);

  // 마일스톤 하이라이트
  const reached = COMBO_BONUS.filter(m => combo >= m.at);
  document.getElementById('ms10').classList.toggle('reached', combo >= 10);
  document.getElementById('ms20').classList.toggle('reached', combo >= 20);
  document.getElementById('ms30').classList.toggle('reached', combo >= 30);

  // 보너스 배지
  const bonusEl = document.getElementById('combo-bonus');
  const milestone = COMBO_BONUS.find(m => m.at === combo);
  if (milestone) {
    bonusEl.textContent = '+' + milestone.bonus + ' BONUS!';
    bonusEl.classList.remove('hidden');
    setTimeout(() => bonusEl.classList.add('hidden'), 1500);
  }
}

// EXP 팝업 (비디오 위에 떠오름)
function showExpPopup(amount, isBonus) {
  const panel = document.getElementById('video-panel');
  const el = document.createElement('div');
  el.className = 'exp-popup';
  el.textContent = (isBonus ? '★ ' : '') + '+' + amount + ' EXP';
  if (isBonus) el.style.color = '#ff8c42';
  el.style.left = (20 + Math.random() * 60) + '%';
  el.style.bottom = '30%';
  panel.appendChild(el);
  setTimeout(() => el.remove(), 950);
}

function showLevelUp() {
  const panel = document.getElementById('video-panel');
  const wrap = document.createElement('div');
  wrap.className = 'levelup-overlay';
  wrap.innerHTML = '<div class="levelup-text">LEVEL UP!</div>';
  panel.appendChild(wrap);
  setTimeout(() => wrap.remove(), 850);
}

// ═══════════════════════════════════════════════
//  관절 각도 계산
// ═══════════════════════════════════════════════
function calcAngle(a, b, c) {
  const rad = Math.atan2(c.y-b.y, c.x-b.x) - Math.atan2(a.y-b.y, a.x-b.x);
  let deg = Math.abs(rad * 180 / Math.PI);
  if (deg > 180) deg = 360 - deg;
  return deg;
}
const LM = {
  L_SHOULDER:11, R_SHOULDER:12,
  L_ELBOW:13,    R_ELBOW:14,
  L_WRIST:15,    R_WRIST:16,
  L_HIP:23,      R_HIP:24,
  L_KNEE:25,     R_KNEE:26,
  L_ANKLE:27,    R_ANKLE:28,
};

// ═══════════════════════════════════════════════
//  운동별 설정
// ═══════════════════════════════════════════════
const EXERCISE_CONFIG = {
  squat:  { label:'SQUAT',   downThr:100, upThr:160 },
  pushup: { label:'PUSH-UP', downThr:90,  upThr:155 },
  pullup: { label:'PULL-UP', downThr:155, upThr:60  },
};

let currentEx = 'squat';
let repCount  = 0;
let phase     = 'IDLE';
let angleHistory = [];
const SMOOTH_N = 5;
let cameraInstance = null;
let repStartTime = 0;

// ── 치팅 방지용 몸통 위치 추적 ──
// 어깨/엉덩이 y좌표 이동 히스토리 (정규화 0~1)
const BODY_HIST_N = 8;
let shoulderYHistory = [];  // 풀업: 어깨가 올라가야 함
let hipYHistory      = [];  // 푸시업: 엉덩이(몸통)가 내려가야 함

// y좌표 이동평균
function smoothY(history, raw) {
  history.push(raw);
  if (history.length > BODY_HIST_N) history.shift();
  return history.reduce((a,b) => a+b, 0) / history.length;
}

// 히스토리 내 최솟값 ~ 최댓값 차이 (이동량)
function yRange(history) {
  if (history.length < 3) return 0;
  return Math.max(...history) - Math.min(...history);
}

function smoothAngle(raw) {
  angleHistory.push(raw);
  if (angleHistory.length > SMOOTH_N) angleHistory.shift();
  return angleHistory.reduce((a,b) => a+b, 0) / angleHistory.length;
}

function getAvgAngle(lms) {
  const g = i => lms[i];
  if (currentEx === 'squat') {
    const lk = g(LM.L_KNEE), rk = g(LM.R_KNEE);
    if (!lk||!rk||lk.visibility<0.4||rk.visibility<0.4) return null;
    const la = calcAngle(g(LM.L_HIP),lk,g(LM.L_ANKLE));
    const ra = calcAngle(g(LM.R_HIP),rk,g(LM.R_ANKLE));
    updateAngleUI(la, ra);
    return (la+ra)/2;
  } else {
    const le = g(LM.L_ELBOW), re = g(LM.R_ELBOW);
    if (!le||!re||le.visibility<0.3||re.visibility<0.3) return null;
    const la = calcAngle(g(LM.L_SHOULDER),le,g(LM.L_WRIST));
    const ra = calcAngle(g(LM.R_SHOULDER),re,g(LM.R_WRIST));
    updateAngleUI(la, ra);
    return (la+ra)/2;
  }
}

// ═══════════════════════════════════════════════
//  [치팅 방지] 풀업 자세 검증
//
//  조건 1: 손목이 어깨보다 위에 있어야 함
//          (화면 y좌표: 위 = 작은 값)
//          → 만세 자세 차단
//
//  조건 2: 레퍼런스 어깨 y 대비 현재 어깨 y가
//          일정량(0.04 = 화면 높이의 4%) 이상 올라가야 UP 인정
//          → 팔만 굽히는 치팅 차단
// ═══════════════════════════════════════════════
let pullupShoulderRefY = null;  // DOWN 시점의 어깨 y (기준점)

function validatePullup(lms) {
  const g = i => lms[i];
  const ls = g(LM.L_SHOULDER), rs = g(LM.R_SHOULDER);
  const lw = g(LM.L_WRIST),    rw = g(LM.R_WRIST);
  if (!ls||!rs||!lw||!rw) return { wristAbove: false, bodyRise: 0, shoulderY: 0.5 };

  const avgShoulderY = (ls.y + rs.y) / 2;
  const avgWristY    = (lw.y + rw.y) / 2;

  // 조건 1: 손목이 어깨보다 위 (y값이 더 작음)
  const wristAbove = avgWristY < avgShoulderY - 0.05;

  // 조건 2: 기준점 대비 어깨 상승량
  const bodyRise = pullupShoulderRefY !== null
    ? pullupShoulderRefY - avgShoulderY  // 양수 = 올라감
    : 0;

  return { wristAbove, bodyRise, shoulderY: avgShoulderY };
}

// ═══════════════════════════════════════════════
//  [치팅 방지] 푸시업 자세 검증
//
//  조건 1: 손목이 어깨보다 아래에 있어야 함
//          → 만세/서있는 자세 차단
//
//  조건 2: 엉덩이(몸통) y좌표가 레퍼런스 대비
//          일정량(0.03) 이상 내려가야 DOWN 인정
//          → 팔만 굽히는 치팅 차단
//
//  조건 3: 몸통이 수평에 가까워야 함
//          어깨y ≈ 엉덩이y (차이 0.2 이하)
//          → 서서 팔굽혀펴기 차단
// ═══════════════════════════════════════════════
let pushupHipRefY = null;  // UP 시점의 엉덩이 y (기준점)

function validatePushup(lms) {
  const g = i => lms[i];
  const ls = g(LM.L_SHOULDER), rs = g(LM.R_SHOULDER);
  const lw = g(LM.L_WRIST),    rw = g(LM.R_WRIST);
  const lh = g(LM.L_HIP),      rh = g(LM.R_HIP);
  if (!ls||!rs||!lw||!rw||!lh||!rh) return { valid: false, hipY: 0.5, bodyDrop: 0 };

  const avgShoulderY = (ls.y + rs.y) / 2;
  const avgWristY    = (lw.y + rw.y) / 2;
  const avgHipY      = (lh.y + rh.y) / 2;

  // 조건 1: 손목이 어깨보다 아래
  const wristBelow = avgWristY > avgShoulderY + 0.05;

  // 조건 3: 몸통이 수평 (어깨-엉덩이 y차이가 작아야 함)
  const isHorizontal = Math.abs(avgShoulderY - avgHipY) < 0.22;

  // 조건 2: 기준점 대비 엉덩이 하강량
  const bodyDrop = pushupHipRefY !== null
    ? avgHipY - pushupHipRefY  // 양수 = 내려감
    : 0;

  return {
    valid: wristBelow && isHorizontal,
    hipY: avgHipY,
    bodyDrop,
  };
}

// ═══════════════════════════════════════════════
//  스쿼트/푸시업/풀업 감지 (치팅 방지 포함)
// ═══════════════════════════════════════════════
function detectRep(avg, lms) {
  const cfg = EXERCISE_CONFIG[currentEx];
  const s = smoothAngle(avg);

  // ── 풀업 ──
  if (currentEx === 'pullup') {
    const { wristAbove, bodyRise, shoulderY } = validatePullup(lms);

    // 손목이 어깨 위에 없으면 아예 감지 안 함 (만세 차단)
    if (!wristAbove) return;

    if (s > cfg.downThr) {
      // DOWN: 팔 펴진 상태 → 어깨 기준점 저장
      if (phase !== 'DOWN') {
        pullupShoulderRefY = shoulderY;
        phase = 'DOWN';
        updatePhaseUI();
      }
    } else if (s < cfg.upThr) {
      // UP: 팔 굽힌 상태 + 몸통이 실제로 올라왔는지 확인
      // bodyRise > 0.04 = 화면 높이의 4% 이상 어깨가 올라가야 인정
      if (phase === 'DOWN' && bodyRise > 0.04) {
        phase = 'UP';
        updatePhaseUI();
      } else if (phase === 'DOWN') {
        // 팔은 굽혔지만 몸이 안 올라옴 → 치팅 경고
        showCheatWarning();
      }
    } else if (s > cfg.downThr && phase === 'UP') {
      finishRep();
      phase = 'DOWN';
      pullupShoulderRefY = shoulderY;
      updatePhaseUI();
    }

  // ── 푸시업 ──
  } else if (currentEx === 'pushup') {
    const { valid, hipY, bodyDrop } = validatePushup(lms);

    // 자세 자체가 푸시업이 아니면 감지 안 함
    if (!valid) return;

    if (s > cfg.upThr) {
      // UP: 팔 펴진 상태 → 엉덩이 기준점 저장
      if (phase !== 'UP') {
        pushupHipRefY = hipY;
        if (phase === 'DOWN') finishRep();
        phase = 'UP';
        updatePhaseUI();
      }
    } else if (s < cfg.downThr) {
      // DOWN: 팔 굽힌 상태 + 몸통이 실제로 내려왔는지 확인
      // bodyDrop > 0.03 = 화면 높이의 3% 이상 엉덩이가 내려가야 인정
      if (phase === 'UP') {
        if (bodyDrop > 0.03) {
          phase = 'DOWN';
          updatePhaseUI();
        } else {
          showCheatWarning();
        }
      }
    }

  // ── 스쿼트 ──
  } else {
    if (s < cfg.downThr && phase !== 'DOWN') {
      repStartTime = Date.now();
      phase = 'DOWN'; updatePhaseUI();
    } else if (s > cfg.upThr) {
      if (phase === 'DOWN') finishRep();
      if (phase !== 'UP') { phase = 'UP'; updatePhaseUI(); }
    }
  }
}

// 치팅 감지 시 경고 표시
let cheatWarnTimer = null;
function showCheatWarning() {
  const panel = document.getElementById('video-panel');
  // 중복 방지
  if (panel.querySelector('.cheat-warn')) return;
  const el = document.createElement('div');
  el.className = 'cheat-warn';
  el.textContent = '⚠ 자세 불인정';
  panel.appendChild(el);
  clearTimeout(cheatWarnTimer);
  cheatWarnTimer = setTimeout(() => el.remove(), 1200);
}

function finishRep() {
  repCount++;
  combo++;
  updateCountUI();
  updateComboUI();
  triggerFlash();
  addExp(currentEx, repCount);
}

// ═══════════════════════════════════════════════
//  Canvas 골격
// ═══════════════════════════════════════════════
const CONNECTIONS = [
  [11,13],[13,15],[12,14],[14,16],
  [11,12],[11,23],[12,24],[23,24],
  [23,25],[25,27],[24,26],[26,28],
  [27,29],[29,31],[28,30],[30,32],
];
const canvas = document.getElementById('canvas');
const ctx    = canvas.getContext('2d');

function drawPose(lms) {
  ctx.clearRect(0,0,canvas.width,canvas.height);
  if (!lms) return;
  ctx.strokeStyle = 'rgba(245,200,66,.6)'; ctx.lineWidth = 2;
  CONNECTIONS.forEach(([i,j]) => {
    const a=lms[i], b=lms[j];
    if (!a||!b||a.visibility<0.3||b.visibility<0.3) return;
    ctx.beginPath();
    ctx.moveTo(a.x*canvas.width, a.y*canvas.height);
    ctx.lineTo(b.x*canvas.width, b.y*canvas.height);
    ctx.stroke();
  });
  lms.forEach(lm => {
    if (lm.visibility<0.3) return;
    ctx.beginPath();
    ctx.arc(lm.x*canvas.width, lm.y*canvas.height, 4, 0, Math.PI*2);
    ctx.fillStyle='#f5c842'; ctx.fill();
  });
}

// ═══════════════════════════════════════════════
//  UI 헬퍼
// ═══════════════════════════════════════════════
const video      = document.getElementById('video');
const countEl    = document.getElementById('count-display');
const phaseBadge = document.getElementById('phase-badge');
const liveBadge  = document.getElementById('live-badge');
const exLabelEl  = document.getElementById('ex-label');
const videoPanel = document.getElementById('video-panel');

function updateCountUI() {
  countEl.textContent = String(repCount).padStart(2,'0');
  countEl.classList.add('bump');
  setTimeout(() => countEl.classList.remove('bump'), 150);
}
function updatePhaseUI() {
  phaseBadge.textContent = phase;
  phaseBadge.className = '';
  if (phase==='DOWN') phaseBadge.classList.add('down');
  if (phase==='UP')   phaseBadge.classList.add('up');
}
function updateAngleUI(l, r) {
  document.getElementById('m1').textContent = Math.round(l)+'°';
  document.getElementById('m2').textContent = Math.round(r)+'°';
  const pl = Math.max(0,Math.min(100,((l-40)/140)*100));
  const pr = Math.max(0,Math.min(100,((r-40)/140)*100));
  const b1 = document.getElementById('b1'), b2 = document.getElementById('b2');
  b1.style.width = pl+'%'; b1.style.background = barColor(pl);
  b2.style.width = pr+'%'; b2.style.background = barColor(pr);
}
function barColor(p) { return p>65?'#50dc64':p>35?'#f5c842':'#f56442'; }
function triggerFlash() {
  const el = document.createElement('div');
  el.className = 'flash-overlay';
  videoPanel.appendChild(el);
  setTimeout(() => el.remove(), 430);
}

// ═══════════════════════════════════════════════
//  MediaPipe 초기화
// ═══════════════════════════════════════════════
function onResults(results) {
  canvas.width  = video.videoWidth  || 640;
  canvas.height = video.videoHeight || 480;
  drawPose(results.poseLandmarks);
  if (results.poseLandmarks) {
    const avg = getAvgAngle(results.poseLandmarks);
    if (avg !== null) detectRep(avg, results.poseLandmarks);
  }
}

async function startCamera() {
  document.getElementById('error-msg').style.display = 'none';
  const btn = document.getElementById('btn-start');
  btn.disabled = true; btn.textContent = 'LOADING...';
  try {
    const pose = new Pose({
      locateFile: f => `https://cdn.jsdelivr.net/npm/@mediapipe/pose/${f}`
    });
    pose.setOptions({ modelComplexity:1, smoothLandmarks:true, enableSegmentation:false,
      minDetectionConfidence:.6, minTrackingConfidence:.6 });
    pose.onResults(onResults);

    cameraInstance = new Camera(video, {
      onFrame: async () => { await pose.send({ image:video }); },
      width:640, height:480,
    });
    await cameraInstance.start();
    liveBadge.textContent = '● LIVE'; liveBadge.classList.add('on');
    btn.textContent = '■ STOP'; btn.disabled = false;
    btn.onclick = stopCamera;
  } catch(e) {
    const em = document.getElementById('error-msg');
    em.style.display = 'block';
    em.textContent = '⚠ 카메라 오류: ' + e.message;
    btn.textContent = '▶ START CAMERA'; btn.disabled = false;
  }
}

function stopCamera() {
  if (cameraInstance) { cameraInstance.stop(); cameraInstance = null; }
  ctx.clearRect(0,0,canvas.width,canvas.height);
  liveBadge.textContent = '○ OFF'; liveBadge.classList.remove('on');
  const btn = document.getElementById('btn-start');
  btn.textContent = '▶ START CAMERA'; btn.onclick = startCamera;
  phase = 'IDLE'; updatePhaseUI(); angleHistory = [];
  document.getElementById('m1').textContent = document.getElementById('m2').textContent = '—';
}

// ═══════════════════════════════════════════════
//  운동 전환
// ═══════════════════════════════════════════════
function switchEx(ex) {
  currentEx = ex; repCount = 0; phase = 'IDLE';
  angleHistory = []; shoulderYHistory = []; hipYHistory = [];
  pullupShoulderRefY = null; pushupHipRefY = null;
  updateCountUI(); updatePhaseUI();
  exLabelEl.textContent = EXERCISE_CONFIG[ex].label;
  document.querySelectorAll('.ex-tab').forEach(t =>
    t.classList.toggle('active', t.dataset.ex === ex));
}

// ═══════════════════════════════════════════════
//  이벤트 바인딩
// ═══════════════════════════════════════════════
document.querySelectorAll('.ex-tab').forEach(t => { t.onclick = () => switchEx(t.dataset.ex); });
document.getElementById('btn-start').onclick = startCamera;
document.getElementById('btn-reset').onclick = () => {
  repCount=0; phase='IDLE'; combo=0;
  sessionExp={squat:0,pushup:0,pullup:0};
  angleHistory=[];
  updateCountUI(); updatePhaseUI(); updateSessionUI();
  document.getElementById('combo-num').textContent='0';
  document.getElementById('combo-bonus').classList.add('hidden');
  ['ms10','ms20','ms30'].forEach(id => document.getElementById(id).classList.remove('reached'));
};

// 초기 렌더
updateRPGUI(false);
updateSessionUI();
</script>

</body>
</html>
