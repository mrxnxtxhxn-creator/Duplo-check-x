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
            margin: 0;
            padding: 1rem;
            background-color: var(--cor-fundo);
            color: var(--cor-texto);
            transition: background-color 0.2s ease-in-out;
        }
        /* ESTILOS PARA O FEEDBACK DE TELA CHEIA */
        body.feedback-sucesso { background-color: var(--cor-sucesso); }
        body.feedback-erro { background-color: var(--cor-erro); }
        .container {
            max-width: 500px; margin: 0 auto; background-color: var(--cor-fundo-card);
            padding: 1.5rem; border-radius: 12px; border: 1px solid var(--cor-borda);
            box-shadow: 0 4px 20px rgba(0,0,0,0.25);
        }
        h1 { text-align: center; margin-top: 0; color: var(--cor-primaria); }
        .secao { margin-bottom: 2rem; }
        label { display: block; font-weight: 600; margin-bottom: 0.5rem; color: #BDBDBD;}
        textarea {
            width: 100%; min-height: 120px; padding: 0.75rem; border: 1px solid var(--cor-borda);
            border-radius: 4px; font-size: 1rem; box-sizing: border-box; background-color: var(--cor-fundo);
            color: var(--cor-texto);
        }
        textarea:focus {
            outline: none; border-color: var(--cor-primaria);
            box-shadow: 0 0 0 2px rgba(255, 193, 7, 0.5);
        }
        button {
            display: block; width: 100%; padding: 0.75rem; font-size: 1.1rem;
            font-weight: 600; color: var(--cor-texto-botao); background-color: var(--cor-primaria);
            border: none; border-radius: 4px; cursor: pointer; margin-top: 1rem;
            transition: background-color 0.2s;
        }
        button:hover { background-color: var(--cor-primaria-hover); }
        button:disabled { background-color: #424242; color: #757575; cursor: not-allowed; }
        #leitor-container {
            width: 100%; border: 2px solid var(--cor-borda); border-radius: 4px;
            overflow: hidden; position: relative;
        }
        #leitor { width: 100%; display: block; }
        #feedback-texto { text-align: center; font-weight: bold; font-size: 1.2rem; margin-top: 1rem; height: 30px;}
        
        /* NOVOS ESTILOS PARA PROGRESSO E LISTA */
        #progresso-container { margin-bottom: 1rem; }
        #progresso-texto { font-size: 1.2rem; font-weight: bold; text-align: center; color: var(--cor-primaria); }
        #lista-container {
            max-height: 200px;
            overflow-y: auto;
            border: 1px solid var(--cor-borda);
            border-radius: 4px;
            padding: 0.5rem;
            margin-top: 1rem;
        }
        #lista-ids { list-style: none; padding: 0; margin: 0; }
        #lista-ids li {
            padding: 0.5rem;
            border-bottom: 1px solid var(--cor-borda);
            transition: all 0.3s ease;
        }
        #lista-ids li:last-child { border-bottom: none; }
        #lista-ids li.encontrado {
            color: var(--cor-sucesso);
            font-weight: bold;
            text-decoration: line-through;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1><i class="fa-solid fa-barcode"></i> Scanner de Pacotes</h1>

        <div class="secao">
            <label for="ids-input">1. Cole os IDs (um por linha):</label>
            <textarea id="ids-input" placeholder="PACOTE-001&#10;PACOTE-002&#10;PACOTE-003"></textarea>
            <button id="carregar-btn">Carregar IDs e Iniciar</button>
        </div>

        <div class="secao">
            <label>2. Escaneamento em Tempo Real</label>
            <div id="leitor-container">
                <div id="leitor"></div>
            </div>
            <div id="feedback-texto">Aguardando leitura...</div>
            <button id="parar-btn" disabled>Parar Câmera</button>
        </div>
        
        <!-- NOVA SEÇÃO DE PROGRESSO -->
        <div class="secao">
            <div id="progresso-container">
                <div id="progresso-texto">Progresso: 0 de 0</div>
            </div>
            <label>Lista de Tarefas:</label>
            <div id="lista-container">
                <ul id="lista-ids">
                    <li>Nenhuma lista carregada.</li>
                </ul>
            </div>
        </div>
    </div>

