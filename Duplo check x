<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scanner de Pacotes</title>
  <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
  <style>
    body { text-align:center; font-family: Arial, sans-serif; }
    #reader { width: 320px; margin: auto; }
    #resultado { margin-top: 15px; font-size: 20px; font-weight: bold; }
    .ok { color: green; }
    .erro { color: red; }
  </style>
</head>
<body>
  <h2>Scanner de Pacotes</h2>
  <div id="reader"></div>
  <div id="resultado">Aguardando leitura...</div>
  <div id="msg" style="color:red;"></div>

  <script>
    const idsValidos = [
      "45521630151","45527442396","45527911927","45528191622",
      "45528811304","45528961246","45529519903","45529519921",
      "45529909862","45530728317","45530892544","45531849631",
      "45533653636","45534947657"
    ];

    const resultadoDiv = document.getElementById("resultado");
    const msgDiv = document.getElementById("msg");
    let scanner = null;

    function verificarCodigo(code) {
      if (idsValidos.includes(code)) {
        resultadoDiv.innerText = "✅ Encontrado: " + code;
        resultadoDiv.className = "ok";
      } else {
        resultadoDiv.innerText = "❌ Não encontrado: " + code;
        resultadoDiv.className = "erro";
      }
    }

    function startScanner(cameraId = null) {
      scanner = new Html5Qrcode("reader");
      const config = { fps: 10, qrbox: { width: 250, height: 250 } };

      scanner.start(
        cameraId ? cameraId : { facingMode: "environment" },
        config,
        verificarCodigo
      ).catch(err => {
        msgDiv.innerText = "Erro ao iniciar câmera: " + err;
        console.warn(err);

        // fallback: tenta pegar a primeira câmera disponível
        Html5Qrcode.getCameras().then(devices => {
          if (devices.length > 0) {
            msgDiv.innerText = "Tentando outra câmera...";
            scanner.start(devices[0].id, config, verificarCodigo)
              .catch(e2 => {
                msgDiv.innerText = "Falhou: " + e2;
              });
          } else {
            msgDiv.innerText = "Nenhuma câmera encontrada.";
          }
        });
      });
    }

    window.addEventListener("load", () => {
      startScanner();
    });
  </script>
</body>
</html>
