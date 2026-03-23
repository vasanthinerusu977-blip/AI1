<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Bias Metrics Dashboard</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500&family=DM+Mono:wght@400;500&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0f0f11;
    --bg2: #16161a;
    --bg3: #1e1e24;
    --border: rgba(255,255,255,0.08);
    --border2: rgba(255,255,255,0.14);
    --text: #e8e6e0;
    --text2: #8a8880;
    --text3: #5a5856;
    --blue: #3d8ef0;
    --blue-dim: rgba(61,142,240,0.12);
    --red: #e05555;
    --red-dim: rgba(224,85,85,0.12);
    --amber: #e09840;
    --amber-dim: rgba(224,152,64,0.12);
    --green: #4caf7d;
    --green-dim: rgba(76,175,125,0.12);
    --purple: #9b7fe8;
    --radius: 10px;
    --radius-sm: 6px;
  }

  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    padding: 0;
  }

  .topbar {
    background: var(--bg2);
    border-bottom: 1px solid var(--border);
    padding: 14px 32px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: sticky;
    top: 0;
    z-index: 100;
  }
  .topbar-left { display: flex; align-items: center; gap: 14px; }
  .logo-mark {
    width: 32px; height: 32px;
    background: var(--blue);
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 14px; font-weight: 500; color: #fff;
  }
  .topbar-title { font-size: 15px; font-weight: 500; color: var(--text); letter-spacing: -0.01em; }
  .topbar-sub { font-size: 12px; color: var(--text3); font-family: 'DM Mono', monospace; }
  .topbar-right { display: flex; gap: 10px; align-items: center; }

  .status-pill {
    font-size: 11px; font-family: 'DM Mono', monospace;
    padding: 4px 10px; border-radius: 99px;
    border: 1px solid var(--border2);
    color: var(--text2);
    display: flex; align-items: center; gap: 5px;
  }
  .status-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--green); animation: pulse 2s infinite; }
  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.4} }

  .main { padding: 28px 32px; max-width: 1280px; margin: 0 auto; }

  .controls-bar {
    display: flex; gap: 12px; align-items: center; flex-wrap: wrap;
    margin-bottom: 24px;
    padding: 14px 18px;
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: var(--radius);
  }
  .ctrl-group { display: flex; align-items: center; gap: 8px; }
  .ctrl-label { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--text3); text-transform: uppercase; letter-spacing: 0.05em; }
  select {
    background: var(--bg3);
    border: 1px solid var(--border2);
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    font-size: 13px;
    padding: 7px 12px;
    border-radius: var(--radius-sm);
    cursor: pointer;
    outline: none;
    appearance: none;
    -webkit-appearance: none;
    padding-right: 28px;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='6' viewBox='0 0 10 6'%3E%3Cpath d='M1 1l4 4 4-4' stroke='%235a5856' stroke-width='1.5' fill='none' stroke-linecap='round'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 10px center;
    transition: border-color 0.15s;
  }
  select:hover { border-color: rgba(255,255,255,0.25); }
  select:focus { border-color: var(--blue); }

  .divider { width: 1px; height: 24px; background: var(--border); margin: 0 4px; }

  .kpi-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-bottom: 20px; }
  .kpi-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 16px 18px;
    position: relative;
    overflow: hidden;
    transition: border-color 0.2s;
  }
  .kpi-card:hover { border-color: var(--border2); }
  .kpi-card::before {
    content: '';
    position: absolute; top: 0; left: 0; right: 0; height: 2px;
    background: var(--accent, var(--blue));
    opacity: 0.6;
  }
  .kpi-label { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--text3); text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 8px; }
  .kpi-value { font-size: 28px; font-weight: 300; letter-spacing: -0.02em; line-height: 1; margin-bottom: 4px; }
  .kpi-sub { font-size: 11px; color: var(--text3); }

  .alert-row { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-bottom: 20px; }
  .alert-card {
    border-radius: var(--radius);
    padding: 12px 14px;
    border: 1px solid;
    display: flex; gap: 10px; align-items: flex-start;
  }
  .alert-card.red { background: var(--red-dim); border-color: rgba(224,85,85,0.25); }
  .alert-card.amber { background: var(--amber-dim); border-color: rgba(224,152,64,0.25); }
  .alert-card.green { background: var(--green-dim); border-color: rgba(76,175,125,0.25); }
  .alert-icon { width: 20px; height: 20px; border-radius: 50%; display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-top: 1px; font-size: 10px; font-weight: 500; }
  .alert-card.red .alert-icon { background: var(--red); color: #fff; }
  .alert-card.amber .alert-icon { background: var(--amber); color: #fff; }
  .alert-card.green .alert-icon { background: var(--green); color: #fff; }
  .alert-title { font-size: 12px; font-weight: 500; margin-bottom: 2px; }
  .alert-card.red .alert-title { color: var(--red); }
  .alert-card.amber .alert-title { color: var(--amber); }
  .alert-card.green .alert-title { color: var(--green); }
  .alert-desc { font-size: 11px; color: var(--text2); }

  .chart-row { display: grid; grid-template-columns: 1.3fr 1fr; gap: 14px; margin-bottom: 16px; }
  .bottom-row { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }
  .chart-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 18px 20px;
  }
  .chart-title { font-size: 12px; font-family: 'DM Mono', monospace; color: var(--text3); text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 12px; }

  .legend { display: flex; flex-wrap: wrap; gap: 12px; margin-bottom: 10px; }
  .legend-item { display: flex; align-items: center; gap: 5px; font-size: 12px; color: var(--text2); }
  .leg-dot { width: 8px; height: 8px; border-radius: 2px; flex-shrink: 0; }

  .chart-wrap { position: relative; width: 100%; height: 200px; }
  .chart-wrap-sm { position: relative; width: 100%; height: 220px; }

  .action-btn {
    margin-top: 14px;
    display: inline-flex; align-items: center; gap: 6px;
    font-size: 12px; font-family: 'DM Mono', monospace;
    color: var(--blue);
    background: var(--blue-dim);
    border: 1px solid rgba(61,142,240,0.2);
    border-radius: var(--radius-sm);
    padding: 7px 12px;
    cursor: pointer;
    text-decoration: none;
    transition: background 0.15s, border-color 0.15s;
  }
  .action-btn:hover { background: rgba(61,142,240,0.2); border-color: rgba(61,142,240,0.35); }
  .action-btn svg { width: 12px; height: 12px; }

  @media (max-width: 900px) {
    .kpi-grid { grid-template-columns: repeat(2,1fr); }
    .chart-row, .bottom-row { grid-template-columns: 1fr; }
    .alert-row { grid-template-columns: 1fr; }
    .main { padding: 16px; }
    .topbar { padding: 12px 16px; }
  }
  @media (max-width: 480px) {
    .kpi-grid { grid-template-columns: 1fr 1fr; }
    .controls-bar { flex-direction: column; align-items: flex-start; }
    .divider { display: none; }
  }

  .fade-in { animation: fadeIn 0.4s ease forwards; }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(4px); } to { opacity: 1; transform: translateY(0); } }
