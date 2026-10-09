
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#173b2b">
<title>WMS Aserradero</title>
<link rel="manifest" href="manifest.json">
<style>
:root{--g:#173b2b;--g2:#285940;--bg:#f1f4f1;--line:#dbe3dc}
*{box-sizing:border-box}
body{margin:0;font:15px Arial,sans-serif;background:var(--bg);color:#202b23}
header{background:var(--g);color:white;padding:20px}
header h1{margin:0 0 5px}
nav{display:flex;flex-wrap:wrap;gap:7px;padding:12px;background:#e3eae3}
button{padding:10px 13px;border:1px solid var(--line);border-radius:7px;background:white;cursor:pointer;font-weight:bold}
nav button.active,.primary{background:var(--g);color:white}
main{max-width:1200px;margin:auto;padding:18px}
.page{display:none}.page.active{display:block}
.cards{display:grid;grid-template-columns:repeat(auto-fit,minmax(170px,1fr));gap:12px;margin:15px 0}
.card{background:white;border:1px solid var(--line);border-radius:10px;padding:16px;margin-bottom:14px}
.metric{font-size:27px;font-weight:bold;margin-top:8px}
label{display:block;font-weight:bold;font-size:13px;margin:10px 0 5px}
input,select,textarea{width:100%;padding:10px;border:1px solid #c8d3ca;border-radius:6px;font:inherit;background:white}
.formgrid{display:grid;grid-template-columns:repeat(auto-fit,minmax(190px,1fr));gap:10px}
table{width:100%;border-collapse:collapse;min-width:720px;background:white}
th,td{text-align:left;padding:10px;border-bottom:1px solid var(--line)}
th{background:#eaf0ea}
.tablewrap{overflow:auto;border:1px solid var(--line);border-radius:8px}
.notice{padding:12px;background:#fff5d9;border-left:4px solid #d9a441;margin:12px 0;font-size:13px}
.small{font-size:12px;color:#59665d}
canvas{width:100%;height:240px}
.actions{display:flex;gap:8px;flex-wrap:wrap;margin-top:15px}
@media(max-width:550px){main{padding:10px}header{padding:15px}}
</style>
</head>
<body>
<header>
  <h1>WMS · Aserradero</h1>
  <div>Recepción · Patio · Inventario · Trazabilidad</div>
</header>
<nav>
  <button class="active" data-page="inicio">Panel</button>
  <button data-page="recepcion">Recepción</button>
  <button data-page="inventario">Inventario</button>
  <button data-page="movimientos">Movimientos</button>
  <button data-page="reportes">Reportes</button>
  <button data-page="ajustes">Parámetros</button>
</nav>
<main>
<section class="page active" id="inicio">
  <h2>Panel de control</h2>
  <p>Resumen operativo del patio de materias primas.</p>
  <div class="cards" id="indicadores"></div>
  <div class="card">
    <h3>Volumen recibido por clase</h3>
    <canvas id="grafico" height="240"></canvas>
  </div>
  <div class="card">
    <h3>Alertas de stock</h3>
    <div id="alertas"></div>
  </div>
  <button class="primary" onclick="ir('recepcion')">+ Nueva recepción</button>
</section>

<section class="page" id="recepcion">
  <h2>Recepción de materia prima</h2>
  <div class="card">
    <form id="formRecepcion">
      <div class="formgrid">
        <div><label>Fecha</label><input id="fecha" type="date" required></div>
        <div><label>Proveedor / procedencia</label><input id="proveedor" required></div>
        <div><label>Guía / documento</label><input id="guia"></div>
        <div><label>Diámetro (cm)</label><input id="diametro" type="number" min="1" step="0.1" required></div>
        <div><label>Largo (m)</label><input id="largo" type="number" min="0.1" step="0.01" value="3.2" required></div>
        <div><label>Clase de producto</label>
          <select id="clase">
            <option>Trozo aserrable</option>
            <option>Pulpable</option>
            <option>Biomasa limpia</option>
            <option>Compost / contaminado</option>
            <option>Otro</option>
          </select>
        </div>
        <div><label>Cantidad de trozos</label><input id="cantidad" type="number" min="1" step="1" required></div>
        <div><label>Sector</label><input id="sector" required></div>
        <div><label>Cancha</label><input id="cancha" required></div>
        <div><label>Pila</label><input id="pila" required></div>
        <div><label>Buzón / rango de diámetro</label><input id="buzon"></div>
        <div><label>Responsable</label><input id="responsable"></div>
        <div><label>Volumen JAS validado (m³)</label><input id="volumen" type="number" min="0" step="0.001" placeholder="Dejar vacío si no está validado"></div>
        <div><label>Estado JAS</label>
          <select id="estadoJas">
            <option>Pendiente de validación</option>
            <option>Validado manualmente</option>
          </select>
        </div>
        <div style="grid-column:1/-1"><label>Observaciones de calidad</label><textarea id="observaciones"></textarea></div>
      </div>
      <div class="notice">
        La fórmula JAS automática no está activada. Registrar diámetro, largo y clase no significa que el volumen haya sido validado.
      </div>
      <div class="actions">
        <button class="primary" type="submit">Guardar recepción</button>
        <button type="reset">Limpiar</button>
      </div>
    </form>
  </div>
</section>

<section class="page" id="inventario">
  <h2>Inventario por ubicación</h2>
  <div class="card">
    <label>Buscar por pila, sector o clase</label>
    <input id="buscar" oninput="dibujarInventario()" placeholder="Escribe para filtrar">
  </div>
  <div class="tablewrap"><table>
    <thead><tr><th>Sector</th><th>Cancha</th><th>Pila</th><th>Clase</th><th>Diámetro</th><th>Trozos</th><th>Volumen m³</th><th>JAS</th></tr></thead>
    <tbody id="tablaInventario"></tbody>
  </table></div>
</section>

<section class="page" id="movimientos">
  <h2>Movimientos y trazabilidad</h2>
  <div class="card">
    <div class="formgrid">
      <div><label>Tipo de movimiento</label>
        <select id="tipoMov">
          <option>Traslado</option><option>Consumo</option>
          <option>Ajuste positivo</option><option>Ajuste negativo</option>
        </select>
      </div>
      <div><label>Fecha</label><input id="fechaMov" type="date"></div>
      <div><label>Documento</label><input id="docMov"></div>
      <div><label>Origen / pila</label><input id="origenMov"></div>
      <div><label>Destino / pila</label><input id="destinoMov"></div>
      <div><label>Clase de producto</label>
        <select id="claseMov">
          <option>Trozo aserrable</option><option>Pulpable</option>
          <option>Biomasa limpia</option><option>Compost / contaminado</option><option>Otro</option>
        </select>
      </div>
      <div><label>Cantidad de trozos</label><input id="cantidadMov" type="number" min="0" value="0"></div>
      <div><label>Volumen m³</label><input id="volMov" type="number" min="0" step="0.001" value="0"></div>
      <div><label>Responsable</label><input id="respMov"></div>
    </div>
    <div class="actions"><button class="primary" onclick="guardarMovimiento()">Registrar movimiento</button></div>
    <div class="notice">Los movimientos quedan registrados. Este prototipo no reasigna automáticamente el stock entre pilas; revisa el inventario antes de operar.</div>
  </div>
  <div class="tablewrap"><table>
    <thead><tr><th>Fecha</th><th>Tipo</th><th>Documento</th><th>Origen</th><th>Destino</th><th>Clase</th><th>Trozos</th><th>m³</th><th>Responsable</th></tr></thead>
    <tbody id="tablaMovimientos"></tbody>
  </table></div>
</section>

<section class="page" id="reportes">
  <h2>Reportes</h2>
  <div class="cards" id="resumenReportes"></div>
  <div class="actions">
    <button onclick="exportarCSV()">Exportar recepciones CSV</button>
    <button onclick="exportarJSON()">Descargar respaldo JSON</button>
    <button onclick="window.print()">Imprimir / Guardar PDF</button>
  </div>
  <p class="small">El CSV se puede abrir con Excel. El respaldo JSON permite recuperar los datos de este navegador.</p>
</section>

<section class="page" id="ajustes">
  <h2>Parámetros operacionales</h2>
  <div class="card">
    <div class="formgrid">
      <div><label>Stock mínimo (m³)</label><input id="minimo" type="number" value="2500" min="0"></div>
      <div><label>Stock de seguridad (m³)</label><input id="seguridad" type="number" value="3500" min="0"></div>
      <div><label>Stock máximo (m³)</label><input id="maximo" type="number" value="5000" min="0"></div>
      <div><label>Consumo diario (m³)</label><input id="consumo" type="number" value="0" min="0" step="0.1"></div>
    </div>
    <div class="actions"><button class="primary" onclick="guardarParametros()">Guardar parámetros</button></div>
    <div class="notice">Los valores iniciales son referenciales y deben ser confirmados por el aserradero. La regla JAS interna sigue pendiente de documentación y aprobación.</div>
  </div>
  <div class="card">
    <h3>Importar respaldo</h3>
    <input id="archivoJSON" type="file" accept=".json,application/json">
    <div class="actions"><button onclick="importarJSON()">Restaurar datos</button></div>
  </div>
</section>
</main>
<script>
const CLAVE="wms_aserradero_v1";
const inicial={recepciones:[],movimientos:[],param:{minimo:2500,seguridad:3500,maximo:5000,consumo:0}};
let db=cargar();
function cargar(){
 try{return JSON.parse(localStorage.getItem(CLAVE))||structuredClone(inicial)}
 catch(e){return JSON.parse(JSON.stringify(inicial))}
}
function guardar(){localStorage.setItem(CLAVE,JSON.stringify(db));renderizar()}
function n(v){return Number(v||0).toLocaleString("es-CL",{maximumFractionDigits:3})}
function texto(v){return String(v??"").replace(/[&<>"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]))}
function ir(id){
 document.querySelectorAll(".page").forEach(p=>p.classList.toggle("active",p.id===id));
 document.querySelectorAll("nav button").forEach(b=>b.classList.toggle("active",b.dataset.page===id));
 renderizar();
}
document.querySelectorAll("nav button").forEach(b=>b.onclick=()=>ir(b.dataset.page));
document.getElementById("fecha").value=new Date().toISOString().slice(0,10);
document.getElementById("fechaMov").value=new Date().toISOString().slice(0,10);
function estado(vol){
 if(vol<db.param.minimo)return "Crítico";
 if(vol<db.param.seguridad)return "Seguridad";
 if(vol>db.param.maximo)return "Sobrestock";
 return "Normal";
}
document.getElementById("formRecepcion").addEventListener("submit",e=>{
 e.preventDefault();
 const r={
 fecha:fecha.value,proveedor:proveedor.value.trim(),guia:guia.value.trim(),
 diametro:+diametro.value,largo:+largo.value,clase:clase.value,cantidad:+cantidad.value,
 sector:sector.value.trim(),cancha:cancha.value.trim(),pila:pila.value.trim(),
 buzon:buzon.value.trim(),responsable:responsable.value.trim(),
 volumen:volumen.value===""?null:+volumen.value,estadoJas:estadoJas.value,
 observaciones:observaciones.value.trim()
 };
 db.recepciones.push(r);
 db.movimientos.push({fecha:r.fecha,tipo:"Recepción",documento:r.guia,
 origen:r.proveedor,destino:r.pila,clase:r.clase,cantidad:r.cantidad,
 volumen:r.volumen||0,responsable:r.responsable});
 guardar();e.target.reset();
 document.getElementById("fecha").value=new Date().toISOString().slice(0,10);
 document.getElementById("largo").value="3.2";
 alert("Recepción guardada en este navegador.");
 ir("inventario");
});
function guardarMovimiento(){
 const m={fecha:fechaMov.value,tipo:tipoMov.value,documento:docMov.value,
 origen:origenMov.value,destino:destinoMov.value,clase:claseMov.value,
 cantidad:+cantidadMov.value,volumen:+volMov.value,responsable:respMov.value};
 if(!m.fecha){alert("Ingresa la fecha.");return}
 db.movimientos.push(m);guardar();alert("Movimiento registrado.");
}
function stock(){
 const mapa={};
 db.recepciones.forEach(r=>{
  const k=[r.sector,r.cancha,r.pila,r.clase,r.diametro].join("|");
  if(!mapa[k])mapa[k]={...r,vol:0,qty:0};
  mapa[k].qty+=r.cantidad;
  mapa[k].vol+=r.volumen||0;
 });
 return Object.values(mapa);
}
function dibujarInventario(){
 const q=(document.getElementById("buscar").value||"").toLowerCase();
 const rows=stock().filter(r=>[r.sector,r.cancha,r.pila,r.clase].join(" ").toLowerCase().includes(q));
 document.getElementById("tablaInventario").innerHTML=rows.length?rows.map(r=>`<tr>
 <td>${texto(r.sector)}</td><td>${texto(r.cancha)}</td><td>${texto(r.pila)}</td>
 <td>${texto(r.clase)}</td><td>${n(r.diametro)} cm</td><td>${n(r.qty)}</td>
 <td>${n(r.vol)}</td><td>${texto(r.estadoJas)}</td></tr>`).join("")
 :'<tr><td colspan="8">Sin registros. Registra una recepción para comenzar.</td></tr>';
}
function dibujarMovimientos(){
 const rows=[...db.movimientos].reverse();
 document.getElementById("tablaMovimientos").innerHTML=rows.length?rows.map(m=>`<tr>
 <td>${texto(m.fecha)}</td><td>${texto(m.tipo)}</td><td>${texto(m.documento)}</td>
 <td>${texto(m.origen)}</td><td>${texto(m.destino)}</td><td>${texto(m.clase)}</td>
 <td>${n(m.cantidad)}</td><td>${n(m.volumen)}</td><td>${texto(m.responsable)}</td></tr>`).join("")
 :'<tr><td colspan="9">Sin movimientos.</td></tr>';
}
function dibujarGrafico(){
 const c=document.getElementById("grafico"),ctx=c.getContext("2d");
 const w=Math.max(c.clientWidth,280),h=240,dpr=devicePixelRatio||1;
 c.width=w*dpr;c.height=h*dpr;ctx.scale(dpr,dpr);ctx.clearRect(0,0,w,h);
 const totals={};db.recepciones.forEach(r=>totals[r.clase]=(totals[r.clase]||0)+(r.volumen||0));
 const entries=Object.entries(totals),max=Math.max(1,...entries.map(x=>x[1]));
 if(!entries.length){ctx.fillStyle="#59665d";ctx.fillText("Sin datos de volumen validados",15,30);return}
 const step=(w-30)/entries.length;
 entries.forEach(([name,val],i)=>{
  const bh=(val/max)*155,x=15+i*step+step*.2;
  ctx.fillStyle="#285940";ctx.fillRect(x,185-bh,step*.55,bh);
  ctx.fillStyle="#202b23";ctx.font="11px Arial";ctx.fillText(n(val),x,Math.max(12,178-bh));
  ctx.save();ctx.translate(x,200);ctx.rotate(-.3);ctx.fillText(name.slice(0,17),0,0);ctx.restore();
 });
}
function renderizar(){
 const rows=stock(),vol=rows.reduce((s,r)=>s+(r.vol||0),0);
 const trozos=rows.reduce((s,r)=>s+r.qty,0);
 const hoy=new Date().toISOString().slice(0,10);
 const recepHoy=db.recepciones.filter(r=>r.fecha===hoy).length;
 document.getElementById("indicadores").innerHTML=[
 ["Stock registrado",n(vol)+" m³"],["Trozos recibidos",n(trozos)],
 ["Recepciones",n(db.recepciones.length)],["Recepciones de hoy",n(recepHoy)]
 ].map(x=>`<div class="card"><div>${x[0]}</div><div class="metric">${x[1]}</div></div>`).join("");
 const totalPorClase={};db.recepciones.forEach(r=>totalPorClase[r.clase]=(totalPorClase[r.clase]||0)+(r.volumen||0));
 document.getElementById("alertas").innerHTML=`Stock actual: <b>${n(vol)} m³</b><br>
 Mínimo: ${n(db.param.minimo)} m³ · Seguridad: ${n(db.param.seguridad)} m³ · Máximo: ${n(db.param.maximo)} m³
 <p><b>Estado general:</b> ${estado(vol)}</p>`;
 document.getElementById("resumenReportes").innerHTML=[
 ["Recepciones",n(db.recepciones.length)],
 ["Volumen recibido",n(db.recepciones.reduce((s,r)=>s+(r.volumen||0),0))+" m³"],
 ["JAS pendiente",n(db.recepciones.filter(r=>r.estadoJas==="Pendiente de validación").length)],
 ["Cobertura estimada",db.param.consumo>0?n(vol/db.param.consumo)+" días":"Sin consumo configurado"]
 ].map(x=>`<div class="card"><div>${x[0]}</div><div class="metric">${x[1]}</div></div>`).join("");
 dibujarInventario();dibujarMovimientos();dibujarGrafico();
}
function guardarParametros(){
 const p={minimo:+minimo.value,seguridad:+seguridad.value,maximo:+maximo.value,consumo:+consumo.value};
 if(p.minimo>p.seguridad||p.seguridad>p.maximo){alert("Ordena los valores: mínimo ≤ seguridad ≤ máximo.");return}
 if(Object.values(p).some(v=>v<0)){alert("No se aceptan valores negativos.");return}
 db.param=p;guardar();alert("Parámetros guardados.");
}
function descargar(contenido,nombre,tipo){
 const a=document.createElement("a");
 a.href=URL.createObjectURL(new Blob([contenido],{type:tipo}));
 a.download=nombre;a.click();URL.revokeObjectURL(a.href);
}
function exportarJSON(){descargar(JSON.stringify(db,null,2),"respaldo-wms.json","application/json")}
function exportarCSV(){
 const cols=["fecha","proveedor","guia","diametro","largo","clase","cantidad","sector","cancha","pila","volumen","estadoJas","responsable"];
 const filas=[cols,...db.recepciones.map(r=>cols.map(k=>r[k]??""))];
 const csv=filas.map(f=>f.map(v=>'"'+String(v).replace(/"/g,'""')+'"').join(";")).join("\r\n");
 descargar("\ufeff"+csv,"recepciones-wms.csv","text/csv;charset=utf-8");
}
function importarJSON(){
 const f=document.getElementById("archivoJSON").files[0];if(!f)return alert("Selecciona un archivo JSON.");
 const reader=new FileReader();
 reader.onload=()=>{
  try{
   const x=JSON.parse(reader.result);
   if(!Array.isArray(x.recepciones)||!Array.isArray(x.movimientos)||!x.param)throw Error();
   if(confirm("Esto reemplazará los datos de este navegador. ¿Continuar?")){db=x;guardar();alert("Respaldo restaurado.");}
  }catch(e){alert("Archivo no válido para este WMS.")}
 };
 reader.readAsText(f);
}
if("serviceWorker" in navigator && location.protocol.startsWith("http")){
 navigator.serviceWorker.register("./sw.js").catch(()=>{});
}
renderizar();
</script>
</body>
</html>
