
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AM/PM MateKen2 - Nicaragua</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }

        body {
            background: linear-gradient(135deg, #2196f3, #00bcd4);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 15px;
        }

        .contenedor {
            background: white;
            width: 100%;
            max-width: 950px;
            border-radius: 25px;
            padding: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,.2);
        }

        h1 {
            text-align: center;
            color: #1565c0;
            margin-bottom: 15px;
        }

        .avatar {
            font-size: 80px;
            text-align: center;
            animation: flotar 2s infinite;
        }

        @keyframes flotar {
            50% { transform: translateY(-10px); }
        }

        .panel {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            margin-bottom: 15px;
        }

        .caja {
            flex: 1;
            min-width: 120px;
            background: #e3f2fd;
            padding: 12px;
            border-radius: 15px;
            text-align: center;
            font-weight: bold;
        }

        .progreso {
            height: 18px;
            background: #ddd;
            border-radius: 20px;
            overflow: hidden;
            margin-bottom: 15px;
        }

        .barra {
            height: 100%;
            width: 0%;
            background: #4caf50;
            transition: .5s;
        }

        .pregunta {
            background: #f1f8ff;
            padding: 20px;
            border-radius: 15px;
            font-size: 22px;
            font-weight: bold;
            text-align: center;
            margin-bottom: 15px;
        }

        .pista {
            background: #fff3cd;
            padding: 10px;
            border-radius: 10px;
            margin-bottom: 10px;
            text-align: center;
        }

        .opcion {
            width: 100%;
            padding: 15px;
            margin-bottom: 10px;
            border: none;
            border-radius: 12px;
            background: #bbdefb;
            font-size: 18px;
            cursor: pointer;
            transition: 0.2s;
        }

        .opcion:hover {
            transform: scale(1.03);
            background: #90caf9;
        }

        .boton {
            width: 100%;
            padding: 15px;
            border: none;
            border-radius: 15px;
            background: #1565c0;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }

        .resultado {
            display: none;
            text-align: center;
        }

        .insignia {
            font-size: 70px;
            margin: 15px;
        }

        @media(max-width:768px){
            .avatar { font-size: 60px; }
            .pregunta { font-size: 18px; }
        }
    </style>
</head>
<body>

<div class="contenedor">
    <h1>🏪 AM/PM MateKen2</h1>

    <div id="inicio">
        <div class="avatar">🤖</div>
        <button class="boton" onclick="iniciarJuego()">▶ Empezar Juego</button>
    </div>

    <div id="juego" style="display:none;">
        <div class="panel">
            <div class="caja">⭐ Puntos<br><span id="puntos">0</span></div>
            <div class="caja">📋 Pregunta<br><span id="numero">1</span>/10</div>
            <div class="caja">⏱ Tiempo<br><span id="tiempo">0</span>s</div>
        </div>

        <div class="progreso"><div class="barra" id="barra"></div></div>
        <div class="avatar" id="avatar">🤖</div>
        <div class="pista" id="pista"></div>
        <div class="pregunta" id="pregunta"></div>
        <div id="opciones"></div>

        <button class="boton" onclick="reiniciar()" style="margin-top:10px; background:#666;">🔄 Reiniciar</button>
    </div>

    <div id="resultado" class="resultado">
        <h2>🎉 Excelente Trabajo 🎉</h2>
        <div class="insignia" id="insignia"></div>
        <h1 id="porcentaje"></h1>
        <p id="resumen" style="margin: 15px 0; font-size: 18px;"></p>
        <button class="boton" onclick="reiniciar()">🔄 Jugar Nuevamente</button>
    </div>
</div>
<script>
    let preguntas = [];
    let indice = 0;
    let aciertos = 0;
    let segundos = 0;
    let cronometro;
    let audioCtx = null;

    // Inicializa el contexto de audio de forma segura
    function initAudio() {
        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
    }

    // Generador de efectos de sonido sintéticos
    function reproducirSonido(tipo) {
        initAudio();
        if (!audioCtx) return;

        let osc = audioCtx.createOscillator();
        let gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        let tiempoActual = audioCtx.currentTime;

        if (tipo === 'correcto') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(523.25, tiempoActual); // Nota C5
            osc.frequency.setValueAtTime(659.25, tiempoActual + 0.1); // Nota E5
            gain.gain.setValueAtTime(0.3, tiempoActual);
            gain.gain.exponentialRampToValueAtTime(0.01, tiempoActual + 0.3);
            osc.start(tiempoActual);
            osc.stop(tiempoActual + 0.3);
        } else if (tipo === 'incorrecto') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(150, tiempoActual);
            osc.frequency.linearRampToValueAtTime(80, tiempoActual + 0.2);
            gain.gain.setValueAtTime(0.3, tiempoActual);
            gain.gain.exponentialRampToValueAtTime(0.01, tiempoActual + 0.2);
            osc.start(tiempoActual);
            osc.stop(tiempoActual + 0.2);
        } else if (tipo === 'victoria') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(523.25, tiempoActual);
            osc.frequency.setValueAtTime(659.25, tiempoActual + 0.1);
            osc.frequency.setValueAtTime(783.99, tiempoActual + 0.2);
            osc.frequency.setValueAtTime(1046.50, tiempoActual + 0.3);
            gain.gain.setValueAtTime(0.4, tiempoActual);
            gain.gain.exponentialRampToValueAtTime(0.01, tiempoActual + 0.5);
            osc.start(tiempoActual);
            osc.stop(tiempoActual + 0.5);
        }
    }

    function C(valor) {
        return "C$ " + valor.toLocaleString('es-NI', { minimumFractionDigits: 0, maximumFractionDigits: 2 });
    }

    function aleatorio(min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    function mezclar(arr) {
        return arr.sort(() => Math.random() - 0.5);
    }

    function generarPreguntas() {
        let banco = [];
        for (let i = 0; i < 100; i++) {
            let cantidad = aleatorio(2, 10);
            let precio = aleatorio(20, 150);

            banco.push({
                icono: "🌭",
                pregunta: `Compraste ${cantidad} hot dogs a ${C(precio)} cada uno. ¿Cuánto pagaste en total?`,
                respuesta: cantidad * precio,
                pista: "Multiplica la cantidad por el precio"
            });

            banco.push({
                icono: "☕",
                pregunta: `Compraste ${cantidad} cafés a ${C(precio)} cada uno. ¿Cuál es el total en córdobas?`,
                respuesta: cantidad * precio,
                pista: "Usa la multiplicación"
            });

            banco.push({
                icono: "🍕",
                pregunta: `Compraste ${cantidad} pizzas a ${C(precio)} cada una. ¿Cuánto es el total?`,
                respuesta: cantidad * precio,
                pista: "Calcula cantidad × precio"
            });

            banco.push({
                icono: "🍔",
                pregunta: `Tenías ${C(precio * 10)} y gastaste ${C(precio)} en hamburguesas. ¿Cuántos córdobas te quedan?`,
                respuesta: (precio * 10) - precio,
                pista: "Debes realizar una resta"
            });

            banco.push({
                icono: "🥤",
                pregunta: `Si cada gaseosa cuesta ${C(precio)} y compras ${cantidad}. ¿Cuánto pagas?`,
                respuesta: cantidad * precio,
                pista: "Multiplicación simple"
            });

            banco.push({
                icono: "🍩",
                pregunta: `Si repartes ${C(cantidad * 100)} en partes iguales entre ${cantidad} personas. ¿Cuánto recibe cada una?`,
                respuesta: 100,
                pista: "Divide el total entre los clientes"
            });
        }
        return mezclar(banco).slice(0, 10);
    }

    function iniciarJuego() {
        initAudio();
        document.getElementById("inicio").style.display = "none";
        document.getElementById("resultado").style.display = "none";
        document.getElementById("juego").style.display = "block";
        preguntas = generarPreguntas();
        indice = 0;
        aciertos = 0;
        segundos = 0;
        document.getElementById("puntos").innerText = aciertos;
        document.getElementById("tiempo").innerText = segundos;
        
        clearInterval(cronometro);
        cronometro = setInterval(() => {
            segundos++;
            document.getElementById("tiempo").innerText = segundos;
        }, 1000);
        mostrarPregunta();
    }

    function mostrarPregunta() {
        let p = preguntas[indice];
        document.getElementById("numero").innerText = indice + 1;
        document.getElementById("pregunta").innerHTML = `${p.icono} ${p.pregunta}`;
        document.getElementById("pista").innerHTML = `💡 ${p.pista}`;
        document.getElementById("barra").style.width = (indice * 10) + "%";
        document.getElementById("avatar").innerText = "🤖";

        let opciones = [
            p.respuesta,
            p.respuesta + aleatorio(5, 20),
            Math.max(1, p.respuesta - aleatorio(5, 15)),
            p.respuesta + aleatorio(21, 50)
        ];
        opciones = mezclar([...new Set(opciones)]);

        let html = "";
        opciones.forEach(valor => {
            html += `<button class="opcion" onclick="verificar(${valor})">${C(valor)}</button>`;
        });
        document.getElementById("opciones").innerHTML = html;
    }

    function verificar(seleccionado) {
        let p = preguntas[indice];
        if (seleccionado === p.respuesta) {
            aciertos++;
            document.getElementById("puntos").innerText = aciertos;
            document.getElementById("avatar").innerText = "😎";
            reproducirSonido('correcto');
        } else {
            document.getElementById("avatar").innerText = "😥";
            reproducirSonido('incorrecto');
        }

        let botones = document.querySelectorAll(".opcion");
        botones.forEach(b => b.disabled = true);

        setTimeout(() => {
            indice++;
            if (indice < 10) {
                mostrarPregunta();
            } else {
                finalizarJuego();
            }
        }, 1000);
    }

    function finalizarJuego() {
        clearInterval(cronometro);
        document.getElementById("juego").style.display = "none";
        document.getElementById("resultado").style.display = "block";

        let porcentaje = (aciertos / 10) * 100;
        document.getElementById("porcentaje").innerText = `${porcentaje}% de Aciertos`;
        
        let insignia = "🥉";
        if (porcentaje >= 90) insignia = "🥇";
        else if (porcentaje >= 70) insignia = "🥈";
        
        document.getElementById("insignia").innerText = insignia;
        document.getElementById("resumen").innerText = `Completaste el juego en ${segundos} segundos respondiendo correctamente ${aciertos} de 10 preguntas.`;
        
        reproducirSonido('victoria');
    }

    function reiniciar() {
        clearInterval(cronometro);
        document.getElementById("juego").style.display = "none";
        document.getElementById("resultado").style.display = "none";
        document.getElementById("inicio").style.display = "block";
    }
</script>
</body>
</html>
