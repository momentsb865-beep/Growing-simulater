# Growing-simulater
Grow the garden
<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0"/>
<title>🌾 Harvest Haven</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=Fredoka+One&family=Nunito:wght@400;600;700;800&display=swap" rel="stylesheet"/>
<style>
  *{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent}
  html,body,#root{height:100%;width:100%;overflow-x:hidden}
  body{font-family:'Nunito',sans-serif;background:#1b3d08}

@keyframes pulse-glow{0%,100%{box-shadow:0 0 8px rgba(118,255,3,.4),inset 0 2px 4px rgba(0,0,0,.2)}50%{box-shadow:0 0 26px rgba(118,255,3,.9),inset 0 2px 4px rgba(0,0,0,.2)}}
@keyframes wobble{0%,100%{transform:rotate(-4deg) scale(1.06)}50%{transform:rotate(4deg) scale(1.13)}}
@keyframes pop-out{0%{transform:scale(1);opacity:1}50%{transform:scale(1.5);opacity:1}100%{transform:scale(0);opacity:0}}
@keyframes flash-card{0%,100%{background:rgba(255,248,220,.1)}50%{background:rgba(255,220,80,.32)}}
@keyframes toast-anim{0%{transform:translateX(-50%) translateY(30px);opacity:0}12%{transform:translateX(-50%) translateY(0);opacity:1}78%{transform:translateX(-50%) translateY(0);opacity:1}100%{transform:translateX(-50%) translateY(-22px);opacity:0}}
@keyframes coin-pop{0%{transform:scale(1)}50%{transform:scale(1.38) rotate(14deg)}100%{transform:scale(1)}}
@keyframes shake{0%,100%{transform:translateX(0)}20%{transform:translateX(-8px)}40%{transform:translateX(8px)}60%{transform:translateX(-5px)}80%{transform:translateX(5px)}}
@keyframes fade-in{from{opacity:0;transform:scale(.95)}to{opacity:1;transform:scale(1)}}
@keyframes bounce-in{0%{transform:scale(.25);opacity:0}55%{transform:scale(1.12)}100%{transform:scale(1);opacity:1}}
@keyframes regrow{0%,100%{opacity:.55;transform:scale(.93)}50%{opacity:1;transform:scale(1.06)}}
@keyframes pest-blink{0%,100%{box-shadow:0 0 14px rgba(255,80,0,.75);border-color:rgba(255,110,0,.95)!important}50%{box-shadow:0 0 28px rgba(255,80,0,1);border-color:#FF6D00!important}}
@keyframes timer-flash{0%,100%{color:#FF5722}50%{color:#FFAB91}}
@keyframes slide-up{from{transform:translateY(18px);opacity:0}to{transform:translateY(0);opacity:1}}
@keyframes star-float{0%,100%{transform:translateY(0) rotate(0)}50%{transform:translateY(-8px) rotate(10deg)}}

.plot-ready{animation:pulse-glow 1.4s ease-in-out infinite;border:2px solid #76FF03!important}
.ready-emoji{animation:wobble .85s ease-in-out infinite;display:inline-block}
.plot-harvest{animation:pop-out .32s ease-out forwards}
.card-flash{animation:flash-card .65s ease}
.toast-el{animation:toast-anim 2.8s ease forwards}
.coin-anim{animation:coin-pop .42s ease}
.shake{animation:shake .42s ease}
.sc-fade{animation:fade-in .38s ease}
.bounce-in{animation:bounce-in .5s cubic-bezier(.34,1.56,.64,1)}
.regrow-b{animation:regrow 1.2s ease-in-out infinite}
.pest-plot{animation:pest-blink .44s ease-in-out infinite;cursor:pointer!important}
.timer-flash{animation:timer-flash .55s ease-in-out infinite}
.slide-up{animation:slide-up .3s ease}
.star-float{animation:star-float 2s ease-in-out infinite}
.upg-card{transition:all .18s ease;cursor:pointer}
.upg-card:hover{transform:translateY(-6px) scale(1.04)!important;box-shadow:0 12px 32px rgba(0,0,0,.55)!important}
.upg-card:active{transform:scale(.96)!important}
.seed-btn:active{transform:scale(.92)}
.plot-btn:active{transform:scale(.89)}
.layout{display:flex;gap:14px;padding:14px;align-items:flex-start}
.side-panel{display:flex;flex-direction:column;gap:11px;min-width:198px;flex:0 0 198px}
.farm-area{flex:1;min-width:270px;display:flex;flex-direction:column;align-items:center;gap:11px}
.grid-area{display:grid;grid-template-columns:repeat(4,1fr);gap:9px;width:100%;max-width:440px}
@media(max-width:600px){
.layout{flex-direction:column!important;padding:9px;gap:9px}
.side-panel{flex-direction:row!important;min-width:100%!important;flex:unset!important}
.farm-area{min-width:unset!important;width:100%!important}
.grid-area{max-width:100%!important}
}
@media(max-width:400px){.side-panel{flex-direction:column!important}}
</style>

</head>
<body>
<div id="root"></div>
<script type="text/babel">
const { useState, useEffect, useRef } = React;

const CROPS = {
carrot: { emoji:“🥕”, name:“Carrot”, growTime:20, sellPrice:16, cost:7,  color:”#FF7043”, multi:false },
tomato: { emoji:“🍅”, name:“Tomato”, growTime:30, sellPrice:5,  cost:15, color:”#E53935”, multi:true  },
potato: { emoji:“🥔”, name:“Potato”, growTime:25, sellPrice:69, cost:30, color:”#8D6E63”, multi:false },
};
const DAY_GOALS = [30, 90, 180, 300, 460];
const DAY_SECS  = 150;
const MAX_DAYS  = 5;
const rn = n => Math.floor(Math.random() * n);

const UPG_POOL = [
{ id:“water”, name:“Watering Can”,   emoji:“💧”, desc:“Crops grow 25% faster”          },
{ id:“green”, name:“Greenhouse”,     emoji:“🏗️”, desc:“Crops grow 20% faster (stacks)” },
{ id:“fert”,  name:“Fertilizer”,     emoji:“🌿”, desc:”+$4 to all sell prices”         },
{ id:“deal”,  name:“Market Deal”,    emoji:“🤝”, desc:”+$8 to all sell prices”         },
{ id:“land”,  name:“Extra Land”,     emoji:“🏡”, desc:“Unlock 4 extra plots”           },
{ id:“spud”,  name:“Perennial Spud”, emoji:“✨”, desc:“Potatoes regrow after harvest”  },
{ id:“pest”,  name:“Pesticide”,      emoji:“🧪”, desc:“Immune to pest attacks”         },
{ id:“bulk”,  name:“Bulk Seeds”,     emoji:“🎒”, desc:“All seeds cost $3 less”         },
];

const applyUpg = (s, id) => {
const n = { …s };
if (id===“water”) n.gm *= 0.75; if (id===“green”) n.gm *= 0.80;
if (id===“fert”)  n.sb += 4;    if (id===“deal”)  n.sb += 8;
if (id===“land”)  n.pc  = 16;   if (id===“spud”)  n.mp  = true;
if (id===“pest”)  n.pi  = true;  if (id===“bulk”)  n.cd += 3;
return n;
};
const defStats  = () => ({ gm:1, sb:0, pc:12, mp:false, pi:false, cd:0 });
const randSeeds = () => ({ carrot:rn(5)+1, tomato:rn(5)+1, potato:rn(5)+1 });
const pickUpgs  = (owned) =>
[…UPG_POOL.filter(u => !owned.includes(u.id))].sort(() => Math.random()-.5).slice(0,3);

function FarmGame() {
const [phase, setPhase]       = useState(“intro”);
const [day, setDay]           = useState(1);
const [timer, setTimer]       = useState(DAY_SECS);
const [coins, setCoins]       = useState(15);
const [earned, setEarned]     = useState(0);
const [stats, setStats]       = useState(defStats());
const [ownedIds, setOwnedIds] = useState([]);
const [upgChoices, setUpgChoices] = useState([]);
const [seeds, setSeeds]       = useState(randSeeds());
const [shopTick, setShopTick] = useState(60);
const [plots, setPlots]       = useState(Array(12).fill(null));
const [inv, setInv]           = useState({carrot:0,tomato:0,potato:0});
const [sel, setSel]           = useState(null);
const [hvAnim, setHvAnim]     = useState({});
const [cardFlash, setFlash]   = useState(false);
const [broke, setBroke]       = useState(false);
const [coinAnim, setCoinAnim] = useState(false);
const [event, setEvent]       = useState(null);
const [toast, setToast]       = useState(null);

const earnedRef  = useRef(0);
const statsRef   = useRef(defStats());
const phaseRef   = useRef(“intro”);
const dayRef     = useRef(1);
const ownedRef   = useRef([]);
const coinRef    = useRef(15);
const rainRef    = useRef(false);
const rainIvRef  = useRef(null);
const toastRef   = useRef(null);
const foundRef   = useRef(false);

statsRef.current = stats;
phaseRef.current = phase;
dayRef.current   = day;
ownedRef.current = ownedIds;

const showToast = (msg) => {
setToast(msg);
clearTimeout(toastRef.current);
toastRef.current = setTimeout(() => setToast(null), 2800);
};

const addEarned = (n) => { earnedRef.current += n; setEarned(e => e + n); };

const getCrop = (type) => {
const s = statsRef.current, b = CROPS[type];
return { …b, sellPrice: b.sellPrice + s.sb, cost: Math.max(1, b.cost - s.cd), multi: b.multi || (type===“potato” && s.mp) };
};

const startDay = (d, s, oids) => {
earnedRef.current = 0; rainRef.current = false;
if (rainIvRef.current) clearInterval(rainIvRef.current);
dayRef.current = d; ownedRef.current = oids; statsRef.current = s;
setDay(d); setStats(s); setOwnedIds(oids);
setTimer(DAY_SECS); setEarned(0);
setSeeds(randSeeds()); setShopTick(60);
setPlots(Array(s.pc).fill(null));
setInv({carrot:0,tomato:0,potato:0});
setSel(null); setEvent(null);
setPhase(“playing”);
};

const startGame = () => {
const s = defStats(); coinRef.current = 15; setCoins(15);
startDay(1, s, []);
};

// Day countdown
const timerRef = useRef(null);
timerRef.current = () => {
setTimer(prev => {
if (prev <= 1) {
const passed = earnedRef.current >= DAY_GOALS[dayRef.current - 1];
if (passed) {
dayRef.current >= MAX_DAYS
? setPhase(“victory”)
: (setUpgChoices(pickUpgs(ownedRef.current)), setPhase(“upgrades”));
} else setPhase(“game_over”);
return 0;
}
return prev - 1;
});
};
useEffect(() => {
if (phase !== “playing”) return;
const iv = setInterval(() => timerRef.current(), 1000);
return () => clearInterval(iv);
}, [phase]);

// Shop restock
const shopRef = useRef(null);
shopRef.current = () => {
if (phaseRef.current !== “playing”) return;
setShopTick(prev => {
if (prev <= 1) {
setSeeds(randSeeds()); setFlash(true);
setTimeout(() => setFlash(false), 700);
showToast(“🛒 Shop restocked!”);
return 60;
}
return prev - 1;
});
};
useEffect(() => {
if (phase !== “playing”) return;
const iv = setInterval(() => shopRef.current(), 1000);
return () => clearInterval(iv);
}, [phase]);

// Crop growth
const growRef = useRef(null);
growRef.current = () => {
if (phaseRef.current !== “playing”) return;
const DT = 0.4, rain = rainRef.current ? 2 : 1, gm = statsRef.current.gm;
setPlots(prev => prev.map(plot => {
if (!plot) return plot;
let pT = plot.pestT;
if (pT !== undefined) { pT -= DT; if (pT <= 0) return null; }
if (plot.state === “ready”) return pT !== undefined ? {…plot, pestT:pT} : plot;
const np = (plot.progress||0) + DT * rain / (CROPS[plot.type].growTime * gm);
return np >= 1 ? {…plot, state:“ready”, progress:1, pestT:pT} : {…plot, progress:np, pestT:pT};
}));
};
useEffect(() => {
if (phase !== “playing”) return;
const iv = setInterval(() => growRef.current(), 400);
return () => clearInterval(iv);
}, [phase]);

// Random events
const triggerRef = useRef(null);
triggerRef.current = () => {
if (phaseRef.current !== “playing”) return;
if (Math.random() < 0.55) {
rainRef.current = true; let t = 15;
setEvent({type:“rain”, timer:t});
showToast(“🌧️ Rain shower! Crops grow 2× faster for 15s!”);
if (rainIvRef.current) clearInterval(rainIvRef.current);
rainIvRef.current = setInterval(() => {
t–;
if (t <= 0) { clearInterval(rainIvRef.current); rainRef.current = false; setEvent(null); }
else setEvent(e => e?.type===“rain” ? {…e, timer:t} : e);
}, 1000);
} else {
if (statsRef.current.pi) { showToast(“🧪 Pest spotted — pesticide blocked it!”); return; }
foundRef.current = false;
setPlots(prev => {
const gi = prev.map((p,i) => (p?.state===“growing” && p.pestT===undefined) ? i : -1).filter(x => x >= 0);
if (!gi.length) return prev;
foundRef.current = true;
const n = […prev], idx = gi[rn(gi.length)];
n[idx] = {…n[idx], pestT:8}; return n;
});
setTimeout(() => { if (foundRef.current) showToast(“🐛 Pest attack! Tap the glowing plot to save your crop!”); }, 50);
}
};
useEffect(() => {
if (phase !== “playing”) return;
const tids = [12+rn(13), 50+rn(22), 95+rn(30)].map(s => setTimeout(() => triggerRef.current(), s * 1000));
return () => { tids.forEach(clearTimeout); rainRef.current = false; if (rainIvRef.current) clearInterval(rainIvRef.current); };
}, [phase]);

// Actions
const plant = (i) => {
if (!sel || plots[i] !== null || (seeds[sel]||0) <= 0) return;
const crop = getCrop(sel);
if (coinRef.current < crop.cost) { setBroke(true); setTimeout(() => setBroke(false), 450); showToast(`💸 Need $${crop.cost}!`); return; }
coinRef.current -= crop.cost; setCoins(c => c - crop.cost);
setPlots(prev => { const n = […prev]; n[i] = {type:sel, state:“growing”, progress:0}; return n; });
setSeeds(prev => { const n = {…prev, [sel]:prev[sel]-1}; if (!n[sel]) setSel(null); return n; });
};

const harvest = (i) => {
const p = plots[i]; if (!p || p.state !== “ready” || p.pestT !== undefined) return;
setHvAnim(h => ({…h, [i]:true}));
setTimeout(() => {
const t = p.type;
setInv(prev => ({…prev, [t]:prev[t]+1}));
setPlots(prev => {
const n = […prev], c = getCrop(t);
n[i] = c.multi ? {type:t, state:“growing”, progress:0, regrow:true} : null;
return n;
});
setHvAnim(h => { const n = {…h}; delete n[i]; return n; });
}, 290);
};

const squash = (i) => {
setPlots(prev => {
if (!prev[i] || prev[i].pestT === undefined) return prev;
const n = […prev]; n[i] = {…n[i], pestT:undefined}; return n;
});
showToast(“✅ Pest squashed! Crop saved!”);
};

const sellAll = () => {
const total = Object.entries(inv).reduce((s,[t,c]) => s + c * getCrop(t).sellPrice, 0);
if (!total) return;
coinRef.current += total; setCoins(c => c + total); addEarned(total);
setInv({carrot:0,tomato:0,potato:0}); setCoinAnim(true);
setTimeout(() => setCoinAnim(false), 500);
showToast(`💰 Sold for $${total}!`);
};

const pickUpg = (u) => {
const ns = applyUpg(stats, u.id), noi = […ownedIds, u.id];
ownedRef.current = noi; startDay(day+1, ns, noi);
};

const goal = DAY_GOALS[day-1];
const invT = Object.values(inv).reduce((a,b) => a+b, 0);
const invV = Object.entries(inv).reduce((s,[t,c]) => s + c * getCrop(t).sellPrice, 0);
const timerPct = (timer / DAY_SECS) * 100;
const earnedPct = Math.min((earned / goal) * 100, 100);
const isUrgent = timer <= 30 && timer > 0;
const bg = event?.type===“rain”
? “linear-gradient(160deg,#0d2b3d,#1a4d60,#0d3b2d)”
: “linear-gradient(160deg,#1b3d08,#2a5c10,#1a4a08)”;

const Card = ({children, style={}}) => (
<div className=“bounce-in” style={{background:“rgba(255,255,255,.07)”,backdropFilter:“blur(16px)”,border:“1px solid rgba(255,255,255,.15)”,borderRadius:24,padding:“36px 28px”,maxWidth:400,width:“100%”,textAlign:“center”,color:”#fff”,…style}}>
{children}
</div>
);

// ── INTRO ──────────────────────────────────────────────
if (phase === “intro”) return (
<div style={{minHeight:“100vh”,background:“linear-gradient(160deg,#0d2b00,#1e5008,#0a1f00)”,display:“flex”,alignItems:“center”,justifyContent:“center”,fontFamily:”‘Nunito’,sans-serif”,padding:16}}>
<Card>
<div style={{fontSize:62,marginBottom:6}} className="star-float">🌾</div>
<div style={{fontFamily:”‘Fredoka One’,cursive”,fontSize:34,letterSpacing:2,marginBottom:4}}>Harvest Haven</div>
<div style={{fontSize:11,opacity:.45,letterSpacing:1.2,marginBottom:22,textTransform:“uppercase”}}>5-Day Farming Challenge</div>
<div style={{background:“rgba(0,0,0,.3)”,borderRadius:14,padding:14,marginBottom:18,textAlign:“left”}}>
{[[“⏰”,“2.5 min/day — hit your daily earnings goal”],
[“🌱”,“Buy seeds, grow crops, harvest & sell”],
[“⬆️”,“Pick a power-up upgrade between days”],
[“🌧️”,“Rain boosts growth · 🐛 Pests destroy crops”],
[“🏆”,“Complete all 5 days to win!”]].map(([e,t]) => (
<div key={t} style={{display:“flex”,gap:10,alignItems:“center”,marginBottom:7,fontSize:12}}>
<span style={{fontSize:16,minWidth:22}}>{e}</span>
<span style={{opacity:.82}}>{t}</span>
</div>
))}
</div>
<div style={{display:“flex”,gap:6,justifyContent:“center”,marginBottom:16,flexWrap:“wrap”}}>
{DAY_GOALS.map((g,i) => (
<div key={i} style={{background:“rgba(255,255,255,.08)”,borderRadius:8,padding:“4px 10px”,fontSize:11,opacity:.65}}>
Day {i+1}<br/><strong>${g}</strong>
</div>
))}
</div>
<div style={{fontSize:11,opacity:.4,marginBottom:18,lineHeight:1.8}}>
🥕 Carrot $7→$16  ·  🍅 Tomato $15→$5 ♻️  ·  🥔 Potato $30→$69
</div>
<button onClick={startGame} style={{width:“100%”,background:“linear-gradient(135deg,#76FF03,#33691E)”,border:“none”,borderRadius:14,color:”#fff”,fontFamily:”‘Fredoka One’,cursive”,fontSize:20,padding:“16px”,cursor:“pointer”,boxShadow:“0 4px 22px rgba(118,255,3,.45)”,letterSpacing:1}}>
🌾 Start Farming
</button>
</Card>
</div>
);

// ── UPGRADES ───────────────────────────────────────────
if (phase === “upgrades”) return (
<div style={{minHeight:“100vh”,background:“linear-gradient(160deg,#0d2b00,#1a4a08,#0d2b00)”,display:“flex”,flexDirection:“column”,alignItems:“center”,justifyContent:“center”,fontFamily:”‘Nunito’,sans-serif”,padding:16}}>
<div className=“bounce-in” style={{color:”#fff”,maxWidth:560,width:“100%”,textAlign:“center”}}>
<div style={{fontSize:52,marginBottom:4}}>🎉</div>
<div style={{fontFamily:”‘Fredoka One’,cursive”,fontSize:26,marginBottom:4}}>Day {day} Complete!</div>
<div style={{fontSize:13,opacity:.6,marginBottom:4}}>Earned <strong style={{color:”#FFD54F”}}>${earned}</strong> · Goal was ${goal}</div>
{earned > goal && <div style={{fontSize:12,color:”#76FF03”,marginBottom:10}}>⭐ {earned-goal > goal*.5 ? “Outstanding!” : “Nice work!”} +${earned-goal} banked.</div>}
<div style={{fontSize:12,opacity:.55,marginBottom:4}}>Day {day+1} goal: <strong style={{color:”#CCFF90”}}>${DAY_GOALS[day]}</strong></div>
<div style={{fontFamily:”‘Fredoka One’,cursive”,fontSize:15,color:”#FFD54F”,marginBottom:14}}>⬆️ Choose your upgrade:</div>
<div style={{display:“flex”,gap:12,flexWrap:“wrap”,justifyContent:“center”,marginBottom:16}}>
{upgChoices.map(u => (
<button key={u.id} className=“upg-card” onClick={() => pickUpg(u)}
style={{flex:“1 1 130px”,maxWidth:168,background:“rgba(255,255,255,.09)”,border:“2px solid rgba(255,255,255,.15)”,borderRadius:18,padding:“22px 12px”,color:”#fff”,boxShadow:“0 4px 18px rgba(0,0,0,.35)”,textAlign:“center”}}>
<div style={{fontSize:38,marginBottom:8}}>{u.emoji}</div>
<div style={{fontFamily:”‘Fredoka One’,cursive”,fontSize:14,marginBottom:6}}>{u.name}</div>
<div style={{fontSize:11,opacity:.62,lineHeight:1.45}}>{u.desc}</div>
</button>
))}
{!upgChoices.length && (
<button onClick={() => startDay(day+1, stats, ownedIds)}
style={{background:“linear-gradient(135deg,#76FF03,#33691E)”,border:“none”,borderRadius:14,color:”#fff”,fontFamily:”‘Fredoka One’,cursive”,fontSize:16,padding:“12px 28px”,cursor:“pointer”}}>
Continue to Day {day+1} →
</button>
)}
</div>
</div>
</div>
);

// ── GAME OVER ──────────────────────────────────────────
if (phase === “game_over”) return (
<div style={{minHeight:“100vh”,background:“linear-gradient(160deg,#1a0000,#3d0808,#1a0000)”,display:“flex”,alignItems:“center”,justifyContent:“center”,fontFamily:”‘Nunito’,sans-serif”,padding:16}}>
<Card style={{border:“1px solid rgba(255,100,100,.3)”}}>
<div style={{fontSize:58,marginBottom:8}}>🥀</div>
<div style={{fontFamily:”‘Fredoka One’,cursive”,fontSize:28,color:”#FF5252”,marginBottom:8}}>Farm Failed</div>
<div style={{fontSize:13,opacity:.65,marginBottom:8}}>Day {day} · Earned ${earned} of ${goal}</div>
<div style={{background:“rgba(255,82,82,.1)”,border:“1px solid rgba(255,82,82,.22)”,borderRadius:12,padding:“10px 14px”,marginBottom:24,fontSize:12,lineHeight:1.6,opacity:.85}}>
You needed <strong>${goal - earned}</strong> more to keep the farm going.<br/>
{day > 1 && `You survived ${day-1} day${day > 2 ? "s" : ""} — try again!`}
</div>
<button onClick={startGame}
style={{width:“100%”,background:“linear-gradient(135deg,#FF5252,#B71C1C)”,border:“none”,borderRadius:14,color:”#fff”,fontFamily:”‘Fredoka One’,cursive”,fontSize:18,padding:“14px”,cursor:“pointer”,boxShadow:“0 4px 20px rgba(255,82,82,.42)”}}>
🔄 Try Again
</button>
</Card>
</div>
);

// ── VICTORY ────────────────────────────────────────────
if (phase === “victory”) return (
<div style={{minHeight:“100vh”,background:“linear-gradient(160deg,#1a1200,#3d2e00,#2a4a00)”,display:“flex”,alignItems:“center”,justifyContent:“center”,fontFamily:”‘Nunito’,sans-serif”,padding:16}}>
<Card style={{border:“2px solid rgba(249,168,37,.55)”}}>
<div style={{fontSize:62,marginBottom:6}} className="star-float">🏆</div>
<div style={{fontFamily:”‘Fredoka One’,cursive”,fontSize:30,color:”#FFD54F”,marginBottom:8}}>Farm Master!</div>
<div style={{fontSize:13,opacity:.65,marginBottom:4}}>You conquered all {MAX_DAYS} days!</div>
<div style={{fontSize:30,fontFamily:”‘Fredoka One’,cursive”,color:”#FFD54F”,margin:“12px 0”}}>${coins} final stash</div>
<div style={{fontSize:11,opacity:.45,marginBottom:10}}>Upgrades collected: {ownedIds.length}/{UPG_POOL.length}</div>
{ownedIds.length > 0 && (
<div style={{background:“rgba(0,0,0,.25)”,borderRadius:12,padding:“10px”,marginBottom:18,display:“flex”,flexWrap:“wrap”,gap:6,justifyContent:“center”}}>
{ownedIds.map(id => { const u = UPG_POOL.find(u => u.id === id); return u ? <span key={id} style={{fontSize:11,background:“rgba(255,255,255,.1)”,padding:“2px 9px”,borderRadius:8}}>{u.emoji} {u.name}</span> : null; })}
</div>
)}
<button onClick={startGame}
style={{width:“100%”,background:“linear-gradient(135deg,#F9A825,#E65100)”,border:“none”,borderRadius:14,color:”#fff”,fontFamily:”‘Fredoka One’,cursive”,fontSize:18,padding:“14px”,cursor:“pointer”,boxShadow:“0 4px 22px rgba(249,168,37,.52)”}}>
🌾 Play Again
</button>
</Card>
</div>
);

// ── PLAYING ────────────────────────────────────────────
return (
<div style={{minHeight:“100vh”,background:bg,fontFamily:”‘Nunito’,sans-serif”,color:”#fff”,paddingBottom:32,transition:“background 1.5s ease”}}>
{toast && (
<div className=“toast-el” style={{position:“fixed”,bottom:70,left:“50%”,background:“rgba(12,30,8,.95)”,border:“1px solid rgba(118,255,3,.3)”,color:”#CCFF90”,padding:“9px 20px”,borderRadius:22,fontSize:12,fontWeight:700,zIndex:999,whiteSpace:“nowrap”,backdropFilter:“blur(8px)”,boxShadow:“0 4px 16px rgba(0,0,0,.5)”}}>
{toast}
</div>
)}

```
  {/* Header */}
  <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"10px 14px",background:"rgba(0,0,0,.38)",backdropFilter:"blur(10px)",borderBottom:"1px solid rgba(255,255,255,.07)"}}>
    <div>
      <div style={{fontFamily:"'Fredoka One',cursive",fontSize:16,letterSpacing:.4}}>🌾 Day {day} / {MAX_DAYS}</div>
      <div style={{fontSize:9,opacity:.38,letterSpacing:.5,textTransform:"uppercase"}}>Goal: ${goal}</div>
    </div>
    <div style={{display:"flex",flexDirection:"column",alignItems:"center",flex:1,maxWidth:148,margin:"0 10px"}}>
      <div className={isUrgent ? "timer-flash" : ""} style={{fontFamily:"'Fredoka One',cursive",fontSize:22,color:isUrgent?"#FF5722":"#fff",lineHeight:1}}>
        {Math.floor(timer/60)}:{String(timer%60).padStart(2,"0")}
      </div>
      <div style={{width:"100%",height:4,background:"rgba(255,255,255,.12)",borderRadius:2,overflow:"hidden",marginTop:3}}>
        <div style={{height:"100%",width:`${timerPct}%`,background:isUrgent?"linear-gradient(90deg,#FF5722,#FF8A65)":"linear-gradient(90deg,#76FF03,#CCFF90)",borderRadius:2,transition:"width 1s linear"}}/>
      </div>
    </div>
    <div className={coinAnim ? "coin-anim" : ""} style={{background:"linear-gradient(135deg,#F9A825,#E65100)",borderRadius:20,padding:"6px 14px",fontSize:16,fontWeight:800,boxShadow:"0 2px 12px rgba(245,127,23,.5)",fontFamily:"'Fredoka One',cursive",whiteSpace:"nowrap"}}>
      ${coins}
    </div>
  </div>

  {/* Earnings bar */}
  <div style={{padding:"7px 14px 0",display:"flex",alignItems:"center",gap:8}}>
    <span style={{fontSize:10,opacity:.5,whiteSpace:"nowrap"}}>💰 ${earned}</span>
    <div style={{flex:1,height:7,background:"rgba(255,255,255,.1)",borderRadius:4,overflow:"hidden"}}>
      <div style={{height:"100%",width:`${earnedPct}%`,background:earnedPct>=100?"linear-gradient(90deg,#76FF03,#33691E)":"linear-gradient(90deg,#FFD54F,#FF8F00)",borderRadius:4,transition:"width .4s ease"}}/>
    </div>
    <span style={{fontSize:10,opacity:.5,whiteSpace:"nowrap"}}>🎯 ${goal}</span>
  </div>

  {/* Event banner */}
  {event && (
    <div className="slide-up" style={{margin:"6px 14px 0",padding:"7px 12px",borderRadius:11,background:event.type==="rain"?"rgba(33,150,243,.2)":"rgba(255,87,34,.2)",border:`1px solid ${event.type==="rain"?"rgba(33,150,243,.4)":"rgba(255,87,34,.4)"}`,fontSize:11,fontWeight:700,display:"flex",alignItems:"center",gap:7}}>
      <span style={{fontSize:15}}>{event.type==="rain" ? "🌧️" : "🐛"}</span>
      <span style={{flex:1}}>
        {event.type==="rain" ? `Rain shower! 2× grow speed · ${event.timer}s left` : "Pest attack! Tap the glowing 🐛 plot to save your crop!"}
      </span>
    </div>
  )}

  <div className="layout">
    {/* Side Panel */}
    <div className="side-panel">

      {/* Seed Shop */}
      <div className={cardFlash ? "card-flash" : ""} style={{background:"rgba(255,248,220,.1)",backdropFilter:"blur(12px)",borderRadius:15,padding:12,border:"1px solid rgba(255,255,255,.12)"}}>
        <div style={{fontFamily:"'Fredoka One',cursive",fontSize:14,marginBottom:8,display:"flex",justifyContent:"space-between",alignItems:"center"}}>
          <span>🛒 Seeds</span>
          <span style={{fontSize:10,color:shopTick<=10?"#FF8A65":"#CCFF90",background:"rgba(118,255,3,.1)",padding:"1px 7px",borderRadius:9}}>{shopTick}s</span>
        </div>
        <div style={{height:3,background:"rgba(255,255,255,.1)",borderRadius:2,marginBottom:9,overflow:"hidden"}}>
          <div style={{height:"100%",width:`${(shopTick/60)*100}%`,background:shopTick<=10?"linear-gradient(90deg,#FF5722,#FF8A65)":"linear-gradient(90deg,#76FF03,#CCFF90)",borderRadius:2,transition:"width 1s linear"}}/>
        </div>
        {Object.entries(CROPS).map(([type, b]) => {
          const crop = getCrop(type), active = sel===type, noS = !seeds[type], noM = coinRef.current < crop.cost, dis = noS || noM;
          return (
            <button key={type} className="seed-btn" onClick={() => !dis && setSel(active ? null : type)}
              style={{display:"flex",alignItems:"center",gap:7,width:"100%",background:active?"rgba(118,255,3,.18)":"rgba(255,255,255,.07)",border:active?"2px solid #76FF03":"2px solid transparent",borderRadius:9,padding:"7px 9px",color:"#fff",cursor:dis?"not-allowed":"pointer",marginBottom:5,opacity:dis?.38:1,fontFamily:"'Nunito',sans-serif"}}>
              <span style={{fontSize:18}}>{b.emoji}</span>
              <div style={{display:"flex",flexDirection:"column",flex:1,textAlign:"left"}}>
                <span style={{fontSize:11,fontWeight:700}}>{b.name}</span>
                <span style={{fontSize:9,opacity:.6}}>×{seeds[type]} · ${crop.cost}</span>
              </div>
              {active && <span style={{fontSize:11,color:"#76FF03"}}>✓</span>}
              {noM && !noS && <span style={{fontSize:9,color:"#FF8A65"}}>💸</span>}
            </button>
          );
        })}
        {sel
          ? <div style={{fontSize:10,textAlign:"center",color:"#CCFF90",background:"rgba(118,255,3,.1)",borderRadius:7,padding:"5px",marginTop:4}}>🌱 Tap a plot · ${getCrop(sel).cost}</div>
          : <div style={{fontSize:10,textAlign:"center",opacity:.3,marginTop:4}}>Select a seed above</div>
        }
      </div>

      {/* Market */}
      <div style={{background:"rgba(255,248,220,.1)",backdropFilter:"blur(12px)",borderRadius:15,padding:12,border:"1px solid rgba(255,255,255,.12)"}}>
        <div style={{fontFamily:"'Fredoka One',cursive",fontSize:14,marginBottom:8}}>💰 Market</div>
        {Object.entries(CROPS).map(([type, b]) => {
          const crop = getCrop(type);
          return (
            <div key={type} style={{display:"flex",justifyContent:"space-between",alignItems:"center",fontSize:11,marginBottom:5,background:inv[type]>0?"rgba(255,220,80,.1)":"transparent",borderRadius:7,padding:"3px 5px"}}>
              <span style={{flex:1}}>{b.emoji} {b.name}</span>
              <span style={{fontWeight:800,minWidth:22,textAlign:"center",color:inv[type]>0?"#FFD54F":"#fff"}}>×{inv[type]}</span>
              <span style={{opacity:.5,fontSize:9,minWidth:40,textAlign:"right"}}>${crop.sellPrice}{crop.multi?"♻":""}</span>
            </div>
          );
        })}
        <div style={{height:1,background:"rgba(255,255,255,.1)",margin:"7px 0"}}/>
        <div style={{display:"flex",justifyContent:"space-between",fontSize:12,fontWeight:700,marginBottom:8}}>
          <span>Total</span>
          <span style={{color:"#FFD54F",fontFamily:"'Fredoka One',cursive",fontSize:13}}>${invV}</span>
        </div>
        <button className="sell-btn" onClick={sellAll} disabled={!invT}
          style={{width:"100%",background:invT?"linear-gradient(135deg,#F9A825,#E65100)":"rgba(255,255,255,.1)",border:"none",borderRadius:9,color:"#fff",fontFamily:"'Fredoka One',cursive",fontSize:11,padding:"9px",cursor:invT?"pointer":"not-allowed",boxShadow:invT?"0 2px 12px rgba(230,81,0,.4)":"none",opacity:invT?1:.4,transition:"all .2s"}}>
          {invT ? `Sell All (${invT})` : "Nothing to sell"}
        </button>
      </div>

      {/* Active upgrades */}
      {ownedIds.length > 0 && (
        <div style={{background:"rgba(255,248,220,.06)",borderRadius:13,padding:"9px 11px",border:"1px solid rgba(255,255,255,.07)"}}>
          <div style={{fontFamily:"'Fredoka One',cursive",fontSize:12,marginBottom:6,opacity:.72}}>⬆️ Upgrades</div>
          {ownedIds.map(id => { const u = UPG_POOL.find(u => u.id === id); return u ? <div key={id} style={{fontSize:10,opacity:.58,marginBottom:2}}>{u.emoji} {u.name}</div> : null; })}
        </div>
      )}
    </div>

    {/* Farm Grid */}
    <div className="farm-area">
      <div style={{fontSize:11,textAlign:"center",color:sel?"#CCFF90":"rgba(255,255,255,.35)",background:"rgba(0,0,0,.22)",padding:"5px 13px",borderRadius:15,fontWeight:700}}>
        {sel ? `🌱 ${CROPS[sel].name} · $${getCrop(sel).cost} each` : "Select a seed to plant"}
      </div>

      <div className={`grid-area${broke ? " shake" : ""}`}>
        {plots.map((plot, i) => {
          const isReady = plot?.state==="ready" && plot.pestT===undefined;
          const hasPest = plot?.pestT !== undefined;
          const isHarv  = hvAnim[i];
          const isReg   = plot?.regrow;
          return (
            <button key={i}
              className={`plot-btn${isReady && !isHarv ? " plot-ready" : ""}${isHarv ? " plot-harvest" : ""}${hasPest ? " pest-plot" : ""}`}
              onClick={() => hasPest ? squash(i) : isReady ? harvest(i) : plant(i)}
              style={{aspectRatio:"1",background:hasPest?"rgba(100,40,0,.72)":isReady?"rgba(101,67,33,.72)":"rgba(80,50,20,.55)",border:"2px solid rgba(120,80,30,.5)",borderRadius:11,display:"flex",alignItems:"center",justifyContent:"center",flexDirection:"column",cursor:"pointer",position:"relative",boxShadow:"inset 0 2px 6px rgba(0,0,0,.35)",overflow:"hidden",minHeight:60,padding:5}}>
              {!plot && <span style={{fontSize:22,color:sel?"rgba(118,255,3,.5)":"rgba(255,255,255,.18)",fontWeight:800}}>{sel ? "＋" : "·"}</span>}
              {plot?.state==="growing" && !hasPest && (
                <div style={{display:"flex",flexDirection:"column",alignItems:"center",gap:3,width:"85%"}}>
                  {isReg && <span className="regrow-b" style={{position:"absolute",top:3,right:3,fontSize:8,background:"rgba(229,57,53,.7)",borderRadius:4,padding:"1px 3px"}}>♻️</span>}
                  <span style={{fontSize:20,filter:`grayscale(${1-(plot.progress||0)})`,opacity:.45+.55*(plot.progress||0)}}>{CROPS[plot.type].emoji}</span>
                  <div style={{width:"100%",height:4,background:"rgba(255,255,255,.15)",borderRadius:2,overflow:"hidden"}}>
                    <div style={{height:"100%",width:`${(plot.progress||0)*100}%`,background:CROPS[plot.type].color,borderRadius:2,transition:"width .4s"}}/>
                  </div>
                  <span style={{fontSize:7,opacity:.4}}>{Math.max(0, Math.ceil(CROPS[plot.type].growTime * statsRef.current.gm * (1-(plot.progress||0))))}s</span>
                </div>
              )}
              {hasPest && (
                <div style={{display:"flex",flexDirection:"column",alignItems:"center",gap:1}}>
                  <span style={{fontSize:22}}>🐛</span>
                  <span style={{fontSize:8,color:"#FF8A65",fontWeight:800}}>{Math.ceil(Math.max(0, plot.pestT))}s!</span>
                </div>
              )}
              {isReady && !isHarv && (
                <div style={{display:"flex",flexDirection:"column",alignItems:"center",gap:1}}>
                  {isReg && <span style={{position:"absolute",top:3,right:3,fontSize:8}}>♻️</span>}
                  <span className="ready-emoji" style={{fontSize:26}}>{CROPS[plot.type].emoji}</span>
                  <span style={{fontSize:7,color:"#76FF03",fontWeight:800,letterSpacing:.5}}>READY!</span>
                </div>
              )}
            </button>
          );
        })}
      </div>

      <div style={{display:"flex",flexWrap:"wrap",gap:4,justifyContent:"center"}}>
        {Object.entries(CROPS).map(([t, b]) => {
          const c = getCrop(t);
          return <span key={t} style={{fontSize:9,color:"rgba(255,255,255,.4)",background:"rgba(0,0,0,.2)",padding:"2px 7px",borderRadius:9}}>{b.emoji} ${c.cost}→${c.sellPrice} · {Math.round(b.growTime * stats.gm)}s{c.multi?"♻":""}</span>;
        })}
      </div>

      {plots.some(p => p?.state==="ready" && !p?.pestT) && (
        <div style={{fontSize:11,color:"#76FF03",fontWeight:700,background:"rgba(118,255,3,.1)",padding:"5px 13px",borderRadius:15,border:"1px solid rgba(118,255,3,.28)"}}>
          🌟 Crops ready — tap to harvest!
        </div>
      )}
    </div>
  </div>
</div>
```

);
}

ReactDOM.createRoot(document.getElementById(“root”)).render(<FarmGame />);
</script>

</body>
</html>
