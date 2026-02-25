<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>StoreIQ — Busy Predictor</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<link rel="preconnect" href="https://fonts.googleapis.com"/>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:opsz,wght@9..40,300;9..40,400;9..40,500;9..40,600;9..40,700;9..40,800&display=swap" rel="stylesheet"/>
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
  --bg:#f2f2f7;
  --surface:rgba(255,255,255,0.82);
  --white:#fff;
  --border:rgba(0,0,0,0.08);
  --bmed:rgba(0,0,0,0.13);
  --t1:#1c1c1e;--t2:#636366;--t3:#aeaeb2;
  --blue:#0071e3;--blue-d:#0077ed;--blue-l:#e8f1fb;
  --green:#34c759;--green-l:#e5f8eb;
  --orange:#ff9500;--orange-l:#fff4e5;
  --red:#ff3b30;--red-l:#ffe5e3;
  --indigo:#5856d6;--indigo-l:#eeeefa;
  --teal:#5ac8fa;--teal-l:#e5f6fe;
  --r:16px;--rsm:12px;
  --s1:0 1px 3px rgba(0,0,0,0.05),0 4px 16px rgba(0,0,0,0.06);
  --s2:0 8px 30px rgba(0,0,0,0.1),0 2px 8px rgba(0,0,0,0.06);
}
html,body{height:100%;font-family:'DM Sans',-apple-system,sans-serif;background:var(--bg);color:var(--t1);-webkit-font-smoothing:antialiased}

/* NAV */
.nav{
  position:sticky;top:0;z-index:200;height:52px;
  background:rgba(242,242,247,0.88);
  backdrop-filter:saturate(200%) blur(28px);
  -webkit-backdrop-filter:saturate(200%) blur(28px);
  border-bottom:1px solid var(--border);
  display:flex;align-items:center;justify-content:space-between;
  padding:0 28px;
}
.nav-brand{display:flex;align-items:center;gap:10px;font-size:16px;font-weight:700;color:var(--t1);letter-spacing:-.4px}
.nav-icon{width:30px;height:30px;background:var(--blue);border-radius:9px;display:flex;align-items:center;justify-content:center;font-size:15px}
.nav-sub{font-size:12px;font-weight:500;color:var(--t3)}
.nav-right{display:flex;align-items:center;gap:10px}

/* SEGMENTED CONTROL */
.seg{display:flex;gap:3px;background:rgba(0,0,0,0.07);border-radius:11px;padding:3px}
.seg-btn{
  padding:6px 14px;border:none;background:none;cursor:pointer;
  font-family:inherit;font-size:13px;font-weight:500;color:var(--t2);
  border-radius:8px;transition:all .18s cubic-bezier(.4,0,.2,1);
  letter-spacing:-.1px;white-space:nowrap;
}
.seg-btn.on{background:var(--white);color:var(--t1);box-shadow:0 1px 4px rgba(0,0,0,0.14),0 .5px 1px rgba(0,0,0,0.1)}

/* UPLOAD */
.upload-screen{
  display:flex;align-items:center;justify-content:center;
  min-height:calc(100vh - 52px);padding:24px;
}
.upload-center{width:100%;max-width:540px}
.upload-hero{text-align:center;margin-bottom:40px}
.upload-hero h1{font-size:40px;font-weight:800;letter-spacing:-1.5px;color:var(--t1);line-height:1.05;margin-bottom:12px}
.upload-hero p{font-size:16px;color:var(--t2);line-height:1.6;font-weight:400}
.drop-zone{
  border:1.5px dashed rgba(0,113,227,.3);border-radius:22px;
  background:rgba(255,255,255,.7);backdrop-filter:blur(14px);
  text-align:center;cursor:pointer;padding:52px 44px;
  transition:all .22s cubic-bezier(.4,0,.2,1);position:relative;overflow:hidden;
}
.drop-zone::before{
  content:'';position:absolute;inset:0;
  background:radial-gradient(ellipse at 50% 0%,rgba(0,113,227,.06),transparent 70%);
  opacity:0;transition:opacity .3s;
}
.drop-zone:hover::before,.drop-zone.drag::before{opacity:1}
.drop-zone:hover,.drop-zone.drag{border-color:var(--blue);background:rgba(255,255,255,.85)}
.drop-zone.drag{transform:scale(1.01)}
.drop-icon{
  width:64px;height:64px;border-radius:18px;background:var(--blue-l);
  display:flex;align-items:center;justify-content:center;
  margin:0 auto 18px;font-size:30px;
  box-shadow:0 4px 16px rgba(0,113,227,.18);
}
.drop-title{font-size:19px;font-weight:700;color:var(--t1);margin-bottom:8px;letter-spacing:-.3px}
.drop-sub{font-size:13px;color:var(--t2);line-height:1.7;margin-bottom:24px}
.drop-sub code{background:rgba(0,0,0,.06);border-radius:5px;padding:1px 7px;font-size:12px;font-family:ui-monospace,monospace}
.btn-primary{
  display:inline-flex;align-items:center;gap:8px;
  background:var(--blue);color:#fff;border:none;border-radius:980px;
  padding:11px 26px;font-size:14px;font-weight:600;cursor:pointer;
  font-family:inherit;letter-spacing:-.2px;
  transition:all .15s cubic-bezier(.4,0,.2,1);
  box-shadow:0 2px 8px rgba(0,113,227,.35);
}
.btn-primary:hover{background:var(--blue-d);transform:scale(1.02);box-shadow:0 4px 16px rgba(0,113,227,.4)}
.btn-primary:active{transform:scale(.98)}
.btn-ghost{
  background:none;border:1px solid var(--bmed);border-radius:9px;
  padding:7px 15px;font-size:13px;font-weight:500;color:var(--t2);
  cursor:pointer;font-family:inherit;transition:all .15s;
}
.btn-ghost:hover{background:rgba(0,0,0,.04);color:var(--t1)}
.chip-row{display:flex;flex-wrap:wrap;gap:8px;margin-top:24px;justify-content:center}
.chip{
  display:inline-flex;align-items:center;gap:6px;
  padding:5px 12px;border-radius:20px;font-size:12px;font-weight:500;
  border:1px solid transparent;transition:transform .15s;
}
.chip:hover{transform:translateY(-1px)}
.err{margin-top:14px;font-size:13px;color:var(--red);font-weight:500}

