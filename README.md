<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>M.A.R.S.O.S. - Painel Tático de Combate 3D</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }
        body {
            background-color: #03030c;
            color: #ff3333;
            font-family: 'Courier New', Courier, monospace;
            overflow: hidden;
        }
        #canvas-container {
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        .hud {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none;
            padding: 20px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-shadow: inset 0 0 100px rgba(0, 240, 255, 0.1);
            border: 2px solid rgba(0, 240, 255, 0.3);
        }
        header {
            background: rgba(3, 3, 12, 0.9);
            border: 1px solid #00f0ff;
            padding: 15px;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.4);
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
        .combat-log {
            position: absolute;
            left: 20px;
            bottom: 40px;
            width: 450px;
            height: 250px;
            background: rgba(3, 3, 12, 0.95);
            border: 1px solid #ff3333;
            padding: 15px;
            font-size: 11px;
            color: #ff9999;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            box-shadow: 0 0 15px rgba(255, 0, 0, 0.2);
            pointer-events: auto;
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
            position: absolute;
            right: 20px;
            bottom: 40px;
            background: rgba(3, 3, 12, 0.9);
            border: 1px solid #00f0ff;
            color: #00f0ff;
            padding: 15px;
            font-size: 12px;
            text-align: right;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.2);
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateX(-10px); }
            to { opacity: 1; transform: translateX(0); }
        }
    </style>
    <!-- Importando biblioteca Three.js estável -->
    <script src="https://cloudflare.com"></script>
