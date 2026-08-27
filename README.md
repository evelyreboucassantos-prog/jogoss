<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Projeto Dupla - Jogo do Labirinto</title>
    <!-- Ligação para o ficheiro CSS externo -->
    <link rel="stylesheet" href="style.css">
</head>
<body>

    <!-- Banner de Identificação da Dupla -->
    <div class="dupla-banner">
        <h2>Projeto Desenvolvido pela Dupla:</h2>
        <div class="dupla-nomes">[Nome do Integrante 1] & [Nome do Integrante 2]</div>
    </div>

    <!-- Contentor Principal -->
    <main class="card-container">
        
        <!-- Título Principal -->
        <h1>Cyber Dot - O Jogo do Labirinto</h1>

        <!-- Painel de Pontuação -->
        <div class="placar">
            <span>Pontos: <strong id="pontos">0</strong></span>
            <span>Vidas: <strong id="vidas">3</strong></span>
        </div>

        <!-- Área do Jogo (Canvas) -->
        <div class="game-wrapper">
            <canvas id="gameCanvas" width="400" height="400"></canvas>
        </div>

        <!-- Botões de Controlo e Ação -->
        <div class="controles-wrapper">
            <button id="btnIniciar" class="btn-interativo">INICIAR JOGO</button>
        </div>

        <!-- Instruções do Jogo -->
        <div class="descricao">
            <p>
                <strong>Como jogar:</strong> Utiliza as <strong>setas do teclado</strong> para mover a esfera amarela pelo labirinto. Recolhe todos os pontos brilhantes e evita os fantasmas vermelhos!
            </p>
        </div>

    </main>

    <footer>
        <p>&copy; 2026 - Desenvolvido no VS Code para a Missão da Dupla.</p>
    </footer>

    <!-- Script de Lógica do Jogo -->
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const elPontos = document.getElementById('pontos');
        const elVidas = document.getElementById('vidas');
        const btnIniciar = document.getElementById('btnIniciar');

        // Configurações do jogo
        const TAMANHO_BLOCO = 40;
        let pontos = 0;
        let vidas = 3;
        let jogoAtivo = false;
        let loopJogo = null;

        // Mapa do labirinto (1 = Parede, 0 = Ponto, 2 = Vazio)
        const mapaOriginal = [
            [1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
            [1, 0, 0, 0, 1, 0, 0, 0, 0, 1],
            [1, 0, 1, 0, 1, 0, 1, 1, 0, 1],
            [1, 0, 1, 0, 0, 0, 0, 1, 0, 1],
            [1, 0, 1, 1, 1, 1, 0, 1, 0, 1],
            [1, 0, 0, 0, 0, 1, 0, 0, 0, 1],
            [1, 1, 1, 0, 1, 1, 1, 0, 1, 1],
            [1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
            [1, 0, 1, 1, 0, 1, 1, 1, 0, 1],
            [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
        ];

        let mapa = JSON.parse(JSON.stringify(mapaOriginal));

        // Jogador (Estilo Pac-Man)
        let jogador = { x: 1, y: 1, dx: 0, dy: 0 };

        // Inimigo (Fantasma)
        let fantasma = { x: 8, y: 7, dx: 1, dy: 0 };

        // Controlo do Teclado
        window.addEventListener('keydown', (e) => {
            if (!jogoAtivo) return;
            
            if (e.key === 'ArrowUp') { jogador.dx = 0; jogador.dy = -1; }
            if (e.key === 'ArrowDown') { jogador.dx = 0; jogador.dy = 1; }
            if (e.key === 'ArrowLeft') { jogador.dx = -1; jogador.dy = 0; }
            if (e.key === 'ArrowRight') { jogador.dx = 1; jogador.dy = 0; }
        });

        function reiniciarPosicoes() {
            jogador = { x: 1, y: 1, dx: 0, dy: 0 };
            fantasma = { x: 8, y: 7, dx: 1, dy: 0 };
        }

        function iniciarJogo() {
            mapa = JSON.parse(JSON.stringify(mapaOriginal));
            pontos = 0;
            vidas = 3;
            elPontos.textContent = pontos;
            elVidas.textContent = vidas;
            reiniciarPosicoes();
            jogoAtivo = true;
            btnIniciar.textContent = "REINICIAR";

            if (loopJogo) clearInterval(loopJogo);
            loopJogo = setInterval(atualizarJogo, 200);
        }

        function atualizarJogo() {
            if (!jogoAtivo) return;

            // Mover Jogador
            let novoX = jogador.x + jogador.dx;
            let novoY = jogador.y + jogador.dy;

            if (mapa[novoY] && mapa[novoY][novoX] !== 1) {
                jogador.x = novoX;
                jogador.y = novoY;

                // Coletar pontos
                if (mapa[jogador.y][jogador.x] === 0) {
                    mapa[jogador.y][jogador.x] = 2;
                    pontos += 10;
                    elPontos.textContent = pontos;
                }
            }

            // Mover Fantasma (Movimento simples)
            let proximoFX = fantasma.x + fantasma.dx;
            let proximoFY = fantasma.y + fantasma.dy;

            if (!mapa[proximoFY] || mapa[proximoFY][proximoFX] === 1 || Math.random() < 0.3) {
                const direcoes = [{dx: 1, dy: 0}, {dx: -1, dy: 0}, {dx: 0, dy: 1}, {dx: 0, dy: -1}];
                const dirValidas = direcoes.filter(d => mapa[fantasma.y + d.dy] && mapa[fantasma.y + d.dy][fantasma.x + d.dx] !== 1);
                if (dirValidas.length > 0) {
                    const sorteada = dirValidas[Math.floor(Math.random() * dirValidas.length)];
                    fantasma.dx = sorteada.dx;
                    fantasma.dy = sorteada.dy;
                }
            } else {
                fantasma.x = proximoFX;
                fantasma.y = proximoFY;
            }

            // Verificar Colisão com Fantasma
            if (jogador.x === fantasma.x && jogador.y === fantasma.y) {
                vidas--;
                elVidas.textContent = vidas;
                if (vidas <= 0) {
                    jogoAtivo = false;
                    clearInterval(loopJogo);
                    alert("Fim de Jogo! Pontuação final: " + pontos);
                } else {
                    reiniciarPosicoes();
                }
            }

            desenhar();
        }

        function desenhar() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Desenhar Mapa
            for (let r = 0; r < mapa.length; r++) {
                for (let c = 0; c < mapa[r].length; c++) {
                    if (mapa[r][c] === 1) {
                        ctx.fillStyle = '#1e293b';
                        ctx.fillRect(c * TAMANHO_BLOCO, r * TAMANHO_BLOCO, TAMANHO_BLOCO, TAMANHO_BLOCO);
                        ctx.strokeStyle = '#00ffcc';
                        ctx.strokeRect(c * TAMANHO_BLOCO, r * TAMANHO_BLOCO, TAMANHO_BLOCO, TAMANHO_BLOCO);
                    } else if (mapa[r][c] === 0) {
                        ctx.fillStyle = '#ffeb3b';
                        ctx.beginPath();
                        ctx.arc(c * TAMANHO_BLOCO + 20, r * TAMANHO_BLOCO + 20, 4, 0, Math.PI * 2);
                        ctx.fill();
                    }
                }
            }

            // Desenhar Jogador
            ctx.fillStyle = '#ffeb3b';
            ctx.beginPath();
            ctx.arc(jogador.x * TAMANHO_BLOCO + 20, jogador.y * TAMANHO_BLOCO + 20, 14, 0, Math.PI * 2);
            ctx.fill();

            // Desenhar Fantasma
            ctx.fillStyle = '#ff0055';
            ctx.beginPath();
            ctx.arc(fantasma.x * TAMANHO_BLOCO + 20, fantasma.y * TAMANHO_BLOCO + 20, 14, 0, Math.PI * 2);
            ctx.fill();
        }

        btnIniciar.addEventListener('click', iniciarJogo);
        desenhar();
    </script>
</body>
</html>