<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pac-Man Style Game</title>
  <style>
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: radial-gradient(circle at top, #111 0%, #000 55%);
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 16px;
    }

    h1 {
      margin: 0 0 10px;
      color: #ffd700;
      letter-spacing: 1px;
      text-align: center;
      text-shadow: 0 0 12px rgba(255, 215, 0, 0.35);
    }

    .game-shell {
      width: 100%;
      max-width: 760px;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 12px;
    }

    .hud {
      display: flex;
      gap: 18px;
      font-size: 18px;
      flex-wrap: wrap;
      justify-content: center;
      text-align: center;
    }

    .instructions {
      color: #ccc;
      text-align: center;
      max-width: 720px;
      line-height: 1.5;
      font-size: 15px;
    }

    .canvas-wrap {
      position: relative;
      width: 100%;
      display: flex;
      justify-content: center;
    }

    canvas {
      border: 4px solid #1e90ff;
      background: #000;
      box-shadow: 0 0 28px rgba(30, 144, 255, 0.35);
      max-width: 100%;
      height: auto;
      border-radius: 10px;
    }

    .start-screen {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
      background: rgba(0, 0, 0, 0.82);
      border-radius: 12px;
    }

    .start-card {
      max-width: 520px;
      text-align: center;
      background: rgba(15, 15, 30, 0.95);
      border: 2px solid #1e90ff;
      border-radius: 20px;
      padding: 24px;
      box-shadow: 0 0 30px rgba(30, 144, 255, 0.25);
    }

    .start-card h2 {
      margin-top: 0;
      color: #ffd700;
      font-size: 34px;
    }

    .start-card p {
      color: #ddd;
      line-height: 1.6;
      margin-bottom: 18px;
    }

    .buttons {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      justify-content: center;
    }

    button {
      padding: 12px 18px;
      font-size: 16px;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      background: #ffd700;
      color: #000;
      font-weight: bold;
      min-width: 130px;
    }

    button:hover {
      opacity: 0.93;
    }

    .mobile-controls {
      display: grid;
      grid-template-columns: 70px 70px 70px;
      grid-template-rows: 70px 70px 70px;
      gap: 8px;
      margin-top: 8px;
      user-select: none;
    }

    .mobile-controls button {
      min-width: 0;
      width: 70px;
      height: 70px;
      font-size: 24px;
      border-radius: 16px;
      background: #1e90ff;
      color: white;
    }

    .mobile-controls .empty {
      visibility: hidden;
    }

    .legend {
      color: #aaa;
      font-size: 14px;
      text-align: center;
    }

    @media (max-width: 640px) {
      h1 {
        font-size: 28px;
      }

      .hud {
        font-size: 16px;
      }

      .start-card h2 {
        font-size: 28px;
      }

      .start-card {
        padding: 18px;
      }
    }
  </style>
