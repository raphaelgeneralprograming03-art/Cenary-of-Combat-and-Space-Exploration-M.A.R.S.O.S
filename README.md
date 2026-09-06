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
            box-shadow: inset 0 0 100px rgba(0, 240, 255, 0.15);
            border: 2px solid rgba(0, 240, 255, 0.3);
        }
        header {
            background: rgba(2, 2, 8, 0.95);
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
            height: 230px;
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
            position: absolute;
            right: 20px;
            bottom: 40px;
            background: rgba(2, 2, 8, 0.95);
            border: 1px solid #00f0ff;
            color: #00f0ff;
            padding: 15px;
            font-size: 12px;
            text-align: right;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.3);
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateX(-10px); }
            to { opacity: 1; transform: translateX(0); }
        }
    </style>
    <!-- Importando Three.js via CDN confiável -->
    <script src="https://cloudflare.com"></script>
</head>
<body>

    <div id="canvas-container"></div>

    <div class="hud">
        <header>
            <h1>SISTEMA M.A.R.S.O.S - DIAGNÓSTICO DIGITAL ACTIVO</h1>
            <div class="system-status">
                <div>M.A.R.S.O.S: <span class="status-online">CONECTADO</span></div>
                <div>SROS MESH: <span class="status-online">SÍNCRONO</span></div>
                <div>ESCUDO MHD: <span class="status-online">ATIVADO</span></div>
                <div>S.A.M.S: <span class="status-online">PRONTO</span></div>
                <div>G.FARADAY: <span class="status-online">PROTEGIDO</span></div>
            </div>
        </header>

        <div class="combat-log" id="log-box"></div>

        <div class="controls-hint">
            <strong>MONITORIZAÇÃO DA ARMADURA</strong><br>
            Fusão MHD: Estável (Ciclo Fechado)<br>
            Ambiente Interno: Estabilizado a 22°C<br>
            Gêmeo Digital: Sincronização Ativa
        </div>
    </div>

    <script>
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x02020a); // Fundo espacial profundo

        // Câmera posicionada perfeitamente na frente
        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 100);
        camera.position.set(0, 0.4, 4.5);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        container.appendChild(renderer.domElement);

        // Iluminação Global redundante para garantir visibilidade
        const lightAmbient = new THREE.AmbientLight(0xffffff, 1.5);
        scene.add(lightAmbient);

        const lightDir = new THREE.DirectionalLight(0x00f0ff, 2);
        lightDir.position.set(2, 4, 3);
        scene.add(lightDir);

        const armorGroup = new THREE.Group();

        // MATERIAIS DE ALTA VISIBILIDADE (Garantem cor mesmo sem luzes configuradas)
        const matBlindagem = new THREE.MeshPhongMaterial({ 
            color: 0x0044aa, 
            emissive: 0x001133, // Brilho azul escuro de fundo
            specular: 0x00f0ff, 
            shininess: 30 
        });
        
        const matNucleoEReator = new THREE.MeshPhongMaterial({ 
            color: 0x00f0ff, 
            emissive: 0x00aaff, // Brilho neon ativo
            shininess: 100 
        });

        const matArticulacoes = new THREE.MeshPhongMaterial({ 
            color: 0xffaa00, 
            emissive: 0x442200, // Brilho dourado ativo
            shininess: 50 
        });

        // --- MODELAGEM DA ARMADURA EM VOLUMES SÓLIDOS BRILHANTES ---

        // 1. CAPACETE
        const helmet = new THREE.Mesh(new THREE.SphereGeometry(0.35, 32, 32), matBlindagem);
        helmet.position.y = 1.3;
        armorGroup.add(helmet);

        // Viseira brilhante
        const visor = new THREE.Mesh(new THREE.SphereGeometry(0.28, 16, 16), matNucleoEReator);
        visor.scale.set(1.1, 0.6, 1);
        visor.position.set(0, 1.35, 0.15);
        armorGroup.add(visor);

        // 2. TORSO / PEITO
        const torso = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.35, 1.0, 16), matBlindagem);
        torso.position.y = 0.5;
        armorGroup.add(torso);

        // Reator MHD Central no peito
        const mhdCore = new THREE.Mesh(new THREE.CylinderGeometry(0.14, 0.14, 0.08, 16), matNucleoEReator);
        mhdCore.rotation.x = Math.PI / 2;
        mhdCore.position.set(0, 0.65, 0.4);
        armorGroup.add(mhdCore);

        // 3. OMBROS E BRAÇOS
        const shoulderL = new THREE.Mesh(new THREE.SphereGeometry(0.18, 16, 16), matArticulacoes);
        shoulderL.position.set(-0.65, 0.85, 0);
        const shoulderR = shoulderL.clone();
        shoulderR.position.x = 0.65;
        armorGroup.add(shoulderL, shoulderR);

        const armL = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.08, 0.6, 16), matBlindagem);
        armL.position.set(-0.65, 0.45, 0);
        const armR = armL.clone();
        armR.position.x = 0.65;
        armorGroup.add(armL, armR);

        // 4. PERNAS E BOTAS PROPULSORAS
        const legL = new THREE.Mesh(new THREE.CylinderGeometry(0.13, 0.09, 0.9, 16), matBlindagem);
        legL.position.set(-0.25, -0.4, 0);
        const legR = legL.clone();
        legR.position.x = 0.25;
        armorGroup.add(legL, legR);

        const bootL = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.15, 0.3), matArticulacoes);
        bootL.position.set(-0.25, -0.9, 0.05);
        const bootR = bootL.clone();
        bootR.position.x = 0.25;
        armorGroup.add(bootL, bootR);

        scene.add(armorGroup);

        // --- ESCUDO ENERGÉTICO MHD EXTERNO ---
        const shieldGeo = new THREE.SphereGeometry(1.7, 32, 16);
        const shieldMat = new THREE.MeshBasicMaterial({
            color: 0x00f0ff,
            wireframe: true,
            transparent: true,
            opacity: 0.1
        });
        const shield = new THREE.Mesh(shieldGeo, shieldMat);
        scene.add(shield);

        // --- MICROMETEORITOS CINÉTICOS (Pontos brilhantes em movimento) ---
        const pCount = 60;
        const pGeo = new THREE.BufferGeometry();
        const pPos = new Float32Array(pCount * 3);
        const pSpeeds = [];

        for(let i=0; i<pCount; i++) {
            pPos[i*3] = (Math.random() - 0.5) * 6;
            pPos[i*3+1] = (Math.random() - 0.5) * 5;
            pPos[i*3+2] = Math.random() * 5 + 2;
            pSpeeds.push(Math.random() * 0.03 + 0.02);
        }

        pGeo.setAttribute('position', new THREE.BufferAttribute(pPos, 3));
        const pMat = new THREE.PointsMaterial({ color: 0xffcc00, size: 0.07 });
        const spaceDebris = new THREE.Points(pGeo, pMat);
        scene.add(spaceDebris);

        // --- FEED DE LOGS EM TEMPO REAL ---
        const logBox = document.getElementById('log-box');
        const logs = [
            "<span class='log-alert'>[SROS]</span> Radiação espacial extrema detectada. Temperatura exterior: -273°C.",
            "<span class='log-system'>[MHD]</span> Campo de plasma térmico ativado. Clima interno: 22°C.",
            "<span class='log-alert'>[SROS]</span> Pulso EMP inimigo direcionado neutralizado pela Gaiola de Faraday.",
            "<span class='log-system'>[S.A.M.S]</span> Impactos cinéticos de detritos espaciais absorvidos sem fadiga corporal.",
            "<span class='log-system'>[M.A.R.S.O.S]</span> Fissura detectada. Injeção de CO2 e Carbono ativa.",
            "<span class='log-system'>[IA]</span> Gêmeo Digital concluiu o realinhamento estético da armadura.",
