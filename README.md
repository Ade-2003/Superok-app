# Superok-app
<!DOCTYPE html>

<html lang="ro">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>SuperOk Bistrița — Școala de Șoferi</title>
  <meta name="description" content="SuperOk Bistrița — Aplicație școală de șoferi"/>
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🚗</text></svg>"/>
  <!-- React + Babel -->
  <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <style>
    *{margin:0;padding:0;box-sizing:border-box}
    body{font-family:'Segoe UI',Arial,sans-serif;background:#F5F5F5;color:#111}
    input,select,textarea,button{font-family:inherit}
    ::-webkit-scrollbar{width:4px}
    ::-webkit-scrollbar-track{background:#f0f0f0}
    ::-webkit-scrollbar-thumb{background:#ccc;border-radius:2px}
    #root{min-height:100vh}
  </style>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel" data-presets="react">
const { useState, useEffect, useRef } = React;

// ── SUPABASE ─────────────────────────────────────────────────────────────────
const SUPA_URL = “https://ywpwqkachruntzibdldz.supabase.co”;
const SUPA_KEY = “sb_publishable_EkesnysWhpXcxMbZ0UkycQ_dSdoTs2V”;

let supabase = null;
function getSupabase() {
if (supabase) return supabase;
// Încărcăm SDK-ul dinamic din CDN
return supabase;
}

// Helper fetch direct pe REST API Supabase (fără npm)
async function sbGet(table, query = “”) {
const res = await fetch(`${SUPA_URL}/rest/v1/${table}${query}`, {
headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}`, “Content-Type”: “application/json” },
});
if (!res.ok) throw new Error(await res.text());
return res.json();
}
async function sbInsert(table, body) {
const res = await fetch(`${SUPA_URL}/rest/v1/${table}`, {
method: “POST”,
headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}`, “Content-Type”: “application/json”, Prefer: “return=representation” },
body: JSON.stringify(body),
});
if (!res.ok) throw new Error(await res.text());
return res.json();
}
async function sbUpdate(table, id, body) {
const res = await fetch(`${SUPA_URL}/rest/v1/${table}?id=eq.${id}`, {
method: “PATCH”,
headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}`, “Content-Type”: “application/json”, Prefer: “return=representation” },
body: JSON.stringify(body),
});
if (!res.ok) throw new Error(await res.text());
return res.json();
}
async function sbDelete(table, id) {
const res = await fetch(`${SUPA_URL}/rest/v1/${table}?id=eq.${id}`, {
method: “DELETE”,
headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}` },
});
if (!res.ok) throw new Error(await res.text());
}
async function sbUpsert(table, body, onConflict) {
const res = await fetch(`${SUPA_URL}/rest/v1/${table}${onConflict ? "?on_conflict=" + onConflict : ""}`, {
method: “POST”,
headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}`, “Content-Type”: “application/json”, Prefer: “resolution=merge-duplicates,return=representation” },
body: JSON.stringify(body),
});
if (!res.ok) throw new Error(await res.text());
return res.json();
}

// Upload fișier în Supabase Storage
async function sbUploadFile(bucket, path, file) {
const res = await fetch(`${SUPA_URL}/storage/v1/object/${bucket}/${path}`, {
method: “POST”,
headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}`, “Content-Type”: file.type },
body: file,
});
if (!res.ok) throw new Error(await res.text());
return `${SUPA_URL}/storage/v1/object/public/${bucket}/${path}`;
}

const CLRED   = “#111111”;
const CLRED2  = “#444444”;
const CLGRN   = “#666666”;
const CLBLU   = “#333333”;
const CLBG    = “#F5F5F5”;
const CLCARD  = “rgba(0,0,0,0.04)”;
const CLBORD  = “rgba(0,0,0,0.09)”;
const CLACCENT = “#D0161B”;

