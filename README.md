<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scanner de Pacotes</title>
  <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
  <style>
    /* Seus estilos CSS aqui */
  </style>
</head>
<body>
  <h2>Scanner de Pacotes</h2>
  <div id="reader" style="width: 300px; margin: auto;"></div>
  <div id="resultado">Aguardando leitura...</div>
  <div id="errorMsg" style="color: red;"></div>

  <script>
    const idsValidos = [ /* seus IDs aqui */ ];
    const resultadoDiv = document.getElementById("resultado");
    const errorMsgDiv = document.getElementById("errorMsg");

    function verificarCodigo(decodedText) {
      if (idsValidos.includes(decodedText)) {
        resultadoDiv.innerText = "✅ Pacote encontrado: " + decodedText;
        resultadoDiv.className = "ok";
        // Emitir bip de sucesso
        playBeepSound();
      } else {
        resultadoDiv.innerText = "❌ Não encontrado: " + decodedText;
        resultadoDiv.className = "erro";
      }
    }

    function playBeepSound() {
      // Implementação simples de um bip
      const context = new (window.AudioContext || window.webkitAudioContext)();
      const oscillator = context.createOscillator();
      oscillator.connect(context.destination);
      oscillator.frequency.value = 800;
      oscillator.start();
      oscillator.stop(context.currentTime + 0.1);
    }

    // Função para inicializar o scanner
    function initScanner() {
      // Primeiro, listar as câmeras disponíveis
      Html5Qrcode.getCameras().then(devices => {
        if (devices && devices.length) {
          // Encontrar a câmera traseira (environment)
          let backCamera = devices.find(device => 
            device.label.toLowerCase().includes('back') || 
            device.label.toLowerCase().includes('rear') ||
            device.label.toLowerCase().includes('environment'));
          
          const cameraId = backCamera ? backCamera.id : devices[0].id;
          
          const html5QrCode = new Html5Qrcode("reader");
          const config = { fps: 10, qrbox: { width: 250, height: 250 } };
          
          html5QrCode.start(
            cameraId, 
            config,
            verificarCodigo,
            (errorMessage) => {
              // Mensagem de erro exibida para o usuário
              errorMsgDiv.innerText = "Erro na câmera: " + errorMessage;
            }
          ).catch(err => {
            errorMsgDiv.innerText = "Erro ao iniciar câmera: " + err;
            console.error("Erro ao iniciar câmera: ", err);
          });
        } else {
          throw new Error("Nenhuma câmera encontrada");
        }
      }).catch(err => {
        errorMsgDiv.innerText = "Erro ao acessar câmeras: " + err;
        console.error("Erro ao acessar câmeras: ", err);
      });
    }

    // Iniciar o scanner quando a página carregar
    document.addEventListener('DOMContentLoaded', initScanner);
  </script>
</body>
</html>