</style>
</head>
<body>

<div class="topbar">
  <div class="topbar-left">
    <div class="logo-mark">AI</div>
    <div>
      <div class="topbar-title">Bias Metrics Dashboard</div>
      <div class="topbar-sub">Fairness Audit System v1.0</div>
    </div>
  </div>
  <div class="topbar-right">
    <div class="status-pill"><span class="status-dot"></span>Live audit</div>
  </div>
</div>

<div class="main">
  <div class="controls-bar">
    <div class="ctrl-group">
      <span class="ctrl-label">Domain</span>
      <select id="domain-sel" onchange="updateDomain()">
        <option value="hiring">Hiring — resume screening</option>
        <option value="credit">Finance — credit scoring</option>
        <option value="health">Healthcare — treatment eligibility</option>
        <option value="justice">Criminal justice — risk scoring</option>
      </select>
    </div>
    <div class="divider"></div>
    <div class="ctrl-group">
      <span class="ctrl-label">Mitigation</span>
      <select id="mit-sel" onchange="updateDomain()">
        <option value="none">None applied</option>
        <option value="resampling">Resampling</option>
        <option value="threshold">Threshold adjustment</option>
        <option value="adversarial">Adversarial debiasing</option>
      </select>
    </div>
  </div>

  <div class="kpi-grid" id="kpi-grid"></div>
  <div class="alert-row" id="alert-row"></div>

  <div class="chart-row">
    <div class="chart-card">
      <div class="chart-title">Accuracy by demographic group</div>
      <div class="legend" id="acc-legend"></div>
      <div class="chart-wrap"><canvas id="accChart"></canvas></div>
    </div>
    <div class="chart-card">
      <div class="chart-title">False positive rate by group</div>
      <div class="legend" id="fpr-legend"></div>
      <div class="chart-wrap"><canvas id="fprChart"></canvas></div>
    </div>
  </div>

  <div class="bottom-row">
    <div class="chart-card">
      <div class="chart-title">Fairness metric scorecard</div>
      <div class="chart-wrap-sm"><canvas id="radarChart"></canvas></div>
    </div>
    <div class="chart-card">
      <div class="chart-title">Bias gap trend — last 6 audits</div>
      <div class="chart-wrap-sm"><canvas id="trendChart"></canvas></div>
    </div>
  </div>
