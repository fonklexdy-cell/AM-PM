
<html lang="es">
<head>

<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Pulpería Don Pepe MateKen PRO</title>

<style>

*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

body{
background:linear-gradient(135deg,#ff9800,#ffc107);
min-height:100vh;
display:flex;
justify-content:center;
align-items:center;
padding:15px;
}

.contenedor{
background:white;
width:100%;
max-width:950px;
border-radius:25px;
padding:20px;
box-shadow:0 10px 30px rgba(0,0,0,.2);
}

h1{
text-align:center;
color:#e65100;
margin-bottom:15px;
}

.avatar{
font-size:80px;
text-align:center;
animation:flotar 2s infinite;
}

@keyframes flotar{
50%{
transform:translateY(-10px);
}
}

.panel{
display:flex;
gap:10px;
flex-wrap:wrap;
margin-bottom:15px;
}

.caja{
flex:1;
min-width:120px;
background:#fff3e0;
padding:12px;
border-radius:15px;
text-align:center;
font-weight:bold;
}

.progreso{
height:18px;
background:#ddd;
border-radius:20px;
overflow:hidden;
margin-bottom:15px;
}

.barra{
height:100%;
width:0%;
background:#4caf50;
transition:.5s;
}

.pregunta{
background:#fff8e1;
padding:20px;
border-radius:15px;
font-size:22px;
font-weight:bold;
text-align:center;
margin-bottom:15px;
}

.pista{
background:#e3f2fd;
padding:10px;
border-radius:10px;
text-align:center;
margin-bottom:15px;
}

.opcion{
width:100%;
padding:15px;
border:none;
border-radius:12px;
margin-bottom:10px;
font-size:18px;
cursor:pointer;
background:#ffe0b2;
transition:.3s;
}

.opcion:hover{
transform:scale(1.03);
}

.boton{
width:100%;
padding:15px;
border:none;
border-radius:15px;
background:#e65100;
color:white;
font-size:18px;
cursor:pointer;
margin-top:10px;
}

.resultado{
display:none;
text-align:center;
}

.insignia{
font-size:70px;
margin:15px;
}

@media(max-width:768px){

.avatar{
font-size:60px;
}

.pregunta{
font-size:18px;
}

.opcion{
font-size:16px;
}

}

</style>

</head>

<body>

<div class="contenedor">

<h1>🏪 Pulpería Don Pepe MateKen PRO</h1>

<div id="inicio">

<div class="avatar">
🤖
</div>

<button
class="boton"
onclick="iniciarAudio(); iniciarJuego();">
▶ Empezar Juego
</button>

</div>

<div id="juego" style="display:none;">

<div class="panel">

<div class="caja">
⭐ Puntos
<br>
<span id="puntos">0</span>
</div>

<div class="caja">
📋 Pregunta
<br>
<span id="numero">1</span>/10
</div>

<div class="caja">
⏱ Tiempo
<br>
<span id="tiempo">0</span>s
</div>

</div>

<div class="progreso">
<div class="barra" id="barra"></div>
</div>

<div class="avatar" id="avatar">
🤖
</div>

<div class="pista" id="pista"></div>

<div class="pregunta" id="pregunta"></div>

<div id="opciones"></div>

<button
class="boton"
onclick="reiniciar()">
🔄 Reiniciar
</button>

</div>

<div
id="resultado"
class="resultado">

<h2>🎉 Felicidades 🎉</h2>

<div
class="insignia"
id="insignia">
</div>

<h1 id="porcentaje"></h1>

<p id="resumen"></p>

<button
class="boton"
onclick="reiniciar()">
🔄 Jugar Nuevamente
</button>

</div>

</div>

<script>

// =========================
// VARIABLES GLOBALES
// =========================

let preguntas = [];
let indice = 0;
let aciertos = 0;
let segundos = 0;
let cronometro;

// =========================
// AUDIO
// =========================

let audioCtx;

function iniciarAudio(){

if(!audioCtx){

audioCtx =
new (
window.AudioContext ||
window.webkitAudioContext
)();

}

if(audioCtx.state === "suspended"){
audioCtx.resume();
}

}
// =========================
// SONIDO CORRECTO
// =========================

function sonidoCorrecto(){

iniciarAudio();

const osc =
audioCtx.createOscillator();

const gain =
audioCtx.createGain();

osc.type = "sine";

osc.frequency.setValueAtTime(
800,
audioCtx.currentTime
);

osc.connect(gain);
gain.connect(audioCtx.destination);

gain.gain.setValueAtTime(
0.2,
audioCtx.currentTime
);

gain.gain.exponentialRampToValueAtTime(
0.001,
audioCtx.currentTime + 0.3
);

osc.start();

osc.stop(
audioCtx.currentTime + 0.3
);

}

// =========================
// SONIDO ERROR
// =========================

function sonidoError(){

iniciarAudio();

const osc =
audioCtx.createOscillator();

const gain =
audioCtx.createGain();

osc.type = "square";

osc.frequency.setValueAtTime(
250,
audioCtx.currentTime
);

osc.connect(gain);
gain.connect(audioCtx.destination);

gain.gain.setValueAtTime(
0.2,
audioCtx.currentTime
);

gain.gain.exponentialRampToValueAtTime(
0.001,
audioCtx.currentTime + 0.4
);

osc.start();

osc.stop(
audioCtx.currentTime + 0.4
);

}

// =========================
// FORMATO CÓRDOBAS
// =========================

function cordoba(valor){

return "C$" + valor;

}

// =========================
// NÚMEROS ALEATORIOS
// =========================

function aleatorio(min,max){

return Math.floor(
Math.random() *
(max-min+1)
)+min;

}

// =========================
// MEZCLAR OPCIONES
// =========================

function mezclar(array){

return array.sort(
()=>Math.random()-0.5
);

}

// =========================
// GENERAR BANCO
// =========================

function generarPreguntas(){

let banco = [];

for(let i=0;i<120;i++){

let cantidad =
aleatorio(2,10);

let precio =
aleatorio(10,80);

banco.push({

icono:"🥤",

pregunta:
`Compraste ${cantidad}
refrescos a
${cordoba(precio)}
cada uno.
¿Cuánto pagaste?`,

respuesta:
cantidad * precio,

pista:
"Multiplica cantidad × precio"

});

banco.push({

icono:"🍪",

pregunta:
`Compraste ${cantidad}
paquetes de galletas
a ${cordoba(precio)}
cada uno.
¿Cuál es el total?`,

respuesta:
cantidad * precio,

pista:
"Multiplicación"

});

banco.push({

icono:"🍫",

pregunta:
`Tenías
${cordoba(precio*10)}
y gastaste
${cordoba(precio)}.
¿Cuánto te queda?`,

respuesta:
(precio*10)-precio,

pista:
"Resta"

});

banco.push({

icono:"🧃",

pregunta:
`Hay
${cantidad*2}
jugos y llegan
${precio}
más.
¿Cuántos hay ahora?`,

respuesta:
(cantidad*2)+precio,

pista:
"Suma"

});

banco.push({

icono:"🥚",

pregunta:
`Se reparten
${cordoba(cantidad*100)}
entre
${cantidad}
personas.
¿Cuánto recibe
cada una?`,

respuesta:
100,

pista:
"División"

});

}

return mezclar(banco)
.slice(0,10);

}

// =========================
// INICIAR JUEGO
// =========================

function iniciarJuego(){

document.getElementById(
"inicio"
).style.display="none";

document.getElementById(
"juego"
).style.display="block";

preguntas =
generarPreguntas();

indice = 0;
aciertos = 0;
segundos = 0;

cronometro =
setInterval(()=>{

segundos++;

document.getElementById(
"tiempo"
).innerText =
segundos;

},1000);

mostrarPregunta();

}
// =========================
// MOSTRAR PREGUNTA
// =========================

function mostrarPregunta(){

let p =
preguntas[indice];

document.getElementById(
"numero"
).innerText =
indice + 1;

document.getElementById(
"pregunta"
).innerHTML =
`${p.icono}
${p.pregunta}`;

document.getElementById(
"pista
).innerHTML =
"💡 " + p.pista;

document.getElementById(
"barra"
).style.width =
(indice * 10) + "%";

let opciones = [

p.respuesta,

p.respuesta +
aleatorio(5,25),

Math.max(
1,
p.respuesta -
aleatorio(3,15)
),

p.respuesta +
aleatorio(26,50)

];

opciones =
mezclar(opciones);

let html = "";

opciones.forEach(valor=>{

html += `

<button
class="opcion"
onclick="verificar(${valor})">

${cordoba(valor)}

</button>

`;

});

document.getElementById(
"opciones"
).innerHTML = html;

}

// =========================
// VERIFICAR RESPUESTA
// =========================

function verificar(valor){

let correcta =
preguntas[indice]
.respuesta;

if(valor === correcta){

aciertos++;

sonidoCorrecto();

document.getElementById(
"avatar"
).innerText =
"🤖🎉";

document.getElementById(
"puntos"
).innerText =
aciertos * 10;

}else{

sonidoError();

document.getElementById(
"avatar"
).innerText =
"🤖😕";

}

setTimeout(()=>{

document.getElementById(
"avatar"
).innerText =
"🤖";

},500);

indice++;

if(indice >= 10){

finalizar();

}else{

mostrarPregunta();

}

}
// =========================
// FINALIZAR JUEGO
// =========================

function finalizar(){

clearInterval(
cronometro
);

document.getElementById(
"juego"
).style.display =
"none";

document.getElementById(
"resultado"
).style.display =
"block";

let porcentaje =
aciertos * 10;

document.getElementById(
"porcentaje"
).innerText =
porcentaje + "%";

// =========================
// INSIGNIAS
// =========================

let insignia =
"🥉 Aprendiz";

if(porcentaje >= 40){

insignia =
"🥇 Matemático";

}

if(porcentaje >= 60){

insignia =
"🏆 Experto";

}

if(porcentaje >= 80){

insignia =
"👑 Maestro MateKen";

}

document.getElementById(
"insignia"
).innerText =
insignia;

document.getElementById(
"resumen"
).innerHTML =

`
✔ Correctas:
${aciertos}
<br>

❌ Incorrectas:
${10-aciertos}
<br>

⏱ Tiempo:
${segundos}
segundos
`;

}

// =========================
// REINICIAR
// =========================

function reiniciar(){

location.reload();

}

</script>

</body>
</html>
