<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>M.A.R.S.O.S. - Painel Tático Absoluto</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }
        body {
            background-color: #020208;
            color: #00f0ff;
            font-family: 'Courier New', Courier, monospace;
            overflow: hidden;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            border: 2px solid rgba(0, 240, 255, 0.3);
            box-shadow: inset 0 0 100px rgba(0, 240, 255, 0.15);
        }
        header {
            background: rgba(2, 2, 8, 0.95);
            border: 1px solid #00f0ff;
            padding: 15px;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.4);
            z-index: 10;
        }
        header h1 {
            font-size: 20px;
            color: #00f0ff;
            text-shadow: 0 0 10px #00f0ff;
        }
        .system-status {
            display: flex;
            gap: 20px;
            margin-top: 5px;
            font-size: 12px;
            color: #ffffff;
        }
        .status-online { color: #00ff00; font-weight: bold; }
        
        /* Área Central do Painel Visual */
        .viewport-central {
            flex-grow: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            margin: 20px 0;
        }

        /* Efeito de Escudo Energético MHD Pulsante */
        .escudo-mhd {
            width: 420px;
            height: 520px;
            border: 2px dashed rgba(0, 240, 255, 0.4);
            border-radius: 50% / 40%;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 40px rgba(0, 240, 255, 0.1), inset 0 0 30px rgba(0, 240, 255, 0.05);
            animation: pulsarEscudo 3s infinite ease-in-out;
        }

        /* Armadura Gráfica em Alta Definição (SVG Vector) */
        .armadura-svg {
            width: 300px;
            height: 450px;
            filter: drop-shadow(0 0 15px rgba(0, 240, 255, 0.6));
        }

        .paineis-inferiores {
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            z-index: 10;
        }

        .combat-log {
            width: 450px;
            height: 180px;
            background: rgba(2, 2, 8, 0.95);
            border: 1px solid #ff3333;
            padding: 15px;
            font-size: 11px;
            color: #ff9999;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            box-shadow: 0 0 15px rgba(255, 0, 0, 0.3);
        }
        .log-entry {
            margin-top: 5px;
            line-height: 1.4;
            border-left: 2px solid #ff3333;
            padding-left: 6px;
            animation: fadeIn 0.3s ease-out;
        }
        .log-system { color: #00f0ff; }
        .log-alert { color: #ffff00; font-weight: bold; }
        
        .controls-hint {
            background: rgba(2, 2, 8, 0.95);
            border: 1px solid #00f0ff;
            color: #00f0ff;
            padding: 15px;
            font-size: 12px;
            text-align: right;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
            width: 300px;
        }

        @keyframes pulsarEscudo {
            0% { transform: scale(1); opacity: 0.7; box-shadow: 0 0 40px rgba(0, 240, 255, 0.2); }
            50% { transform: scale(1.03); opacity: 1; box-shadow: 0 0 60px rgba(0, 240, 255, 0.4); }
            100% { transform: scale(1); opacity: 0.7; box-shadow: 0 0 40px rgba(0, 240, 255, 0.2); }
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateX(-10px); }
            to { opacity: 1; transform: translateX(0); }
        }
    </style>
</head>
<body>

    <header>
        <h1>SISTEMA M.A.R.S.O.S - MAPEAMENTO DA ARQUITETURA TÁTICA</h1>
        <div class="system-status">
            <div>M.A.R.S.O.S: <span class="status-online">CONECTADO</span></div>
            <div>SROS MESH: <span class="status-online">SÍNCRONO</span></div>
            <div>ESCUDO MHD: <span class="status-online">ATIVADO</span></div>
            <div>S.A.M.S: <span class="status-online">PRONTO</span></div>
            <div>G.FARADAY: <span class="status-online">PROTEGIDO</span></div>
        </div>
    </header>

    <!-- CONTAINER VISUAL DA ARMADURA COM ESCUDO ATIVO -->
    <div class="viewport-central">
        <div class="escudo-mhd">
            
            <!-- Desenho Direto da Armadura via Vetores SVG Sólidos -->
            <svg class="armadura-svg" viewBox="0 0 200 300" xmlns="http://w3.org">
                <!-- Definições de Cores Metálicas e Efeitos Neon -->
                <defs>
                    <linearGradient id="blindagem" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" stop-color="#0033aa" />
                        <stop offset="100%" stop-color="#001144" />
                    </linearGradient>
                    <linearGradient id="articulacao" x1="0%" y1="0%" x2="100%" y2="0%">
                        <stop offset="0%" stop-color="#ffaa00" />
                        <stop offset="100%" stop-color="#885500" />
                    </linearGradient>
                </defs>

                <!-- Pernas e Coxas -->
                <rect x="65" y="180" width="25" height="80" rx="5" fill="url(#blindagem)" stroke="#00f0ff" stroke-width="1.5"/>
                <rect x="110" y="180" width="25" height="80" rx="5" fill="url(#blindagem)" stroke="#00f0ff" stroke-width="1.5"/>
                
                <!-- Botas Propulsoras MHD -->
                <rect x="60" y="260" width="32" height="20" rx="4" fill="url(#articulacao)" stroke="#fff" stroke-width="1"/>
                <rect x="108" y="260" width="32" height="20" rx="4" fill="url(#articulacao)" stroke="#fff" stroke-width="1"/>

                <!-- Torso / Placa Peitoral -->
                <path d="M 50 80 L 150 80 L 135 180 L 65 180 Z" fill="url(#blindagem)" stroke="#00f0ff" stroke-width="2"/>
                
                <!-- Reator de Fusão MHD Central (Peito) -->
                <circle cx="100" cy="120" r="18" fill="#00f0ff" filter="drop-shadow(0 0 8px #00f0ff)" stroke="#fff" stroke-width="1.5"/>
                <circle cx="100" cy="120" r="8" fill="#ffffff" />

                <!-- Ombreiras de Amortecimento S.A.M.S -->
                <circle cx="42" cy="90" r="15" fill="url(#articulacao)" stroke="#00f0ff" stroke-width="1"/>
                <circle cx="158" cy="90" r="15" fill="url(#articulacao)" stroke="#00f0ff" stroke-width="1"/>

                <!-- Braços Direito e Esquerdo -->
                <rect x="30" y="105" width="20" height="65" rx="4" fill="url(#blindagem)" stroke="#00f0ff" stroke-width="1.5"/>
                <rect x="150" y="105" width="20" height="65" rx="4" fill="url(#blindagem)" stroke="#00f0ff" stroke-width="1.5"/>

                <!-- Capacete e Cúpula de Comando -->
                <circle cx="100" cy="45" r="25" fill="url(#blindagem)" stroke="#00f0ff" stroke-width="2"/>
                <!-- Viseira com HUD Holográfico -->
                <ellipse cx="100" cy="45" rx="18" ry="10" fill="#00f0ff" opacity="0.8" stroke="#fff" stroke-width="1"/>
            </svg>

        </div>
    </div>

    <div class="paineis-inferiores">
        <div class="combat-log" id="log-box"></div>

        <div class="controls-hint">
            <strong>MONITORIZAÇÃO DA ARMADURA</strong><br>
            Fusão MHD: Estável (Ciclo Fechado)<br>
            Ambiente Interno: Estabilizado a 22°C<br>
            Gêmeo Digital: Sincronização Ativa
        </div>
    </div>

    <script>
        // --- LOGS DE EVENTOS DO CENÁRIO DE COMBATE ---
        const logBox = document.getElementById('log-box');
        const logs = [
            "<span class='log-alert'>[SROS]</span> Radiação espacial extrema detectada. Temperatura exterior: -273°C.",
            "<span class='log-system'>[MHD]</span> Campo de plasma térmico ativado. Clima interno: 22°C.",
            "<span class='log-alert'>[SROS]</span> Pulso EMP inimigo direcionado neutralizado pela Gaiola de Faraday.",
            "<span class='log-system'>[S.A.M.S]</span> Impactos cinéticos de detritos espaciais absorvidos sem fadiga corporal.",
            "<span class='log-system'>[M.A.R.S.O.S]</span> Fissura detectada. Injeção de CO2 e Carbono ativa.",
            "<span class='log-system'>[IA]</span> Gêmeo Digital concluiu o realinhamento estético da armadura.",
            "<span class='log-system'>[BIOLÓGICO]</span> Níveis de cortisol regulados. Estresse mental anulado.",
            "<span class='log-system'>[PROPULSÃO]</span> Ciclo fechado ativo. Recombustão de Hélio/Hidrogênio a 100%."
        ];
        
        let currentLog = 0;
        function updateHUD() {
            const div = document.createElement('div');
            div.className = 'log-entry';
            div.innerHTML = `[${new Date().toLocaleTimeString()}] ${logs[currentLog]}`;
            logBox.appendChild(div);
            currentLog = (currentLog + 1) % logs.length;

            if (logBox.children.length > 5) logBox.removeChild(logBox.firstChild);
            setTimeout(updateHUD, 2800);
        }
        updateHUD();
    </script>
</body>
</html>
