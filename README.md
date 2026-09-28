[detector_clima_preview (2) (2).html](https://github.com/user-attachments/files/32775373/detector_clima_preview.2.2.html)[Uploadin<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Detector do Clima — Preview</title>
<style>
:root{font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,sans-serif;color:#172033;background:#f5f7fb}
*{box-sizing:border-box} body{margin:0}
.app{min-height:100vh;background:linear-gradient(180deg,#eef6ff 0,#f7f9fc 330px)}
.top{background:#fff;border-bottom:1px solid #e5e9f0;position:sticky;top:0;z-index:3}
.bar{max-width:980px;margin:auto;padding:18px 20px 14px}
.brand{font-size:21px;font-weight:800}.search{display:flex;gap:8px;margin-top:14px}
input{flex:1;border:0;background:#f0f3f8;border-radius:14px;padding:14px 16px;font-size:16px;outline:none}
button{border:0;border-radius:12px;background:#1769e0;color:white;padding:0 18px;font-size:20px;cursor:pointer}
.wrap{max-width:980px;margin:auto;padding:20px}
.location{color:#6c7585;font-size:14px;margin:3px 0 14px}
.tabs{display:flex;gap:8px;overflow:auto;padding-bottom:6px}
.tab{background:#fff;border:1px solid #e0e5ed;color:#596477;padding:10px 15px;border-radius:12px;white-space:nowrap;cursor:pointer}
.tab.active{background:#1769e0;color:white;border-color:#1769e0}
.hero{margin-top:14px;background:#fff;border-radius:24px;padding:28px;text-align:center;box-shadow:0 8px 30px rgba(20,50,90,.08)}
.temp{font-size:76px;font-weight:850;letter-spacing:-4px}
.condition{font-size:20px;color:#1769e0;font-weight:650}
.grid{display:grid;grid-template-columns:repeat(3,1fr);gap:12px;margin-top:24px}
.card{background:#f8fafc;border:1px solid #e6eaf0;border-radius:18px;padding:18px;text-align:left}
.part{font-weight:750;font-size:17px}.icon{font-size:30px}.meta{color:#687386;font-size:14px;margin-top:8px}.ct{font-size:22px;font-weight:750;margin-top:8px}
@media(max-width:700px){.grid{grid-template-columns:1fr}.temp{font-size:62px}.hero{padding:22px}.wrap{padding:14px}}
</style>
</head>
<body>
<div class="app">
  <header class="top"><div class="bar">
    <div class="brand">🌤️ Detector do Clima</div>
    <div class="search">
      <input id="city" value="Belo Horizonte" placeholder="Pesquisar outra região...">
      <button onclick="load()">→</button>
    </div>
  </div></header>
  <main class="wrap">
    <div id="location" class="location">Belo Horizonte, Minas Gerais, Brasil</div>
    <div id="tabs" class="tabs"></div>
    <section class="hero">
      <div id="temp" class="temp">--°C</div>
      <div id="condition" class="condition">Carregando previsão...</div>
      <div id="cards" class="grid"></div>
    </section>
  </main>
</div>
<script>
const rainCodes=new Set([51,53,55,56,57,61,63,65,66,67,80,81,82,95,96,99]);
function condition(c){
 const m={0:"Ensolarado",1:"Parcialmente nublado",2:"Parcialmente nublado",3:"Nublado",
 45:"Neblina",48:"Neblina",51:"Garoa",53:"Garoa",55:"Garoa",56:"Garoa",57:"Garoa",
 61:"Chuvoso",63:"Chuvoso",65:"Chuvoso",66:"Chuvoso",67:"Chuvoso",
 71:"Neve",73:"Neve",75:"Neve",77:"Neve",80:"Pancadas de chuva",81:"Pancadas de chuva",
 82:"Pancadas de chuva",85:"Pancadas de neve",86:"Pancadas de neve",95:"Tempestade",96:"Tempestade",99:"Tempestade"};
 return m[c]||"Indefinido";
}
let days=[],selected=0;
async function load(){
 const city=document.getElementById("city").value.trim();
 if(!city)return;
 document.getElementById("condition").textContent="Buscando previsão...";
 try{
  const g=await fetch("https://geocoding-api.open-meteo.com/v1/search?name="+encodeURIComponent(city)+"&count=1&language=pt&format=json").then(r=>r.json());
  if(!g.results?.length) throw new Error("Cidade não encontrada.");
  const p=g.results[0];
  document.getElementById("location").textContent=[p.name,p.admin1,p.country].filter(Boolean).join(", ");
  const w=await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${p.latitude}&longitude=${p.longitude}&hourly=temperature_2m,weather_code,precipitation_probability&timezone=auto&forecast_days=7`).then(r=>r.json());
  days=[];
  for(let d=0;d<7;d++){
    const parts=[["Manhã",9],["Tarde",15],["Noite",21]].map(([name,h])=>{
      const i=d*24+h; return {name,temp:w.hourly.temperature_2m[i],code:w.hourly.weather_code[i],rain:rainCodes.has(w.hourly.weather_code[i])}
    });
    days.push({date:w.hourly.time[d*24].slice(5,10).split("-").reverse().join("/"),parts});
  }
  selected=0;render();
 }catch(e){document.getElementById("condition").textContent=e.message}
}
function render(){
 const tabs=document.getElementById("tabs"); tabs.innerHTML="";
 days.forEach((d,i)=>{const b=document.createElement("button");b.className="tab"+(i===selected?" active":"");b.textContent=i===0?"Hoje":d.date;b.onclick=()=>{selected=i;render()};tabs.appendChild(b)});
 const d=days[selected], avg=d.parts.reduce((s,p)=>s+p.temp,0)/d.parts.length;
 document.getElementById("temp").textContent=Math.round(avg)+"°C";
 const rainy=d.parts.some(p=>p.rain), c=condition(d.parts[1].code);
 document.getElementById("condition").textContent=rainy?c+" com chance de chuva":c;
 document.getElementById("cards").innerHTML=d.parts.map(p=>`<div class="card"><div class="icon">${p.rain?"☔":p.name==="Noite"?"🌙":p.name==="Manhã"?"🌅":"☀️"}</div><div class="part">${p.name}</div><div class="meta">Céu: ${condition(p.code)} · Chuva: ${p.rain?"Sim":"Não"}</div><div class="ct">${Number(p.temp).toFixed(1)}°C</div></div>`).join("");
}
load();
</script>
</body>
</html>g detector_clima_preview (2) (2).html…]()
