import React, { useState, useEffect, useCallback, useRef } from 'react';
import { Html5Qrcode } from 'html5-qrcode';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Progress } from '@/components/ui/progress';
import { useToast } from '@/hooks/use-toast';
import { 
  Camera, 
  Save, 
  Trash2, 
  Play, 
  Square, 
  FileText, 
  Barcode,
  Check
} from 'lucide-react';

interface CameraDevice {
  id: string;
  label: string;
}

interface ScannerState {
  idsToFind: Set<string>;
  foundIds: Map<string, string>;
  cameras: CameraDevice[];
  isScanning: boolean;
}

export default function InventoryScanner() {
  const { toast } = useToast();
  const scannerRef = useRef<Html5Qrcode | null>(null);
  const readerRef = useRef<HTMLDivElement>(null);
  const audioContextRef = useRef<AudioContext | null>(null);

  const [state, setState] = useState<ScannerState>({
    idsToFind: new Set(),
    foundIds: new Map(),
    cameras: [],
    isScanning: false,
  });

  const [idInput, setIdInput] = useState('');
  const [selectedCamera, setSelectedCamera] = useState<string>('');
  const [feedback, setFeedback] = useState({ type: 'info', message: 'Aguardando operação...' });

  // Initialize audio context
  useEffect(() => {
    audioContextRef.current = new (window.AudioContext || (window as any).webkitAudioContext)();
    return () => {
      if (audioContextRef.current) {
        audioContextRef.current.close();
      }
    };
  }, []);

  const playBeep = useCallback((type: 'success' | 'error' | 'warning') => {
    if (!audioContextRef.current) return;

    const oscillator = audioContextRef.current.createOscillator();
    const gainNode = audioContextRef.current.createGain();
    
    oscillator.connect(gainNode);
    gainNode.connect(audioContextRef.current.destination);
    
    gainNode.gain.setValueAtTime(0.1, audioContextRef.current.currentTime);

    switch (type) {
      case 'success':
        oscillator.frequency.value = 800;
        oscillator.type = 'sine';
        break;
      case 'error':
        oscillator.frequency.value = 400;
        oscillator.type = 'triangle';
        break;
      case 'warning':
        oscillator.frequency.value = 600;
        oscillator.type = 'sawtooth';
        break;
    }
    
    oscillator.start();
    oscillator.stop(audioContextRef.current.currentTime + (type === 'error' ? 0.2 : 0.1));
  }, []);

  // Load state from localStorage
  useEffect(() => {
    const savedState = localStorage.getItem('inventoryScannerState');
    if (savedState) {
      const parsedState = JSON.parse(savedState);
      setState(prev => ({
        ...prev,
        idsToFind: new Set(parsedState.idsToFind),
        foundIds: new Map(parsedState.foundIds)
      }));
      setIdInput(parsedState.idsToFind.join('\n'));
    }
  }, []);

  // Save state to localStorage
  const saveState = useCallback(() => {
    const stateToSave = {
      idsToFind: Array.from(state.idsToFind),
      foundIds: Array.from(state.foundIds.entries())
    };
    localStorage.setItem('inventoryScannerState', JSON.stringify(stateToSave));
  }, [state.idsToFind, state.foundIds]);

  // Load cameras
  useEffect(() => {
    const loadCameras = async () => {
      try {
        const devices = await Html5Qrcode.getCameras();
        setState(prev => ({ ...prev, cameras: devices }));
        
        // Try to select back camera
        const backCamera = devices.find(d => 
          d.label.toLowerCase().includes('back') || 
          d.label.toLowerCase().includes('traseira')
        );
        if (backCamera) {
          setSelectedCamera(backCamera.id);
        } else if (devices.length > 0) {
          setSelectedCamera(devices[0].id);
        }
      } catch (error) {
        console.error('Error loading cameras:', error);
        setFeedback({ type: 'error', message: 'Não foi possível acessar as câmeras.' });
      }
    };

    loadCameras();
  }, []);

  const saveIds = () => {
    const ids = idInput.split('\n').map(id => id.trim()).filter(Boolean);
    setState(prev => ({ ...prev, idsToFind: new Set(ids) }));
    toast({ title: 'Lista de IDs salva com sucesso!' });
  };

  const clearAll = () => {
    setState(prev => ({ ...prev, idsToFind: new Set(), foundIds: new Map() }));
    setIdInput('');
    toast({ title: 'Dados limpos com sucesso!' });
  };

  const onScanSuccess = useCallback((decodedText: string) => {
    if (!state.isScanning) return;

    if (state.foundIds.has(decodedText)) {
      setFeedback({ type: 'warning', message: `Já escaneado: ${decodedText}` });
      playBeep('warning');
    } else if (state.idsToFind.has(decodedText)) {
      setState(prev => ({
        ...prev,
        foundIds: new Map(prev.foundIds).set(decodedText, new Date().toISOString())
      }));
      setFeedback({ type: 'success', message: `Encontrado: ${decodedText}` });
      playBeep('success');
      toast({ title: `Pacote encontrado: ${decodedText}` });
    } else {
      setFeedback({ type: 'error', message: `Inválido: ${decodedText}` });
      playBeep('error');
    }
  }, [state.isScanning, state.foundIds, state.idsToFind, playBeep, toast]);

  const startScanner = async () => {
    if (!selectedCamera || !readerRef.current) {
      setFeedback({ type: 'error', message: 'Nenhuma câmera selecionada.' });
      return;
    }

    try {
      scannerRef.current = new Html5Qrcode('scanner-reader');
      const config = { fps: 10, qrbox: { width: 250, height: 250 } };
      
      await scannerRef.current.start(
        selectedCamera,
        config,
        onScanSuccess,
        (errorMessage) => console.warn(errorMessage)
      );

      setState(prev => ({ ...prev, isScanning: true }));
      setFeedback({ type: 'success', message: 'Scanner iniciado. Aponte para um código.' });
    } catch (error) {
      console.error('Error starting scanner:', error);
      setFeedback({ type: 'error', message: 'Falha ao iniciar a câmera.' });
    }
  };

  const stopScanner = async () => {
    if (!scannerRef.current || !state.isScanning) return;

    try {
      await scannerRef.current.stop();
      setState(prev => ({ ...prev, isScanning: false }));
      setFeedback({ type: 'warning', message: 'Scanner parado.' });
      if (readerRef.current) {
        readerRef.current.innerHTML = '';
      }
    } catch (error) {
      console.error('Error stopping scanner:', error);
    }
  };

  const exportToCSV = () => {
    let csvContent = "data:text/csv;charset=utf-8,ID_PACOTE,DATA_HORA_LEITURA\n";
    state.foundIds.forEach((timestamp, id) => {
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
    
    toast({ title: 'Arquivo CSV exportado com sucesso!' });
  };

  // Save state when it changes
  useEffect(() => {
    saveState();
  }, [saveState]);

  const progress = state.idsToFind.size > 0 ? (state.foundIds.size / state.idsToFind.size) * 100 : 0;

  return (
    <div className="min-h-screen bg-background p-4">
      <div className="max-w-2xl mx-auto space-y-6">
        {/* Header */}
        <Card className="p-6">
          <div className="text-center">
            <h1 className="text-2xl font-bold flex items-center justify-center gap-2">
              <Barcode className="h-6 w-6" />
              Scanner de Inventário Pro
            </h1>
          </div>
        </Card>

        {/* Setup Section */}
        <Card className="p-6">
          <div className="space-y-4">
            <label className="text-sm font-medium">
              Cole os IDs a encontrar (um por linha):
            </label>
            <Textarea
              value={idInput}
              onChange={(e) => setIdInput(e.target.value)}
              placeholder="ID-PACOTE-001&#10;ID-PACOTE-002"
              className="min-h-[100px]"
            />
            <div className="flex gap-2">
              <Button onClick={saveIds} className="flex-1">
                <Save className="h-4 w-4 mr-2" />
                Salvar Lista
              </Button>
              <Button onClick={clearAll} variant="secondary" className="flex-1">
                <Trash2 className="h-4 w-4 mr-2" />
                Limpar Tudo
              </Button>
            </div>
          </div>
        </Card>

        {/* Scanner Section */}
        <Card className="p-6">
          <div className="space-y-4">
            <div>
              <label className="text-sm font-medium mb-2 block">Câmera:</label>
              <Select value={selectedCamera} onValueChange={setSelectedCamera}>
                <SelectTrigger>
                  <SelectValue placeholder="Selecione uma câmera" />
                </SelectTrigger>
                <SelectContent>
                  {state.cameras.map((camera) => (
                    <SelectItem key={camera.id} value={camera.id}>
                      {camera.label || `Câmera ${camera.id}`}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>

            <div className="flex gap-2">
              <Button 
                onClick={startScanner} 
                disabled={state.isScanning || !selectedCamera}
                className="flex-1"
              >
                <Camera className="h-4 w-4 mr-2" />
                Iniciar Scanner
              </Button>
              <Button 
                onClick={stopScanner} 
                disabled={!state.isScanning}
                variant="secondary"
                className="flex-1"
              >
                <Square className="h-4 w-4 mr-2" />
                Parar Scanner
              </Button>
            </div>

            <div 
              id="scanner-reader" 
              ref={readerRef}
              className="w-full aspect-square border rounded-lg overflow-hidden bg-card"
            />

            <div className={`p-4 rounded-lg text-center font-medium ${
              feedback.type === 'success' ? 'bg-success text-success-foreground' :
              feedback.type === 'error' ? 'bg-destructive text-destructive-foreground' :
              feedback.type === 'warning' ? 'bg-warning text-warning-foreground' :
              'bg-muted text-muted-foreground'
            }`}>
              {feedback.message}
            </div>
          </div>
        </Card>

        {/* Results Section */}
        <Card className="p-6">
          <div className="space-y-4">
            <div>
              <label className="text-sm font-medium">
                Progresso: {state.foundIds.size} de {state.idsToFind.size}
              </label>
              <Progress value={progress} className="mt-2" />
            </div>

            <div className="max-h-48 overflow-y-auto space-y-2">
              {Array.from(state.foundIds.entries()).reverse().map(([id, timestamp]) => {
                const time = new Date(timestamp).toLocaleTimeString('pt-BR');
                return (
                  <div key={id} className="flex justify-between items-center p-2 bg-muted rounded">
                    <span className="flex items-center gap-2">
                      <Check className="h-4 w-4 text-success" />
                      {id}
                    </span>
                    <small className="text-muted-foreground">{time}</small>
                  </div>
                );
              })}
            </div>

            <Button 
              onClick={exportToCSV} 
              disabled={state.foundIds.size === 0}
              className="w-full"
            >
              <FileText className="h-4 w-4 mr-2" />
              Exportar Encontrados (CSV)
            </Button>
          </div>
        </Card>
      </div>
    </div>
  );
}
