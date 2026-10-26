<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Modern Tic-Tac-Toe Deluxe</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Canvas Confetti library for win animations -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Inter Font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    animation: {
                        'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
                        'glow-x': 'glowX 1.5s ease-in-out infinite alternate',
                        'glow-o': 'glowO 1.5s ease-in-out infinite alternate',
                    },
                    keyframes: {
                        glowX: {
                            '0%': { filter: 'drop-shadow(0 0 2px rgba(239, 68, 68, 0.6))' },
                            '100%': { filter: 'drop-shadow(0 0 15px rgba(239, 68, 68, 0.9))' },
                        },
                        glowO: {
                            '0%': { filter: 'drop-shadow(0 0 2px rgba(59, 130, 246, 0.6))' },
                            '100%': { filter: 'drop-shadow(0 0 15px rgba(59, 130, 246, 0.9))' },
                        }
                    }
                }
            }
        }
    </script>
    <style>
        /* Custom dynamic background gradient and theme transitions */
        body {
            transition: background 0.5s ease, color 0.5s ease;
            user-select: none;
            -webkit-user-select: none;
        }

        /* Themes setup */
        /* Theme 1: Neon Cyberpunk (Default) */
        body.theme-cyberpunk {
            background: radial-gradient(circle at center, #111827, #030712);
            color: #f3f4f6;
        }
        .theme-cyberpunk .cell-btn {
            background-color: rgba(31, 41, 55, 0.6);
            border-color: rgba(75, 85, 99, 0.4);
            backdrop-filter: blur(8px);
        }
        .theme-cyberpunk .cell-btn:hover {
            background-color: rgba(55, 65, 81, 0.8);
        }

        /* Theme 2: Minimalist Dark */
        body.theme-minimalist {
            background: linear-gradient(135deg, #18181b, #09090b);
            color: #e4e4e7;
        }
        .theme-minimalist .cell-btn {
            background-color: #27272a;
            border-color: #3f3f46;
        }
        .theme-minimalist .cell-btn:hover {
            background-color: #3f3f46;
        }

        /* Theme 3: Classic Vibrant */
        body.theme-classic {
            background: linear-gradient(135deg, #0f172a, #1e1b4b);
            color: #f8fafc;
        }
        .theme-classic .cell-btn {
            background-color: rgba(30, 41, 59, 0.7);
            border-color: #334155;
        }
        .theme-classic .cell-btn:hover {
            background-color: rgba(51, 65, 85, 0.8);
        }

        /* Grid SVG Line overlay styling */
        .winning-line {
            stroke-dasharray: 1000;
            stroke-dashoffset: 1000;
            animation: drawLine 0.5s forwards ease-in-out;
        }

        @keyframes drawLine {
            to {
                stroke-dashoffset: 0;
            }
        }

        /* Cell Marker Pop Animation */
        .marker-pop {
            animation: popIn 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }

        @keyframes popIn {
            0% { transform: scale(0) rotate(-10deg); opacity: 0; }
            100% { transform: scale(1) rotate(0deg); opacity: 1; }
        }
    </style>
</head>
<body class="theme-cyberpunk min-h-screen flex flex-col items-center justify-between p-4 font-sans overflow-x-hidden">

    <!-- Header Section -->
    <header class="w-full max-w-md flex justify-between items-center py-3 px-2">
        <div class="flex items-center gap-2">
            <i class="fa-solid font-bold text-2xl text-cyan-400 fa-table-cells text-red-500"></i>
            <h1 class="text-2xl font-extrabold tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-red-500 via-purple-500 to-blue-500">
                TIC TAC TOE
            </h1>
        </div>
        <div class="flex items-center gap-2">
            <!-- Audio Toggle Button -->
            <button id="soundToggleBtn" class="p-2.5 rounded-xl bg-gray-800/80 hover:bg-gray-700 border border-gray-700 text-gray-300 transition-all duration-200" title="Toggle Sound">
                <i id="soundIcon" class="fa-solid fa-volume-high text-lg"></i>
            </button>
            <!-- Settings Modal Toggle -->
            <button id="settingsBtn" class="p-2.5 rounded-xl bg-gray-800/80 hover:bg-gray-700 border border-gray-700 text-gray-300 transition-all duration-200" title="Settings">
                <i class="fa-solid fa-gear text-lg"></i>
            </button>
        </div>
    </header>

    <!-- Main Container -->
    <main class="w-full max-w-md flex flex-col items-center gap-5 my-auto">

        <!-- Game Mode Selection Bar -->
        <div class="w-full bg-gray-900/70 p-1.5 rounded-2xl border border-gray-800 flex gap-1 shadow-lg backdrop-blur-md">
            <button id="modePvAI" class="flex-1 py-2 px-3 rounded-xl font-semibold text-sm transition-all duration-200 flex items-center justify-center gap-2 bg-indigo-600 text-white shadow">
                <i class="fa-solid fa-robot"></i> Solo vs AI
            </button>
            <button id="modePvP" class="flex-1 py-2 px-3 rounded-xl font-semibold text-sm transition-all duration-200 flex items-center justify-center gap-2 text-gray-400 hover:text-white">
                <i class="fa-solid fa-users"></i> 2 Players
            </button>
        </div>

        <!-- AI Difficulty Options (Visible in Solo Mode) -->
        <div id="aiDifficultyContainer" class="w-full flex items-center justify-between bg-gray-900/40 px-4 py-2 rounded-xl border border-gray-800/60">
            <span class="text-xs font-semibold text-gray-400 uppercase tracking-wider">AI Difficulty</span>
            <div class="flex gap-1">
                <button data-diff="easy" class="diff-btn px-2.5 py-1 text-xs font-semibold rounded-lg text-gray-400 hover:text-white transition-all">Easy</button>
                <button data-diff="medium" class="diff-btn px-2.5 py-1 text-xs font-semibold rounded-lg bg-indigo-600/80 text-white transition-all">Medium</button>
                <button data-diff="unbeatable" class="diff-btn px-2.5 py-1 text-xs font-semibold rounded-lg text-gray-400 hover:text-white transition-all">Unbeatable</button>
            </div>
        </div>

        <!-- Status & Turn Display -->
        <div class="w-full text-center">
            <div id="statusBadge" class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-gray-800/80 border border-gray-700/80 shadow-md">
                <span id="turnIndicatorDot" class="w-3 h-3 rounded-full bg-red-500 animate-pulse"></span>
                <span id="statusText" class="text-sm font-semibold tracking-wide text-gray-200">Player X's Turn</span>
            </div>
        </div>

        <!-- Game Board Card -->
        <div class="relative w-full aspect-square max-w-[360px] p-3 rounded-3xl bg-gray-900/60 border border-gray-800 shadow-2xl backdrop-blur-xl">
            <!-- Grid Container -->
            <div id="board" class="grid grid-cols-3 grid-rows-3 gap-2.5 w-full h-full relative">
                <!-- 9 Cells dynamically constructed or hardcoded for absolute reliability -->
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="0" aria-label="Cell 1"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="1" aria-label="Cell 2"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="2" aria-label="Cell 3"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="3" aria-label="Cell 4"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="4" aria-label="Cell 5"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="5" aria-label="Cell 6"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="6" aria-label="Cell 7"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="7" aria-label="Cell 8"></button>
                <button class="cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none" data-index="8" aria-label="Cell 9"></button>

                <!-- SVG Overlay for Win Line Animation -->
                <svg id="winningLineSvg" class="absolute inset-0 w-full h-full pointer-events-none z-10 hidden" viewBox="0 0 100 100">
                    <line id="winningLine" x1="0" y1="0" x2="0" y2="0" stroke="url(#lineGradient)" stroke-width="6" stroke-linecap="round" class="winning-line"/>
                    <defs>
                        <linearGradient id="lineGradient" x1="0%" y1="0%" x2="100%" y2="100%">
                            <stop offset="0%" stop-color="#ef4444" />
                            <stop offset="100%" stop-color="#3b82f6" />
                        </linearGradient>
                    </defs>
                </svg>
            </div>
        </div>

        <!-- Score Board -->
        <div class="w-full grid grid-cols-3 gap-3">
            <div class="bg-gray-900/60 border border-gray-800/80 rounded-2xl p-3 text-center shadow-lg">
                <div class="text-xs font-bold text-red-400 uppercase tracking-wider mb-1 flex items-center justify-center gap-1">
                    <span id="scoreLabelX">Player X</span>
                </div>
                <div id="scoreX" class="text-2xl font-black text-gray-100">0</div>
            </div>
            <div class="bg-gray-900/60 border border-gray-800/80 rounded-2xl p-3 text-center shadow-lg">
                <div class="text-xs font-bold text-gray-400 uppercase tracking-wider mb-1">Ties</div>
                <div id="scoreTies" class="text-2xl font-black text-gray-100">0</div>
            </div>
            <div class="bg-gray-900/60 border border-gray-800/80 rounded-2xl p-3 text-center shadow-lg">
                <div class="text-xs font-bold text-blue-400 uppercase tracking-wider mb-1 flex items-center justify-center gap-1">
                    <span id="scoreLabelO">AI (O)</span>
                </div>
                <div id="scoreO" class="text-2xl font-black text-gray-100">0</div>
            </div>
        </div>

        <!-- Action Controls -->
        <div class="w-full flex gap-3">
            <button id="restartBtn" class="flex-1 py-3 px-4 bg-indigo-600 hover:bg-indigo-500 active:scale-95 text-white font-bold rounded-2xl shadow-lg transition-all duration-200 flex items-center justify-center gap-2">
                <i class="fa-solid fa-rotate-right"></i> Reset Match
            </button>
            <button id="resetScoreBtn" class="py-3 px-4 bg-gray-800/80 hover:bg-gray-700 active:scale-95 text-gray-300 font-semibold rounded-2xl border border-gray-700 transition-all duration-200" title="Reset All Scores">
                <i class="fa-solid fa-trash-can"></i>
            </button>
        </div>

    </main>

    <!-- Footer -->
    <footer class="mt-6 text-center text-xs text-gray-500">
        Crafted for Web • Sound Synthesized with Web Audio API
    </footer>

    <!-- Settings Modal -->
    <div id="settingsModal" class="fixed inset-0 bg-black/70 backdrop-blur-sm z-50 flex items-center justify-center p-4 opacity-0 pointer-events-none transition-opacity duration-300">
        <div class="bg-gray-900 border border-gray-800 rounded-3xl p-6 w-full max-w-sm shadow-2xl transform scale-95 transition-transform duration-300" id="settingsModalContent">
            <div class="flex justify-between items-center mb-5">
                <h3 class="text-lg font-bold text-white flex items-center gap-2">
                    <i class="fa-solid fa-sliders text-indigo-400"></i> Settings
                </h3>
                <button id="closeSettingsBtn" class="text-gray-400 hover:text-white p-1">
                    <i class="fa-solid fa-xmark text-xl"></i>
                </button>
            </div>

            <div class="space-y-5">
                <!-- Theme Selection -->
                <div>
                    <label class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">Visual Theme</label>
                    <div class="grid grid-cols-3 gap-2">
                        <button data-theme="theme-cyberpunk" class="theme-select-btn p-2 rounded-xl border border-indigo-500 bg-gray-800 text-xs font-semibold text-white flex flex-col items-center gap-1">
                            <span class="w-4 h-4 rounded-full bg-gradient-to-r from-red-500 to-blue-500"></span>
                            Cyberpunk
                        </button>
                        <button data-theme="theme-minimalist" class="theme-select-btn p-2 rounded-xl border border-gray-700 bg-gray-800 text-xs font-semibold text-gray-400 flex flex-col items-center gap-1">
                            <span class="w-4 h-4 rounded-full bg-zinc-600"></span>
                            Minimal
                        </button>
                        <button data-theme="theme-classic" class="theme-select-btn p-2 rounded-xl border border-gray-700 bg-gray-800 text-xs font-semibold text-gray-400 flex flex-col items-center gap-1">
                            <span class="w-4 h-4 rounded-full bg-slate-700"></span>
                            Classic
                        </button>
                    </div>
                </div>

                <!-- Symbol Style Selection -->
                <div>
                    <label class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">Symbol Set</label>
                    <select id="symbolSelect" class="w-full bg-gray-800 border border-gray-700 text-white text-sm rounded-xl p-2.5 focus:outline-none focus:border-indigo-500">
                        <option value="classic">Standard (X & O)</option>
                        <option value="neon">Glow Symbols (✖ & ◯)</option>
                        <option value="emoji">Emoji (🔥 & ❄️)</option>
                        <option value="tech">Tech (⚡ & 🛡️)</option>
                    </select>
                </div>

                <!-- First Player Selection -->
                <div>
                    <label class="block text-xs font-semibold text-gray-400 uppercase tracking-wider mb-2">First Player (Solo)</label>
                    <div class="grid grid-cols-2 gap-2">
                        <button id="firstPlayerX" class="first-player-btn p-2 rounded-xl border border-indigo-500 bg-indigo-600/30 text-xs font-semibold text-white">Player X First</button>
                        <button id="firstPlayerO" class="first-player-btn p-2 rounded-xl border border-gray-700 bg-gray-800 text-xs font-semibold text-gray-400">AI (O) First</button>
                    </div>
                </div>
            </div>

            <button id="saveSettingsBtn" class="w-full mt-6 py-3 bg-indigo-600 hover:bg-indigo-500 text-white font-bold rounded-xl shadow-lg transition-all">
                Apply & Save
            </button>
        </div>
    </div>

    <!-- JavaScript Logic -->
    <script>
        /* ===================================================================
           SOUND SYNTHESIZER (Web Audio API)
           =================================================================== */
        class SoundFX {
            constructor() {
                this.ctx = null;
                this.enabled = true;
            }

            init() {
                if (!this.ctx) {
                    const AudioContext = window.AudioContext || window.webkitAudioContext;
                    if (AudioContext) {
                        this.ctx = new AudioContext();
                    }
                }
            }

            playTone(freq, type, duration, startVol = 0.1, endVol = 0) {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();

                    osc.type = type;
                    osc.frequency.setValueAtTime(freq, this.ctx.currentTime);

                    gain.gain.setValueAtTime(startVol, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(Math.max(endVol, 0.0001), this.ctx.currentTime + duration);

                    osc.connect(gain);
                    gain.connect(this.ctx.destination);

                    osc.start();
                    osc.stop(this.ctx.currentTime + duration);
                } catch (e) {
                    console.error("Audio playback error:", e);
                }
            }

            playX() {
                // High crisp pop sound for X
                this.playTone(480, 'sine', 0.12, 0.2);
            }

            playO() {
                // Warm resonant tone for O
                this.playTone(320, 'triangle', 0.15, 0.25);
            }

            playWin() {
                if (!this.enabled) return;
                this.init();
                // Simple cheerful arpeggio
                const notes = [440, 554.37, 659.25, 880];
                notes.forEach((freq, index) => {
                    setTimeout(() => {
                        this.playTone(freq, 'sine', 0.2, 0.2);
                    }, index * 90);
                });
            }

            playTie() {
                // Descending low tone for tie
                this.playTone(220, 'sawtooth', 0.2, 0.15);
                setTimeout(() => this.playTone(180, 'sawtooth', 0.3, 0.15), 150);
            }

            playClick() {
                this.playTone(600, 'sine', 0.05, 0.05);
            }
        }

        const sound = new SoundFX();

        /* ===================================================================
           GAME STATE & CONFIGURATION
           =================================================================== */
        const WINNING_COMBINATIONS = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8], // Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8], // Columns
            [0, 4, 8], [2, 4, 6]             // Diagonals
        ];

        const SYMBOL_SETS = {
            classic: { X: 'X', O: 'O' },
            neon: { X: '✖', O: '◯' },
            emoji: { X: '🔥', O: '❄️' },
            tech: { X: '⚡', O: '🛡️' }
        };

        let gameState = {
            board: Array(9).fill(null),
            currentPlayer: 'X',
            gameActive: true,
            mode: 'pvai', // 'pvai' or 'pvp'
            difficulty: 'medium', // 'easy', 'medium', 'unbeatable'
            firstPlayer: 'X', // 'X' or 'O'
            theme: 'theme-cyberpunk',
            symbolSet: 'classic',
            scores: { X: 0, O: 0, ties: 0 },
            soundEnabled: true
        };

        /* ===================================================================
           DOM ELEMENTS
           =================================================================== */
        const cellElements = document.querySelectorAll('.cell-btn');
        const statusText = document.getElementById('statusText');
        const turnIndicatorDot = document.getElementById('turnIndicatorDot');
        const scoreXEl = document.getElementById('scoreX');
        const scoreOEl = document.getElementById('scoreO');
        const scoreTiesEl = document.getElementById('scoreTies');
        const scoreLabelX = document.getElementById('scoreLabelX');
        const scoreLabelO = document.getElementById('scoreLabelO');

        const modePvAIBtn = document.getElementById('modePvAI');
        const modePvPBtn = document.getElementById('modePvP');
        const diffContainer = document.getElementById('aiDifficultyContainer');
        const diffBtns = document.querySelectorAll('.diff-btn');

        const restartBtn = document.getElementById('restartBtn');
        const resetScoreBtn = document.getElementById('resetScoreBtn');
        const soundToggleBtn = document.getElementById('soundToggleBtn');
        const soundIcon = document.getElementById('soundIcon');

        const settingsBtn = document.getElementById('settingsBtn');
        const settingsModal = document.getElementById('settingsModal');
        const settingsModalContent = document.getElementById('settingsModalContent');
        const closeSettingsBtn = document.getElementById('closeSettingsBtn');
        const saveSettingsBtn = document.getElementById('saveSettingsBtn');
        const symbolSelect = document.getElementById('symbolSelect');
        const themeBtns = document.querySelectorAll('.theme-select-btn');
        const firstPlayerBtns = document.querySelectorAll('.first-player-btn');

        const winningLineSvg = document.getElementById('winningLineSvg');
        const winningLine = document.getElementById('winningLine');

        /* ===================================================================
           LOCAL STORAGE & INITIALIZATION
           =================================================================== */
        function loadSavedState() {
            const savedScores = localStorage.getItem('ttt_scores');
            if (savedScores) {
                try { gameState.scores = JSON.parse(savedScores); } catch(e){}
            }

            const savedSettings = localStorage.getItem('ttt_settings');
            if (savedSettings) {
                try {
                    const settings = JSON.parse(savedSettings);
                    gameState.theme = settings.theme || 'theme-cyberpunk';
                    gameState.symbolSet = settings.symbolSet || 'classic';
                    gameState.soundEnabled = settings.soundEnabled ?? true;
                    gameState.difficulty = settings.difficulty || 'medium';
                    gameState.mode = settings.mode || 'pvai';
                    gameState.firstPlayer = settings.firstPlayer || 'X';
                } catch(e){}
            }
        }

        function saveSettingsToStorage() {
            localStorage.setItem('ttt_settings', JSON.stringify({
                theme: gameState.theme,
                symbolSet: gameState.symbolSet,
                soundEnabled: gameState.soundEnabled,
                difficulty: gameState.difficulty,
                mode: gameState.mode,
                firstPlayer: gameState.firstPlayer
            }));
        }

        function saveScoresToStorage() {
            localStorage.setItem('ttt_scores', JSON.stringify(gameState.scores));
        }

        /* ===================================================================
           UI & THEME UPDATES
           =================================================================== */
        function applySettingsUI() {
            // Apply Theme to body
            document.body.className = `min-h-screen flex flex-col items-center justify-between p-4 font-sans overflow-x-hidden ${gameState.theme}`;
            
            // Apply Sound state
            sound.enabled = gameState.soundEnabled;
            soundIcon.className = gameState.soundEnabled ? 'fa-solid fa-volume-high text-lg' : 'fa-solid fa-volume-xmark text-lg text-red-400';

            // Apply Mode active styling
            if (gameState.mode === 'pvai') {
                modePvAIBtn.className = 'flex-1 py-2 px-3 rounded-xl font-semibold text-sm transition-all duration-200 flex items-center justify-center gap-2 bg-indigo-600 text-white shadow';
                modePvPBtn.className = 'flex-1 py-2 px-3 rounded-xl font-semibold text-sm transition-all duration-200 flex items-center justify-center gap-2 text-gray-400 hover:text-white';
                diffContainer.classList.remove('hidden');
                scoreLabelO.textContent = 'AI (O)';
            } else {
                modePvPBtn.className = 'flex-1 py-2 px-3 rounded-xl font-semibold text-sm transition-all duration-200 flex items-center justify-center gap-2 bg-indigo-600 text-white shadow';
                modePvAIBtn.className = 'flex-1 py-2 px-3 rounded-xl font-semibold text-sm transition-all duration-200 flex items-center justify-center gap-2 text-gray-400 hover:text-white';
                diffContainer.classList.add('hidden');
                scoreLabelO.textContent = 'Player O';
            }

            // Apply Difficulty active styling
            diffBtns.forEach(btn => {
                if (btn.dataset.diff === gameState.difficulty) {
                    btn.className = 'diff-btn px-2.5 py-1 text-xs font-semibold rounded-lg bg-indigo-600 text-white transition-all shadow';
                } else {
                    btn.className = 'diff-btn px-2.5 py-1 text-xs font-semibold rounded-lg text-gray-400 hover:text-white transition-all';
                }
            });

            // Apply Theme Modal selector styling
            themeBtns.forEach(btn => {
                if (btn.dataset.theme === gameState.theme) {
                    btn.classList.add('border-indigo-500');
                    btn.classList.remove('border-gray-700');
                } else {
                    btn.classList.remove('border-indigo-500');
                    btn.classList.add('border-gray-700');
                }
            });

            // Symbol Select dropdown
            symbolSelect.value = gameState.symbolSet;

            // Update scores display
            updateScoreDisplay();
        }

        function updateScoreDisplay() {
            scoreXEl.textContent = gameState.scores.X;
            scoreOEl.textContent = gameState.scores.O;
            scoreTiesEl.textContent = gameState.scores.ties;
        }

        function updateTurnDisplay() {
            if (!gameState.gameActive) return;

            const currentSym = SYMBOL_SETS[gameState.symbolSet][gameState.currentPlayer];
            if (gameState.currentPlayer === 'X') {
                statusText.textContent = `Player X's Turn (${currentSym})`;
                turnIndicatorDot.className = 'w-3 h-3 rounded-full bg-red-500 animate-pulse';
            } else {
                const p2Label = gameState.mode === 'pvai' ? 'AI' : 'Player O';
                statusText.textContent = `${p2Label}'s Turn (${currentSym})`;
                turnIndicatorDot.className = 'w-3 h-3 rounded-full bg-blue-500 animate-pulse';
            }
        }

        /* ===================================================================
           GAME LOGIC
           =================================================================== */
        function initGame() {
            gameState.board = Array(9).fill(null);
            gameState.gameActive = true;
            gameState.currentPlayer = gameState.firstPlayer;

            // Clear cells & hide winning line SVG
            cellElements.forEach(cell => {
                cell.textContent = '';
                cell.className = 'cell-btn rounded-2xl flex items-center justify-center text-4xl font-black border transition-all duration-200 focus:outline-none';
            });
            winningLineSvg.classList.add('hidden');

            updateTurnDisplay();

            // If AI is set to go first
            if (gameState.mode === 'pvai' && gameState.currentPlayer === 'O') {
                setTimeout(makeAIMove, 400);
            }
        }

        function handleCellClick(e) {
            const index = parseInt(e.target.dataset.index);

            // Block click if cell occupied, game over, or AI turn
            if (gameState.board[index] !== null || !gameState.gameActive) return;
            if (gameState.mode === 'pvai' && gameState.currentPlayer === 'O') return;

            makeMove(index, gameState.currentPlayer);
        }

        function makeMove(index, player) {
            gameState.board[index] = player;
            const cell = cellElements[index];

            const symbol = SYMBOL_SETS[gameState.symbolSet][player];
            cell.textContent = symbol;

            // Styling based on player marker
            if (player === 'X') {
                cell.classList.add('text-red-500', 'border-red-500/30', 'marker-pop', 'animate-glow-x');
                sound.playX();
            } else {
                cell.classList.add('text-blue-500', 'border-blue-500/30', 'marker-pop', 'animate-glow-o');
                sound.playO();
            }

            // Check game result
            const winInfo = checkWin(gameState.board);

            if (winInfo) {
                handleWin(winInfo);
            } else if (isBoardFull(gameState.board)) {
                handleTie();
            } else {
                // Switch turn
                gameState.currentPlayer = gameState.currentPlayer === 'X' ? 'O' : 'X';
                updateTurnDisplay();

                // Trigger AI if applicable
                if (gameState.gameActive && gameState.mode === 'pvai' && gameState.currentPlayer === 'O') {
                    setTimeout(makeAIMove, 450);
                }
            }
        }

        function checkWin(board) {
            for (let i = 0; i < WINNING_COMBINATIONS.length; i++) {
                const [a, b, c] = WINNING_COMBINATIONS[i];
                if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                    return { winner: board[a], comboIndex: i, combination: [a, b, c] };
                }
            }
            return null;
        }

        function isBoardFull(board) {
            return board.every(cell => cell !== null);
        }

        function handleWin(winInfo) {
            gameState.gameActive = false;
            const winner = winInfo.winner;
            gameState.scores[winner]++;
            saveScoresToStorage();
            updateScoreDisplay();

            // Status message update
            if (gameState.mode === 'pvai') {
                statusText.textContent = winner === 'X' ? '🎉 You Won!' : '🤖 AI Wins!';
            } else {
                statusText.textContent = `🎉 Player ${winner} Wins!`;
            }
            turnIndicatorDot.className = 'w-3 h-3 rounded-full bg-green-500';

            // Draw line overlay
            drawWinningLine(winInfo.comboIndex);

            // Audio & Visual Effects
            sound.playWin();
            triggerConfetti();
        }

        function handleTie() {
            gameState.gameActive = false;
            gameState.scores.ties++;
            saveScoresToStorage();
            updateScoreDisplay();

            statusText.textContent = "It's a Draw / Tie!";
            turnIndicatorDot.className = 'w-3 h-3 rounded-full bg-yellow-500';
            sound.playTie();
        }

        /* ===================================================================
           WINNING LINE SVG CALCULATION
           =================================================================== */
        function drawWinningLine(comboIndex) {
            // Coordinates for SVG grid (0 to 100 percentage based)
            // Cell positions: Column 0: 16.6%, Column 1: 50%, Column 2: 83.3%
            // Row positions: Row 0: 16.6%, Row 1: 50%, Row 2: 83.3%
            const coords = [
                // Rows
                { x1: 5, y1: 16.6, x2: 95, y2: 16.6 },
                { x1: 5, y1: 50,   x2: 95, y2: 50 },
                { x1: 5, y1: 83.3, x2: 95, y2: 83.3 },
                // Columns
                { x1: 16.6, y1: 5, x2: 16.6, y2: 95 },
                { x1: 50,   y1: 5, x2: 50,   y2: 95 },
                { x1: 83.3, y1: 5, x2: 83.3, y2: 95 },
                // Diagonals
                { x1: 5, y1: 5, x2: 95, y2: 95 },
                { x1: 95, y1: 5, x2: 5, y2: 95 }
            ];

            const lineCoord = coords[comboIndex];
            winningLine.setAttribute('x1', lineCoord.x1);
            winningLine.setAttribute('y1', lineCoord.y1);
            winningLine.setAttribute('x2', lineCoord.x2);
            winningLine.setAttribute('y2', lineCoord.y2);

            winningLineSvg.classList.remove('hidden');
        }

        /* ===================================================================
           CELEBRATION CONFETTI
           =================================================================== */
        function triggerConfetti() {
            if (typeof confetti === 'function') {
                confetti({
                    particleCount: 70,
                    spread: 60,
                    origin: { y: 0.65 },
                    colors: ['#ef4444', '#3b82f6', '#10b981', '#f59e0b', '#8b5cf6']
                });
            }
        }

        /* ===================================================================
           AI LOGIC (Easy, Medium, Minimax Unbeatable)
           =================================================================== */
        function makeAIMove() {
            if (!gameState.gameActive) return;

            const emptyIndices = gameState.board
                .map((val, idx) => val === null ? idx : null)
                .filter(val => val !== null);

            if (emptyIndices.length === 0) return;

            let targetIndex;

            if (gameState.difficulty === 'easy') {
                // Completely random choice
                targetIndex = emptyIndices[Math.floor(Math.random() * emptyIndices.length)];
            } else if (gameState.difficulty === 'medium') {
                // 50% Minimax, 50% Random choice
                if (Math.random() > 0.5) {
                    targetIndex = getBestMinimaxMove();
                } else {
                    targetIndex = emptyIndices[Math.floor(Math.random() * emptyIndices.length)];
                }
            } else {
                // Unbeatable (Minimax algorithm)
                targetIndex = getBestMinimaxMove();
            }

            if (targetIndex !== undefined && targetIndex !== null) {
                makeMove(targetIndex, 'O');
            }
        }

        function getBestMinimaxMove() {
            let bestScore = -Infinity;
            let move = null;

            for (let i = 0; i < 9; i++) {
                if (gameState.board[i] === null) {
                    gameState.board[i] = 'O';
                    let score = minimax(gameState.board, 0, false);
                    gameState.board[i] = null;
                    if (score > bestScore) {
                        bestScore = score;
                        move = i;
                    }
                }
            }
            return move;
        }

        function minimax(board, depth, isMaximizing) {
            const winInfo = checkWin(board);
            if (winInfo) {
                return winInfo.winner === 'O' ? (10 - depth) : (depth - 10);
            }
            if (isBoardFull(board)) {
                return 0;
            }

            if (isMaximizing) {
                let maxEval = -Infinity;
                for (let i = 0; i < 9; i++) {
                    if (board[i] === null) {
                        board[i] = 'O';
                        let evaluation = minimax(board, depth + 1, false);
                        board[i] = null;
                        maxEval = Math.max(maxEval, evaluation);
                    }
                }
                return maxEval;
            } else {
                let minEval = Infinity;
                for (let i = 0; i < 9; i++) {
                    if (board[i] === null) {
                        board[i] = 'X';
                        let evaluation = minimax(board, depth + 1, true);
                        board[i] = null;
                        minEval = Math.min(minEval, evaluation);
                    }
                }
                return minEval;
            }
        }

        /* ===================================================================
           EVENT LISTENERS & HANDLERS
           =================================================================== */
        cellElements.forEach(cell => {
            cell.addEventListener('click', handleCellClick);
        });

        restartBtn.addEventListener('click', () => {
            sound.playClick();
            initGame();
        });

        resetScoreBtn.addEventListener('click', () => {
            sound.playClick();
            gameState.scores = { X: 0, O: 0, ties: 0 };
            saveScoresToStorage();
            updateScoreDisplay();
        });

        // Mode Switching
        modePvAIBtn.addEventListener('click', () => {
            if (gameState.mode !== 'pvai') {
                sound.playClick();
                gameState.mode = 'pvai';
                saveSettingsToStorage();
                applySettingsUI();
                initGame();
            }
        });

        modePvPBtn.addEventListener('click', () => {
            if (gameState.mode !== 'pvp') {
                sound.playClick();
                gameState.mode = 'pvp';
                saveSettingsToStorage();
                applySettingsUI();
                initGame();
            }
        });

        // AI Difficulty Switching
        diffBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                sound.playClick();
                gameState.difficulty = e.target.dataset.diff;
                saveSettingsToStorage();
                applySettingsUI();
                initGame();
            });
        });

        // Sound Toggle
        soundToggleBtn.addEventListener('click', () => {
            gameState.soundEnabled = !gameState.soundEnabled;
            saveSettingsToStorage();
            applySettingsUI();
            sound.playClick();
        });

        // Settings Modal Open/Close
        settingsBtn.addEventListener('click', () => {
            sound.playClick();
            settingsModal.classList.remove('opacity-0', 'pointer-events-none');
            settingsModalContent.classList.remove('scale-95');
            settingsModalContent.classList.add('scale-100');
        });

        function closeSettingsModal() {
            sound.playClick();
            settingsModal.classList.add('opacity-0', 'pointer-events-none');
            settingsModalContent.classList.remove('scale-100');
            settingsModalContent.classList.add('scale-95');
        }

        closeSettingsBtn.addEventListener('click', closeSettingsModal);

        themeBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                sound.playClick();
                gameState.theme = e.currentTarget.dataset.theme;
                themeBtns.forEach(b => {
                    b.classList.remove('border-indigo-500');
                    b.classList.add('border-gray-700');
                });
                e.currentTarget.classList.add('border-indigo-500');
                e.currentTarget.classList.remove('border-gray-700');
            });
        });

        firstPlayerBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                sound.playClick();
                if (e.target.id === 'firstPlayerX') {
                    gameState.firstPlayer = 'X';
                    document.getElementById('firstPlayerX').className = 'first-player-btn p-2 rounded-xl border border-indigo-500 bg-indigo-600/30 text-xs font-semibold text-white';
                    document.getElementById('firstPlayerO').className = 'first-player-btn p-2 rounded-xl border border-gray-700 bg-gray-800 text-xs font-semibold text-gray-400';
                } else {
                    gameState.firstPlayer = 'O';
                    document.getElementById('firstPlayerO').className = 'first-player-btn p-2 rounded-xl border border-indigo-500 bg-indigo-600/30 text-xs font-semibold text-white';
                    document.getElementById('firstPlayerX').className = 'first-player-btn p-2 rounded-xl border border-gray-700 bg-gray-800 text-xs font-semibold text-gray-400';
                }
            });
        });

        saveSettingsBtn.addEventListener('click', () => {
            gameState.symbolSet = symbolSelect.value;
            saveSettingsToStorage();
            applySettingsUI();
            closeSettingsModal();
            initGame();
        });

        /* ===================================================================
           INITIAL SETUP ON LOAD
           =================================================================== */
        window.onload = function () {
            loadSavedState();
            applySettingsUI();
            initGame();
        };
    </script>
</body>
</html>
