<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scanner de Pacotes Avançado</title>
  <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      margin: 0;
      transition: background-color 0.3s ease; /* Transição suave para as cores */
    }
    #container {
      background-color: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
      text-align: center;
      width: 90%;
      max-width: 500px;
    }
    h2 {
      color: #333;
    }
    #reader {
      width: 100%;
      max-width: 300px; /* Mantém a largura do scanner controlável */
      margin: 20px auto;
    }
    #resultado {
      margin-top: 20px;
      font-size: 1.2em;
      font-weight: bold;
      padding: 10px;
      border-radius: 5px;
    }
    .ok {
      color: green;
      background-color: #e6ffe6;
    }
    .erro {
      color: red;
      background-color: #ffe6e6;
    }
    #errorMsg {
      color: red;
      margin-top: 10px;
    }
    #idInputContainer {
      margin-top: 20px;
      text-align: left;
    }
    #idInputContainer label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    #idInput {
      width: calc(100% - 22px);
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 4px;
      margin-bottom: 10px;
      resize: vertical; /* Permite redimensionar verticalmente */
      min-height: 80px;
    }
    #saveIds {
      background-color: #007bff;
      color: white;
      padding: 10px 15px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 1em;
    }
    #saveIds:hover {
      background-color: #0056b3;
    }
    .status-green {
      background-color: #d4edda; /* Verde claro para feedback visual na tela */
    }
    .status-red {
      background-color: #f8d7da; /* Vermelho claro para feedback visual na tela */
    }
    #foundItems {
      margin-top: 20px;
      text-align: left;
      border-top: 1px solid #eee;
      padding-top: 15px;
    }
    #foundItems h3 {
      margin-top: 0;
      color: #333;
    }
    #foundList {
      list-style: none;
      padding: 0;
      max-height: 150px;
      overflow-y: auto;
      border: 1px solid #eee;
      padding: 10px;
      border-radius: 4px;
    }
    #foundList li {
      padding: 5px 0;
      border-bottom: 1px dashed #eee;
    }
    #foundList li:last-child {
      border-bottom: none;
    }
  </style>
</head>
<body>
  <div id="container">
    <h2>Scanner de Pacotes</h2>

    <div id="idInputContainer">
      <label for="idInput">IDs dos pacotes a encontrar (um por linha):</label>
      <textarea id="idInput" placeholder="Ex:&#10;12345&#10;67890&#10;ABC-123"></textarea>
      <button id="saveIds">Salvar IDs</button>
      <p style="font-size: 0.8em; color: #666;">Os IDs serão salvos no navegador para uso futuro.</p>
    </div>

    <div id="reader"></div>
    <div id="resultado">Aguardando leitura...</div>
    <div id="errorMsg" style="color: red;"></div>

    <div id="foundItems">
        <h3>Pacotes Encontrados:</h3>
        <ul id="foundList">
            <!-- Itens encontrados serão adicionados aqui -->
        </ul>
    </div>
  </div>

  <script>
    let idsValidos = []; // Array para armazenar os IDs
    const resultadoDiv = document.getElementById("resultado");
    const errorMsgDiv = document.getElementById("errorMsg");
    const idInput = document.getElementById("idInput");
    const saveIdsButton = document.getElementById("saveIds");
    const foundList = document.getElementById("foundList");

    // Carregar IDs salvos localmente
    function loadSavedIds() {
      const savedIds = localStorage.getItem('inventoryIds');
      if (savedIds) {
        idInput.value = savedIds;
        idsValidos = savedIds.split('\n').map(id => id.trim()).filter(id => id.length > 0);
      }
    }

    // Salvar IDs inseridos pelo usuário
    saveIdsButton.addEventListener('click', () => {
      const inputIds = idInput.value;
      localStorage.setItem('inventoryIds', inputIds);
      idsValidos = inputIds.split('\n').map(id => id.trim()).filter(id => id.length > 0);
      alert('IDs salvos com sucesso!');
      console.log("IDs válidos carregados:", idsValidos);
    });

    // Função para tocar bip de sucesso
    function playSuccessBeep() {
      const context = new (window.AudioContext || window.webkitAudioContext)();
      const oscillator = context.createOscillator();
      oscillator.connect(context.destination);
      oscillator.frequency.value = 800; // Frequência mais alta para sucesso
      oscillator.type = 'sine';
      oscillator.start();
      oscillator.stop(context.currentTime + 0.1);
    }

    // Função para tocar bip de falha
    function playFailureBeep() {
      const context = new (window.AudioContext || window.webkitAudioContext)();
      const oscillator = context.createOscillator();
      oscillator.connect(context.destination);
      oscillator.frequency.value = 400; // Frequência mais baixa para falha
      oscillator.type = 'triangle';
      oscillator.start();
      oscillator.stop(context.currentTime + 0.2); // Um pouco mais longo para falha
    }

    function setScreenColor(isSuccess) {
        document.body.classList.remove('status-green', 'status-red');
        if (isSuccess) {
            document.body.classList.add('status-green');
        } else {
            document.body.classList.add('status-red');
        }
        // Volta a cor normal após um tempo
        setTimeout(() => {
            document.body.classList.remove('status-green', 'status-red');
        }, 1000); // Remove a cor após 1 segundo
    }

    function verificarCodigo(decodedText) {
      if (idsValidos.includes(decodedText)) {
        resultadoDiv.innerText = "✅ Pacote encontrado: " + decodedText;
        resultadoDiv.className = "ok";
        playSuccessBeep();
        setScreenColor(true);

        // Adicionar à lista de encontrados se ainda não estiver lá
        if (!Array.from(foundList.children).some(li => li.textContent === decodedText)) {
            const listItem = document.createElement('li');
            listItem.textContent = decodedText;
            foundList.appendChild(listItem);
            foundList.scrollTop = foundList.scrollHeight; // Rolar para o final
        }
      } else {
        resultadoDiv.innerText = "❌ Não encontrado: " + decodedText;
        resultadoDiv.className = "erro";
        playFailureBeep();
        setScreenColor(false);
      }
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
            device.label.toLowerCase().includes('environment') ||
            device.label.toLowerCase().includes('câmera traseira') // Adicionei mais termos para português
          );

          const cameraId = backCamera ? backCamera.id : devices[0].id; // Usa a traseira ou a primeira disponível

          const html5QrCode = new Html5Qrcode("reader");
          const config = {
            fps: 10,
            qrbox: { width: 250, height: 250 },
            rememberLastUsedCamera: true // Lembrar a última câmera usada
          };

          html5QrCode.start(
            cameraId,
            config,
            verificarCodigo,
            (errorMessage) => {
              // Mensagem de erro exibida para o usuário
              errorMsgDiv.innerText = "Erro na leitura: " + errorMessage;
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
    document.addEventListener('DOMContentLoaded', () => {
        loadSavedIds(); // Carrega os IDs ao iniciar
        initScanner();
    });
  </script>
</body>
</html>
