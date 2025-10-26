<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Wave Dash</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: #1e1e1e;
      font-family: Arial;
    }
    canvas {
      display: block;
      margin: auto;
      background: #222;
    }
    #skinSelector {
      position: absolute;
      top: 10px;
      right: 10px;
      z-index: 10;
      font-size: 16px;
    }
    #restartBtn {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, 100px);
      padding: 12px 24px;
      font-size: 18px;
      background-color: #ff4444;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      display: none;
      z-index: 20;
    }
  </style>
</head>
<body>
  <select id="skinSelector">
    <option value="orange">Orange</option>
    <option value="lime">Lime</option>
    <option value="cyan">Cyan</option>
  </select>
  <button id="restartBtn">다시 시작</button>
  <canvas id="gameCanvas" width="480" height="640"></canvas>
  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const restartBtn = document.getElementById("restartBtn");

    let player = { x: 100, y: 320, radius: 15, velocity: 0, gravity: 0.5, lift: -8 };
    let wave = { frequency: 0.015, offset: 0, speed: 2 };
    let waveAmplitudes = Array(canvas.width).fill(0).map(() => 60 + Math.random() * 40);
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    let gameOver = false;
    let gameStarted = false;
    let difficultyTimer = 0;
    let playerColor = "orange";

    document.getElementById("skinSelector").addEventListener("change", (e) => {
      playerColor = e.target.value;
    });

    restartBtn.addEventListener("click", () => {
      resetGame();
      restartBtn.style.display = "none";
    });

    function getWaveY(x) {
      let amp = waveAmplitudes[Math.floor(x)];
      return canvas.height / 2 + amp * Math.sin(wave.frequency * (x + wave.offset));
    }

    function drawWave() {
      ctx.fillStyle = "#00BFFF";
      ctx.beginPath();
      ctx.moveTo(0, getWaveY(0) + waveAmplitudes[0]);
      for (let x = 0; x <= canvas.width; x++) {
        ctx.lineTo(x, getWaveY(x) + waveAmplitudes[x]);
      }
      ctx.lineTo(canvas.width, canvas.height);
      ctx.lineTo(0, canvas.height);
      ctx.closePath();
      ctx.fill();

      ctx.beginPath();
      ctx.moveTo(0, getWaveY(0) - waveAmplitudes[0]);
      for (let x = 0; x <= canvas.width; x++) {
        ctx.lineTo(x, getWaveY(x) - waveAmplitudes[x]);
      }
      ctx.lineTo(canvas.width, 0);
      ctx.lineTo(0, 0);
      ctx.closePath();
      ctx.fill();
    }

    function drawPlayer() {
      ctx.beginPath();
      ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
      ctx.fillStyle = playerColor;
      ctx.fill();
      ctx.closePath();
    }

    function updatePlayer() {
      player.velocity += player.gravity;
      player.y += player.velocity;
      if (player.y < 0) {
        player.y = 0;
        player.velocity = 0;
      }
      if (player.y > canvas.height) {
        player.y = canvas.height;
        player.velocity = 0;
      }
    }

    function checkCollision() {
      let waveY = getWaveY(player.x);
      let amp = waveAmplitudes[Math.floor(player.x)];
      if (player.y < waveY - amp || player.y > waveY + amp) {
        gameOver = true;
        updateHighScore();
        restartBtn.style.display = "block";
      }
    }

    function drawScore() {
      ctx.fillStyle = "#fff";
      ctx.font = "20px Arial";
      ctx.fillText("Score: " + score, 20, 30);
      ctx.fillText("High Score: " + highScore, 20, 60);
    }

    function drawStartScreen() {
      ctx.fillStyle = "#fff";
      ctx.font = "30px Arial";
      ctx.fillText("Wave Dash", canvas.width / 2 - 80, canvas.height / 2 - 40);
      ctx.font = "20px Arial";
      ctx.fillText("Click to Start", canvas.width / 2 - 60, canvas.height / 2);
    }

    function resetGame() {
      player.y = 320;
      player.velocity = 0;
      wave.offset = 0;
      score = 0;
      gameOver = false;
      difficultyTimer = 0;
      wave.speed = 2;
      waveAmplitudes = waveAmplitudes.map(() => 60 + Math.random() * 40);
    }

    function updateHighScore() {
      if (score > highScore) {
        highScore = score;
        localStorage.setItem("highScore", highScore);
      }
    }

    function randomizeAmplitudes() {
      waveAmplitudes = waveAmplitudes.map(() => 60 + Math.random() * 40);
    }

    setInterval(randomizeAmplitudes, 5000);

    function playBackgroundMusic() {
      const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      const tempo = 120;
      const notes = [261.63, 293.66, 329.63, 392.00];
      let startTime = audioCtx.currentTime;
      for (let i = 0; i < 16; i++) {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = "square";
        osc.frequency.setValueAtTime(notes[i % notes.length], startTime + i * 0.5);
        gain.gain.setValueAtTime(0.1, startTime + i * 0.5);
        osc.connect(gain).connect(audioCtx.destination);
        osc.start(startTime + i * 0.5);
        osc.stop(startTime + i * 0.5 + 0.4);
      }
      setTimeout(playBackgroundMusic, (60 / tempo) * 1000 * 8);
    }

    function gameLoop() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      if (!gameStarted) {
        drawStartScreen();
        return requestAnimationFrame(gameLoop);
      }
      drawWave();
      drawPlayer();
      drawScore();
      if (!gameOver) {
        updatePlayer();
        wave.offset += wave.speed;
        checkCollision();
        score += 1;
        difficultyTimer++;
        if (difficultyTimer % 600 === 0) {
          wave.speed += 0.5;
        }
      }
      requestAnimationFrame(gameLoop);
    }

    canvas.addEventListener("mousedown", () => {
      if (!gameStarted) {
        gameStarted = true;
        playBackgroundMusic();
      } else if (!gameOver) {
        player.velocity = player.lift;
      }
    });

    canvas.addEventListener("touchstart", () => {
      if (!gameStarted) {
        gameStarted = true;
        playBackgroundMusic();
      } else if (!gameOver) {
        player.velocity = player.lift;
      }
    });

    gameLoop();
  </script>
</body>
</html>
