# かきかたれんしゅうアプリ
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>たのしく かきかた れんしゅう！</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti for Celebration -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@400;700;800&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'M PLUS Rounded 1c', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        /* Custom scrollbar for character selection grid */
        .char-grid::-webkit-scrollbar {
            width: 8px;
        }
        .char-grid::-webkit-scrollbar-track {
            background: #f1f5f9;
            border-radius: 8px;
        }
        .char-grid::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 8px;
        }
        .char-grid::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        /* Canvas layer stacking */
        .canvas-container {
            position: relative;
            width: 100%;
            max-width: 360px;
            aspect-ratio: 1 / 1;
        }
        .canvas-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border-radius: 1rem;
        }
        /* Rainbow pen effect gradient */
        .rainbow-bg {
            background: linear-gradient(135deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #9400d3);
        }
        /* Bouncing animation for result modal */
        @keyframes popIn {
            0% { transform: scale(0.3); opacity: 0; }
            70% { transform: scale(1.1); opacity: 1; }
            100% { transform: scale(1); }
        }
        .pop-in {
            animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
    </style>
</head>
<body class="bg-amber-50 min-h-screen text-slate-800 flex flex-col justify-between">

    <header class="bg-amber-400 text-white shadow-md p-3 sm:p-4 text-center sticky top-0 z-20">
        <div class="max-w-4xl mx-auto flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <span class="text-2xl sm:text-3xl">✏️</span>
                <h1 class="text-xl sm:text-2xl font-extrabold tracking-wide drop-shadow-sm">かきかた れんしゅう</h1>
            </div>
            <!-- Audio toggle button -->
            <div class="flex items-center space-x-2">
                <button id="soundToggleBtn" onclick="toggleSound()" class="bg-amber-500 hover:bg-amber-600 text-white px-3 py-1.5 rounded-full font-bold text-sm shadow transition flex items-center space-x-1">
                    <i class="fas fa-volume-high" id="soundIcon"></i>
                    <span id="soundText" class="hidden sm:inline">おと ON</span>
                </button>
            </div>
        </div>
    </header>

    <main class="max-w-4xl w-full mx-auto p-2 sm:p-4 flex-grow flex flex-col items-center justify-start space-y-4">
        
        <!-- Mode Switcher Tabs (Hiragana / Katakana / Numbers) -->
        <div class="flex rounded-2xl bg-amber-200/70 p-1.5 w-full max-w-md shadow-inner">
            <button id="tabHiragana" onclick="switchCategory('hiragana')" class="flex-1 py-2 rounded-xl font-bold text-base sm:text-lg transition-all text-amber-900 bg-white shadow-sm">
                ひらがな
            </button>
            <button id="tabKatakana" onclick="switchCategory('katakana')" class="flex-1 py-2 rounded-xl font-bold text-base sm:text-lg transition-all text-amber-800 hover:bg-white/50">
                カタカナ
            </button>
            <button id="tabNumbers" onclick="switchCategory('numbers')" class="flex-1 py-2 rounded-xl font-bold text-base sm:text-lg transition-all text-amber-800 hover:bg-white/50">
                すうじ
            </button>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-12 gap-4 w-full items-start">
            
            <!-- Left Panel: Character Selection Grid -->
            <div class="md:col-span-5 bg-white p-3 sm:p-4 rounded-3xl shadow-md border-2 border-amber-200 flex flex-col h-[280px] md:h-[480px]">
                <div class="flex justify-between items-center mb-2">
                    <span class="text-sm font-bold text-amber-700" id="pickerTitle">もじを えらんでね</span>
                    <button onclick="speakCurrentChar()" class="bg-amber-100 hover:bg-amber-200 text-amber-800 px-3 py-1 rounded-full text-xs font-bold transition flex items-center space-x-1">
                        <i class="fas fa-volume-high"></i>
                        <span>きく</span>
                    </button>
                </div>
                <!-- Scrollable Character Grid -->
                <div id="charGrid" class="char-grid grid grid-cols-5 gap-1.5 overflow-y-auto p-1 flex-grow">
                    <!-- Dynamic Buttons Generated via JavaScript -->
                </div>
            </div>

            <div class="md:col-span-7 bg-white p-3 sm:p-5 rounded-3xl shadow-md border-2 border-amber-200 flex flex-col items-center justify-between space-y-3">
                
                <!-- Active letter status display -->
                <div class="flex items-center justify-between w-full px-2">
                    <div class="flex items-center space-x-2">
                        <span class="text-xs font-bold bg-amber-100 text-amber-800 px-2.5 py-1 rounded-full" id="currentCharType">ひらがな</span>
                        <span class="text-3xl font-extrabold text-amber-600" id="currentCharDisplay">あ</span>
                    </div>
                    <button onclick="speakCurrentChar()" class="bg-amber-400 hover:bg-amber-500 active:scale-95 text-white px-3 py-1.5 rounded-2xl font-bold text-sm shadow transition flex items-center space-x-1.5">
                        <i class="fas fa-bullhorn"></i>
                        <span>よみあげる</span>
                    </button>
                </div>

                <!-- Canvas Stack (Grid / Template Guide / User Stroke) -->
                <div class="canvas-container bg-amber-50 rounded-2xl shadow-inner border-4 border-amber-300 overflow-hidden cursor-crosshair">
                    <canvas id="bgCanvas" width="360" height="360" class="canvas-layer"></canvas>
                    <canvas id="templateCanvas" width="360" height="360" class="canvas-layer"></canvas>
                    <canvas id="drawCanvas" width="360" height="360" class="canvas-layer"></canvas>
                </div>

                <!-- Pen Tool Options Toolbar -->
                <div class="w-full flex flex-wrap items-center justify-between gap-2 pt-1 border-t border-slate-100">
                    <!-- Colors -->
                    <div class="flex items-center space-x-1.5 sm:space-x-2" id="colorPalette">
                        <button onclick="setPenColor('#1e293b')" class="color-btn w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-slate-800 ring-2 ring-offset-2 ring-amber-500 transition-transform active:scale-90" data-color="#1e293b"></button>
                        <button onclick="setPenColor('#ef4444')" class="color-btn w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-red-500 transition-transform active:scale-90" data-color="#ef4444"></button>
                        <button onclick="setPenColor('#2563eb')" class="color-btn w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-blue-600 transition-transform active:scale-90" data-color="#2563eb"></button>
                        <button onclick="setPenColor('#16a34a')" class="color-btn w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-green-600 transition-transform active:scale-90" data-color="#16a34a"></button>
                        <button onclick="setPenColor('rainbow')" class="color-btn w-8 h-8 sm:w-9 sm:h-9 rounded-full rainbow-bg transition-transform active:scale-90 relative" data-color="rainbow" title="にじいろ">
                            <span class="absolute -top-1 -right-1 text-xs">✨</span>
                        </button>
                    </div>

                    <!-- Line Width Picker -->
                    <div class="flex items-center space-x-1 bg-slate-100 p-1 rounded-xl">
                        <button onclick="setLineWidth(14)" id="sizeThin" class="px-2.5 py-1 rounded-lg text-xs font-bold text-slate-600 hover:bg-white transition">ほそい</button>
                        <button onclick="setLineWidth(22)" id="sizeMed" class="px-2.5 py-1 rounded-lg text-xs font-bold bg-white text-amber-700 shadow-sm transition">ふつう</button>
                        <button onclick="setLineWidth(32)" id="sizeThick" class="px-2.5 py-1 rounded-lg text-xs font-bold text-slate-600 hover:bg-white transition">ふとい</button>
                    </div>

                    <!-- Guide Opacity Toggle -->
                    <button onclick="toggleTemplateGuide()" id="guideToggleBtn" class="bg-slate-100 hover:bg-slate-200 text-slate-700 px-2.5 py-1.5 rounded-xl text-xs font-bold transition flex items-center space-x-1">
                        <i class="fas fa-eye"></i>
                        <span id="guideText">おてほん：こい</span>
                    </button>
                </div>

                <!-- Bottom Action Buttons -->
                <div class="grid grid-cols-2 gap-3 w-full pt-1">
                    <button onclick="clearUserCanvas()" class="py-3 px-4 bg-slate-200 hover:bg-slate-300 active:scale-95 text-slate-700 font-bold rounded-2xl shadow-sm transition flex items-center justify-center space-x-2 text-base">
                        <i class="fas fa-eraser"></i>
                        <span>ぜんぶけす</span>
                    </button>
                    <button onclick="evaluateDrawing()" class="py-3 px-4 bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white font-extrabold rounded-2xl shadow-lg shadow-emerald-200 transition flex items-center justify-center space-x-2 text-lg">
                        <i class="fas fa-check-circle"></i>
                        <span>できた！</span>
                    </button>
                </div>

            </div>
        </div>
    </main>

    <!-- Footer -->
    <footer class="text-center p-3 text-xs text-amber-700/70 font-bold">
        しょうがく１ねんせい の ための たのしい かきかた アプリ 🌸
    </footer>

    <div id="resultModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white rounded-3xl p-6 max-w-sm w-full text-center shadow-2xl pop-in border-4 border-amber-300 relative overflow-hidden">
            <div id="modalHanamaruIcon" class="text-7xl my-2 animate-bounce">🌸</div>
            <h2 id="resultTitle" class="text-2xl font-extrabold text-amber-600 mb-1">たいへん よくできました！</h2>
            <p id="resultScoreText" class="text-slate-600 font-bold text-sm mb-4">おてほん通りに とても上手になぞれました！</p>
            
            <div class="flex justify-center space-x-1 text-amber-400 text-3xl mb-6" id="starContainer">
                <i class="fas fa-star"></i>
                <i class="fas fa-star"></i>
                <i class="fas fa-star"></i>
            </div>

            <div class="flex flex-col space-y-2">
                <button onclick="nextCharacter()" class="w-full py-3 bg-amber-500 hover:bg-amber-600 text-white font-extrabold rounded-2xl shadow-md text-base transition transform active:scale-95 flex items-center justify-center space-x-2">
                    <span>つぎの もじへ</span>
                    <i class="fas fa-arrow-right"></i>
                </button>
                <button onclick="closeModal()" class="w-full py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-600 font-bold rounded-2xl text-sm transition">
                    もういちど れんしゅう
                </button>
            </div>
        </div>
    </div>

    <script>
        // Character datasets for Hiragana, Katakana, and Numbers
        const CHAR_DATA = {
            hiragana: [
                'あ','い','う','え','お',
                'か','き','く','け','こ',
                'さ','し','す','せ','そ',
                'た','ち','つ','て','と',
                'な','に','ぬ','ね','の',
                'は','ひ','ふ','へ','ほ',
                'ま','み','む','め','も',
                'や','ゆ','よ',
                'ら','り','る','れ','ろ',
                'わ','を','ん'
            ],
            katakana: [
                'ア','イ','ウ','エ','オ',
                'カ','キ','ク','ケ','コ',
                'サ','シ','ス','セ','ソ',
                'タ','チ','ツ','テ','ト',
                'ナ','ニ','ヌ','ネ','ノ',
                'ハ','ヒ','フ','ヘ','ホ',
                'マ','ミ','ム','メ','モ',
                'ヤ','ユ','ヨ',
                'ラ','リ','ル','レ','ロ',
                'ワ','ヲ','ン'
            ],
            numbers: [
                '0','1','2','3','4',
                '5','6','7','8','9','10'
            ]
        };

        // Application state variables
        let currentCategory = 'hiragana';
        let currentChar = 'あ';
        let isSoundOn = true;
        let templateOpacityMode = 0;
        const opacityValues = [0.22, 0.08, 0.0];
        const opacityLabels = ['おてほん：こい', 'おてほん：うすい', 'おてほん：なし'];

        // Canvas context definitions
        let bgCanvas, templateCanvas, drawCanvas;
        let bgCtx, templateCtx, drawCtx;
        
        // Drawing options
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;
        let penColor = '#1e293b';
        let isRainbowPen = false;
        let rainbowHue = 0;
        let lineWidth = 22;

        // Audio synthesizer context
        let audioCtx = null;

        window.onload = function() {
            initCanvases();
            buildCharGrid();
            selectChar(CHAR_DATA.hiragana[0]);
            setupCanvasEvents();
            
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);
        };

        function getAudioContext() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
            return audioCtx;
        }

        function playSound(type) {
            if (!isSoundOn) return;
            try {
                const ctx = getAudioContext();
                const now = ctx.currentTime;

                if (type === 'click') {
                    const osc = ctx.createOscillator();
                    const gain = ctx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(400, now);
                    osc.frequency.exponentialRampToValueAtTime(800, now + 0.08);
                    gain.gain.setValueAtTime(0.15, now);
                    gain.gain.linearRampToValueAtTime(0.01, now + 0.08);
                    osc.connect(gain);
                    gain.connect(ctx.destination);
                    osc.start(now);
                    osc.stop(now + 0.08);
                } else if (type === 'clear') {
                    const osc = ctx.createOscillator();
                    const gain = ctx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(300, now);
                    osc.frequency.linearRampToValueAtTime(150, now + 0.15);
                    gain.gain.setValueAtTime(0.2, now);
                    gain.gain.linearRampToValueAtTime(0.01, now + 0.15);
                    osc.connect(gain);
                    gain.connect(ctx.destination);
                    osc.start(now);
                    osc.stop(now + 0.15);
                } else if (type === 'fanfare') {
                    const notes = [523.25, 659.25, 783.99, 1046.50]; // C5, E5, G5, C6
                    notes.forEach((freq, i) => {
                        const osc = ctx.createOscillator();
                        const gain = ctx.createGain();
                        osc.type = 'triangle';
                        osc.frequency.setValueAtTime(freq, now + i * 0.09);
                        gain.gain.setValueAtTime(0.25, now + i * 0.09);
                        gain.gain.exponentialRampToValueAtTime(0.001, now + i * 0.09 + 0.3);
                        osc.connect(gain);
                        gain.connect(ctx.destination);
                        osc.start(now + i * 0.09);
                        osc.stop(now + i * 0.09 + 0.3);
                    });
                }
            } catch (e) {
                console.log("Audio play error:", e);
            }
        }

        function toggleSound() {
            isSoundOn = !isSoundOn;
            const icon = document.getElementById('soundIcon');
            const text = document.getElementById('soundText');
            if (isSoundOn) {
                icon.className = 'fas fa-volume-high';
                text.textContent = 'おと ON';
                playSound('click');
            } else {
                icon.className = 'fas fa-volume-xmark';
                text.textContent = 'おと OFF';
            }
        }

        function speakCurrentChar() {
            if (!('speechSynthesis' in window)) return;
            window.speechSynthesis.cancel();
            
            const utterance = new SpeechSynthesisUtterance(currentChar);
            utterance.lang = 'ja-JP';
            utterance.rate = 0.85;
            utterance.pitch = 1.1;
            window.speechSynthesis.speak(utterance);
        }

        function initCanvases() {
            bgCanvas = document.getElementById('bgCanvas');
            templateCanvas = document.getElementById('templateCanvas');
            drawCanvas = document.getElementById('drawCanvas');

            bgCtx = bgCanvas.getContext('2d');
            templateCtx = templateCanvas.getContext('2d');
            drawCtx = drawCanvas.getContext('2d');
        }

        function resizeCanvas() {
            const container = bgCanvas.parentElement;
            const rect = container.getBoundingClientRect();
            const size = rect.width;

            [bgCanvas, templateCanvas, drawCanvas].forEach(canvas => {
                canvas.width = size;
                canvas.height = size;
            });

            drawBackgroundGrid();
            drawTemplateText();
        }

        // Draw Japanese manuscript style 2x2 grid with dotted cross
        function drawBackgroundGrid() {
            const w = bgCanvas.width;
            const h = bgCanvas.height;
            bgCtx.clearRect(0, 0, w, h);

            bgCtx.fillStyle = '#ffffff';
            bgCtx.fillRect(0, 0, w, h);

            bgCtx.strokeStyle = '#fcd34d';
            bgCtx.lineWidth = 6;
            bgCtx.strokeRect(3, 3, w - 6, h - 6);

            bgCtx.beginPath();
            bgCtx.strokeStyle = '#fef08a';
            bgCtx.lineWidth = 3;
            bgCtx.setLineDash([8, 6]);

            bgCtx.moveTo(w / 2, 0);
            bgCtx.lineTo(w / 2, h);

            bgCtx.moveTo(0, h / 2);
            bgCtx.lineTo(w, h / 2);

            bgCtx.stroke();
            bgCtx.setLineDash([]);
        }

        function drawTemplateText() {
            const w = templateCanvas.width;
            const h = templateCanvas.height;
            templateCtx.clearRect(0, 0, w, h);

            const opacity = opacityValues[templateOpacityMode];
            if (opacity === 0) return;

            templateCtx.fillStyle = `rgba(30, 41, 59, ${opacity})`;
            templateCtx.textAlign = 'center';
            templateCtx.textBaseline = 'middle';

            const fontSize = w * 0.72;
            templateCtx.font = `bold ${fontSize}px "M PLUS Rounded 1c", "Hiragino Kaku Gothic ProN", "Meiryo", sans-serif`;

            templateCtx.fillText(currentChar, w / 2, h / 2 + fontSize * 0.04);
        }

        function switchCategory(category) {
            playSound('click');
            currentCategory = category;

            ['hiragana', 'katakana', 'numbers'].forEach(cat => {
                const tab = document.getElementById('tab' + cat.charAt(0).toUpperCase() + cat.slice(1));
                if (cat === category) {
                    tab.className = 'flex-1 py-2 rounded-xl font-bold text-base sm:text-lg transition-all text-amber-900 bg-white shadow-sm';
                } else {
                    tab.className = 'flex-1 py-2 rounded-xl font-bold text-base sm:text-lg transition-all text-amber-800 hover:bg-white/50';
                }
            });

            const typeLabels = { hiragana: 'ひらがな', katakana: 'カタカナ', numbers: 'すうじ' };
            document.getElementById('currentCharType').textContent = typeLabels[category];

            buildCharGrid();
            selectChar(CHAR_DATA[category][0]);
        }

        function buildCharGrid() {
            const grid = document.getElementById('charGrid');
            grid.innerHTML = '';

            CHAR_DATA[currentCategory].forEach(ch => {
                const btn = document.createElement('button');
                btn.className = `aspect-square flex items-center justify-center text-xl font-bold rounded-xl transition border ${
                    ch === currentChar
                        ? 'bg-amber-400 text-white border-amber-500 shadow-md scale-105'
                        : 'bg-amber-50 hover:bg-amber-100 text-amber-900 border-amber-200'
                }`;
                btn.textContent = ch;
                btn.onclick = () => {
                    playSound('click');
                    selectChar(ch);
                };
                grid.appendChild(btn);
            });
        }

        function selectChar(ch) {
            currentChar = ch;
            document.getElementById('currentCharDisplay').textContent = ch;

            const buttons = document.getElementById('charGrid').querySelectorAll('button');
            buttons.forEach(btn => {
                if (btn.textContent === ch) {
                    btn.className = 'aspect-square flex items-center justify-center text-xl font-bold rounded-xl transition border bg-amber-400 text-white border-amber-500 shadow-md scale-105';
                } else {
                    btn.className = 'aspect-square flex items-center justify-center text-xl font-bold rounded-xl transition border bg-amber-50 hover:bg-amber-100 text-amber-900 border-amber-200';
                }
            });

            clearUserCanvas();
            drawTemplateText();
            speakCurrentChar();
        }

        function setupCanvasEvents() {
            drawCanvas.addEventListener('mousedown', startDrawing);
            drawCanvas.addEventListener('mousemove', draw);
            drawCanvas.addEventListener('mouseup', stopDrawing);
            drawCanvas.addEventListener('mouseleave', stopDrawing);

            drawCanvas.addEventListener('touchstart', (e) => {
                e.preventDefault();
                startDrawing(e.touches[0]);
            }, { passive: false });

            drawCanvas.addEventListener('touchmove', (e) => {
                e.preventDefault();
                draw(e.touches[0]);
            }, { passive: false });

            drawCanvas.addEventListener('touchend', (e) => {
                e.preventDefault();
                stopDrawing();
            }, { passive: false });
        }

        function getCanvasCoordinates(e) {
            const rect = drawCanvas.getBoundingClientRect();
            return {
                x: (e.clientX - rect.left) * (drawCanvas.width / rect.width),
                y: (e.clientY - rect.top) * (drawCanvas.height / rect.height)
            };
        }

        function startDrawing(e) {
            isDrawing = true;
            const coords = getCanvasCoordinates(e);
            lastX = coords.x;
            lastY = coords.y;

            drawCtx.beginPath();
            drawCtx.arc(lastX, lastY, lineWidth / 2, 0, Math.PI * 2);
            drawCtx.fillStyle = getCurrentPenStyle();
            drawCtx.fill();
        }

        function draw(e) {
            if (!isDrawing) return;
            const coords = getCanvasCoordinates(e);

            drawCtx.beginPath();
            drawCtx.moveTo(lastX, lastY);
            drawCtx.lineTo(coords.x, coords.y);

            drawCtx.strokeStyle = getCurrentPenStyle();
            drawCtx.lineWidth = lineWidth;
            drawCtx.lineCap = 'round';
            drawCtx.lineJoin = 'round';
            drawCtx.stroke();

            lastX = coords.x;
            lastY = coords.y;
        }

        function stopDrawing() {
            isDrawing = false;
        }

        function getCurrentPenStyle() {
            if (isRainbowPen) {
                rainbowHue = (rainbowHue + 3) % 360;
                return `hsl(${rainbowHue}, 90%, 55%)`;
            }
            return penColor;
        }

        function setPenColor(color) {
            playSound('click');
            if (color === 'rainbow') {
                isRainbowPen = true;
            } else {
                isRainbowPen = false;
                penColor = color;
            }

            const btns = document.querySelectorAll('.color-btn');
            btns.forEach(btn => {
                if (btn.dataset.color === color) {
                    btn.classList.add('ring-2', 'ring-offset-2', 'ring-amber-500');
                } else {
                    btn.classList.remove('ring-2', 'ring-offset-2', 'ring-amber-500');
                }
            });
        }

        function setLineWidth(width) {
            playSound('click');
            lineWidth = width;

            const btnThin = document.getElementById('sizeThin');
            const btnMed = document.getElementById('sizeMed');
            const btnThick = document.getElementById('sizeThick');

            [btnThin, btnMed, btnThick].forEach(btn => {
                btn.className = 'px-2.5 py-1 rounded-lg text-xs font-bold text-slate-600 hover:bg-white transition';
            });

            if (width === 14) btnThin.className = 'px-2.5 py-1 rounded-lg text-xs font-bold bg-white text-amber-700 shadow-sm transition';
            if (width === 22) btnMed.className = 'px-2.5 py-1 rounded-lg text-xs font-bold bg-white text-amber-700 shadow-sm transition';
            if (width === 32) btnThick.className = 'px-2.5 py-1 rounded-lg text-xs font-bold bg-white text-amber-700 shadow-sm transition';
        }

        function toggleTemplateGuide() {
            playSound('click');
            templateOpacityMode = (templateOpacityMode + 1) % opacityValues.length;
            document.getElementById('guideText').textContent = opacityLabels[templateOpacityMode];
            drawTemplateText();
        }

        function clearUserCanvas() {
            playSound('clear');
            drawCtx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
        }

        function evaluateDrawing() {
            const w = drawCanvas.width;
            const h = drawCanvas.height;

            const evalCanvas = document.createElement('canvas');
            evalCanvas.width = w;
            evalCanvas.height = h;
            const evalCtx = evalCanvas.getContext('2d');

            const fontSize = w * 0.72;
            evalCtx.font = `bold ${fontSize}px "M PLUS Rounded 1c", "Hiragino Kaku Gothic ProN", "Meiryo", sans-serif`;
            evalCtx.textAlign = 'center';
            evalCtx.textBaseline = 'middle';
            evalCtx.fillStyle = '#000000';
            evalCtx.fillText(currentChar, w / 2, h / 2 + fontSize * 0.04);

            const templateImgData = evalCtx.getImageData(0, 0, w, h).data;
            const userImgData = drawCtx.getImageData(0, 0, w, h).data;

            let templatePixelCount = 0;
            let userPixelCount = 0;
            let overlapPixelCount = 0;

            for (let i = 3; i < templateImgData.length; i += 16) {
                const hasTemplate = templateImgData[i] > 50;
                const hasUser = userImgData[i] > 50;

                if (hasTemplate) templatePixelCount++;
                if (hasUser) userPixelCount++;
                if (hasTemplate && hasUser) overlapPixelCount++;
            }

            if (userPixelCount < 30) {
                showResultModal('more');
                return;
            }

            const coverageRatio = overlapPixelCount / Math.max(templatePixelCount, 1);
            const precisionRatio = overlapPixelCount / Math.max(userPixelCount, 1);

            const totalScore = (coverageRatio * 0.65 + precisionRatio * 0.35) * 100;

            if (totalScore >= 35 || (coverageRatio > 0.4)) {
                showResultModal('perfect');
            } else if (totalScore >= 20) {
                showResultModal('good');
            } else {
                showResultModal('more');
            }
        }

        function showResultModal(grade) {
            const modal = document.getElementById('resultModal');
            const title = document.getElementById('resultTitle');
            const scoreText = document.getElementById('resultScoreText');
            const icon = document.getElementById('modalHanamaruIcon');
            const starContainer = document.getElementById('starContainer');

            modal.classList.remove('hidden');

            if (grade === 'perfect') {
                playSound('fanfare');
                triggerConfetti();
                icon.textContent = '🌸';
                title.textContent = 'たいへん よくできました！';
                title.className = 'text-2xl sm:text-3xl font-extrabold text-amber-600 mb-1';
                scoreText.textContent = `「${currentChar}」を とても じょうずに かけました！`;
                starContainer.innerHTML = `
                    <i class="fas fa-star text-amber-400"></i>
                    <i class="fas fa-star text-amber-400"></i>
                    <i class="fas fa-star text-amber-400"></i>
                `;
            } else if (grade === 'good') {
                playSound('fanfare');
                icon.textContent = '💮';
                title.textContent = 'よくできました！';
                title.className = 'text-2xl sm:text-3xl font-extrabold text-emerald-600 mb-1';
                scoreText.textContent = `いい かんじ！ もうすこしで かんぺきです！`;
                starContainer.innerHTML = `
                    <i class="fas fa-star text-amber-400"></i>
                    <i class="fas fa-star text-amber-400"></i>
                    <i class="far fa-star text-slate-300"></i>
                `;
            } else {
                playSound('click');
                icon.textContent = '✏️';
                title.textContent = 'もういちど がんばろう！';
                title.className = 'text-2xl sm:text-3xl font-extrabold text-blue-600 mb-1';
                scoreText.textContent = `おてほんを しっかり なぞってみよう！`;
                starContainer.innerHTML = `
                    <i class="fas fa-star text-amber-400"></i>
                    <i class="far fa-star text-slate-300"></i>
                    <i class="far fa-star text-slate-300"></i>
                `;
            }
        }

        function triggerConfetti() {
            if (typeof confetti === 'function') {
                confetti({
                    particleCount: 70,
                    spread: 60,
                    origin: { y: 0.6 }
                });
            }
        }

        function closeModal() {
            playSound('click');
            document.getElementById('resultModal').classList.add('hidden');
        }

        function nextCharacter() {
            closeModal();
            const list = CHAR_DATA[currentCategory];
            const currentIndex = list.indexOf(currentChar);
            const nextIndex = (currentIndex + 1) % list.length;
            selectChar(list[nextIndex]);
        }
    </script>
</body>
</html>
