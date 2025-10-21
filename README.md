<script>
    let preguntaActual = 0;
    let puntuacion = 0;
    let respuestasIncorrectas = [];
    let respuestasCorrectas = 0;
    let temporizador;
    let tiempoRestante = 15;
    let preguntasActuales = [];
    let categoriaActual = '';
    const clickSound = new Audio('click.mp3');
    const correctSound = new Audio('correct.mp3');
    const wrongSound = new Audio('wrong.mp3');

    const pantallaInicio = document.getElementById('pantalla-inicio');
    const pantallaJuego = document.getElementById('pantalla-juego');
    const pantallaResultados = document.getElementById('pantalla-resultados');
    const pantallaRanking = document.getElementById('pantalla-ranking');
    const opcionesContenedor = document.getElementById('opciones-contenedor');
    const btnSiguiente = document.getElementById('btn-siguiente');
    const puntajeElemento = document.getElementById('puntaje');
    const progresoBarra = document.getElementById('progreso-barra');
    const numeroPregunta = document.getElementById('numero-pregunta');
    const categoriaIndicador = document.getElementById('categoria-indicador');
    const preguntaTexto = document.getElementById('pregunta-texto');
    const temporizadorElemento = document.getElementById('temporizador');
    const mensajeLoco = document.getElementById('mensaje-loco');
    const puntajeFinal = document.getElementById('puntaje-final');
    const resumenCorrectas = document.getElementById('resumen-correctas');
    const resumenErrores = document.getElementById('resumen-errores');
    const rankingLista = document.getElementById('ranking-lista');

    function mostrarPregunta() {
        tiempoRestante = 15;
        temporizadorElemento.textContent = tiempoRestante;
        progresoBarra.style.width = `${(preguntaActual / 10) * 100}%`;

        numeroPregunta.textContent = `${preguntaActual + 1}/10`;
        categoriaIndicador.textContent = categoriaActual.toUpperCase();

        const pregunta = preguntasActuales[preguntaActual];
        preguntaTexto.textContent = pregunta.pregunta;

        opcionesContenedor.innerHTML = '';

        const letras = ['A', 'B', 'C', 'D'];
        pregunta.opciones.forEach((opcion, index) => {
            const elementoOpcion = document.createElement('div');
            elementoOpcion.className = 'opcion';
            elementoOpcion.innerHTML = `
                <div class="letra-opcion">${letras[index]}</div>
                <div>${opcion}</div>
            `;

            elementoOpcion.addEventListener('click', () => {
                playSound(clickSound);
                seleccionarOpcion(index);
            });
            opcionesContenedor.appendChild(elementoOpcion);
        });

        btnSiguiente.classList.add('oculto');
        iniciarTemporizador();
    }

    function iniciarTemporizador() {
        clearInterval(temporizador);
        temporizador = setInterval(() => {
            tiempoRestante--;
            temporizadorElemento.textContent = tiempoRestante;

            if (tiempoRestante <= 0) {
                clearInterval(temporizador);
                tiempoAgotado();
            }
        }, 1000);
    }

    function seleccionarOpcion(indice) {
        clearInterval(temporizador);

        const pregunta = preguntasActuales[preguntaActual];
        const opciones = document.querySelectorAll('.opcion');

        opciones[pregunta.correcta].classList.add('correcta');

        if (indice !== pregunta.correcta) {
            opciones[indice].classList.add('incorrecta');
            playSound(wrongSound);
            respuestasIncorrectas.push({
                pregunta: pregunta.pregunta,
                respuestaUsuario: pregunta.opciones[indice],
                respuestaCorrecta: pregunta.opciones[pregunta.correcta]
            });
        } else {
            playSound(correctSound);
            puntuacion += 10;
            respuestasCorrectas++;
            puntajeElemento.textContent = `Puntos: ${puntuacion}`;
            mostrarMensajeLoco("¡Correcto! 🎉");
        }

        opciones.forEach(opcion => {
            opcion.style.pointerEvents = 'none';
        });

        btnSiguiente.classList.remove('oculto');
    }

    function tiempoAgotado() {
        const opciones = document.querySelectorAll('.opcion');
        const pregunta = preguntasActuales[preguntaActual];

        opciones[pregunta.correcta].classList.add('correcta');

        respuestasIncorrectas.push({
            pregunta: pregunta.pregunta,
            respuestaUsuario: "Tiempo agotado",
            respuestaCorrecta: pregunta.opciones[pregunta.correcta]
        });

        opciones.forEach(opcion => {
            opcion.style.pointerEvents = 'none';
        });

        btnSiguiente.classList.remove('oculto');
        mostrarMensajeLoco("¡Tiempo agotado! ⏰");
    }

    function siguientePregunta() {
        preguntaActual++;

        if (preguntaActual < preguntasActuales.length) {
            mostrarPregunta();
        } else {
            terminarJuego();
        }
    }

    function terminarJuego() {
        actualizarRanking(puntuacion, categoriaActual);

        puntajeFinal.textContent = puntuacion;
        resumenCorrectas.textContent = `${respuestasCorrectas}/10 preguntas correctas`;
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

        respuestasIncorrectas.forEach(error => {
            const errorElemento = document.createElement('div');
            errorElemento.className = 'error-item';
            errorElemento.innerHTML = `
                <div class="error-pregunta">${error.pregunta}</div>
                <div class="error-respuesta">Tu respuesta: ${error.respuestaUsuario}</div>
                <div class="correcta-respuesta">Respuesta correcta: ${error.respuestaCorrecta}</div>
            `;
            resumenErrores.appendChild(errorElemento);
        });
    }

    function cambiarPantalla(pantalla) {
        document.querySelectorAll('.pantalla').forEach(p => {
            p.classList.remove('activa');
        });
        pantalla.classList.add('activa');
    }

    function volverInicio() {
        clearInterval(temporizador);
        preguntaActual = 0;
        puntuacion = 0;
        respuestasIncorrectas = [];
        respuestasCorrectas = 0;

        opcionesContenedor.innerHTML = '';
        progresoBarra.style.width = '0%';
        mensajeLoco.style.display = 'none';

        cambiarPantalla(pantallaInicio);
    }

    function mostrarRanking() {
        const ranking = obtenerRanking();

        rankingLista.innerHTML = '';

        if (ranking.length === 0) {
            rankingLista.innerHTML = '<p>No hay puntuaciones registradas todavía.</p>';
        } else {
            ranking.forEach((puntuacion, index) => {
                const elementoRanking = document.createElement('div');
                elementoRanking.className = 'ranking-item';
                elementoRanking.innerHTML = `
                    <div class="ranking-posicion">${index + 1}.</div>
                    <div>${puntuacion.categoria}</div>
                    <div>${puntuacion.puntos} puntos</div>
                `;
                rankingLista.appendChild(elementoRanking);
            });
        }

        cambiarPantalla(pantallaRanking);
    }

    function obtenerRanking() {
        const rankingGuardado = localStorage.getItem('triviaRanking');
        return rankingGuardado ? JSON.parse(rankingGuardado) : [];
    }

    function actualizarRanking(puntos, categoria) {
        const ranking = obtenerRanking();

        ranking.push({
            puntos: puntos,
            categoria: categoria,
            fecha: new Date().toLocaleDateString()
        });

        ranking.sort((a, b) => b.puntos - a.puntos);

        if (ranking.length > 10) {
            ranking.splice(10);
        }

        localStorage.setItem('triviaRanking', JSON.stringify(ranking));
    }

    // ✅ Aquí está la conexión del botón "Siguiente" corregida
    btnSiguiente.addEventListener("click", () => {
        siguientePregunta();
    });
</script>
