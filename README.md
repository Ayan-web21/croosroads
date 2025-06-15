 <!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Basic Crossroads Game</title>
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
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const roadWidth = 200;
const roadX = (canvas.width - roadWidth) / 2;
const laneCount = 3;
const laneWidth = roadWidth / laneCount;

let keys = {};
document.addEventListener('keydown', e => keys[e.key] = true);
document.addEventListener('keyup', e => keys[e.key] = false);

const player = {
  lane: 1,
  width: 40,
  height: 70,
  y: canvas.height - 90,
  color: 'red'
};

let enemies = [];
const enemyWidth = 40;
const enemyHeight = 70;
let enemySpeed = 3;
let enemySpawnInterval = 1500;
let lastEnemySpawn = 0;

let gameOver = false;
let score = 0;

function laneToX(lane) {
  return roadX + lane * laneWidth + (laneWidth - player.width) / 2;
}

function drawRoad() {
  ctx.fillStyle = '#555';
  ctx.fillRect(roadX, 0, roadWidth, canvas.height);

  ctx.strokeStyle = 'white';
  ctx.lineWidth = 4;
  ctx.beginPath();
  ctx.moveTo(roadX, 0);
  ctx.lineTo(roadX, canvas.height);
  ctx.moveTo(roadX + roadWidth, 0);
  ctx.lineTo(roadX + roadWidth, canvas.height);
  ctx.stroke();

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

function drawPlayer() {
  ctx.fillStyle = player.color;
  const x = laneToX(player.lane);
  ctx.fillRect(x, player.y, player.width, player.height);
}

function drawEnemies() {
  ctx.fillStyle = 'blue';
  enemies.forEach(e => {
    const x = laneToX(e.lane);
    ctx.fillRect(x, e.y, enemyWidth, enemyHeight);
  });
}

function updatePlayer() {
  if(keys['ArrowLeft'] && player.lane > 0){
    player.lane--;
    keys['ArrowLeft'] = false;
  }
  if(keys['ArrowRight'] && player.lane < laneCount - 1){
    player.lane++;
    keys['ArrowRight'] = false;
  }
}

function updateEnemies(deltaTime) {
  for(let i = enemies.length - 1; i >= 0; i--) {
    enemies[i].y += enemySpeed;

    if(enemies[i].lane === player.lane &&
       enemies[i].y + enemyHeight > player.y &&
       enemies[i].y < player.y + player.height) {
      gameOver = true;
    }

    if(enemies[i].y > canvas.height) {
      enemies.splice(i, 1);
      score++;
      if(score % 5 === 0) enemySpeed += 0.5;
    }
  }

  if(!gameOver && performance.now() - lastEnemySpawn > enemySpawnInterval) {
    enemies.push({lane: Math.floor(Math.random() * laneCount), y: -enemyHeight});
    lastEnemySpawn = performance.now();
  }
}

function drawUI() {
  ctx.fillStyle = 'white';
  ctx.font = '20px Arial';
  ctx.fillText('Score: ' + score, 10, 30);

  if(gameOver) {
    ctx.fillStyle = 'rgba(0,0,0,0.7)';
    ctx.fillRect(0, canvas.height/2 - 50, canvas.width, 100);
    ctx.fillStyle = 'red';
    ctx.font = '40px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('GAME OVER', canvas.width/2, canvas.height/2);
    ctx.font = '20px Arial';
    ctx.fillText('Refresh to play again', canvas.width/2, canvas.height/2 + 40);
    ctx.textAlign = 'start';
  }
}

let lastTime = 0;
function gameLoop(time=0) {
  const deltaTime = time - lastTime;
  lastTime = time;

  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawRoad();

  if(!gameOver) {
    updatePlayer();
    updateEnemies(deltaTime);
  }
  drawPlayer();
  drawEnemies();
  drawUI();

  requestAnimationFrame(gameLoop);
}

gameLoop();
</script>
</body>
</html>