</head>
<body>
  <div class="game-shell">
    <h1>Pac-Man Style Game</h1>

    <div class="hud">
      <div>Score: <span id="score">0</span></div>
      <div>Lives: <span id="lives">3</span></div>
      <div>Dots Left: <span id="dots">0</span></div>
    </div>

    <div class="instructions">
      Use arrow keys on desktop or the on-screen controls on mobile. Eat every dot, avoid the ghosts, and press Start to begin.
    </div>

    <div class="canvas-wrap">
      <canvas id="game" width="532" height="608"></canvas>

      <div class="start-screen" id="startScreen">
        <div class="start-card">
          <h2>Ready to Play?</h2>
          <p>
            Guide Pac-Man through the maze, collect every dot, and avoid the ghosts. This version includes mobile controls, sound effects, and a stronger arcade layout for GitHub.
          </p>
          <div class="buttons">
            <button id="startBtn">Start Game</button>
            <button id="restartBtn">Restart</button>
          </div>
        </div>
      </div>
    </div>

    <div class="mobile-controls" aria-label="Mobile Controls">
      <div class="empty"></div>
      <button data-dir="up">▲</button>
      <div class="empty"></div>
      <button data-dir="left">◀</button>
      <button data-dir="down">▼</button>
      <button data-dir="right">▶</button>
      <div class="empty"></div>
      <div class="empty"></div>
      <div class="empty"></div>
    </div>

    <div class="legend">Tip: click or tap Start Game first so audio can play in the browser.</div>
  </div>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');

    const scoreEl = document.getElementById('score');
    const livesEl = document.getElementById('lives');
    const dotsEl = document.getElementById('dots');
    const restartBtn = document.getElementById('restartBtn');
    const startBtn = document.getElementById('startBtn');
    const startScreen = document.getElementById('startScreen');
    const mobileButtons = document.querySelectorAll('.mobile-controls button[data-dir]');

    const tileSize = 38;
    const rows = 16;
    const cols = 14;

    const originalMap = [
      [1,1,1,1,1,1,1,1,1,1,1,1,1,1],
      [1,2,2,2,2,2,1,1,2,2,2,2,2,1],
      [1,2,1,1,2,2,2,2,2,2,1,1,2,1],
      [1,2,1,1,2,1,1,1,1,2,1,1,2,1],
      [1,2,2,2,2,2,2,1,2,2,2,2,2,1],
      [1,2,1,1,2,1,2,2,2,1,2,1,2,1],
      [1,2,2,1,2,1,1,0,1,1,2,1,2,1],
      [1,1,2,1,2,2,2,0,2,2,2,1,2,1],
      [1,2,2,2,2,1,0,0,1,2,2,2,2,1],
      [1,2,1,1,2,1,1,1,1,2,1,1,2,1],
      [1,2,2,1,2,2,2,2,2,2,1,2,2,1],
      [1,1,2,1,1,1,2,1,1,2,1,2,1,1],
      [1,2,2,2,2,1,2,2,1,2,2,2,2,1],
      [1,2,1,1,2,2,2,2,2,2,1,1,2,1],
      [1,2,2,2,2,1,2,1,2,2,2,2,2,1],
      [1,1,1,1,1,1,1,1,1,1,1,1,1,1]
    ];

    let map;
    let score = 0;
    let lives = 3;
    let dotsRemaining = 0;
    let gameOver = false;
    let gameWon = false;
    let gameStarted = false;
    let animationId;
    let audioReady = false;
    let audioCtx;

    const player = {
      x: 1 * tileSize + tileSize / 2,
      y: 1 * tileSize + tileSize / 2,
      radius: 13,
      speed: 2.2,
      direction: { x: 0, y: 0 },
      nextDirection: { x: 0, y: 0 },
      mouthOpen: 0.2,
      mouthSpeed: 0.07,
      mouthDir: 1
    };

    const ghostStartPositions = [
      { x: 6 * tileSize + tileSize / 2, y: 7 * tileSize + tileSize / 2, color: '#ff4d4d' },
      { x: 7 * tileSize + tileSize / 2, y: 7 * tileSize + tileSize / 2, color: '#00e5ff' },
      { x: 7 * tileSize + tileSize / 2, y: 8 * tileSize + tileSize / 2, color: '#ff9ff3' }
    ];

    let ghosts = [];

    function cloneMap() {
      return originalMap.map(row => [...row]);
    }

    function initAudio() {
      if (audioReady) return;
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      audioReady = true;
    }

    function beep(freq, duration, type = 'sine', volume = 0.03) {
      if (!audioReady || !audioCtx) return;
      const oscillator = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      oscillator.type = type;
      oscillator.frequency.value = freq;
      gain.gain.value = volume;
      oscillator.connect(gain);
      gain.connect(audioCtx.destination);
      oscillator.start();
      oscillator.stop(audioCtx.currentTime + duration);
    }

    function playDotSound() {
      beep(620, 0.05, 'square', 0.02);
    }

    function playHitSound() {
      beep(180, 0.18, 'sawtooth', 0.05);
      setTimeout(() => beep(120, 0.2, 'sawtooth', 0.04), 60);
    }

    function playWinSound() {
      beep(523, 0.08, 'triangle', 0.04);
      setTimeout(() => beep(659, 0.08, 'triangle', 0.04), 90);
      setTimeout(() => beep(784, 0.18, 'triangle', 0.05), 180);
    }

    function playStartSound() {
      beep(392, 0.08, 'triangle', 0.04);
      setTimeout(() => beep(523, 0.08, 'triangle', 0.04), 90);
    }

    function resetGhosts() {
      ghosts = ghostStartPositions.map(g => ({
        x: g.x,
        y: g.y,
        radius: 13,
        speed: 1.8,
        direction: randomDirection(),
        color: g.color
      }));
    }

    function randomDirection() {
      const dirs = [
        { x: 1, y: 0 },
        { x: -1, y: 0 },
        { x: 0, y: 1 },
        { x: 0, y: -1 }
      ];
      return dirs[Math.floor(Math.random() * dirs.length)];
    }

    function resetPlayerPosition() {
      player.x = 1 * tileSize + tileSize / 2;
      player.y = 1 * tileSize + tileSize / 2;
      player.direction = { x: 0, y: 0 };
      player.nextDirection = { x: 0, y: 0 };
    }

    function countDots() {
      dotsRemaining = 0;
      for (let row = 0; row < rows; row++) {
        for (let col = 0; col < cols; col++) {
          if (map[row][col] === 2) dotsRemaining++;
        }
      }
    }

    function updateHUD() {
      scoreEl.textContent = score;
      livesEl.textContent = lives;
      dotsEl.textContent = dotsRemaining;
    }

    function initializeGame() {
      map = cloneMap();
      score = 0;
      lives = 3;
      gameOver = false;
      gameWon = false;
      resetPlayerPosition();
      resetGhosts();
      countDots();
      updateHUD();
      draw();
    }

    function startGame() {
      initAudio();
      if (audioCtx && audioCtx.state === 'suspended') {
        audioCtx.resume();
      }
      playStartSound();
      initializeGame();
      gameStarted = true;
      startScreen.style.display = 'none';
      cancelAnimationFrame(animationId);
      gameLoop();
    }

    function tileAtPixel(x, y) {
      const col = Math.floor(x / tileSize);
      const row = Math.floor(y / tileSize);
      if (row < 0 || row >= rows || col < 0 || col >= cols) return 1;
      return map[row][col];
    }

    function isWallAt(x, y, radius = 0) {
      const points = [
        { x: x - radius, y: y - radius },
        { x: x + radius, y: y - radius },
        { x: x - radius, y: y + radius },
        { x: x + radius, y: y + radius }
      ];
      return points.some(point => tileAtPixel(point.x, point.y) === 1);
    }

    function canMove(entity, dir) {
      if (dir.x === 0 && dir.y === 0) return true;
      const nextX = entity.x + dir.x * entity.speed;
      const nextY = entity.y + dir.y * entity.speed;
      return !isWallAt(nextX, nextY, entity.radius);
    }

    function centerOfTile(value) {
      return Math.floor(value / tileSize) * tileSize + tileSize / 2;
    }

    function nearTileCenter(entity) {
      return Math.abs(entity.x - centerOfTile(entity.x)) < 3 && Math.abs(entity.y - centerOfTile(entity.y)) < 3;
    }

    function setDirection(dir) {
      if (dir === 'up') player.nextDirection = { x: 0, y: -1 };
      if (dir === 'down') player.nextDirection = { x: 0, y: 1 };
      if (dir === 'left') player.nextDirection = { x: -1, y: 0 };
      if (dir === 'right') player.nextDirection = { x: 1, y: 0 };
    }

    function updatePlayer() {
      if (nearTileCenter(player) && canMove(player, player.nextDirection)) {
        player.direction = { ...player.nextDirection };
        player.x = centerOfTile(player.x);
        player.y = centerOfTile(player.y);
      }

      if (canMove(player, player.direction)) {
        player.x += player.direction.x * player.speed;
        player.y += player.direction.y * player.speed;
      }

      player.mouthOpen += player.mouthSpeed * player.mouthDir;
      if (player.mouthOpen > 0.8 || player.mouthOpen < 0.15) {
        player.mouthDir *= -1;
      }

      eatDot();
    }

    function eatDot() {
      const col = Math.floor(player.x / tileSize);
      const row = Math.floor(player.y / tileSize);

      if (map[row] && map[row][col] === 2) {
        map[row][col] = 0;
        score += 10;
        dotsRemaining--;
        playDotSound();
        updateHUD();

        if (dotsRemaining === 0) {
          gameWon = true;
          playWinSound();
        }
      }
    }

    function updateGhosts() {
      ghosts.forEach(ghost => {
        if (nearTileCenter(ghost)) {
          ghost.x = centerOfTile(ghost.x);
          ghost.y = centerOfTile(ghost.y);

          const directions = [
            { x: 1, y: 0 },
            { x: -1, y: 0 },
            { x: 0, y: 1 },
            { x: 0, y: -1 }
          ];

          let options = directions.filter(dir => canMove(ghost, dir));

          const opposite = { x: -ghost.direction.x, y: -ghost.direction.y };
          const filtered = options.filter(dir => !(dir.x === opposite.x && dir.y === opposite.y));
          if (filtered.length > 0) options = filtered;

          const forwardBlocked = !canMove(ghost, ghost.direction);
          const atIntersection = options.length >= 2;

          if (forwardBlocked || atIntersection) {
            ghost.direction = options[Math.floor(Math.random() * options.length)] || ghost.direction;
          }
        }

        if (canMove(ghost, ghost.direction)) {
          ghost.x += ghost.direction.x * ghost.speed;
          ghost.y += ghost.direction.y * ghost.speed;
        }
      });
    }

    function checkCollisions() {
      for (const ghost of ghosts) {
        const dx = player.x - ghost.x;
        const dy = player.y - ghost.y;
        const distance = Math.sqrt(dx * dx + dy * dy);

        if (distance < player.radius + ghost.radius - 4) {
          lives--;
          playHitSound();
          updateHUD();

          if (lives <= 0) {
            gameOver = true;
            startScreen.style.display = 'flex';
          } else {
            resetPlayerPosition();
            resetGhosts();
          }
          break;
        }
      }
    }

    function drawRoundedWall(x, y) {
      ctx.fillStyle = '#003cff';
      ctx.beginPath();
      ctx.roundRect(x + 2, y + 2, tileSize - 4, tileSize - 4, 8);
      ctx.fill();
      ctx.strokeStyle = '#66a3ff';
      ctx.lineWidth = 2;
      ctx.stroke();
    }

    function drawMap() {
      for (let row = 0; row < rows; row++) {
        for (let col = 0; col < cols; col++) {
          const x = col * tileSize;
          const y = row * tileSize;

          if (map[row][col] === 1) {
            drawRoundedWall(x, y);
          } else if (map[row][col] === 2) {
            ctx.fillStyle = '#ffd700';
            ctx.beginPath();
            ctx.arc(x + tileSize / 2, y + tileSize / 2, 4, 0, Math.PI * 2);
            ctx.fill();
          }
        }
      }
    }

    function drawPlayer() {
      let angle = 0;
      if (player.direction.x === 1) angle = 0;
      if (player.direction.x === -1) angle = Math.PI;
      if (player.direction.y === 1) angle = Math.PI / 2;
      if (player.direction.y === -1) angle = -Math.PI / 2;

      ctx.save();
      ctx.translate(player.x, player.y);
      ctx.rotate(angle);
      ctx.fillStyle = '#ffd700';
      ctx.beginPath();
      ctx.moveTo(0, 0);
      ctx.arc(0, 0, player.radius, player.mouthOpen, Math.PI * 2 - player.mouthOpen);
      ctx.closePath();
      ctx.fill();
      ctx.restore();
    }

    function drawGhost(ghost) {
      const x = ghost.x;
      const y = ghost.y;
      const r = ghost.radius;

      ctx.fillStyle = ghost.color;
      ctx.beginPath();
      ctx.arc(x, y, r, Math.PI, 0);
      ctx.lineTo(x + r, y + r);
      ctx.lineTo(x + r / 2, y + r - 4);
      ctx.lineTo(x, y + r);
      ctx.lineTo(x - r / 2, y + r - 4);
      ctx.lineTo(x - r, y + r);
      ctx.closePath();
      ctx.fill();

      ctx.fillStyle = '#fff';
      ctx.beginPath();
      ctx.arc(x - 5, y - 2, 4, 0, Math.PI * 2);
      ctx.arc(x + 5, y - 2, 4, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = '#000';
      ctx.beginPath();
      ctx.arc(x - 5, y - 2, 2, 0, Math.PI * 2);
      ctx.arc(x + 5, y - 2, 2, 0, Math.PI * 2);
      ctx.fill();
    }

    function drawGhosts() {
      ghosts.forEach(drawGhost);
    }

    function drawOverlay(text, subtext) {
      ctx.fillStyle = 'rgba(0, 0, 0, 0.72)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.fillStyle = '#fff';
      ctx.font = 'bold 38px Arial';
      ctx.textAlign = 'center';
      ctx.fillText(text, canvas.width / 2, canvas.height / 2 - 10);

      ctx.font = '20px Arial';
      ctx.fillStyle = '#ffd700';
      ctx.fillText(subtext, canvas.width / 2, canvas.height / 2 + 28);
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawMap();
      drawPlayer();
      drawGhosts();

      if (gameOver) {
        drawOverlay('Game Over', 'Tap Restart or Start Game');
      } else if (gameWon) {
        drawOverlay('You Win!', 'You cleared the maze');
        startScreen.style.display = 'flex';
      }
    }

    function gameLoop() {
      updatePlayer();
      updateGhosts();
      checkCollisions();
      draw();

      if (!gameOver && !gameWon) {
        animationId = requestAnimationFrame(gameLoop);
      }
    }

    document.addEventListener('keydown', event => {
      if (event.key === 'ArrowUp') setDirection('up');
      if (event.key === 'ArrowDown') setDirection('down');
      if (event.key === 'ArrowLeft') setDirection('left');
      if (event.key === 'ArrowRight') setDirection('right');
    });

    mobileButtons.forEach(button => {
      const dir = button.dataset.dir;
      button.addEventListener('touchstart', event => {
        event.preventDefault();
        setDirection(dir);
      }, { passive: false });
      button.addEventListener('click', () => setDirection(dir));
    });

    restartBtn.addEventListener('click', () => {
      initAudio();
      initializeGame();
      if (!gameStarted) return;
      startScreen.style.display = 'none';
      gameOver = false;
      gameWon = false;
      cancelAnimationFrame(animationId);
      gameLoop();
    });

    startBtn.addEventListener('click', startGame);

    initializeGame();
  </script>
</body>
</html>