</head>
<body>

    <div id="canvas-container"></div>

    <div class="hud">
        <header>
            <h1>SISTEMA M.A.R.S.O.S - RENDERIZAÇÃO TÁTICA DA ARMADURA</h1>
            <div class="system-status">
                <div>M.A.R.S.O.S: <span class="status-online">CONECTADO</span></div>
                <div>SROS MESH: <span class="status-online">SÍNCRONO</span></div>
                <div>ESCUDO MHD: <span class="status-online">ATIVADO</span></div>
                <div>S.A.M.S: <span class="status-online">PRONTO</span></div>
                <div>G.FARADAY: <span class="status-online">BLINDADO</span></div>
            </div>
        </header>

        <div class="combat-log" id="log-box">
            <!-- Os eventos entrarão aqui via JS -->
        </div>

        <div class="controls-hint">
            <strong>STATUS DO TRAJE</strong><br>
            Núcleo de Fusão: Estável<br>
            Ambiente: Exposição a Extremos Cósmicos<br>
            Material: Deposição de Carbono Ativa
        </div>
    </div>

    <script>
        // Configuração Inicial do Ambiente 3D
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x020208); // Fundo escuro controlado

        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 100);
        camera.position.set(0, 0, 5); // Posiciona a câmera bem em frente ao modelo

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        container.appendChild(renderer.domElement);

        // Sistema de Iluminação (Crucial para dar volume aos objetos e torná-los visíveis)
        const light1 = new THREE.DirectionalLight(0xffffff, 2.5); // Luz branca forte de frente
        light1.position.set(1, 2, 4);
        scene.add(light1);

        const light2 = new THREE.DirectionalLight(0x00f0ff, 2.0); // Luz azul lateral futurista
        light2.position.set(-3, 1, 2);
        scene.add(light2);

        const light3 = new THREE.PointLight(0xff3333, 1.5, 10); // Luz vermelha vinda de baixo
        light3.position.set(0, -2, 1);
        scene.add(light3);

        // Grupo que vai conter todas as partes da armadura
        const armorGroup = new THREE.Group();

        // Materiais Sólidos Visíveis (Substituído o wireframe transparente por material metálico futurista)
        const materialPlacas = new THREE.MeshStandardMaterial({ 
            color: 0x112244, 
            roughness: 0.2, 
            metalness: 0.8 
        });
        
        const materialDetalhes = new THREE.MeshStandardMaterial({ 
            color: 0x00f0ff, 
            emissive: 0x005577,
            roughness: 0.1
        });

        const materialArticulacao = new THREE.MeshStandardMaterial({ 
            color: 0xffaa00, 
            metalness: 0.9 
        });

        // --- CONSTRUÇÃO DA ARMADURA SÓLIDA ---
        
        // 1. Capacete (Esfera sólida brilhante)
        const capacete = new THREE.Mesh(new THREE.SphereGeometry(0.35, 32, 32), materialDetalhes);
        capacete.position.y = 1.3;
        armorGroup.add(capacete);

        // Viseira do Capacete
        const visor = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 0.15, 16), materialPlacas);
        visor.rotation.x = Math.PI / 2;
        visor.position.set(0, 1.3, 0.2);
        armorGroup.add(visor);

        // 2. Torso / Peito (Cilindro robusto)
        const torso = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.35, 1.0, 16), materialPlacas);
        torso.position.y = 0.5;
        armorGroup.add(torso);

        // Núcleo MHD no Peito (Detalhe circular)
        const nucleoMHD = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 0.05, 16), materialDetalhes);
        nucleoMHD.rotation.x = Math.PI / 2;
        nucleoMHD.position.set(0, 0.7, 0.36);
        armorGroup.add(nucleoMHD);

        // 3. Ombros (Ombreiras S.A.M.S.)
        const ombroEsq = new THREE.Mesh(new THREE.SphereGeometry(0.18, 16, 16), materialArticulacao);
        ombroEsq.position.set(-0.65, 0.9, 0);
        const ombroDir = ombroEsq.clone();
        ombroDir.position.x = 0.65;
        armorGroup.add(ombroEsq, ombroDir);

        // 4. Braços
        const bracoEsq = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.08, 0.6, 16), materialPlacas);
        bracoEsq.position.set(-0.65, 0.5, 0);
        const bracoDir = bracoEsq.clone();
        bracoDir.position.x = 0.65;
        armorGroup.add(bracoEsq, bracoDir);

        // 5. Pernas Robustas
        const pernaEsq = new THREE.Mesh(new THREE.CylinderGeometry(0.14, 0.1, 0.9, 16), materialPlacas);
        pernaEsq.position.set(-0.25, -0.4, 0);
        const pernaDir = pernaEsq.clone();
        pernaDir.position.x = 0.25;
        armorGroup.add(pernaEsq, pernaDir);

        // Botas Propulsoras MHD
        const botaEsq = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.15, 0.3), materialArticulacao);
        botaEsq.position.set(-0.25, -0.9, 0.05);
        const botaDir = botaEsq.clone();
        botaDir.position.x = 0.25;
        armorGroup.add(botaEsq, botaDir);

        // Adiciona a armadura montada à cena
        scene.add(armorGroup);

        // --- ESCUDO ENERGÉTICO MHD (Esfera externa translúcida) ---
        const escudoGeo = new THREE.SphereGeometry(1.8, 32, 16);
        const escudoMat = new THREE.MeshBasicMaterial({
            color: 0x00f0ff,
            wireframe: true,
            transparent: true,
            opacity: 0.08
        });
        const campoEscudo = new THREE.Mesh(escudoGeo, escudoMat);
        scene.add(campoEscudo);

        // --- SISTEMA DE PARTÍCULAS (Chuvas Cinéticas / Meteoritos) ---
        const quantParticulas = 80;
        const particulasGeo = new THREE.BufferGeometry();
        const posicoes = new Float32Array(quantParticulas * 3);
        const velocidades = [];

        for(let i=0; i<quantParticulas; i++) {
            posicoes[i*3] = (Math.random() - 0.5) * 8;     // Eixo X aleatório
            posicoes[i*3+1] = (Math.random() - 0.5) * 6;   // Eixo Y aleatório
            posicoes[i*3+2] = Math.random() * 6 + 2;       // Distância Z (vêm de frente)
            velocidades.push(Math.random() * 0.04 + 0.02);
        }

        particulasGeo.setAttribute('position', new THREE.BufferAttribute(posicoes, 3));
        const particulaMat = new THREE.PointsMaterial({ color: 0xffaa00, size: 0.06 });
        const nuvemMeteoritos = new THREE.Points(particulasGeo, particulaMat);
        scene.add(nuvemMeteoritos);


        // --- LÓGICA DE EVENTOS DO HUD (Histórico Tático) ---
