<!DOCTYPE html><html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>CRONOS - Oráculo Digital</title>
<link rel="manifest" href="manifest.json">
<link rel="icon" href="icon.png">
<link href="https://fonts.googleapis.com/css2?family=Cinzel&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body {
background: url('https://upload.wikimedia.org/wikipedia/commons/9/97/Parthenon_from_west.jpg') center/cover fixed;
color: #fffbe6;
font-family: 'Cinzel', serif;
display: flex;
justify-content: center;
align-items: center;
text-align: center;
min-height: 100vh;
margin: 0;
padding: 20px;
}
#chatbox {
background: rgba(0,0,0,0.85);
padding: 30px;
max-width: 700px;
width: 100%;
border: 3px solid gold;
border-radius: 15px;
box-shadow: 0 0 20px goldenrod;
display: flex;
flex-direction: column;
gap: 15px;
}
h1 { color: gold; text-shadow: 2px 2px 4px black; }
input, button {
font-family: 'Cinzel', serif;
padding: 12px;
font-size: 16px;
border-radius: 8px;
border: 2px solid goldenrod;
}
button {
background: gold;
color: black;
cursor: pointer;
transition: 0.3s;
}
button:hover { background: darkgoldenrod; }
#respuesta, #canvasBox {
background: rgba(255,255,255,0.1);
padding: 15px;
border: 1px solid gold;
border-radius: 10px;
color: #e6e6e6;
}
#canvasBox { display: none; }
</style>
</head>
<body>
<div id="chatbox">
<h1>CRONOS - Oráculo Digital</h1>
<input type="text" id="pregunta" placeholder="Pregunta al Oráculo...">
<button onclick="consultar()">Consultar</button>
<button onclick="escuchar()">🎤 Voz</button>
<div id="respuesta"></div>
<div id="canvasBox"><canvas id="grafico" width="400" height="300"></canvas></div>
</div>
<script>
if ('serviceWorker' in navigator) navigator.serviceWorker.register('service-worker.js');
const apiKey = "TU_API_KEY_OPENAI";
function consultar() {
const q = document.getElementById('pregunta').value.trim();
if (!q) return;
mostrar('Consultando al Oráculo...');
fetch("https://api.openai.com/v1/chat/completions",{
method:"POST", headers:{"Content-Type":"application/json","Authorization":"Bearer "+apiKey},
body:JSON.stringify({model:"gpt-4",messages:[{role:"system",content:"Sos CRONOS, sabio oráculo autónomo."},{role:"user",content:q}],temperature:0.7})
})
.then(r=>r.json())
.then(d=>{
const r = d.choices[0].message.content;
mostrar(r);
if(/y ?= ?[\dx\+\-\*/\^\.\s]+/i.test(q)) graficar(q);
})
.catch(()=>mostrar("Error al contactar al Oráculo"));
}
function mostrar(t) {
document.getElementById('respuesta').innerHTML = t.replace(/(https?:\/\/\S+)/g,'<a href="$1" target="_blank">$1</a>');
}
function escuchar() {
const r = new(window.SpeechRecognition||window.webkitSpeechRecognition)();
r.lang="es-ES";r.start();r.onresult=e=>{document.getElementById('pregunta').value=e.results[0][0].transcript;consultar();}
}
function graficar(exp) {
const expr = exp.match(/y ?= ?(.+)/i)[1], x=[], y=[];
for(let i=-10;i<=10;i+=0.5)x.push(i),y.push(Function("x",`return ${expr}`)(i));
const ctx = document.getElementById("grafico").getContext("2d");
document.getElementById("canvasBox").style.display="block";
new Chart(ctx,{type:"line",data:{labels:x,datasets:[{label:`y=${expr}`,data:y,borderColor:"gold",backgroundColor:"rgba(255,215,0,0.3)",tension:0.3}]}});
}
</script>
</body>
</html><!-- manifest.json -->{ "name": "CRONOS Oráculo Digital", "short_name": "CRONOS", "start_url": ".", "display": "standalone", "background_color": "#000", "theme_color": "#FFD700", "icons": [ { "src": "icon.png", "sizes": "192x192", "type": "image/png" }, { "src": "icon.png", "sizes": "512x512", "type": "image/png" } ] }

<!-- service-worker.js -->self.addEventListener('install', e => { e.waitUntil(caches.open('cronos-cache').then(cache => { return cache.addAll([ './', './index.html', './manifest.json', './icon.png', 'https://fonts.googleapis.com/css2?family=Cinzel&display=swap' ]); })); }); self.addEventListener('fetch', e => { e.respondWith( caches.match(e.request).then(res => res || fetch(e.request)) ); });