/* DASHBOARD */
.dash{max-width:1180px;margin:0 auto;padding:28px 24px;display:flex;flex-direction:column;gap:18px}

/* CARDS */
.card{background:var(--white);border:1px solid var(--border);border-radius:var(--r);box-shadow:var(--s1);overflow:hidden}
.card-pad{padding:24px}
.card-hover{transition:transform .18s cubic-bezier(.4,0,.2,1),box-shadow .18s}
.card-hover:hover{transform:translateY(-2px);box-shadow:var(--s2)}

/* KPI */
.kpi-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
.kpi{background:var(--white);border:1px solid var(--border);border-radius:var(--r);padding:20px 22px;box-shadow:var(--s1);transition:transform .18s,box-shadow .18s;cursor:default}
.kpi:hover{transform:translateY(-2px);box-shadow:var(--s2)}
.kpi-label{font-size:12px;font-weight:500;color:var(--t2);margin-bottom:7px;letter-spacing:-.1px}
.kpi-val{font-size:32px;font-weight:800;color:var(--t1);letter-spacing:-1.5px;line-height:1}
.kpi-sub{font-size:12px;color:var(--t3);margin-top:5px}

/* SECTION */
.sec-title{font-size:17px;font-weight:700;color:var(--t1);letter-spacing:-.4px;margin-bottom:3px}
.sec-sub{font-size:13px;color:var(--t2);font-weight:400}

/* GRIDS */
.g2{display:grid;grid-template-columns:1fr 1fr;gap:18px}
.g7{display:grid;grid-template-columns:repeat(7,1fr);gap:10px}

/* BANNER */
.banner{border-radius:var(--r);padding:18px 20px;display:flex;align-items:center;gap:16px;border:1px solid transparent}
.banner-icon{font-size:32px;flex-shrink:0}
.banner-label{font-size:12px;font-weight:500;margin-bottom:3px}
.banner-val{font-size:24px;font-weight:800;letter-spacing:-.8px}
.banner-sub{font-size:12px;margin-top:2px}

/* CORR */
.corr-row{display:flex;align-items:center;gap:14px;padding:10px 0;border-bottom:1px solid rgba(0,0,0,.05)}
.corr-row:last-child{border:none}
.corr-name{font-size:13px;font-weight:500;color:var(--t1);width:210px;flex-shrink:0}
.corr-track{flex:1;height:6px;background:rgba(0,0,0,.06);border-radius:3px;overflow:hidden}
.corr-fill{height:100%;border-radius:3px;transition:width 1s cubic-bezier(.16,1,.3,1)}
.corr-val{width:40px;text-align:right;font-size:13px;font-weight:700;flex-shrink:0}

/* FEATURE */
.feat-row{display:flex;align-items:center;gap:14px;padding:14px 0;border-bottom:1px solid rgba(0,0,0,.05)}
.feat-row:last-child{border:none}
.feat-icon{width:28px;text-align:center;font-size:20px;flex-shrink:0}
.feat-name{font-size:14px;font-weight:600;color:var(--t1);width:240px;flex-shrink:0}
.feat-track{flex:1;height:7px;background:rgba(0,0,0,.06);border-radius:4px;overflow:hidden}
.feat-fill{height:100%;border-radius:4px;transition:width 1.1s cubic-bezier(.16,1,.3,1)}
.feat-val{width:44px;text-align:right;font-size:14px;font-weight:700;flex-shrink:0}
.badge{display:inline-flex;align-items:center;justify-content:center;font-size:11px;font-weight:600;padding:3px 9px;border-radius:6px;width:70px}

/* INSIGHT */
.ins-row{display:flex;gap:14px;padding:16px 0;border-bottom:1px solid rgba(0,0,0,.05)}
.ins-row:last-child{border:none}
.ins-icon{font-size:22px;flex-shrink:0;margin-top:1px}
.ins-title{font-size:14px;font-weight:700;color:var(--t1);letter-spacing:-.2px}
.ins-body{font-size:13px;color:var(--t2);margin-top:3px;line-height:1.6}

/* PREDICT */
.pcard{background:var(--white);border:1px solid var(--border);border-radius:var(--rsm);padding:18px 10px;text-align:center;box-shadow:var(--s1);transition:transform .18s,box-shadow .18s;cursor:default}
.pcard:hover{transform:translateY(-3px);box-shadow:var(--s2)}
.pday{font-size:10px;font-weight:700;color:var(--t2);text-transform:uppercase;letter-spacing:.8px;margin-bottom:10px}
.pnum{font-size:36px;font-weight:800;letter-spacing:-2px;line-height:1;margin-bottom:10px}
.plabel{display:inline-block;font-size:11px;font-weight:600;padding:3px 10px;border-radius:7px}

/* CHART WRAPPER */
.chart-wrap{position:relative;width:100%}

