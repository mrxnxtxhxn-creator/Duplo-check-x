<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner Rápido</title>
    <!-- Ícones -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
    <style>
        :root {
            --cor-fundo: #121212;
            --cor-fundo-card: #1E1E1E;
            --cor-primaria: #FFC107; /* Amarelo */
            --cor-primaria-hover: #FFA000;
            --cor-sucesso: #28a745; /* Verde para feedback */
            --cor-erro: #dc3545; /* Vermelho para feedback */
            --cor-texto: #E0E0E0;
            --cor-texto-botao: #121212;
            --cor-borda: #424242;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            margin: 0; padding: 1rem; background-color: var(--cor-fundo);
            color: var(--cor-texto); transition: background-color 0.2s ease-in-out;
        }
        body.feedback-sucesso { background-color: var(--cor-sucesso); }
        body.feedback-erro { background-color: var(--cor-erro); }
        .container { max-width: 500px; margin: 0 auto; display: flex; flex-direction: column; gap: 1.5rem; }
        .card { background-color: var(--cor-fundo-card); padding: 1.5rem; border-radius: 12px; border: 1px solid var(--cor-borda); }
        h1, h2 { text-align: center; margin-top: 0; color: var(--cor-primaria); }
        h2 { font-size: 1.2rem; margin-bottom: 1rem; border-bottom: 1px solid var(--cor-borda); padding-bottom: 0.5rem; }
        label { display: block; font-weight: 600; margin-bottom: 0.5rem; color: #BDBDBD;}
        textarea, select {
            width: 100%; padding: 0.75rem; border: 1px solid var(--cor-borda);
            border-radius: 4px; font-size: 1rem; box-sizing: border-box; background-color: var(--cor-fundo);
            color: var(--cor-texto);
        }
        button {
            display: flex; align-items: center; justify-content: center; gap: 0.5rem;
            width: 100%; padding: 0.75rem; font-size: 1.1rem;
            font-weight: 600; color: var(--cor-texto-botao); background-color: var(--cor-primaria);
            border: none; border-radius: 4px; cursor: pointer; margin-top: 1rem;
            transition: background-color 0.2s;
        }
        button:hover { background-color: var(--cor-primaria-hover); }
        button:disabled { background-color: #424242; color: #757575; cursor: not-allowed; }
        #leitor-container { width: 100%; border: 2px solid var(--cor-borda); border-radius: 4px; overflow: hidden; }
        #leitor { width: 100%; display: block; }
        #feedback-texto { text-align: center; font-weight: bold; font-size: 1.2rem; margin-top: 1rem; height: 30px;}
        #lista-container { max-height: 200px; overflow-y: auto; border: 1px solid var(--cor-borda); border-radius: 4px; padding: 0.5rem; margin-top: 1rem; }
        #lista-ids { list-style: none; padding: 0; margin: 0; }
        #lista-ids li { padding: 0.5rem; border-bottom: 1px solid var(--cor-borda); }
        #lista-ids li.encontrado { color: var(--cor-sucesso); font-weight: bold; text-decoration: line-through; }
    </style>
</head>
<body>

    <div class="container">
        <h1><i class="fa-solid fa-barcode"></i> Scanner de Pacotes</h1>

        <div class="card">
            <h2><i class="fa-solid fa-list-check"></i> 1. Lista de Pacotes</h2>
            <label for="ids-input">Cole os IDs (um por linha):</label>
            <textarea id="ids-input" placeholder="PACOTE-001&#10;PACOTE-002"></textarea>
            <button id="carregar-btn"><i class="fa-solid fa-save"></i> Carregar Lista</button>
        </div>

        <div class="card">
            <h2><i class="fa-solid fa-camera"></i> 2. Câmera</h2>
            <label for="camera-select">Selecione a Câmera:</label>
            <select id="camera-select" disabled></select>
            <div style="display: flex; gap: 1rem;">
                <button id="iniciar-btn" disabled><i class="fa-solid fa-play"></i> Iniciar</button>
                <button id="parar-btn" disabled><i class="fa-solid fa-stop"></i> Parar</button>
            </div>
            <div id="leitor-container"><div id="leitor"></div></div>
        </div>
        
        <div class="card">
            <h2><i class="fa-solid fa-chart-line"></i> 3. Progresso</h2>
            <div id="progresso-texto" style="font-size: 1.2rem; font-weight: bold; text-align: center; color: var(--cor-primaria);">Progresso: 0 de 0</div>
            <div id="lista-container">
                <ul id="lista-ids"><li>Nenhuma lista carregada.</li></ul>
            </div>
        </div>
    </div>

<!-- A biblioteca do scanner DEVE ser carregada antes do script que a utiliza -->
<script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
<script>
    // Envolve todo o código em um evento DOMContentLoaded para garantir que o HTML está pronto
    document.addEventListener('DOMContentLoaded', () => {
        // Mapeamento dos elementos do HTML
        const idsInput = document.getElementById('ids-input');
        const carregarBtn = document.getElementById('carregar-btn');
        const iniciarBtn = document.getElementById('iniciar-btn');
        const pararBtn = document.getElementById('parar-btn');
        const cameraSelect = document.getElementById('camera-select');
        const progressoTexto = document.getElementById('progresso-texto');
        const listaIdsUL = document.getElementById('lista-ids');
        const leitorDivId = 'leitor';
        
        let html5QrCode = null;
        let idsParaEncontrar = new Set();
        let idsEncontrados = new Set();
        let feedbackTimeout = null;

        // Função para carregar os IDs da área de texto
        function carregarIds() {
            const ids = idsInput.value.split('\n').map(id => id.trim()).filter(id => id);
            if (ids.length === 0) {
                alert("Por favor, insira pelo menos um ID.");
                return;
            }
            idsParaEncontrar = new Set(ids);
            idsEncontrados.clear();
            atualizarProgressoELista();
            alert(`${ids.length} IDs carregados com sucesso.`);
        }

        // Função para encontrar e listar as câmeras disponíveis
        function popularSelecaoDeCamera() {
            // Verifica se a biblioteca Html5Qrcode está disponível antes de usá-la
            if (typeof Html5Qrcode === 'undefined') {
                alert("A biblioteca da câmera não conseguiu ser carregada. Verifique a sua ligação à internet.");
                return;
            }
            Html5Qrcode.getCameras().then(devices => {
                if (devices && devices.length) {
                    cameraSelect.innerHTML = '';
                    devices.forEach(device => {
                        const option = document.createElement('option');
                        option.value = device.id;
                        option.textContent = device.label || `Câmera ${device.id}`;
                        cameraSelect.appendChild(option);
                    });
                    // Tenta pré-selecionar a câmara traseira
                    const backCamera = devices.find(d => d.label.toLowerCase().includes('back') || d.label.toLowerCase().includes('traseira'));
                    if (backCamera) cameraSelect.value = backCamera.id;
                    
                    cameraSelect.disabled = false;
                    iniciarBtn.disabled = false;
                } else {
                    alert("Nenhuma câmera foi encontrada neste dispositivo.");
                }
            }).catch(err => {
                alert("Erro ao aceder às câmeras. Por favor, verifique as permissões do navegador.");
                console.error(err);
            });
        }

        // Função para iniciar o scanner
        function iniciarScanner() {
            const cameraId = cameraSelect.value;
            if (!cameraId) {
                alert("Por favor, selecione uma câmera.");
                return;
            }

            html5QrCode = new Html5Qrcode(leitorDivId);
            html5QrCode.start(
                cameraId, 
                { fps: 10, qrbox: { width: 250, height: 250 } },
                (decodedText) => onScanSuccess(decodedText),
                (errorMessage) => { /* Ignorar erros contínuos de leitura */ }
            ).then(() => {
                iniciarBtn.disabled = true;
                pararBtn.disabled = false;
            }).catch(err => {
                alert("Falha ao iniciar a câmera. Certifique-se de que deu permissão e está a usar um link HTTPS.");
            });
        }

        // Função para parar o scanner
        function pararScanner() {
            if (html5QrCode && html5QrCode.isScanning) {
                html5QrCode.stop().then(() => {
                    iniciarBtn.disabled = false;
                    pararBtn.disabled = true;
                }).catch(err => console.error("Erro ao parar a câmera.", err));
            }
        }

        // Função chamada a cada leitura bem-sucedida
        function onScanSuccess(codigoLido) {
            if (feedbackTimeout) return; // Impede leituras múltiplas muito rápidas

            if (idsParaEncontrar.has(codigoLido) && !idsEncontrados.has(codigoLido)) {
                idsEncontrados.add(codigoLido);
                mostrarFeedback(true);
                atualizarProgressoELista();
            } else if (!idsParaEncontrar.has(codigoLido)) {
                mostrarFeedback(false);
            }
        }
        
        // Função para atualizar o contador e a lista visual
        function atualizarProgressoELista() {
            progressoTexto.textContent = `Progresso: ${idsEncontrados.size} de ${idsParaEncontrar.size}`;
            listaIdsUL.innerHTML = '';
            idsParaEncontrar.forEach(id => {
                const li = document.createElement('li');
                li.textContent = id;
                if (idsEncontrados.has(id)) {
                    li.className = 'encontrado';
                    li.innerHTML = `<i class="fa-solid fa-check"></i> ${id}`;
                }
                listaIdsUL.appendChild(li);
            });
        }

        // Função para o feedback visual (cor da tela) e vibração
        function mostrarFeedback(isSuccess) {
            if (isSuccess && navigator.vibrate) navigator.vibrate(200);
            const feedbackClass = isSuccess ? 'feedback-sucesso' : 'feedback-erro';
            document.body.classList.add(feedbackClass);
            feedbackTimeout = setTimeout(() => {
                document.body.classList.remove(feedbackClass);
                feedbackTimeout = null;
            }, 1500);
        }

        // Configuração dos botões
        carregarBtn.addEventListener('click', carregarIds);
        iniciarBtn.addEventListener('click', iniciarScanner);
        pararBtn.addEventListener('click', pararScanner);

        // Inicia a deteção de câmeras assim que a página carrega
        popularSelecaoDeCamera();
    });
</script>

</body>
</html>

