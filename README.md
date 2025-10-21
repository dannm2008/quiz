<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trivilocura</title>
    <style>
        :root {
            --color-fondo: #1e1e2f;
            --color-principal: #a084dc;
            --color-secundario: #c8b6ff;
            --color-texto: #fff;
            --color-correcto: #8fbc8f;
            --color-incorrecto: #ff6b6b;
        }

        body {
            font-family: 'Poppins', sans-serif;
            background: var(--color-fondo);
            color: var(--color-texto);
            margin: 0;
            padding: 0;
            overflow-x: hidden;
        }

        .container {
            position: relative;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            text-align: center;
        }

        .pantalla {
            display: none;
            width: 90%;
            max-width: 600px;
            background: rgba(255, 255, 255, 0.05);
            padding: 2rem;
            border-radius: 20px;
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.1);
            transition: all 0.3s ease;
        }

        .pantalla.activa {
            display: block;
            animation: aparecer 0.5s ease;
        }

        @keyframes aparecer {
            from { opacity: 0; transform: scale(0.9); }
            to { opacity: 1; transform: scale(1); }
        }

        h1 {
            color: var(--color-principal);
            margin-bottom: 1rem;
        }

        .categoria {
            background: var(--color-principal);
            color: #fff;
            border: none;
            padding: 0.8rem 1.5rem;
            border-radius: 10px;
            margin: 0.5rem;
            cursor: pointer;
            font-size: 1rem;
            transition: background 0.3s;
        }

        .categoria:hover {
            background: var(--color-secundario);
        }

        .opcion {
            background: rgba(255, 255, 255, 0.1);
            padding: 0.8rem;
            border-radius: 10px;
            margin: 0.5rem 0;
            cursor: pointer;
            transition: background 0.3s;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .opcion:hover {
            background: rgba(255, 255, 255, 0.2);
        }

        .correcta {
            background: var(--color-correcto) !important;
        }

        .incorrecta {
            background: var(--color-incorrecto) !important;
        }

        .boton {
            background: var(--color-principal);
            color: white;
            border: none;
            padding: 0.8rem 1.5rem;
            border-radius: 10px;
            cursor: pointer;
            font-size: 1rem;
            margin-top: 1rem;
            transition: 0.3s;
        }

        .boton:hover {
            background: var(--color-secundario);
        }

        .oculto {
            display: none;
        }

        .elemento-loco {
            position: absolute;
            font-size: 1.5rem;
            opacity: 0.6;
            animation: flotar 12s infinite linear;
        }

        @keyframes flotar {
            0% { transform: translateY(0) rotate(0deg); }
            100% { transform: translateY(-100vh) rotate(720deg); }
        }

        .confeti {
            position: fixed;
            top: -10px;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            animation: caerConfeti linear forwards;
        }

        @keyframes caerConfeti {
            to { transform: translateY(100vh) rotate(720deg); opacity: 0; }
        }

        #mensaje-loco {
            display: none;
            position: fixed;
            top: 20%;
            left: 50%;
            transform: translateX(-50%);
            background: var(--color-secundario);
            padding: 1rem 2rem;
            border-radius: 15px;
            font-size: 1.3rem;
            color: #1e1e2f;
            font-weight: bold;
            z-index: 10;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Mensaje loco -->
        <div id="mensaje-loco"></div>

        <!-- Pantalla de inicio -->
        <div id="pantalla-inicio" class="pantalla activa">
            <h1>🌈 TRIVILOCURA 🎉</h1>
            <p>Selecciona una categoría para comenzar:</p>
            <button class="categoria" data-categoria="historia">Historia</button>
            <button class="categoria" data-categoria="ciencia">Ciencia</button>
            <button class="categoria" data-categoria="arte">Arte</button>
            <button id="btn-ranking" class="boton">🏆 Ver Ranking</button>
        </div>

        <!-- Pantalla del juego -->
        <div id="pantalla-juego" class="pantalla">
            <div id="categoria-indicador"></div>
            <div id="numero-pregunta"></div>
            <h2 id="pregunta-texto"></h2>
            <div id="opciones-contenedor"></div>
            <div>⏰ <span id="temporizador">15</span>s</div>
            <div id="puntaje">Puntos: 0</div>
            <button id="btn-siguiente" class="boton oculto">Siguiente ➡️</button>
        </div>

        <!-- Pantalla de resultados -->
        <div id="pantalla-resultados" class="pantalla">
            <h2>🎊 Resultados 🎊</h2>
            <p>Puntaje final: <span id="puntaje-final"></span></p>
            <p id="resumen-correctas"></p>
            <div id="resumen-errores"></div>
            <button id="btn-jugar-otra-vez" class="boton">🔁 Jugar otra vez</button>
            <button id="btn-volver-inicio" class="boton">🏠 Volver al inicio</button>
        </div>

        <!-- Pantalla de ranking -->
        <div id="pantalla-ranking" class="pantalla">
            <h2>🏆 Ranking 🏆</h2>
            <div id="ranking-lista"></div>
            <button id="btn-volver-inicio-ranking" class="boton">Volver al inicio</button>
        </div>
    </div>

    <!-- Audios -->
    <audio id="click-sound" src="click.mp3"></audio>
    <audio id="correct-sound" src="correct.mp3"></audio>
    <audio id="wrong-sound" src="wrong.mp3"></audio>

    <script>
        // Variables globales
        let categoriaActual = '';
        let preguntasActuales = [];
        let preguntaActual = 0;
        let puntuacion = 0;
        let respuestasIncorrectas = [];
        let respuestasCorrectas = 0;
        let temporizador;
        let tiempoRestante = 15;

        // Preguntas de ejemplo (puedes agregar más)
        const preguntas = {
            historia: [
                { pregunta: "¿En qué año cayó el Imperio Romano de Occidente?", opciones: ["395 d.C.", "476 d.C.", "410 d.C.", "532 d.C."], correcta: 1 },
                { pregunta: "¿Quién descubrió América en 1492?", opciones: ["Cristóbal Colón", "Marco Polo", "Magallanes", "Américo Vespucio"], correcta: 0 }
            ],
            ciencia: [
                { pregunta: "¿Cuál es el planeta más grande del sistema solar?", opciones: ["Marte", "Saturno", "Júpiter", "Neptuno"], correcta: 2 },
                { pregunta: "¿Qué gas respiramos principalmente los humanos?", opciones: ["Oxígeno", "Hidrógeno", "Nitrógeno", "Dióxido de carbono"], correcta: 0 }
            ],
            arte: [
                { pregunta: "¿Quién pintó la Mona Lisa?", opciones: ["Van Gogh", "Picasso", "Da Vinci", "Miguel Ángel"], correcta: 2 },
                { pregunta: "¿En qué país nació Pablo Picasso?", opciones: ["Francia", "España", "Italia", "Portugal"], correcta: 1 }
            ]
        };

        // Elementos del DOM
        const pantallaInicio = document.getElementById("pantalla-inicio");
        const pantallaJuego = document.getElementById("pantalla-juego");
        const pantallaResultados = document.getElementById("pantalla-resultados");
        const pantallaRanking = document.getElementById("pantalla-ranking");

        const btnRanking = document.getElementById("btn-ranking");
        const btnSiguiente = document.getElementById("btn-siguiente");
        const btnJugarOtraVez = document.getElementById("btn-jugar-otra-vez");
        const btnVolverInicio = document.getElementById("btn-volver-inicio");
        const btnVolverInicioRanking = document.getElementById("btn-volver-inicio-ranking");

        const preguntaTexto = document.getElementById("pregunta-texto");
        const opcionesContenedor = document.getElementById("opciones-contenedor");
        const temporizadorElemento = document.getElementById("temporizador");
        const puntajeElemento = document.getElementById("puntaje");
        const puntajeFinal = document.getElementById("puntaje-final");
        const resumenCorrectas = document.getElementById("resumen-correctas");
        const resumenErrores = document.getElementById("resumen-errores");
        const rankingLista = document.getElementById("ranking-lista");
        const mensajeLoco = document.getElementById("mensaje-loco");
        const numeroPregunta = document.getElementById("numero-pregunta");
        const categoriaIndicador = document.getElementById("categoria-indicador");

        const clickSound = document.getElementById("click-sound");
        const correctSound = document.getElementById("correct-sound");
        const wrongSound = document.getElementById("wrong-sound");

        // Funciones
        function playSound(sound) {
            sound.currentTime = 0;
            sound.play().catch(() => {});
        }

        function crearElementosLocos() {
            const elementos = ['🌟','💫','🔥','💖','🎉','🚀','⭐','✨'];
            const container = document.querySelector('.container');
            for (let i = 0; i < 15; i++) {
                const elemento = document.createElement('div');
                elemento.className = 'elemento-loco';
                elemento.textContent = elementos[Math.floor(Math.random() * elementos.length)];
                elemento.style.left = `${Math.random() * 100}%`;
                elemento.style.top = `${Math.random() * 100}%`;
                elemento.style.animationDuration = `${10 + Math.random() * 20}s`;
                container.appendChild(elemento);
            }
        }

        function crearConfeti() {
            const colores = ['#ff85a1','#ff9e64','#ffd166','#957fef','#8fbc8f'];
            for (let i = 0; i < 50; i++) {
                const confeti = document.createElement('div');
                confeti.className = 'confeti';
                confeti.style.left = `${Math.random() * 100}%`;
                confeti.style.animationDuration = `${3 + Math.random() * 4}s`;
                confeti.style.background = colores[Math.floor(Math.random() * colores.length)];
                document.body.appendChild(confeti);
                setTimeout(() => confeti.remove(), 5000);
            }
        }

        function mostrarMensajeLoco(mensaje) {
            mensajeLoco.textContent = mensaje;
            mensajeLoco.style.display = 'block';
            setTimeout(() => mensajeLoco.style.display = 'none', 2000);
        }

        document.querySelectorAll('.categoria').forEach(cat => {
            cat.addEventListener('click', () => {
                playSound(clickSound);
                categoriaActual = cat.getAttribute('data-categoria');
                iniciarJuego();
            });
        });

        btnRanking.addEventListener('click', () => {
            playSound(clickSound);
            mostrarRanking();
        });

        btnJugarOtraVez.addEventListener('click', () => {
            playSound(clickSound);
            iniciarJuego();
        });

        btnVolverInicio.addEventListener('click', () => {
            playSound(clickSound);
            volverInicio();
        });

        btnVolverInicioRanking.addEventListener('click', () => {
            playSound(clickSound);
            volverInicio();
        });

        crearElementosLocos();

        function iniciarJuego() {
            preguntasActuales = [...preguntas[categoriaActual]];
            preguntaActual = 0;
            puntuacion = 0;
            respuestasIncorrectas = [];
            respuestasCorrectas = 0;
            puntajeElemento.textContent = `Puntos: ${puntuacion}`;
            cambiarPantalla(pantallaJuego);
            mostrarPregunta();
        }

        function mostrarPregunta() {
            clearInterval(temporizador);
            tiempoRestante = 15;
            temporizadorElemento.textContent = tiempoRestante;
            numeroPregunta.textContent = `${preguntaActual + 1}/${preguntasActuales.length}`;
            categoriaIndicador.textContent = categoriaActual.toUpperCase();
            const pregunta = preguntasActuales[preguntaActual];
            preguntaTexto.textContent = pregunta.pregunta;
            opcionesContenedor.innerHTML = '';
            const letras = ['A','B','C','D'];
            pregunta.opciones.forEach((opcion, i) => {
                const el = document.createElement('div');
                el.className = 'opcion';
                el.innerHTML = `<div class="letra-opcion">${letras[i]}</div><div>${opcion}</div>`;
                el.addEventListener('click', () => {
                    playSound(clickSound);
                    seleccionarOpcion(i);
                });
                opcionesContenedor.appendChild(el);
            });
            btnSiguiente.classList.add('oculto');
            iniciarTemporizador();
        }

        function iniciarTemporizador() {
            temporizador = setInterval(() => {
                tiempoRestante--;
                temporizadorElemento.textContent = tiempoRestante;
                if (tiempoRestante <= 0) {
                    clearInterval(temporizador);
                    tiempoAgotado();
                }
            }, 1000);
        }

        function seleccionarOpcion(i) {
            clearInterval(temporizador);
            const pregunta = preguntasActuales[preguntaActual];
            const opciones = document.querySelectorAll('.opcion');
            opciones[pregunta.correcta].classList.add('correcta');
            if (i !== pregunta.correcta) {
                opciones[i].classList.add('incorrecta');
                playSound(wrongSound);
                respuestasIncorrectas.push({pregunta: pregunta.pregunta,respuestaUsuario: pregunta.opciones[i],respuestaCorrecta: pregunta.opciones[pregunta.correcta]});
            } else {
                playSound(correctSound);
                puntuacion += 10;
                respuestasCorrectas++;
                puntajeElemento.textContent = `Puntos: ${puntuacion}`;
                mostrarMensajeLoco("¡Correcto! 🎉");
            }
            opciones.forEach(op => op.style.pointerEvents = 'none');
            btnSiguiente.classList.remove('oculto');
        }

        function tiempoAgotado() {
            const opciones = document.querySelectorAll('.opcion');
            const pregunta = preguntasActuales[preguntaActual];
            opciones[pregunta.correcta].classList.add('correcta');
            respuestasIncorrectas.push({pregunta: pregunta.pregunta,respuestaUsuario: "Tiempo agotado",respuestaCorrecta: pregunta.opciones[pregunta.correcta]});
            opciones.forEach(op => op.style.pointerEvents = 'none');
            btnSiguiente.classList.remove('oculto');
            mostrarMensajeLoco("¡Tiempo agotado! ⏰");
        }

        // ✅ Corregido: el botón Siguiente ahora sí funciona en la última pregunta
        btnSiguiente.addEventListener("click", () => {
            preguntaActual++;
            if (preguntaActual < preguntasActuales.length) {
                mostrarPregunta();
            } else {
                terminarJuego();
            }
        });

        function terminarJuego() {
            actualizarRanking(puntuacion, categoriaActual);
            puntajeFinal.textContent = puntuacion;
            resumenCorrectas.textContent = `${respuestasCorrectas}/${preguntasActuales.length} preguntas correctas`;
            mostrarErrores();
            if (puntuacion >= 70) {
                crearConfeti();
                mostrarMensajeLoco("¡Excelente trabajo! 🏆");
            }
            cambiarPantalla(pantallaResultados);
        }

        function mostrarErrores() {
            resumenErrores.innerHTML = '<h3>Preguntas incorrectas:</h3>';
            if (respuestasIncorrectas.length === 0) {
                resumenErrores.innerHTML += '<p>¡Perfecto! No tuviste errores.</p>';
                return;
            }
            respuestasIncorrectas.forEach(err => {
                const e = document.createElement('div');
                e.className = 'error-item';
                e.innerHTML = `<div class="error-pregunta">${err.pregunta}</div>
                <div class="error-respuesta">Tu respuesta: ${err.respuestaUsuario}</div>
                <div class="correcta-respuesta">Respuesta correcta: ${err.respuestaCorrecta}</div>`;
                resumenErrores.appendChild(e);
            });
        }

        function cambiarPantalla(p) {
            document.querySelectorAll('.pantalla').forEach(x => x.classList.remove('activa'));
            p.classList.add('activa');
        }

        function volverInicio() {
            cambiarPantalla(pantallaInicio);
        }

        function mostrarRanking() {
            rankingLista.innerHTML = '';
            const ranking = JSON.parse(localStorage.getItem('rankingTrivilocura')) || [];
            if (ranking.length === 0) {
                rankingLista.innerHTML = '<p>No hay puntajes aún.</p>';
                cambiarPantalla(pantallaRanking);
                return;
            }
            ranking.sort((a,b) => b.puntaje - a.puntaje);
            ranking.forEach((r, i) => {
                const item = document.createElement('div');
                item.textContent = `${i+1}. ${r.categoria} — ${r.puntaje} pts`;
                rankingLista.appendChild(item);
            });
            cambiarPantalla(pantallaRanking);
        }

        function actualizarRanking(puntaje, categoria) {
            const ranking = JSON.parse(localStorage.getItem('rankingTrivilocura')) || [];
            ranking.push({puntaje, categoria});
            localStorage.setItem('rankingTrivilocura', JSON.stringify(ranking));
        }
    </script>
</body>
</html>
