# rosa-0

        .thorn-color {
            color: #1e8449;
        }

        /* Glassmorphism Controls Panel */
        .glass-panel {
            background: rgba(15, 23, 42, 0.65);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.5);
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #050508;
        }
        ::-webkit-scrollbar-thumb {
            background: #1e293b;
            border-radius: 3px;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between items-center relative overflow-x-hidden selection:bg-pink-500 selection:text-white">

    <!-- Canvas para Estrelas no Fundo -->
    <canvas id="stars-canvas"></canvas>
    
    <!-- Canvas para Efeitos Sparkle no Clique -->
    <canvas id="particles-canvas"></canvas>

    <!-- Top Terminal Stats Header -->
    <header class="w-full max-w-4xl px-4 pt-4 z-20">
        <div class="glass-panel rounded-xl p-3 flex flex-wrap justify-between items-center text-xs sm:text-sm text-slate-400 font-mono gap-2 border-b border-pink-500/20">
            <div class="flex items-center space-x-3">
                <span class="inline-block w-2.5 h-2.5 rounded-full bg-red-500 animate-pulse"></span>
                <span class="text-slate-300 font-bold">py_art.py</span>
                <span class="text-slate-600">|</span>
                <span id="stat-fps" class="text-pink-400">30 fps</span>
            </div>
            <div class="flex items-center space-x-4">
                <span>bloom: <span id="stat-bloom" class="text-emerald-400">100.0%</span></span>
                <span>sway: <span id="stat-sway" class="text-cyan-400">0.00°</span></span>
                <span>letters: <span id="stat-letters" class="text-amber-400">0</span></span>
            </div>
        </div>
    </header>

    <!-- Main ASCII Container -->
    <main class="w-full flex-1 flex flex-col justify-center items-center relative z-10 my-4 px-2">
        <div id="rose-wrapper" class="ascii-container transition-transform duration-700 ease-in-out p-4 flex justify-center items-center min-h-[420px] sm:min-h-[500px]">
            <pre id="rose-display" class="select-none text-center font-mono"></pre>
        </div>
    </main>

    <!-- Controls & Customization Section -->
    <footer class="w-full max-w-2xl px-4 pb-6 z-20 flex flex-col items-center gap-4">
        
        <!-- Controls Buttons -->
        <div class="glass-panel p-2.5 rounded-full flex items-center gap-2 sm:gap-4 shadow-2xl">
            <button id="btn-replay" onclick="triggerBloom()" class="flex items-center gap-2 px-4 py-2 rounded-full bg-pink-600/30 hover:bg-pink-600/50 text-pink-200 border border-pink-500/30 transition-all text-xs sm:text-sm active:scale-95">
                <i class="fa-solid fa-play text-pink-400"></i> Bloom (Florescer)
            </button>

            <button id="btn-sway" onclick="toggleSway()" class="flex items-center gap-2 px-4 py-2 rounded-full bg-emerald-600/30 hover:bg-emerald-600/50 text-emerald-200 border border-emerald-500/30 transition-all text-xs sm:text-sm active:scale-95">
                <i class="fa-solid fa-wind text-emerald-400"></i> <span id="sway-btn-text">Balançar</span>
            </button>

            <button onclick="burstSparkles()" class="flex items-center gap-2 px-4 py-2 rounded-full bg-amber-600/30 hover:bg-amber-600/50 text-amber-200 border border-amber-500/30 transition-all text-xs sm:text-sm active:scale-95">
                <i class="fa-solid fa-wand-magic-sparkles text-amber-400"></i> Brilhar
            </button>
        </div>

        <!-- Custom Message Footer -->
        <div class="w-full glass-panel rounded-2xl p-4 flex flex-col items-center text-center gap-2 border-t border-slate-700/50">
            <div class="flex items-center gap-2 w-full justify-center">
                <i class="fa-solid fa-heart text-pink-500 animate-bounce"></i>
                <input type="text" id="custom-message" value="Send this rose to someone special ❤️" 
                       class="bg-transparent text-center text-slate-200 font-semibold text-sm sm:text-base focus:outline-none border-b border-transparent hover:border-slate-600 focus:border-pink-500 transition-all w-full max-w-md px-2 py-1"
                       title="Clique para editar a mensagem">
            </div>
            <p class="text-xs text-slate-500">Dica: Clique nas pétalas ou caule para disparar partículas de luz!</p>
        </div>
    </footer>

    <script>
        // Matrix/Template Arte ASCII da Rosa com Tags de Estilização
        // [P] = Petal Light, [D] = Petal Dark, [H] = Petal Highlight, [S] = Stem, [L] = Leaf, [T] = Thorn
        const RAW_ROSE_TEMPLATE = [
            "                    [H].--.[P]~.~[H].--.",
            "                 [D].'  [P]/   \\   [D]'.",
            "               [P]/    [H]|  [P]/ \\  [H]|    [P]\\",
            "              [D]|   [P]/   [H]|   |[P]   \\   [D]|",
            "               [P]\\  [D]\\   [H]|   |[P]   /  [P]/",
            "                [H]'. [P]'--'[H]|   |[P]'--' [H].'",
            "              [D]___[P]'.    [H]\\ /    [P].'[D]___",
            "            [P]/  [D].--.  [P]'---'  [D].--.  [P]\\",
            "           [D]|  /    [P]\\       /    [D]\\  |",
            "            [P]\\ \\    [H]/       \\    [P]/ /",
            "             [D]'.[P]'--'         '--'[P].'",
            "               [H]'-.[P]             [H].-'",
            "                   [P]'--.       .--'",
            "                        [P]\\     /",
            "                         [H]|   |",
            "                         [P]\\ /",
            "                          [S]|",
            "                          [S]|  [L].--.",
            "                          [S]| [L]/    \\",
            "                 [L].--.  [S]|[L]|  [S]|  [L]|",
            "                [L]/    \\[S]|[L]\\  [S]|  [L]/",
            "                [L]|  [S]|  [L]|[S]  |[S]|  [L]'--.",
            "                [L]\\  [S]|  [L]/[S]  |[S]|     [L]\\",
            "                 [L]'--'[S]|  [S]|  [S]|  [L]/'",
            "                        [S]|  [S]|  [S]|",
            "                        [S]|  [S]|  [S]|",
            "                        [T]|/ [S]|  [S]|",
            "                          [S]|  [S]|",
            "                          [S]|  [S]|",
            "                          [S]|  [S]|",
            "                          [S]|  [S]|",
            "                          [S]|  [S]|",
            "                          [S]|  [S]|",
            "                         [S]/    \\",
            "                        [S]'------'"
        ];

        // Mapeamento de Tags para Classes CSS de Cores
        const COLOR_MAP = {
            'P': 'petal-color',
            'D': 'petal-dark',
            'H': 'petal-highlight',
            'S': 'stem-color',
            'L': 'leaf-color',
            'T': 'thorn-color'
        };

        // Estado Global da Aplicação
        let isSwaying = false;
        let isBloomAnimating = false;
        let bloomProgress = 1.0;
        let totalLettersCount = 0;
        let currentSwayAngle = 0;

        // -------------------------------------------------------------
        // 1. ANIMAÇÃO DE ESTRELAS NO CANVAS (FUNDO)
        // -------------------------------------------------------------
        const starsCanvas = document.getElementById('stars-canvas');
        const starsCtx = starsCanvas.getContext('2d');
        let stars = [];

        function resizeStarsCanvas() {
            starsCanvas.width = window.innerWidth;
            starsCanvas.height = window.innerHeight;
            initStars();
        }

        function initStars() {
            stars = [];
            const count = Math.floor((starsCanvas.width * starsCanvas.height) / 3000);
            for (let i = 0; i < count; i++) {
                stars.push({
                    x: Math.random() * starsCanvas.width,
                    y: Math.random() * starsCanvas.height,
                    radius: Math.random() * 1.4 + 0.3,
                    alpha: Math.random(),
                    speed: Math.random() * 0.015 + 0.005,
                    isCross: Math.random() > 0.92
                });
            }
        }

        function drawCrossStar(ctx, x, y, size, alpha) {
            ctx.save();
            ctx.strokeStyle = `rgba(255, 255, 255, ${alpha})`;
            ctx.lineWidth = 0.8;
            ctx.beginPath();
            ctx.moveTo(x - size, y);
            ctx.lineTo(x + size, y);
            ctx.moveTo(x, y - size);
            ctx.lineTo(x, y + size);
            ctx.stroke();
            ctx.restore();
        }

        function animateStars() {
            starsCtx.clearRect(0, 0, starsCanvas.width, starsCanvas.height);
            
            stars.forEach(star => {
                star.alpha += star.speed;
                if (star.alpha > 1 || star.alpha < 0) {
                    star.speed = -star.speed;
                }

                const currentAlpha = Math.max(0, Math.min(1, star.alpha));

                if (star.isCross) {
                    drawCrossStar(starsCtx, star.x, star.y, star.radius * 6, currentAlpha * 0.7);
                } else {
                    starsCtx.beginPath();
                    starsCtx.arc(star.x, star.y, star.radius, 0, Math.PI * 2);
                    starsCtx.fillStyle = `rgba(255, 255, 255, ${currentAlpha})`;
                    starsCtx.shadowBlur = 4;
                    starsCtx.shadowColor = '#ffffff';
                    starsCtx.fill();
                }
            });

            requestAnimationFrame(animateStars);
        }

        window.addEventListener('resize', resizeStarsCanvas);
        resizeStarsCanvas();
        animateStars();

        // -------------------------------------------------------------
        // 2. PARSER E RENDERIZADOR DA ROSA ASCII
        // -------------------------------------------------------------
        function parseLineToHTML(line, visibleCharsLimit = Infinity) {
            let html = '';
            let charCounter = 0;
            let i = 0;

            while (i < line.length) {
                if (charCounter >= visibleCharsLimit) break;

                if (line[i] === '[') {
                    const tag = line[i + 1];
                    if (line[i + 2] === ']') {
                        i += 3;
                        // Próximo caractere terá este estilo
                        if (i < line.length && line[i] !== '[') {
                            const char = line[i];
                            const cssClass = COLOR_MAP[tag] || '';
                            html += `<span class="${cssClass}" onclick="createSparkleEvent(event)">${escapeHtml(char)}</span>`;
                            charCounter++;
                            i++;
                        }
                        continue;
                    }
                }

                const char = line[i];
                html += escapeHtml(char);
                charCounter++;
                i++;
            }

            return { html, count: charCounter };
        }

        function escapeHtml(str) {
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
        }

        function calculateTotalLetters() {
            let total = 0;
            RAW_ROSE_TEMPLATE.forEach(line => {
                const cleaned = line.replace(/\[[PDHSL]\]/g, '');
                total += cleaned.length;
            });
            return total;
        }

        function renderRose(progress = 1.0) {
            const display = document.getElementById('rose-display');
            totalLettersCount = calculateTotalLetters();
            const targetVisibleCount = Math.floor(totalLettersCount * progress);

            let renderedHtml = '';
            let currentVisible = 0;

            for (let line of RAW_ROSE_TEMPLATE) {
                if (currentVisible >= targetVisibleCount) break;
                
                const remaining = targetVisibleCount - currentVisible;
                const { html, count } = parseLineToHTML(line, remaining);
                renderedHtml += html + '\n';
                currentVisible += count;
            }

            display.innerHTML = renderedHtml;

            // Atualiza estatísticas do topo
            document.getElementById('stat-bloom').innerText = (progress * 100).toFixed(1) + '%';
            document.getElementById('stat-letters').innerText = currentVisible;
        }

        // -------------------------------------------------------------
        // 3. ANIMAÇÃO BLOOM (FLORESCER)
        // -------------------------------------------------------------
        function triggerBloom() {
            if (isBloomAnimating) return;
            isBloomAnimating = true;
            bloomProgress = 0;
            
            const duration = 2500; // ms
            const startTime = performance.now();

            function step(currentTime) {
                const elapsed = currentTime - startTime;
                bloomProgress = Math.min(1.0, elapsed / duration);
                
                renderRose(bloomProgress);

                if (bloomProgress < 1.0) {
                    requestAnimationFrame(step);
                } else {
                    isBloomAnimating = false;
                    burstSparkles();
                }
            }

            requestAnimationFrame(step);
        }

        // -------------------------------------------------------------
        // 4. EFEITO DE BALANÇO (SWAY) & SIMULAÇÃO DE VALORES
        // -------------------------------------------------------------
        function toggleSway() {
            const wrapper = document.getElementById('rose-wrapper');
            const btnText = document.getElementById('sway-btn-text');
            
            isSwaying = !isSwaying;

            if (isSwaying) {
                wrapper.classList.add('swaying');
                btnText.innerText = "Parar";
            } else {
                wrapper.classList.remove('swaying');
                btnText.innerText = "Balançar";
                document.getElementById('stat-sway').innerText = "0.00°";
            }
        }

        // Atualizador dos valores no terminal superior
        setInterval(() => {
            if (isSwaying) {
                // Simula variação senoidal do ângulo de sway
                const angle = (Math.sin(Date.now() / 600) * 1.8).toFixed(2);
                document.getElementById('stat-sway').innerText = (angle >= 0 ? '+' : '') + angle + '°';
            }
            // Pequena variação aleatória no FPS simulado
            const fps = Math.floor(29 + Math.random() * 3);
            document.getElementById('stat-fps').innerText = fps + ' fps';
        }, 150);

        // -------------------------------------------------------------
        // 5. CANVAS DE PARTÍCULAS / BRILHO (INTERATIVIDADE)
        // -------------------------------------------------------------
        const particlesCanvas = document.getElementById('particles-canvas');
        const particlesCtx = particlesCanvas.getContext('2d');
        let particles = [];

        function resizeParticlesCanvas() {
            particlesCanvas.width = window.innerWidth;
            particlesCanvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeParticlesCanvas);
        resizeParticlesCanvas();

        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.color = color || '#ff7597';
                this.radius = Math.random() * 3 + 1.5;
                this.vx = (Math.random() - 0.5) * 4;
                this.vy = (Math.random() - 0.5) * 4 - 1;
                this.alpha = 1;
                this.decay = Math.random() * 0.02 + 0.015;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.alpha -= this.decay;
            }

            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.alpha);
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        function createSparkleEvent(e) {
            const rect = e.target.getBoundingClientRect();
            const x = rect.left + rect.width / 2;
            const y = rect.top + rect.height / 2;

            const isStem = e.target.classList.contains('stem-color') || e.target.classList.contains('leaf-color');
            const color = isStem ? '#58d68d' : '#ff7597';

            for (let i = 0; i < 12; i++) {
                particles.push(new Particle(x, y, color));
            }
        }

        function burstSparkles() {
            const wrapper = document.getElementById('rose-wrapper');
            const rect = wrapper.getBoundingClientRect();
            const centerX = rect.left + rect.width / 2;
            const centerY = rect.top + rect.height / 3;

            const colors = ['#ff3b6b', '#ff7597', '#58d68d', '#ffd166', '#ffffff'];

            for (let i = 0; i < 40; i++) {
                const randomColor = colors[Math.floor(Math.random() * colors.length)];
                particles.push(new Particle(centerX, centerY, randomColor));
            }
        }

        function animateParticles() {
            particlesCtx.clearRect(0, 0, particlesCanvas.width, particlesCanvas.height);

            for (let i = particles.length - 1; i >= 0; i--) {
                particles[i].update();
                particles[i].draw(particlesCtx);

                if (particles[i].alpha <= 0) {
                    particles.splice(i, 1);
                }
            }

            requestAnimationFrame(animateParticles);
        }
        animateParticles();

        // -------------------------------------------------------------
        // INICIALIZAÇÃO DA PÁGINA
        // -------------------------------------------------------------
        window.onload = () => {
            renderRose(1.0);
            // Inicia com um bloom suave inicial
            setTimeout(() => {
                triggerBloom();
            }, 300);
        };
    </script>
</body>
</html>