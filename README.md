<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Crossroads Game</title>
<style>
  body, html {
    margin: 0; padding: 0; background: #222;
    display: flex; justify-content: center; align-items: center; height: 100vh;
  }
  canvas {
    background: #444;
    display: block;
    border: 2px solid #ccc;
  }
</style>
</head>
<body>
<canvas id="game" width="400" height="600"></canvas>

<script>
// Setup canvas and context
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const roadWidth = 200;
const roadX = (canvas.width - roadWidth) / 2;
const laneCount = 3;
const laneWidth = roadWidth / laneCount;

let keys = {};
document.addEventListener('keydown', e => keys[e.key] = true);
document.addEventListener('keyup', e => keys[e.key] = false);

// Base64 images for player car and enemy car (simple colored rectangles)
const playerImg = new Image();
playerImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAvElEQVRYR+3XwQ2DMBBF0fW5Cd2ACNgNrg6kYcBO6DTaAGxU0pLkLFPE6jBgb0vuRpUQgxFDw/dwN3pfg3BGHMeqzmIjiF2gDx/nmB9kRJwIvB+gFLgOiPZF3AHDkArsAPYFGzjI5r1IKn/maGOS8VyCR6QPh6nHdXAap+nCZTVHvHJOn3Mf89Qq/E4Ju2r/dl5cl8RReeGJQKxjG6OG9FQvoUOJHfMPZ2BTANb4jfHLgC4N+EEkHzECfoDLbD6Agsmc0nY+Qb6z0/kB3sEQV3EgncUAAAAASUVORK5CYII=';

const enemyImg = new Image();
enemyImg.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAv0lEQVRYR+3WQQrDMAxF0Xk+AlVkNXCfAOqg3UBnsBHZBJyRrBLiC4UPnMhuhR3jbb3PGb+OFkDzfCNh9CSH8DfANH/ABYdo9j1xVjQw8DB7A4PfSxRZnXKDSexZ86ht0prMq/jUFE2o+HT6AtwdAMViAbqA4eQKu0zDlzBng6yAS6QHZ4E2gEtyQkXrXcEVYh2BAvAcXqDX+A8tjCXwRfDXk9AK/DfhPCrQkA6PA3F4Df14AMq0q5yoV1iYAAAAASUVORK5CYII=';

// Player info
const player = {
  lane: 1, // middle lane (0,1,2)
  width: 40,
  height: 70,
  y: canvas.height - 90,
  speed: 5
};

// Enemy cars array
let enemies = [];
const enemyWidth = 40;
const enemyHeight = 70;
const enemySpeedStart = 3;
let enemySpeed = enemySpeedStart;
let enemySpawnInterval = 1500; // ms
let lastEnemySpawn = 0;

// Game state
let gameOver = false;
let score = 0;

// Helper: get x position for a lane index
function laneToX(lane) {
  return roadX + lane * laneWidth + (laneWidth - player.width)/2;
}

// Draw road (gray with white dashed lines between lanes)
function drawRoad() {
  // Road background
  ctx.fillStyle = '#555';
  ctx.fillRect(roadX, 0, roadWidth, canvas.height);

  // Left and right road borders
  ctx.strokeStyle = 'white';
  ctx.lineWidth = 4;
  ctx.beginPath();
  ctx.moveTo(roadX, 0);
  ctx.lineTo(roadX, canvas.height);
  ctx.moveTo(roadX + roadWidth, 0);
  ctx.lineTo(roadX + roadWidth, canvas.height);
  ctx.stroke();

  // Lane dividers (dashed)
  ctx.strokeStyle = 'white';
  ctx.lineWidth = 2;
  ctx.setLineDash([20, 20]);
  for(let i=1; i<laneCount; i++) {
    let x = roadX + i * laneWidth;
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, canvas.height);
    ctx.stroke();
  }
  ctx.setLineDash([]);
}

// Draw player car
function drawPlayer() {
  const x = laneToX(player.lane);
  ctx.drawImage(playerImg, x, player.y, player.width, player.height);
}

// Draw enemies
function drawEnemies() {
  enemies.forEach(e => {
    const x = laneToX(e.lane);
    ctx.drawImage(enemyImg, x, e.y, enemyWidth, enemyHeight);
  });
}

// Update player position based on keys
function updatePlayer() {
  if (keys['ArrowLeft'] && player.lane > 0) {
    player.lane--;
    keys['ArrowLeft'] = false; // Prevent continuous movement
  }
  if (keys['ArrowRight'] && player.lane < laneCount -1) {
    player.lane++;
    keys['ArrowRight'] = false;
  }
}

// Update enemies position, remove off screen, check collisions
function updateEnemies(deltaTime) {
  for (let i = enemies.length -1; i>=0; i--) {
    enemies[i].y += enemySpeed;

    // Collision check with player
    if (
      enemies[i].lane === player.lane &&
      enemies[i].y + enemyHeight > player.y &&
      enemies[i].y < player.y + player.height
    ) {
      gameOver = true;
    }

    // Remove enemies off screen
    if (enemies[i].y > canvas.height) {
      enemies.splice(i, 1);
      score++;
      // Increase speed every 5 cars dodged
      if(score % 5 === 0) enemySpeed += 0.5;
    }
  }

  // Spawn enemies periodically
  if (!gameOver && performance.now() - lastEnemySpawn > enemySpawnInterval) {
    let lane = Math.floor(Math.random() * laneCount);
    enemies.push({ lane: lane, y: -enemyHeight });
    lastEnemySpawn = performance.now();
  }
}

// Draw score and game over message
function drawUI() {
  ctx.fillStyle = 'white';
  ctx.font = '20px Arial';
  ctx.fillText('Score: ' + score, 10, 30);

  if (gameOver) {
    ctx.fillStyle = 'rgba(0,0,0,0.7)';
    ctx.fillRect(0, canvas.height/2 - 50, canvas.width, 100);

    ctx.fillStyle = 'red';
    ctx.font = '40px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('GAME OVER', canvas.width/2, canvas.height/2);
    ctx.font = '20px Arial';
    ctx.fillText('Refresh to try again', canvas.width/2, canvas.height/2 + 40);
    ctx.textAlign = 'start';
  }
}

let lastTime = 0;
function gameLoop(time=0) {
  const deltaTime = time - lastTime;
  lastTime = time;

  ctx.clearRect(0,0,canvas.width,canvas.height);
  drawRoad();

  if (!gameOver) {
    updatePlayer();
    updateEnemies(deltaTime);
  }
  drawPlayer();
  drawEnemies();
  drawUI();

  requestAnimationFrame(gameLoop);
}

// Start game
gameLoop();
</script>
</body>
</html>