/* ANIMATIONS */
@keyframes fadeUp{from{opacity:0;transform:translateY(18px)}to{opacity:1;transform:translateY(0)}}
@keyframes scaleIn{from{opacity:0;transform:scale(.96)}to{opacity:1;transform:scale(1)}}
.fu{animation:fadeUp .42s cubic-bezier(.16,1,.3,1) both}
.fu1{animation-delay:.06s}.fu2{animation-delay:.12s}.fu3{animation-delay:.18s}.fu4{animation-delay:.24s}
.si{animation:scaleIn .35s cubic-bezier(.16,1,.3,1) both}

/* SCROLLBAR */
::-webkit-scrollbar{width:6px}
::-webkit-scrollbar-track{background:transparent}
::-webkit-scrollbar-thumb{background:rgba(0,0,0,.15);border-radius:3px}

/* RESPONSIVE */
@media(max-width:900px){
  .kpi-grid{grid-template-columns:repeat(2,1fr)}
  .g2{grid-template-columns:1fr}
  .g7{grid-template-columns:repeat(4,1fr)}
  .nav{padding:0 16px}
  .dash{padding:20px 16px}
  .seg-btn{padding:6px 10px;font-size:12px}
}
@media(max-width:500px){.g7{grid-template-columns:repeat(2,1fr)}.kpi-grid{grid-template-columns:1fr}}
</style>
</head>
<body>

<!-- NAV -->
<nav class="nav">
  <div class="nav-brand">
    <div class="nav-icon">📊</div>
    StoreIQ
    <span class="nav-sub" id="nav-sub" style="display:none"></span>
  </div>
  <div class="nav-right" id="nav-right" style="display:none">
    <div class="seg" id="tab-bar">
      <button class="seg-btn on" data-tab="overview">Overview</button>
      <button class="seg-btn" data-tab="trends">Trends</button>
      <button class="seg-btn" data-tab="features">Features</button>
      <button class="seg-btn" data-tab="predictor">Predictor</button>
    </div>
    <button class="btn-ghost" id="btn-new">New File</button>
  </div>
</nav>

<!-- UPLOAD SCREEN -->
<div class="upload-screen" id="upload-screen">
  <div class="upload-center">
    <div class="upload-hero">
      <h1>Store Busy<br/>Predictor</h1>
      <p>Upload your daily sales data to get correlations,<br/>seasonality analysis, and 7-day traffic forecasts.</p>
    </div>
    <div class="drop-zone" id="drop-zone">
      <div class="drop-icon">📄</div>
      <div class="drop-title">Drop your CSV here</div>
      <div class="drop-sub">
        Required: <code>Date</code> <code>Total_Sales</code> <code>Transactions</code> <code>Staff_Count</code><br/>
        Optional: <code>Promotion</code>
      </div>
      <button class="btn-primary" id="btn-upload">Choose File</button>
      <input type="file" id="file-input" accept=".csv" style="display:none"/>
      <div class="err" id="err-msg"></div>
    </div>
    <div class="chip-row">
      <div class="chip" style="background:#0071e312;border-color:#0071e328;color:#0071e3">📅 Day of week</div>
      <div class="chip" style="background:#5ac8fa12;border-color:#5ac8fa28;color:#5ac8fa">🌤️ Seasonality</div>
      <div class="chip" style="background:#ff3b3012;border-color:#ff3b3028;color:#ff3b30">🎉 Public holidays</div>
      <div class="chip" style="background:#ff950012;border-color:#ff950028;color:#ff9500">🎒 School holidays</div>
      <div class="chip" style="background:#5856d612;border-color:#5856d628;color:#5856d6">🏷️ Promotions</div>
      <div class="chip" style="background:#34c75912;border-color:#34c75928;color:#34c759">👥 Staffing</div>
    </div>
  </div>
</div>

<!-- DASHBOARD -->
<div class="dash" id="dashboard" style="display:none"></div>

