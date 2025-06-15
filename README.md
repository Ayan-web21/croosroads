<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Crossroads Game with Graphics</title>
<style>
  body {
    margin: 0; 
    background: #333;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    user-select: none;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    color: white;
  }
  #game {
    position: relative;
    width: 480px;
    height: 640px;
    background: url('https://i.ibb.co/47rjHXT/road-background.png') no-repeat center center;
    background-size: cover;
    border: 5px solid #555;
    box-shadow: 0 0 20px #000;
    overflow: hidden;
  }
  #player {
    position: absolute;
    width: 50px;
    height: 100px;
    bottom: 10px;
    left: 215px;
    z-index: 10;
    background: url('https://i.ibb.co/5TLmnmr/red-car.png') no-repeat center center;
    background-size: contain;
  }
  .car {
    position: absolute;
    width: 50px;
    height: 100px;
    background-size: contain;
    background-repeat: no-repeat;
    border-radius: 10px;
  }
  #scoreboard {
    position: absolute;
    top: 10px;
    left: 10px;
    font-size: 22px;
    font-weight: 700;
    text-shadow: 1px 1px 3px black;
    z-index: 20;
  }
  #message {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 28px;
    color: yellow;
    font-weight: 700;
    display: none;
    z-index: 30;
    text-align: center;
    text-shadow: 2px 2px 5px black;
  }
</style>
</head>
<body>
  <div id="game">
    <div id="scoreboard">Score: 0</div>
    <div id="message">Game Over!<br>Press R to Restart</div>
    <div id="player"></div>
  </div>

<script>
(() => {
  const game = document.getElementById('game');
  const player = document.getElementById('player');
  const scoreboard = document.getElementById('scoreboard');
  const message = document.getElementById('message');

  const gameWidth = game.clientWidth;
  const gameHeight = game.clientHeight;

  const playerWidth = 50;
  const playerHeight = 100;
  const lanePositions = [70, 170, 270, 370]; // 4 lanes horizontally

  let playerX = lanePositions[1]; // Start in lane 2
  let playerY = gameHeight - playerHeight - 10;
  const playerSpeed = 8;

  let cars = [];
  let carSpawnInterval = 1500; // milliseconds
  let lastSpawnTime = 0;

  let score = 0;
  let gameOver = false;

  const enemyCarImages = [
    'https://i.ibb.co/fCnL7MF/blue-car.png',
    'https://i.ibb.co/1zptm1D/yellow-car.png',
    'https://i.ibb.co/FVZ7Hvc/green-car.png'
  ];

  // Controls
  const keys = {};

  window.addEventListener('keydown', e => {
    keys[e.key] = true;
    if (gameOver && (e.key === 'r' || e.key === 'R')) {
      resetGame();
    }
  });
  window.addEventListener('keyup', e => {
    keys[e.key] = false;
  });

  function createCar() {
    // Choose random lane and enemy car image
    let laneIndex = Math.floor(Math.random() * lanePositions.length);
    let imgIndex = Math.floor(Math.random() * enemyCarImages.length);

    const car = document.createElement('div');
    car.classList.add('car');
    car.style.backgroundImage = `url('${enemyCarImages[imgIndex]}')`;
    game.appendChild(car);

    // Start at top of screen at lane x
    let x = lanePositions[laneIndex];
    let y = -playerHeight;

    car.style.left = x + 'px';
    car.style.top = y + 'px';

    cars.push({element: car, x, y, speed: 3 + Math.random() * 3, lane: laneIndex});
  }

  function movePlayer() {
    if (keys['ArrowLeft']) {
      playerX -= playerSpeed;
      if (playerX < lanePositions[0]) playerX = lanePositions[0];
    }
    if (keys['ArrowRight']) {
      playerX += playerSpeed;
      if (playerX > lanePositions[lanePositions.length - 1]) playerX = lanePositions[lanePositions.length - 1];
    }
    if (keys['ArrowUp']) {
      playerY -= playerSpeed;
      if (playerY < 0) playerY = 0;
    }
    if (keys['ArrowDown']) {
      playerY += playerSpeed;
      if (playerY > gameHeight - playerHeight) playerY = gameHeight - playerHeight;
    }
    player.style.left = playerX + 'px';
    player.style.top = playerY + 'px';
  }

  function updateCars(deltaTime) {
    for (let i = cars.length - 1; i >= 0; i--) {
      let car = cars[i];
      car.y += car.speed;
      car.element.style.top = car.y + 'px';

      if (car.y > gameHeight) {
        car.element.remove();
        cars.splice(i, 1);
        if (!gameOver) score++;
      }
    }
  }

  function checkCollision() {
    const playerRect = player.getBoundingClientRect();

    for (let car of cars) {
      const carRect = car.element.getBoundingClientRect();
      if (
        !(playerRect.right < carRect.left ||
          playerRect.left > carRect.right ||
          playerRect.bottom < carRect.top ||
          playerRect.top > carRect.bottom)
      ) {
        gameOver = true;
        message.style.display = 'block';
        break;
      }
    }
  }

  let lastTime = 0;
  function gameLoop(timestamp = 0) {
    if (!lastTime) lastTime = timestamp;
    const deltaTime = timestamp - lastTime;
    lastTime = timestamp;

    if (!gameOver) {
      movePlayer();
      updateCars(deltaTime);
      checkCollision();

      if (timestamp - lastSpawnTime > carSpawnInterval) {
        createCar();
        lastSpawnTime = timestamp;
        if (carSpawnInterval > 700) carSpawnInterval -= 10;
      }

      scoreboard.textContent = 'Score: ' + score;
    }

    requestAnimationFrame(gameLoop);
  }

  function resetGame() {
    for(let car of cars) {
      car.element.remove();
    }
    cars = [];
    score = 0;
    carSpawnInterval = 1500;
    lastSpawnTime = 0;
    playerX = lanePositions[1];
    playerY = gameHeight - playerHeight - 10;
    player.style.left = playerX + 'px';
    player.style.top = playerY + 'px';
    gameOver = false;
    message.style.display = 'none';
  }

  // Init player position
  player.style.left = playerX + 'px';
  player.style.top = playerY + 'px';

  gameLoop();
})();
</script>
</body>
</html>