</div>

<script>
const DOMAINS = {
  hiring: {
    groups: ['Men','Women','Non-binary'],
    colors: ['#3d8ef0','#9b7fe8','#e09840'],
    acc: { none:[96,71,68], resampling:[94,86,84], threshold:[92,89,87], adversarial:[93,91,90] },
    fpr: { none:[4,19,22], resampling:[6,12,11], threshold:[5,8,8], adversarial:[5,6,7] },
    radar: { none:[38,52,70,45,30], resampling:[62,70,74,68,55], threshold:[70,72,78,74,60], adversarial:[80,82,84,79,74] },
    trend: { none:[25,27,24,26,25,24], resampling:[24,20,16,12,10,8], threshold:[22,18,14,10,8,5], adversarial:[20,15,10,7,5,3] },
    alerts: {
      none:   [{t:'red',title:'Gender gap: 28 pts',desc:'Women 71% vs men 96% accuracy'},{t:'amber',title:'FPR disparity: 18 pts',desc:'Women rejected 4× more falsely'},{t:'red',title:'Calibration: failed',desc:'Scores miscalibrated for women'}],
      resampling:[{t:'amber',title:'Gender gap: 8 pts',desc:'Gap narrowed from 28 to 8 pts'},{t:'amber',title:'FPR disparity: 6 pts',desc:'Improved but still present'},{t:'green',title:'Calibration: pass',desc:'Scores now well-calibrated'}],
      threshold:[{t:'green',title:'Gender gap: 3 pts',desc:'Near-parity achieved'},{t:'green',title:'FPR disparity: 3 pts',desc:'False rejection rates equalised'},{t:'green',title:'Accuracy: 92% overall',desc:'Slight accuracy reduction'}],
      adversarial:[{t:'green',title:'Gender gap: 2 pts',desc:'Best result across methods'},{t:'green',title:'FPR disparity: 2 pts',desc:'Near-equal false rejection'},{t:'green',title:'All metrics: pass',desc:'Most comprehensive mitigation'}],
    }
  },
  credit: {
    groups: ['Group A','Group B','Group C'],
    colors: ['#3d8ef0','#e05555','#4caf7d'],
    acc: { none:[95,74,78], resampling:[93,85,87], threshold:[91,88,89], adversarial:[92,90,91] },
    fpr: { none:[3,18,15], resampling:[5,10,9], threshold:[5,7,6], adversarial:[4,6,5] },
    radar: { none:[35,48,65,42,28], resampling:[58,68,72,65,52], threshold:[68,70,76,72,58], adversarial:[78,80,82,77,72] },
    trend: { none:[22,24,21,23,22,21], resampling:[21,17,14,10,8,6], threshold:[20,16,12,8,6,4], adversarial:[18,13,9,6,4,2] },
    alerts: {
      none:   [{t:'red',title:'Racial proxy: zip code',desc:'21 pt accuracy gap detected'},{t:'red',title:'FPR disparity: 15 pts',desc:'Minority groups denied 5× more'},{t:'amber',title:'Calibration: partial',desc:'Scores drift for Groups B & C'}],
      resampling:[{t:'amber',title:'Gap reduced to 8 pts',desc:'Resampling improved balance'},{t:'amber',title:'FPR disparity: 5 pts',desc:'Better but not resolved'},{t:'green',title:'Calibration: pass',desc:'Score meaning now consistent'}],
      threshold:[{t:'green',title:'Gap: 3 pts',desc:'Effective gap reduction'},{t:'green',title:'FPR disparity: 2 pts',desc:'Near-parity achieved'},{t:'amber',title:'Accuracy: 91%',desc:'Minor accuracy tradeoff'}],
      adversarial:[{t:'green',title:'Gap: 2 pts',desc:'Best performance overall'},{t:'green',title:'FPR disparity: 2 pts',desc:'Lowest disparity achieved'},{t:'green',title:'All metrics: pass',desc:'Proxy features neutralised'}],
    }
  },
  health: {
    groups: ['White','Black','Hispanic'],
    colors: ['#3d8ef0','#e05555','#e09840'],
    acc: { none:[94,70,72], resampling:[92,83,84], threshold:[90,87,88], adversarial:[91,89,90] },
    fpr: { none:[4,21,20], resampling:[5,12,11], threshold:[5,7,8], adversarial:[5,6,6] },
    radar: { none:[32,45,62,40,25], resampling:[55,65,70,62,50], threshold:[66,68,74,70,56], adversarial:[76,78,80,75,70] },
    trend: { none:[24,26,23,25,24,23], resampling:[23,19,15,11,9,7], threshold:[21,17,13,9,7,5], adversarial:[19,14,10,7,5,3] },
    alerts: {
      none:   [{t:'red',title:'Care gap: 24 pts',desc:'Black patients 70% vs white 94%'},{t:'red',title:'Cost proxy detected',desc:'Spending used instead of need'},{t:'red',title:'FPR: 17 pts disparity',desc:'Minority patients under-treated'}],
      resampling:[{t:'amber',title:'Gap reduced to 9 pts',desc:'Significant improvement'},{t:'amber',title:'FPR disparity: 7 pts',desc:'Still clinically significant'},{t:'green',title:'Proxy removed',desc:'Need-based scoring restored'}],
      threshold:[{t:'green',title:'Gap: 3 pts',desc:'Near clinical parity'},{t:'green',title:'FPR disparity: 3 pts',desc:'Under-treatment nearly resolved'},{t:'green',title:'Accuracy: 90%',desc:'Safe accuracy tradeoff'}],
      adversarial:[{t:'green',title:'Gap: 2 pts',desc:'Best-in-class result'},{t:'green',title:'FPR: 1 pt disparity',desc:'Clinical equity achieved'},{t:'green',title:'All metrics: pass',desc:'Holistic fairness achieved'}],
    }
  },
  justice: {
    groups: ['White','Black','Hispanic'],
    colors: ['#3d8ef0','#e05555','#e09840'],
    acc: { none:[93,70,73], resampling:[91,82,83], threshold:[89,86,87], adversarial:[90,88,89] },
    fpr: { none:[3,22,18], resampling:[4,13,11], threshold:[5,8,7], adversarial:[5,7,6] },
    radar: { none:[30,42,60,38,22], resampling:[52,62,68,60,48], threshold:[64,66,72,68,54], adversarial:[74,76,78,73,68] },
    trend: { none:[26,28,25,27,26,25], resampling:[25,21,17,13,11,9], threshold:[23,19,15,11,9,7], adversarial:[21,16,12,9,7,5] },
    alerts: {
      none:   [{t:'red',title:'Racial gap: 23 pts',desc:'Black defendants scored 22 pts higher'},{t:'red',title:'FPR: 19 pts disparity',desc:'False high-risk rate 7x higher'},{t:'red',title:'Calibration: failed',desc:'Scores not meaningful across groups'}],
      resampling:[{t:'amber',title:'Gap reduced to 10 pts',desc:'Improvement but still significant'},{t:'red',title:'FPR: 9 pts disparity',desc:'Still legally problematic'},{t:'amber',title:'Calibration: partial',desc:'Partial calibration improvement'}],
      threshold:[{t:'amber',title:'Gap: 3 pts',desc:'Near-parity in accuracy'},{t:'green',title:'FPR: 3 pts disparity',desc:'Major false-risk reduction'},{t:'green',title:'Calibration: pass',desc:'Score meaning equalised'}],
      adversarial:[{t:'green',title:'Gap: 2 pts',desc:'Near-complete racial parity'},{t:'green',title:'FPR: 2 pts disparity',desc:'Lowest false-risk rate achieved'},{t:'green',title:'All metrics: pass',desc:'Most equitable outcome'}],
    }
  }
};

