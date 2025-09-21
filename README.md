<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner de Inventário</title>
    <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
    <style>
        :root {
            --bg-color: #f0f2f5; --card-color: #ffffff; --text-color: #1a202c;
            --text-muted-color: #718096; --primary-color: #4299e1; --primary-dark-color: #2b6cb0;
            --success-color: #48bb78; --border-color: #e2e8f0;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color); color: var(--text-color); margin: 0; padding: 1rem;
        }
        #app-container { max-width: 600px; margin: auto; space-y: 1rem; display: flex; flex-direction: column; gap: 1rem; }
        .card { background-color: var(--card-color); border: 1px solid var(--border-color); border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06); padding: 1.5rem; }
        h2, h3 { margin: 0 0 1rem 0; font-size: 1.25rem; }
        label { font-weight: 600; display: block; margin-bottom: 0.5rem; font-size: 0.875rem; }
        textarea, select { width: 100%; padding: 0.75rem; border: 1px solid var(--border-color); border-radius: 8px; margin-bottom: 1rem; box-sizing: border-box; font-size: 1rem; }
        .button-group { display: flex; gap: 0.5rem; }
        button { flex-grow: 1; padding: 0.75rem 1rem; border: none; border-radius: 8px; font-size: 1rem; font-weight: 600; cursor: pointer; transition: background-color 0.2s; display: flex; align-items: center; justify-content: center; gap: 0.5rem; background-color: var(--primary-color); color: white; }
        button:hover { background-color: var(--primary-dark-color); }
        button:disabled { background-color: #cbd5e0; cursor: not-allowed; }
        button.secondary { background-color: #a0aec0; }
        button.secondary:hover { background-color: var(--text-muted-color); }
        #reader { width: 100%; border-radius: 8px; overflow: hidden; margin-top: 1rem; aspect-ratio: 1/1; border: 1px solid var(--border-color); }
        #progress-bar { width: 100%; background-color: var(--border-color); border-radius: 8px; overflow: hidden; height: 1rem; }
        #progress-fill { height: 100%; background-color: var(--success-color); width: 0%; transition: width 0.3s; }
        #task-list-container { max-height: 200px; overflow-y: auto; border: 1px solid var(--border-color); border-radius: 8px; padding: 0.5rem; }
        #task-list li { display: flex; align-items: center; justify-content: space-between; padding: 0.5rem; transition: all 0.3s ease; }
        #task-list li:not(:last-child) { border-bottom: 1px solid var(--border-color); }
        .item-encontrado { color: var(--success-color); font-weight: 500; }
    </style>
</head>
<body>
    <div id="app-container">
        <div class="card">
            <h2 style="text-align:center;"><i class="fa-solid fa-barcode"></i> Scanner de Inventário</h2>
        </div>

        <div class="card">
            <h3><i class="fa-solid fa-list-check"></i> 1. Preparação</h3>
            <label for="id-input">Cole os IDs a encontrar (um por linha):</label>
            <textarea id="id-input" rows="5"></textarea>
            <div class="button-group">
                <button id="save-btn"><i class="fa-solid fa-save"></i> Carregar Lista</button>
            </div>
        </div>

        <div class="card">
            <h3><i class="fa-solid fa-camera"></i> 2. Escaneamento</h3>
            <label for="camera-select">Câmera:</label>
            <select id="camera-select"></select>
            <div class="button-group">
                <button id="start-scan-btn"><i class="fa-solid fa-play"></i> Iniciar</button>
                <button id="stop-scan-btn" class="secondary"><i class="fa-solid fa-stop"></i> Parar</button>
            </div>
            <div id="reader"></div>
        </div>

        <div class="card">
            <h3><i class="fa-solid fa-chart-line"></i> 3. Resultados</h3>
            <label id="progress-label">Progresso: 0 de 0</label>
            <div id="progress-bar"><div id="progress-fill"></div></div>
            <div id="task-list-container" style="margin-top: 1rem;">
                <ul id="task-list" style="list-style: none; padding: 0; margin: 0;"></ul>
            </div>
        </div>
    </div>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const App = {
        state: {
            tarefas: [],
            scanner: null,
            isScannerRunning: false,
        },
        DOM: {
            idInput: document.getElementById('id-input'),
            saveBtn: document.getElementById('save-btn'),
            cameraSelect: document.getElementById('camera-select'),
            startScanBtn: document.getElementById('start-scan-btn'),
            stopScanBtn: document.getElementById('stop-scan-btn'),
            reader: document.getElementById('reader'),
            progressLabel: document.getElementById('progress-label'),
            progressFill: document.getElementById('progress-fill'),
            taskList: document.getElementById('task-list'),
        },
        init() {
            this.loadState();
            this.bindEvents();
            this.populateCameraSelect();
            this.updateUI();
        },
        bindEvents() {
            this.DOM.saveBtn.addEventListener('click', () => this.loadIdsFromTextarea());
            this.DOM.startScanBtn.addEventListener('click', () => this.startScanner());
            this.DOM.stopScanBtn.addEventListener('click', () => this.stopScanner());
        },
        saveState() {
            localStorage.setItem('scannerTarefasHTML', JSON.stringify(this.state.tarefas));
        },
        loadState() {
            const savedTarefas = localStorage.getItem('scannerTarefasHTML');
            if (savedTarefas) {
                this.state.tarefas = JSON.parse(savedTarefas);
                this.DOM.idInput.value = this.state.tarefas.map(t => t.id).join('\n');
            }
        },
        loadIdsFromTextarea() {
            const ids = this.DOM.idInput.value.split('\n').map(id => id.trim()).filter(Boolean);
            this.state.tarefas = ids.map(id => ({ id: id, status: 'pendente' }));
            this.saveState();
            this.updateUI();
            alert(`${ids.length} IDs carregados.`);
        },
        updateUI() {
            const total = this.state.tarefas.length;
            const encontrados = this.state.tarefas.filter(t => t.status === 'encontrado').length;
            this.DOM.progressLabel.textContent = `Progresso: ${encontrados} de ${total}`;
            const percentage = total > 0 ? (encontrados / total) * 100 : 0;
            this.DOM.progressFill.style.width = `${percentage}%`;
            this.DOM.taskList.innerHTML = '';
            if (total === 0) {
                this.DOM.taskList.innerHTML = `<li>Nenhuma lista carregada.</li>`;
            } else {
                this.state.tarefas.forEach(tarefa => {
                    const li = document.createElement('li');
                    li.className = tarefa.status === 'encontrado' ? 'item-encontrado' : 'item-pendente';
                    const iconClass = tarefa.status === 'encontrado' ? 'fa-solid fa-check-circle' : 'fa-regular fa-circle';
                    li.innerHTML = `<span><i class="${iconClass}"></i> ${tarefa.id}</span>`;
                    this.DOM.taskList.appendChild(li);
                });
            }
            this.DOM.stopScanBtn.disabled = !this.state.isScannerRunning;
            this.DOM.startScanBtn.disabled = this.state.isScannerRunning;
        },
        async populateCameraSelect() {
            try {
                const devices = await Html5Qrcode.getCameras();
                if (devices && devices.length) {
                    this.DOM.cameraSelect.innerHTML = devices.map(device => `<option value="${device.id}">${device.label}</option>`).join('');
                }
            } catch (err) {
                alert('Erro ao obter câmeras: ' + err);
            }
        },
        startScanner() {
            const selectedCameraId = this.DOM.cameraSelect.value;
            this.state.scanner = new Html5Qrcode('reader');
            this.state.scanner.start(
                selectedCameraId, { fps: 10, qrbox: { width: 250, height: 250 } },
                (decodedText) => this.onScanSuccess(decodedText),
                (errorMessage) => {}
            ).then(() => {
                this.state.isScannerRunning = true;
                this.updateUI();
            }).catch(err => {
                alert('Falha ao iniciar a câmera. Verifique as permissões e se está em HTTPS.');
            });
        },
        stopScanner() {
            if (!this.state.isScannerRunning) return;
            this.state.scanner.stop().then(() => {
                this.state.isScannerRunning = false;
                this.DOM.reader.innerHTML = '';
                this.updateUI();
            });
        },
        onScanSuccess(decodedText) {
            const tarefa = this.state.tarefas.find(t => t.id === decodedText);
            if (tarefa && tarefa.status === 'pendente') {
                tarefa.status = 'encontrado';
                this.saveState();
                this.updateUI();
                // Efeito visual de "flash"
                document.body.style.backgroundColor = 'var(--success-color)';
                setTimeout(() => { document.body.style.backgroundColor = 'var(--bg-color)'; }, 500);
            }
        },
    };
    App.init();
});
</script>
</body>
</html>
