---
layout: index
title: "Flappy Bird"
heading: "Flappy Bird"
subheading: "Canvas Game"
description: "A Flappy Bird clone built with HTML canvas"
user-story: "As a player, I want to flap a bird through gaps in pipes by clicking, tapping, or pressing space so that I can score points and see my final score when I lose."
---

<style>
  .game-wrap{padding:0}
  .game-stage{position:relative;width:100%;height:70vh;min-height:420px}
  #gameCanvas{width:100%;height:100%;display:block;background:#66ccff;touch-action:none}
  .overlay{position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:12px;background:rgba(0,0,0,0.55);color:#fff;text-align:center}
  .score-display{position:absolute;top:14px;left:50%;transform:translateX(-50%);font-size:32px;font-weight:700;color:#fff;text-shadow:2px 2px 0 #000}
</style>

<div class="game-wrap">
  <div id="gameStage" class="game-stage">
    <canvas id="gameCanvas"></canvas>

    <div id="startScreen" class="overlay">
      <h2>Flappy Bird</h2>
      <p>Click, tap, or press Space to flap</p>
      <button id="startBtn" class="btn btn-warning btn-lg" type="button">Start Game</button>
    </div>

    <div id="gameOverScreen" class="overlay d-none">
      <h2>Game Over</h2>
      <p id="finalScore">Score: 0</p>
      <button id="restartBtn" class="btn btn-warning btn-lg" type="button">Play Again</button>
    </div>

    <div id="scoreDisplay" class="score-display d-none">0</div>
  </div>
</div>

<script>
  (function () {
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const startScreen = document.getElementById('startScreen');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const scoreDisplay = document.getElementById('scoreDisplay');
    const finalScoreEl = document.getElementById('finalScore');
    const startBtn = document.getElementById('startBtn');
    const restartBtn = document.getElementById('restartBtn');

    const GRAVITY = 0.45;
    const FLAP_VELOCITY = -8;
    const PIPE_WIDTH = 70;
    const PIPE_CAP_HEIGHT = 24;
    const PIPE_GAP = 170;
    const PIPE_SPEED = 3;
    const PIPE_INTERVAL = 90; // frames
    const GROUND_HEIGHT = 60;

    function resizeCanvas() {
      const rect = gameStage.getBoundingClientRect();
      canvas.width = Math.round(rect.width);
      canvas.height = Math.round(rect.height);
    }
    const gameStage = document.getElementById('gameStage');
    resizeCanvas();
    window.addEventListener('resize', () => {
      if (!running) resizeCanvas();
    });

    let audioCtx = null;
    // synthesized beeps avoid needing external audio files
    function tone(freq, duration, type, glideTo) {
      if (!audioCtx) return;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = type || 'sine';
      osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
      if (glideTo) osc.frequency.exponentialRampToValueAtTime(glideTo, audioCtx.currentTime + duration);
      gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
      osc.connect(gain).connect(audioCtx.destination);
      osc.start();
      osc.stop(audioCtx.currentTime + duration);
    }
    const playFlap = () => tone(500, 0.09, 'square');
    const playPoint = () => tone(880, 0.12, 'sine', 1200);
    const playDeath = () => tone(300, 0.5, 'sawtooth', 60);

    let bird, pipes, clouds, frame, score, running, started, groundOffset, W, H;

    function resetGame() {
      resizeCanvas();
      W = canvas.width;
      H = canvas.height;
      bird = { x: W * 0.2, y: H / 2, velocity: 0, radius: 15, angle: 0 };
      pipes = [];
      clouds = Array.from({ length: 4 }, (_, i) => ({ x: (W / 4) * i, y: 30 + Math.random() * 80, scale: 0.7 + Math.random() * 0.6 }));
      frame = 0;
      score = 0;
      running = true;
      groundOffset = 0;
      scoreDisplay.textContent = '0';
    }

    function spawnPipe() {
      const topHeight = 40 + Math.random() * (H - GROUND_HEIGHT - PIPE_GAP - 80);
      pipes.push({ x: W, top: topHeight, bottom: topHeight + PIPE_GAP, passed: false });
    }

    function flap() {
      if (!running) return;
      bird.velocity = FLAP_VELOCITY;
      playFlap();
    }

    function endGame() {
      running = false;
      playDeath();
      finalScoreEl.textContent = 'Score: ' + score;
      gameOverScreen.classList.remove('d-none');
      scoreDisplay.classList.add('d-none');
    }

    function update() {
      frame++;
      bird.velocity += GRAVITY;
      bird.y += bird.velocity;
      bird.angle = Math.max(-0.5, Math.min(1.2, bird.velocity / 10));

      groundOffset = (groundOffset - PIPE_SPEED) % 40;

      if (frame % PIPE_INTERVAL === 0) spawnPipe();

      pipes.forEach((p) => (p.x -= PIPE_SPEED));
      while (pipes.length && pipes[0].x < -PIPE_WIDTH) pipes.shift();

      clouds.forEach((c) => {
        c.x -= PIPE_SPEED * 0.3;
        if (c.x < -80) c.x = W + 40;
      });

      pipes.forEach((p) => {
        if (!p.passed && p.x + PIPE_WIDTH < bird.x) {
          p.passed = true;
          score++;
          scoreDisplay.textContent = score;
          playPoint();
        }
        const hitX = bird.x + bird.radius > p.x && bird.x - bird.radius < p.x + PIPE_WIDTH;
        const hitY = bird.y - bird.radius < p.top || bird.y + bird.radius > p.bottom;
        if (hitX && hitY) endGame();
      });

      if (bird.y + bird.radius > H - GROUND_HEIGHT || bird.y - bird.radius < 0) endGame();
    }

    function drawCloud(c) {
      ctx.save();
      ctx.translate(c.x, c.y);
      ctx.scale(c.scale, c.scale);
      ctx.fillStyle = 'rgba(255,255,255,0.85)';
      ctx.beginPath();
      ctx.arc(0, 0, 16, 0, Math.PI * 2);
      ctx.arc(18, -8, 14, 0, Math.PI * 2);
      ctx.arc(34, 0, 16, 0, Math.PI * 2);
      ctx.arc(16, 8, 18, 0, Math.PI * 2);
      ctx.fill();
      ctx.restore();
    }

    function drawPipe(p) {
      const bodyGrad = ctx.createLinearGradient(p.x, 0, p.x + PIPE_WIDTH, 0);
      bodyGrad.addColorStop(0, '#3fae4a');
      bodyGrad.addColorStop(0.5, '#6fd97a');
      bodyGrad.addColorStop(1, '#2e8f3a');

      // top pipe
      ctx.fillStyle = bodyGrad;
      ctx.fillRect(p.x, 0, PIPE_WIDTH, p.top - PIPE_CAP_HEIGHT);
      ctx.fillRect(p.x - 5, p.top - PIPE_CAP_HEIGHT, PIPE_WIDTH + 10, PIPE_CAP_HEIGHT);

      // bottom pipe
      ctx.fillRect(p.x, p.bottom + PIPE_CAP_HEIGHT, PIPE_WIDTH, H - GROUND_HEIGHT - (p.bottom + PIPE_CAP_HEIGHT));
      ctx.fillRect(p.x - 5, p.bottom, PIPE_WIDTH + 10, PIPE_CAP_HEIGHT);

      ctx.strokeStyle = 'rgba(0,0,0,0.35)';
      ctx.lineWidth = 2;
      ctx.strokeRect(p.x, 0, PIPE_WIDTH, p.top - PIPE_CAP_HEIGHT);
      ctx.strokeRect(p.x - 5, p.top - PIPE_CAP_HEIGHT, PIPE_WIDTH + 10, PIPE_CAP_HEIGHT);
      ctx.strokeRect(p.x, p.bottom + PIPE_CAP_HEIGHT, PIPE_WIDTH, H - GROUND_HEIGHT - (p.bottom + PIPE_CAP_HEIGHT));
      ctx.strokeRect(p.x - 5, p.bottom, PIPE_WIDTH + 10, PIPE_CAP_HEIGHT);
    }

    function drawGround() {
      ctx.fillStyle = '#ded28a';
      ctx.fillRect(0, H - GROUND_HEIGHT, W, GROUND_HEIGHT);
      ctx.fillStyle = '#6fae3a';
      ctx.fillRect(0, H - GROUND_HEIGHT, W, 10);
      ctx.fillStyle = '#5a9330';
      for (let x = groundOffset; x < W + 40; x += 40) {
        ctx.beginPath();
        ctx.moveTo(x, H - GROUND_HEIGHT + 2);
        ctx.lineTo(x + 20, H - GROUND_HEIGHT + 2);
        ctx.lineTo(x + 10, H - GROUND_HEIGHT + 10);
        ctx.closePath();
        ctx.fill();
      }
    }

    function drawBird() {
      ctx.save();
      ctx.translate(bird.x, bird.y);
      ctx.rotate(bird.angle);

      // body
      ctx.fillStyle = '#ffd800';
      ctx.beginPath();
      ctx.ellipse(0, 0, bird.radius, bird.radius * 0.85, 0, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = '#c99a00';
      ctx.lineWidth = 2;
      ctx.stroke();

      // wing
      ctx.fillStyle = '#f2b900';
      ctx.beginPath();
      ctx.ellipse(-3, 4, 8, 5, Math.PI / 6, 0, Math.PI * 2);
      ctx.fill();

      // eye
      ctx.fillStyle = '#000';
      ctx.beginPath();
      ctx.arc(bird.radius * 0.35, -bird.radius * 0.3, 2.5, 0, Math.PI * 2);
      ctx.fill();

      // beak
      ctx.fillStyle = '#ff8c00';
      ctx.beginPath();
      ctx.moveTo(bird.radius * 0.7, 0);
      ctx.lineTo(bird.radius * 1.4, -3);
      ctx.lineTo(bird.radius * 1.4, 5);
      ctx.closePath();
      ctx.fill();

      ctx.restore();
    }

    function draw() {
      const sky = ctx.createLinearGradient(0, 0, 0, H);
      sky.addColorStop(0, '#4ec0f2');
      sky.addColorStop(1, '#bdeeff');
      ctx.fillStyle = sky;
      ctx.fillRect(0, 0, W, H);

      clouds.forEach(drawCloud);
      pipes.forEach(drawPipe);
      drawGround();
      drawBird();
    }

    function loop() {
      if (!running) return;
      update();
      draw();
      requestAnimationFrame(loop);
    }

    function startGame() {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      started = true;
      resetGame();
      startScreen.classList.add('d-none');
      gameOverScreen.classList.add('d-none');
      scoreDisplay.classList.remove('d-none');
      requestAnimationFrame(loop);
    }

    startBtn.addEventListener('click', startGame);
    restartBtn.addEventListener('click', startGame);

    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') {
        e.preventDefault();
        if (!started || (!running && !gameOverScreen.classList.contains('d-none'))) return;
        flap();
      }
    });
    canvas.addEventListener('mousedown', flap);
    canvas.addEventListener('touchstart', (e) => {
      e.preventDefault();
      flap();
    });
  })();
</script>