const TREND_LABELS = ['Audit 1','Audit 2','Audit 3','Audit 4','Audit 5','Audit 6'];
const RADAR_LABELS = ['Dem. parity','Equal opp.','Calibration','Equal. odds','Individual'];
const GRID_COLOR = 'rgba(255,255,255,0.06)';
const TICK_COLOR = '#5a5856';

let accChart, fprChart, radarChart, trendChart;

function getDomain() { return DOMAINS[document.getElementById('domain-sel').value]; }
function getMit() { return document.getElementById('mit-sel').value; }

function gapColor(gap) {
  if (gap <= 5) return { main:'#4caf7d', accent:'--green' };
  if (gap <= 15) return { main:'#e09840', accent:'--amber' };
  return { main:'#e05555', accent:'--red' };
}

function updateDomain() {
  const d = getDomain(), m = getMit();
  const acc = d.acc[m], fpr = d.fpr[m], radar = d.radar[m], trend = d.trend[m];
  const maxGap = Math.max(...acc) - Math.min(...acc);
  const maxFpr = Math.max(...fpr) - Math.min(...fpr);
  const overallAcc = Math.round(acc.reduce((a,b)=>a+b,0)/acc.length);
  const radarAvg = Math.round(radar.reduce((a,b)=>a+b,0)/radar.length);

  const gc = gapColor(maxGap), fc = gapColor(maxFpr);
  const rc = radarAvg >= 70 ? '#4caf7d' : radarAvg >= 50 ? '#e09840' : '#e05555';

  document.getElementById('kpi-grid').innerHTML = `
    <div class="kpi-card fade-in" style="--accent:#3d8ef0">
      <div class="kpi-label">Overall accuracy</div>
      <div class="kpi-value" style="color:#e8e6e0">${overallAcc}%</div>
      <div class="kpi-sub">Across all groups</div>
    </div>
    <div class="kpi-card fade-in" style="--accent:${gc.main}">
      <div class="kpi-label">Max accuracy gap</div>
      <div class="kpi-value" style="color:${gc.main}">${maxGap} pts</div>
      <div class="kpi-sub" style="color:${gc.main}">${maxGap<=5?'Pass — within threshold':maxGap<=15?'Warning — review needed':'Critical — action required'}</div>
    </div>
    <div class="kpi-card fade-in" style="--accent:${fc.main}">
      <div class="kpi-label">Max FPR disparity</div>
      <div class="kpi-value" style="color:${fc.main}">${maxFpr} pts</div>
      <div class="kpi-sub" style="color:${fc.main}">${maxFpr<=5?'Pass':maxFpr<=12?'Warning':'Critical'}</div>
    </div>
    <div class="kpi-card fade-in" style="--accent:${rc}">
      <div class="kpi-label">Composite fairness</div>
      <div class="kpi-value" style="color:${rc}">${radarAvg}<span style="font-size:16px;color:#5a5856">/100</span></div>
      <div class="kpi-sub">Across 5 metrics</div>
    </div>
  `;

  const iconMap = { red:'!', amber:'~', green:'✓' };
  document.getElementById('alert-row').innerHTML = d.alerts[m].map(a=>`
    <div class="alert-card ${a.t} fade-in">
      <div class="alert-icon">${iconMap[a.t]}</div>
      <div><div class="alert-title">${a.title}</div><div class="alert-desc">${a.desc}</div></div>
    </div>`).join('');

  ['acc','fpr'].forEach(id => {
    document.getElementById(id+'-legend').innerHTML = d.groups.map((g,i)=>
      `<div class="legend-item"><span class="leg-dot" style="background:${d.colors[i]}"></span>${g}</div>`).join('');
  });

  const baseOpts = (max) => ({
    responsive: true, maintainAspectRatio: false, animation: { duration: 400 },
    plugins: { legend: { display: false }, tooltip: { backgroundColor:'#1e1e24', titleColor:'#8a8880', bodyColor:'#e8e6e0', borderColor:'rgba(255,255,255,0.1)', borderWidth:1, padding:10 } },
    scales: {
      x: { grid: { color: GRID_COLOR }, ticks: { color: TICK_COLOR, font: { size: 11, family: 'DM Mono' } } },
      y: { min: 0, max: max, grid: { color: GRID_COLOR }, ticks: { color: TICK_COLOR, font: { size: 11, family: 'DM Mono' }, callback: v => v + '%' } }
    }
  });

  if (accChart) accChart.destroy();
  accChart = new Chart(document.getElementById('accChart'), {
    type: 'bar',
    data: { labels: d.groups, datasets: d.groups.map((g,i)=>({ label:g, data:[acc[i]], backgroundColor:d.colors[i]+'99', borderColor:d.colors[i], borderWidth:1, borderRadius:4 })) },
    options: baseOpts(100)
  });

  if (fprChart) fprChart.destroy();
  fprChart = new Chart(document.getElementById('fprChart'), {
    type: 'bar',
    data: { labels: d.groups, datasets: d.groups.map((g,i)=>({ label:g, data:[fpr[i]], backgroundColor:d.colors[i]+'99', borderColor:d.colors[i], borderWidth:1, borderRadius:4 })) },
    options: baseOpts(30)
  });

  if (radarChart) radarChart.destroy();
  radarChart = new Chart(document.getElementById('radarChart'), {
    type: 'radar',
    data: {
      labels: RADAR_LABELS,
      datasets: [{ label:'Score', data:radar, backgroundColor:'rgba(61,142,240,0.1)', borderColor:'#3d8ef0', borderWidth:1.5, pointBackgroundColor:'#3d8ef0', pointRadius:3 }]
    },
    options: {
      responsive:true, maintainAspectRatio:false, animation:{duration:400},
      plugins:{ legend:{display:false} },
      scales:{ r:{ min:0, max:100, ticks:{stepSize:25,color:TICK_COLOR,font:{size:10,family:'DM Mono'},backdropColor:'transparent'}, grid:{color:GRID_COLOR}, angleLines:{color:GRID_COLOR}, pointLabels:{color:TICK_COLOR,font:{size:11,family:'DM Mono'}} } }
    }
  });

  if (trendChart) trendChart.destroy();
  trendChart = new Chart(document.getElementById('trendChart'), {
    type:'line',
    data:{
      labels:TREND_LABELS,
      datasets:[{ label:'Bias gap', data:trend, borderColor:'#3d8ef0', backgroundColor:'rgba(61,142,240,0.06)', borderWidth:2, pointBackgroundColor:'#3d8ef0', pointRadius:4, tension:0.3, fill:true }]
    },
    options:{
      responsive:true, maintainAspectRatio:false, animation:{duration:400},
      plugins:{ legend:{display:false}, tooltip:{backgroundColor:'#1e1e24',titleColor:'#8a8880',bodyColor:'#e8e6e0',borderColor:'rgba(255,255,255,0.1)',borderWidth:1,padding:10, callbacks:{label:ctx=>` Gap: ${ctx.raw} pts`}} },
      scales:{
        x:{grid:{color:GRID_COLOR},ticks:{color:TICK_COLOR,font:{size:11,family:'DM Mono'}}},
        y:{min:0,grid:{color:GRID_COLOR},ticks:{color:TICK_COLOR,font:{size:11,family:'DM Mono'},callback:v=>v+' pts'}}
      }
    }
  });
}

updateDomain();
</script>
</body>
</html>