<!-- A biblioteca do scanner DEVE ser carregada antes do script que a utiliza -->
<script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', () => {
    const idsInput = document.getElementById('ids-input');
    const carregarBtn = document.getElementById('carregar-btn');
    const pararBtn = document.getElementById('parar-btn');
    const feedbackTexto = document.getElementById('feedback-texto');
    const progressoTexto = document.getElementById('progresso-texto');
    const listaIdsUL = document.getElementById('lista-ids');
    const leitorDivId = 'leitor';
    
    let html5QrCode = null;
    let idsParaEncontrar = new Set();
    let idsEncontrados = new Set(); // NOVO: Guarda os IDs já encontrados
    let feedbackTimeout = null;

    function carregarIds() {
        const ids = idsInput.value.split('\n').map(id => id.trim()).filter(id => id);
        idsParaEncontrar = new Set(ids);
        idsEncontrados.clear(); // Limpa a contagem anterior

        if (ids.length > 0) {
            atualizarProgressoELista();
            iniciarScanner();
        } else {
            alert("Por favor, insira pelo menos um ID.");
        }
    }

    function iniciarScanner() {
        if (!html5QrCode) html5QrCode = new Html5Qrcode(leitorDivId);
        
        Html5Qrcode.getCameras().then(devices => {
            if (devices && devices.length) {
                const cameraId = devices[devices.length - 1].id;
                html5QrCode.start(cameraId, { fps: 10, qrbox: { width: 250, height: 250 } },
                    (decodedText) => onScanSuccess(decodedText),
                    (errorMessage) => {})
                .then(() => {
                    carregarBtn.disabled = true;
                    pararBtn.disabled = false;
                    idsInput.disabled = true;
                    feedbackTexto.textContent = "Câmera ativa!";
                })
                .catch(err => alert("Erro ao iniciar a câmera. Verifique as permissões e se está usando HTTPS."));
            } else { alert("Nenhuma câmera encontrada."); }
        }).catch(err => alert("Erro ao acessar as câmeras."));
    }

    function pararScanner() {
        if (html5QrCode && html5QrCode.isScanning) {
            html5QrCode.stop().then(() => {
                carregarBtn.disabled = false;
                pararBtn.disabled = true;
                idsInput.disabled = false;
                feedbackTexto.textContent = "Câmera parada.";
            }).catch(err => console.error("Falha ao parar a câmera.", err));
        }
    }

    function onScanSuccess(codigoLido) {
        if (feedbackTimeout) return;
        
        // Verifica se o ID está na lista E se ainda não foi encontrado
        if (idsParaEncontrar.has(codigoLido) && !idsEncontrados.has(codigoLido)) {
            idsEncontrados.add(codigoLido);
            feedbackTexto.textContent = `✅ ENCONTRADO: ${codigoLido}`;
            mostrarFeedback(true);
            atualizarProgressoELista(); // Atualiza a lista visual
        } else if (!idsParaEncontrar.has(codigoLido)) {
            feedbackTexto.textContent = `❌ INVÁLIDO: ${codigoLido}`;
            mostrarFeedback(false);
        }
        // Se já foi encontrado, não faz nada para manter o fluxo rápido
    }
    
    // NOVO: Atualiza o contador de progresso e a lista visual
    function atualizarProgressoELista() {
        progressoTexto.textContent = `Progresso: ${idsEncontrados.size} de ${idsParaEncontrar.size}`;
        
        listaIdsUL.innerHTML = '';
        idsParaEncontrar.forEach(id => {
            const li = document.createElement('li');
            li.textContent = id;
            li.id = `item-${id}`; // Adiciona um ID para fácil acesso
            if (idsEncontrados.has(id)) {
                li.className = 'encontrado';
                li.innerHTML = `<i class="fa-solid fa-check"></i> ${id}`;
            }
            listaIdsUL.appendChild(li);
        });
    }

    function mostrarFeedback(isSuccess) {
        if (isSuccess && navigator.vibrate) navigator.vibrate(200);
        const feedbackClass = isSuccess ? 'feedback-sucesso' : 'feedback-erro';
        document.body.classList.add(feedbackClass);
        feedbackTimeout = setTimeout(() => {
            document.body.classList.remove(feedbackClass);
            feedbackTimeout = null;
        }, 1500);
    }

    carregarBtn.addEventListener('click', carregarIds);
    pararBtn.addEventListener('click', pararScanner);
});
</script>

</body>
</html>

