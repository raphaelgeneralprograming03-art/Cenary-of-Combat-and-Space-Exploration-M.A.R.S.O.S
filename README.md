<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>M.A.R.S.O.S. - Simulação de Combate Extremo</title>
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
            box-shadow: inset 0 0 100px rgba(255, 0, 0, 0.2);
            border: 2px solid rgba(255, 51, 51, 0.3);
        }
        header {
            background: rgba(3, 3, 12, 0.85);
            border: 1px solid #ff3333;
            padding: 15px;
            box-shadow: 0 0 15px rgba(255, 51, 51, 0.4);
        }
        header h1 {
            font-size: 20px;
            text-shadow: 0 0 10px #ff3333;
        }
        .system-status {
            display: flex;
            gap: 20px;
            margin-top: 5px;
            font-size: 12px;
            color: #00f0ff;
        }
        .status-online { color: #00ff00; font-weight: bold; }
        .combat-log {
            position: absolute;
            left: 20px;
            bottom: 40px;
            width: 450px;
            height: 250px;
            background: rgba(3, 3, 12, 0.9);
            border: 1px solid #ff3333;
            padding: 15px;
            font-size: 11px;
            color: #ff9999;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            box-shadow: 0 0 15px rgba(255, 0, 0, 0.2);
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
            background: rgba(3, 3, 12, 0.8);
            border: 1px solid #00f0ff;
            color: #00f0ff;
            padding: 10px;
            font-size: 12px;
            text-align: right;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateX(-10px); }
            to { opacity: 1; transform: translateX(0); }
        }
    </style>
    <script src="https://cloudflare.com"></script>
