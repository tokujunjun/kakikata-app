<!DOCTYPE html>
<html lang="ja">
<head>

    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>たのしく かきかた れんしゅう！</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Canvas Confetti for celebrations -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.2/dist/confetti.browser.min.js"></script>
    
    <!-- Google Fonts: Klee One (教科書・手書き硬筆体風フォント) & Zen Maru Gothic -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Klee+One:wght@400;600&family=Zen+Maru+Gothic:wght@500;700;900&display=swap" rel="stylesheet">
    
    <style>
        /* 教科書体を最優先にしたフォントスタック設定 */
        :root {
            --font-kyokasho: 'UD デジタル 教科書体 N-R', 'UD Digital Kyokasho-tai', 'Klee One', 'Zen Maru Gothic', serif;
            --font-ui: 'Zen Maru Gothic', 'M PLUS Rounded 1c', sans-serif;
        }

        body {
            font-family: var(--font-ui);
            touch-action: manipulation;
            user-select: none;
            -webkit-user-select: none;
            background-color: #fef3c7; /* amber-100 */
        }

        .font-kyokasho {
            font-family: var(--font-kyokasho);
        }

        /* 横スクロールバーのデザイン */
        .gojuon-scroll::-webkit-scrollbar {
            height: 6px;
            width: 6px;
        }
        .gojuon-scroll::-webkit-scrollbar-track {
            background: #fef3c7;
            border-radius: 4px;
        }
        .gojuon-scroll::-webkit-scrollbar-thumb {
            background: #fcd34d;
            border-radius: 4px;
        }

        /* Canvas stacked layers */
        .canvas-container {
            position: relative;
            width: 100%;
            aspect-ratio: 1 / 1;
            touch-action: none; /* スマホなぞり書き時の画面スクロール防止 */
            margin: 0 auto;
        }
        .canvas-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border-radius: 1.25rem;
        }

        /* Rainbow pen animated gradient effect */
        .rainbow-bg {
            background: linear-gradient(135deg, #ef4444, #f97316, #eab308, #22c55e, #3b82f6, #a855f7);
        }

        /* Bounce animation for icons */
        @keyframes bounce-gentle {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-4px); }
        }
        .animate-bounce-gentle {
            animation: bounce-gentle 2.5s infinite ease-in-out;
        }

        /* Japanese traditional right-to-left layout for 50-on chart */
        .tategaki-grid {
            direction: rtl; /* 右から左へ あ行・か行...と配置 */
        }
        .tategaki-grid > * {
            direction: ltr; /* 文字自体は正位置表示 */
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between text-slate-800 pb-6">

    <!-- Sticky Header -->
    <header class="bg-white border-b-4 border-amber-300 px-4 py-2.5 shadow-sm sticky top-0 z-30 flex items-center justify-between">
        <div class="flex items-center space-x-2">
            <span class="text-xl md:text-2xl animate-bounce-gentle">✏️</span>
            <h1 class="text-base sm:text-lg md:text-xl font-black text-amber-900 tracking-wide font-kyokasho">
                たのしく かきかた れんしゅう！
            </h1>
        </div>
        <button onclick="speakCurrentChar()" class="bg-amber-100 hover:bg-amber-200 active:scale-95 text-amber-900 px-3 py-1.5 rounded-full text-xs md:text-sm font-bold border border-amber-300 transition flex items-center space-x-1.5 shadow-sm">
            <i class="fas fa-volume-high text-amber-600"></i>
            <span>おと</span>
        </button>
    </header>

    <main class="max-w-5xl w-full mx-auto px-3 sm:px-6 py-4 flex-grow flex flex-col md:flex-row gap-4 md:gap-6 items-start">
        
        <!-- LEFT COLUMN (PC/Tablet) / TOP SECTION (Mobile): Category Tabs & 50-on Chart -->
        <div class="w-full md:w-1/2 flex flex-col space-y-3">
            
            <!-- Category Switching Tabs -->
            <div class="flex rounded-2xl bg-amber-200 p-1 w-full shadow-inner text-sm md:text-base font-bold">
                <button id="tabHiragana" onclick="switchCategory('hiragana')" class="flex-1 py-2 rounded-xl transition-all text-amber-900 bg-white shadow-sm">
                    ひらがな
                </button>
                <button id="tabKatakana" onclick="switchCategory('katakana')" class="flex-1 py-2 rounded-xl transition-all text-amber-800 hover:bg-white/50">
                    カタカナ
                </button>
                <button id="tabNumbers" onclick="switchCategory('numbers')" class="flex-1 py-2 rounded-xl transition-all text-amber-800 hover:bg-white/50">
                    すうじ
                </button>
            </div>

            <!-- Vertical Japanese 50-on Character Selector (教科書体・縦書き並び) -->
            <div class="bg-white p-3 rounded-3xl border-2 border-amber-300 shadow-sm flex flex-col space-y-2">
                <div class="flex items-center justify-between text-xs sm:text-sm font-bold text-amber-800 px-1">
                    <span><i class="fas fa-hand-pointer mr-1.5 text-amber-500"></i>もじを えらんでね（たてがき）</span>
                    <span class="text-[11px] text-amber-600">← みぎ・ひだりに スライド →</span>
                </div>

                <!-- Horizontal container scrolling the vertical 50-on columns -->
                <div id="chartContainer" class="gojuon-scroll overflow-x-auto py-2 px-1">
                    <div id="gojuonGrid" class="tategaki-grid flex space-x-2 space-x-reverse min-w-max justify-center">
                        <!-- Dynamic Japanese 50-on Vertical Grid Injection -->
                    </div>
                </div>
            </div>

            <!-- Helpful tip for kids on tablet/PC -->
            <div class="hidden md:flex items-center space-x-3 bg-amber-50 border-2 border-amber-200 p-3 rounded-2xl text-amber-800 text-xs font-bold">
                <span class="text-2xl">💡</span>
                <span>ひだりの「もじ」をえらんだら、みぎのノートに おてほんを よくみて ていねいに かいてみよう！</span>
            </div>
        </div>

        <!-- RIGHT COLUMN (PC/Tablet) / BOTTOM SECTION (Mobile): Practice Canvas Area -->
        <div class="w-full md:w-1/2 bg-white p-3.5 sm:p-5 rounded-3xl shadow-md border-2 border-amber-300 flex flex-col items-center space-y-3">
            
            <!-- Target Character Display & Speech Button -->
            <div class="flex items-center justify-between w-full px-1">
                <div class="flex items-center space-x-2">
                    <span class="text-xs sm:text-sm font-extrabold bg-amber-100 text-amber-800 px-3 py-1 rounded-full" id="currentCharType">ひらがな</span>
                    <span class="text-3xl sm:text-4xl font-black text-amber-600 font-kyokasho" id="currentCharDisplay">あ</span>
                </div>
                <button onclick="speakCurrentChar()" class="bg-amber-400 hover:bg-amber-500 active:scale-95 text-white px-3.5 py-1.5 rounded-xl font-extrabold text-xs sm:text-sm shadow transition flex items-center space-x-1.5 border-b-2 border-amber-600">
                    <i class="fas fa-bullhorn"></i>
                    <span>よみあげる</span>
                </button>
            </div>

            <!-- Canvas Notebook Box (2x2 Dashed Grid) -->
            <div id="canvasBox" class="canvas-container bg-amber-50 rounded-2xl border-4 border-amber-300 shadow-inner overflow-hidden cursor-crosshair max-w-[340px] sm:max-w-[380px]">
                <canvas id="bgCanvas" width="380" height="380" class="canvas-layer"></canvas>
                <canvas id="templateCanvas" width="380" height="380" class="canvas-layer"></canvas>
                <canvas id="drawCanvas" width="380" height="380" class="canvas-layer"></canvas>
            </div>

            <!-- Pen Tools Toolbar -->
            <div class="w-full flex flex-wrap items-center justify-between gap-2 pt-1 border-t border-slate-100">
                <!-- Color Palette -->
                <div class="flex items-center space-x-1.5">
                    <button onclick="setPenColor('#1e293b', this)" class="color-btn w-8 h-8 rounded-full bg-slate-800 border-2 border-white shadow ring-2 ring-slate-800 transition active:scale-95" title="くろ"></button>
                    <button onclick="setPenColor('#ef4444', this)" class="color-btn w-8 h-8 rounded-full bg-red-500 border-2 border-white shadow transition active:scale-95" title="あか"></button>
                    <button onclick="setPenColor('#3b82f6', this)" class="color-btn w-8 h-8 rounded-full bg-blue-500 border-2 border-white shadow transition active:scale-95" title="あお"></button>
                    <button onclick="setPenColor('#22c55e', this)" class="color-btn w-8 h-8 rounded-full bg-green-500 border-2 border-white shadow transition active:scale-95" title="みどり"></button>
                    <button onclick="setPenColor('rainbow', this)" class="color-btn w-8 h-8 rounded-full rainbow-bg border-2 border-white shadow transition active:scale-95 text-xs text-white flex items-center justify-center font-black" title="にじいろ">✨</button>
                </div>

                <!-- Line Thickness Toggle -->
                <div class="flex items-center bg-amber-100 p-0.5 rounded-xl text-xs font-bold text-amber-900">
                    <button onclick="setPenWidth(12, this)" class="width-btn px-2.5 py-1 rounded-lg transition bg-white shadow-sm">ほそい</button>
                    <button onclick="setPenWidth(22, this)" class="width-btn px-2.5 py-1 rounded-lg transition text-amber-800">ふつう</button>
                    <button onclick="setPenWidth(32, this)" class="width-btn px-2.5 py-1 rounded-lg transition text-amber-800">ふとい</button>
                </div>

                <!-- Template Opacity Toggle -->
                <button onclick="toggleGuideOpacity()" class="text-xs bg-slate-100 hover:bg-slate-200 text-slate-700 px-2.5 py-1 rounded-xl font-bold border border-slate-300 transition">
                    おてほん: <span id="opacityLabel">こい</span>
                </button>
            </div>

            <!-- Action Buttons (Clear & Grade) -->
            <div class="grid grid-cols-2 gap-3 w-full pt-1">
                <button onclick="clearUserCanvas()" class="bg-slate-100 hover:bg-slate-200 active:scale-95 text-slate-700 py-3 rounded-2xl font-bold text-sm shadow border border-slate-300 transition flex items-center justify-center space-x-1.5">
                    <i class="fas fa-eraser text-slate-500"></i>
                    <span>けす</span>
                </button>
                <button onclick="evaluateDrawing()" class="bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white py-3 rounded-2xl font-extrabold text-sm sm:text-base shadow-lg transition flex items-center justify-center space-x-1.5 border-b-4 border-emerald-700">
                    <i class="fas fa-check-circle"></i>
                    <span>できた！</span>
                </button>
            </div>

        </div>
    </main>

    <!-- Result / Evaluation Modal -->
    <div id="resultModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white rounded-3xl p-6 max-w-xs w-full text-center border-4 border-amber-300 shadow-2xl space-y-3">
            <div id="modalHanamaru" class="text-6xl animate-bounce">🌸</div>
            <h3 id="modalTitle" class="text-xl font-black text-amber-900 font-kyokasho">たいへんよくできました！</h3>
            <p id="modalScore" class="text-amber-700 font-bold text-sm">ひらがな「あ」のかきかた</p>
            <div id="modalStars" class="text-2xl text-amber-400">⭐⭐⭐</div>
            
            <div class="pt-2">
                <button onclick="closeModal()" class="w-full bg-amber-400 hover:bg-amber-500 active:scale-95 text-white font-extrabold py-3 rounded-2xl shadow-md transition border-b-4 border-amber-600">
                    つぎも がんばる！
                </button>
            </div>
        </div>
    </div>

    <script>
        /* Traditional Japanese 50-On Vertical Column Data Matrix (教科書と同じ 右から「あ行」「か行」...の構造) */
        const GOJUON_MATRIX = {
            hiragana: [
                ['あ', 'か', 'さ', 'た', 'な', 'は', 'ま', 'や', 'ら', 'わ'],
                ['い', 'き', 'し', 'ち', 'に', 'ひ', 'み', ' ', 'り', ' '],
                ['う', 'く', 'す', 'つ', 'ぬ', 'ふ', 'む', 'ゆ', 'る', 'ん'],
                ['え', 'け', 'せ', 'て', 'ね', 'へ', 'め', ' ', 'れ', ' '],
                ['お', 'こ', 'そ', 'と', 'の', 'ほ', 'も', 'よ', 'ろ', 'を']
            ],
            katakana: [
                ['ア', 'カ', 'サ', 'タ', 'ナ', 'ハ', 'マ', 'ヤ', 'ラ', 'ワ'],
                ['イ', 'キ', 'シ', 'チ', 'ニ', 'ヒ', 'ミ', ' ', 'リ', ' '],
                ['ウ', 'ク', 'ス', 'ツ', 'ヌ', 'フ', 'ム', 'ユ', 'ル', 'ン'],
                ['エ', 'ケ', 'セ', 'テ', 'ネ', 'ヘ', 'メ', ' ', 'レ', ' '],
                ['オ', 'コ', 'ソ', 'ト', 'ノ', 'ホ', 'モ', 'ヨ', 'ロ', 'ヲ']
            ],
            numbers: [
                ['1', '2', '3', '4', '5'],
                ['6', '7', '8', '9', '0']
            ]
        };

        let currentCategory = 'hiragana';
        let currentChar = 'あ';
        let guideOpacity = 0.35; // こい(0.35), うすい(0.15), なし(0)
        let penColor = '#1e293b';
        let penWidth = 12;
        let isRainbow = false;
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;

        // Canvas context initialization
        const bgCanvas = document.getElementById('bgCanvas');
        const bgCtx = bgCanvas.getContext('2d');
        const templateCanvas = document.getElementById('templateCanvas');
        const templateCtx = templateCanvas.getContext('2d');
        const drawCanvas = document.getElementById('drawCanvas');
        const drawCtx = drawCanvas.getContext('2d');

        // Web Audio API Sound Generator
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        let audioCtx = null;

        function playSound(type) {
            try {
                if (!audioCtx) audioCtx = new AudioContext();
                if (audioCtx.state === 'suspended') audioCtx.resume();

                const now = audioCtx.currentTime;
                if (type === 'click') {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(523.25, now);
                    osc.frequency.exponentialRampToValueAtTime(880, now + 0.08);
                    gain.gain.setValueAtTime(0.15, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.08);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now);
                    osc.stop(now + 0.08);
                } else if (type === 'clear') {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(440, now);
                    osc.frequency.exponentialRampToValueAtTime(220, now + 0.15);
                    gain.gain.setValueAtTime(0.2, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now);
                    osc.stop(now + 0.15);
                } else if (type === 'fanfare') {
                    const freqs = [523.25, 659.25, 783.99, 1046.50];
                    freqs.forEach((freq, idx) => {
                        const osc = audioCtx.createOscillator();
                        const gain = audioCtx.createGain();
                        osc.type = 'triangle';
                        osc.frequency.setValueAtTime(freq, now + idx * 0.08);
                        gain.gain.setValueAtTime(0.25, now + idx * 0.08);
                        gain.gain.exponentialRampToValueAtTime(0.01, now + idx * 0.08 + 0.3);
                        osc.connect(gain);
                        gain.connect(audioCtx.destination);
                        osc.start(now + idx * 0.08);
                        osc.stop(now + idx * 0.08 + 0.3);
                    });
                }
            } catch (e) {
                console.log('Audio Context Error:', e);
            }
        }

        function speakCurrentChar() {
            if (!('speechSynthesis' in window)) return;
            window.speechSynthesis.cancel(); // Stop active speech

            // 「あ…… あ」の形式でゆっくりはっきり2回発音
            const textToSpeak = `${currentChar}…… ${currentChar}`;
            const uttr = new SpeechSynthesisUtterance(textToSpeak);
            uttr.lang = 'ja-JP';
            uttr.rate = 0.8;
            uttr.pitch = 1.2;
            window.speechSynthesis.speak(uttr);
        }

        function drawNotebookGrid() {
            const w = bgCanvas.width;
            const h = bgCanvas.height;

            bgCtx.clearRect(0, 0, w, h);
            
            // White page background
            bgCtx.fillStyle = '#ffffff';
            bgCtx.fillRect(0, 0, w, h);

            // Red notebook border
            bgCtx.strokeStyle = '#f87171';
            bgCtx.lineWidth = 4;
            bgCtx.strokeRect(2, 2, w - 4, h - 4);

            // Dashed center cross lines (2x2 grid)
            bgCtx.strokeStyle = '#fca5a5';
            bgCtx.lineWidth = 2;
            bgCtx.setLineDash([8, 6]);

            // Vertical centerline
            bgCtx.beginPath();
            bgCtx.moveTo(w / 2, 0);
            bgCtx.lineTo(w / 2, h);
            bgCtx.stroke();

            // Horizontal centerline
            bgCtx.beginPath();
            bgCtx.moveTo(0, h / 2);
            bgCtx.lineTo(w, h / 2);
            bgCtx.stroke();

            bgCtx.setLineDash([]);
        }

        function drawTemplateText() {
            const w = templateCanvas.width;
            const h = templateCanvas.height;

            templateCtx.clearRect(0, 0, w, h);
            if (guideOpacity <= 0) return;

            templateCtx.fillStyle = `rgba(51, 65, 85, ${guideOpacity})`;
            // 教科書体（UDデジタル教科書体 / Klee One）でお手本を描画
            templateCtx.font = '600 260px "UD デジタル 教科書体 N-R", "UD Digital Kyokasho-tai", "Klee One", serif';
            templateCtx.textAlign = 'center';
            templateCtx.textBaseline = 'middle';
            templateCtx.fillText(currentChar, w / 2, h / 2 + 10);
        }

        function getCanvasCoordinates(e) {
            const rect = drawCanvas.getBoundingClientRect();
            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            
            const scaleX = drawCanvas.width / rect.width;
            const scaleY = drawCanvas.height / rect.height;

            return {
                x: (clientX - rect.left) * scaleX,
                y: (clientY - rect.top) * scaleY
            };
        }

        function startDrawing(e) {
            isDrawing = true;
            const coords = getCanvasCoordinates(e);
            lastX = coords.x;
            lastY = coords.y;

            drawCtx.beginPath();
            drawCtx.arc(lastX, lastY, penWidth / 2, 0, Math.PI * 2);
            drawCtx.fillStyle = isRainbow ? getRainbowColor() : penColor;
            drawCtx.fill();
        }

        function draw(e) {
            if (!isDrawing) return;
            e.preventDefault();

            const coords = getCanvasCoordinates(e);

            drawCtx.beginPath();
            drawCtx.moveTo(lastX, lastY);
            drawCtx.lineTo(coords.x, coords.y);
            drawCtx.strokeStyle = isRainbow ? getRainbowColor() : penColor;
            drawCtx.lineWidth = penWidth;
            drawCtx.lineCap = 'round';
            drawCtx.lineJoin = 'round';
            drawCtx.stroke();

            lastX = coords.x;
            lastY = coords.y;
        }

        function stopDrawing() {
            isDrawing = false;
        }

        let rainbowHue = 0;
        function getRainbowColor() {
            rainbowHue = (rainbowHue + 12) % 360;
            return `hsl(${rainbowHue}, 90%, 50%)`;
        }

        function switchCategory(cat) {
            currentCategory = cat;

            // Category Tab button active styling
            ['Hiragana', 'Katakana', 'Numbers'].forEach(type => {
                const tab = document.getElementById('tab' + type);
                if (type.toLowerCase() === cat) {
                    tab.className = 'flex-1 py-2 rounded-xl transition-all text-amber-900 bg-white shadow-sm font-bold';
                } else {
                    tab.className = 'flex-1 py-2 rounded-xl transition-all text-amber-800 hover:bg-white/50 font-bold';
                }
            });

            const labelMap = { hiragana: 'ひらがな', katakana: 'カタカナ', numbers: 'すうじ' };
            document.getElementById('currentCharType').textContent = labelMap[cat];

            buildGojuonGrid();
            // Select default first character
            const firstChar = cat === 'numbers' ? '1' : (cat === 'katakana' ? 'ア' : 'あ');
            selectChar(firstChar);
        }

        function buildGojuonGrid() {
            const grid = document.getElementById('gojuonGrid');
            grid.innerHTML = '';

            const matrix = GOJUON_MATRIX[currentCategory];
            const numCols = matrix[0].length;

            // Construct vertical columns (あ行, か行, さ行...) in Textbook Font
            for (let col = 0; col < numCols; col++) {
                const colDiv = document.createElement('div');
                colDiv.className = 'flex flex-col space-y-1.5 items-center';

                for (let row = 0; row < matrix.length; row++) {
                    const ch = matrix[row][col];
                    if (!ch || ch === ' ') {
                        // Empty spacer cell
                        const spacer = document.createElement('div');
                        spacer.className = 'w-10 h-10';
                        colDiv.appendChild(spacer);
                    } else {
                        const btn = document.createElement('button');
                        btn.className = `w-10 h-10 flex items-center justify-center text-lg font-bold rounded-xl transition border font-kyokasho ${
                            ch === currentChar
                                ? 'bg-amber-400 text-white border-amber-500 shadow-md scale-105'
                                : 'bg-amber-50 hover:bg-amber-100 text-amber-900 border-amber-200'
                        }`;
                        btn.textContent = ch;
                        btn.onclick = () => {
                            playSound('click');
                            selectChar(ch);
                        };
                        colDiv.appendChild(btn);
                    }
                }
                grid.appendChild(colDiv);
            }
        }

        function selectChar(ch) {
            currentChar = ch;
            document.getElementById('currentCharDisplay').textContent = ch;

            // Update button active state
            const grid = document.getElementById('gojuonGrid');
            grid.querySelectorAll('button').forEach(btn => {
                if (btn.textContent === ch) {
                    btn.className = 'w-10 h-10 flex items-center justify-center text-lg font-bold rounded-xl transition border font-kyokasho bg-amber-400 text-white border-amber-500 shadow-md scale-105';
                    btn.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' });
                } else {
                    btn.className = 'w-10 h-10 flex items-center justify-center text-lg font-bold rounded-xl transition border font-kyokasho bg-amber-50 hover:bg-amber-100 text-amber-900 border-amber-200';
                }
            });

            clearUserCanvas();
            drawTemplateText();
            speakCurrentChar();

            // Auto-scroll down smoothly to canvas practice area on mobile screens
            const canvasBox = document.getElementById('canvasBox');
            if (canvasBox && window.innerWidth < 768) {
                canvasBox.scrollIntoView({ behavior: 'smooth', block: 'center' });
            }
        }

        function setPenColor(color, btn) {
            playSound('click');
            if (color === 'rainbow') {
                isRainbow = true;
            } else {
                isRainbow = false;
                penColor = color;
            }

            document.querySelectorAll('.color-btn').forEach(b => {
                b.classList.remove('ring-2', 'ring-slate-800', 'scale-110');
            });
            btn.classList.add('ring-2', 'ring-slate-800', 'scale-110');
        }

        function setPenWidth(w, btn) {
            playSound('click');
            penWidth = w;
            document.querySelectorAll('.width-btn').forEach(b => {
                b.className = 'width-btn px-2.5 py-1 rounded-lg transition text-amber-800';
            });
            btn.className = 'width-btn px-2.5 py-1 rounded-lg transition bg-white shadow-sm font-bold text-amber-900';
        }

        function toggleGuideOpacity() {
            playSound('click');
            if (guideOpacity === 0.35) {
                guideOpacity = 0.15;
                document.getElementById('opacityLabel').textContent = 'うすい';
            } else if (guideOpacity === 0.15) {
                guideOpacity = 0;
                document.getElementById('opacityLabel').textContent = 'なし';
            } else {
                guideOpacity = 0.35;
                document.getElementById('opacityLabel').textContent = 'こい';
            }
            drawTemplateText();
        }

        function clearUserCanvas() {
            playSound('clear');
            drawCtx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
        }

        function evaluateDrawing() {
            // Render template character to an offline canvas for overlap evaluation
            const evalCanvas = document.createElement('canvas');
            evalCanvas.width = 380;
            evalCanvas.height = 380;
            const evalCtx = evalCanvas.getContext('2d');

            evalCtx.fillStyle = '#000000';
            evalCtx.font = '600 260px "UD デジタル 教科書体 N-R", "UD Digital Kyokasho-tai", "Klee One", serif';
            evalCtx.textAlign = 'center';
            evalCtx.textBaseline = 'middle';
            evalCtx.fillText(currentChar, 190, 200);

            const guideImg = evalCtx.getImageData(0, 0, 380, 380).data;
            const userImg = drawCtx.getImageData(0, 0, 380, 380).data;

            let guidePixels = 0;
            let matchPixels = 0;
            let userPixels = 0;

            for (let i = 0; i < guideImg.length; i += 4) {
                const guideAlpha = guideImg[i + 3];
                const userAlpha = userImg[i + 3];

                if (guideAlpha > 50) guidePixels++;
                if (userAlpha > 50) {
                    userPixels++;
                    if (guideAlpha > 50) matchPixels++;
                }
            }

            if (userPixels < 50) {
                showModal('もういちど かいてみよう！', 'まだ なにも かかれていないよ ✏️', '⭐', '🌸');
                return;
            }

            const coverage = matchPixels / (guidePixels || 1);
            const accuracy = matchPixels / (userPixels || 1);
            const score = Math.round((coverage * 0.7 + accuracy * 0.3) * 100);

            if (score >= 40) {
                playSound('fanfare');
                confetti({
                    particleCount: 80,
                    spread: 70,
                    origin: { y: 0.6 }
                });

                if (score >= 65) {
                    showModal('たいへんよくできました！', `「${currentChar}」の かきかた ばっちり！`, '⭐⭐⭐', '💮');
                } else {
                    showModal('よくできました！', `「${currentChar}」が きれいに かけたね！`, '⭐⭐', '🌸');
                }
            } else {
                playSound('click');
                showModal('あとすこし！', 'おてほんを よくみて もういちど かいてみよう！', '⭐', '✏️');
            }
        }

        function showModal(title, subtitle, stars, icon) {
            document.getElementById('modalTitle').textContent = title;
            document.getElementById('modalScore').textContent = subtitle;
            document.getElementById('modalStars').textContent = stars;
            document.getElementById('modalHanamaru').textContent = icon;
            document.getElementById('resultModal').classList.remove('hidden');
        }

        function closeModal() {
            playSound('click');
            document.getElementById('resultModal').classList.add('hidden');
            clearUserCanvas();
        }

        // Canvas touch & mouse event listeners
        drawCanvas.addEventListener('mousedown', startDrawing);
        drawCanvas.addEventListener('mousemove', draw);
        drawCanvas.addEventListener('mouseup', stopDrawing);
        drawCanvas.addEventListener('mouseleave', stopDrawing);

        drawCanvas.addEventListener('touchstart', startDrawing, { passive: false });
        drawCanvas.addEventListener('touchmove', draw, { passive: false });
        drawCanvas.addEventListener('touchend', stopDrawing);

        // Application initialization on window load
        window.onload = function() {
            drawNotebookGrid();
            buildGojuonGrid();
            selectChar('あ');
        };
    </script>
</body>
</html>
