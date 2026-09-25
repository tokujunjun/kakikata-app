<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>たのしく かきかた れんしゅう！</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti for celebration -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.2/dist/confetti.browser.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@400;700;800;900&display=swap');
        
        body {
            font-family: 'M PLUS Rounded 1c', 'Hiragino Kaku Gothic ProN', sans-serif;
            touch-action: manipulation;
            user-select: none;
            -webkit-user-select: none;
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

        /* Canvas layer stacking */
        .canvas-container {
            position: relative;
            width: 100%;
            max-width: 320px;
            aspect-ratio: 1 / 1;
            touch-action: none; /* スマホ描画時の画面移動を防止 */
        }
        .canvas-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border-radius: 1rem;
        }

        /* Hide scrollbar for mobile character bar */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }

        /* Rainbow pen effect gradient */
        .rainbow-bg {
            background: linear-gradient(135deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #8b00ff);
        }

        @keyframes bounce-gentle {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-4px); }
        }
        .animate-bounce-gentle {
            animation: bounce-gentle 2s infinite ease-in-out;
        }
    </style>
</head>
<body class="bg-amber-50 min-h-screen flex flex-col justify-between text-slate-800">

    <!-- Application Top Header -->
    <header class="bg-white border-b-4 border-amber-300 px-3 py-2 sm:px-6 sm:py-3 shadow-sm flex items-center justify-between sticky top-0 z-20">
        <div class="flex items-center space-x-2">
            <span class="text-2xl sm:text-3xl animate-bounce-gentle">✏️</span>
            <h1 class="text-base sm:text-2xl font-black text-amber-900 tracking-wide">
                たのしく かきかた れんしゅう！
            </h1>
        </div>
        <button onclick="speakCurrentChar()" class="bg-amber-100 hover:bg-amber-200 active:scale-95 text-amber-900 px-2.5 py-1 sm:px-3 sm:py-1.5 rounded-full text-xs sm:text-sm font-bold border border-amber-300 transition flex items-center space-x-1">
            <i class="fas fa-volume-high text-amber-600"></i>
            <span>おと</span>
        </button>
    </header>

    <main class="max-w-4xl w-full mx-auto p-2 sm:p-4 flex-grow flex flex-col items-center justify-start space-y-2 sm:space-y-4">
        
        <!-- Category Tabs (Hiragana / Katakana / Numbers) -->
        <div class="flex rounded-2xl bg-amber-200/80 p-1 w-full max-w-md shadow-inner text-sm sm:text-base">
            <button id="tabHiragana" onclick="switchCategory('hiragana')" class="flex-1 py-1.5 rounded-xl font-bold transition-all text-amber-900 bg-white shadow-sm">
                ひらがな
            </button>
            <button id="tabKatakana" onclick="switchCategory('katakana')" class="flex-1 py-1.5 rounded-xl font-bold transition-all text-amber-800 hover:bg-white/50">
                カタカナ
            </button>
            <button id="tabNumbers" onclick="switchCategory('numbers')" class="flex-1 py-1.5 rounded-xl font-bold transition-all text-amber-800 hover:bg-white/50">
                すうじ
            </button>
        </div>

        <!-- Mobile Horizontal Character Selector (Visible on mobile) -->
        <div class="w-full md:hidden bg-amber-100/90 p-2 rounded-2xl border border-amber-200 shadow-sm">
            <div class="flex justify-between items-center mb-1 px-1">
                <span class="text-xs font-bold text-amber-800"><i class="fas fa-hand-pointer mr-1"></i>もじを えらんでね</span>
                <span class="text-[10px] text-amber-600 font-bold">← よこに スライド →</span>
            </div>
            <div id="mobileCharBar" class="flex overflow-x-auto space-x-1.5 py-1 px-0.5 no-scrollbar">
                <!-- Mobile Buttons Dynamic Injection -->
            </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-12 gap-3 sm:gap-4 w-full items-start">
            
            <!-- Left Panel: Character Selection Grid (Desktop) -->
            <div class="hidden md:flex md:col-span-5 bg-white p-3 sm:p-4 rounded-3xl shadow-md border-2 border-amber-200 flex-col h-[460px]">
                <div class="flex justify-between items-center mb-2">
                    <span class="text-sm font-bold text-amber-700" id="pickerTitle">もじを えらんでね</span>
                    <button onclick="speakCurrentChar()" class="bg-amber-100 hover:bg-amber-200 text-amber-800 px-3 py-1 rounded-full text-xs font-bold transition flex items-center space-x-1">
                        <i class="fas fa-volume-high"></i>
                        <span>きく</span>
                    </button>
                </div>
                <!-- Scrollable Character Grid -->
                <div id="charGrid" class="char-grid grid grid-cols-5 gap-1.5 overflow-y-auto p-1 flex-grow">
                    <!-- Dynamic Grid Buttons -->
                </div>
            </div>

            <!-- Right Panel: Canvas & Writing Area -->
            <div class="md:col-span-7 bg-white p-2.5 sm:p-5 rounded-3xl shadow-md border-2 border-amber-200 flex flex-col items-center justify-between space-y-2.5 sm:space-y-3">
                
                <!-- Display Active Character & Speech Trigger -->
                <div class="flex items-center justify-between w-full px-1 sm:px-2">
                    <div class="flex items-center space-x-2">
                        <span class="text-xs font-bold bg-amber-100 text-amber-800 px-2.5 py-1 rounded-full" id="currentCharType">ひらがな</span>
                        <span class="text-2xl sm:text-3xl font-extrabold text-amber-600" id="currentCharDisplay">あ</span>
                    </div>
                    <button onclick="speakCurrentChar()" class="bg-amber-400 hover:bg-amber-500 active:scale-95 text-white px-3 py-1.5 rounded-2xl font-bold text-xs sm:text-sm shadow transition flex items-center space-x-1.5">
                        <i class="fas fa-bullhorn"></i>
                        <span>よみあげる</span>
                    </button>
                </div>

                <!-- Canvas Notebook Area -->
                <div id="canvasBox" class="canvas-container bg-amber-50 rounded-2xl shadow-inner border-4 border-amber-300 overflow-hidden cursor-crosshair">
                    <canvas id="bgCanvas" width="320" height="320" class="canvas-layer"></canvas>
                    <canvas id="templateCanvas" width="320" height="320" class="canvas-layer"></canvas>
                    <canvas id="drawCanvas" width="320" height="320" class="canvas-layer"></canvas>
                </div>

                <!-- Pen Tool Settings Toolbar -->
                <div class="w-full flex flex-wrap items-center justify-between gap-1.5 sm:gap-2 pt-1 border-t border-slate-100">
                    
                    <!-- Color Palette -->
                    <div class="flex items-center space-x-1 sm:space-x-1.5">
                        <button onclick="setPenColor('#1e293b', this)" class="color-btn w-7 h-7 sm:w-8 sm:h-8 rounded-full bg-slate-800 border-2 border-white shadow ring-2 ring-slate-800 transition active:scale-95" title="くろ"></button>
                        <button onclick="setPenColor('#ef4444', this)" class="color-btn w-7 h-7 sm:w-8 sm:h-8 rounded-full bg-red-500 border-2 border-white shadow transition active:scale-95" title="あか"></button>
                        <button onclick="setPenColor('#3b82f6', this)" class="color-btn w-7 h-7 sm:w-8 sm:h-8 rounded-full bg-blue-500 border-2 border-white shadow transition active:scale-95" title="あお"></button>
                        <button onclick="setPenColor('#22c55e', this)" class="color-btn w-7 h-7 sm:w-8 sm:h-8 rounded-full bg-green-500 border-2 border-white shadow transition active:scale-95" title="みどり"></button>
                        <button onclick="setPenColor('rainbow', this)" class="color-btn w-7 h-7 sm:w-8 sm:h-8 rounded-full rainbow-bg border-2 border-white shadow transition active:scale-95 text-xs text-white flex items-center justify-center font-black" title="にじいろ">✨</button>
                    </div>

                    <!-- Line Thickness Toggle -->
                    <div class="flex items-center bg-amber-100 p-0.5 rounded-xl text-xs font-bold text-amber-900">
                        <button onclick="setPenWidth(12, this)" class="width-btn px-2 py-1 rounded-lg transition bg-white shadow-sm">ほそい</button>
                        <button onclick="setPenWidth(20, this)" class="width-btn px-2 py-1 rounded-lg transition text-amber-800">ふつう</button>
                        <button onclick="setPenWidth(28, this)" class="width-btn px-2 py-1 rounded-lg transition text-amber-800">ふとい</button>
                    </div>

                    <!-- Template Guide Density Toggle -->
                    <button onclick="toggleGuideOpacity()" class="text-xs bg-slate-100 hover:bg-slate-200 text-slate-700 px-2.5 py-1 rounded-xl font-bold border border-slate-300 transition">
                        おてほん: <span id="opacityLabel">こい</span>
                    </button>
                </div>

                <!-- Bottom Action Buttons (Clear & Grade) -->
                <div class="grid grid-cols-2 gap-2 w-full pt-1">
                    <button onclick="clearUserCanvas()" class="bg-slate-100 hover:bg-slate-200 active:scale-95 text-slate-700 py-2.5 rounded-2xl font-bold text-sm shadow border border-slate-300 transition flex items-center justify-center space-x-1.5">
                        <i class="fas fa-eraser text-slate-500"></i>
                        <span>けす</span>
                    </button>
                    <button onclick="evaluateDrawing()" class="bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white py-2.5 rounded-2xl font-extrabold text-sm shadow-lg transition flex items-center justify-center space-x-1.5 border-b-4 border-emerald-700">
                        <i class="fas fa-check-circle"></i>
                        <span>できた！</span>
                    </button>
                </div>

            </div>
        </div>
    </main>

    <!-- Result / Evaluation Modal -->
    <div id="resultModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white rounded-3xl p-6 max-w-xs w-full text-center border-4 border-amber-300 shadow-2xl transform transition-all scale-100 space-y-4">
            <div id="modalHanamaru" class="text-6xl animate-bounce">🌸</div>
            <h3 id="modalTitle" class="text-xl font-black text-amber-900">たいへんよくできました！</h3>
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
        /* Character Data Definitions */
        const CHAR_DATA = {
            hiragana: [
                'あ','い','う','え','お','か','き','く','け','こ',
                'さ','し','す','せ','そ','た','ち','つ','て','と',
                'な','に','ぬ','ね','の','は','ひ','ふ','へ','ほ',
                'ま','み','む','め','も','や','ゆ','よ',
                'ら','り','る','れ','ろ','わ','を','ん'
            ],
            katakana: [
                'ア','イ','ウ','エ','オ','カ','キ','ク','ケ','コ',
                'サ','シ','ス','セ','ソ','タ','チ','ツ','テ','ト',
                'ナ','ニ','ヌ','ネ','ノ','ハ','ヒ','フ','ヘ','ホ',
                'マ','ミ','ム','メモ','ヤ','ユ','ヨ',
                'ラ','リ','ル','レ','ロ','ワ','ヲ','ン'
            ],
            numbers: ['0','1','2','3','4','5','6','7','8','9']
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

        // Canvas element setup
        const bgCanvas = document.getElementById('bgCanvas');
        const bgCtx = bgCanvas.getContext('2d');
        const templateCanvas = document.getElementById('templateCanvas');
        const templateCtx = templateCanvas.getContext('2d');
        const drawCanvas = document.getElementById('drawCanvas');
        const drawCtx = drawCanvas.getContext('2d');

        // Web Audio API Sound System
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
                    osc.frequency.setValueAtTime(523.25, now); // C5
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
                    const freqs = [523.25, 659.25, 783.99, 1046.50]; // C, E, G, C
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
            window.speechSynthesis.cancel(); // Stop ongoing speech

            // ゆっくり・はっきり読み上げるため「あ…… あ」の形式にする
            const textToSpeak = `${currentChar}…… ${currentChar}`;
            const uttr = new SpeechSynthesisUtterance(textToSpeak);
            uttr.lang = 'ja-JP';
            uttr.rate = 0.8;  // 少しゆっくり
            uttr.pitch = 1.2; // こどもに聞き取りやすいトーン
            window.speechSynthesis.speak(uttr);
        }

        function drawNotebookGrid() {
            const w = bgCanvas.width;
            const h = bgCanvas.height;

            bgCtx.clearRect(0, 0, w, h);
            
            // White Background
            bgCtx.fillStyle = '#ffffff';
            bgCtx.fillRect(0, 0, w, h);

            // Red Outer Border
            bgCtx.strokeStyle = '#f87171';
            bgCtx.lineWidth = 4;
            bgCtx.strokeRect(2, 2, w - 4, h - 4);

            // Dashed Center Cross Lines (2x2 grid)
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

            bgCtx.setLineDash([]); // Reset line dash
        }

        function drawTemplateText() {
            const w = templateCanvas.width;
            const h = templateCanvas.height;

            templateCtx.clearRect(0, 0, w, h);
            if (guideOpacity <= 0) return;

            templateCtx.fillStyle = `rgba(51, 65, 85, ${guideOpacity})`;
            templateCtx.font = 'bold 220px "M PLUS Rounded 1c", "Hiragino Kaku Gothic ProN", sans-serif';
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

            // Dot on single touch tap
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
            rainbowHue = (rainbowHue + 10) % 360;
            return `hsl(${rainbowHue}, 90%, 50%)`;
        }

        function switchCategory(cat) {
            currentCategory = cat;
            currentChar = CHAR_DATA[cat][0];

            // Tab button styles update
            ['Hiragana', 'Katakana', 'Numbers'].forEach(type => {
                const tab = document.getElementById('tab' + type);
                if (type.toLowerCase() === cat) {
                    tab.className = 'flex-1 py-1.5 rounded-xl font-bold transition-all text-amber-900 bg-white shadow-sm';
                } else {
                    tab.className = 'flex-1 py-1.5 rounded-xl font-bold transition-all text-amber-800 hover:bg-white/50';
                }
            });

            // Update label
            const labelMap = { hiragana: 'ひらがな', katakana: 'カタカナ', numbers: 'すうじ' };
            document.getElementById('currentCharType').textContent = labelMap[cat];

            buildCharGrid();
            selectChar(currentChar, false);
        }

        function buildCharGrid() {
            const grid = document.getElementById('charGrid');
            const mobileBar = document.getElementById('mobileCharBar');

            if (grid) grid.innerHTML = '';
            if (mobileBar) mobileBar.innerHTML = '';

            CHAR_DATA[currentCategory].forEach(ch => {
                // Desktop Grid Buttons
                if (grid) {
                    const btn = document.createElement('button');
                    btn.className = `aspect-square flex items-center justify-center text-xl font-bold rounded-xl transition border ${
                        ch === currentChar
                            ? 'bg-amber-400 text-white border-amber-500 shadow-md scale-105'
                            : 'bg-amber-50 hover:bg-amber-100 text-amber-900 border-amber-200'
                    }`;
                    btn.textContent = ch;
                    btn.onclick = () => {
                        playSound('click');
                        selectChar(ch, false);
                    };
                    grid.appendChild(btn);
                }

                // Mobile Horizontal Bar Buttons
                if (mobileBar) {
                    const mBtn = document.createElement('button');
                    mBtn.className = `flex-shrink-0 w-10 h-10 flex items-center justify-center text-base font-bold rounded-xl transition border ${
                        ch === currentChar
                            ? 'bg-amber-400 text-white border-amber-500 shadow-md scale-105'
                            : 'bg-white hover:bg-amber-50 text-amber-900 border-amber-200'
                    }`;
                    mBtn.textContent = ch;
                    mBtn.onclick = () => {
                        playSound('click');
                        selectChar(ch, false);
                    };
                    mobileBar.appendChild(mBtn);
                }
            });
        }

        function selectChar(ch, shouldScroll = false) {
            currentChar = ch;
            document.getElementById('currentCharDisplay').textContent = ch;

            // Desktop selection state
            const grid = document.getElementById('charGrid');
            if (grid) {
                grid.querySelectorAll('button').forEach(btn => {
                    if (btn.textContent === ch) {
                        btn.className = 'aspect-square flex items-center justify-center text-xl font-bold rounded-xl transition border bg-amber-400 text-white border-amber-500 shadow-md scale-105';
                    } else {
                        btn.className = 'aspect-square flex items-center justify-center text-xl font-bold rounded-xl transition border bg-amber-50 hover:bg-amber-100 text-amber-900 border-amber-200';
                    }
                });
            }

            // Mobile selection state
            const mobileBar = document.getElementById('mobileCharBar');
            if (mobileBar) {
                mobileBar.querySelectorAll('button').forEach(btn => {
                    if (btn.textContent === ch) {
                        btn.className = 'flex-shrink-0 w-10 h-10 flex items-center justify-center text-base font-bold rounded-xl transition border bg-amber-400 text-white border-amber-500 shadow-md scale-105';
                        btn.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' });
                    } else {
                        btn.className = 'flex-shrink-0 w-10 h-10 flex items-center justify-center text-base font-bold rounded-xl transition border bg-white hover:bg-amber-50 text-amber-900 border-amber-200';
                    }
                });
            }

            clearUserCanvas();
            drawTemplateText();
            speakCurrentChar();
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
                b.className = 'width-btn px-2 py-1 rounded-lg transition text-amber-800';
            });
            btn.className = 'width-btn px-2 py-1 rounded-lg transition bg-white shadow-sm font-bold text-amber-900';
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
            // Render hidden guide text for stroke overlap evaluation
            const evalCanvas = document.createElement('canvas');
            evalCanvas.width = 320;
            evalCanvas.height = 320;
            const evalCtx = evalCanvas.getContext('2d');

            evalCtx.fillStyle = '#000000';
            evalCtx.font = 'bold 220px "M PLUS Rounded 1c", "Hiragino Kaku Gothic ProN", sans-serif';
            evalCtx.textAlign = 'center';
            evalCtx.textBaseline = 'middle';
            evalCtx.fillText(currentChar, 160, 170);

            const guideImg = evalCtx.getImageData(0, 0, 320, 320).data;
            const userImg = drawCtx.getImageData(0, 0, 320, 320).data;

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
                // Empty stroke check
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

        // Attach Canvas Event Listeners
        drawCanvas.addEventListener('mousedown', startDrawing);
        drawCanvas.addEventListener('mousemove', draw);
        drawCanvas.addEventListener('mouseup', stopDrawing);
        drawCanvas.addEventListener('mouseleave', stopDrawing);

        drawCanvas.addEventListener('touchstart', startDrawing, { passive: false });
        drawCanvas.addEventListener('touchmove', draw, { passive: false });
        drawCanvas.addEventListener('touchend', stopDrawing);

        // Application Initialization on Window Load
        window.onload = function() {
            drawNotebookGrid();
            buildCharGrid();
            selectChar('あ', false);
        };
    </script>
</body>
</html>
