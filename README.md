
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Mestre - Seletor de Tempos</title>
    <style>
        body {
            background-color: #0b0e11;
            color: #eaecef;
            font-family: monospace;
            padding: 10px;
            margin: 0;
        }
        h2 { text-align: center; color: #f0b90b; font-size: 16px; margin-bottom: 10px; }
        
        .botoes-container {
            display: flex;
            flex-wrap: wrap;
            gap: 5px;
            justify-content: center;
            margin-bottom: 15px;
        }
        .btn-tempo {
            background-color: #1e2329;
            color: #eaecef;
            border: 1px solid #474d57;
            padding: 8px 12px;
            font-family: monospace;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
        }
        .btn-tempo.ativo {
            background-color: #f0b90b;
            color: #000;
            border-color: #f0b90b;
        }

        .alerta-box {
            background-color: #1e2329;
            border: 2px solid #f0b90b;
            padding: 10px;
            text-align: center;
            font-size: 14px;
            font-weight: bold;
            margin-bottom: 10px;
            border-radius: 5px;
        }
        .painel-info {
            background-color: #181a20;
            border: 1px solid #2b313a;
            padding: 12px;
            border-radius: 6px;
            font-size: 13px;
        }
        .linha-info {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            border-bottom: 1px solid #2b313a;
            padding-bottom: 6px;
        }
        .medias-titulo {
            color: #f0b90b;
            margin-top: 10px;
            margin-bottom: 6px;
            font-weight: bold;
        }
        .medias-grid {
            color: #848e9c;
            font-size: 11px;
            line-height: 1.5;
        }
    </style>
</head>
<body>

    <h2>BTUSD - SELETOR DE CENTROS & MÉDIAS</h2>

    <div class="botoes-container">
        <button class="btn-tempo ativo" onclick="mudarTempo('1m', this)">M1</button>
        <button class="btn-tempo" onclick="mudarTempo('5m', this)">M5</button>
        <button class="btn-tempo" onclick="mudarTempo('15m', this)">M15</button>
        <button class="btn-tempo" onclick="mudarTempo('30m', this)">M30</button>
        <button class="btn-tempo" onclick="mudarTempo('1h', this)">H1</button>
        <button class="btn-tempo" onclick="mudarTempo('4h', this)">H4</button>
        <button class="btn-tempo" onclick="mudarTempo('1d', this)">D1</button>
    </div>

    <div id="status-sinal" class="alerta-box" style="color: #f0b90b;">
        🔍 CARREGANDO...
    </div>

    <div id="painel-detalhes" class="painel-info">
        Selecione um tempo acima...
    </div>

<script>
let tempoAtualBinance = '1m';
const periodosMa = [19, 38, 97, 191, 383, 575, 979];
let dadosGlobais = [];

function mudarTempo(intervalo, elemento) {
    tempoAtualBinance = intervalo;
    
    // Atualiza botão ativo visualmente
    document.querySelectorAll('.btn-tempo').forEach(b => b.classList.remove('ativo'));
    elemento.classList.add('ativo');

    carregarDados();
}

function calcularCentro(velas) {
    let maxima = -Infinity;
    let minima = Infinity;
    velas.forEach(v => {
        let alta = parseFloat(v[2]);
        let baixa = parseFloat(v[3]);
        if (alta > maxima) maxima = alta;
        if (baixa < minima) minima = baixa;
    });
    return (maxima + minima) / 2;
}

function calcularMedia(velas, periodo) {
    if (velas.length < periodo) return null;
    let soma = 0;
    for (let i = velas.length - periodo; i < velas.length; i++) {
        soma += parseFloat(velas[i][4]);
    }
    return soma / periodo;
}

async function carregarDados() {
    try {
        let limiteVelas = (tempoAtualBinance === '1d') ? 100 : 1000;
        let res = await fetch(`https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=${tempoAtualBinance}&limit=${limiteVelas}`);
        dadosGlobais = await res.json();
        atualizarTela();
    } catch (e) {
        document.getElementById("status-sinal").innerText = "❌ ERRO AO CONECTAR NA BINANCE";
    }
}

function atualizarTela() {
    if (!dadosGlobais.length) return;

    let centro = calcularCentro(dadosGlobais);
    let precoAtual = parseFloat(dadosGlobais[dadosGlobais.length - 1][4]);

    let sinalTexto = "";
    let corSinal = "";

    if (precoAtual > centro) {
        sinalTexto = `🚀 [${tempoAtualBinance.toUpperCaseAsync ? '' : tempoAtualBinance.toUpperCase()}] PREÇO ACIMA DO CENTRO -> COMPRA FORTE!`;
        corSinal = "#0ecb81";
    } else {
        sinalTexto = `📉 [${tempoAtualBinance.toUpperCase()}] PREÇO ABAIXO DO CENTRO -> VENDA FORTE!`;
        corSinal = "#f6465d";
    }

    let caixaSinal = document.getElementById("status-sinal");
    caixaSinal.innerText = sinalTexto;
    caixaSinal.style.color = corSinal;
    caixaSinal.style.borderColor = corSinal;

    let mediasHtml = "";
    periodosMa.forEach(p => {
        let maVal = calcularMedia(dadosGlobais, p);
        if (maVal) {
            mediasHtml += `MA ${p}: <b>$ ${maVal.toFixed(2)}</b><br>`;
        } else {
            mediasHtml += `MA ${p}: <i>Dados insuficientes</i><br>`;
        }
    });

    document.getElementById("painel-detalhes").innerHTML = `
        <div class="linha-info">
            <span>Preço Atual:</span> <b>$ ${precoAtual.toFixed(2)}</b>
        </div>
        <div class="linha-info">
            <span>Centro Comprimido:</span> <b style="color: #0ecb81;">$ ${centro.toFixed(2)}</b>
        </div>
        <div class="linha-info">
            <span id="relogio-vela">Contagem Regressiva:</span> <b id="timer-txt" style="color: #f0b90b;">--:--</b>
        </div>
        <div class="medias-titulo">📈 MÉDIAS MÉSTRES NESTE TEMPO:</div>
        <div class="medias-grid">${mediasHtml}</div>
    `;
}

// Relógio dinâmico para qualquer tempo escolhido
function atualizarTimer() {
    const agora = new Date();
    let segundos = agora.getSeconds();
    let minutos = agora.getMinutes();
    let horas = agora.getHours();
    
    let restoSegundos = 59 - segundos;
    let sFormatado = restoSegundos < 10 ? "0" + restoSegundos : restoSegundos;

    let tempoTexto = "";
    if (tempoAtualBinance === '1m') {
        tempoTexto = `00:${sFormatado}`;
    } else if (tempoAtualBinance === '5m') {
        let restoMin = 4 - (minutos % 5);
        let mFormatado = restoMin < 10 ? "0" + restoMin : restoMin;
        tempoTexto = `${mFormatado}:${sFormatado}`;
    } else if (tempoAtualBinance === '15m') {
        let restoMin = 14 - (minutos % 15);
        let mFormatado = restoMin < 10 ? "0" + restoMin : restoMin;
        tempoTexto = `${mFormatado}:${sFormatado}`;
    } else if (tempoAtualBinance === '30m') {
        let restoMin = 29 - (minutos % 30);
        let mFormatado = restoMin < 10 ? "0" + restoMin : restoMin;
        tempoTexto = `${mFormatado}:${sFormatado}`;
    } else if (tempoAtualBinance === '1h') {
        let restoMin = 59 - minutos;
        let mFormatado = restoMin < 10 ? "0" + restoMin : restoMin;
        tempoTexto = `${mFormatado}:${sFormatado}`;
    } else if (tempoAtualBinance === '4h') {
        let restoHoras = 3 - (horas % 4);
        let restoMin = 59 - minutos;
        tempoTexto = `0${restoHoras}:${restoMin < 10 ? '0'+restoMin : restoMin}:${sFormatado}`;
    } else {
        tempoTexto = "Contagem Diária";
    }

    let elemTimer = document.getElementById("timer-txt");
    if (elemTimer) elemTimer.innerText = tempoTexto;
}

// Inicia
carregarDados();
setInterval(carregarDados, 10000); // Atualiza os dados a cada 10 segundos
setInterval(atualizarTimer, 1000);   // Roda o relógio a cada 1 segundo
</script>

</body>
</html>
