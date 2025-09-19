# Duplo-check-x<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scanner de Pacotes</title>
  <script src="https://unpkg.com/html5-qrcode/minified/html5-qrcode.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background: #f3f3f3;
    }
    #resultado {
      margin-top: 20px;
      font-size: 24px;
      font-weight: bold;
      padding: 15px;
      border-radius: 8px;
      display: inline-block;
    }
    .ok {
      background: #28a745;
      color: white;
    }
    .erro {
      background: #dc3545;
      color: white;
    }
  </style>
</head>
<body>
  <h2>Scanner de Pacotes</h2>
  <div id="reader" style="width: 300px; margin: auto;"></div>
  <div id="resultado">Aguardando leitura...</div>

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

    function verificarCodigo(decodedText) {
      if (idsValidos.includes(decodedText)) {
        resultadoDiv.innerText = "✅ Pacote encontrado: " + decodedText;
        resultadoDiv.className = "ok";
      } else {
        resultadoDiv.innerText = "❌ Não encontrado: " + decodedText;
        resultadoDiv.className = "erro";
      }
    }

    const html5QrCode = new Html5Qrcode("reader");
    const config = { fps: 10, qrbox: { width: 250, height: 250 } };

    html5QrCode.start(
      { facingMode: "environment" }, // Câmera traseira
      config,
      verificarCodigo
    ).catch(err => {
      console.error("Erro ao iniciar câmera: ", err);
    });
  </script>
</body>
</html>