</head>
<body>

    <div id="canvas-container"></div>

    <div class="hud">
        <header>
            <h1>ALERTA DE COMBATE E EXPLORAÇÃO EXTREMA</h1>
            <div class="system-status">
                <div>M.A.R.S.O.S: <span class="status-online">ATIVO</span></div>
                <div>SROS MESH: <span class="status-online">SÍNCRONO</span></div>
                <div>ESCUDO MHD: <span class="status-online">100%</span></div>
                <div>S.A.M.S: <span class="status-online">PRONTO</span></div>
                <div>G.FARADAY: <span class="status-online">ISOLADO</span></div>
            </div>
        </header>

        <div class="combat-log" id="log-box">
            <!-- Os eventos em tempo real entrarão aqui via JS -->
        </div>

        <div class="controls-hint">
            AMBIENTE: -273°C (Vácuo / Radiação Solar Classe X)<br>
            SIMULAÇÃO DE IMPACTO CINETÍCO & EMP ATIVA
        </div>
    </div>

    <script>
        // Configuração de Cena do Three.js
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x03030c, 0.08);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 1, 4);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setClearColor(scene.fog.color);
        container.appendChild(renderer.domElement);

        // Luzes Ambientais e Alertas Vermelhos
        const ambientLight = new THREE.AmbientLight(0x330000, 1.5);
        scene.add(ambientLight);

        const tacticalLight = new THREE.PointLight(0xff3333, 3, 20);
        tacticalLight.position.set(0, 2, 2);
        scene.add(tacticalLight);

        const shieldLight = new THREE.PointLight(0x00f0ff, 2, 10);
        shieldLight.position.set(0, 0, 0);
        scene.add(shieldLight);

        // Grupo Principal (Armadura)
        const armorGroup = new THREE.Group();
        const matArmadura = new THREE.MeshPhongMaterial({ color: 0xff3333, wireframe: true });

        // Geometria básica da armadura para representação
        const cap = new THREE.Mesh(new THREE.SphereGeometry(0.35, 12, 12), matArmadura); cap.position.y = 1.4;
        const tor = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.3, 1.0, 8), matArmadura); tor.position.y = 0.6;
        const p1 = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.08, 1.0, 6), matArmadura); p1.position.set(-0.25, -0.4, 0);
        const p2 = p1.clone(); p2.position.x = 0.25;
        armorGroup.add(cap, tor, p1, p2);
        scene.add(armorGroup);

        // Criando a Esfera de Escudo MHD (Transparente / Holográfica)
        const shieldGeo = new THREE.SphereGeometry(1.8, 32, 32);
        const shieldMat = new THREE.MeshBasicMaterial({
            color: 0x00f0ff,
            wireframe: true,
            transparent: true,
            opacity: 0.15
        });
        const shieldMesh = new THREE.Mesh(shieldGeo, shieldMat);
        scene.add(shieldMesh);

        // Sistema de Partículas (Micrometeoritos Cósmicos)
        const particleCount = 60;
        const particlesGeo = new THREE.BufferGeometry();
        const positions = new Float32Array(particleCount * 3);
        const speeds = [];

        for(let i=0; i<particleCount; i++) {
            // Nascem longe e viajam em direção à armadura
            positions[i*3] = (Math.random() - 0.5) * 10;
            positions[i*3+1] = (Math.random() - 0.5) * 10;
            positions[i*3+2] = Math.random() * 8 + 4; // Vêm de frente
            speeds.push(Math.random() * 0.1 + 0.05);
        }

        particlesGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
        const pMaterial = new THREE.PointsMaterial({ color: 0xffff00, size: 0.08 });
        const particleSystem = new THREE.Points(particlesGeo, pMaterial);
        scene.add(particleSystem);

        // Lógica dos Logs Táticos no HUD
        const logBox = document.getElementById('log-box');
        const eventos = [
            "<span class='log-alert'>[ALERTA]</span> Tempestade solar detectada. Radiação externa: Crítica.",
            "<span class='log-system'>[MHD]</span> Campo deflector ativado. Absorção térmica em -273°C estabilizada.",
            "<span class='log-alert'>[SROS]</span> Varredura detecta pulso EMP direcionado inimigo na velocidade da luz.",
            "<span class='log-system'>[FARADAY]</span> Malha passiva absorveu a indução. Eletrônica blindada.",
            "<span class='log-system'>[S.A.M.S]</span> Impacto de micro-rochas a 20 km/s repelido por amortecedores pneumáticos.",
            "<span class='log-alert'>[AVISO]</span> Fissura menor detectada na placa torácica esquerda.",
            "<span class='log-system'>[M.A.R.S.O.S]</span> Células de Carbono injetando CO2 da respiração. Selamento em 1.4s.",
            "<span class='log-system'>[IA]</span> Gêmeo Digital carregado. Design da armadura remoldado ao estado original.",
            "<span class='log-system'>[BIOLÓGICO]</span> Cortisol elevado. Injeção subdérmica de dopamina efetuada.",
            "<span class='log-system'>[PROPULSÃO]</span> Reabastecimento por ciclo fechado. Captura de He atômico: 100%."
        ];
        
        let logIndex = 0;
        function addLog() {
            if (logIndex >= eventos.length) logIndex = 0;
            const entry = document.createElement('div');
            entry.className = 'log-entry';
            entry.innerHTML = `[${new Date().toLocaleTimeString()}] ${eventos[logIndex]}`;
            logBox.appendChild(entry);
            logIndex++;

            if (logBox.children.length > 6) {
                logBox.removeChild(logBox.children[0]);
            }
            setTimeout(addLog, 2500);
        }
        addLog();

        // Loop de Animação e Renderização
        const clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);
            const time = clock.getElapsedTime();

            // Rotação da Armadura
            armorGroup.rotation.y = time * 0.5;

            // Pulsação do Escudo MHD ao sofrer impactos simulados
            shieldMesh.rotation.x = time * 0.1;
            shieldMesh.rotation.y = time * 0.1;
            shieldMat.opacity = 0.1 + Math.abs(Math.sin(time * 2)) * 0.15;

            // Movimentação dos Micrometeoritos
            const coords = particleSystem.geometry.attributes.position.array;
            for(let i=0; i<particleCount; i++) {
                coords[i*3+2] -= speeds[i]; // Move no eixo Z (em direção à tela)

                // Se colidir com o raio do escudo ou passar do jogador, reseta
                if(coords[i*3+2] < 0.1) {
