
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Systems Under Attack: Nuclear Bunker Simulation</title>
    <style>
        :root {
            --hud-green: #39ff14;
            --hud-red: #ff3333;
            --hud-blue: #00e5ff;
            --panel-bg: rgba(5, 12, 5, 0.95);
            --border-style: 1px solid rgba(57, 255, 20, 0.3);
        }

        body {
            margin: 0;
            padding: 10px;
            background-color: #010401;
            color: var(--hud-green);
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow: hidden;
            user-select: none;
        }

        .header-system {
            width: 100%;
            max-width: 1200px;
            display: flex;
            justify-content: space-between;
            border-bottom: var(--border-style);
            padding-bottom: 5px;
            margin-bottom: 10px;
            font-size: 11px;
            letter-spacing: 1px;
        }

        .main-frame {
            display: flex;
            gap: 15px;
            max-width: 1240px;
            width: 100%;
            justify-content: center;
        }

        .screen-container {
            position: relative;
            border: 2px solid var(--hud-green);
            box-shadow: 0 0 25px rgba(57, 255, 20, 0.15);
            border-radius: 4px;
        }

        canvas {
            display: block;
            background-color: #000;
        }

        .side-panel {
            background: var(--panel-bg);
            border: 1px solid var(--hud-green);
            border-radius: 4px;
            padding: 15px;
            width: 320px;
            box-sizing: border-box;
            display: flex;
            flex-direction: column;
        }

        h2 {
            font-size: 13px;
            margin: 12px 0 6px 0;
            text-transform: uppercase;
            border-bottom: var(--border-style);
            padding-bottom: 4px;
            color: #ffffff;
        }

        h2:first-of-type { margin-top: 0; }

        .telemetry-row {
            display: flex;
            justify-content: space-between;
            font-size: 11px;
            margin: 4px 0;
        }

        .status-ok { color: var(--hud-green); font-weight: bold; }
        .status-blue { color: var(--hud-blue); font-weight: bold; }
        .status-red { color: var(--hud-red); font-weight: bold; }

        .intel-box {
            margin-top: 10px;
            background: rgba(0, 0, 0, 0.6);
            border: 1px dashed rgba(57, 255, 20, 0.2);
            padding: 8px;
            font-size: 10px;
            height: 90px;
            overflow-y: auto;
            line-height: 1.4;
            color: #a0cca0;
        }

        .btn-group {
            display: flex;
            gap: 8px;
            margin-top: auto;
        }

        button {
            flex: 1;
            background: #2b0000;
            border: 1px solid var(--hud-red);
            color: var(--hud-red);
            padding: 10px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
            letter-spacing: 1px;
            transition: all 0.2s;
            font-family: inherit;
            font-size: 11px;
        }

        button:hover {
            background: var(--hud-red);
            color: #000;
            box-shadow: 0 0 15px var(--hud-red);
        }

        #reset-btn {
            background: #002b2b;
            border-color: var(--hud-blue);
            color: var(--hud-blue);
        }

        #reset-btn:hover {
            background: var(--hud-blue);
            color: #000;
            box-shadow: 0 0 15px var(--hud-blue);
        }
    </style>