<script>
// ─── Constants ────────────────────────────────────────────────────────────────
const AU_HOL = new Set(["2024-01-01","2024-01-26","2024-03-29","2024-04-01","2024-04-25","2024-05-12","2024-06-03","2024-12-25","2024-12-26","2025-01-01","2025-01-27","2025-04-18","2025-04-21","2025-04-25","2025-05-11","2025-06-09","2025-12-25","2025-12-26"]);
const SCH = new Set(["04-13","04-14","04-15","04-16","04-17","04-18","04-19","04-20","04-21","04-22","04-23","04-24","04-25","04-26","04-27","07-01","07-02","07-03","07-04","07-05","07-06","07-07","07-08","07-09","07-10","07-11","07-12","07-13","09-21","09-22","09-23","09-24","09-25","09-26","09-27","09-28","09-29","09-30","10-01","10-02","10-03","10-04","10-05","12-20","12-21","12-22","12-23","12-24","12-25","12-26","12-27","12-28","12-29","12-30","12-31","01-01","01-02","01-03","01-04","01-05","01-06","01-07","01-08","01-09","01-10","01-11","01-12","01-13","01-14","01-15","01-16","01-17","01-18","01-19","01-20","01-21","01-22","01-23","01-24","01-25","01-26","01-27","01-28","01-29","01-30","01-31"]);
const SEASONS={12:"Summer",1:"Summer",2:"Summer",3:"Autumn",4:"Autumn",5:"Autumn",6:"Winter",7:"Winter",8:"Winter",9:"Spring",10:"Spring",11:"Spring"};
const DOWNAMES=["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];
const DOWORD=["Mon","Tue","Wed","Thu","Fri","Sat","Sun"];
const MONTHS=["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
const LEVEL={Busy:{bg:"#fff4e5",fg:"#ff9500"},Steady:{bg:"#e5f8eb",fg:"#34c759"},Quiet:{bg:"#e8f1fb",fg:"#0071e3"}};
const SEAS_COLORS=["#5ac8fa","#ff9500","#5856d6","#34c759"];

// ─── Math ─────────────────────────────────────────────────────────────────────
const mean=a=>a.length?a.reduce((x,y)=>x+y,0)/a.length:0;
function pearson(xs,ys){
  const n=xs.length; if(n<2)return 0;
  const mx=mean(xs),my=mean(ys);
  const num=xs.reduce((s,x,i)=>s+(x-mx)*(ys[i]-my),0);
  const den=Math.sqrt(xs.reduce((s,x)=>s+(x-mx)**2,0)*ys.reduce((s,y)=>s+(y-my)**2,0));
  return den===0?0:+(num/den).toFixed(3);
}
function movAvg(arr,w){return arr.map((_,i)=>{const s=arr.slice(Math.max(0,i-w+1),i+1);return s.reduce((a,b)=>a+b,0)/s.length});}
function linReg(xs,ys){
  const mx=mean(xs),my=mean(ys);
  const sl=xs.reduce((s,x,i)=>s+(x-mx)*(ys[i]-my),0)/(xs.reduce((s,x)=>s+(x-mx)**2,1e-9));
  return{slope:sl,intercept:my-sl*mx};
}

// ─── Enrich ───────────────────────────────────────────────────────────────────
function enrich(rows){
  return rows.map(r=>{
    const d=new Date(r.Date),dow=DOWNAMES[d.getDay()],month=d.getMonth()+1;
    return{...r,dow,month,monthName:MONTHS[month-1],season:SEASONS[month],
      isWeekend:dow==="Sat"||dow==="Sun"?1:0,
      isHoliday:AU_HOL.has(r.Date)?1:0,
      isSchoolHol:SCH.has(r.Date.slice(5))?1:0,
      isPromo:r.Promotion!=null&&r.Promotion>0?1:0};
  });
}

// ─── Predict ──────────────────────────────────────────────────────────────────
function predict7(E){
  const recent=E.slice(-60),xs=recent.map((_,i)=>i),ys=recent.map(d=>d.Transactions);
  const{slope,intercept}=linReg(xs,ys);
  const gm=mean(E.map(d=>d.Transactions));
  const dm={};
  E.forEach(d=>{if(!dm[d.dow])dm[d.dow]=[];dm[d.dow].push(d.Transactions);});
  const mult={};
  DOWORD.forEach(day=>{mult[day]=dm[day]?mean(dm[day])/gm:1;});
  return DOWORD.map((day,i)=>{
    const pred=Math.max(0,Math.round((intercept+slope*(recent.length+i))*(mult[day]||1)));
    return{day,predicted:pred,level:pred>gm*1.2?"Busy":pred<gm*0.8?"Quiet":"Steady"};
  });
}

// ─── Feature importance ───────────────────────────────────────────────────────
function features(E){
  const txns=E.map(d=>d.Transactions);
  return[
    {name:"Day of Week (Weekend)",values:E.map(d=>d.isWeekend),icon:"📅",color:"#0071e3"},
    {name:"Promotions / Marketing",values:E.map(d=>d.isPromo),icon:"🏷️",color:"#5856d6"},
    {name:"School Holidays",values:E.map(d=>d.isSchoolHol),icon:"🎒",color:"#ff9500"},
    {name:"Public Holidays",values:E.map(d=>d.isHoliday),icon:"🎉",color:"#ff3b30"},
    {name:"Staff Count",values:E.map(d=>d.Staff_Count||0),icon:"👥",color:"#34c759"},
    {name:"Month / Seasonality",values:E.map(d=>d.month),icon:"🌤️",color:"#5ac8fa"},
  ].map(f=>({...f,r:pearson(f.values,txns)})).sort((a,b)=>Math.abs(b.r)-Math.abs(a.r));
}

// ─── Chart registry ───────────────────────────────────────────────────────────
const charts={};
function destroyChart(id){if(charts[id]){charts[id].destroy();delete charts[id];}}

// ─── Analyse ──────────────────────────────────────────────────────────────────
let A=null;
function analyse(raw){
  const E=enrich(raw);
  const sales=E.map(d=>d.Total_Sales),txns=E.map(d=>d.Transactions),staff=E.map(d=>d.Staff_Count||0);
  const ma7=movAvg(sales,7),ma14=movAvg(sales,14);
  const avgS=Math.round(mean(sales)),avgT=Math.round(mean(txns)),avgSt=+mean(staff).toFixed(1);
  const totalRev=sales.reduce((a,b)=>a+b,0);

  const dowAgg={};
  E.forEach(d=>{if(!dowAgg[d.dow])dowAgg[d.dow]={s:[],t:[],st:[]};dowAgg[d.dow].s.push(d.Total_Sales);dowAgg[d.dow].t.push(d.Transactions);dowAgg[d.dow].st.push(d.Staff_Count||0);});
  const dowData=DOWORD.map(day=>({day,avgSales:dowAgg[day]?Math.round(mean(dowAgg[day].s)):0,avgTxns:dowAgg[day]?Math.round(mean(dowAgg[day].t)):0,avgStaff:dowAgg[day]?+mean(dowAgg[day].st).toFixed(1):0}));

  const mAgg={};
  E.forEach(d=>{if(!mAgg[d.monthName])mAgg[d.monthName]={s:[],t:[]};mAgg[d.monthName].s.push(d.Total_Sales);mAgg[d.monthName].t.push(d.Transactions);});
  const monthData=MONTHS.filter(m=>mAgg[m]).map(m=>({month:m,avgSales:Math.round(mean(mAgg[m].s)),avgTxns:Math.round(mean(mAgg[m].t))}));

  const sAgg={};
  E.forEach(d=>{if(!sAgg[d.season])sAgg[d.season]=[];sAgg[d.season].push(d.Transactions);});
  const seasonData=["Summer","Autumn","Winter","Spring"].filter(s=>sAgg[s]).map(s=>({season:s,avgTxns:Math.round(mean(sAgg[s]))}));

  const step=Math.max(1,Math.floor(E.length/100));
  const trendData=E.filter((_,i)=>i%step===0).map((d,idx)=>({date:d.Date.slice(5),sales:d.Total_Sales,txns:d.Transactions,ma7:Math.round(ma7[idx*step]),ma14:Math.round(ma14[idx*step])}));

  const corrs={salesVsStaff:pearson(sales,staff),txnsVsStaff:pearson(txns,staff),salesVsTxns:pearson(sales,txns),salesVsPromo:pearson(sales,E.map(d=>d.isPromo)),txnsVsHol:pearson(txns,E.map(d=>d.isHoliday)),txnsVsSchool:pearson(txns,E.map(d=>d.isSchoolHol))};

  const promoR=E.filter(d=>d.isPromo),noPromo=E.filter(d=>!d.isPromo);
  const promoLift=promoR.length&&noPromo.length?((mean(promoR.map(d=>d.Transactions))/mean(noPromo.map(d=>d.Transactions))-1)*100).toFixed(1):null;
  const holR=E.filter(d=>d.isHoliday),noHol=E.filter(d=>!d.isHoliday);
  const holLift=holR.length&&noHol.length?((mean(holR.map(d=>d.Transactions))/mean(noHol.map(d=>d.Transactions))-1)*100).toFixed(1):null;

  const feats=features(E);
  const preds=predict7(E);
  const peakDay=dowData.reduce((a,b)=>b.avgTxns>a.avgTxns?b:a);
  const quietDay=dowData.reduce((a,b)=>b.avgTxns<a.avgTxns?b:a);

  return{E,raw,corrs,dowData,monthData,seasonData,trendData,avgS,avgT,avgSt,totalRev,preds,feats,peakDay,quietDay,promoLift,holLift};
}

// ─── Render helpers ───────────────────────────────────────────────────────────
const $ =s=>document.querySelector(s);
const $$ =s=>document.querySelectorAll(s);
const el=(tag,cls,html='')=>{const e=document.createElement(tag);if(cls)e.className=cls;if(html)e.innerHTML=html;return e;};
const fmt=n=>n.toLocaleString();

function chartDefaults(){
  return{
    responsive:true,maintainAspectRatio:false,
    plugins:{legend:{display:false},tooltip:{
      backgroundColor:'rgba(255,255,255,.95)',
      titleColor:'#6e6e73',bodyColor:'#1c1c1e',
      borderColor:'rgba(0,0,0,.12)',borderWidth:1,
      padding:10,cornerRadius:10,
      titleFont:{family:'DM Sans',size:11,weight:'500'},
      bodyFont:{family:'DM Sans',size:13,weight:'600'},
    }},
    scales:{
      x:{grid:{display:false},border:{display:false},ticks:{color:'#6e6e73',font:{family:'DM Sans',size:11}}},
      y:{grid:{color:'rgba(0,0,0,.05)'},border:{display:false},ticks:{color:'#6e6e73',font:{family:'DM Sans',size:11}}},
    },
  };
}

// ─── Tab rendering ────────────────────────────────────────────────────────────
function renderOverview(){
  const d=$('#dashboard');
  d.innerHTML='';

  // KPIs
  const kgrid=el('div','kpi-grid fu');
  [
    {l:'Total Revenue',v:`$${(A.totalRev/1000).toFixed(1)}k`,s:`$${fmt(A.avgS)} avg/day`},
    {l:'Avg Transactions / Day',v:fmt(A.avgT),s:'customers per day'},
    {l:'Avg Staff / Day',v:A.avgSt,s:'headcount'},
    {l:'Busiest Day',v:A.peakDay.day,s:`${fmt(A.peakDay.avgTxns)} avg transactions`},
  ].forEach(k=>{
    kgrid.innerHTML+=`<div class="kpi"><div class="kpi-label">${k.l}</div><div class="kpi-val">${k.v}</div><div class="kpi-sub">${k.s}</div></div>`;
  });
  d.appendChild(kgrid);

  // Banners
  if(A.promoLift||A.holLift){
    const brow=el('div','g2 fu fu1');
    if(A.promoLift){
      const sign=parseFloat(A.promoLift)>0?'+':'';
      brow.innerHTML+=`<div class="banner" style="background:#eeeefa;border-color:#5856d630;"><div class="banner-icon">🏷️</div><div><div class="banner-label" style="color:#5856d6">Promotion Impact</div><div class="banner-val" style="color:#5856d6">${sign}${A.promoLift}%</div><div class="banner-sub" style="color:#8886e0">traffic vs non-promo days</div></div></div>`;
    }
    if(A.holLift){
      const sign=parseFloat(A.holLift)>0?'+':'';
      brow.innerHTML+=`<div class="banner" style="background:#ffe5e3;border-color:#ff3b3030;"><div class="banner-icon">🎉</div><div><div class="banner-label" style="color:#ff3b30">Public Holiday Impact</div><div class="banner-val" style="color:#ff3b30">${sign}${A.holLift}%</div><div class="banner-sub" style="color:#ff7b73">traffic vs regular days</div></div></div>`;
    }
    d.appendChild(brow);
  }

  // DOW chart + Correlations
  const row2=el('div','g2 fu fu2');

  // DOW
  const dowCard=el('div','card card-pad');
  dowCard.innerHTML=`<div class="sec-title">Traffic by Day of Week</div><div class="sec-sub" style="margin-bottom:18px">Average customer transactions</div><div class="chart-wrap" style="height:200px"><canvas id="chart-dow"></canvas></div>`;
  row2.appendChild(dowCard);

  // Corr
  const corrCard=el('div','card card-pad');
  corrCard.innerHTML=`<div class="sec-title">Key Correlations</div><div class="sec-sub" style="margin-bottom:14px">How strongly each factor relates to transactions</div>
  ${[
    {l:'Sales ↔ Transactions',r:A.corrs.salesVsTxns,c:'#0071e3'},
    {l:'Sales ↔ Staffing',r:A.corrs.salesVsStaff,c:'#34c759'},
    {l:'Traffic ↔ Staffing',r:A.corrs.txnsVsStaff,c:'#5ac8fa'},
    {l:'Sales ↔ Promotion',r:A.corrs.salesVsPromo,c:'#5856d6'},
    {l:'Traffic ↔ Public Holiday',r:A.corrs.txnsVsHol,c:'#ff3b30'},
    {l:'Traffic ↔ School Holiday',r:A.corrs.txnsVsSchool,c:'#ff9500'},
  ].map(c=>`<div class="corr-row"><div class="corr-name">${c.l}</div><div class="corr-track"><div class="corr-fill" style="width:${Math.abs(c.r)*100}%;background:${c.c}"></div></div><div class="corr-val" style="color:${c.c}">${c.r}</div></div>`).join('')}`;
  row2.appendChild(corrCard);
  d.appendChild(row2);

  // Monthly
  const monthCard=el('div','card card-pad fu fu3');
  monthCard.innerHTML=`<div class="sec-title">Monthly Seasonality</div><div class="sec-sub" style="margin-bottom:18px">Average daily customer transactions by month</div><div class="chart-wrap" style="height:195px"><canvas id="chart-month"></canvas></div>`;
  d.appendChild(monthCard);

  // Draw charts after DOM ready
  requestAnimationFrame(()=>{
    // DOW bar
    destroyChart('dow');
    const peakIdx=DOWORD.indexOf(A.peakDay.day);
    charts['dow']=new Chart($('#chart-dow'),{
      type:'bar',
      data:{
        labels:A.dowData.map(d=>d.day),
        datasets:[{data:A.dowData.map(d=>d.avgTxns),backgroundColor:A.dowData.map((d,i)=>i===peakIdx?'#0071e3':'rgba(0,113,227,.18)'),borderRadius:7,borderSkipped:false}]
      },
      options:{...chartDefaults(),plugins:{...chartDefaults().plugins,tooltip:{...chartDefaults().plugins.tooltip,callbacks:{label:c=>`${fmt(c.raw)} avg transactions`}}}}
    });

    // Month bar
    destroyChart('month');
    charts['month']=new Chart($('#chart-month'),{
      type:'bar',
      data:{
        labels:A.monthData.map(d=>d.month),
        datasets:[{data:A.monthData.map(d=>d.avgTxns),backgroundColor:A.monthData.map((_,i)=>`hsl(${200+i*12},68%,${50+i%2*8}%)`),borderRadius:7,borderSkipped:false}]
      },
      options:{...chartDefaults(),plugins:{...chartDefaults().plugins,tooltip:{...chartDefaults().plugins.tooltip,callbacks:{label:c=>`${fmt(c.raw)} avg transactions`}}}}
    });
  });
}

function renderTrends(){
  const d=$('#dashboard');
  d.innerHTML='';

  const c1=el('div','card card-pad fu');
  c1.innerHTML=`<div class="sec-title">Sales Trend</div><div class="sec-sub" style="margin-bottom:20px">Daily revenue with 7-day and 14-day moving averages</div><div class="chart-wrap" style="height:270px"><canvas id="chart-trend"></canvas></div>`;
  d.appendChild(c1);

  const row=el('div','g2 fu fu1');

  const c2=el('div','card card-pad');
  c2.innerHTML=`<div class="sec-title">Season Comparison</div><div class="sec-sub" style="margin-bottom:18px">Avg transactions by season</div><div class="chart-wrap" style="height:185px"><canvas id="chart-season"></canvas></div>`;
  row.appendChild(c2);

  const c3=el('div','card card-pad');
  c3.innerHTML=`<div class="sec-title">Transaction Volume</div><div class="sec-sub" style="margin-bottom:18px">Daily customer count over time</div><div class="chart-wrap" style="height:185px"><canvas id="chart-txn"></canvas></div>`;
  row.appendChild(c3);
  d.appendChild(row);

  requestAnimationFrame(()=>{
    const labels=A.trendData.map(d=>d.date);
    const interval=Math.max(1,Math.floor(labels.length/8));

    // Trend
    destroyChart('trend');
    charts['trend']=new Chart($('#chart-trend'),{
      type:'line',
      data:{labels,datasets:[
        {label:'Daily Sales',data:A.trendData.map(d=>d.sales),borderColor:'rgba(0,113,227,.25)',backgroundColor:'rgba(0,113,227,.05)',fill:true,tension:.3,pointRadius:0,borderWidth:1.5},
        {label:'7-day avg',data:A.trendData.map(d=>d.ma7),borderColor:'#0071e3',fill:false,tension:.3,pointRadius:0,borderWidth:2.5},
        {label:'14-day avg',data:A.trendData.map(d=>d.ma14),borderColor:'#5856d6',fill:false,tension:.3,pointRadius:0,borderWidth:2,borderDash:[6,3]},
      ]},
      options:{...chartDefaults(),plugins:{...chartDefaults().plugins,legend:{display:true,labels:{font:{family:'DM Sans',size:12},color:'#6e6e73',boxWidth:20,usePointStyle:true}}},scales:{...chartDefaults().scales,x:{...chartDefaults().scales.x,ticks:{...chartDefaults().scales.x.ticks,maxTicksLimit:8}},y:{...chartDefaults().scales.y,ticks:{...chartDefaults().scales.y.ticks,callback:v=>`$${(v/1000).toFixed(0)}k`}}}}
    });

    // Season
    destroyChart('season');
    charts['season']=new Chart($('#chart-season'),{
      type:'bar',
      data:{labels:A.seasonData.map(d=>d.season),datasets:[{data:A.seasonData.map(d=>d.avgTxns),backgroundColor:A.seasonData.map((_,i)=>SEAS_COLORS[i]),borderRadius:9,borderSkipped:false}]},
      options:{...chartDefaults(),plugins:{...chartDefaults().plugins,tooltip:{...chartDefaults().plugins.tooltip,callbacks:{label:c=>`${fmt(c.raw)} avg transactions`}}}}
    });

    // Txn
    destroyChart('txn');
    charts['txn']=new Chart($('#chart-txn'),{
      type:'line',
      data:{labels,datasets:[{label:'Transactions',data:A.trendData.map(d=>d.txns),borderColor:'#34c759',backgroundColor:'rgba(52,199,89,.08)',fill:true,tension:.3,pointRadius:0,borderWidth:2}]},
      options:{...chartDefaults(),scales:{...chartDefaults().scales,x:{...chartDefaults().scales.x,ticks:{...chartDefaults().scales.x.ticks,maxTicksLimit:5}}}}
    });
  });
}

function renderFeatures(){
  const d=$('#dashboard');
  d.innerHTML='';

  // Feature importance
  const c1=el('div','card card-pad fu');
  let rows='';
  A.feats.forEach(f=>{
    const strength=Math.abs(f.r)>.5?{bg:'var(--green-l)',fg:'var(--green)',txt:'Strong'}:Math.abs(f.r)>.3?{bg:'var(--orange-l)',fg:'var(--orange)',txt:'Moderate'}:{bg:'rgba(0,0,0,.05)',fg:'var(--t3)',txt:'Weak'};
    rows+=`<div class="feat-row">
      <div class="feat-icon">${f.icon}</div>
      <div class="feat-name">${f.name}</div>
      <div class="feat-track"><div class="feat-fill" style="width:${Math.abs(f.r)*100}%;background:${f.color}"></div></div>
      <div class="feat-val" style="color:${f.color}">${f.r}</div>
      <span class="badge" style="background:${strength.bg};color:${strength.fg}">${strength.txt}</span>
    </div>`;
  });
  c1.innerHTML=`<div class="sec-title">Feature Importance</div><div class="sec-sub" style="margin-bottom:22px">Correlation with customer transaction volume — ranked by impact</div>${rows}`;
  d.appendChild(c1);

  // Insights
  const c2=el('div','card card-pad fu fu1');
  const insights=[
    {icon:'📅',title:'Day of Week',body:`Weekdays vs weekends follow consistent patterns. Your busiest day is <strong>${A.peakDay.day}</strong> and quietest is <strong>${A.quietDay.day}</strong>. Pre-schedule staff around this rhythm.`},
    {icon:'🌤️',title:'Month & Season',body:'Sales naturally fluctuate with seasons. Summer and holiday months typically see higher foot traffic across most retail categories.'},
    {icon:'🎉',title:'Public Holidays',body:A.holLift?`Public holidays shift your traffic by <strong>${A.holLift}%</strong> vs a normal day. Mother's Day, Easter, and Christmas are significant events.`:"Track holidays to anticipate spikes — Mother's Day, Easter, and Christmas are high-traffic events."},
    {icon:'🎒',title:'School Holidays',body:'Families shop more during school breaks, especially midweek. Daytime traffic increases as parents and children are out.'},
    {icon:'🏷️',title:'Promotions',body:A.promoLift?`Promotional days boosted transactions by <strong>${A.promoLift}%</strong> on average — measurable return on marketing spend.`:'Logging promotions lets you isolate their impact and calculate the exact ROI per campaign.'},
    {icon:'👥',title:'Staff Count',body:`Staff-to-sales correlation: <strong>${A.corrs.salesVsStaff}</strong>. ${Math.abs(A.corrs.salesVsStaff)>.4?'Higher staffing correlates with higher sales — faster service, shorter queues.':'Staffing alone doesn\'t strongly predict sales here — other factors are at play.'}`},
  ];
  c2.innerHTML=`<div class="sec-title" style="margin-bottom:18px">Why Each Factor Matters</div>`+insights.map(i=>`<div class="ins-row"><div class="ins-icon">${i.icon}</div><div><div class="ins-title">${i.title}</div><div class="ins-body">${i.body}</div></div></div>`).join('');
  d.appendChild(c2);
}

function renderPredictor(){
  const d=$('#dashboard');
  d.innerHTML='';

  // Header
  const hdr=el('div','fu');
  hdr.innerHTML=`<div style="font-size:30px;font-weight:800;color:var(--t1);letter-spacing:-1px;margin-bottom:5px">Next 7 Days</div><div style="font-size:15px;color:var(--t2)">Predicted customer traffic using historical trends, day-of-week patterns &amp; seasonal signals</div>`;
  d.appendChild(hdr);

  // Prediction cards
  const pgrid=el('div','g7 fu fu1');
  A.preds.forEach(p=>{
    const lv=LEVEL[p.level];
    pgrid.innerHTML+=`<div class="pcard"><div class="pday">${p.day}</div><div class="pnum" style="color:${lv.fg}">${fmt(p.predicted)}</div><span class="plabel" style="background:${lv.bg};color:${lv.fg}">${p.level}</span></div>`;
  });
  d.appendChild(pgrid);

  // Bar chart
  const bc=el('div','card card-pad fu fu2');
  bc.innerHTML=`<div class="sec-title" style="margin-bottom:18px">Predicted vs Historical Average</div><div class="chart-wrap" style="height:225px"><canvas id="chart-pred"></canvas></div>`;
  d.appendChild(bc);

  // Info
  const info=el('div','card card-pad fu fu3');
  info.style.background='rgba(255,255,255,.55)';
  info.innerHTML=`<div style="display:flex;gap:14px;align-items:flex-start"><div style="font-size:22px;flex-shrink:0">ℹ️</div><div><div style="font-size:14px;font-weight:700;color:var(--t1);margin-bottom:4px">About this model</div><div style="font-size:13px;color:var(--t2);line-height:1.7">Linear regression on the trailing 60 days, weighted by day-of-week multipliers from full history. Public holidays, school holidays, and promotions improve the feature analysis — treat predictions as a strong baseline, not a guarantee.</div></div></div>`;
  d.appendChild(info);

  requestAnimationFrame(()=>{
    destroyChart('pred');
    const avgLine=A.avgT;
    charts['pred']=new Chart($('#chart-pred'),{
      type:'bar',
      data:{
        labels:A.preds.map(p=>p.day),
        datasets:[{
          label:'Predicted Transactions',
          data:A.preds.map(p=>p.predicted),
          backgroundColor:A.preds.map(p=>LEVEL[p.level].fg+'cc'),
          borderRadius:9,borderSkipped:false,
        }]
      },
      options:{
        ...chartDefaults(),
        plugins:{
          ...chartDefaults().plugins,
          tooltip:{...chartDefaults().plugins.tooltip,callbacks:{label:c=>`${fmt(c.raw)} customers`}},
          annotation:{annotations:{avg:{type:'line',yMin:avgLine,yMax:avgLine,borderColor:'#aeaeb2',borderWidth:1.5,borderDash:[5,5],label:{display:true,content:`Hist. avg: ${fmt(avgLine)}`,position:'end',color:'#aeaeb2',font:{family:'DM Sans',size:11}}}}},
        }
      }
    });
  });
}

// ─── Tab switching ────────────────────────────────────────────────────────────
let currentTab='overview';
const tabFns={overview:renderOverview,trends:renderTrends,features:renderFeatures,predictor:renderPredictor};

function switchTab(tab){
  currentTab=tab;
  $$('.seg-btn').forEach(b=>{b.classList.toggle('on',b.dataset.tab===tab);});
  Object.values(charts).forEach(c=>c.destroy&&c.destroy());
  Object.keys(charts).forEach(k=>delete charts[k]);
  tabFns[tab]();
}

$('#tab-bar').addEventListener('click',e=>{
  if(e.target.dataset.tab) switchTab(e.target.dataset.tab);
});

// ─── File handling ────────────────────────────────────────────────────────────
function processFile(file){
  if(!file)return;
  $('#err-msg').textContent='';
  Papa.parse(file,{
    header:true,skipEmptyLines:true,dynamicTyping:true,
    complete:({data:rows,errors})=>{
      if(errors.length){$('#err-msg').textContent='Parse error: '+errors[0].message;return;}
      const req=['Date','Total_Sales','Transactions','Staff_Count'];
      const cols=Object.keys(rows[0]||{});
      const miss=req.filter(r=>!cols.includes(r));
      if(miss.length){$('#err-msg').textContent='Missing columns: '+miss.join(', ');return;}
      const clean=rows.filter(r=>r.Date&&r.Total_Sales!=null&&r.Transactions!=null).sort((a,b)=>new Date(a.Date)-new Date(b.Date));
      if(clean.length<7){$('#err-msg').textContent='Need at least 7 rows of data.';return;}
      A=analyse(clean);
      showDashboard(clean.length);
    }
  });
}

function showDashboard(n){
  $('#upload-screen').style.display='none';
  $('#dashboard').style.display='flex';
  $('#nav-right').style.display='flex';
  $('#nav-sub').style.display='inline';
  $('#nav-sub').textContent=`— ${n} days loaded`;
  switchTab('overview');
}

function showUpload(){
  A=null;
  Object.values(charts).forEach(c=>c.destroy&&c.destroy());
  Object.keys(charts).forEach(k=>delete charts[k]);
  $('#dashboard').style.display='none';
  $('#upload-screen').style.display='flex';
  $('#nav-right').style.display='none';
  $('#nav-sub').style.display='none';
  $('#file-input').value='';
  // reset tab
  $$('.seg-btn').forEach(b=>b.classList.toggle('on',b.dataset.tab==='overview'));
}

$('#btn-upload').addEventListener('click',()=>$('#file-input').click());
$('#file-input').addEventListener('change',e=>processFile(e.target.files[0]));
$('#btn-new').addEventListener('click',showUpload);

// Drag & drop
const dz=$('#drop-zone');
dz.addEventListener('dragover',e=>{e.preventDefault();dz.classList.add('drag');});
dz.addEventListener('dragleave',()=>dz.classList.remove('drag'));
dz.addEventListener('drop',e=>{e.preventDefault();dz.classList.remove('drag');processFile(e.dataTransfer.files[0]);});
</script>
</body>
</html>

