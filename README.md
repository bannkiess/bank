<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tic Tac Toe Deluxe</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        cyber: {
                            bg: '#0f172a',
                            card: '#1e293b',
                            x: '#38bdf8',
                            o: '#f43f5e',
                            accent: '#a855f7'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Outfit:wght@400;600;800&display=swap');
        body {
            font-family: 'Outfit', sans-serif;
            user-select: none;
        }
        .cell-glow-x {
            text-shadow: 0 0 15px rgba(56, 189, 248, 0.6);
        }
        .cell-glow-o {
            text-shadow: 0 0 15px rgba(244, 63, 94, 0.6);
        }
        .pop-in {
            animation: popIn 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
        @keyframes popIn {
            0% { transform: scale(0); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
    </style>
</head>
<body class="bg-cyber-bg text-slate-100 min-h-screen flex flex-col items-center justify-between p-4 relative overflow-x-hidden">

    <!-- Header & Mode Controls -->
    <header class="w-full max-w-md flex flex-col items-center gap-4 mt-2">
        <h1 class="text-4xl font-extrabold tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-cyber-x via-cyber-accent to-cyber-o">
            TIC TAC TOE
        </h1>
        
        <!-- Game Settings Bar -->
        <div class="flex items-center gap-2 bg-cyber-card p-1.5 rounded-xl border border-slate-700/60 shadow-lg text-sm w-full justify-between">
            <div class="flex items-center gap-1 bg-slate-900/50 p-1 rounded-lg">
                <button id="btn-pvp" onclick="setMode('pvp')" class="px-3 py-1.5 rounded-md font-semibold transition-all duration-200">
                    <i class="fa-solid fa-users mr-1.5"></i> 2P
                </button>
                <button id="btn-ai" onclick="setMode('ai')" class="px-3 py-1.5 rounded-md font-semibold transition-all duration-200">
                    <i class="fa-solid fa-robot mr-1.5"></i> VS AI
                </button>
            </div>

            <!-- AI Difficulty Selection -->
            <select id="ai-difficulty" onchange="setDifficulty(this.value)" class="bg-slate-900 text-slate-200 px-2 py-1.5 rounded-lg border border-slate-700 font-medium focus:outline-none cursor-pointer">
                <option value="easy">Easy</option>
                <option value="medium">Medium</option>
                <option value="unbeatable" selected>Unbeatable</option>
            </select>
