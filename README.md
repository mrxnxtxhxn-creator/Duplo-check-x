<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scanner de Pacotes</title>
  <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      text-align: center;
      background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
      margin: 0;
      padding: 20px;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    
    .container {
      background: white;
      border-radius: 15px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
      padding: 25px;
      width: 90%;
      max-width: 500px;
    }
    
    h2 {
      color: #2c3e50;
      margin-bottom: 20px;
    }
    
    #reader {
      width: 100%;
      margin: 20px auto;
      position: relative;
      overflow: hidden;
      border-radius: 10px;
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
    }
    
    #reader__dashboard_section {
      padding: 10px;
    }
    
    #resultado {
      margin-top: 20px;
      font-size: 18px;
      font-weight: bold;
      padding: 15px;
      border-radius: 8px;
      display: inline-block;
      transition: all 0.3s ease;
      width: 100%;
      box-sizing: border-box;
    }
    
    .ok {
      background: #4CAF50;
      color: white;
      animation: pulse 1s;
    }
    
    .erro {
      background: #FF5252;
      color: white;
    }
    
    .aguardando {
      background: #FF9800;
      color: white;
    }
    
    .camera-error {
      background: #757575;
      color: white;
    }
    
    .button {
      background: #2196F3;
      color: white;
      border: none;
      padding: 12px 25px;
      border-radius: 50px;
      font-size: 16px;
      cursor: pointer;
      margin: 15px 5px;
      transition: all 0.3s ease;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }
    
    .button:hover {
      background: #1976D2;
      transform: translateY(-2px);
      box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
    }
    
    .button:active {
      transform: translateY(0);
    }
    
    .button.stop {
      background: #FF5252;
    }
    
    .button.stop:hover {
      background: #D32F2F;
    }
    
    @keyframes pulse {
      0% { transform: scale(1); }
      50% { transform: scale(1.05); }
      100% { transform: scale(1); }
    }
    
    .status {
      margin: 15px 0;
      font-size: 14px;
      color: #666;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Scanner de Pacotes</h2>
    
    <div id="reader"></div>
    
    <div class="status" id="status">Preparando câmera...</div>
    
    <div id="resultado" class="aguardando">Aguardando leitura...</div>
    
    <div>
      <button id="toggleCamera" class="button">Trocar Câmera</button>
      <button id="restartScanner" class="button">Reiniciar Scanner</button>
      <button id="stopScanner" class="button stop">Parar Scanner</button>
    </div>
  </div>

  <script>
    // IDs extraídos da sua planilha/imagem
    const idsValidos = [
      "45521630151",
      "455277442396",
      "45527911927",
      "45528191622",
      "45528811304",
      "45528961246",
      "45529519903",
      "45529519921",
      "45529909862",
      "45530728317",
      "45530892544",
      "45531849631",
      "45533653636",
      "45534947657"
    ];

    const resultadoDiv = document.getElementById("resultado");
    const statusDiv = document.getElementById("status");
    let html5QrCode;
    let cameraId = null;
    let usingBackCamera = true;

    // Função para verificar se o código é válido
    function verificarCodigo(decodedText) {
      if (idsValidos.includes(decodedText)) {
        resultadoDiv.innerText = "✅ Pacote encontrado: " + decodedText;
        resultadoDiv.className = "ok";
        
        // Emite um bip (beep) para indicar sucesso
        try {
          const context = new (window.AudioContext || window.webkitAudioContext)();
          const oscillator = context.createOscillator();
          oscillator.type = 'sine';
          oscillator.frequency.setValueAtTime(800, context.currentTime);
          oscillator.connect(context.destination);
          oscillator.start();
          oscillator.stop(context.currentTime + 0.2);
        } catch (e) {
          console.error("Não foi possível emitir o som:", e);
        }
      } else {
        resultadoDiv.innerText = "❌ Não encontrado: " + decodedText;
        resultadoDiv.className = "erro";
      }
    }

    // Função para iniciar o scanner
    function iniciarScanner() {
      statusDiv.textContent = "Iniciando câmera...";
      
      // Configurações do scanner
      const config = { 
        fps: 10, 
        qrbox: { width: 250, height: 250 },
        supportedScanTypes: [Html5QrcodeScanType.SCAN_TYPE_CAMERA]
      };
      
      html5QrCode = new Html5Qrcode("reader");
      
      // Tenta obter as câmeras disponíveis
      Html5Qrcode.getCameras().then(devices => {
        if (devices && devices.length) {
          // Tenta usar a câmera traseira primeiro
          let backCamera = devices.find(device => device.label.toLowerCase().includes('back'));
          
          if (backCamera) {
            cameraId = backCamera.id;
            usingBackCamera = true;
          } else {
            cameraId = devices[0].id;
            usingBackCamera = false;
          }
          
          // Inicia o scanner com a câmera selecionada
          html5QrCode.start(
            cameraId,
            config,
            (decodedText) => {
              statusDiv.textContent = "Câmera funcionando...";
              verificarCodigo(decodedText);
            },
            () => {} // Ignorar mensagem de erro contínuo
          ).then(() => {
            statusDiv.textContent = usingBackCamera ? 
              "Usando câmera traseira" : "Usando câmera frontal";
          }).catch(err => {
            statusDiv.textContent = "Erro ao iniciar câmera: " + err;
            resultadoDiv.textContent = "Falha ao acessar a câmera";
            resultadoDiv.className = "camera-error";
          });
        } else {
          statusDiv.textContent = "Nenhuma câmera encontrada";
          resultadoDiv.textContent = "Dispositivo sem câmera ou permissão negada";
          resultadoDiv.className = "camera-error";
        }
      }).catch(err => {
        statusDiv.textContent = "Não foi possível listar as câmeras";
        resultadoDiv.textContent = "Erro ao acessar a câmera: " + err;
        resultadoDiv.className = "camera-error";
      });
    }

    // Função para parar o scanner
    function pararScanner() {
      if (html5QrCode && html5QrCode.isScanning) {
        html5QrCode.stop().then(() => {
          statusDiv.textContent = "Scanner parado";
          resultadoDiv.textContent = "Scanner parado - clique em Reiniciar";
          resultadoDiv.className = "aguardando";
        }).catch(err => {
          console.error("Erro ao parar o scanner:", err);
        });
      }
    }

    // Função para trocar a câmera
    function trocarCamera() {
      pararScanner();
      
      // Aguarda um pouco antes de reiniciar com a outra câmera
      setTimeout(() => {
        usingBackCamera = !usingBackCamera;
        iniciarScanner();
      }, 500);
    }

    // Adiciona event listeners aos botões
    document.getElementById('toggleCamera').addEventListener('click', trocarCamera);
    document.getElementById('restartScanner').addEventListener('click', iniciarScanner);
    document.getElementById('stopScanner').addEventListener('click', pararScanner);

    // Inicia o scanner quando a página carrega
    document.addEventListener('DOMContentLoaded', iniciarScanner);
  </script>
</body>
</html>
