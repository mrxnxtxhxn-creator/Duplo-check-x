// ...existing code...
<script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', () => {
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
    let isScanning = false;

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

    async function popularSelecaoDeCamera() {
        if (typeof Html5Qrcode === 'undefined') {
            alert("Biblioteca html5-qrcode não carregada.");
            return;
        }

        // Requer contexto seguro (https ou localhost)
        if (!location.protocol.startsWith('https') && location.hostname !== 'localhost') {
            console.warn('Contexto não seguro:', location.protocol, location.hostname);
            // ainda tenta listar devices (alguns navegadores não permitem)
        }

        try {
            const devices = await Html5Qrcode.getCameras();
            if (devices && devices.length) {
                cameraSelect.innerHTML = '';
                devices.forEach(device => {
                    const option = document.createElement('option');
                    option.value = device.id;
                    option.textContent = device.label || `Câmera ${device.id}`;
                    cameraSelect.appendChild(option);
                });
                // tenta escolher traseira se existir
                const back = devices.find(d => (d.label || '').toLowerCase().includes('back') || (d.label || '').toLowerCase().includes('traseira') || (d.label || '').toLowerCase().includes('rear'));
                if (back) cameraSelect.value = back.id;

                cameraSelect.disabled = false;
                iniciarBtn.disabled = false;
                pararBtn.disabled = true;
            } else {
                // fallback: solicitar permissão e usar enumerateDevices
                console.warn('Nenhuma câmera retornada por Html5Qrcode.getCameras(), tentando enumerateDevices');
                await navigator.mediaDevices.getUserMedia({ video: true }).catch(e => {
                    console.error('Permissão negada ou erro getUserMedia:', e);
                    alert('Permissão de câmera necessária. Verifique as configurações do navegador.');
                    throw e;
                });
                const all = await navigator.mediaDevices.enumerateDevices();
                const videoInputs = all.filter(d => d.kind === 'videoinput');
                if (videoInputs.length === 0) {
                    alert('Nenhuma câmera encontrada.');
                    return;
                }
                cameraSelect.innerHTML = '';
                videoInputs.forEach(d => {
                    const opt = document.createElement('option');
                    opt.value = d.deviceId;
                    opt.textContent = d.label || `Câmera ${d.deviceId}`;
                    cameraSelect.appendChild(opt);
                });
                cameraSelect.disabled = false;
                iniciarBtn.disabled = false;
                pararBtn.disabled = true;
            }
        } catch (err) {
            console.error('Erro ao listar câmeras:', err);
            alert('Erro ao aceder às câmeras. Verifique permissões e servidor (HTTPS). Veja console para detalhes.');
        }
    }

    async function iniciarScanner() {
        if (isScanning) return;
        const cameraId = cameraSelect.value;
        if (!cameraId) {
            alert("Por favor, selecione uma câmera.");
            return;
        }

        // Garante tamanho do leitor para mobile
        const leitorEl = document.getElementById(leitorDivId);
        leitorEl.style.minHeight = '300px';

        html5QrCode = new Html5Qrcode(leitorDivId, /* verbose= */ false);
        isScanning = true;

        // use constraint com deviceId para evitar problemas em alguns navegadores
        const cameraConfig = { deviceId: { exact: cameraId } };

        try {
            await html5QrCode.start(
                cameraConfig,
                { fps: 10, qrbox: { width: 250, height: 250 } },
                decodedText => onScanSuccess(decodedText),
                errorMessage => { /* opcional: console.log(errorMessage) */ }
            );
            iniciarBtn.disabled = true;
            pararBtn.disabled = false;
        } catch (err) {
            isScanning = false;
            console.error('Falha ao iniciar câmera:', err);
            // Mensagem orientando sobre HTTPS/permissões
            alert('Falha ao iniciar a câmera. Certifique-se de que deu permissão e que a página está servida por HTTPS (ou localhost). Verifique o console para detalhes.');
        }
    }

    async function pararScanner() {
        if (!html5QrCode || !isScanning) return;
        try {
            await html5QrCode.stop();
        } catch (err) {
            console.error('Erro ao parar a câmera:', err);
        } finally {
            isScanning = false;
            iniciarBtn.disabled = false;
            pararBtn.disabled = true;
            // limpa reader
            try { html5QrCode.clear(); } catch(e){/* ignore */ }
            html5QrCode = null;
        }
    }

    function onScanSuccess(codigoLido) {
        if (feedbackTimeout) return;
        if (idsParaEncontrar.has(codigoLido) && !idsEncontrados.has(codigoLido)) {
            idsEncontrados.add(codigoLido);
            mostrarFeedback(true);
            atualizarProgressoELista();
        } else if (!idsParaEncontrar.has(codigoLido)) {
            mostrarFeedback(false);
        }
    }

    function atualizarProgressoELista() {
        progressoTexto.textContent = `Progresso: ${idsEncontrados.size} de ${idsParaEncontrar.size}`;
        listaIdsUL.innerHTML = '';
        if (idsParaEncontrar.size === 0) {
            listaIdsUL.innerHTML = '<li>Nenhuma lista carregada.</li>';
            return;
        }
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

    function mostrarFeedback(isSuccess) {
        if (isSuccess && navigator.vibrate) navigator.vibrate(200);
        const feedbackClass = isSuccess ? 'feedback-sucesso' : 'feedback-erro';
        document.body.classList.add(feedbackClass);
        feedbackTimeout = setTimeout(() => {
            document.body.classList.remove(feedbackClass);
            feedbackTimeout = null;
        }, 1200);
    }

    carregarBtn.addEventListener('click', carregarIds);
    iniciarBtn.addEventListener('click', iniciarScanner);
    pararBtn.addEventListener('click', pararScanner);

    // Inicializa seleção de câmeras
    popularSelecaoDeCamera();
});
</script>
// ...existing code...
