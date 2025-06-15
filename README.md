<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Crossroads Running Car Game</title>
<style>
  body, html {
    margin: 0; padding: 0; background: #222;
    display: flex; justify-content: center; align-items: center; height: 100vh;
    user-select: none;
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

const player = {
  lane: 1,
  targetLane: 1,
  x: 0,
  y: canvas.height - 120,
  width: 50,
  height: 90,
  color: 'red',
  speed: 0.15,  // lane change animation speed
};

let keys = {};
document.addEventListener('keydown', e => {
  keys[e.key.toLowerCase()] = true;
});
document.addEventListener('keyup', e => {
  keys[e.key.toLowerCase()] = false;
});

const enemyWidth = 50;
const enemyHeight = 90;
let enemies = [];
let enemySpeed = 4;
let enemySpawnInterval = 1400;
let lastEnemySpawn = 0;

let score = 0;
let gameOver = false;

let roadLineOffset = 0;
const roadLineHeight = 40;
const roadLineGap = 30;

function laneToX(lane) {
  return roadX + lane * laneWidth + (laneWidth - player.width) / 2;
}

function drawRoad() {
  // road background
  ctx.fillStyle = '#555';
  ctx.fillRect(roadX, 0, roadWidth, canvas.height);

  // road edges
  ctx.strokeStyle = 'white';
  ctx.lineWidth = 4;
  ctx.beginPath();
  ctx.moveTo(roadX, 0);
  ctx.lineTo(roadX, canvas.height);
  ctx.moveTo(roadX + roadWidth, 0);
  ctx.lineTo(roadX + roadWidth, canvas.height);
  ctx.stroke();

  // lane dashed lines (scrolling)
  ctx.strokeStyle = 'white';
  ctx.lineWidth = 2;
  ctx.setLineDash([20, 20]);
  for(let i=1; i<laneCount; i++) {
    let x = roadX + i * laneWidth;
    ctx.beginPath();
    let yStart = -roadLineOffset;
    while(yStart < canvas.height) {
      ctx.moveTo(x, yStart);
      ctx.lineTo(x, yStart + roadLineHeight);
      yStart += roadLineHeight + roadLineGap;
    }
    ctx.stroke();
  }
  ctx.setLineDash([]);
}

function drawCar(x, y, color) {
  // body
  ctx.fillStyle = color;
  ctx.fillRect(x, y, player.width, player.height);

  // windows (simple shapes)
  ctx.fillStyle = '#ccc';
  ctx.fillRect(x + 10, y + 20, player.width - 20, 25);

  // wheels
  ctx.fillStyle = 'black';
  const wheelRadius = 8;
  // front wheels
  ctx.beginPath();
  ctx.arc(x + 12, y + player.height - 10, wheelRadius, 0, Math.PI * 2);
  ctx.arc(x + player.width - 12, y + player.height - 10, wheelRadius, 0, Math.PI * 2);
  // back wheels
  ctx.arc(x + 12, y + 10, wheelRadius, 0, Math.PI * 2);
  ctx.arc(x + player.width - 12, y + 10, wheelRadius, 0, Math.PI * 2);
  ctx.fill();
}

function drawPlayer() {
  drawCar(player.x, player.y, player.color);
}

function drawEnemies() {
  enemies.forEach(e => {
    drawCar(laneToX(e.lane), e.y, 'blue');
  });
}

function updatePlayer() {
  // check controls to change lanes
  if((keys['arrowleft'] || keys['a']) && player.targetLane > 0) {
    player.targetLane = player.targetLane - 1;
    keys['arrowleft'] = false;
    keys['a'] = false;
  }
  if((keys['arrowright'] || keys['d']) && player.targetLane < laneCount - 1) {
    player.targetLane = player.targetLane + 1;
    keys['arrowright'] = false;
    keys['d'] = false;
  }

  // smooth lane movement
  const targetX = laneToX(player.targetLane);
  if (Math.abs(player.x - targetX) > 1) {
    player.x += (targetX - player.x) * player.speed;
  } else {
    player.x = targetX;
    player.lane = player.targetLane;
  }
}

function updateEnemies(deltaTime) {
  for(let i = enemies.length - 1; i >= 0; i--) {
    enemies[i].y += enemySpeed;

    // collision check
    if(enemies[i].lane === player.lane &&
       enemies[i].y + enemyHeight > player.y + 20 &&
       enemies[i].y < player.y + player.height - 20) {
      gameOver = true;
    }

    // remove offscreen enemies and increase score
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
  
  roadLineOffset += enemySpeed;
  if(roadLineOffset > roadLineHeight + roadLineGap) {
    roadLineOffset = 0;
  }

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

// Start player in middle lane
player.x = laneToX(player.lane);

gameLoop();
</script>
</body>
</html>
