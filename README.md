 <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Crossroads Game</title>
  <style>
    body {
      margin: 0;
      background: #444;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      font-family: sans-serif;
      overflow: hidden;
    }

    #game {
      position: relative;
      width: 400px;
      height: 600px;
      background: url('https://i.ibb.co/47rjHXT/road-background.png') no-repeat center;
      background-size: cover;
      border: 4px solid #111;
      box-shadow: 0 0 15px black;
      overflow: hidden;
    }

    #player {
      position: absolute;
      width: 50px;
      height: 100px;
      background: url('https://i.ibb.co/5TLmnmr/red-car.png') no-repeat center;
      background-size: contain;
      bottom: 20px;
      left: 175px;
    }

    .enemy {
      position: absolute;
      width: 50px;
      height: 100px;
      background: url('https://i.ibb.co/fCnL7MF/blue-car.png') no-repeat center;
      background-size: contain;
    }

    #score {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-weight: bold;
      text-shadow: 1px 1px 3px black;
    }

    #gameOver {
      position: absolute;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.7);
      display: none;
      color: yellow;
      font-size: 24px;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      z-index: 10;
    }
  </style>
</head>
<body>

<div id="game">
  <div id="score">Score: 0</div>
  <div id="gameOver">
    <div>Game Over</div>
    <div>Press R to Restart</div>
  </div>
  <div id="player"></div>
</div>

<script>
  const game = document.getElementById("game");
  const player = document.getElementById("player");
  const scoreBoard = document.getElementById("score");
  const gameOverScreen = document.getElementById("gameOver");

  let keys = {};
  let cars = [];
  let score = 0;
  let gameOver = false;
  let playerX = 175;
  let playerY = 480;

  const speed = 5;
  const lanes = [50, 150, 250, 350];

  function spawnEnemy() {
    const car = document.createElement("div");
    car.className = "enemy";
    car.style.left = lanes[Math.floor(Math.random() * lanes.length)] + "px";
    car.style.top = "-120px";
    game.appendChild(car);
    cars.push({ el: car, y: -120, speed: 3 + Math.random() * 2 });
  }

  function movePlayer() {
    if (keys["a"] || keys["ArrowLeft"]) playerX -= speed;
    if (keys["d"] || keys["ArrowRight"]) playerX += speed;
    if (keys["w"] || keys["ArrowUp"]) playerY -= speed;
    if (keys["s"] || keys["ArrowDown"]) playerY += speed;

    playerX = Math.max(0, Math.min(playerX, 350));
    playerY = Math.max(0, Math.min(playerY, 500));

    player.style.left = playerX + "px";
    player.style.top = playerY + "px";
  }

  function moveEnemies() {
    for (let i = cars.length - 1; i >= 0; i--) {
      cars[i].y += cars[i].speed;
      cars[i].el.style.top = cars[i].y + "px";

      if (cars[i].y > 600) {
        game.removeChild(cars[i].el);
        cars.splice(i, 1);
        score++;
        scoreBoard.innerText = "Score: " + score;
      }

      // Collision
      const rect1 = player.getBoundingClientRect();
      const rect2 = cars[i].el.getBoundingClientRect();
      if (
        rect1.x < rect2.x + rect2.width &&
        rect1.x + rect1.width > rect2.x &&
        rect1.y < rect2.y + rect2.height &&
        rect1.y + rect1.height > rect2.y
      ) {
        gameOver = true;
        gameOverScreen.style.display = "flex";
      }
    }
  }

  function resetGame() {
    score = 0;
    playerX = 175;
    playerY = 480;
    gameOver = false;
    cars.forEach(c => c.el.remove());
    cars = [];
    gameOverScreen.style.display = "none";
    scoreBoard.innerText = "Score: 0";
  }

  function gameLoop() {
    if (!gameOver) {
      movePlayer();
      if (Math.random() < 0.03) spawnEnemy();
      moveEnemies();
    }
    requestAnimationFrame(gameLoop);
  }

  document.addEventListener("keydown", (e) => {
    keys[e.key.toLowerCase()] = true;
    if (gameOver && e.key.toLowerCase() === "r") resetGame();
  });

  document.addEventListener("keyup", (e) => {
    keys[e.key.toLowerCase()] = false;
  });

  gameLoop();
</script>
</body>
</html>