const ROLES = [
{ id:“elev”,       label:“Elev”,                icon:“🎓”, clr:CLACCENT },
{ id:“instructor”, label:“Mașină / Instructor”,  icon:“🚗”, clr:”#FFFFFF” },
{ id:“birou”,      label:“Birou”,                icon:“🏢”, clr:”#AAAAAA” },
];

// Parole implicite: numele mic cu minuscule (ex: “alexandru”, “maria”)
const ELEVI_INIT = [
{ id:1, nume:“Alexandru Mihai”, parola:“alexandru”, ore:12, oreMax:20, teorie:85, masina:“B-01-SUK”, status:“activ”,    plata:“achitat”, data:“Ian 2026”, telefon:”+40 721 234 567” },
{ id:2, nume:“Maria Ionescu”,   parola:“maria”,     ore:18, oreMax:20, teorie:92, masina:“B-02-SUK”, status:“activ”,    plata:“achitat”, data:“Feb 2026”, telefon:”+40 732 111 222” },
{ id:3, nume:“Andrei Pop”,      parola:“andrei”,    ore:6,  oreMax:20, teorie:45, masina:“B-01-SUK”, status:“activ”,    plata:“restant”, data:“Mar 2026”, telefon:”+40 743 333 444” },
{ id:4, nume:“Elena Mureșan”,   parola:“elena”,     ore:20, oreMax:20, teorie:98, masina:“B-03-SUK”, status:“promovat”, plata:“achitat”, data:“Dec 2025”, telefon:”+40 754 555 666” },
{ id:5, nume:“Bogdan Rus”,      parola:“bogdan”,    ore:3,  oreMax:20, teorie:22, masina:“B-02-SUK”, status:“activ”,    plata:“restant”, data:“Apr 2026”, telefon:”+40 765 777 888” },
];

// Parole fixe pentru instructor și birou
const CREDENTIALS = {
birou: “superok2026”,
};

// Parole instructori — fiecare cu parola lui
const INSTRUCTORI_INIT = [
{ id:1, nume:“Prof. Vaida Adrian”,  parola:“vaida123”,   telefon:”+40 744 100 200”, autorizatie:“AUT-2024-0892”, valabilitate:“31.12.2026”, masina:“B-01-SUK”, foto:null },
{ id:2, nume:“Instr. Moldovan Ion”, parola:“moldovan123”, telefon:”+40 744 300 400”, autorizatie:“AUT-2023-0451”, valabilitate:“30.06.2026”, masina:“B-02-SUK”, foto:null },
];

const MASINI = [
{ id:“B-01-SUK”, model:“Dacia Logan”,     an:2022, airtag:“AT-A1B2C3”, km:47230, status:“în mișcare” },
{ id:“B-02-SUK”, model:“Volkswagen Golf”, an:2021, airtag:“AT-D4E5F6”, km:61540, status:“parcat”     },
{ id:“B-03-SUK”, model:“Škoda Octavia”,   an:2023, airtag:“AT-G7H8I9”, km:28910, status:“parcat”     },
];

const SESIUNI = [
{ elev:“Alexandru Mihai”, masina:“B-01-SUK”, data:“21 Mai”, durata:“1h 20m”, km:18.4 },
{ elev:“Maria Ionescu”,   masina:“B-02-SUK”, data:“21 Mai”, durata:“1h 05m”, km:14.2 },
{ elev:“Andrei Pop”,      masina:“B-01-SUK”, data:“20 Mai”, durata:“0h 50m”, km:11.7 },
{ elev:“Bogdan Rus”,      masina:“B-02-SUK”, data:“19 Mai”, durata:“1h 10m”, km:15.3 },
];

const PROGRAMARI = [
{ zi:“LUN”, data:“19”, ora:“10:00”, tip:“Practică”, elev:“Alexandru Mihai”, masina:“B-01-SUK” },
{ zi:“LUN”, data:“19”, ora:“12:00”, tip:“Practică”, elev:“Maria Ionescu”,   masina:“B-02-SUK” },
{ zi:“MIE”, data:“21”, ora:“14:30”, tip:“Teorie”,   elev:“Andrei Pop”,      masina:”-”        },
{ zi:“MIE”, data:“21”, ora:“16:00”, tip:“Practică”, elev:“Bogdan Rus”,      masina:“B-03-SUK” },
{ zi:“SAM”, data:“24”, ora:“09:00”, tip:“Practică”, elev:“Alexandru Mihai”, masina:“B-01-SUK” },
];

const ROUTE_GPS = [
{lat:47.1365,lng:24.4974},{lat:47.1380,lng:24.5010},{lat:47.1410,lng:24.5030},
{lat:47.1440,lng:24.5010},{lat:47.1450,lng:24.4970},{lat:47.1440,lng:24.4930},
{lat:47.1410,lng:24.4900},{lat:47.1380,lng:24.4890},{lat:47.1350,lng:24.4900},
{lat:47.1325,lng:24.4930},{lat:47.1320,lng:24.4970},{lat:47.1335,lng:24.5010},
{lat:47.1355,lng:24.5020},{lat:47.1365,lng:24.4974},
];

function lerpNum(a, b, t) { return a + (b - a) * t; }
function fmtTime(s) {
const hh = String(Math.floor(s / 3600)).padStart(2,“0”);
const mm = String(Math.floor((s % 3600) / 60)).padStart(2,“0”);
const ss = String(s % 60).padStart(2,“0”);
return hh + “:” + mm + “:” + ss;
}
function fmtShort(s) {
const h = Math.floor(s / 3600), m = Math.floor((s % 3600) / 60);
return h > 0 ? h + “h “ + m + “m” : m + “m”;
}

// ── Leaflet map ──────────────────────────────────────────────────────────────
// ── SVG MAP — Bistrița, fără tile-uri externe ────────────────────────────────
// Coordonate GPS reale → pixel în SVG 400×320
const MAP_W = 400, MAP_H = 320;
const LAT_MIN = 47.110, LAT_MAX = 47.165;
const LNG_MIN = 24.470, LNG_MAX = 24.540;

function gpsToXY(lat, lng) {
const x = ((lng - LNG_MIN) / (LNG_MAX - LNG_MIN)) * MAP_W;
const y = MAP_H - ((lat - LAT_MIN) / (LAT_MAX - LAT_MIN)) * MAP_H;
return { x: parseFloat(x.toFixed(1)), y: parseFloat(y.toFixed(1)) };
}

// Traseu mașina activă
const ROUTE_XY = ROUTE_GPS.map(p => gpsToXY(p.lat, p.lng));

// Strazi principale Bistrița (coordonate GPS aproximate)
const STREETS = [
// Strada Principală / Liviu Rebreanu
{ pts: [[47.150,24.488],[47.145,24.490],[47.140,24.492],[47.135,24.494],[47.130,24.496],[47.125,24.500]], w:3, clr:”#d4c8b8” },
// Bd. Independenței
{ pts: [[47.148,24.480],[47.146,24.490],[47.144,24.500],[47.142,24.510],[47.140,24.520]], w:3, clr:”#d4c8b8” },
// Str. Bistricioarei
{ pts: [[47.155,24.490],[47.150,24.492],[47.145,24.495],[47.140,24.498]], w:2, clr:”#ddd3c5” },
// Calea Moldovei
{ pts: [[47.130,24.485],[47.133,24.492],[47.136,24.498],[47.139,24.505]], w:2, clr:”#ddd3c5” },
// Str. Decebal
{ pts: [[47.142,24.482],[47.143,24.488],[47.144,24.494],[47.145,24.500]], w:2, clr:”#ddd3c5” },
// Str. Gării
{ pts: [[47.135,24.500],[47.137,24.505],[47.139,24.510],[47.141,24.515],[47.143,24.520]], w:2, clr:”#ddd3c5” },
// Aleea Parcului
{ pts: [[47.148,24.497],[47.147,24.500],[47.146,24.503],[47.145,24.506]], w:1.5, clr:”#e5ddd0” },
// Str. Andrei Mureșanu
{ pts: [[47.140,24.502],[47.142,24.504],[47.144,24.506],[47.146,24.508]], w:1.5, clr:”#e5ddd0” },
// Bd. 1 Decembrie
{ pts: [[47.136,24.488],[47.138,24.492],[47.140,24.496],[47.142,24.500],[47.144,24.504]], w:2.5, clr:”#d4c8b8” },
// Râul Bistrița
{ pts: [[47.125,24.480],[47.128,24.490],[47.131,24.498],[47.134,24.508],[47.137,24.518]], w:4, clr:”#a8c8e8” },
];

// Zone / blocuri
const ZONES = [
{ lat:47.145, lng:24.495, w:18, h:12, clr:”#ede8e0”, label:“Centru” },
{ lat:47.138, lng:24.500, w:14, h:10, clr:”#e8e4dc”, label:“Piața” },
{ lat:47.152, lng:24.485, w:20, h:14, clr:”#eae5dd”, label:“Nord” },
{ lat:47.130, lng:24.490, w:16, h:10, clr:”#e8e4dc”, label:“Sud” },
{ lat:47.143, lng:24.515, w:18, h:12, clr:”#eae5dd”, label:“Est” },
{ lat:47.148, lng:24.508, w:12, h:8,  clr:”#e5e0d8”, label:“Parc” },
{ lat:47.135, lng:24.480, w:14, h:10, clr:”#e8e4dc”, label:“Vest” },
{ lat:47.140, lng:24.510, w:10, h:8,  clr:”#eae5dd”, label:”” },
{ lat:47.128, lng:24.502, w:12, h:8,  clr:”#e5e0d8”, label:”” },
{ lat:47.155, lng:24.500, w:16, h:10, clr:”#eae5dd”, label:”” },
];

// Parc verde
const PARKS = [
{ lat:47.147, lng:24.503, w:14, h:10 },
{ lat:47.133, lng:24.495, w:10, h:7  },
];

function LiveMap({ simActive, simPos, simTrail, mapHeight, masini: masiniProp }) {
const carsToShow = masiniProp || MASINI_STATIC;

// Poziții fixe pt mașini statice
const carPositions = [
{ id:“B-01-SUK”, lat:47.1365, lng:24.4974 },
{ id:“B-02-SUK”, lat:47.1480, lng:24.5080 },
{ id:“B-03-SUK”, lat:47.1260, lng:24.4820 },
];

const activePos = simActive && simPos ? simPos : { lat:47.1365, lng:24.4974 };
const activeXY  = gpsToXY(activePos.lat, activePos.lng);

// Trail points
const trailPts = simActive && simTrail && simTrail.length > 1
? simTrail.map(p => gpsToXY(p.lat, p.lng))
: [];

const h = mapHeight || 240;
const scale = h / MAP_H;
const w = MAP_W * scale;

return (
<div style={{ width:“100%”, borderRadius:16, overflow:“hidden”, border:“1px solid “ + CLBORD, position:“relative”, background:”#f0ebe3” }}>
<svg width=“100%” viewBox={`0 0 ${MAP_W} ${MAP_H}`} style={{ display:“block” }}>
{/* Fundal */}
<rect width={MAP_W} height={MAP_H} fill="#f0ebe3"/>

```
    {/* Zone blocuri */}
    {ZONES.map((z,i) => {
      const c = gpsToXY(z.lat, z.lng);
      return <rect key={i} x={c.x - z.w/2} y={c.y - z.h/2} width={z.w} height={z.h} rx="2" fill={z.clr} stroke="#e0d8cc" strokeWidth="0.5"/>;
    })}

    {/* Parcuri */}
    {PARKS.map((p,i) => {
      const c = gpsToXY(p.lat, p.lng);
      return <rect key={i} x={c.x - p.w/2} y={c.y - p.h/2} width={p.w} height={p.h} rx="3" fill="#c8ddb8" stroke="#b8cda8" strokeWidth="0.5"/>;
    })}

    {/* Strazi */}
    {STREETS.map((st,i) => {
      const pts = st.pts.map(([lat,lng]) => { const p = gpsToXY(lat,lng); return p.x+","+p.y; }).join(" ");
      return (
        <g key={i}>
          <polyline points={pts} fill="none" stroke="#c8bfb0" strokeWidth={st.w+1.5} strokeLinecap="round" strokeLinejoin="round"/>
          <polyline points={pts} fill="none" stroke={st.clr} strokeWidth={st.w} strokeLinecap="round" strokeLinejoin="round"/>
        </g>
      );
    })}

    {/* Ruta simulare (cercul) */}
    {(() => {
      const pts = ROUTE_XY.map(p => p.x+","+p.y).join(" ");
      return <polyline points={pts} fill="none" stroke={CLACCENT+"33"} strokeWidth="2" strokeDasharray="4,3"/>;
    })()}

    {/* Trail sesiune activă */}
    {trailPts.length > 1 && (
      <polyline points={trailPts.map(p=>p.x+","+p.y).join(" ")} fill="none" stroke={CLACCENT} strokeWidth="3" strokeLinecap="round" strokeLinejoin="round" opacity="0.85"/>
    )}

    {/* Label Bistrița centru */}
    {(() => { const c = gpsToXY(47.1430, 24.4970); return <text x={c.x} y={c.y} textAnchor="middle" fontSize="8" fill="#9a8f82" fontWeight="600" fontFamily="Arial">Bistrița</text>; })()}

    {/* Mașini statice (B-02, B-03) */}
    {carPositions.filter(cp => cp.id !== "B-01-SUK").map((cp, i) => {
      const p = gpsToXY(cp.lat, cp.lng);
      return (
        <g key={i}>
          <circle cx={p.x} cy={p.y} r="9" fill={CLACCENT} stroke="#fff" strokeWidth="1.5" opacity="0.9"/>
          <text x={p.x} y={p.y+4} textAnchor="middle" fontSize="9">🚗</text>
          <rect x={p.x-14} y={p.y+10} width="28" height="9" rx="2" fill="rgba(255,255,255,0.92)" stroke={CLACCENT} strokeWidth="0.5"/>
          <text x={p.x} y={p.y+17} textAnchor="middle" fontSize="5.5" fill={CLACCENT} fontWeight="700" fontFamily="Arial">{cp.id}</text>
        </g>
      );
    })}

    {/* Mașina activă B-01 — cu puls când e în mișcare */}
    {simActive && (
      <>
        <circle cx={activeXY.x} cy={activeXY.y} r="14" fill={CLACCENT} opacity="0.15">
          <animate attributeName="r" values="10;18;10" dur="2s" repeatCount="indefinite"/>
          <animate attributeName="opacity" values="0.2;0;0.2" dur="2s" repeatCount="indefinite"/>
        </circle>
      </>
    )}
    {(() => {
      const p = activeXY;
      return (
        <g>
          <circle cx={p.x} cy={p.y} r="11" fill="#3B82F6" stroke="#fff" strokeWidth="2"/>
          <text x={p.x} y={p.y+4} textAnchor="middle" fontSize="10">📱</text>
          <rect x={p.x-18} y={p.y+12} width="36" height="9" rx="2" fill="rgba(255,255,255,0.95)" stroke="#3B82F6" strokeWidth="0.5"/>
          <text x={p.x} y={p.y+19} textAnchor="middle" fontSize="5.5" fill="#3B82F6" fontWeight="700" fontFamily="Arial">B-01-SUK {simActive?"●":""}</text>
        </g>
      );
    })()}

    {/* Busola */}
    <g transform={`translate(${MAP_W-22}, 16)`}>
      <circle cx="0" cy="0" r="10" fill="rgba(255,255,255,0.9)" stroke="#ddd" strokeWidth="0.5"/>
      <text x="0" y="-3" textAnchor="middle" fontSize="7" fill="#666" fontWeight="700">N</text>
      <line x1="0" y1="-8" x2="0" y2="-2" stroke={CLACCENT} strokeWidth="1.5" strokeLinecap="round"/>
      <line x1="0" y1="2" x2="0" y2="8" stroke="#bbb" strokeWidth="1" strokeLinecap="round"/>
    </g>

    {/* Scară */}
    <g transform={`translate(12, ${MAP_H-14})`}>
      <rect x="0" y="0" width="30" height="4" rx="1" fill="#888"/>
      <rect x="0" y="0" width="15" height="4" rx="0" fill="#fff"/>
      <text x="0" y="-2" fontSize="5" fill="#666">0</text>
      <text x="28" y="-2" fontSize="5" fill="#666">1km</text>
    </g>
  </svg>

  {/* Badge LIVE */}
  {simActive && (
    <div style={{ position:"absolute", top:10, left:10, display:"flex", alignItems:"center", gap:5, background:"rgba(255,255,255,0.95)", border:"1px solid "+CLACCENT+"44", borderRadius:20, padding:"4px 10px", boxShadow:"0 2px 8px rgba(0,0,0,0.1)" }}>
      <div style={{ width:6, height:6, borderRadius:"50%", background:CLACCENT, animation:"liveDot 1s infinite" }}/>
      <span style={{ fontSize:10, color:CLACCENT, fontWeight:700, letterSpacing:1 }}>LIVE GPS</span>
    </div>
  )}

  {/* Stats overlay când e activ */}
  {simActive && (
    <div style={{ position:"absolute", bottom:10, right:10, background:"rgba(255,255,255,0.95)", border:"1px solid #eee", borderRadius:10, padding:"6px 12px", boxShadow:"0 2px 8px rgba(0,0,0,0.1)", display:"flex", gap:12 }}>
      <div style={{ textAlign:"center" }}>
        <div style={{ fontSize:13, fontWeight:900, color:CLACCENT }}>Live</div>
        <div style={{ fontSize:9, color:"#888" }}>GPS</div>
      </div>
    </div>
  )}
</div>
```

);
}

// Variabilă pentru mașini statice accesibilă în LiveMap
const MASINI_STATIC = [
{ id:“B-01-SUK”, model:“Dacia Logan”,     an:2022, airtag:“AT-A1B2C3”, km:47230, status:“parcat”, culoare:“Alb”,   foto:null },
{ id:“B-02-SUK”, model:“Volkswagen Golf”, an:2021, airtag:“AT-D4E5F6”, km:61540, status:“parcat”, culoare:“Negru”, foto:null },
{ id:“B-03-SUK”, model:“Škoda Octavia”,   an:2023, airtag:“AT-G7H8I9”, km:28910, status:“parcat”, culoare:“Gri”,   foto:null },
];

// ── LOGIN ────────────────────────────────────────────────────────────────────
function LoginPage({ onLogin, elevi, instructori }) {
const [step, setStep]         = useState(“role”);   // role | selectElev | selectInstr | password
const [selRole, setSelRole]   = useState(null);
const [selElev, setSelElev]   = useState(””);
const [selInstr, setSelInstr] = useState(null); // instructor object
const [parola, setParola]     = useState(””);
const [eroare, setEroare]     = useState(””);
const [loading, setLoading]   = useState(false);
const [showPass, setShowPass] = useState(false);

const Logo = () => (
<div style={{ margin:“0 auto 12px”, width:80, height:80 }}>
<svg viewBox="0 0 200 200" width="80" height="80">
<rect x="30" y="30" width="115" height="115" rx="18" transform="rotate(45 87.5 87.5)" fill="none" stroke={CLACCENT} strokeWidth="14"/>
<polyline points="52,100 80,130 130,68" fill="none" stroke="#111" strokeWidth="16" strokeLinecap="round" strokeLinejoin="round"/>
</svg>
</div>
);

function selectRole(role) {
setSelRole(role); setParola(””); setEroare(””); setSelElev(””); setSelInstr(null);
if (role.id === “elev”) setStep(“selectElev”);
else if (role.id === “instructor”) setStep(“selectInstr”);
else setStep(“password”);
}

function selectElev(elevNume) {
setSelElev(elevNume); setParola(””); setEroare(””); setStep(“password”);
}

function selectInstr(instr) {
setSelInstr(instr); setParola(””); setEroare(””); setStep(“password”);
}

function goBack() {
if (step === “password” && selRole?.id === “elev”) { setStep(“selectElev”); setParola(””); setEroare(””); }
else if (step === “password” && selRole?.id === “instructor”) { setStep(“selectInstr”); setParola(””); setEroare(””); }
else { setStep(“role”); setEroare(””); }
}

function tryLogin() {
setLoading(true); setEroare(””);
setTimeout(() => {
if (selRole.id === “birou”) {
if (parola === CREDENTIALS.birou) { onLogin({ role: selRole, elevData: null, instrData: null }); }
else { setEroare(“Parolă incorectă”); setLoading(false); }
} else if (selRole.id === “instructor”) {
if (selInstr && parola.toLowerCase() === selInstr.parola.toLowerCase()) {
onLogin({ role: selRole, elevData: null, instrData: selInstr });
} else { setEroare(“Parolă incorectă”); setLoading(false); }
} else {
const found = elevi.find(e => e.nume === selElev);
if (found && found.parola && found.parola.toLowerCase() === parola.toLowerCase()) {
onLogin({ role: selRole, elevData: found, instrData: null });
} else { setEroare(“Parolă incorectă”); setLoading(false); }
}
}, 700);
}

const inpStyle = { width:“100%”, background:”#fff”, border:“1px solid “ + (eroare ? CLACCENT : “#ddd”), borderRadius:12, padding:“14px 16px”, fontSize:15, outline:“none”, boxSizing:“border-box”, color:”#111” };

const BackBar = ({ label }) => (
<div style={{ display:“flex”, alignItems:“center”, gap:10, marginBottom:22 }}>
<button onClick={goBack} style={{ background:”#fff”, border:“1px solid #eee”, borderRadius:10, width:36, height:36, display:“flex”, alignItems:“center”, justifyContent:“center”, fontSize:16, cursor:“pointer”, flexShrink:0 }}>←</button>
<div style={{ flex:1, background:”#fff”, border:“1px solid #eee”, borderRadius:12, padding:“10px 14px”, display:“flex”, alignItems:“center”, gap:10 }}>
<span style={{ fontSize:20 }}>{selRole?.icon}</span>
<div style={{ fontWeight:700, fontSize:13, color:”#111” }}>{selRole?.label}{label ? “ · “ + label : “”}</div>
</div>
</div>
);

return (
<div style={{ minHeight:“100vh”, background:CLBG, display:“flex”, flexDirection:“column”, alignItems:“center”, justifyContent:“center”, padding:“24px”, fontFamily:”‘DM Sans’,‘Segoe UI’,sans-serif” }}>
<style>{`@keyframes spin2 { to { transform:rotate(360deg); } } @keyframes fadeUp { from{opacity:0;transform:translateY(14px)} to{opacity:1;transform:translateY(0)} } @keyframes shake { 0%,100%{transform:translateX(0)} 25%{transform:translateX(-8px)} 75%{transform:translateX(8px)} }`}</style>

```
  {/* Logo */}
  <div style={{ textAlign:"center", marginBottom:28, animation:"fadeUp 0.5s ease" }}>
    <Logo/>
    <div style={{ fontSize:26, fontWeight:900, color:"#111", letterSpacing:-1 }}>Super OK</div>
    <div style={{ fontSize:11, color:CLACCENT, marginTop:3, letterSpacing:3, fontWeight:700 }}>ȘCOALA / ȘOFERI</div>
  </div>

  {/* STEP 1 — alege rol */}
  {step === "role" && (
    <div style={{ width:"100%", maxWidth:340 }}>
      <div style={{ fontSize:11, color:"#888", letterSpacing:2, textTransform:"uppercase", marginBottom:14, textAlign:"center" }}>Cine ești?</div>
      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr 1fr", gap:10 }}>
      {ROLES.map((role, idx) => (
        <div key={role.id} onClick={() => selectRole(role)}
          style={{ background:"#fff", border:"1px solid #eee", borderRadius:16, padding:"20px 10px", cursor:"pointer", display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center", gap:10, boxShadow:"0 2px 8px rgba(0,0,0,0.06)", animation:"fadeUp 0.4s ease both", animationDelay: idx*0.08+"s" }}>
          <div style={{ width:52, height:52, borderRadius:14, background:CLACCENT+"12", border:"1px solid "+CLACCENT+"30", display:"flex", alignItems:"center", justifyContent:"center", fontSize:24 }}>{role.icon}</div>
          <div style={{ fontWeight:800, fontSize:13, color:"#111", textAlign:"center", lineHeight:1.2 }}>{role.label}</div>
        </div>
      ))}
      </div>
    </div>
  )}

  {/* STEP 2 (doar elev) — selectează elevul din dropdown */}
  {step === "selectElev" && (
    <div style={{ width:"100%", maxWidth:340, animation:"fadeUp 0.3s ease" }}>
      <BackBar label="Selectează elevul" />
      <div style={{ fontSize:13, color:"#555", marginBottom:10, fontWeight:600 }}>Numele tău:</div>

      {/* Dropdown custom — carduri selectabile */}
      <div style={{ background:"#fff", border:"1px solid #eee", borderRadius:14, overflow:"hidden", boxShadow:"0 2px 12px rgba(0,0,0,0.07)", marginBottom:16 }}>
        {elevi.length === 0 && (
          <div style={{ padding:"20px", textAlign:"center", color:"#aaa", fontSize:13 }}>Niciun elev înregistrat</div>
        )}
        {elevi.map((elev, i) => (
          <div key={elev.id} onClick={() => selectElev(elev.nume)}
            style={{ padding:"13px 16px", display:"flex", alignItems:"center", gap:12, cursor:"pointer", borderBottom: i < elevi.length-1 ? "1px solid #f5f5f5" : "none", background:"#fff", transition:"background 0.15s" }}
            onMouseEnter={e => e.currentTarget.style.background="#fafafa"}
            onMouseLeave={e => e.currentTarget.style.background="#fff"}>
            <div style={{ width:38, height:38, borderRadius:"50%", background:"linear-gradient(135deg,"+CLACCENT+"22,"+CLACCENT+"10)", border:"1px solid "+CLACCENT+"22", display:"flex", alignItems:"center", justifyContent:"center", fontSize:16, flexShrink:0 }}>
              {elev.status === "promovat" ? "🏆" : "👤"}
            </div>
            <div style={{ flex:1 }}>
              <div style={{ fontWeight:700, fontSize:14, color:"#111" }}>{elev.nume}</div>
              <div style={{ fontSize:10, color:"#aaa", marginTop:2 }}>
                {elev.masina} · {elev.ore}/{elev.oreMax}h conduse
              </div>
            </div>
            <div style={{ fontSize:16, color:"#ddd" }}>›</div>
          </div>
        ))}
      </div>
    </div>
  )}

  {/* STEP 2b — selectează instructorul */}
  {step === "selectInstr" && (
    <div style={{ width:"100%", maxWidth:340, animation:"fadeUp 0.3s ease" }}>
      <BackBar label="Selectează instructorul" />
      <div style={{ fontSize:13, color:"#555", marginBottom:10, fontWeight:600 }}>Numele tău:</div>
      <div style={{ background:"#fff", border:"1px solid #eee", borderRadius:14, overflow:"hidden", boxShadow:"0 2px 12px rgba(0,0,0,0.07)", marginBottom:16 }}>
        {instructori.length === 0 && (
          <div style={{ padding:"20px", textAlign:"center", color:"#aaa", fontSize:13 }}>Niciun instructor înregistrat</div>
        )}
        {instructori.map((instr, i) => (
          <div key={instr.id} onClick={() => selectInstr(instr)}
            style={{ padding:"13px 16px", display:"flex", alignItems:"center", gap:12, cursor:"pointer", borderBottom: i < instructori.length-1 ? "1px solid #f5f5f5" : "none", background:"#fff", transition:"background 0.15s" }}
            onMouseEnter={e => e.currentTarget.style.background="#fafafa"}
            onMouseLeave={e => e.currentTarget.style.background="#fff"}>
            <div style={{ width:42, height:42, borderRadius:"50%", background:"linear-gradient(135deg,#3B82F622,#3B82F610)", border:"1px solid #3B82F622", display:"flex", alignItems:"center", justifyContent:"center", fontSize:18, flexShrink:0, overflow:"hidden" }}>
              {instr.foto ? <img src={instr.foto} alt="" style={{ width:"100%", height:"100%", objectFit:"cover" }}/> : "👨‍🏫"}
            </div>
            <div style={{ flex:1 }}>
              <div style={{ fontWeight:700, fontSize:14, color:"#111" }}>{instr.nume}</div>
              <div style={{ fontSize:10, color:"#aaa", marginTop:2 }}>
                {instr.masina} · Aut: {instr.autorizatie}
              </div>
            </div>
            <div style={{ fontSize:16, color:"#ddd" }}>›</div>
          </div>
        ))}
      </div>
    </div>
  )}

  {/* STEP 3 — parolă */}
  {step === "password" && (
    <div style={{ width:"100%", maxWidth:340, animation:"fadeUp 0.3s ease" }}>
      <BackBar label={selElev || selInstr?.nume || ""} />

      {/* Avatar elev selectat */}
      {selRole?.id === "elev" && selElev && (
        <div style={{ background:"#fff", border:"1px solid #eee", borderRadius:13, padding:"12px 16px", display:"flex", alignItems:"center", gap:12, marginBottom:18, boxShadow:"0 1px 6px rgba(0,0,0,0.05)" }}>
          <div style={{ width:44, height:44, borderRadius:"50%", background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)", display:"flex", alignItems:"center", justifyContent:"center", fontSize:20, flexShrink:0 }}>👤</div>
          <div>
            <div style={{ fontWeight:800, fontSize:15, color:"#111" }}>{selElev}</div>
            <div style={{ fontSize:11, color:"#888", marginTop:2 }}>Introdu parola ta personală</div>
          </div>
        </div>
      )}

      {/* Avatar instructor selectat */}
      {selRole?.id === "instructor" && selInstr && (
        <div style={{ background:"#fff", border:"1px solid #eee", borderRadius:13, padding:"12px 16px", display:"flex", alignItems:"center", gap:12, marginBottom:18, boxShadow:"0 1px 6px rgba(0,0,0,0.05)" }}>
          <div style={{ width:44, height:44, borderRadius:"50%", background:"linear-gradient(135deg,#3B82F6,#1D4ED8)", display:"flex", alignItems:"center", justifyContent:"center", fontSize:20, flexShrink:0, overflow:"hidden" }}>
            {selInstr.foto ? <img src={selInstr.foto} alt="" style={{ width:"100%", height:"100%", objectFit:"cover" }}/> : "👨‍🏫"}
          </div>
          <div>
            <div style={{ fontWeight:800, fontSize:15, color:"#111" }}>{selInstr.nume}</div>
            <div style={{ fontSize:11, color:"#888", marginTop:2 }}>Introdu parola ta de acces</div>
          </div>
        </div>
      )}

      <div style={{ fontSize:13, color:"#555", marginBottom:8, fontWeight:600 }}>Parolă:</div>

      <div style={{ position:"relative", marginBottom:10 }}>
        <input
          type={showPass ? "text" : "password"}
          value={parola}
          onChange={e => { setParola(e.target.value); setEroare(""); }}
          onKeyDown={e => e.key === "Enter" && parola && tryLogin()}
          placeholder="••••••••"
          autoFocus
          style={{ ...inpStyle, letterSpacing: showPass ? 1 : 4, animation: eroare ? "shake 0.35s ease" : "none" }}
        />
        <button onClick={() => setShowPass(p=>!p)}
          style={{ position:"absolute", right:14, top:"50%", transform:"translateY(-50%)", background:"none", border:"none", cursor:"pointer", fontSize:16, color:"#aaa" }}>
          {showPass ? "🙈" : "👁️"}
        </button>
      </div>

      {eroare && (
        <div style={{ background:"#fff0f0", border:"1px solid "+CLACCENT+"44", borderRadius:9, padding:"9px 12px", fontSize:12, color:CLACCENT, fontWeight:600, marginBottom:10, display:"flex", alignItems:"center", gap:6 }}>
          ⚠️ {eroare}
        </div>
      )}

      {selRole?.id === "elev" && (
        <div style={{ fontSize:11, color:"#bbb", marginBottom:14 }}>
          💡 Parola implicită: prenumele tău cu litere mici
        </div>
      )}

      <button onClick={tryLogin} disabled={!parola || loading}
        style={{ width:"100%", background:(!parola||loading)?"#e5e5e5":"linear-gradient(135deg,"+CLACCENT+",#8B000A)", color:(!parola||loading)?"#aaa":"#fff", border:"none", borderRadius:12, padding:"14px", fontWeight:800, fontSize:15, cursor:(!parola||loading)?"default":"pointer", boxShadow:(!parola||loading)?"none":"0 4px 18px "+CLACCENT+"44", transition:"all 0.2s" }}>
        {loading
          ? <span style={{ display:"inline-flex", alignItems:"center", gap:8 }}>
              <span style={{ width:16, height:16, border:"2px solid #aaa", borderTopColor:"transparent", borderRadius:"50%", display:"inline-block", animation:"spin2 0.7s linear infinite" }}/>
              Se verifică...
            </span>
          : "Intră în cont →"
        }
      </button>
    </div>
  )}
</div>
```

);
}

// ── STUDENT ──────────────────────────────────────────────────────────────────
function ElevInterface({ onLogout, elevi, elevData, programari, sesiuni, materiale, citite, setCitite, marcheazaCititDB }) {
const me = elevData || elevi[0] || ELEVI_INIT[0];
const myMateriale = (materiale||[]).filter(m => m.alocatLa.includes(me.nume));
const myProgramari = (programari || []).filter(p => p.elev === me.nume)
.sort((a,b) => a.data.localeCompare(b.data) || a.ora.localeCompare(b.ora));

function marcheazaCitit(matId) {
const key = me.nume + “::” + matId;
const dataAzi = new Date().toLocaleDateString(“ro-RO”, { day:“numeric”, month:“short” });
setCitite(prev => ({ …prev, [key]: dataAzi }));
// Salvează în Supabase
if (marcheazaCititDB && me.id) {
marcheazaCititDB(matId, me.id, me.nume).catch(console.warn);
}
}
function esteCitit(matId) {
return !!((citite||{})[me.nume + “::” + matId]);
}
function dataCitit(matId) {
return (citite||{})[me.nume + “::” + matId] || “”;
}

const totalMat   = myMateriale.length;
const totalCitite = myMateriale.filter(m => esteCitit(m.id)).length;

const [tab, setTab] = useState(“acasa”);
const [expanded, setExpanded] = useState(null);

const navTabs = [
{ id:“acasa”,  icon:“⊞”,  label:“Acasă”  },
{ id:“lectii”, icon:“📖”, label:“Lecții” },
{ id:“orar”,   icon:“📅”, label:“Orar”   },
{ id:“profil”, icon:“👤”, label:“Profil” },
];
const lectii = [
{ title:“Regulile de circulație”, icon:“📋”, pct:85, clr:CLRED  },
{ title:“Semnele de circulație”,  icon:“🚦”, pct:60, clr:CLBLU  },
{ title:“Manevre de conducere”,   icon:“🚗”, pct:40, clr:CLGRN  },
{ title:“Situații de urgență”,    icon:“🆘”, pct:20, clr:”#8B5CF6” },
{ title:“Conduita preventivă”,    icon:“🛡️”, pct:10, clr:”#F59E0B” },
];

return (
<div style={{ minHeight:“100vh”, background:CLBG, fontFamily:”‘DM Sans’,‘Segoe UI’,sans-serif”, color:”#111”, maxWidth:420, margin:“0 auto” }}>
<style>{`@keyframes blip{0%,100%{opacity:1}50%{opacity:0.25}}`}</style>

```
  {/* header */}
  <div style={{ padding:"52px 20px 18px", background:"linear-gradient(180deg," + CLRED + "15 0%,transparent 100%)" }}>
    <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
      <div>
        <p style={{ color:"#555", fontSize:11, margin:0, letterSpacing:2, textTransform:"uppercase" }}>Bună dimineața</p>
        <h1 style={{ fontSize:23, fontWeight:900, margin:"4px 0 0", background:"linear-gradient(135deg,#111 0%," + CLRED + " 100%)", WebkitBackgroundClip:"text", WebkitTextFillColor:"transparent" }}>Alexandru M.</h1>
        <p style={{ color:CLACCENT, fontSize:10, margin:"4px 0 0", fontWeight:700, letterSpacing:1, textTransform:"uppercase" }}>🎓 Elev · SuperOk Bistrița</p>
      </div>
      <button onClick={onLogout} style={{ background:"rgba(0,0,0,0.06)", border:"1px solid " + CLBORD, color:"#555", borderRadius:9, padding:"6px 12px", fontSize:11, cursor:"pointer" }}>Ieșire</button>
    </div>
    {/* countdown */}
    <div style={{ marginTop:14, background:"linear-gradient(135deg,rgba(208,22,27,0.12),rgba(208,22,27,0.04))", border:"1px solid " + CLACCENT + "44", borderRadius:14, padding:"13px 16px", display:"flex", justifyContent:"space-between", alignItems:"center" }}>
      <div>
        <p style={{ margin:0, fontSize:10, color:CLACCENT, letterSpacing:1.5, textTransform:"uppercase", fontWeight:700 }}>Examen Final</p>
        <p style={{ margin:"4px 0 0", fontSize:13, fontWeight:700 }}>12 Iunie 2026 • 09:00</p>
      </div>
      <div style={{ background:CLACCENT + "22", border:"1px solid " + CLACCENT + "55", borderRadius:10, padding:"7px 13px", textAlign:"center" }}>
        <div style={{ fontSize:22, fontWeight:900, color:CLACCENT, lineHeight:1 }}>29</div>
        <div style={{ fontSize:9, color:CLACCENT, opacity:0.8, textTransform:"uppercase" }}>zile</div>
      </div>
    </div>
  </div>

  <div style={{ padding:"0 20px 100px" }}>

    {tab === "acasa" && (
      <div>
        {/* stats */}
        <div style={{ display:"flex", gap:8, marginBottom:18 }}>
          {[
            { label:"Ore condus", val:me.ore, max:me.oreMax, unit:"h" },
            { label:"Teorie",     val:me.teorie, max:100,    unit:"%" },
          ].map((st, i) => (
            <div key={i} style={{ flex:1, background:CLCARD, border:"1px solid " + CLBORD, borderRadius:12, padding:"13px 10px", textAlign:"center" }}>
              <div style={{ fontSize:21, fontWeight:900, color:CLRED }}>{st.val}<span style={{ fontSize:10, color:"#555" }}>{st.unit}</span></div>
              <div style={{ fontSize:9, color:"#666", marginTop:1 }}>din {st.max}{st.unit}</div>
              <div style={{ marginTop:7, height:3, borderRadius:2, background:"rgba(0,0,0,0.05)" }}>
                <div style={{ height:"100%", borderRadius:2, background:"linear-gradient(90deg," + CLRED + "," + CLRED2 + ")", width:(st.val / st.max * 100) + "%" }} />
              </div>
              <div style={{ fontSize:9, color:"#666", marginTop:4 }}>{st.label}</div>
            </div>
          ))}
          <div style={{ flex:1, background:CLCARD, border:"1px solid " + CLBORD, borderRadius:12, padding:"13px 10px", textAlign:"center" }}>
            <div style={{ fontSize:13, fontWeight:800, color:CLGRN, marginTop:4 }}>{me.masina}</div>
            <div style={{ fontSize:9, color:"#666", marginTop:4 }}>mașina ta</div>
          </div>
        </div>

        {/* Progres materiale */}
        {totalMat > 0 && (
          <div style={{ background:"#fff", border:"1px solid " + (totalCitite===totalMat ? "#1A936F44" : CLBORD), borderRadius:14, padding:"14px 16px", marginBottom:18 }}>
            <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:10 }}>
              <div>
                <div style={{ fontWeight:800, fontSize:14, color:"#111" }}>
                  {totalCitite===totalMat ? "🎉 Toate materialele citite!" : "📚 Materiale de studiu"}
                </div>
                <div style={{ fontSize:11, color:"#888", marginTop:2 }}>
                  {totalCitite} din {totalMat} {totalMat===1?"material":"materiale"} citit{totalCitite!==1?"e":""}
                </div>
              </div>
              <div style={{ fontSize:22, fontWeight:900, color: totalCitite===totalMat ? "#1A936F" : CLACCENT }}>
                {Math.round(totalCitite/totalMat*100)}<span style={{ fontSize:12 }}>%</span>
              </div>
            </div>
            <div style={{ height:8, borderRadius:4, background:"rgba(0,0,0,0.06)", marginBottom:8 }}>
              <div style={{ height:"100%", borderRadius:4, transition:"width 0.5s ease",
                background: totalCitite===totalMat ? "linear-gradient(90deg,#1A936F,#2ECC71)" : "linear-gradient(90deg,"+CLACCENT+",#8B000A)",
                width: (totalMat > 0 ? totalCitite/totalMat*100 : 0) + "%" }} />
            </div>
            <div style={{ display:"flex", gap:5, flexWrap:"wrap" }}>
              {myMateriale.map((m,i) => (
                <div key={i} title={m.titlu} style={{ width:30, height:30, borderRadius:8, display:"flex", alignItems:"center", justifyContent:"center", fontSize:15,
                  background: esteCitit(m.id) ? "#1A936F15" : "rgba(0,0,0,0.05)",
                  border:"1px solid " + (esteCitit(m.id) ? "#1A936F44" : "#eee") }}>
                  {esteCitit(m.id) ? "✅" : (m.tip==="Curs"?"📖":m.tip==="Test"?"📝":m.tip==="Simulator"?"🖥️":"🎬")}
                </div>
              ))}
            </div>
            {totalCitite < totalMat && (
              <div style={{ marginTop:10, fontSize:11, color:CLACCENT, fontWeight:600 }}>
                → Mai ai {totalMat-totalCitite} material{totalMat-totalCitite!==1?"e":""} de studiat — tab-ul 📖 Lecții
              </div>
            )}
          </div>
        )}

        {/* driving hours bar */}
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:10 }}>Ore de Condus</p>
        <div style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:14, padding:"15px 16px", marginBottom:18 }}>
          <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-end", marginBottom:10 }}>
            <div>
              <div style={{ fontSize:30, fontWeight:900, color:CLRED, lineHeight:1 }}>{me.ore}<span style={{ fontSize:13, color:"#555", fontWeight:400 }}>h</span></div>
              <div style={{ fontSize:11, color:"#555", marginTop:2 }}>din {me.oreMax}h obligatorii</div>
            </div>
            <div style={{ textAlign:"right" }}>
              <div style={{ fontSize:24, fontWeight:900, color:"#111" }}>{Math.round(me.ore / me.oreMax * 100)}<span style={{ fontSize:12, color:CLRED }}>%</span></div>
              <div style={{ fontSize:10, color:"#555", marginTop:2 }}>completat</div>
            </div>
          </div>
          <div style={{ position:"relative", marginBottom:7 }}>
            <div style={{ height:10, borderRadius:5, background:"rgba(0,0,0,0.05)", overflow:"hidden" }}>
              <div style={{ height:"100%", borderRadius:5, background:"linear-gradient(90deg," + CLRED + "," + CLRED2 + ")", width:(me.ore / me.oreMax * 100) + "%" }} />
            </div>
            {[25,50,75].map((p) => (
              <div key={p} style={{ position:"absolute", top:0, bottom:0, left:p + "%", width:1, background:"rgba(0,0,0,0.5)" }} />
            ))}
          </div>
          <div style={{ display:"flex", justifyContent:"space-between" }}>
            {[0,5,10,15,20].map((h) => (
              <div key={h} style={{ fontSize:9, color: h <= me.ore ? CLRED : "#333", fontWeight: (h === me.ore || h === me.oreMax) ? 700 : 400 }}>{h}h</div>
            ))}
          </div>
          <div style={{ marginTop:11, padding:"8px 13px", background:CLRED + "10", border:"1px solid " + CLRED + "22", borderRadius:9, display:"flex", justifyContent:"space-between", alignItems:"center" }}>
            <span style={{ fontSize:11, color:"#555" }}>Mai ai nevoie de</span>
            <span style={{ fontSize:13, fontWeight:800, color:CLRED }}>{me.oreMax - me.ore} ore condus</span>
          </div>
        </div>

        {/* next class */}
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:10 }}>Următoarea Lecție</p>
        <div style={{ background:"linear-gradient(135deg," + CLGRN + "18," + CLGRN + "05)", border:"1px solid " + CLGRN + "33", borderRadius:14, padding:"13px 15px", display:"flex", gap:12, alignItems:"center" }}>
          <div style={{ width:46, height:46, borderRadius:11, background:CLGRN + "25", border:"1px solid " + CLGRN + "55", display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center" }}>
            <div style={{ fontSize:10, color:CLGRN, fontWeight:700 }}>LUN</div>
            <div style={{ fontSize:17, fontWeight:900, color:"#111", lineHeight:1.1 }}>19</div>
          </div>
          <div>
            <div style={{ fontWeight:700, fontSize:14 }}>Practică — Bistrița</div>
            <div style={{ color:"#555", fontSize:11, marginTop:2 }}>10:00 · Prof. Vaida Adrian · {me.masina}</div>
          </div>
        </div>
      </div>
    )}

    {tab === "lectii" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:14 }}>Materialele Mele</p>

        {myMateriale.length === 0 && (
          <div style={{ background:CLCARD, border:"1px solid "+CLBORD, borderRadius:13, padding:"30px", textAlign:"center", color:"#aaa" }}>
            <div style={{ fontSize:36, marginBottom:10 }}>📚</div>
            <div style={{ fontSize:13 }}>Niciun material alocat încă</div>
            <div style={{ fontSize:11, marginTop:4 }}>Biroul va adăuga cursuri și teste</div>
          </div>
        )}

        {/* Filtre rapide */}
        {myMateriale.length > 0 && (
          <div style={{ display:"flex", gap:6, marginBottom:12, flexWrap:"wrap" }}>
            {["Curs","Test","Simulator","Video"]
              .filter(t => myMateriale.some(m => m.tip===t))
              .map(t => (
                <div key={t} style={{ background:CLACCENT+"10", border:"1px solid "+CLACCENT+"25", borderRadius:20, padding:"4px 11px", fontSize:10, fontWeight:600, color:CLACCENT }}>
                  {t==="Curs"?"📖":t==="Test"?"📝":t==="Simulator"?"🖥️":"🎬"} {t} ({myMateriale.filter(m=>m.tip===t).length})
                </div>
              ))
            }
          </div>
        )}

        {myMateriale.map((mat, i) => {
          const tipIcon = mat.tip==="Curs"?"📖":mat.tip==="Test"?"📝":mat.tip==="Simulator"?"🖥️":"🎬";
          const tipClr  = mat.tip==="Curs"?"#3B82F6":mat.tip==="Test"?CLACCENT:mat.tip==="Simulator"?"#8B5CF6":"#1A936F";
          return (
            <div key={i} onClick={()=>setExpanded(expanded===i?null:i)}
              style={{ background: esteCitit(mat.id) ? "#f8fff8" : "#fff", border:"1px solid "+(expanded===i?tipClr+"44": esteCitit(mat.id) ? "#1A936F33" : CLBORD), borderRadius:13, padding:"13px 15px", marginBottom:9, cursor:"pointer", boxShadow:"0 1px 5px rgba(0,0,0,0.05)", borderLeft:"3px solid "+(esteCitit(mat.id)?"#1A936F":tipClr) }}>
              <div style={{ display:"flex", alignItems:"center", gap:11 }}>
                <div style={{ width:42, height:42, borderRadius:11, background: esteCitit(mat.id) ? "#1A936F15" : tipClr+"15", border:"1px solid "+(esteCitit(mat.id)?"#1A936F33":tipClr+"33"), display:"flex", alignItems:"center", justifyContent:"center", fontSize:20, flexShrink:0 }}>
                  {esteCitit(mat.id) ? "✅" : tipIcon}
                </div>
                <div style={{ flex:1, minWidth:0 }}>
                  <div style={{ display:"flex", alignItems:"center", gap:6, marginBottom:3 }}>
                    <div style={{ fontSize:9, fontWeight:700, color: esteCitit(mat.id) ? "#1A936F" : tipClr, background: esteCitit(mat.id) ? "#1A936F12" : tipClr+"12", borderRadius:5, padding:"1px 6px" }}>
                      {esteCitit(mat.id) ? "✓ CITIT" : mat.tip.toUpperCase()}
                    </div>
                    <div style={{ fontSize:9, color:"#bbb" }}>{mat.marime}</div>
                  </div>
                  <div style={{ fontWeight:700, fontSize:13, color:"#111" }}>{mat.titlu}</div>
                  <div style={{ fontSize:11, color:"#888", marginTop:2 }}>{mat.descriere}</div>
                  {esteCitit(mat.id) && <div style={{ fontSize:10, color:"#1A936F", marginTop:3 }}>📅 {dataCitit(mat.id)}</div>}
                </div>
                <div style={{ fontSize:16, color:"#ccc" }}>{expanded===i?"▲":"▼"}</div>
              </div>

              {expanded === i && (
                <div style={{ marginTop:11, paddingTop:11, borderTop:"1px solid #f5f5f5" }}>
                  <div style={{ fontSize:10, color:"#bbb", marginBottom:10 }}>📎 {mat.fisier} · Adăugat: {mat.data}</div>
                  <div style={{ display:"flex", gap:8 }}>
                    {mat.fileUrl
                      ? <a href={mat.fileUrl} target="_blank" rel="noreferrer" download={mat.fisier}
                          onClick={() => marcheazaCitit(mat.id)}
                          style={{ flex:1, background: esteCitit(mat.id) ? "linear-gradient(135deg,#1A936F,#137a5a)" : "linear-gradient(135deg,"+CLACCENT+",#8B000A)", color:"#fff", border:"none", borderRadius:9, padding:"10px", fontWeight:700, fontSize:12, cursor:"pointer", textDecoration:"none", textAlign:"center", display:"block" }}>
                          {esteCitit(mat.id)
                            ? (mat.fileType?.includes("video") ? "✅ Vizionat" : "✅ Citit / Descărcat")
                            : (mat.fileType?.includes("video") ? "▶ Vizionează" : "📄 Deschide / Descarcă")
                          }
                        </a>
                      : <div style={{ flex:1, background:"#fff8f0", border:"1px solid "+CLACCENT+"33", borderRadius:9, padding:"10px 12px" }}>
                          <div style={{ fontWeight:700, fontSize:11, color:CLACCENT, marginBottom:3 }}>📎 Fișier demo</div>
                          <div style={{ fontSize:10, color:"#888" }}>Biroul trebuie să încarce fișierul real din tab-ul Materiale → + Adaugă</div>
                        </div>
                    }
                    {mat.tip==="Test" && (
                      <button onClick={() => marcheazaCitit(mat.id)} style={{ flex:1, background: esteCitit(mat.id) ? "#f0fff8" : "rgba(59,130,246,0.08)", border:"1px solid " + (esteCitit(mat.id) ? "#1A936F44" : "rgba(59,130,246,0.3)"), color: esteCitit(mat.id) ? "#1A936F" : "#3B82F6", borderRadius:9, padding:"10px", fontWeight:700, fontSize:12, cursor:"pointer" }}>
                        {esteCitit(mat.id) ? "✅ Completat" : "✏️ Începe Testul"}
                      </button>
                    )}
                  </div>
                  {esteCitit(mat.id) && (
                    <div style={{ marginTop:8, fontSize:10, color:"#1A936F", fontWeight:600, display:"flex", alignItems:"center", gap:4 }}>
                      ✓ Marcat ca citit pe {dataCitit(mat.id)}
                      <span onClick={() => setCitite(prev => { const n={...prev}; delete n[me.nume+"::"+mat.id]; return n; })}
                        style={{ marginLeft:6, color:"#bbb", cursor:"pointer", fontWeight:400, fontSize:11 }}>
                        (resetează)
                      </span>
                    </div>
                  )}
                </div>
              )}
            </div>
          );
        })}
      </div>
    )}

    {tab === "orar" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:14 }}>Programul Tău</p>
        {myProgramari.length === 0 && (
          <div style={{ background:CLCARD, border:"1px solid "+CLBORD, borderRadius:13, padding:"24px", textAlign:"center", color:"#aaa", fontSize:13 }}>
            <div style={{ fontSize:30, marginBottom:8 }}>📅</div>
            Nicio lecție programată încă.<br/>
            <span style={{ fontSize:11 }}>Contactează instructorul sau biroul.</span>
          </div>
        )}
        {myProgramari.map((p, i) => {
          const d = new Date(p.data);
          const today = new Date().toISOString().slice(0,10);
          const isToday = p.data === today;
          const isUpcoming = p.data >= today;
          const tipColor = p.tip==="Practică" ? CLACCENT : "#3B82F6";
          const durata = p.durata < 60 ? p.durata+"min" : (p.durata/60)+"h"+(p.durata%60?p.durata%60+"min":"");
          return (
            <div key={i} style={{ background:"#fff", border:"1px solid "+(isToday?CLACCENT+"55":CLBORD), borderLeft:"3px solid "+(isUpcoming?tipColor:"#ddd"), borderRadius:13, padding:"13px 15px", marginBottom:9, boxShadow:"0 1px 6px rgba(0,0,0,0.05)" }}>
              <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
                <div style={{ flex:1 }}>
                  <div style={{ display:"flex", alignItems:"center", gap:7, marginBottom:5 }}>
                    <div style={{ background:tipColor+"15", border:"1px solid "+tipColor+"33", borderRadius:6, padding:"2px 8px", fontSize:9, fontWeight:700, color:tipColor }}>
                      {p.tip==="Practică"?"🚗":p.tip==="Teorie"?"📖":"🖥️"} {p.tip}
                    </div>
                    <div style={{ fontSize:9, color:"#aaa", background:"rgba(0,0,0,0.04)", borderRadius:6, padding:"2px 8px" }}>{durata}</div>
                    {isToday && <div style={{ fontSize:9, fontWeight:700, color:CLACCENT, background:CLACCENT+"12", borderRadius:6, padding:"2px 8px" }}>AZI</div>}
                  </div>
                  <div style={{ fontWeight:700, fontSize:14 }}>{d.toLocaleDateString("ro-RO",{weekday:"long",day:"numeric",month:"long"})}</div>
                  <div style={{ fontSize:11, color:"#888", marginTop:3 }}>
                    🕐 {p.ora} · 👨‍🏫 {p.instructor}{p.masina&&p.masina!=="-"?" · 🚗 "+p.masina:""}
                  </div>
                </div>
                <div style={{ fontSize:20, fontWeight:900, color: isUpcoming?CLACCENT:"#ccc", marginLeft:12 }}>{p.ora}</div>
              </div>
            </div>
          );
        })}
      </div>
    )}

    {tab === "profil" && (
      <div>
        <div style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:17, padding:"20px", marginBottom:13, textAlign:"center" }}>
          <div style={{ width:66, height:66, borderRadius:"50%", background:"linear-gradient(135deg," + CLRED + "," + CLRED2 + ")", display:"flex", alignItems:"center", justifyContent:"center", fontSize:27, margin:"0 auto 11px", boxShadow:"0 8px 24px " + CLRED + "44" }}>👤</div>
          <div style={{ fontWeight:800, fontSize:19 }}>{me.nume}</div>
          <div style={{ color:"#555", fontSize:12, marginTop:3 }}>Cursant din {me.data}</div>
          <div style={{ display:"inline-flex", alignItems:"center", gap:5, marginTop:9, background:CLGRN + "18", border:"1px solid " + CLGRN + "33", borderRadius:20, padding:"4px 12px" }}>
            <div style={{ width:6, height:6, borderRadius:"50%", background:CLGRN }} />
            <span style={{ fontSize:11, color:CLGRN, fontWeight:600 }}>Activ</span>
          </div>
        </div>
        {[
          { icon:"🏫", label:"Școala",           val:"SuperOk Bistrița" },
          { icon:"👨‍🏫", label:"Instructor",       val:"Prof. Vaida Adrian" },
          { icon:"🚗", label:"Mașina atribuită", val:me.masina },
          { icon:"📞", label:"Telefon",           val:me.telefon },
          { icon:"🎯", label:"Categorie",         val:"Permis B" },
          { icon:"💳", label:"Plată",             val: me.plata === "achitat" ? "✅ Achitat" : "⚠️ Restant" },
        ].map((item, i) => (
          <div key={i} style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:11, padding:"12px 15px", display:"flex", alignItems:"center", gap:11, marginBottom:7 }}>
            <span style={{ fontSize:17 }}>{item.icon}</span>
            <div style={{ flex:1 }}>
              <div style={{ fontSize:9, color:"#666", letterSpacing:1, textTransform:"uppercase" }}>{item.label}</div>
              <div style={{ fontSize:13, fontWeight:600, marginTop:2 }}>{item.val}</div>
            </div>
          </div>
        ))}
      </div>
    )}
  </div>

  {/* nav */}
  <div style={{ position:"fixed", bottom:0, left:"50%", transform:"translateX(-50%)", width:"100%", maxWidth:420, background:"rgba(250,250,250,0.97)", backdropFilter:"blur(20px)", borderTop:"1px solid rgba(0,0,0,0.1)", padding:"11px 4px 24px", display:"flex", justifyContent:"space-around", zIndex:100 }}>
    {navTabs.map((t) => (
      <button key={t.id} onClick={() => setTab(t.id)} style={{ background:"none", border:"none", cursor:"pointer", display:"flex", flexDirection:"column", alignItems:"center", gap:3, padding:"5px 10px", color: tab === t.id ? CLRED : "#444" }}>
        <span style={{ fontSize:17 }}>{t.icon}</span>
        <span style={{ fontSize:9, fontWeight: tab === t.id ? 700 : 500, letterSpacing:0.5, textTransform:"uppercase" }}>{t.label}</span>
        {tab === t.id && <div style={{ width:4, height:4, borderRadius:"50%", background:CLACCENT, boxShadow:"0 0 7px " + CLACCENT }} />}
      </button>
    ))}
  </div>
</div>
```

);
}

// ── INSTRUCTOR ───────────────────────────────────────────────────────────────
function InstructorInterface({ onLogout, elevi, instrData, programari, sesiuni, setSesiuni, setElevi }) {
const [tab, setTab]         = useState(“sesiune”);
const [phase, setPhase]     = useState(“idle”);
const [elevSel, setElevSel] = useState(null);
const [simActive, setSimActive] = useState(false);
const [elapsed, setElapsed] = useState(0);
const [simKm, setSimKm]     = useState(0);
const [simSpd, setSimSpd]   = useState(0);
const [simPos, setSimPos]   = useState(ROUTE_GPS[0]);
const [simTrail, setSimTrail] = useState([]);
const rIdxRef  = useRef(0);
const rTRef    = useRef(0);
const kmRef    = useRef(0);
const intRef   = useRef(null);

// Filtrăm elevii și programările după instructorul logat
const myElevi = instrData
? elevi.filter(e => e.masina === instrData.masina)
: elevi;
const myProgramari = instrData
? (programari || []).filter(p => p.instructor === instrData.nume)
: (programari || []);
const myMasina = instrData?.masina || “B-01-SUK”;

function startSession(elev) {
setElevSel(elev);
rIdxRef.current = 0; rTRef.current = 0; kmRef.current = 0;
setElapsed(0); setSimKm(0); setSimSpd(0);
setSimPos(ROUTE_GPS[0]); setSimTrail([ROUTE_GPS[0]]);
setSimActive(true); setPhase(“active”);
}

function stopSession() {
setSimActive(false);
clearInterval(intRef.current);

```
// Formatare durată
const h = Math.floor(elapsed / 3600);
const m = Math.floor((elapsed % 3600) / 60);
const durataStr = h > 0 ? h + "h " + String(m).padStart(2,"0") + "m" : m + "m";
const durataMin = Math.round(elapsed / 60);
const dataStr = new Date().toLocaleDateString("ro-RO", { day:"numeric", month:"long", year:"numeric" });

// Salvează sesiunea în istoricul global
if (setSesiuni && elevSel) {
  setSesiuni(prev => [...prev, {
    id: Date.now(),
    elev: elevSel.nume,
    masina: myMasina,
    instructor: instrData?.nume || "Instructor",
    data: dataStr,
    durata: durataStr,
    durataMin,
    km: simKm,
  }]);
}

// Actualizează orele conduse ale elevului
if (setElevi && elevSel && durataMin > 0) {
  setElevi(prev => prev.map(e =>
    e.id === elevSel.id
      ? { ...e, ore: Math.min(e.oreMax, parseFloat((e.ore + durataMin/60).toFixed(1))) }
      : e
  ));
}

setPhase("done");
```

}

useEffect(() => {
if (simActive) {
intRef.current = setInterval(() => {
setElapsed((e) => e + 1);
const spd = 28 + Math.random() * 42;
setSimSpd(Math.round(spd));
kmRef.current += spd / 3600;
setSimKm(parseFloat(kmRef.current.toFixed(2)));
rTRef.current += 0.06;
if (rTRef.current >= 1) { rTRef.current = 0; rIdxRef.current = (rIdxRef.current + 1) % (ROUTE_GPS.length - 1); }
const fromPt = ROUTE_GPS[rIdxRef.current];
const toPt   = ROUTE_GPS[(rIdxRef.current + 1) % ROUTE_GPS.length];
const pos    = { lat: lerpNum(fromPt.lat, toPt.lat, rTRef.current), lng: lerpNum(fromPt.lng, toPt.lng, rTRef.current) };
setSimPos(pos);
setSimTrail((prev) => […prev.slice(-40), pos]);
}, 1000);
} else {
clearInterval(intRef.current);
}
return () => clearInterval(intRef.current);
}, [simActive]);

const navTabs = [
{ id:“sesiune”, icon:“▶”,  label:“Sesiune” },
{ id:“harta”,   icon:“🗺️”, label:“Hartă”   },
{ id:“istoric”, icon:“📋”, label:“Istoric”  },
];

return (
<div style={{ minHeight:“100vh”, background:CLBG, fontFamily:”‘DM Sans’,‘Segoe UI’,sans-serif”, color:”#111”, maxWidth:420, margin:“0 auto” }}>
<style>{`@keyframes liveDot{0%,100%{opacity:1}50%{opacity:0.2}}`}</style>

```
  <div style={{ padding:"52px 20px 16px", background:"linear-gradient(180deg," + CLBLU + "12 0%,transparent 100%)" }}>
    <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
      <div>
        <p style={{ color:"#555", fontSize:11, margin:0, letterSpacing:2, textTransform:"uppercase" }}>Tabletă Mașină</p>
        <h1 style={{ fontSize:21, fontWeight:900, margin:"4px 0 0", background:"linear-gradient(135deg,#111 0%," + CLBLU + " 100%)", WebkitBackgroundClip:"text", WebkitTextFillColor:"transparent" }}>{instrData?.nume || "Instructor"}</h1>
        <p style={{ color:CLACCENT, fontSize:10, margin:"4px 0 0", fontWeight:700, letterSpacing:1, textTransform:"uppercase" }}>🚗 {myMasina}</p>
      </div>
      <button onClick={onLogout} style={{ background:"rgba(0,0,0,0.06)", border:"1px solid " + CLBORD, color:"#555", borderRadius:9, padding:"6px 12px", fontSize:11, cursor:"pointer" }}>Ieșire</button>
    </div>
  </div>

  <div style={{ padding:"0 20px 100px" }}>

    {tab === "sesiune" && phase === "idle" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:13 }}>Pornește Sesiune Nouă</p>

        {/* Programări ale instructorului azi */}
        {myProgramari.filter(p => p.data === new Date().toISOString().slice(0,10)).length > 0 && (
          <div style={{ marginBottom:14 }}>
            <p style={{ fontSize:10, color:CLACCENT, letterSpacing:1.5, textTransform:"uppercase", marginBottom:8, fontWeight:700 }}>📅 Programate Azi</p>
            {myProgramari.filter(p => p.data === new Date().toISOString().slice(0,10)).map((p,i) => (
              <div key={i} style={{ background:"#fff", border:"1px solid "+CLACCENT+"33", borderLeft:"3px solid "+CLACCENT, borderRadius:11, padding:"10px 14px", marginBottom:7, display:"flex", justifyContent:"space-between", alignItems:"center" }}>
                <div>
                  <div style={{ fontWeight:700, fontSize:13 }}>{p.elev}</div>
                  <div style={{ fontSize:10, color:"#888", marginTop:2 }}>{p.ora} · {p.tip}{p.masina&&p.masina!=="-"?" · "+p.masina:""}</div>
                </div>
                <div style={{ fontSize:13, fontWeight:800, color:CLACCENT }}>{p.ora}</div>
              </div>
            ))}
          </div>
        )}

        <p style={{ fontSize:12, color:"#555", marginBottom:10 }}>Elevii tăi ({myElevi.length}):</p>
        {myElevi.length === 0 && (
          <div style={{ background:CLCARD, border:"1px solid "+CLBORD, borderRadius:13, padding:"20px", textAlign:"center", color:"#aaa", fontSize:13 }}>
            Niciun elev atribuit mașinii tale
          </div>
        )}
        {myElevi.map((elev, i) => (
          <div key={i} onClick={() => startSession(elev)} style={{ background:"#fff", border:"1px solid " + CLBORD, borderRadius:13, padding:"13px 15px", marginBottom:9, display:"flex", alignItems:"center", gap:11, cursor:"pointer", boxShadow:"0 1px 6px rgba(0,0,0,0.05)" }}>
            <div style={{ width:42, height:42, borderRadius:11, background:"#3B82F615", border:"1px solid #3B82F622", display:"flex", alignItems:"center", justifyContent:"center", fontSize:19 }}>👤</div>
            <div style={{ flex:1 }}>
              <div style={{ fontWeight:700, fontSize:13 }}>{elev.nume}</div>
              <div style={{ fontSize:11, color:"#555", marginTop:2 }}>{elev.ore}h / {elev.oreMax}h conduse · {elev.masina}</div>
              <div style={{ marginTop:6, height:3, borderRadius:2, background:"rgba(0,0,0,0.05)" }}>
                <div style={{ height:"100%", borderRadius:2, background:"linear-gradient(90deg," + CLACCENT + ",#8B000A)", width:(elev.ore / elev.oreMax * 100) + "%" }} />
              </div>
            </div>
            <div style={{ fontSize:13, color: elev.plata === "achitat" ? "#1A936F" : CLACCENT, fontWeight:700 }}>{elev.plata === "achitat" ? "✓" : "⚠"}</div>
          </div>
        ))}
      </div>
    )}

    {tab === "sesiune" && phase === "active" && (
      <div>
        <div style={{ display:"flex", alignItems:"center", justifyContent:"space-between", marginBottom:13 }}>
          <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", margin:0 }}>Sesiune Activă</p>
          <div style={{ display:"flex", alignItems:"center", gap:6, background:CLACCENT + "18", border:"1px solid " + CLACCENT + "44", borderRadius:20, padding:"4px 11px" }}>
            <div style={{ width:7, height:7, borderRadius:"50%", background:CLACCENT, animation:"liveDot 1s infinite" }} />
            <span style={{ fontSize:11, color:CLACCENT, fontWeight:700, letterSpacing:1 }}>LIVE</span>
          </div>
        </div>
        <div style={{ background:CLBLU + "08", border:"1px solid " + CLBLU + "25", borderRadius:13, padding:"11px 15px", display:"flex", alignItems:"center", gap:11, marginBottom:13 }}>
          <span style={{ fontSize:21 }}>👤</span>
          <div style={{ flex:1 }}>
            <div style={{ fontWeight:700, fontSize:13 }}>{elevSel && elevSel.nume}</div>
            <div style={{ fontSize:11, color:"#3B82F6", marginTop:2 }}>Lecție practică · {myMasina} · {instrData?.nume||"Instructor"}</div>
          </div>
          <div style={{ fontSize:10, color:CLGRN, background:CLGRN + "15", border:"1px solid " + CLGRN + "33", borderRadius:8, padding:"3px 9px", fontWeight:700 }}>GPS 🛰️</div>
        </div>
        {/* timer */}
        <div style={{ background:"linear-gradient(135deg,rgba(208,22,27,0.12),rgba(208,22,27,0.04))", border:"1px solid " + CLACCENT + "30", borderRadius:17, padding:"22px 18px", textAlign:"center", marginBottom:13 }}>
          <div style={{ fontSize:10, color:CLACCENT, letterSpacing:2, textTransform:"uppercase", marginBottom:7 }}>Timp Sesiune</div>
          <div style={{ fontSize:50, fontWeight:900, letterSpacing:3, color:"#111", lineHeight:1 }}>{fmtTime(elapsed)}</div>
        </div>
        <div style={{ display:"flex", gap:10, marginBottom:13 }}>
          {[
            { label:"Distanță", val:simKm,  unit:"km",   clr:CLRED },
            { label:"Viteză",   val:simSpd,  unit:"km/h", clr:CLGRN },
          ].map((st, i) => (
            <div key={i} style={{ flex:1, background:CLCARD, border:"1px solid " + CLBORD, borderRadius:13, padding:"13px", textAlign:"center" }}>
              <div style={{ fontSize:10, color:"#555", letterSpacing:1.5, textTransform:"uppercase", marginBottom:5 }}>{st.label}</div>
              <div style={{ fontSize:27, fontWeight:900, color:st.clr, lineHeight:1 }}>{st.val}</div>
              <div style={{ fontSize:10, color:"#666", marginTop:4 }}>{st.unit}</div>
            </div>
          ))}
        </div>
        <div style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:11, padding:"11px 15px", marginBottom:13 }}>
          <div style={{ height:7, borderRadius:4, background:"rgba(0,0,0,0.05)" }}>
            <div style={{ height:"100%", borderRadius:4, width: Math.min(simSpd / 130 * 100, 100) + "%", transition:"width 0.7s ease", background: simSpd < 50 ? "linear-gradient(90deg," + CLGRN + ",#2ECC71)" : simSpd < 90 ? "linear-gradient(90deg,#F59E0B,#FCD34D)" : "linear-gradient(90deg," + CLRED + "," + CLRED2 + ")" }} />
          </div>
          <div style={{ fontSize:11, marginTop:7, textAlign:"center", fontWeight:600, color: simSpd < 50 ? CLGRN : simSpd < 90 ? "#F59E0B" : CLRED }}>
            {simSpd < 50 ? "🟢 Viteză sigură" : simSpd < 90 ? "🟡 Moderată" : "🔴 Reduce viteza!"}
          </div>
        </div>
        <button onClick={stopSession} style={{ width:"100%", background:CLACCENT + "18", border:"1px solid " + CLACCENT + "44", color:CLACCENT, borderRadius:13, padding:"14px", fontWeight:800, fontSize:14, cursor:"pointer" }}>
          ⏹ Finalizează Sesiunea
        </button>
      </div>
    )}

    {tab === "sesiune" && phase === "done" && (
      <div>
        <div style={{ textAlign:"center", padding:"20px 0 22px" }}>
          <div style={{ width:66, height:66, borderRadius:"50%", background:CLGRN + "18", border:"2px solid " + CLGRN + "66", display:"flex", alignItems:"center", justifyContent:"center", fontSize:29, margin:"0 auto 13px", boxShadow:"0 0 26px " + CLGRN + "30" }}>✓</div>
          <div style={{ fontWeight:800, fontSize:21 }}>Sesiune Salvată!</div>
          <div style={{ color:"#555", fontSize:12, marginTop:4 }}>Date transmise la birou automat</div>
        </div>
        <div style={{ background:CLGRN + "08", border:"1px solid " + CLGRN + "22", borderRadius:17, padding:"17px", marginBottom:13 }}>
          {[
            { l:"Elev",        v: elevSel && elevSel.nume },
            { l:"Durata",      v: fmtShort(elapsed) },
            { l:"Distanță GPS",v: simKm + " km" },
            { l:"Mașina",      v: "B-01-SUK · Dacia Logan" },
            { l:"Data",        v: new Date().toLocaleDateString("ro-RO", { day:"numeric", month:"long" }) },
          ].map((row, i, arr) => (
            <div key={i} style={{ display:"flex", justifyContent:"space-between", paddingBottom: i < arr.length - 1 ? 9 : 0, marginBottom: i < arr.length - 1 ? 9 : 0, borderBottom: i < arr.length - 1 ? "1px solid " + CLBORD : "none" }}>
              <span style={{ fontSize:12, color:"#555" }}>{row.l}</span>
              <span style={{ fontSize:12, fontWeight:700 }}>{row.v}</span>
            </div>
          ))}
        </div>
        <button onClick={() => { setPhase("idle"); setSimActive(false); setElapsed(0); setSimKm(0); }} style={{ width:"100%", background:"linear-gradient(135deg," + CLACCENT + ",#8B000A)", color:"#111", border:"none", borderRadius:13, padding:"14px", fontWeight:800, fontSize:14, cursor:"pointer", boxShadow:"0 4px 20px " + CLACCENT + "44" }}>
          + Sesiune Nouă
        </button>
      </div>
    )}

    {tab === "harta" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:11 }}>Hartă Live</p>
        <div style={{ position:"relative", marginBottom:13 }}>
          <LiveMap simActive={simActive} simPos={simPos} simTrail={simTrail} mapHeight={300} />
          <div style={{ position:"absolute", top:10, right:10, zIndex:999, display:"flex", alignItems:"center", gap:5, background:"rgba(245,245,245,0.92)", border:"1px solid " + (simActive ? CLGRN + "55" : CLBORD), borderRadius:20, padding:"5px 11px" }}>
            <div style={{ width:7, height:7, borderRadius:"50%", background: simActive ? CLGRN : "#444", animation: simActive ? "liveDot 1.2s infinite" : "none" }} />
            <span style={{ fontSize:10, color: simActive ? CLGRN : "#444", fontWeight:700, letterSpacing:1 }}>{simActive ? "LIVE GPS" : "STANDBY"}</span>
          </div>
          {simActive && (
            <div style={{ position:"absolute", bottom:10, left:10, zIndex:999, background:"rgba(245,245,245,0.95)", border:"1px solid " + CLGRN + "33", borderRadius:10, padding:"7px 12px", display:"flex", gap:14 }}>
              <div style={{ textAlign:"center" }}><div style={{ fontSize:13, fontWeight:900, color:CLRED }}>{simKm}</div><div style={{ fontSize:9, color:"#555" }}>km</div></div>
              <div style={{ textAlign:"center" }}><div style={{ fontSize:13, fontWeight:900, color:CLGRN }}>{simSpd}</div><div style={{ fontSize:9, color:"#555" }}>km/h</div></div>
              <div style={{ textAlign:"center" }}><div style={{ fontSize:13, fontWeight:900, color:"#111" }}>{fmtTime(elapsed)}</div><div style={{ fontSize:9, color:"#555" }}>timp</div></div>
            </div>
          )}
        </div>
        {!simActive && <div style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:11, padding:"13px 15px", textAlign:"center", color:"#666", fontSize:12 }}>Pornește o sesiune pentru tracking live</div>}
      </div>
    )}

    {tab === "istoric" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:13 }}>
          Sesiunile Mele ({(sesiuni||[]).filter(s => instrData ? s.instructor===instrData.nume : true).length})
        </p>
        {(sesiuni||[])
          .filter(s => instrData ? s.instructor===instrData.nume : true)
          .slice().reverse()
          .map((s, i) => (
          <div key={s.id||i} style={{ background:"#fff", border:"1px solid " + CLBORD, borderRadius:13, padding:"13px 15px", marginBottom:9, display:"flex", justifyContent:"space-between", alignItems:"center", boxShadow:"0 1px 5px rgba(0,0,0,0.05)" }}>
            <div>
              <div style={{ fontWeight:700, fontSize:13 }}>{s.elev}</div>
              <div style={{ fontSize:11, color:"#888", marginTop:2 }}>{s.data} · {s.masina}</div>
            </div>
            <div style={{ textAlign:"right" }}>
              <div style={{ fontWeight:700, fontSize:13, color:CLACCENT }}>{s.durata}</div>
              <div style={{ fontSize:11, color:"#888", marginTop:1 }}>{s.km} km</div>
            </div>
          </div>
        ))}
        {(sesiuni||[]).filter(s => instrData ? s.instructor===instrData.nume : true).length === 0 && (
          <div style={{ textAlign:"center", color:"#bbb", padding:"30px 0", fontSize:13 }}>
            <div style={{ fontSize:32, marginBottom:8 }}>📋</div>
            Nicio sesiune finalizată încă
          </div>
        )}
      </div>
    )}
  </div>

  <div style={{ position:"fixed", bottom:0, left:"50%", transform:"translateX(-50%)", width:"100%", maxWidth:420, background:"rgba(250,250,250,0.97)", backdropFilter:"blur(20px)", borderTop:"1px solid rgba(0,0,0,0.1)", padding:"11px 4px 24px", display:"flex", justifyContent:"space-around", zIndex:100 }}>
    {navTabs.map((t) => (
      <button key={t.id} onClick={() => setTab(t.id)} style={{ background:"none", border:"none", cursor:"pointer", display:"flex", flexDirection:"column", alignItems:"center", gap:3, padding:"5px 10px", color: tab === t.id ? CLBLU : "#444" }}>
        <span style={{ fontSize:17 }}>{t.icon}</span>
        <span style={{ fontSize:9, fontWeight: tab === t.id ? 700 : 500, letterSpacing:0.5, textTransform:"uppercase" }}>{t.label}</span>
        {tab === t.id && <div style={{ width:4, height:4, borderRadius:"50%", background:CLBLU, boxShadow:"0 0 7px " + CLBLU }} />}
      </button>
    ))}
  </div>
</div>
```

);
}

// ── Field input — defined outside any component to prevent focus loss ────────
function Field({ label, value, onChange, type, options }) {
const inputStyle = { width:“100%”, background:“rgba(0,0,0,0.06)”, border:“1px solid “ + CLBORD, color:”#111”, borderRadius:9, padding:“10px 11px”, fontSize:12, outline:“none”, boxSizing:“border-box” };
return (
<div style={{ marginBottom:11 }}>
<div style={{ fontSize:9, color:”#555”, letterSpacing:1, textTransform:“uppercase”, marginBottom:5 }}>{label}</div>
{options
? <select value={value} onChange={e => onChange(e.target.value)} style={{…inputStyle}}>
{options.map(o => <option key={o.val} value={o.val} style={{ background:”#f0f0f0” }}>{o.lbl}</option>)}
</select>
: <input type={type || “text”} value={value} placeholder={label} onChange={e => onChange(e.target.value)} style={inputStyle} />
}
</div>
);
}

// ── OFFICE ───────────────────────────────────────────────────────────────────
function BirouInterface({ onLogout, elevi, setElevi, instructori, setInstructori, masini, setMasini, programari, setProgramari, sesiuni, materiale, setMateriale, citite, addElevDB, addMaterialDB, deleteMaterialDB, alocaMaterialDB, addProgramareDB, deleteProgramareDB, refreshElevi }) {
const [tab, setTab]           = useState(“dashboard”);
const [showAdd, setShowAdd]   = useState(false);
const [detailId, setDetailId] = useState(null);
const [newElev, setNewElev]   = useState({ nume:””, telefon:””, parola:””, masina:“B-01-SUK”, plata:“achitat” });
const [pdfLoading, setPdfLoading] = useState(null);
const [foaieModal, setFoaieModal] = useState(null);
const [flotaTab, setFlotaTab] = useState(“masini”);
const [matTab, setMatTab]       = useState(“toate”);
const [showAddMat, setShowAddMat] = useState(false);
const [savingMat, setSavingMat]   = useState(false);
const [showAlocaMat, setShowAlocaMat] = useState(null);
const [formMat, setFormMat]     = useState({ tip:“Curs”, titlu:””, descriere:””, fisier:null, numeFisier:””, marime:”” });
const [alocaElevi, setAlocaElevi] = useState([]);
const [planTab, setPlanTab]   = useState(“lista”);
const [formPlan, setFormPlan] = useState({ tip:“Practică”, elev:””, instructor:””, masina:””, data:””, ora:””, durata:60 });
const [editPlan, setEditPlan] = useState(null);
const [filterZi,  setFilterZi]  = useState(“toate”);
const [filterTip, setFilterTip] = useState(“toate”);
const [editMasina,     setEditMasina]     = useState(null);
const [editInstructor, setEditInstructor] = useState(null);
const [formMasina,     setFormMasina]     = useState({});
const [formInstr,      setFormInstr]      = useState({});

const restanti  = elevi.filter((e) => e.plata === “restant”).length;
const activi    = elevi.filter((e) => e.status === “activ”).length;
const totalOre  = elevi.reduce((acc, e) => acc + e.ore, 0);
const promovati = elevi.filter((e) => e.status === “promovat”).length;

function generateFoaieParcurs(masina) {
setFoaieModal(masina);
}

async function addElev() {
if (!newElev.nume.trim()) return;
try {
if (addElevDB) {
await addElevDB(newElev);
} else {
const parola = newElev.parola.trim() || newElev.nume.trim().split(” “)[0].toLowerCase();
setElevi((prev) => […prev, { id: prev.length + 1, …newElev, parola, ore:0, oreMax:20, teorie:0, status:“activ”, data:“Mai 2026” }]);
}
setNewElev({ nume:””, telefon:””, parola:””, masina:“B-01-SUK”, plata:“achitat” });
setShowAdd(false);
} catch(e) { alert(“Eroare la salvare: “ + e.message); }
}

const navTabs = [
{ id:“dashboard”,  icon:“📊”, label:“Board”    },
{ id:“elevi”,      icon:“🎓”, label:“Elevi”    },
{ id:“programari”, icon:“📅”, label:“Program”  },
{ id:“materiale”,  icon:“📚”, label:“Materiale”},
{ id:“flota”,      icon:“🚗”, label:“Flotă”    },
];

return (
<div style={{ minHeight:“100vh”, background:CLBG, fontFamily:”‘DM Sans’,‘Segoe UI’,sans-serif”, color:”#111”, maxWidth:420, margin:“0 auto” }}>
<style>{`@keyframes slideUp{from{transform:translateY(100%);opacity:0}to{transform:translateY(0);opacity:1}}`}</style>

```
  <div style={{ padding:"52px 20px 16px", background:"linear-gradient(180deg," + CLGRN + "12 0%,transparent 100%)" }}>
    <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
      <div>
        <p style={{ color:"#555", fontSize:11, margin:0, letterSpacing:2, textTransform:"uppercase" }}>Panou Administrare</p>
        <h1 style={{ fontSize:21, fontWeight:900, margin:"4px 0 0", background:"linear-gradient(135deg,#111 0%," + CLGRN + " 100%)", WebkitBackgroundClip:"text", WebkitTextFillColor:"transparent" }}>SuperOk Bistrița</h1>
        <p style={{ color:CLACCENT, fontSize:10, margin:"4px 0 0", fontWeight:700, letterSpacing:1, textTransform:"uppercase" }}>🏢 Birou · Admin</p>
      </div>
      <button onClick={onLogout} style={{ background:"rgba(0,0,0,0.06)", border:"1px solid " + CLBORD, color:"#555", borderRadius:9, padding:"6px 12px", fontSize:11, cursor:"pointer" }}>Ieșire</button>
    </div>
  </div>

  <div style={{ padding:"0 20px 100px" }}>

    {tab === "dashboard" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:11 }}>Statistici Generale</p>
        <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:9, marginBottom:18 }}>
          {[
            { icon:"🎓", label:"Elevi activi",    val:activi,   clr:CLGRN },
            { icon:"⚠️", label:"Restanți plată",  val:restanti, clr:CLRED },
            { icon:"⏱️", label:"Total ore condus", val:totalOre + "h", clr:CLBLU },
            { icon:"✅", label:"Promovați",        val:promovati, clr:"#8B5CF6" },
          ].map((kpi, i) => (
            <div key={i} style={{ background:CLCARD, border:"1px solid " + kpi.clr + "33", borderRadius:13, padding:"15px 13px" }}>
              <div style={{ fontSize:21, marginBottom:5 }}>{kpi.icon}</div>
              <div style={{ fontSize:25, fontWeight:900, color:kpi.clr, lineHeight:1 }}>{kpi.val}</div>
              <div style={{ fontSize:11, color:"#555", marginTop:4 }}>{kpi.label}</div>
            </div>
          ))}
        </div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:11 }}>Status Flotă</p>
        {MASINI.map((m, i) => (
          <div key={i} style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:11, padding:"12px 15px", marginBottom:7, display:"flex", alignItems:"center", gap:11 }}>
            <div style={{ width:38, height:38, borderRadius:9, background: m.status === "în mișcare" ? CLGRN + "18" : CLRED + "10", border:"1px solid " + (m.status === "în mișcare" ? CLGRN : CLRED) + "33", display:"flex", alignItems:"center", justifyContent:"center", fontSize:17 }}>🚗</div>
            <div style={{ flex:1 }}>
              <div style={{ fontWeight:700, fontSize:12 }}>{m.model} <span style={{ color:"#666", fontSize:10, fontWeight:400 }}>({m.id})</span></div>
              <div style={{ fontSize:10, color:"#555", marginTop:1 }}>{m.km.toLocaleString()} km · {m.airtag}</div>
            </div>
            <div style={{ fontSize:11, color: m.status === "în mișcare" ? CLGRN : "#555", fontWeight:600 }}>{m.status === "în mișcare" ? "🟢 Live" : "⚫ Parcat"}</div>
          </div>
        ))}
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:11, marginTop:18 }}>Sesiuni Recente</p>
        {(sesiuni||[]).slice().reverse().slice(0, 3).map((s, i) => (
          <div key={i} style={{ background:CLCARD, border:"1px solid " + CLBORD, borderRadius:11, padding:"11px 15px", marginBottom:7, display:"flex", justifyContent:"space-between", alignItems:"center" }}>
            <div>
              <div style={{ fontWeight:600, fontSize:12 }}>{s.elev}</div>
              <div style={{ fontSize:10, color:"#555", marginTop:2 }}>{s.data} · {s.masina}</div>
            </div>
            <div style={{ textAlign:"right" }}>
              <div style={{ fontWeight:700, fontSize:12, color:CLRED }}>{s.durata}</div>
              <div style={{ fontSize:10, color:"#555", marginTop:1 }}>{s.km} km</div>
            </div>
          </div>
        ))}
      </div>
    )}

    {tab === "elevi" && (
      <div>
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:13 }}>
          <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", margin:0 }}>Toți Elevii ({elevi.length})</p>
          <button onClick={() => setShowAdd(true)} style={{ background:"linear-gradient(135deg," + CLACCENT + ",#8B000A)", color:"#111", border:"none", borderRadius:9, padding:"7px 13px", fontWeight:700, fontSize:11, cursor:"pointer" }}>+ Adaugă</button>
        </div>
        {elevi.map((elev, i) => (
          <div key={i} onClick={() => setDetailId(detailId === elev.id ? null : elev.id)} style={{ background:CLCARD, border:"1px solid " + (detailId === elev.id ? CLRED + "55" : CLBORD), borderRadius:13, padding:"13px 15px", marginBottom:9, cursor:"pointer" }}>
            <div style={{ display:"flex", alignItems:"center", gap:11 }}>
              <div style={{ width:42, height:42, borderRadius:11, background:CLRED + "14", border:"1px solid " + CLRED + "30", display:"flex", alignItems:"center", justifyContent:"center", fontSize:19 }}>
                {elev.status === "promovat" ? "🏆" : "👤"}
              </div>
              <div style={{ flex:1 }}>
                <div style={{ fontWeight:700, fontSize:13 }}>{elev.nume}</div>
                <div style={{ display:"flex", gap:7, marginTop:4, alignItems:"center" }}>
                  <span style={{ fontSize:10, color: elev.status === "promovat" ? CLGRN : CLRED, background: elev.status === "promovat" ? CLGRN + "18" : CLRED + "14", border:"1px solid " + (elev.status === "promovat" ? CLGRN : CLRED) + "33", borderRadius:6, padding:"2px 7px", fontWeight:600 }}>{elev.status}</span>
                  <span style={{ fontSize:10, color: elev.plata === "achitat" ? CLGRN : CLRED, fontWeight:600 }}>{elev.plata === "achitat" ? "💳 OK" : "⚠️ Restant"}</span>
                </div>
              </div>
              <div style={{ textAlign:"right" }}>
                <div style={{ fontSize:12, fontWeight:800, color:CLRED }}>{elev.ore}<span style={{ fontSize:10, color:"#555" }}>/{elev.oreMax}h</span></div>
                <div style={{ fontSize:10, color:"#555", marginTop:2 }}>Teorie: {elev.teorie}%</div>
              </div>
            </div>
            <div style={{ marginTop:9, height:3, borderRadius:2, background:"rgba(0,0,0,0.05)" }}>
              <div style={{ height:"100%", borderRadius:2, background:"linear-gradient(90deg," + CLRED + "," + CLRED2 + ")", width:(elev.ore / elev.oreMax * 100) + "%" }} />
            </div>
            {detailId === elev.id && (
              <div style={{ marginTop:11, paddingTop:11, borderTop:"1px solid rgba(0,0,0,0.1)" }}>
                {[
                  { l:"Mașina",          v: elev.masina },
                  { l:"Telefon",          v: elev.telefon },
                  { l:"Data înscrierii", v: elev.data },
                  { l:"Ore condus",       v: elev.ore + " / " + elev.oreMax + "h" },
                  { l:"Progres teorie",   v: elev.teorie + "%" },
                ].map((row, j) => (
                  <div key={j} style={{ display:"flex", justifyContent:"space-between", padding:"6px 0", borderBottom: j < 4 ? "1px solid " + CLBORD : "none" }}>
                    <span style={{ fontSize:11, color:"#555" }}>{row.l}</span>
                    <span style={{ fontSize:11, fontWeight:600 }}>{row.v}</span>
                  </div>
                ))}
                {/* Progres materiale */}
                {(() => {
                  const matElev = materiale.filter(m => m.alocatLa.includes(elev.nume));
                  const cititeElev = matElev.filter(m => !!(citite||{})[elev.nume+"::"+m.id]).length;
                  if (matElev.length === 0) return null;
                  return (
                    <div style={{ marginTop:10, paddingTop:8, borderTop:"1px solid "+CLBORD }}>
                      <div style={{ display:"flex", justifyContent:"space-between", marginBottom:5 }}>
                        <span style={{ fontSize:11, color:"#555" }}>📚 Materiale citite</span>
                        <span style={{ fontSize:11, fontWeight:700, color: cititeElev===matElev.length?"#1A936F":CLACCENT }}>
                          {cititeElev} / {matElev.length}
                          {cititeElev===matElev.length?" ✅":""}
                        </span>
                      </div>
                      <div style={{ height:4, borderRadius:2, background:"rgba(0,0,0,0.06)" }}>
                        <div style={{ height:"100%", borderRadius:2, width:(cititeElev/matElev.length*100)+"%",
                          background: cititeElev===matElev.length ? "linear-gradient(90deg,#1A936F,#2ECC71)" : "linear-gradient(90deg,"+CLACCENT+",#8B000A)",
                          transition:"width 0.4s" }}/>
                      </div>
                      <div style={{ display:"flex", flexWrap:"wrap", gap:4, marginTop:6 }}>
                        {matElev.map((m,j)=>(
                          <div key={j} title={m.titlu} style={{ fontSize:9, padding:"2px 7px", borderRadius:5, fontWeight:600,
                            background: (citite||{})[elev.nume+"::"+m.id] ? "#1A936F12" : "rgba(0,0,0,0.05)",
                            color: (citite||{})[elev.nume+"::"+m.id] ? "#1A936F" : "#aaa",
                            border:"1px solid " + ((citite||{})[elev.nume+"::"+m.id] ? "#1A936F33" : "#eee") }}>
                            {(citite||{})[elev.nume+"::"+m.id] ? "✓ " : ""}{m.titlu.substring(0,22)}{m.titlu.length>22?"...":""}
                          </div>
                        ))}
                      </div>
                    </div>
                  );
                })()}
                <div style={{ display:"flex", gap:8, marginTop:11 }}>
                  <button style={{ flex:1, background:CLRED + "14", border:"1px solid " + CLRED + "33", color:CLRED, borderRadius:9, padding:"8px", fontWeight:700, fontSize:11, cursor:"pointer" }}>✏️ Editează</button>
                  <button style={{ flex:1, background:CLGRN + "12", border:"1px solid " + CLGRN + "33", color:CLGRN, borderRadius:9, padding:"8px", fontWeight:700, fontSize:11, cursor:"pointer" }}>📅 Programează</button>
                </div>
              </div>
            )}
          </div>
        ))}
      </div>
    )}

    {tab === "programari" && (
      <div>
        <style>{`@keyframes pIn{from{opacity:0;transform:translateY(24px)}to{opacity:1;transform:translateY(0)}}`}</style>

        {/* Header */}
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:14}}>
          <p style={{fontSize:10,color:"#888",letterSpacing:2,textTransform:"uppercase",margin:0}}>Planificare Lecții</p>
          <button onClick={()=>setPlanTab(planTab==="adauga"?"lista":"adauga")}
            style={{background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:9,padding:"7px 14px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
            {planTab==="adauga"?"← Listă":"+ Lecție Nouă"}
          </button>
        </div>

        {/* FORMULAR ADĂUGARE / EDITARE */}
        {planTab==="adauga" && (
          <div style={{background:"#fff",border:"1px solid "+CLBORD,borderRadius:14,padding:"16px",marginBottom:14,animation:"pIn 0.3s ease",boxShadow:"0 2px 10px rgba(0,0,0,0.06)"}}>
            <div style={{fontWeight:800,fontSize:15,marginBottom:14,color:"#111"}}>
              {editPlan?"✏️ Editează Lecția":"📅 Lecție Nouă"}
            </div>

            {/* Tip */}
            <div style={{marginBottom:11}}>
              <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:6}}>Tip Lecție</div>
              <div style={{display:"flex",gap:8}}>
                {["Practică","Teorie","Simulator"].map(t=>(
                  <button key={t} onClick={()=>setFormPlan(p=>({...p,tip:t,masina:t==="Teorie"?"-":p.masina}))}
                    style={{flex:1,padding:"9px",borderRadius:9,border:"none",fontWeight:700,fontSize:11,cursor:"pointer",
                      background:formPlan.tip===t?"linear-gradient(135deg,"+CLACCENT+",#8B000A)":CLCARD,
                      color:formPlan.tip===t?"#fff":"#555"}}>
                    {t==="Practică"?"🚗 "+t:t==="Teorie"?"📖 "+t:"🖥️ "+t}
                  </button>
                ))}
              </div>
            </div>

            {/* Elev */}
            <div style={{marginBottom:11}}>
              <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Elev</div>
              <select value={formPlan.elev} onChange={e=>setFormPlan(p=>({...p,elev:e.target.value}))}
                style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",color:formPlan.elev?"#111":"#aaa"}}>
                <option value="">— Selectează elev —</option>
                {elevi.map(e=><option key={e.id} value={e.nume}>{e.nume} ({e.ore}/{e.oreMax}h)</option>)}
              </select>
            </div>

            {/* Instructor */}
            <div style={{marginBottom:11}}>
              <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Instructor</div>
              <select value={formPlan.instructor} onChange={e=>{
                const instr=instructori.find(x=>x.nume===e.target.value);
                setFormPlan(p=>({...p,instructor:e.target.value,masina:formPlan.tip!=="Teorie"&&instr?instr.masina:p.masina}));
              }} style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",color:formPlan.instructor?"#111":"#aaa"}}>
                <option value="">— Selectează instructor —</option>
                {instructori.map(ins=><option key={ins.id} value={ins.nume}>{ins.nume}</option>)}
              </select>
            </div>

            {/* Mașina (doar dacă nu e Teorie) */}
            {formPlan.tip !== "Teorie" && (
              <div style={{marginBottom:11}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Mașina</div>
                <select value={formPlan.masina} onChange={e=>setFormPlan(p=>({...p,masina:e.target.value}))}
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",color:formPlan.masina?"#111":"#aaa"}}>
                  <option value="">— Selectează mașina —</option>
                  {masini.map(m=><option key={m.id} value={m.id}>{m.id} · {m.model}</option>)}
                </select>
              </div>
            )}

            {/* Data + Ora */}
            <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:11}}>
              <div>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Data</div>
                <input type="date" value={formPlan.data} onChange={e=>setFormPlan(p=>({...p,data:e.target.value}))}
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",boxSizing:"border-box",color:"#111"}}/>
              </div>
              <div>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Ora</div>
                <input type="time" value={formPlan.ora} onChange={e=>setFormPlan(p=>({...p,ora:e.target.value}))}
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",boxSizing:"border-box",color:"#111"}}/>
              </div>
            </div>

            {/* Durată */}
            <div style={{marginBottom:16}}>
              <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:6}}>Durată</div>
              <div style={{display:"flex",gap:8}}>
                {[30,60,90,120].map(d=>(
                  <button key={d} onClick={()=>setFormPlan(p=>({...p,durata:d}))}
                    style={{flex:1,padding:"8px 4px",borderRadius:8,border:"none",fontWeight:700,fontSize:11,cursor:"pointer",
                      background:formPlan.durata===d?"linear-gradient(135deg,"+CLACCENT+",#8B000A)":CLCARD,
                      color:formPlan.durata===d?"#fff":"#555"}}>
                    {d<60?d+"m":d/60+"h"+(d%60?d%60+"m":"")}
                  </button>
                ))}
              </div>
            </div>

            {/* Save */}
            <button onClick={()=>{
              if(!formPlan.elev||!formPlan.instructor||!formPlan.data||!formPlan.ora)return;
              if(editPlan){
                setProgramari(prev=>prev.map(x=>x.id===editPlan.id?{...x,...formPlan}:x));
                setEditPlan(null);
              } else {
                setProgramari(prev=>[...prev,{...formPlan,id:Date.now(),status:"confirmat"}]);
              }
              setFormPlan({tip:"Practică",elev:"",instructor:"",masina:"",data:"",ora:"",durata:60});
              setPlanTab("lista");
            }} style={{width:"100%",background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:11,padding:"13px",fontWeight:800,fontSize:14,cursor:"pointer",boxShadow:"0 4px 16px "+CLACCENT+"33"}}>
              ✓ {editPlan?"Salvează Modificările":"Adaugă Lecția"}
            </button>
            {editPlan && (
              <button onClick={()=>{setEditPlan(null);setFormPlan({tip:"Practică",elev:"",instructor:"",masina:"",data:"",ora:"",durata:60});setPlanTab("lista");}}
                style={{width:"100%",background:"rgba(0,0,0,0.04)",border:"1px solid #eee",color:"#888",borderRadius:11,padding:"11px",fontWeight:700,fontSize:13,cursor:"pointer",marginTop:8}}>
                Anulează
              </button>
            )}
          </div>
        )}

        {/* LISTA PROGRAMĂRI */}
        {planTab==="lista" && (
          <div>
            {/* Filtre */}
            <div style={{display:"flex",gap:6,marginBottom:14,flexWrap:"wrap"}}>
              {["toate","Practică","Teorie","Simulator"].map(f=>(
                <button key={f} onClick={()=>setFilterTip(f)}
                  style={{padding:"5px 11px",borderRadius:20,border:"none",fontWeight:600,fontSize:10,cursor:"pointer",
                    background:filterTip===f?CLACCENT:CLCARD,color:filterTip===f?"#fff":"#666"}}>
                  {f==="toate"?"Toate":f}
                </button>
              ))}
            </div>

            {/* Calendar view — group by date */}
            {(() => {
              const filtered = programari
                .filter(p=>filterTip==="toate"||p.tip===filterTip)
                .sort((a,b)=>a.data.localeCompare(b.data)||a.ora.localeCompare(b.ora));
              const grouped = filtered.reduce((acc,p)=>{
                if(!acc[p.data]) acc[p.data]=[];
                acc[p.data].push(p);
                return acc;
              },{});
              const today = new Date().toISOString().slice(0,10);
              if(filtered.length===0) return <div style={{textAlign:"center",color:"#ccc",padding:"30px 0",fontSize:13}}>Nicio programare găsită</div>;
              return Object.entries(grouped).map(([data,items])=>{
                const d = new Date(data);
                const isToday = data===today;
                const ziNume = d.toLocaleDateString("ro-RO",{weekday:"long"});
                const dataNume = d.toLocaleDateString("ro-RO",{day:"numeric",month:"long"});
                return (
                  <div key={data} style={{marginBottom:16}}>
                    {/* Day header */}
                    <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:8}}>
                      <div style={{height:1,flex:1,background:"#eee"}}/>
                      <div style={{fontSize:10,fontWeight:700,color:isToday?CLACCENT:"#aaa",letterSpacing:1,textTransform:"uppercase",
                        background:isToday?CLACCENT+"12":"transparent",padding:"3px 10px",borderRadius:20,
                        border:isToday?"1px solid "+CLACCENT+"33":"none"}}>
                        {isToday?"AZI · ":""}{ziNume.toUpperCase()} · {dataNume}
                      </div>
                      <div style={{height:1,flex:1,background:"#eee"}}/>
                    </div>
                    {/* Items */}
                    {items.map((prog,i)=>{
                      const tipColor = prog.tip==="Practică"?CLACCENT:prog.tip==="Teorie"?"#3B82F6":"#8B5CF6";
                      const tipBg    = prog.tip==="Practică"?CLACCENT+"12":prog.tip==="Teorie"?"rgba(59,130,246,0.1)":"rgba(139,92,246,0.1)";
                      const tipBord  = prog.tip==="Practică"?CLACCENT+"33":prog.tip==="Teorie"?"rgba(59,130,246,0.3)":"rgba(139,92,246,0.3)";
                      return (
                        <div key={prog.id} style={{background:"#fff",border:"1px solid "+CLBORD,borderRadius:13,padding:"12px 14px",marginBottom:8,
                          boxShadow:"0 1px 6px rgba(0,0,0,0.05)",borderLeft:"3px solid "+tipColor}}>
                          <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:8}}>
                            <div style={{flex:1}}>
                              <div style={{display:"flex",alignItems:"center",gap:7,marginBottom:4}}>
                                <div style={{background:tipBg,border:"1px solid "+tipBord,borderRadius:6,padding:"2px 8px",fontSize:9,fontWeight:700,color:tipColor}}>
                                  {prog.tip==="Practică"?"🚗":prog.tip==="Teorie"?"📖":"🖥️"} {prog.tip}
                                </div>
                                <div style={{fontSize:9,background:"rgba(0,0,0,0.04)",borderRadius:6,padding:"2px 8px",color:"#888",fontWeight:600}}>
                                  {prog.durata<60?prog.durata+"min":prog.durata/60+"h"+(prog.durata%60?prog.durata%60+"min":"")}
                                </div>
                              </div>
                              <div style={{fontWeight:800,fontSize:14,color:"#111"}}>{prog.elev}</div>
                              <div style={{fontSize:11,color:"#888",marginTop:2}}>
                                👨‍🏫 {prog.instructor}
                                {prog.masina&&prog.masina!=="-"?" · 🚗 "+prog.masina:""}
                              </div>
                            </div>
                            <div style={{textAlign:"right",flexShrink:0,marginLeft:8}}>
                              <div style={{fontSize:18,fontWeight:900,color:CLACCENT,lineHeight:1}}>{prog.ora}</div>
                              <div style={{display:"flex",gap:5,marginTop:8,justifyContent:"flex-end"}}>
                                <button onClick={()=>{
                                  setFormPlan({tip:prog.tip,elev:prog.elev,instructor:prog.instructor,masina:prog.masina,data:prog.data,ora:prog.ora,durata:prog.durata});
                                  setEditPlan(prog);setPlanTab("adauga");
                                }} style={{background:"rgba(0,0,0,0.04)",border:"1px solid #eee",borderRadius:7,padding:"4px 9px",fontSize:10,fontWeight:700,cursor:"pointer",color:"#555"}}>✏️</button>
                                <button onClick={()=>setProgramari(prev=>prev.filter(x=>x.id!==prog.id))}
                                  style={{background:"rgba(192,0,8,0.05)",border:"1px solid #fcc",borderRadius:7,padding:"4px 9px",fontSize:10,fontWeight:700,cursor:"pointer",color:"#c00"}}>🗑️</button>
                              </div>
                            </div>
                          </div>
                          {/* Progress bar for student */}
                          {(() => {
                            const el = elevi.find(e=>e.nume===prog.elev);
                            if(!el) return null;
                            return (
                              <div style={{marginTop:6,paddingTop:6,borderTop:"1px solid #f5f5f5"}}>
                                <div style={{display:"flex",justifyContent:"space-between",marginBottom:4}}>
                                  <span style={{fontSize:9,color:"#bbb"}}>Ore condus</span>
                                  <span style={{fontSize:9,color:CLACCENT,fontWeight:700}}>{el.ore}/{el.oreMax}h</span>
                                </div>
                                <div style={{height:3,borderRadius:2,background:"#f0f0f0"}}>
                                  <div style={{height:"100%",borderRadius:2,background:"linear-gradient(90deg,"+CLACCENT+",#8B000A)",width:(el.ore/el.oreMax*100)+"%"}}/>
                                </div>
                              </div>
                            );
                          })()}
                        </div>
                      );
                    })}
                  </div>
                );
              });
            })()}

            {/* Summary cards */}
            <div style={{marginTop:8,display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:8}}>
              {[
                {label:"Total",val:programari.length,clr:"#111"},
                {label:"Practică",val:programari.filter(p=>p.tip==="Practică").length,clr:CLACCENT},
                {label:"Teorie",val:programari.filter(p=>p.tip==="Teorie").length,clr:"#3B82F6"},
              ].map((s,i)=>(
                <div key={i} style={{background:CLCARD,border:"1px solid "+CLBORD,borderRadius:11,padding:"12px",textAlign:"center"}}>
                  <div style={{fontSize:20,fontWeight:900,color:s.clr}}>{s.val}</div>
                  <div style={{fontSize:9,color:"#aaa",marginTop:3,textTransform:"uppercase",letterSpacing:1}}>{s.label}</div>
                </div>
              ))}
            </div>
          </div>
        )}
      </div>
    )}

    {tab === "materiale" && (
      <div>
        <style>{`@keyframes matIn{from{opacity:0;transform:translateY(20px)}to{opacity:1;transform:translateY(0)}}`}</style>

        {/* Header */}
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:14}}>
          <p style={{fontSize:10,color:"#888",letterSpacing:2,textTransform:"uppercase",margin:0}}>Materiale Didactice</p>
          <button onClick={()=>{ setFormMat({tip:"Curs",titlu:"",descriere:"",fisier:null,numeFisier:"",marime:""}); setShowAddMat(true); }}
            style={{background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:9,padding:"7px 14px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
            + Adaugă
          </button>
        </div>

        {/* Filtre tip */}
        <div style={{display:"flex",gap:6,marginBottom:14,flexWrap:"wrap"}}>
          {["toate","Curs","Test","Simulator","Video"].map(t=>(
            <button key={t} onClick={()=>setMatTab(t)}
              style={{padding:"5px 12px",borderRadius:20,border:"none",fontWeight:600,fontSize:10,cursor:"pointer",
                background:matTab===t?CLACCENT:"rgba(0,0,0,0.05)",
                color:matTab===t?"#fff":"#666"}}>
              {t==="toate"?"📚 Toate":t==="Curs"?"📖 Cursuri":t==="Test"?"📝 Teste":t==="Simulator"?"🖥️ Simulator":"🎬 Video"}
            </button>
          ))}
        </div>

        {/* Lista materiale */}
        {materiale.filter(m=>matTab==="toate"||m.tip===matTab).map((mat,i)=>{
          const tipIcon = mat.tip==="Curs"?"📖":mat.tip==="Test"?"📝":mat.tip==="Simulator"?"🖥️":"🎬";
          const tipClr  = mat.tip==="Curs"?"#3B82F6":mat.tip==="Test"?CLACCENT:mat.tip==="Simulator"?"#8B5CF6":"#1A936F";
          return (
            <div key={mat.id} style={{background:"#fff",border:"1px solid "+CLBORD,borderRadius:14,marginBottom:10,overflow:"hidden",boxShadow:"0 1px 6px rgba(0,0,0,0.05)"}}>
              <div style={{padding:"13px 15px"}}>
                <div style={{display:"flex",alignItems:"flex-start",gap:12}}>
                  {/* Icon tip */}
                  <div style={{width:44,height:44,borderRadius:11,background:tipClr+"15",border:"1px solid "+tipClr+"33",display:"flex",alignItems:"center",justifyContent:"center",fontSize:20,flexShrink:0}}>
                    {tipIcon}
                  </div>
                  <div style={{flex:1,minWidth:0}}>
                    <div style={{display:"flex",alignItems:"center",gap:6,marginBottom:4}}>
                      <div style={{fontSize:9,fontWeight:700,color:tipClr,background:tipClr+"12",border:"1px solid "+tipClr+"22",borderRadius:6,padding:"2px 7px",letterSpacing:0.5}}>{mat.tip.toUpperCase()}</div>
                      <div style={{fontSize:9,color:"#aaa"}}>{mat.marime}</div>
                    </div>
                    <div style={{fontWeight:800,fontSize:13,color:"#111",marginBottom:3}}>{mat.titlu}</div>
                    <div style={{fontSize:11,color:"#888"}}>{mat.descriere}</div>
                    <div style={{fontSize:10,color:"#bbb",marginTop:4}}>📎 {mat.fisier} · {mat.data}</div>
                  </div>
                </div>

                {/* Elevi alocați */}
                <div style={{marginTop:10,paddingTop:10,borderTop:"1px solid #f5f5f5"}}>
                  <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:6}}>
                    <div style={{fontSize:9,color:"#aaa",letterSpacing:1,textTransform:"uppercase"}}>
                      Alocat la {mat.alocatLa.length} elev{mat.alocatLa.length!==1?"i":""}
                    </div>
                    <button onClick={()=>{ setShowAlocaMat(mat); setAlocaElevi([...mat.alocatLa]); }}
                      style={{background:CLACCENT+"10",border:"1px solid "+CLACCENT+"30",color:CLACCENT,borderRadius:7,padding:"4px 10px",fontSize:10,fontWeight:700,cursor:"pointer"}}>
                      ✏️ Alocă
                    </button>
                  </div>
                  {mat.alocatLa.length===0
                    ? <div style={{fontSize:10,color:"#ccc",fontStyle:"italic"}}>Niciun elev alocat</div>
                    : <div style={{display:"flex",flexWrap:"wrap",gap:5}}>
                        {mat.alocatLa.map((e,j)=>(
                          <div key={j} style={{background:"#f5f5f5",border:"1px solid #eee",borderRadius:20,padding:"3px 10px",fontSize:10,fontWeight:600,color:"#555",display:"flex",alignItems:"center",gap:5}}>
                            👤 {e}
                            <span onClick={()=>setMateriale(prev=>prev.map(m=>m.id===mat.id?{...m,alocatLa:m.alocatLa.filter(x=>x!==e)}:m))} style={{cursor:"pointer",color:"#ccc",fontSize:12,lineHeight:1}}>×</span>
                          </div>
                        ))}
                      </div>
                  }
                </div>

                {/* Butoane */}
                  <div style={{display:"flex",gap:7,marginTop:10}}>
                    {mat.fileUrl && (
                      <a href={mat.fileUrl} target="_blank" rel="noreferrer" download={mat.fisier}
                        style={{flex:1,background:"rgba(0,0,0,0.05)",border:"1px solid #eee",color:"#555",borderRadius:8,padding:"8px",fontWeight:700,fontSize:11,cursor:"pointer",textDecoration:"none",textAlign:"center"}}>
                        {mat.fileType?.includes("video")?"▶ Preview":"📄 Preview"}
                      </a>
                    )}
                    <button onClick={()=>{ setShowAlocaMat(mat); setAlocaElevi([...mat.alocatLa]); }}
                      style={{flex:1,background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:8,padding:"8px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                      👥 Alocă Elevi
                    </button>
                    <button onClick={()=>setMateriale(prev=>prev.filter(m=>m.id!==mat.id))}
                      style={{background:"#fff",border:"1px solid #fcc",color:"#c00",borderRadius:8,padding:"8px 12px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                      🗑️
                    </button>
                  </div>
              </div>
            </div>
          );
        })}

        {materiale.filter(m=>matTab==="toate"||m.tip===matTab).length===0 && (
          <div style={{textAlign:"center",padding:"40px 0",color:"#ccc"}}>
            <div style={{fontSize:40,marginBottom:10}}>📚</div>
            <div style={{fontSize:13}}>Niciun material în această categorie</div>
          </div>
        )}

        {/* MODAL ADAUGĂ MATERIAL */}
        {showAddMat && (
          <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.5)",zIndex:500,display:"flex",alignItems:"flex-end",justifyContent:"center"}}>
            <div style={{width:"100%",maxWidth:420,background:"#fff",borderRadius:"20px 20px 0 0",padding:"22px 20px 34px",animation:"matIn 0.3s ease",maxHeight:"90vh",overflowY:"auto"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:16}}>
                <div style={{fontWeight:800,fontSize:17}}>📚 Material Nou</div>
                <button onClick={()=>setShowAddMat(false)} style={{background:"rgba(0,0,0,0.07)",border:"none",borderRadius:8,width:30,height:30,cursor:"pointer",fontSize:15}}>✕</button>
              </div>

              {/* Tip */}
              <div style={{marginBottom:12}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:6}}>Tip Material</div>
                <div style={{display:"flex",gap:6,flexWrap:"wrap"}}>
                  {["Curs","Test","Simulator","Video"].map(t=>(
                    <button key={t} onClick={()=>setFormMat(p=>({...p,tip:t}))}
                      style={{flex:1,padding:"8px 4px",borderRadius:9,border:"none",fontWeight:700,fontSize:11,cursor:"pointer",minWidth:60,
                        background:formMat.tip===t?"linear-gradient(135deg,"+CLACCENT+",#8B000A)":"rgba(0,0,0,0.05)",
                        color:formMat.tip===t?"#fff":"#666"}}>
                      {t==="Curs"?"📖 Curs":t==="Test"?"📝 Test":t==="Simulator"?"🖥️ Sim":"🎬 Video"}
                    </button>
                  ))}
                </div>
              </div>

              {/* Titlu */}
              <div style={{marginBottom:11}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Titlu</div>
                <input value={formMat.titlu} onChange={e=>setFormMat(p=>({...p,titlu:e.target.value}))} placeholder="ex: Regulile de circulație Cap. 1"
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",boxSizing:"border-box",color:"#111"}}/>
              </div>

              {/* Descriere */}
              <div style={{marginBottom:11}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Descriere</div>
                <textarea value={formMat.descriere} onChange={e=>setFormMat(p=>({...p,descriere:e.target.value}))} placeholder="Scurtă descriere a materialului..." rows={2}
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",boxSizing:"border-box",color:"#111",resize:"none",fontFamily:"inherit"}}/>
              </div>

              {/* Upload fișier */}
              <div style={{marginBottom:16}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:6}}>Fișier (PDF, MP4, DOC, PPT)</div>
                <label style={{display:"flex",alignItems:"center",justifyContent:"center",minHeight:90,borderRadius:12,border:"2px dashed "+(formMat.numeFisier?"#1A936F":"#ddd"),background:formMat.numeFisier?"#f0fff8":"#fafafa",cursor:"pointer",gap:10,transition:"all 0.2s",padding:"12px",flexDirection:"column"}}>
                  {formMat.numeFisier
                    ? <>
                        <div style={{fontSize:32}}>{formMat.numeFisier.endsWith(".pdf")?"📄":formMat.numeFisier.endsWith(".mp4")?"🎬":formMat.numeFisier.endsWith(".ppt")||formMat.numeFisier.endsWith(".pptx")?"📊":"📝"}</div>
                        <div style={{fontWeight:700,fontSize:13,color:"#1A936F",textAlign:"center"}}>{formMat.numeFisier}</div>
                        <div style={{fontSize:11,color:"#aaa"}}>{formMat.marime} · Încărcat ✅</div>
                      </>
                    : <>
                        <div style={{fontSize:32}}>📂</div>
                        <div style={{fontSize:13,color:"#aaa",fontWeight:600}}>Apasă pentru a selecta fișierul</div>
                        <div style={{fontSize:10,color:"#ccc",marginTop:2}}>PDF, MP4, DOC, PPT — orice mărime</div>
                      </>
                  }
                  <input type="file" accept=".pdf,.mp4,.doc,.docx,.ppt,.pptx,.png,.jpg" style={{display:"none"}} onChange={ev=>{
                    const f=ev.target.files[0]; if(!f) return;
                    const kb=f.size/1024;
                    const marime=kb>1024?`${(kb/1024).toFixed(1)} MB`:`${Math.round(kb)} KB`;
                    const reader=new FileReader();
                    reader.onload=e2=>{
                      setFormMat(p=>({...p,numeFisier:f.name,marime,fileUrl:e2.target.result,fileType:f.type,_file:f}));
                    };
                    reader.readAsDataURL(f);
                  }}/>
                </label>
                {formMat.numeFisier && (
                  <button onClick={()=>setFormMat(p=>({...p,numeFisier:"",marime:"",fileUrl:null,fileType:""}))}
                    style={{marginTop:6,background:"none",border:"none",color:"#c00",fontSize:11,cursor:"pointer",fontWeight:600}}>
                    ✕ Elimină fișierul
                  </button>
                )}
              </div>

              <button onClick={async ()=>{
                if(!formMat.titlu.trim() || savingMat) return;
                setSavingMat(true);
                try {
                  if (addMaterialDB) {
                    const file = formMat._file || null;
                    await addMaterialDB({
                      tip:formMat.tip, titlu:formMat.titlu, descriere:formMat.descriere,
                      marime:formMat.marime||"", fileUrl:formMat.fileUrl||null,
                    }, file);
                  } else {
                    setMateriale(prev=>[...prev,{
                      id:Date.now(), tip:formMat.tip, titlu:formMat.titlu, descriere:formMat.descriere,
                      fisier:formMat.numeFisier||"—", marime:formMat.marime||"—",
                      fileUrl:formMat.fileUrl||null, fileType:formMat.fileType||"",
                      data:new Date().toLocaleDateString("ro-RO",{day:"numeric",month:"long",year:"numeric"}), alocatLa:[],
                    }]);
                  }
                  setFormMat({tip:"Curs",titlu:"",descriere:"",fisier:null,numeFisier:"",marime:"",fileUrl:null,fileType:"",_file:null});
                  setShowAddMat(false);
                } catch(e) { alert("Eroare la salvare: " + e.message); }
                finally { setSavingMat(false); }
              }} style={{width:"100%",background:(!formMat.titlu.trim()||savingMat)?"#ddd":"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:(!formMat.titlu.trim()||savingMat)?"#aaa":"#fff",border:"none",borderRadius:11,padding:"13px",fontWeight:800,fontSize:14,cursor:(!formMat.titlu.trim()||savingMat)?"default":"pointer",boxShadow:(!formMat.titlu.trim()||savingMat)?"none":"0 4px 16px "+CLACCENT+"33"}}>
                {savingMat ? "⏳ Se salvează..." : "✓ Salvează Material"}
              </button>
            </div>
          </div>
        )}

        {/* MODAL ALOCARE ELEVI */}
        {showAlocaMat && (
          <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.5)",zIndex:500,display:"flex",alignItems:"flex-end",justifyContent:"center"}}>
            <div style={{width:"100%",maxWidth:420,background:"#fff",borderRadius:"20px 20px 0 0",padding:"22px 20px 34px",animation:"matIn 0.3s ease",maxHeight:"85vh",overflowY:"auto"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:6}}>
                <div style={{fontWeight:800,fontSize:16}}>👥 Alocă Elevi</div>
                <button onClick={()=>setShowAlocaMat(null)} style={{background:"rgba(0,0,0,0.07)",border:"none",borderRadius:8,width:30,height:30,cursor:"pointer",fontSize:15}}>✕</button>
              </div>
              <div style={{fontSize:12,color:"#888",marginBottom:14}}>📖 {showAlocaMat.titlu}</div>

              {/* Selectează tot / nimic */}
              <div style={{display:"flex",gap:8,marginBottom:12}}>
                <button onClick={()=>setAlocaElevi(elevi.map(e=>e.nume))} style={{flex:1,background:"rgba(0,0,0,0.05)",border:"1px solid #eee",borderRadius:8,padding:"7px",fontSize:11,fontWeight:700,cursor:"pointer",color:"#555"}}>✓ Toți</button>
                <button onClick={()=>setAlocaElevi([])} style={{flex:1,background:"rgba(0,0,0,0.05)",border:"1px solid #eee",borderRadius:8,padding:"7px",fontSize:11,fontWeight:700,cursor:"pointer",color:"#555"}}>✗ Niciunul</button>
              </div>

              {/* Lista elevi cu checkbox */}
              {elevi.map((elev,i)=>{
                const sel = alocaElevi.includes(elev.nume);
                return (
                  <div key={i} onClick={()=>setAlocaElevi(prev=>sel?prev.filter(x=>x!==elev.nume):[...prev,elev.nume])}
                    style={{display:"flex",alignItems:"center",gap:12,padding:"11px 14px",borderRadius:11,marginBottom:7,cursor:"pointer",
                      background:sel?CLACCENT+"08":"#fafafa",
                      border:"1px solid "+(sel?CLACCENT+"44":"#eee"),
                      transition:"all 0.15s"}}>
                    {/* Checkbox custom */}
                    <div style={{width:22,height:22,borderRadius:6,border:"2px solid "+(sel?CLACCENT:"#ddd"),background:sel?CLACCENT:"#fff",display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0,transition:"all 0.15s"}}>
                      {sel && <span style={{color:"#fff",fontSize:13,fontWeight:900}}>✓</span>}
                    </div>
                    <div style={{flex:1}}>
                      <div style={{fontWeight:700,fontSize:13,color:"#111"}}>{elev.nume}</div>
                      <div style={{fontSize:10,color:"#aaa",marginTop:1}}>{elev.masina} · {elev.ore}/{elev.oreMax}h</div>
                    </div>
                    <div style={{fontSize:10,color:elev.status==="promovat"?"#1A936F":CLACCENT,fontWeight:600}}>{elev.status}</div>
                  </div>
                );
              })}

              <div style={{fontSize:11,color:"#aaa",textAlign:"center",marginBottom:12}}>{alocaElevi.length} elev{alocaElevi.length!==1?"i":"" } selectat{alocaElevi.length!==1?"i":""}</div>

              <button onClick={async ()=>{
                try {
                  // alocaElevi = array de nume elevi → convertim la ID-uri
                  const elevIds = alocaElevi.map(nume => elevi.find(e => e.nume === nume)?.id).filter(Boolean);
                  if (alocaMaterialDB) {
                    await alocaMaterialDB(showAlocaMat.id, elevIds);
                  } else {
                    setMateriale(prev=>prev.map(m=>m.id===showAlocaMat.id?{...m,alocatLa:[...alocaElevi]}:m));
                  }
                  setShowAlocaMat(null);
                } catch(e) { alert("Eroare: " + e.message); }
              }} style={{width:"100%",background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:11,padding:"13px",fontWeight:800,fontSize:14,cursor:"pointer",boxShadow:"0 4px 16px "+CLACCENT+"33"}}>
                ✓ Salvează Alocarea
              </button>
            </div>
          </div>
        )}
      </div>
    )}

    {tab === "flota" && (
      <div>
        <style>{`@keyframes mIn{from{opacity:0;transform:translateY(28px)}to{opacity:1;transform:translateY(0)}}`}</style>

        {/* Sub-tabs */}
        <div style={{display:"flex",gap:8,marginBottom:16}}>
          {[{id:"masini",label:"🚗 Mașini"},{id:"instructori",label:"👨‍🏫 Instructori"}].map(t=>(
            <button key={t.id} onClick={()=>setFlotaTab(t.id)} style={{flex:1,padding:"9px",borderRadius:9,border:"none",fontWeight:700,fontSize:12,cursor:"pointer",
              background:flotaTab===t.id?"linear-gradient(135deg,"+CLACCENT+",#8B000A)":CLCARD,
              color:flotaTab===t.id?"#fff":"#555",
              boxShadow:flotaTab===t.id?"0 3px 12px "+CLACCENT+"33":"none"}}>{t.label}</button>
          ))}
        </div>

        {/* MAȘINI */}
        {flotaTab==="masini" && (
          <div>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
              <p style={{fontSize:10,color:"#888",letterSpacing:2,textTransform:"uppercase",margin:0}}>Vehicule ({masini.length})</p>
              <button onClick={()=>{setFormMasina({id:"",model:"",an:"",airtag:"",km:"",culoare:"",status:"parcat",foto:null});setEditMasina("new");}}
                style={{background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:9,padding:"7px 14px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                + Adaugă Mașină
              </button>
            </div>
            <LiveMap simActive={false} simPos={ROUTE_GPS[0]} simTrail={[]} mapHeight={170}/>
            <div style={{marginTop:12}}>
              {masini.map((m,i)=>(
                <div key={i} style={{background:"#fff",border:"1px solid "+CLBORD,borderRadius:14,marginBottom:10,overflow:"hidden",boxShadow:"0 2px 8px rgba(0,0,0,0.06)"}}>
                  {/* Photo area */}
                  <div style={{position:"relative",height:m.foto?150:80,background:m.foto?"#000":"#f5f5f5",display:"flex",alignItems:"center",justifyContent:"center",overflow:"hidden"}}>
                    {m.foto
                      ? <img src={m.foto} alt={m.model} style={{width:"100%",height:"100%",objectFit:"cover"}}/>
                      : <div style={{textAlign:"center"}}><div style={{fontSize:32}}>🚗</div><div style={{fontSize:10,color:"#ccc",marginTop:4}}>Fără fotografie</div></div>
                    }
                    <div style={{position:"absolute",top:8,left:8,background:m.status==="în mișcare"?"#1A936F":"rgba(0,0,0,0.55)",color:"#fff",fontSize:9,fontWeight:700,padding:"3px 9px",borderRadius:20,letterSpacing:0.5}}>
                      {m.status==="în mișcare"?"🟢 LIVE":"⚫ Parcat"}
                    </div>
                    <div style={{position:"absolute",top:8,right:8,display:"flex",gap:6}}>
                      <label style={{background:"rgba(255,255,255,0.95)",border:"1px solid #ddd",borderRadius:7,padding:"5px 10px",fontWeight:700,fontSize:10,cursor:"pointer",color:"#333"}}>
                        📷 Foto
                        <input type="file" accept="image/*" style={{display:"none"}} onChange={ev=>{
                          const file=ev.target.files[0];if(!file)return;
                          const rd=new FileReader();rd.onload=e2=>setMasini(prev=>prev.map((x,j)=>j===i?{...x,foto:e2.target.result}:x));rd.readAsDataURL(file);
                        }}/>
                      </label>
                      <button onClick={()=>{setFormMasina({...m});setEditMasina(m);}}
                        style={{background:"rgba(255,255,255,0.95)",border:"1px solid #ddd",borderRadius:7,padding:"5px 10px",fontWeight:700,fontSize:10,cursor:"pointer",color:"#333"}}>
                        ✏️
                      </button>
                    </div>
                  </div>
                  {/* Info */}
                  <div style={{padding:"12px 14px"}}>
                    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:9}}>
                      <div>
                        <div style={{fontWeight:800,fontSize:14}}>{m.model}</div>
                        <div style={{fontSize:10,color:"#888",marginTop:2}}>{m.id} · {m.an} · {m.culoare}</div>
                      </div>
                      <div style={{background:CLACCENT+"12",border:"1px solid "+CLACCENT+"30",borderRadius:8,padding:"4px 10px",textAlign:"center"}}>
                        <div style={{fontSize:13,fontWeight:800,color:CLACCENT}}>{Number(m.km).toLocaleString()}</div>
                        <div style={{fontSize:8,color:"#aaa"}}>km</div>
                      </div>
                    </div>
                    {[
                      ["AirTag", m.airtag],
                      ["Instructor", instructori.find(x=>x.masina===m.id)?.nume||"—"],
                    ].map(([l,v],j)=>(
                      <div key={j} style={{display:"flex",justifyContent:"space-between",padding:"5px 0",borderTop:"1px solid rgba(0,0,0,0.06)"}}>
                        <span style={{fontSize:10,color:"#888"}}>{l}</span>
                        <span style={{fontSize:10,fontWeight:600}}>{v}</span>
                      </div>
                    ))}
                    <div style={{display:"flex",gap:7,marginTop:10}}>
                      <button onClick={()=>generateFoaieParcurs(m)}
                        style={{flex:1,background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:8,padding:"8px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                        📄 Foaie Parcurs
                      </button>
                      <button onClick={()=>setMasini(prev=>prev.filter((_,j)=>j!==i))}
                        style={{background:"#fff",border:"1px solid #fcc",color:"#c00",borderRadius:8,padding:"8px 12px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                        🗑️
                      </button>
                    </div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* INSTRUCTORI */}
        {flotaTab==="instructori" && (
          <div>
            <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12}}>
              <p style={{fontSize:10,color:"#888",letterSpacing:2,textTransform:"uppercase",margin:0}}>Instructori ({instructori.length})</p>
              <button onClick={()=>{setFormInstr({id:Date.now(),nume:"",telefon:"",autorizatie:"",valabilitate:"",masina:masini[0]?.id||"",foto:null});setEditInstructor("new");}}
                style={{background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:9,padding:"7px 14px",fontWeight:700,fontSize:11,cursor:"pointer"}}>
                + Adaugă Instructor
              </button>
            </div>
            {instructori.map((instr,i)=>(
              <div key={i} style={{background:"#fff",border:"1px solid "+CLBORD,borderRadius:14,marginBottom:10,overflow:"hidden",boxShadow:"0 2px 8px rgba(0,0,0,0.06)"}}>
                <div style={{display:"flex",gap:14,padding:"14px",alignItems:"flex-start"}}>
                  {/* Avatar */}
                  <div style={{position:"relative",flexShrink:0}}>
                    <div style={{width:76,height:76,borderRadius:12,overflow:"hidden",background:"#f5f5f5",border:"2px solid #eee",display:"flex",alignItems:"center",justifyContent:"center"}}>
                      {instr.foto
                        ? <img src={instr.foto} alt={instr.nume} style={{width:"100%",height:"100%",objectFit:"cover"}}/>
                        : <span style={{fontSize:30}}>👤</span>
                      }
                    </div>
                    <label style={{position:"absolute",bottom:-5,right:-5,background:"#fff",border:"1px solid #ddd",borderRadius:6,padding:"3px 7px",fontSize:10,cursor:"pointer",color:"#555",fontWeight:700,boxShadow:"0 1px 4px rgba(0,0,0,0.1)"}}>
                      📷
                      <input type="file" accept="image/*" style={{display:"none"}} onChange={ev=>{
                        const file=ev.target.files[0];if(!file)return;
                        const rd=new FileReader();rd.onload=e2=>setInstructori(prev=>prev.map((x,j)=>j===i?{...x,foto:e2.target.result}:x));rd.readAsDataURL(file);
                      }}/>
                    </label>
                  </div>
                  {/* Info */}
                  <div style={{flex:1,minWidth:0}}>
                    <div style={{fontWeight:800,fontSize:14}}>{instr.nume}</div>
                    <div style={{fontSize:10,color:"#888",marginTop:2}}>{instr.telefon}</div>
                    <div style={{fontSize:10,color:"#888",marginTop:1}}>Mașina: <strong>{instr.masina}</strong></div>
                    <div style={{display:"flex",gap:5,marginTop:8,flexWrap:"wrap"}}>
                      <div style={{background:CLACCENT+"12",border:"1px solid "+CLACCENT+"30",borderRadius:6,padding:"3px 8px",fontSize:9,fontWeight:700,color:CLACCENT}}>{instr.autorizatie}</div>
                      <div style={{background:"rgba(0,0,0,0.04)",border:"1px solid #eee",borderRadius:6,padding:"3px 8px",fontSize:9,fontWeight:600,color:"#666"}}>Val: {instr.valabilitate}</div>
                    </div>
                  </div>
                  {/* Actions */}
                  <div style={{display:"flex",flexDirection:"column",gap:6,flexShrink:0}}>
                    <button onClick={()=>{setFormInstr({...instr});setEditInstructor(instr);}}
                      style={{background:"rgba(0,0,0,0.04)",border:"1px solid #eee",borderRadius:7,padding:"6px 11px",fontWeight:700,fontSize:10,cursor:"pointer",color:"#333"}}>✏️ Edit</button>
                    <button onClick={()=>setInstructori(prev=>prev.filter((_,j)=>j!==i))}
                      style={{background:"rgba(192,0,8,0.05)",border:"1px solid #fcc",borderRadius:7,padding:"6px 11px",fontWeight:700,fontSize:10,cursor:"pointer",color:"#c00"}}>🗑️</button>
                  </div>
                </div>
                {/* Elevi */}
                <div style={{borderTop:"1px solid #f0f0f0",padding:"8px 14px",background:"#fafafa"}}>
                  <div style={{fontSize:9,color:"#bbb",marginBottom:5,letterSpacing:1,textTransform:"uppercase"}}>Elevi Atribuiți</div>
                  <div style={{display:"flex",flexWrap:"wrap",gap:5}}>
                    {elevi.filter(e=>e.masina===instr.masina).map((e,j)=>(
                      <div key={j} style={{background:"#fff",border:"1px solid #eee",borderRadius:6,padding:"3px 9px",fontSize:10,fontWeight:600}}>{e.nume}</div>
                    ))}
                    {elevi.filter(e=>e.masina===instr.masina).length===0 &&
                      <span style={{fontSize:10,color:"#ccc"}}>Niciun elev</span>}
                  </div>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* MODAL MAȘINĂ */}
        {editMasina && (
          <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.5)",zIndex:500,display:"flex",alignItems:"flex-end",justifyContent:"center"}}>
            <div style={{width:"100%",maxWidth:420,background:"#fff",borderRadius:"20px 20px 0 0",padding:"22px 20px 34px",animation:"mIn 0.3s ease",maxHeight:"90vh",overflowY:"auto"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:16}}>
                <div style={{fontWeight:800,fontSize:17}}>{editMasina==="new"?"Mașină Nouă":"Editează Mașina"}</div>
                <button onClick={()=>setEditMasina(null)} style={{background:"rgba(0,0,0,0.07)",border:"none",borderRadius:8,width:30,height:30,cursor:"pointer",fontSize:15}}>✕</button>
              </div>
              {/* Photo upload */}
              <label style={{display:"flex",alignItems:"center",justifyContent:"center",height:120,borderRadius:10,border:"2px dashed #ddd",background:"#fafafa",cursor:"pointer",overflow:"hidden",marginBottom:14}}>
                {formMasina.foto
                  ? <img src={formMasina.foto} alt="" style={{width:"100%",height:"100%",objectFit:"cover"}}/>
                  : <div style={{textAlign:"center"}}><div style={{fontSize:32}}>🚗</div><div style={{fontSize:11,color:"#bbb",marginTop:4}}>Apasă pentru poză mașină</div></div>
                }
                <input type="file" accept="image/*" style={{display:"none"}} onChange={ev=>{
                  const file=ev.target.files[0];if(!file)return;
                  const rd=new FileReader();rd.onload=e=>setFormMasina(p=>({...p,foto:e.target.result}));rd.readAsDataURL(file);
                }}/>
              </label>
              {[
                ["Nr. Înmatriculare","id","text"],
                ["Marcă / Model","model","text"],
                ["An fabricație","an","number"],
                ["Culoare","culoare","text"],
                ["Km la bord","km","number"],
                ["AirTag ID","airtag","text"],
              ].map(([lbl,key,type])=>(
                <div key={key} style={{marginBottom:10}}>
                  <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>{lbl}</div>
                  <input type={type} value={formMasina[key]||""} onChange={e=>setFormMasina(p=>({...p,[key]:e.target.value}))}
                    style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",boxSizing:"border-box",color:"#111"}}/>
                </div>
              ))}
              <div style={{marginBottom:14}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Status</div>
                <select value={formMasina.status||"parcat"} onChange={e=>setFormMasina(p=>({...p,status:e.target.value}))}
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",color:"#111"}}>
                  <option value="parcat">⚫ Parcat</option>
                  <option value="în mișcare">🟢 În mișcare</option>
                  <option value="service">🔧 Service</option>
                </select>
              </div>
              <button onClick={()=>{
                const saved={...formMasina,km:Number(formMasina.km)||0,an:Number(formMasina.an)||2024};
                if(editMasina==="new") setMasini(prev=>[...prev,saved]);
                else setMasini(prev=>prev.map(x=>x.id===editMasina.id?{...x,...saved}:x));
                setEditMasina(null);
              }} style={{width:"100%",background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:11,padding:"13px",fontWeight:800,fontSize:14,cursor:"pointer",boxShadow:"0 4px 16px "+CLACCENT+"33"}}>
                ✓ Salvează
              </button>
            </div>
          </div>
        )}

        {/* MODAL INSTRUCTOR */}
        {editInstructor && (
          <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.5)",zIndex:500,display:"flex",alignItems:"flex-end",justifyContent:"center"}}>
            <div style={{width:"100%",maxWidth:420,background:"#fff",borderRadius:"20px 20px 0 0",padding:"22px 20px 34px",animation:"mIn 0.3s ease",maxHeight:"90vh",overflowY:"auto"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:16}}>
                <div style={{fontWeight:800,fontSize:17}}>{editInstructor==="new"?"Instructor Nou":"Editează Instructor"}</div>
                <button onClick={()=>setEditInstructor(null)} style={{background:"rgba(0,0,0,0.07)",border:"none",borderRadius:8,width:30,height:30,cursor:"pointer",fontSize:15}}>✕</button>
              </div>
              {/* Photo upload */}
              <label style={{display:"flex",alignItems:"center",justifyContent:"center",height:120,borderRadius:10,border:"2px dashed #ddd",background:"#fafafa",cursor:"pointer",overflow:"hidden",marginBottom:14}}>
                {formInstr.foto
                  ? <img src={formInstr.foto} alt="" style={{width:"100%",height:"100%",objectFit:"cover"}}/>
                  : <div style={{textAlign:"center"}}><div style={{fontSize:36}}>👤</div><div style={{fontSize:11,color:"#bbb",marginTop:4}}>Apasă pentru poza instructorului</div></div>
                }
                <input type="file" accept="image/*" style={{display:"none"}} onChange={ev=>{
                  const file=ev.target.files[0];if(!file)return;
                  const rd=new FileReader();rd.onload=e=>setFormInstr(p=>({...p,foto:e.target.result}));rd.readAsDataURL(file);
                }}/>
              </label>
              {[
                ["Nume complet","nume","text"],
                ["Telefon","telefon","tel"],
                ["Nr. Autorizație","autorizatie","text"],
                ["Valabilitate (ex: 31.12.2026)","valabilitate","text"],
              ].map(([lbl,key,type])=>(
                <div key={key} style={{marginBottom:10}}>
                  <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>{lbl}</div>
                  <input type={type} value={formInstr[key]||""} onChange={e=>setFormInstr(p=>({...p,[key]:e.target.value}))}
                    style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",boxSizing:"border-box",color:"#111"}}/>
                </div>
              ))}
              <div style={{marginBottom:14}}>
                <div style={{fontSize:9,color:"#888",letterSpacing:1,textTransform:"uppercase",marginBottom:5}}>Mașina Atribuită</div>
                <select value={formInstr.masina||""} onChange={e=>setFormInstr(p=>({...p,masina:e.target.value}))}
                  style={{width:"100%",background:"#f8f8f8",border:"1px solid #e8e8e8",borderRadius:9,padding:"10px 11px",fontSize:12,outline:"none",color:"#111"}}>
                  {masini.map(m=><option key={m.id} value={m.id}>{m.id} · {m.model}</option>)}
                </select>
              </div>
              <button onClick={()=>{
                if(editInstructor==="new") setInstructori(prev=>[...prev,{...formInstr}]);
                else setInstructori(prev=>prev.map(x=>x.id===editInstructor.id?{...x,...formInstr}:x));
                setEditInstructor(null);
              }} style={{width:"100%",background:"linear-gradient(135deg,"+CLACCENT+",#8B000A)",color:"#fff",border:"none",borderRadius:11,padding:"13px",fontWeight:800,fontSize:14,cursor:"pointer",boxShadow:"0 4px 16px "+CLACCENT+"33"}}>
                ✓ Salvează
              </button>
            </div>
          </div>
        )}

      </div>
    )}

    {tab === "plati" && (
      <div>
        <p style={{ fontSize:10, color:"#555", letterSpacing:2, textTransform:"uppercase", marginBottom:11 }}>Situație Plăți</p>
        <div style={{ display:"flex", gap:9, marginBottom:16 }}>
          {[
            { label:"Achitat",  val: elevi.filter((e) => e.plata === "achitat").length, clr:CLGRN },
            { label:"Restanți", val: restanti, clr:CLRED },
            { label:"Total",    val: elevi.length, clr:CLBLU },
          ].map((st, i) => (
            <div key={i} style={{ flex:1, background:st.clr + "10", border:"1px solid " + st.clr + "33", borderRadius:11, padding:"13px", textAlign:"center" }}>
              <div style={{ fontSize:22, fontWeight:900, color:st.clr }}>{st.val}</div>
              <div style={{ fontSize:10, color:"#555", marginTop:3 }}>{st.label}</div>
            </div>
          ))}
        </div>
        {elevi.map((elev, i) => (
          <div key={i} style={{ background:CLCARD, border:"1px solid " + (elev.plata === "restant" ? CLRED + "33" : CLBORD), borderRadius:11, padding:"12px 15px", marginBottom:7, display:"flex", alignItems:"center", gap:11 }}>
            <div style={{ width:34, height:34, borderRadius:9, background: elev.plata === "achitat" ? CLGRN + "14" : CLRED + "14", border:"1px solid " + (elev.plata === "achitat" ? CLGRN : CLRED) + "33", display:"flex", alignItems:"center", justifyContent:"center", fontSize:15 }}>
              {elev.plata === "achitat" ? "✅" : "⚠️"}
            </div>
            <div style={{ flex:1 }}>
              <div style={{ fontWeight:700, fontSize:12 }}>{elev.nume}</div>
              <div style={{ fontSize:10, color:"#555", marginTop:2 }}>Din {elev.data}</div>
            </div>
            <div style={{ textAlign:"right" }}>
              <div style={{ fontSize:11, fontWeight:700, color: elev.plata === "achitat" ? CLGRN : CLRED }}>{elev.plata === "achitat" ? "Achitat" : "Restant"}</div>
              {elev.plata === "restant" && (
                <button style={{ marginTop:4, background:CLRED + "14", border:"1px solid " + CLRED + "33", color:CLRED, borderRadius:7, padding:"4px 9px", fontSize:10, fontWeight:700, cursor:"pointer" }}>Notifică</button>
              )}
            </div>
          </div>
        ))}
      </div>
    )}
  </div>

  {/* Foaie de Parcurs Modal — mobile-first */}
  {foaieModal && (() => {
    const masina = foaieModal;
    const elevCar = elevi.filter(e => e.masina === masina.id);
    const sesiuniCar = (sesiuni||[]).filter(s => s.masina === masina.id);
    const el = elevCar[0] || {};
    const totalKm = sesiuniCar.reduce((a,s) => a + s.km, 0).toFixed(1);
    const nrDoc = "1" + masina.id.replace(/[^0-9]/g,"") + "24";
    const dataAzi = new Date().toLocaleDateString("ro-RO",{day:"numeric",month:"long",year:"numeric"});
    const instr = instructori.find(x=>x.masina===masina.id) || {};
    const emptyRows = Array.from({length:Math.max(0,6-sesiuniCar.length)});
    const Row = ({l,v}) => (
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",padding:"6px 0",borderBottom:"1px solid #f0f0f0"}}>
        <span style={{fontSize:11,color:"#888",flexShrink:0,marginRight:8}}>{l}</span>
        <span style={{fontSize:11,fontWeight:700,textAlign:"right"}}>{v}</span>
      </div>
    );
    return (
      <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.7)",zIndex:99999,display:"flex",flexDirection:"column",fontFamily:"'DM Sans','Arial',sans-serif"}}>
        <style>{`.leaflet-pane,.leaflet-top,.leaflet-bottom,.leaflet-control{z-index:0 !important}`}</style>

        {/* Toolbar */}
        <div style={{background:"#fff",borderBottom:"2px solid "+CLACCENT,padding:"12px 16px",display:"flex",justifyContent:"space-between",alignItems:"center",flexShrink:0}}>
          <div>
            <div style={{fontWeight:800,fontSize:13,color:"#111"}}>📄 Foaie de Parcurs</div>
            <div style={{fontSize:10,color:"#888",marginTop:1}}>{masina.id} · {masina.model}</div>
          </div>
          <button onClick={()=>setFoaieModal(null)}
            style={{background:"rgba(0,0,0,0.08)",border:"1px solid #ddd",color:"#333",borderRadius:8,padding:"8px 14px",fontWeight:700,fontSize:12,cursor:"pointer"}}>
            ✕
          </button>
        </div>

        {/* Scrollable content */}
        <div style={{flex:1,overflowY:"auto",background:"#f5f5f5",padding:"12px"}}>
          <div style={{background:"#fff",borderRadius:12,overflow:"hidden",boxShadow:"0 2px 12px rgba(0,0,0,0.1)"}}>

            {/* Doc header */}
            <div style={{background:CLACCENT,padding:"14px 16px",display:"flex",justifyContent:"space-between",alignItems:"center"}}>
              <div style={{display:"flex",alignItems:"center",gap:10}}>
                <svg width="36" height="36" viewBox="0 0 200 200">
                  <rect x="30" y="30" width="115" height="115" rx="18" fill="none" stroke="#fff" strokeWidth="16" transform="rotate(45 87.5 87.5)"/>
                  <polyline points="52,100 80,130 130,68" fill="none" stroke="#fff" strokeWidth="16" strokeLinecap="round" strokeLinejoin="round"/>
                </svg>
                <div>
                  <div style={{fontSize:16,fontWeight:900,color:"#fff",letterSpacing:-0.5}}>Super OK</div>
                  <div style={{fontSize:9,color:"rgba(255,255,255,0.7)",letterSpacing:1.5}}>ȘCOALA DE ȘOFERI</div>
                </div>
              </div>
              <div style={{textAlign:"right"}}>
                <div style={{fontSize:10,fontWeight:800,color:"#fff",letterSpacing:1}}>FOAIE DE PARCURS</div>
                <div style={{fontSize:9,color:"rgba(255,255,255,0.75)",marginTop:2}}>Nr. {nrDoc}</div>
                <div style={{fontSize:9,color:"rgba(255,255,255,0.75)",marginTop:1}}>{new Date().toLocaleDateString("ro-RO")}</div>
              </div>
            </div>

            {/* Sections */}
            <div style={{padding:"14px 16px"}}>

              {/* Vehicul */}
              <div style={{marginBottom:14}}>
                <div style={{fontSize:9,fontWeight:700,letterSpacing:1.5,textTransform:"uppercase",color:CLACCENT,marginBottom:8}}>🚗 Date Vehicul</div>
                <Row l="Marcă / Model" v={masina.model}/>
                <Row l="Nr. înmatriculare" v={masina.id}/>
                <Row l="An fabricație" v={String(masina.an)}/>
                <Row l="Km la bord" v={Number(masina.km).toLocaleString()+" km"}/>
                <Row l="AirTag ID" v={masina.airtag}/>
              </div>

              {/* Instructor */}
              <div style={{marginBottom:14}}>
                <div style={{fontSize:9,fontWeight:700,letterSpacing:1.5,textTransform:"uppercase",color:CLACCENT,marginBottom:8}}>👨‍🏫 Date Instructor</div>
                <Row l="Instructor" v={instr.nume||"Prof. Vaida Adrian"}/>
                <Row l="Autorizație" v={instr.autorizatie||"AUT-2024-0892"}/>
                <Row l="Valabilitate" v={instr.valabilitate||"31.12.2026"}/>
                <Row l="Categoria" v="B"/>
              </div>

              {/* Cursant */}
              {el.nume && (
                <div style={{marginBottom:14}}>
                  <div style={{fontSize:9,fontWeight:700,letterSpacing:1.5,textTransform:"uppercase",color:CLACCENT,marginBottom:8}}>🎓 Date Cursant</div>
                  <Row l="Nume" v={el.nume}/>
                  <Row l="Telefon" v={el.telefon||"-"}/>
                  <Row l="Înscris din" v={el.data}/>
                  <Row l="Ore efectuate" v={el.ore+" / "+el.oreMax+"h"}/>
                  <Row l="Progres teorie" v={el.teorie+"%"}/>
                  <Row l="Status plată" v={el.plata==="achitat"?"✓ Achitat":"⚠ Restant"}/>
                </div>
              )}

              {/* Sesiuni */}
              <div style={{marginBottom:14}}>
                <div style={{fontSize:9,fontWeight:700,letterSpacing:1.5,textTransform:"uppercase",color:CLACCENT,marginBottom:8}}>📋 Sesiuni de Conducere</div>
                {sesiuniCar.map((s,i)=>(
                  <div key={i} style={{background:i%2===0?"#fafafa":"#fff",border:"1px solid #f0f0f0",borderRadius:8,padding:"8px 10px",marginBottom:6}}>
                    <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
                      <div style={{display:"flex",alignItems:"center",gap:8}}>
                        <div style={{width:22,height:22,borderRadius:"50%",background:CLACCENT,color:"#fff",fontSize:10,fontWeight:800,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0}}>{i+1}</div>
                        <div>
                          <div style={{fontSize:12,fontWeight:700}}>{s.data}</div>
                          <div style={{fontSize:10,color:"#888"}}>Bistrița - Centru</div>
                        </div>
                      </div>
                      <div style={{textAlign:"right"}}>
                        <div style={{fontSize:12,fontWeight:700,color:CLACCENT}}>{s.km} km</div>
                        <div style={{fontSize:10,color:"#888"}}>{s.durata}</div>
                      </div>
                    </div>
                  </div>
                ))}
                {emptyRows.map((_,i)=>(
                  <div key={"e"+i} style={{border:"1px dashed #eee",borderRadius:8,padding:"8px 10px",marginBottom:6,display:"flex",alignItems:"center",gap:8}}>
                    <div style={{width:22,height:22,borderRadius:"50%",background:"#f0f0f0",color:"#ccc",fontSize:10,fontWeight:800,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0}}>{sesiuniCar.length+i+1}</div>
                    <div style={{fontSize:10,color:"#ddd"}}>—</div>
                  </div>
                ))}
              </div>

              {/* Totals */}
              <div style={{background:CLACCENT,borderRadius:10,padding:"12px 14px",marginBottom:16}}>
                <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
                  <span style={{color:"rgba(255,255,255,0.75)",fontSize:10}}>Total ore conduse</span>
                  <span style={{color:"#fff",fontWeight:800,fontSize:13}}>{masina.id==="B-01-SUK"?"12h 35m":masina.id==="B-02-SUK"?"21h 15m":"8h 50m"}</span>
                </div>
                <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
                  <span style={{color:"rgba(255,255,255,0.75)",fontSize:10}}>Total km parcurși</span>
                  <span style={{color:"#fff",fontWeight:800,fontSize:13}}>{totalKm} km</span>
                </div>
                <div style={{display:"flex",justifyContent:"space-between"}}>
                  <span style={{color:"rgba(255,255,255,0.75)",fontSize:10}}>Număr sesiuni</span>
                  <span style={{color:"#fff",fontWeight:800,fontSize:13}}>{sesiuniCar.length}</span>
                </div>
              </div>

              {/* Signatures */}
              <div style={{marginBottom:14}}>
                <div style={{fontSize:9,fontWeight:700,letterSpacing:1.5,textTransform:"uppercase",color:CLACCENT,marginBottom:10}}>✍️ Semnături</div>
                {[["Instructor",instr.nume||"Prof. Vaida Adrian"],["Cursant",el.nume||""],["Ștampilă Școală",""]].map(([lbl,name],i)=>(
                  <div key={i} style={{marginBottom:14}}>
                    <div style={{fontSize:10,color:"#888",marginBottom:4,fontWeight:600}}>{lbl}</div>
                    <div style={{borderBottom:"1.5px solid #ccc",paddingBottom:20,paddingLeft:4,fontSize:10,color:"#555"}}>{name}</div>
                  </div>
                ))}
              </div>

              {/* Footer */}
              <div style={{borderTop:"1px solid #eee",paddingTop:10,textAlign:"center",fontSize:9,color:"#bbb"}}>
                SuperOk Bistrița · www.superok.ro · +40 263 123 456<br/>
                Generat automat la {new Date().toLocaleString("ro-RO")}
              </div>
            </div>
          </div>
        </div>
      </div>
    );
  })()}

  {/* Add student modal */}
  {showAdd && (
    <div style={{ position:"fixed", inset:0, background:"rgba(0,0,0,0.85)", zIndex:200, display:"flex", alignItems:"flex-end", justifyContent:"center" }}>
      <div style={{ width:"100%", maxWidth:420, background:"#FFFFFF", borderRadius:"20px 20px 0 0", padding:"22px 20px 38px", animation:"slideUp 0.3s ease" }}>
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:18 }}>
          <div style={{ fontWeight:800, fontSize:17 }}>Adaugă Elev Nou</div>
          <button onClick={() => setShowAdd(false)} style={{ background:"rgba(0,0,0,0.08)", border:"none", color:"#111", borderRadius:8, width:30, height:30, cursor:"pointer", fontSize:15 }}>✕</button>
        </div>
        <Field label="Nume complet" value={newElev.nume} onChange={v => setNewElev(p=>({...p, nume:v}))} />
        <Field label="Telefon" type="tel" value={newElev.telefon} onChange={v => setNewElev(p=>({...p, telefon:v}))} />
        <Field label="Parolă elev" type="password" value={newElev.parola} onChange={v => setNewElev(p=>({...p, parola:v}))} />
        <div style={{ fontSize:10, color:"#aaa", marginTop:-6, marginBottom:10 }}>💡 Lasă gol pentru parola implicită = prenumele</div>
        <Field label="Mașina atribuită" value={newElev.masina} onChange={v => setNewElev(p=>({...p, masina:v}))} options={[
          { val:"B-01-SUK", lbl:"B-01-SUK · Dacia Logan"     },
          { val:"B-02-SUK", lbl:"B-02-SUK · VW Golf"         },
          { val:"B-03-SUK", lbl:"B-03-SUK · Škoda Octavia"   },
        ]} />
        <Field label="Status plată" value={newElev.plata} onChange={v => setNewElev(p=>({...p, plata:v}))} options={[
          { val:"achitat", lbl:"✅ Achitat" },
          { val:"restant", lbl:"⚠️ Restant" },
        ]} />
        <button onClick={addElev} style={{ width:"100%", background:"linear-gradient(135deg," + CLACCENT + ",#8B000A)", color:"#111", border:"none", borderRadius:11, padding:"13px", fontWeight:800, fontSize:14, cursor:"pointer", marginTop:7, boxShadow:"0 4px 20px " + CLACCENT + "44" }}>
          ✓ Salvează Elevul
        </button>
      </div>
    </div>
  )}

  <div style={{ position:"fixed", bottom:0, left:"50%", transform:"translateX(-50%)", width:"100%", maxWidth:420, background:"rgba(250,250,250,0.97)", backdropFilter:"blur(20px)", borderTop:"1px solid rgba(0,0,0,0.1)", padding:"10px 2px 22px", display:"flex", justifyContent:"space-around", zIndex:100 }}>
    {navTabs.map((t) => (
      <button key={t.id} onClick={() => setTab(t.id)} style={{ background:"none", border:"none", cursor:"pointer", display:"flex", flexDirection:"column", alignItems:"center", gap:2, padding:"5px 6px", color: tab === t.id ? CLGRN : "#444" }}>
        <span style={{ fontSize:16 }}>{t.icon}</span>
        <span style={{ fontSize:8, fontWeight: tab === t.id ? 700 : 500, letterSpacing:0.3, textTransform:"uppercase" }}>{t.label}</span>
        {tab === t.id && <div style={{ width:4, height:4, borderRadius:"50%", background:CLGRN, boxShadow:"0 0 7px " + CLGRN }} />}
      </button>
    ))}
  </div>
</div>
```

);
}

// ── ROOT ─────────────────────────────────────────────────────────────────────
function App() {
const [auth, setAuth]               = useState(null);
const [elevi, setElevi]             = useState([]);
const [instructori, setInstructori] = useState([]);
const [masini, setMasini]           = useState([]);
const [sesiuni, setSesiuni]         = useState([]);
const [programari, setProgramari]   = useState([]);
const [materiale, setMateriale]     = useState([]);
const [citite, setCitite]           = useState({});
const [loading, setLoading]         = useState(true);
const [dbError, setDbError]         = useState(null);

// ── Încarcă toate datele din Supabase la start ─────────────────────────
useEffect(() => {
async function loadAll() {
try {
const [el, ins, mas, ses, prog, mat, cit] = await Promise.all([
sbGet(“elevi”, “?select=*&order=nume”),
sbGet(“instructori”, “?select=*&order=nume”),
sbGet(“masini”, “?select=*&order=id”),
sbGet(“sesiuni”, “?select=*&order=created_at.desc”),
sbGet(“programari”, “?select=*&order=data,ora”),
sbGet(“materiale”, “?select=*&order=created_at.desc”),
sbGet(“materiale_elevi”, “?select=*”),
]);

```
    // Normalizează elevi
    setElevi(el.map(e => ({
      id: e.id, nume: e.nume, parola: e.parola, telefon: e.telefon,
      masina: e.masina, ore: parseFloat(e.ore) || 0, oreMax: e.ore_max || 20,
      teorie: e.teorie || 0, status: e.status, plata: e.plata,
      data: e.data_inscriere,
    })));

    // Normalizează instructori
    setInstructori(ins.map(i => ({
      id: i.id, nume: i.nume, parola: i.parola, telefon: i.telefon,
      autorizatie: i.autorizatie, valabilitate: i.valabilitate,
      masina: i.masina, foto: i.foto_url || null,
    })));

    // Normalizează masini
    setMasini(mas.map(m => ({
      id: m.id, model: m.model, an: m.an, culoare: m.culoare,
      airtag: m.airtag, km: m.km, status: m.status, foto: m.foto_url || null,
    })));

    // Normalizează sesiuni
    setSesiuni(ses.map(s => ({
      id: s.id, elev_id: s.elev_id, instructor_id: s.instructor_id,
      masina: s.masina_id, data: s.data,
      durata: s.durata_min ? `${Math.floor(s.durata_min/60)}h ${String(s.durata_min%60).padStart(2,"0")}m` : "—",
      durataMin: s.durata_min, km: s.km,
    })));

    // Normalizează programari
    setProgramari(prog.map(p => ({
      id: p.id, tip: p.tip, elev_id: p.elev_id, instructor_id: p.instructor_id,
      masina: p.masina_id || "-", data: p.data, ora: p.ora.slice(0,5),
      durata: p.durata_min, status: p.status,
    })));

    // Normalizează materiale + alocări
    const cititeMap = {};
    cit.forEach(c => {
      if (c.citit) cititeMap[`${c.elev_id}::${c.material_id}`] = c.data_citit || true;
    });

    setMateriale(mat.map(m => ({
      id: m.id, tip: m.tip, titlu: m.titlu, descriere: m.descriere,
      fisier: m.fisier_nume || "—", marime: m.marime || "—",
      fileUrl: m.fisier_url || null, fileType: m.fisier_url ? (m.fisier_url.endsWith(".mp4") ? "video/mp4" : "application/pdf") : "",
      data: m.data_adaugare || "—",
      alocatLa: cit.filter(c => c.material_id === m.id).map(c => c.elev_id),
    })));

    setCitite(cititeMap);
    setDbError(null);
  } catch(err) {
    console.error("Eroare DB:", err);
    setDbError(err.message);
    // Fallback la date locale dacă DB nu e disponibil
    setElevi(ELEVI_INIT);
    setInstructori(INSTRUCTORI_INIT);
  } finally {
    setLoading(false);
  }
}
loadAll();
```

}, []);

// ── Funcții CRUD conectate la Supabase ──────────────────────────────────

async function addElevDB(newElev) {
const parola = newElev.parola?.trim() || newElev.nume.split(” “)[0].toLowerCase();
const rows = await sbInsert(“elevi”, [{
nume: newElev.nume, parola, telefon: newElev.telefon || “”,
masina: newElev.masina || “”, plata: newElev.plata || “achitat”,
ore: 0, ore_max: 20, teorie: 0, status: “activ”,
data_inscriere: new Date().toLocaleDateString(“ro-RO”, { month:“long”, year:“numeric” }),
}]);
const e = rows[0];
setElevi(prev => […prev, { id:e.id, nume:e.nume, parola:e.parola, telefon:e.telefon, masina:e.masina, ore:0, oreMax:20, teorie:0, status:“activ”, plata:e.plata, data:e.data_inscriere }]);
}

async function updateElevOreDB(elevId, oreNoi) {
await sbUpdate(“elevi”, elevId, { ore: oreNoi });
setElevi(prev => prev.map(e => e.id === elevId ? { …e, ore: oreNoi } : e));
}

async function addSesiuneDB(ses) {
const rows = await sbInsert(“sesiuni”, [{
elev_id: ses.elev_id, instructor_id: ses.instructor_id,
masina_id: ses.masina_id, data: ses.data, durata_min: ses.durata_min, km: ses.km,
}]);
const s = rows[0];
setSesiuni(prev => [{ id:s.id, elev_id:s.elev_id, masina:s.masina_id, data:ses.data, durata:ses.durata, durataMin:ses.durata_min, km:ses.km }, …prev]);
// Actualizează orele elevului
const elev = elevi.find(e => e.id === ses.elev_id);
if (elev) {
const oreNoi = Math.min(elev.oreMax, parseFloat((elev.ore + ses.durata_min/60).toFixed(1)));
await updateElevOreDB(ses.elev_id, oreNoi);
}
}

async function addProgramareDB(prog) {
const rows = await sbInsert(“programari”, [{
tip: prog.tip, elev_id: prog.elev_id, instructor_id: prog.instructor_id,
masina_id: prog.masina || null, data: prog.data, ora: prog.ora,
durata_min: prog.durata, status: “confirmat”,
}]);
const p = rows[0];
setProgramari(prev => […prev, { id:p.id, tip:p.tip, elev_id:p.elev_id, instructor_id:p.instructor_id, masina:p.masina_id||”-”, data:p.data, ora:p.ora.slice(0,5), durata:p.durata_min, status:p.status }]);
}

async function deleteProgramareDB(id) {
await sbDelete(“programari”, id);
setProgramari(prev => prev.filter(p => p.id !== id));
}

async function addMaterialDB(mat, file) {
try {
// 1. Salvează metadatele în DB
const rows = await sbInsert(“materiale”, [{
tip: mat.tip, titlu: mat.titlu, descriere: mat.descriere || “”,
fisier_nume: file ? file.name : “”, marime: mat.marime || “”,
data_adaugare: new Date().toLocaleDateString(“ro-RO”, { day:“numeric”, month:“long”, year:“numeric” }),
}]);
const m = rows[0];

```
  let fileUrl = null;
  let fileType = file?.type || "";

  // 2. Dacă există fișier, încearcă upload în Storage
  if (file) {
    try {
      const ext = file.name.split(".").pop();
      const path = `materiale/${m.id}_${Date.now()}.${ext}`;
      fileUrl = await sbUploadFile("materiale", path, file);
      await sbUpdate("materiale", m.id, { fisier_url: fileUrl });
    } catch(uploadErr) {
      console.warn("Upload Storage eșuat (bucket poate lipsă):", uploadErr.message);
      // Fallback: stochează ca Data URL local (temporar)
      fileUrl = mat.fileUrl || null;
    }
  }

  // 3. Actualizează state-ul local
  setMateriale(prev => [{
    id: m.id, tip: m.tip, titlu: m.titlu, descriere: m.descriere,
    fisier: file?.name || "—", marime: mat.marime || "—",
    fileUrl, fileType,
    data: m.data_adaugare, alocatLa: [],
  }, ...prev]);

  return m.id;
} catch(err) {
  console.error("Eroare addMaterialDB:", err);
  throw err;
}
```

}

async function deleteMaterialDB(id) {
await sbDelete(“materiale”, id);
setMateriale(prev => prev.filter(m => m.id !== id));
}

async function alocaMaterialDB(materialId, elevIds) {
// Șterge alocările vechi
const res = await fetch(`${SUPA_URL}/rest/v1/materiale_elevi?material_id=eq.${materialId}`, {
method: “DELETE”, headers: { apikey: SUPA_KEY, Authorization: `Bearer ${SUPA_KEY}` },
});
// Adaugă noi
if (elevIds.length > 0) {
await sbInsert(“materiale_elevi”, elevIds.map(eid => ({ material_id: materialId, elev_id: eid, citit: false })));
}
setMateriale(prev => prev.map(m => m.id === materialId ? { …m, alocatLa: elevIds } : m));
}

async function marcheazaCititDB(materialId, elevId, elevNume) {
const key = `${elevId}::${materialId}`;
const dataAzi = new Date().toLocaleDateString(“ro-RO”, { day:“numeric”, month:“short” });
await sbUpsert(“materiale_elevi”, [{ material_id: materialId, elev_id: elevId, citit: true, data_citit: new Date().toISOString() }], “material_id,elev_id”);
setCitite(prev => ({ …prev, [key]: dataAzi, [`${elevNume}::${materialId}`]: dataAzi }));
}

// ── Refresh date din DB ─────────────────────────────────────────────────
async function refreshElevi() {
const el = await sbGet(“elevi”, “?select=*&order=nume”);
setElevi(el.map(e => ({ id:e.id, nume:e.nume, parola:e.parola, telefon:e.telefon, masina:e.masina, ore:parseFloat(e.ore)||0, oreMax:e.ore_max||20, teorie:e.teorie||0, status:e.status, plata:e.plata, data:e.data_inscriere })));
}

function logout() { setAuth(null); }

if (loading) return (
<div style={{ minHeight:“100vh”, background:CLBG, display:“flex”, flexDirection:“column”, alignItems:“center”, justifyContent:“center”, fontFamily:”‘DM Sans’,sans-serif” }}>
<svg width=“60” height=“60” viewBox=“0 0 200 200” style={{ marginBottom:20 }}>
<rect x="30" y="30" width="115" height="115" rx="18" fill="none" stroke={CLACCENT} strokeWidth="14" transform="rotate(45 87.5 87.5)"/>
<polyline points="52,100 80,130 130,68" fill="none" stroke="#111" strokeWidth="16" strokeLinecap="round" strokeLinejoin="round"/>
</svg>
<div style={{ fontWeight:800, fontSize:18, color:”#111” }}>SuperOk Bistrița</div>
<div style={{ fontSize:13, color:”#888”, marginTop:8 }}>Se conectează la baza de date…</div>
{dbError && <div style={{ marginTop:12, fontSize:11, color:CLACCENT, background:”#fff0f0”, border:“1px solid “+CLACCENT+“33”, borderRadius:8, padding:“8px 14px”, maxWidth:300, textAlign:“center” }}>⚠️ {dbError}<br/><span style={{color:”#888”}}>Se folosesc date locale</span></div>}
</div>
);

if (!auth) return <LoginPage onLogin={setAuth} elevi={elevi} instructori={instructori} />;

const { role, elevData, instrData } = auth;

// Îmbogățim datele cu numele elvilor/instructorilor pentru afișare
const sesiuniRich = sesiuni.map(s => ({
…s,
elev: elevi.find(e => e.id === s.elev_id)?.nume || s.elev || “—”,
instructor: instructori.find(i => i.id === s.instructor_id)?.nume || s.instructor || “—”,
}));
const programariRich = programari.map(p => ({
…p,
elev: elevi.find(e => e.id === p.elev_id)?.nume || p.elev || “—”,
instructor: instructori.find(i => i.id === p.instructor_id)?.nume || p.instructor || “—”,
}));
// Materiale cu nume elevi în loc de ID-uri
const materialeRich = materiale.map(m => ({
…m,
alocatLa: Array.isArray(m.alocatLa)
? m.alocatLa.map(eid => typeof eid === “number” ? (elevi.find(e => e.id === eid)?.nume || “”) : eid).filter(Boolean)
: m.alocatLa || [],
}));

if (role.id === “elev”)
return <ElevInterface onLogout={logout} elevi={elevi} elevData={elevData} programari={programariRich} sesiuni={sesiuniRich} materiale={materialeRich} citite={citite} setCitite={(fn) => { const next = typeof fn === “function” ? fn(citite) : fn; setCitite(next); }} marcheazaCititDB={marcheazaCititDB} />;
if (role.id === “instructor”)
return <InstructorInterface onLogout={logout} elevi={elevi} instrData={instrData} programari={programariRich} sesiuni={sesiuniRich} setSesiuni={setSesiuni} setElevi={setElevi} addSesiuneDB={addSesiuneDB} />;
if (role.id === “birou”)
return <BirouInterface onLogout={logout} elevi={elevi} setElevi={setElevi} instructori={instructori} setInstructori={setInstructori} masini={masini} setMasini={setMasini} programari={programariRich} setProgramari={setProgramari} sesiuni={sesiuniRich} materiale={materialeRich} setMateriale={setMateriale} citite={citite} addElevDB={addElevDB} addMaterialDB={addMaterialDB} deleteMaterialDB={deleteMaterialDB} alocaMaterialDB={alocaMaterialDB} addProgramareDB={addProgramareDB} deleteProgramareDB={deleteProgramareDB} refreshElevi={refreshElevi} />;
return null;
}

ReactDOM.createRoot(document.getElementById(“root”)).render(React.createElement(App));
</script>

</body>
</html>