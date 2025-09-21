<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner de Inventário Pro</title>
    <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
    <style>
        :root {
            --bg-color: #1a202c;
            --card-color: #2d3748;
            --text-color: #e2e8f0;
            --text-muted-color: #a0aec0;
            --primary-color: #4299e1;
            --success-color: #48bb78;
            --error-color: #f56565;
            --warning-color: #f6ad55;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 1rem;
        }
        #app-container {
            max-width: 600px;
            margin: auto;
            background-color: var(--card-color);
            border-radius: 12px;
            box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1), 0 4px 6px -2px rgba(0,0,0,0.05);
            overflow: hidden;
        }
        .header, .section {
            padding: 1.5rem;
            border-bottom: 1px solid #4a5568;
        }
        .header { text-align: center; }
        h2 { margin: 0 0 0.5rem 0; font-size: 1.5rem; }
        label { font-weight: 600; display: block; margin-bottom: 0.5rem; }
        textarea, select {
            width: 100%;
            padding: 0.75rem;
            background-color: #1a202c;
            border: 1px solid #4a5568;
            color: var(--text-color);
            border-radius: 8px;
            margin-bottom: 1rem;
            box-sizing: border-box;
        }
        textarea { min-height: 100px; resize: vertical; }
        .button-group { display: flex; gap: 0.5rem; flex-wrap: wrap; }
        button {
            flex-grow: 1;
            padding: 0.75rem 1rem;
            border: none;
            border-radius: 8px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.2s;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 0.5rem;
        }
        .btn-primary { background-color: var(--primary-color); color: white; }
        .btn-primary:hover { background-color: #2b6cb0; }
        .btn-secondary { background-color: #4a5568; color: white; }
        .btn-secondary:hover { background-color: #718096; }
        .btn-success { background-color: var(--success-color); color: white; }
        #reader { width: 100%; border-radius: 8px; overflow: hidden; margin: 1rem 0; aspect-ratio: 1/1; }
        #feedback {
            font-size: 1.2rem;
            font-weight: bold;
            padding: 1rem;
            border-radius: 8px;
            text-align: center;
            margin-top: 1rem;
            transition: all 0.3s;
        }
        .feedback-success { background-color: var(--success-color); color: white; }
        .feedback-error { background-color: var(--error-color); color: white; }
        .feedback-warning { background-color: var(--warning-color); color: black; }
        #progress-container { margin-bottom: 1rem; }
        #progress-bar { width: 100%; background-color: #4a5568; border-radius: 8px; overflow: hidden; }
        #progress-fill { height: 10px; background-color: var(--success-color); width: 0%; transition: width 0.3s; }
        #found-list-container { max-height: 200px; overflow-y: auto; padding-right: 10px; }
        #found-list li {
            display: flex;
            justify-content: space-between;
            padding: 0.5rem 0;
            border-bottom: 1px solid #4a5568;
        }
        #found-list li small { color: var(--text-muted-color); }
    </style>
</head>
<body>
    <div id="app-container">
        <div class="header">
            <h2><i class="fa-solid fa-barcode"></i> Scanner de Inventário Pro</h2>
        </div>

        <div class="section" id="setup-section">
            <label for="id-input">Cole os IDs a encontrar (um por linha):</label>
            <textarea id="id-input" placeholder="ID-PACOTE-001&#10;ID-PACOTE-002"></textarea>
            <div class="button-group">
                <button id="save-btn" class="btn-primary"><i class="fa-solid fa-save"></i> Salvar Lista</button>
                <button id="clear-btn" class="btn-secondary"><i class="fa-solid fa-trash"></i> Limpar Tudo</button>
            </div>
        </div>

        <div class="section" id="scanner-section">
            <label for="camera-select">Câmera:</label>
            <select id="camera-select"></select>
            <div class="button-group">
                <button id="start-scan-btn" class="btn-primary"><i class="fa-solid fa-camera"></i> Iniciar Scanner</button>
                <button id="stop-scan-btn" class="btn-secondary"><i class="fa-solid fa-stop"></i> Parar Scanner</button>
            </div>
            <div id="reader"></div>
            <div id="feedback">Aguardando operação...</div>
        </div>

        <div class="section" id="results-section">
            <div id="progress-container">
                <label id="progress-label">Progresso: 0 de 0</label>
                <div id="progress-bar"><div id="progress-fill"></div></div>
            </div>
            <ul id="found-list"></ul>
            <div class="button-group" style="margin-top: 1rem;">
                <button id="export-btn" class="btn-success"><i class="fa-solid fa-file-csv"></i> Exportar Encontrados (CSV)</button>
            </div>
        </div>
    </div>

    <script>
    document.addEventListener('DOMContentLoaded', () => {
        const App = {
            // Gerenciamento de Estado
            state: {
                idsParaEncontrar: new Set(),
                idsEncontrados: new Map(),
                cameras: [],
                scanner: null,
                isScannerRunning: false,
            },

            // Elementos do DOM
            DOM: {
                idInput: document.getElementById('id-input'),
                saveBtn: document.getElementById('save-btn'),
                clearBtn: document.getElementById('clear-btn'),
                cameraSelect: document.getElementById('camera-select'),
                startScanBtn: document.getElementById('start-scan-btn'),
                stopScanBtn: document.getElementById('stop-scan-btn'),
                reader: document.getElementById('reader'),
                feedback: document.getElementById('feedback'),
                progressLabel: document.getElementById('progress-label'),
                progressFill: document.getElementById('progress-fill'),
                foundList: document.getElementById('found-list'),
                exportBtn: document.getElementById('export-btn'),
            },

            // Funções de Áudio
            audioContext: new (window.AudioContext || window.webkitAudioContext)(),
            playBeep(type) {
                const oscillator = this.audioContext.createOscillator();
                const gainNode = this.audioContext.createGain();
                oscillator.connect(gainNode);
                gainNode.connect(this.audioContext.destination);
                
                gainNode.gain.setValueAtTime(0.1, this.audioContext.currentTime);

                if (type === 'success') {
                    oscillator.frequency.value = 800;
                    oscillator.type = 'sine';
                } else if (type === 'error') {
                    oscillator.frequency.value = 400;
                    oscillator.type = 'triangle';
                } else if (type === 'warning') {
                    oscillator.frequency.value = 600;
                    oscillator.type = 'sawtooth';
                }
                
                oscillator.start();
                oscillator.stop(this.audioContext.currentTime + (type === 'error' ? 0.2 : 0.1));
            },

            // Métodos Principais
            init() {
                this.loadState();
                this.bindEvents();
                this.populateCameraSelect();
                this.updateUI();
                this.DOM.stopScanBtn.disabled = true;
            },

            bindEvents() {
                this.DOM.saveBtn.addEventListener('click', () => this.saveIds());
                this.DOM.clearBtn.addEventListener('click', () => this.clearAll());
                this.DOM.startScanBtn.addEventListener('click', () => this.startScanner());
                this.DOM.stopScanBtn.addEventListener('click', () => this.stopScanner());
                this.DOM.exportBtn.addEventListener('click', () => this.exportToCSV());
            },

            saveState() {
                const stateToSave = {
                    idsParaEncontrar: Array.from(this.state.idsParaEncontrar),
                    idsEncontrados: Array.from(this.state.idsEncontrados.entries())
                };
                localStorage.setItem('scannerProState', JSON.stringify(stateToSave));
            },

            loadState() {
                const savedState = localStorage.getItem('scannerProState');
                if (savedState) {
                    const parsedState = JSON.parse(savedState);
                    this.state.idsParaEncontrar = new Set(parsedState.idsParaEncontrar);
                    this.state.idsEncontrados = new Map(parsedState.idsEncontrados);
                    this.DOM.idInput.value = Array.from(this.state.idsParaEncontrar).join('\n');
                }
            },
            
            saveIds() {
                const ids = this.DOM.idInput.value.split('\n').map(id => id.trim()).filter(Boolean);
                this.state.idsParaEncontrar = new Set(ids);
                this.saveState();
                this.updateUI();
                alert('Lista de IDs salva com sucesso!');
            },

            clearAll() {
                if (confirm('Tem certeza que deseja apagar a lista de IDs e todos os pacotes encontrados?')) {
                    this.state.idsParaEncontrar.clear();
                    this.state.idsEncontrados.clear();
                    this.DOM.idInput.value = '';
                    this.saveState();
                    this.updateUI();
                }
            },

            updateUI() {
                const total = this.state.idsParaEncontrar.size;
                const found = this.state.idsEncontrados.size;
                
                this.DOM.progressLabel.textContent = `Progresso: ${found} de ${total}`;
                
                const progressPercentage = total > 0 ? (found / total) * 100 : 0;
                this.DOM.progressFill.style.width = `${progressPercentage}%`;

                this.DOM.foundList.innerHTML = '';
                this.state.idsEncontrados.forEach((timestamp, id) => {
                    const li = document.createElement('li');
                    const time = new Date(timestamp).toLocaleTimeString('pt-BR');
                    li.innerHTML = `<span><i class="fa-solid fa-check" style="color: var(--success-color);"></i> ${id}</span> <small>${time}</small>`;
                    this.DOM.foundList.prepend(li);
                });
                
                this.DOM.exportBtn.disabled = found === 0;
            },

            showFeedback(type, message) {
                const feedback = this.DOM.feedback;
                feedback.textContent = message;
                feedback.className = `feedback-${type}`;
            },

            async populateCameraSelect() {
                try {
                    const devices = await Html5Qrcode.getCameras();
                    this.state.cameras = devices;
                    if (devices && devices.length) {
                        this.DOM.cameraSelect.innerHTML = '';
                        devices.forEach(device => {
                            const option = document.createElement('option');
                            option.value = device.id;
                            option.textContent = device.label || `Câmera ${this.DOM.cameraSelect.length + 1}`;
                            this.DOM.cameraSelect.appendChild(option);
                        });
                        // Tenta pré-selecionar a câmera traseira
                        const backCamera = devices.find(d => d.label.toLowerCase().includes('back') || d.label.toLowerCase().includes('traseira'));
                        if (backCamera) {
                            this.DOM.cameraSelect.value = backCamera.id;
                        }
                    }
                } catch (err) {
                    console.error("Erro ao obter câmeras:", err);
                    this.showFeedback('error', 'Não foi possível acessar as câmeras.');
                }
            },

            startScanner() {
                if (this.state.isScannerRunning) return;

                const selectedCameraId = this.DOM.cameraSelect.value;
                if (!selectedCameraId) {
                    this.showFeedback('error', 'Nenhuma câmera selecionada.');
                    return;
                }

                this.state.scanner = new Html5Qrcode('reader');
                const config = { fps: 10, qrbox: { width: 250, height: 250 } };
                
                this.state.scanner.start(selectedCameraId, config, 
                    (decodedText) => this.onScanSuccess(decodedText),
                    (errorMessage) => console.warn(errorMessage)
                ).then(() => {
                    this.state.isScannerRunning = true;
                    this.DOM.startScanBtn.disabled = true;
                    this.DOM.stopScanBtn.disabled = false;
                    this.showFeedback('success', 'Scanner iniciado. Aponte para um código.');
                }).catch(err => {
                    console.error("Erro ao iniciar scanner:", err);
                    this.showFeedback('error', 'Falha ao iniciar a câmera.');
                });
            },

            stopScanner() {
                if (!this.state.isScannerRunning || !this.state.scanner) return;
                
                this.state.scanner.stop().then(() => {
                    this.state.isScannerRunning = false;
                    this.DOM.startScanBtn.disabled = false;
                    this.DOM.stopScanBtn.disabled = true;
                    this.showFeedback('warning', 'Scanner parado.');
                    this.DOM.reader.innerHTML = '';
                }).catch(err => {
                    console.error("Erro ao parar scanner:", err);
                });
            },
            
            onScanSuccess(decodedText) {
                if (!this.state.isScannerRunning) return;

                if (this.state.idsEncontrados.has(decodedText)) {
                    this.showFeedback('warning', `Já escaneado: ${decodedText}`);
                    this.playBeep('warning');
                } else if (this.state.idsParaEncontrar.has(decodedText)) {
                    this.state.idsEncontrados.set(decodedText, new Date().toISOString());
                    this.showFeedback('success', `Encontrado: ${decodedText}`);
                    this.playBeep('success');
                    this.updateUI();
                    this.saveState();
                } else {
                    this.showFeedback('error', `Inválido: ${decodedText}`);
                    this.playBeep('error');
                }
            },

            exportToCSV() {
                let csvContent = "data:text/csv;charset=utf-8,ID_PACOTE,DATA_HORA_LEITURA\n";
                this.state.idsEncontrados.forEach((timestamp, id) => {
                    const date = new Date(timestamp);
                    const formattedDate = `${date.toLocaleDateString('pt-BR')} ${date.toLocaleTimeString('pt-BR')}`;
                    csvContent += `${id},${formattedDate}\n`;
                });

                const encodedUri = encodeURI(csvContent);
                const link = document.createElement("a");
                link.setAttribute("href", encodedUri);
                link.setAttribute("download", `inventario_encontrados_${new Date().toISOString().split('T')[0]}.csv`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            }
        };

        App.init();
    });
    </script>
</body>
</html>
