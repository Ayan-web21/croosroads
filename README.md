 <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Crossroads Game</title>
  <style>
    body {
      margin: 0;
      background: #333;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
      font-family: sans-serif;
      color: white;
    }
    #game {
      position: relative;
      width: 400px;
      height: 600px;
      background: url('https://i.ibb.co/47rjHXT/road-background.png') no-repeat center center;
      background-size: cover;
      border: 4px solid #555;
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
    .car {
      position: absolute;
      width: 50px;
      height: 100px;
      background-size: contain;
      background-repeat: no-repeat;
    }
    #score {
      position: absolute;
      top: 10px;
      left: 10px;
      font-size: 18px;
      font-weight: bold;
      text-shadow: 1px 1px 3px black;
    }
    #gameOver {
      position: absolute;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.7);
      color: yellow;
      font-size: 24px;
      display: none;
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
      <div>Game Over!</div>
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

    const lanes = [50, 150, 250, 350];
    let playerX = 175;
    let playerY = 480;
    let speed = 5;

    const enemyImages = [
      'https://i.ibb.co/fCnL7MF/blue-car.png',
      'https://i.ibb.co/1zptm1D/yellow-car.png',
      'https://i.ibb.co/FVZ7Hvc/green-car.png'
    ];

    function createCar() {
      const car = document.createElement("div");
      car.classList.add("car");
      const lane = lanes[Math.floor(Math.random() * lanes.length)];
      const img = enemyImages[Math.floor(Math.random() * enemyImages.length)];
      car.style.backgroundImage = `url(${img})`;
      car.style.left = lane + "px";
      car.style.top = "-120px";
      game.appendChild(car);
      cars.push({ el: car, y: -120, lane, speed: 3 + Math.random() * 2 });
    }

    function movePlayer() {
      if (keys["a"] || keys["ArrowLeft"]) playerX -= speed;
      if (keys["d"] || keys["ArrowRight"]) playerX += speed;
      if (keys["w"] || keys["ArrowUp"]) playerY -= speed;
      if (keys["s"] || keys["ArrowDown"]) playerY += speed;

      // bounds
      if (playerX < 0) playerX = 0;
      if (playerX > 350) playerX = 350;
      if (playerY < 0) playerY = 0;
      if (playerY > 500) playerY = 500;

      player.style.left = playerX + "px";
      player.style.top = playerY + "px";
    }

    function updateCars() {
      for (let i = cars.length - 1; i >= 0; i--) {
        const car = cars[i];
        car.y += car.speed;
        car.el.style.top = car.y + "px";

        if (car.y > 600) {
          car.el.remove();
          cars.splice(i, 1);
          score++;
        }

        // collision
        const rect1 = player.getBoundingClientRect();
        const rect2 = car.el.getBoundingClientRect();
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
      gameOver = false;
      gameOverScreen.style.display = "none";
      playerX = 175;
      playerY = 480;
      cars.forEach(c => c.el.remove());
      cars = [];
    }

    function gameLoop() {
      if (!gameOver) {
        movePlayer();
        if (Math.random() < 0.03) createCar();
        updateCars();
        scoreBoard.innerText = "Score: " + score;
      }
      requestAnimationFrame(gameLoop);
    }

    window.addEventListener("keydown", e => {
      keys[e.key.toLowerCase()] = true;
      if (gameOver && e.key.toLowerCase() === "r") resetGame();
    });

    window.addEventListener("keyup", e => {
      keys[e.key.toLowerCase()] = false;
    });

    gameLoop();
  </script>
</body>
</html>

