<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>AM/PM MateKen2</title>

<style>

*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

body{
background:linear-gradient(135deg,#2196f3,#00bcd4);
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
color:#1565c0;
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
background:#e3f2fd;
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
background:#f1f8ff;
padding:20px;
border-radius:15px;
font-size:22px;
font-weight:bold;
text-align:center;
margin-bottom:15px;
}

.pista{
background:#fff3cd;
padding:10px;
border-radius:10px;
margin-bottom:10px;
text-align:center;
}

.opcion{
width:100%;
padding:15px;
margin-bottom:10px;
border:none;
border-radius:12px;
background:#bbdefb;
font-size:18px;
cursor:pointer;
}

.opcion:hover{
transform:scale(1.03);
}

.boton{
width:100%;
padding:15px;
border:none;
border-radius:15px;
background:#1565c0;
color:white;
font-size:18px;
cursor:pointer;
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

}

</style>
</head>

<body>

<div class="contenedor">

<h1>🏪 AM/PM MateKen2</h1>

<div id="inicio">

<div class="avatar">🤖</div>

<button class="boton"
onclick="iniciarJuego()">
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

<div class="avatar" id="avatar">🤖</div>

<div class="pista" id="pista"></div>

<div class="pregunta" id="pregunta"></div>

<div id="opciones"></div>

<button class="boton"
onclick="reiniciar()">
🔄 Reiniciar
</button>

</div>

<div id="resultado" class="resultado">

<h2>🎉 Excelente Trabajo 🎉</h2>

<div class="insignia" id="insignia"></div>

<h1 id="porcentaje"></h1>

<p id="resumen"></p>

<button class="boton"
onclick="reiniciar()">
🔄 Jugar Nuevamente
</button>

</div>

</div>

<script>

let preguntas=[];
let indice=0;
let aciertos=0;
let segundos=0;
let cronometro;

function C(valor){
return "C$"+valor;
}

function aleatorio(min,max){
return Math.floor(Math.random()*(max-min+1))+min;
}

function mezclar(arr){
return arr.sort(()=>Math.random()-0.5);
}

function generarPreguntas(){

let banco=[];

for(let i=0;i<100;i++){

let cantidad=aleatorio(2,10);
let precio=aleatorio(20,120);

banco.push({
icono:"🌭",
pregunta:`Compraste ${cantidad} hot dogs a ${C(precio)} cada uno. ¿Cuánto pagaste?`,
respuesta:cantidad*precio,
pista:"Multiplica cantidad × precio"
});

banco.push({
icono:"☕",
pregunta:`Compraste ${cantidad} cafés a ${C(precio)} cada uno. ¿Total?`,
respuesta:cantidad*precio,
pista:"Multiplicación"
});

banco.push({
icono:"🍕",
pregunta:`Compraste ${cantidad} pizzas a ${C(precio)} cada una. ¿Total?`,
respuesta:cantidad*precio,
pista:"Multiplicación"
});

banco.push({
icono:"🍔",
pregunta:`Tenías ${C(precio*10)} y gastaste ${C(precio)} en hamburguesas. ¿Cuánto te queda?`,
respuesta:(precio*10)-precio,
pista:"Resta"
});

banco.push({
icono:"🍟",
pregunta:`Hay ${cantidad*2} papas fritas vendidas y luego ${precio} más. ¿Cuántas en total?`,
respuesta:(cantidad*2)+precio,
pista:"Suma"
});

banco.push({
icono:"🍩",
pregunta:`Se reparten ${C(cantidad*100)} entre ${cantidad} clientes. ¿Cuánto recibe cada uno?`,
respuesta:100,
pista:"División"
});

banco.push({
icono:"🥤",
pregunta:`Compraste ${cantidad} bebidas a ${C(precio)} cada una. ¿Cuánto pagaste?`,
respuesta:cantidad*precio,
pista:"Multiplicación"
});

}

return mezclar(banco).slice(0,10);

}

function iniciarJuego(){

document.getElementById("inicio").style.display="none";
document.getElementById("juego").style.display="block";

preguntas=generarPreguntas();

indice=0;
aciertos=0;
segundos=0;

cronometro=setInterval(()=>{
segundos++;
document.getElementById("tiempo").innerText=segundos;
},1000);

mostrarPregunta();

}

function mostrarPregunta(){

let p=preguntas[indice];

document.getElementById("numero").innerText=indice+1;

document.getElementById("pregunta").innerHTML=
`${p.icono} ${p.pregunta}`;

document.getElementById("pista").innerHTML=
`💡 ${p.pista}`;

document.getElementById("barra").style.width=
(indice*10)+"%";

let opciones=[
p.respuesta,
p.respuesta+aleatorio(10,40),
Math.max(1,p.respuesta-aleatorio(5,25)),
p.respuesta+aleatorio(41,80)
];

opciones=mezclar(opciones);

let html="";

opciones.forEach(valor=>{

html+=`
<button class="opcion"
onclick="verificar(${valor})">
${C(valor)}
</button>
`;

});

document.getElementById("opciones").innerHTML=html;

}

function verificar(valor){

let correcta=preguntas[indice].respuesta;

if(valor===correcta){

aciertos++;

document.getElementById("avatar").innerText="🤖🎉";

document.getElementById("puntos").innerText=
aciertos*10;

}else{

document.getElementById("avatar").innerText="🤖😕";

}

setTimeout(()=>{
document.getElementById("avatar").innerText="🤖";
},500);

indice++;

if(indice>=10){
finalizar();
}else{
mostrarPregunta();
}

}

function finalizar(){

clearInterval(cronometro);

document.getElementById("juego").style.display="none";
document.getElementById("resultado").style.display="block";

let porcentaje=aciertos*10;

document.getElementById("porcentaje").innerText=
porcentaje+"%";

let insignia="🥉 Aprendiz";

if(porcentaje>=40) insignia="🥇 Matemático";
if(porcentaje>=60) insignia="🏆 Experto";
if(porcentaje>=80) insignia="👑 Maestro MateKen";

document.getElementById("insignia").innerText=
insignia;

document.getElementById("resumen").innerHTML=
`
✔ Correctas: ${aciertos}<br>
❌ Incorrectas: ${10-aciertos}<br>
⏱ Tiempo: ${segundos} segundos
`;

}

function reiniciar(){
location.reload();
}

</script>

</body>
</html>
