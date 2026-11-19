<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contador de Cofres por Voz</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #1e1e2e;
            color: #ffffff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        .contenedor {
            background-color: #2b2b40;
            padding: 40px;
            border-radius: 15px;
            text-align: center;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
            max-width: 400px;
            width: 90%;
        }
        h1 {
            margin-top: 0;
            font-size: 24px;
            color: #f9e2af;
        }
        .numero {
            font-size: 90px;
            font-weight: bold;
            margin: 20px 0;
            color: #a6e3a1;
        }
        button {
            font-size: 16px;
            font-weight: bold;
            padding: 12px 24px;
            margin: 8px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: transform 0.1s;
        }
        button:active {
            transform: scale(0.96);
        }
        #btnPrender {
            background-color: #a6e3a1;
            color: #11111b;
        }
        #btnApagar {
            background-color: #f38ba8;
            color: #11111b;
            display: none;
        }
        #btnReiniciar {
            background-color: #89b4fa;
            color: #11111b;
        }
        .estado {
            margin-top: 15px;
            font-size: 14px;
            color: #cdd6f4;
        }
        .escuchado {
            margin-top: 10px;
            font-size: 13px;
            color: #9399b2;
            font-style: italic;
            min-height: 20px;
        }
    </style>
</head>
<body>

<div class="contenedor">
    <h1>Contador de Cofres</h1>
    <div class="numero" id="contador">0</div>
    
    <button id="btnPrender">Prender Micrófono</button>
    <button id="btnApagar">Apagar Micrófono</button>
    <button id="btnReiniciar">Reiniciar a 0</button>

    <div class="estado" id="estado">Micrófono apagado</div>
    <div class="escuchado" id="textoEscuchado"></div>
</div>

<script>
    // Configurar reconocimiento de voz
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
        alert("Tu navegador no soporta reconocimiento de voz. Por favor usa Google Chrome o Microsoft Edge.");
    }

    const recognition = new SpeechRecognition();
    recognition.lang = 'es-ES'; // Idioma español
    recognition.continuous = true; // Escucha continua
    recognition.interimResults = false; // Solo resultados confirmados para no contar doble

    let conteo = 0;
    let escuchandoActivo = false;

    const elContador = document.getElementById('contador');
    const btnPrender = document.getElementById('btnPrender');
    const btnApagar = document.getElementById('btnApagar');
    const btnReiniciar = document.getElementById('btnReiniciar');
    const elEstado = document.getElementById('estado');
    const elTextoEscuchado = document.getElementById('textoEscuchado');

    // Al recibir lo que dices
    recognition.onresult = (event) => {
        const ultimoIndice = event.results.length - 1;
        const texto = event.results[ultimoIndice][0].transcript.toLowerCase().trim();
        
        elTextoEscuchado.textContent = `Escuché: "${texto}"`;

        // Busca cuántas veces aparece la palabra "cofre" en la frase
        const coincidencias = texto.match(/\bcofre\b/g);
        if (coincidencias) {
            conteo += coincidencias.length;
            elContador.textContent = conteo;
        }
    };

    // Si el navegador pausa el micrófono solo por silencio largo, lo vuelve a encender
    recognition.onend = () => {
        if (escuchandoActivo) {
            recognition.start();
        }
    };

    recognition.onerror = (event) => {
        console.error("Error de voz:", event.error);
        if (event.error === 'not-allowed') {
            elEstado.textContent = "Permiso de micrófono denegado.";
            escuchandoActivo = false;
            btnPrender.style.display = 'inline-block';
            btnApagar.style.display = 'none';
        }
    };

    // Botón Prender
    btnPrender.addEventListener('click', () => {
        escuchandoActivo = true;
        recognition.start();
        btnPrender.style.display = 'none';
        btnApagar.style.display = 'inline-block';
        elEstado.textContent = "🟢 Escuchando... Di 'cofre'";
    });

    // Botón Apagar
    btnApagar.addEventListener('click', () => {
        escuchandoActivo = false;
        recognition.stop();
        btnPrender.style.display = 'inline-block';
        btnApagar.style.display = 'none';
        elEstado.textContent = "🔴 Micrófono apagado";
    });

    // Botón Reiniciar
    btnReiniciar.addEventListener('click', () => {
        conteo = 0;
        elContador.textContent = conteo;
        elTextoEscuchado.textContent = "";
    });
</script>

</body>
</html>