</head>
<body>

    <div class="header-system">
        <div>CORE LINK: BUNKER_FORTRESS_MATRIX // SYSTEM ONLINE</div>
        <div>SROS ACQUISITION: ACTIVE</div>
    </div>

    <div class="main-frame">
        <div class="screen-container">
            <canvas id="tacticalCanvas" width="850" height="550"></canvas>
        </div>

        <div class="side-panel">
            <h2>STRATEGIC BUNKER OVERVIEW</h2>
            <div class="telemetry-row"><span>ESTRUTURA FISICA:</span><span id="txt-struct" class="status-ok">100%</span></div>
            <div class="telemetry-row"><span>PRESSÃO INTERNA:</span><span id="txt-press">1.00 atm</span></div>
            <div class="telemetry-row"><span>ACELEROMETRO SÍSMICO:</span><span id="txt-quake">0.00 G</span></div>

            <h2>MHD PROTECTION SYSTEM</h2>
            <div class="telemetry-row"><span>ANÉIS DE DISSIPIÇÃO:</span><span class="status-blue" id="txt-mhd-status">STANDBY</span></div>
            <div class="telemetry-row"><span>ABSORÇÃO ATIVA:</span><span style="color: var(--hud-blue);" id="txt-mhd-load">0.00 MW</span></div>

            <h2>S.A.M.S AIR CELL</h2>
            <div class="telemetry-row"><span>SILO DEFENSIVO:</span><span class="status-ok" id="txt-silo">ARMADO</span></div>
            <div class="telemetry-row"><span>STATUS DO CÉU:</span><span style="color: #fff;" id="txt-sky">LIMPO</span></div>

            <h2>SROS FEEDER LINK</h2>
            <div class="telemetry-row"><span>VARREDURA GEOLÓGICA:</span><span class="status-ok">COMPLETA</span></div>
            <div class="telemetry-row"><span>SONAR SNR / HYDRO:</span><span class="status-ok" id="txt-sonar">45.2 dB</span></div>
            
            <div class="intel-box" id="intel-logs">
                [SYSTEM] BUNKER SUBTERRÂNEO CONECTADO.<br>
                [MHD] AMORTECEDORES ELETROMAGNETICOS ONLINE.<br>
                [SROS] MONITORANDO ASSINATURA DE ENERGIA EXTERNA.<br>
            </div>

            <div class="btn-group">
                <button id="detonate-btn" onclick="iniciarDetonacao()">DETONAR ICBM</button>
                <button id="reset-btn" onclick="reiniciarSimulacao()">RESET</button>
            </div>
        </div>
    </div>

    <script>
        // =========================================================================
        // 1. NÚCLEO FÍSICO E MATEMÁTICO (MathPhysicsCore)
        // =========================================================================
        class MathPhysicsCore {
            static calcularEquacaoSonar(sl = 120, profundidadeM = 100, mhdAtivo = false) {
                const tl = 30 + (profundidadeM * 0.1);
                const nl = mhdAtivo ? 75 : 45;
                const di = 15;
                const snr = sl - tl - (nl - di);
                
                let status = 'RUIDOSO';
                if (snr > 40) status = 'EXCELENTE';
                else if (snr > 10) status = 'BOM';

                return {
                    snr: parseFloat(snr.toFixed(2)),
                    tl: parseFloat(tl.toFixed(2)),
                    nl,
                    status
                };
            }

            static calcularForcaLorentz(correnteJ, teslaB = 1.5, volumeV = 1.5) {
                return parseFloat((correnteJ * teslaB * volumeV).toFixed(2));
            }

            static calcularTemperaturaHipersonica(mach, tempAmbienteK = 288.15) {
                return parseFloat((tempAmbienteK * (1 + 0.2 * Math.pow(mach, 2))).toFixed(2));
            }

            static calcularPressaoHidrostatica(profundidadeM) {
                const p0 = 0.101325; // 1 atm em MPa
                const rho = 1025;
                const g = 9.81;
                return parseFloat((p0 + (rho * g * profundidadeM) / 1000000).toFixed(3));
            }

            static calcularAceleracaoSismica(tempoSeg, amplitudeA0 = 14.2, zeta = 0.15) {
                if (tempoSeg <= 0) return { gForca: 0, fatorAtenuacao: 1 };
                const omegaN = 45;
                const atenuacao = Math.max(0, Math.exp(-zeta * tempoSeg));
                const respostaOnda = Math.sin(omegaN * tempoSeg);
                const gForca = Math.abs(amplitudeA0 * atenuacao * respostaOnda);

                return {
                    gForca: parseFloat(gForca.toFixed(2)),
                    fatorAtenuacao: parseFloat(atenuacao.toFixed(4))
                };
            }
        }

        // =========================================================================
        // 2. CONFIGURAÇÃO DE CANVAS E TEXTURAS GEOLÓGICAS
        // =========================================================================
        const canvas = document.getElementById('tacticalCanvas');
        const ctx = canvas.getContext('2d');

        // Textura procedural da crosta terrestre
        const soloBackground = document.createElement('canvas');
        soloBackground.width = 850; 
        soloBackground.height = 390;
        const sCtx = soloBackground.getContext('2d');
        
        let gradGeologico = sCtx.createLinearGradient(0, 0, 0, 390);
        gradGeologico.addColorStop(0.0, '#a18b68'); // Solo superficial
        gradGeologico.addColorStop(0.2, '#6e5d44'); // Sedimentar
        gradGeologico.addColorStop(0.6, '#423829'); // Rocha matriz
        gradGeologico.addColorStop(1.0, '#1c1710'); // Crosta profunda
        sCtx.fillStyle = gradGeologico; 
        sCtx.fillRect(0, 0, 850, 390);

        for(let i = 0; i < 45000; i++) {
            sCtx.fillStyle = Math.random() > 0.5 ? 'rgba(0,0,0,0.15)' : 'rgba(255,255,255,0.05)';
            sCtx.fillRect(Math.random() * 850, Math.random() * 390, 1.5, 1.5);
        }

        // =========================================================================
        // 3. ESTADO DA SIMULAÇÃO E ENTIDADES
        // =========================================================================
        let nuclearAtivo = false;
        let cronometroNuke = 0;
        let raioExplosao = 0;
        let tremorX = 0;
        let tremorY = 0;
        let integridadeEstrutura = 100;
        let icbmY = -50;
        let icbmX = 425;
        let icbmVelocidade = 4.5;
        let icbmImpactado = false;
        let faiscasMhd = [];
        let projetilSam = null;

        function adicionarLog(texto) {
            const box = document.getElementById('intel-logs');
            box.innerHTML += texto + "<br>";
            box.scrollTop = box.scrollHeight;
        }

        function iniciarDetonacao() {
            if (nuclearAtivo) return;
            nuclearAtivo = true;
            document.getElementById('detonate-btn').disabled = true;
            document.getElementById('detonate-btn').style.opacity = '0.4';
            
            adicionarLog("[SROS_ALERT] ICBM HIPERSÔNICO DETECTADO EM ROTA.");
            adicionarLog("[SAMS] DISPARANDO DEFESA ANTIMÍSSIL DE INTERCEPTAÇÃO.");
            
            document.getElementById('txt-sky').innerText = "AMEAÇA BALÍSTICA";
            document.getElementById('txt-sky').style.color = '#ff3333';

            // Lança míssil interceptador S.A.M.S
            projetilSam = {
                x: 425,
                y: 350,
                vy: -6.5,
                ativo: true
            };
            document.getElementById('txt-silo').innerText = "DISPARADO";
            document.getElementById('txt-silo').className = "status-blue";
        }

        function reiniciarSimulacao() {
            nuclearAtivo = false;
            cronometroNuke = 0;
            raioExplosao = 0;
            tremorX = 0;
            tremorY = 0;
            integridadeEstrutura = 100;
            icbmY = -50;
            icbmImpactado = false;
            faiscasMhd = [];
            projetilSam = null;

            document.getElementById('detonate-btn').disabled = false;
            document.getElementById('detonate-btn').style.opacity = '1.0';
            
            document.getElementById('txt-struct').innerText = '100%';
            document.getElementById('txt-struct').className = 'status-ok';
            document.getElementById('txt-press').innerText = '1.00 atm';
            document.getElementById('txt-quake').innerText = '0.00 G';
            document.getElementById('txt-mhd-status').innerText = 'STANDBY';
            document.getElementById('txt-mhd-status').className = 'status-blue';
            document.getElementById('txt-mhd-load').innerText = '0.00 MW';
            document.getElementById('txt-sky').innerText = 'LIMPO';
            document.getElementById('txt-sky').style.color = '#fff';
            document.getElementById('txt-silo').innerText = 'ARMADO';
            document.getElementById('txt-silo').className = 'status-ok';

            document.getElementById('intel-logs').innerHTML = 
                '[SYSTEM] REARME CONCLUÍDO. SISTEMA PRONTO.<br>' +
                '[MHD] AMORTECEDORES ELETROMAGNETICOS ONLINE.<br>';
        }

        // =========================================================================
        // 4. CICLO PRINCIPAL DE ATUALIZAÇÃO E RENDERIZAÇÃO (Canvas 60FPS)
        // =========================================================================
        function updateAndRender() {
            // --- CÁLCULOS DE FÍSICA ---
            if (nuclearAtivo) {
                cronometroNuke += 0.016;

                // Descente do ICBM até a superfície (y = 160)
                if (!icbmImpactado) {
                    icbmY += icbmVelocidade;
                    
                    // Movimento do míssil SAM
                    if (projetilSam && projetilSam.ativo) {
                        projetilSam.y += projetilSam.vy;
                        if (projetilSam.y <= icbmY) {
                            // Interceptação na alta atmosfera
                            adicionarLog("[SAMS] INTERCEPTAÇÃO PARCIAL NA SUPERFÍCIE!");
                        }
                    }

                    if (icbmY >= 160) {
                        icbmImpactado = true;
                        adicionarLog("[CRITICAL] IMPACTO EM GROUND ZERO! DETONAÇÃO ATÔMICA.");
                        document.getElementById('txt-sky').innerText = "RADIAÇÃO CRÍTICA";
                    }
                } else {
                    // Expansão do plasma atômico
                    if (raioExplosao < 380) {
                        raioExplosao += 3.5;
                    }

                    // Cálculo da onda sísmica subterrânea usando MathPhysicsCore
                    const dadosSismicos = MathPhysicsCore.calcularAceleracaoSismica(cronometroNuke - 0.5);
                    const atenuacao = dadosSismicos.fatorAtenuacao;

                    tremorX = (Math.sin(cronometroNuke * 45) * 6 * atenuacao);
                    tremorY = (Math.cos(cronometroNuke * 45) * 6 * atenuacao);

                    if (integridadeEstrutura > 68) {
                        integridadeEstrutura -= 0.12;
                        if (integridadeEstrutura < 68) integridadeEstrutura = 68;
                    }

                    // Atualização da Telemetria no DOM
                    document.getElementById('txt-struct').innerText = `${Math.floor(integridadeEstrutura)}%`;
                    document.getElementById('txt-quake').innerText = `${dadosSismicos.gForca} G`;
                    document.getElementById('txt-press').innerText = `${(1.00 + (cronometroNuke * 0.45)).toFixed(2)} atm`;
                    
                    const cargaMhd = (atenuacao * 450.8).toFixed(1);
                    document.getElementById('txt-mhd-load').innerText = `${cargaMhd} MW`;
                    document.getElementById('txt-mhd-status').innerText = "DISSIPANDO IMPACTO";
                    
                    if (integridadeEstrutura < 85) {
                        document.getElementById('txt-struct').className = 'status-red';
                    }

                    // Gera faíscas de alta tensão no escudo MHD
                    if (Math.random() < 0.4) {
                        faiscasMhd.push({
                            ang: Math.PI + (Math.random() * Math.PI),
                            raio: 95 + Math.random() * 10,
                            vida: 1.0
                        });
                    }
                }
            }

            // Atualização dinâmica do Sonar
            const dadosSonar = MathPhysicsCore.calcularEquacaoSonar(120, 180, icbmImpactado);
            document.getElementById('txt-sonar').innerText = `${dadosSonar.snr} dB`;

            // --- DESENHO NO CANVAS ---
            ctx.save();
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Aplica o tremor sísmico da tela
            ctx.translate(tremorX, tremorY);

            // 1. Céu Atmosférico
            let gradCeu = ctx.createLinearGradient(0, 0, 0, 160);
            if (icbmImpactado) {
                gradCeu.addColorStop(0, '#5a0000');
                gradCeu.addColorStop(1, '#ff3300');
            } else {
                gradCeu.addColorStop(0, '#020b14');
                gradCeu.addColorStop(1, '#102536');
            }
            ctx.fillStyle = gradCeu;
            ctx.fillRect(0, 0, 850, 160);

            // Grid tático no céu
            ctx.strokeStyle = 'rgba(0, 229, 255, 0.08)';
            ctx.lineWidth = 1;
            for(let x=0; x<850; x+=40) {
                ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, 160); ctx.stroke();
            }

            // 2. Crosta Terrestre (Camadas Geológicas)
            ctx.drawImage(soloBackground, 0, 160);

            // Linha de Superfície Ground Zero
            ctx.strokeStyle = '#39ff14';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(0, 160);
            ctx.lineTo(850, 160);
            ctx.stroke();

            // 3. Renderização do Bunker Subterrâneo (x=425, y=380)
            const bunkerX = 425;
            const bunkerY = 380;

            // Escudo MHD Eletromagnético (Domo Eletromagnético)
            ctx.save();
            ctx.beginPath();
            ctx.arc(bunkerX, bunkerY + 10, 100, Math.PI, 0);
            ctx.strokeStyle = icbmImpactado ? '#00e5ff' : 'rgba(0, 229, 255, 0.3)';
            ctx.lineWidth = icbmImpactado ? 4 : 2;
            if (icbmImpactado) {
                ctx.shadowColor = '#00e5ff';
                ctx.shadowBlur = 20;
            }
            ctx.stroke();
            ctx.restore();

            // Desenhando faíscas no escudo MHD
            faiscasMhd.forEach((f, idx) => {
                let fx = bunkerX + Math.cos(f.ang) * f.raio;
                let fy = (bunkerY + 10) + Math.sin(f.ang) * f.raio;
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(fx, fy, 3, 3);
                f.vida -= 0.05;
            });
            faiscasMhd = faiscasMhd.filter(f => f.vida > 0);

            // Estrutura de Concreto Armado do Bunker
            ctx.fillStyle = '#22252a';
            ctx.strokeStyle = '#555e6b';
            ctx.lineWidth = 3;
            ctx.fillRect(bunkerX - 80, bunkerY - 40, 160, 80);
            ctx.strokeRect(bunkerX - 80, bunkerY - 40, 160, 80);

            // Compartimentos Internos
            ctx.fillStyle = '#0a0d12';
            ctx.fillRect(bunkerX - 70, bunkerY - 30, 65, 60); // Sala de Comando
            ctx.fillRect(bunkerX + 5, bunkerY - 30, 65, 60);  // Reator MHD

            // Luzes dos Servidores/Telas Internas
            ctx.fillStyle = Math.random() > 0.1 ? '#39ff14' : '#ff3333';
            ctx.fillRect(bunkerX - 60, bunkerY - 20, 6, 4);
            ctx.fillRect(bunkerX - 50, bunkerY - 20, 6, 4);
            ctx.fillStyle = '#00e5ff';
            ctx.fillRect(bunkerX + 20, bunkerY - 10, 15, 20); // Reator Core

            // Mastro/Conduíte do Bunker para a Superfície
            ctx.fillStyle = '#444';
            ctx.fillRect(bunkerX - 5, 160, 10, 180);

            // 4. Animação de Detonação e Ondas Sísmicas
            if (icbmImpactado) {
                // Crateramento na superfície
                ctx.fillStyle = '#000000';
                ctx.beginPath();
                ctx.ellipse(bunkerX, 160, raioExplosao * 0.4, 25, 0, 0, Math.PI * 2);
                ctx.fill();

                // Ondas de Choque Subterrâneas Radiadas
                ctx.strokeStyle = 'rgba(255, 51, 51, ' + Math.max(0, 1 - raioExplosao/380) + ')';
                ctx.lineWidth = 3;
                for (let r = 20; r < raioExplosao; r += 40) {
                    ctx.beginPath();
                    ctx.arc(bunkerX, 160, r, 0, Math.PI);
                    ctx.stroke();
                }

                // Cogumelo de Plasma / Explosão no Céu
                let gradExplosao = ctx.createRadialGradient(bunkerX, 150, 10, bunkerX, 120, raioExplosao * 0.6);
                gradExplosao.addColorStop(0, 'rgba(255, 255, 255, 0.95)');
                gradExplosao.addColorStop(0.3, 'rgba(255, 200, 0, 0.8)');
                gradExplosao.addColorStop(0.7, 'rgba(255, 50, 0, 0.5)');
                gradExplosao.addColorStop(1, 'rgba(0, 0, 0, 0)');
                
                ctx.fillStyle = gradExplosao;
                ctx.beginPath();
                ctx.arc(bunkerX, 120, raioExplosao * 0.6, 0, Math.PI * 2);
                ctx.fill();
            }

            // 5. Míssil ICBM caindo
            if (nuclearAtivo && !icbmImpactado) {
                // Rastro do ICBM
                ctx.strokeStyle = 'rgba(255, 200, 0, 0.6)';
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(icbmX, -50);
                ctx.lineTo(icbmX, icbmY);
                ctx.stroke();

                // Cabeça do ICBM
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(icbmX - 3, icbmY, 6, 12);
            }

            // 6. Míssil de Defesa S.A.M.S
            if (projetilSam && projetilSam.ativo && !icbmImpactado) {
                ctx.strokeStyle = '#00e5ff';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(425, 350);
                ctx.lineTo(projetilSam.x, projetilSam.y);
                ctx.stroke();

                ctx.fillStyle = '#00e5ff';
                ctx.fillRect(projetilSam.x - 2, projetilSam.y, 4, 8);
            }

            ctx.restore();

            // Próximo quadro de animação
            requestAnimationFrame(updateAndRender);
        }

        // Inicia o laço contínuo da simulação
        requestAnimationFrame(updateAndRender);
    </script>
</body>
</html>
