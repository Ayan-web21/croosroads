<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Crossroads Game</title>
  <style>
    html, body {
      margin: 0; padding: 0; overflow: hidden; background: #222;
    }
    canvas {
      display: block;
      margin: 0 auto;
      background: #444;
    }
  </style>
</head>
<body>
<canvas id="game" width="400" height="600"></canvas>

<script>
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  // Images with direct, confirmed working URLs
  const roadImage = new Image();
  const playerImage = new Image();
  const enemyImage = new Image();

  roadImage.src = 'https://i.postimg.cc/fbqYX2yD/road.png'; // a road texture
  playerImage.src = 'https://i.postimg.cc/3RdZ5T1c/red-car.png'; // red car
  enemyImage.src = 'https://i.postimg.cc/4y6Cg9QR/blue-car.png'; // blue car

  let player = {
    x: 175,
    y: 480,
    width: 50,
    height: 100,
    speed: 5
  };

  let keys = {};
  let enemies = [];
  let score = 0;
  let gameOver = false;
  let bgY = 0;

  function spawnEnemy() {
    const lanes = [50, 150, 250, 350];
    let laneX = lanes[Math.floor(Math.random() * lanes.length)];
    enemies.push({
      x: laneX,
      y: -120,
      width: 50,
      height: 100,
      speed: 3 + Math.random() * 2
    });
  }

  function drawBackground() {
    bgY += 3;
    if (bgY > canvas.height) bgY = 0;
    ctx.drawImage(roadImage, 0, bgY - canvas.height, canvas.width, canvas.height);
    ctx.drawImage(roadImage, 0, bgY, canvas.width, canvas.height);
  }

  function drawPlayer() {
    ctx.drawImage(playerImage, player.x, player.y, player.width, player.height);
  }

  function drawEnemies() {
    enemies.forEach(e => {
      e.y += e.speed;
      ctx.drawImage(enemyImage, e.x, e.y, e.width, e.height);

      // Collision detection
      if (
        player.x < e.x + e.width &&
        player.x + player.width > e.x &&
        player.y < e.y + e.height &&
        player.y + player.height > e.y
      ) {
        gameOver = true;
      }
    });
    enemies = enemies.filter(e => e.y < canvas.height + 120);
  }

  function movePlayer() {
    if (keys['a'] || keys['ArrowLeft']) player.x -= player.speed;
    if (keys['d'] || keys['ArrowRight']) player.x += player.speed;
    if (keys['w'] || keys['ArrowUp']) player.y -= player.speed;
    if (keys['s'] || keys['ArrowDown']) player.y += player.speed;

    // Clamp inside canvas
    player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));
    player.y = Math.max(0, Math.min(canvas.height - player.height, player.y));
  }

  function drawScore() {
    ctx.fillStyle = 'white';
    ctx.font = '20px Arial';
    ctx.fillText(`Score: ${score}`, 10, 30);
  }

  function loop() {
    if (gameOver) {
      ctx.fillStyle = 'rgba(0,0,0,0.7)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = 'yellow';
      ctx.font = '30px Arial';
      ctx.fillText('Game Over!', 130, 300);
      ctx.font = '20px Arial';
      ctx.fillText('Press R to Restart', 115, 340);
      return;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawBackground();
    movePlayer();
    drawPlayer();
    drawEnemies();
    drawScore();

    if (Math.random() < 0.02) spawnEnemy();
    score++;

    requestAnimationFrame(loop);
  }

  document.addEventListener('keydown', e => {
    keys[e.key.toLowerCase()] = true;
    if (e.key.toLowerCase() === 'r' && gameOver) {
      // Reset game
      player.x = 175;
      player.y = 480;
      enemies = [];
      score = 0;
      gameOver = false;
      loop();
    }
  });

  document.addEventListener('keyup', e => {
    keys[e.key.toLowerCase()] = false;
  });

  // Start the game loop when road image loads
  roadImage.onload = () => {
    loop();
  }
</script>
</body>
</html>
