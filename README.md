<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sohini's Contribution Collector</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;700&display=swap');
        
        body {
            background-color: #0d1117;
            color: #c9d1d9;
            font-family: 'Fira Code', monospace;
            margin: 0;
            overflow: hidden;
            touch-action: none;
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        canvas {
            border: 2px solid #30363d;
            border-radius: 12px;
            background-color: #010409;
            box-shadow: 0 0 30px rgba(56, 139, 253, 0.15);
            max-width: 95%;
            max-height: 70vh;
        }

        .stats-bar {
            display: flex;
            justify-content: space-between;
            width: 400px;
            max-width: 95%;
            margin-bottom: 12px;
            font-size: 1rem;
            background: #161b22;
            padding: 8px 16px;
            border-radius: 8px;
            border: 1px solid #30363d;
        }

        .commit-text { color: #39d353; font-weight: bold; }
        .high-score-text { color: #e3b341; font-weight: bold; }

        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(13, 17, 23, 0.9);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10;
            text-align: center;
            padding: 20px;
        }

        .btn {
            background-color: #238636;
            color: white;
            padding: 12px 32px;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
            border: none;
            margin-top: 24px;
            font-size: 1.1rem;
            box-shadow: 0 4px 14px rgba(35, 134, 54, 0.4);
        }

        .btn:hover {
            background-color: #2ea043;
            transform: translateY(-2px);
        }

        .consistency-quote {
            font-style: italic;
            color: #8b949e;
            margin-top: 20px;
            font-size: 0.85rem;
        }

        .controls-hint {
            margin-top: 15px;
            font-size: 0.8rem;
            color: #58a6ff;
        }
    </style>
</head>
<body>

<div id="game-container">
    <div class="stats-bar">
        <div>Commits: <span id="score" class="commit-text">0</span></div>
        <div>Best: <span id="highScore" class="high-score-text">0</span></div>
    </div>
    
    <canvas id="gameCanvas"></canvas>

    <div id="overlay">
        <h1 id="title" class="text-3xl font-bold mb-2 text-white">Sohini's Builder Challenge</h1>
        <p id="message" class="max-w-md mb-2 text-gray-400">
            Catch the green <b>Commits</b>.<br>
            Dodge the red <b>Bugs</b> to maintain your streak.
        </p>
        <button id="start-btn" class="btn">Deploy App</button>
        <p class="consistency-quote">"Consistency beats motivation — code every day."</p>
        <p class="controls-hint">← → / A D / Tap sides to move</p>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const highScoreEl = document.getElementById('highScore');
    const overlay = document.getElementById('overlay');
    const startBtn = document.getElementById('start-btn');
    const titleEl = document.getElementById('title');
    const messageEl = document.getElementById('message');

    const CANVAS_WIDTH = 400;
    const CANVAS_HEIGHT = 500;
    canvas.width = CANVAS_WIDTH;
    canvas.height = CANVAS_HEIGHT;

    let score = 0;
    let highScore = 0;
    let gameActive = false;
    let player = {
        x: CANVAS_WIDTH / 2 - 25,
        y: CANVAS_HEIGHT - 45,
        width: 60,
        height: 12,
        speed: 8,
        dx: 0
    };

    let items = [];
    let frameCount = 0;

    const keys = {};
    document.addEventListener('keydown', (e) => keys[e.code] = true);
    document.addEventListener('keyup', (e) => keys[e.code] = false);

    // Responsive Touch
    function handleTouch(e) {
        if (!gameActive) return;
        const touchX = e.touches[0].clientX;
        const rect = canvas.getBoundingClientRect();
        const canvasX = (touchX - rect.left) * (CANVAS_WIDTH / rect.width);
        player.dx = canvasX < CANVAS_WIDTH / 2 ? -player.speed : player.speed;
        e.preventDefault();
    }
    canvas.addEventListener('touchstart', handleTouch, {passive: false});
    canvas.addEventListener('touchend', () => player.dx = 0);

    function spawnItem() {
        const isBug = Math.random() < 0.25; 
        const size = 22;
        items.push({
            x: Math.random() * (CANVAS_WIDTH - size),
            y: -size,
            size: size,
            type: isBug ? 'bug' : 'commit',
            speed: 3 + (score / 15) 
        });
    }

    function resetGame() {
        score = 0;
        items = [];
        player.x = CANVAS_WIDTH / 2 - player.width / 2;
        scoreEl.innerText = score;
        gameActive = true;
        overlay.style.display = 'none';
        requestAnimationFrame(update);
    }

    function gameOver() {
        gameActive = false;
        if (score > highScore) {
            highScore = score;
            highScoreEl.innerText = highScore;
        }
        overlay.style.display = 'flex';
        titleEl.innerText = "Build Crashed!";
        titleEl.style.color = "#f85149";
        messageEl.innerHTML = `You collected <span class="commit-text">${score}</span> commits.<br>Your builder mindset is strong!`;
        startBtn.innerText = "Re-Deploy";
    }

    function update() {
        if (!gameActive) return;

        // Input Logic
        if (keys['ArrowLeft'] || keys['KeyA']) player.dx = -player.speed;
        else if (keys['ArrowRight'] || keys['KeyD']) player.dx = player.speed;
        else if (!('ontouchstart' in window)) player.dx = 0;

        player.x += player.dx;
        if (player.x < 0) player.x = 0;
        if (player.x + player.width > CANVAS_WIDTH) player.x = CANVAS_WIDTH - player.width;

        // Spawning
        frameCount++;
        const spawnInterval = Math.max(15, 50 - Math.floor(score / 4));
        if (frameCount % spawnInterval === 0) spawnItem();

        ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

        // Draw Player (The Code Block)
        ctx.fillStyle = '#58a6ff';
        ctx.beginPath();
        ctx.roundRect(player.x, player.y, player.width, player.height, 4);
        ctx.fill();
        // Glow
        ctx.shadowBlur = 15;
        ctx.shadowColor = '#58a6ff';
        ctx.strokeStyle = '#c9d1d9';
        ctx.stroke();
        ctx.shadowBlur = 0;

        // Draw Items
        for (let i = items.length - 1; i >= 0; i--) {
            const item = items[i];
            item.y += item.speed;

            if (item.type === 'commit') {
                ctx.fillStyle = '#39d353';
                ctx.beginPath();
                ctx.roundRect(item.x, item.y, item.size, item.size, 4);
                ctx.fill();
            } else {
                ctx.fillStyle = '#f85149';
                ctx.font = 'bold 24px monospace';
                ctx.fillText('×', item.x + 2, item.y + 18);
            }

            // Collision
            if (
                item.y + item.size > player.y &&
                item.x < player.x + player.width &&
                item.x + item.size > player.x
            ) {
                if (item.type === 'commit') {
                    score++;
                    scoreEl.innerText = score;
                    items.splice(i, 1);
                } else {
                    gameOver();
                }
            } else if (item.y > CANVAS_HEIGHT) {
                items.splice(i, 1);
            }
        }

        // Grid Lines
        ctx.strokeStyle = 'rgba(48, 54, 61, 0.5)';
        ctx.lineWidth = 0.5;
        for(let x=0; x<CANVAS_WIDTH; x+=40) {
            ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, CANVAS_HEIGHT); ctx.stroke();
        }

        requestAnimationFrame(update);
    }

    startBtn.addEventListener('click', resetGame);
</script>
</body>
</html>
