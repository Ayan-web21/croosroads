<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Crossroads Game - Self-contained</title>
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

  // Base64 encoded images (road, red car, blue car)
  const roadImgSrc = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAYAAABccqhmAAAABHNCSVQICAgIfAhkiAAABzVJREFUeJzt3TFv20kQheGXcwTiD//W+uAJuMGfJuCbOpd/2HNmTiHEOXLHqlrjYOBHbyM8r6K9AKBDnTs1kq2KjSknhRJM1AoFgqFrNJg6Nkf8y+W0I3pq/x8aP5P9q+/nLvuMxp8/rvs57NzqQSRJEmSJEmSJEmSJEmSJEmSJEmSJEmSxPeHQR7Hpgu+Mth0iAt1Q2Oe95k8tdWMiE3BvLq2fEyXFE6StDcT9c14Db2GPh37uHTMQy+QXALFvGkH6lDk7rNwt6V/CpPP+qtxvE82YxP41ifw3ig3gIHjeHRBX7N9n1Yhh0j74iS9yZsv8Y6G9xdQwHCXe0kz60Y9Gn6egX1A7a0CvxCcOLfiv5aTf4Dx02AG/BneGRR7UuIP3tKnDRaQEtLkFfWZ6kHekOo5CsbdcQ/GQWPd35Hcn6z3f5xlX1KQPMRmkN+FsA1rP+d4e4IDrglRUYY9JjT6PYA88X7FAvE1EgA9nkaGmrHbMqvZlxnEYPr04KM/K5P8RHlfISd0od8DRmObyx6K6CP9RlKKWDV8BDjW6vA5riqOuYnxu57XHH7km/66J+AT59ccKw3E89ClLZrW39W7eDK2djfQ7fsT8+7A94EHQrbMdXZJKvnNQv6rKJdzNhK18B1m31HJ/4JbZ69Tb78PwG9J1+XYr1gt6ck9xltdBOq8+1P8n/QB9Bv0MsHVdG0BZd6pyQozFbCBY0zBdoIpWbSQcHOjO/17wz2YOofbX7HqnSllXgHKqxhvGoGrYlelt4K9VkFtIDGDtjveQqDDtU+lZL8CCwq/Q/ukT7uJvMTgvUy/GgP9N9nmvGP14TqTf4dbk4u8LM0FX6rCjPBpJctg6wI59qknuqv6wQsvVfpAAu3q9P1cw6QvazC38GoTxqulSPRYZ8NMCv8tYlwfvDcC1F1EO78XSlHtW3Eq6/mMUPsHNjNCH7Y1u+7Y8b5WenbP6CNqjL+Ayq1wr/j9d0i2jfdwbs8Jdp7kZJkyRJkiRJkiRJkiRJkiRJkiRJkiRJkv39H/wAv84WqWc82VMAAAAASUVORK5CYII=';
  const playerImgSrc = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAZCAYAAAB89yFLAAAACXBIWXMAAAsTAAALEwEAmpwYAAAHEklEQVRoge2Xz2sUURTHP3Of7b2IUaKraB9DBxIqrSpIcRJcoIYVqQ/IT+wHkm4EU8Awe7SAkuIxR0pqkuQtmSiRCdHYklBWqRQQUipbiKAXFpJU+2sjHrfv92Z15AruCrf7+bf/Nz3ndM2vMkHwlIj/ZZ9spRx6Rb2HfXFEG9h7eFHXnOdhuQXwFwji7hmcEjewCxQOLZ7Dlve5DjXCPsGZQYhGuBguZQ0vDKD0m7kp0GWtBpKHbzvY7lXOcZhz2JAV+U1Q0JIGXgwxQoRmw6O+icOBKzELbDAnNjOKdoT5wD0+BjPG9Fo0Z2+C2Ng3N7qOGHqQCU0iCUWk3w3YZyWIEdDKeQXg22Nmy8DHRAc2YFx7dBQjzMxNPbX2VvjNQ6scclXB7wBv2uwDFYfvoB9xxvlMh50bcMPphzwNq56QlxkEPlM59F0mDc6nd1vE84wrx2MKHyfBrbHo0MF0ZEBU6Ld6IfRUXGckBQqGd8nY8FwV3AN+LdPJB0Q0Gy8yrDAv7BBWEMT6h6Y96Y3e0DeQhHRdx5GRPIMjDs+GgfuYLqH44RoXTU3QGfXv8rg4vEr+/Mjr9yLeFDZrA6ztbS8JvGQzGu1YefFfQap59ycyRx1Af8AM0HVAjq9bqvQAAAABJRU5ErkJggg==';
  const enemyImgSrc = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAZCAYAAAB89yFLAAAACXBIWXMAAAsTAAALEwEAmpwYAAAHR0lEQVRoge2Xv2tUQRSGv9CjQ4aQWB6GoWGFAgx0/gEUtNBQVRJqSrWKtiKYhKJSFQIEAEWvZ6Kh3YgEXFhGIfRTTZQEtQTX9S7p/s/n5nlmXlmdmXkrvOfd7ve/zfu5Mz2ZB+1AY3nHO4hp1mC9cnOWdLvILh0/f8H0CPzQHlM35z1BtgztANHE9yN2CvPQO0Rk9iHFiqKxTImbsmPEzQLZ4aNcYQ2IUkFrKsmfArYB/eV0bV0z8PnAf4i3iCwgXa6aB2PhbIK0B5cfb8rF7ATu5v0k+AzLPp9G8AVxbRXYGfQP+jrDsCv2vHge/JrBfAf6IOLzwZ4HcEzy4i+ScxQ/BJwM+54KnK66zBQYOL6KUsXEBjDHscT3RmXQQrD0r26iD4K+DyZ5H6Gb6JqG0rNMZc3pP35HLxURwqj6M5pZDzPB6aDc6QTAfRg5ccXBy5xI+OWChnSQ7FzOYD/W0zYXC2LOA/kpTRzAwlPjvmz57qOQjGpM7IPyzLOnQpeOXU6GcxR+U8Qo37uE1AnscD95p+rcDLr5nN3cOTqcy2NjC9Np7nQYqL0i7E09DoP04sdAaU0L+g9Z9U8LmB78V4NbZZfEywTscBn9R+9HXg7WEwxNHlEj/8T2rTp4GShah2twMr3PPpdMr/j0jNPvdeM+XPm6Zk9y0PzAldS5AcM1+Q4KPpA/tPxkQkyc6U7wJvA0eEoPX7bZLOFAx3OmDH7g0wRGf5aL4XEEi/A96kgThORd6dAUZ/Et9j8O+A9RAHOry3LxQ8AoA4U/C4IPD6df5WnN3xof+/ArEXALGE6zIuCdf8AAAAASUVORK5CYII=';

  const roadImage = new Image();
  const playerImage = new Image();
  const enemyImage = new Image();

  roadImage.src = roadImgSrc;
  playerImage.src = playerImgSrc;
  enemyImage.src = enemyImgSrc;

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

  roadImage.onload = () => {
    loop();
  }
</script>
</body>







