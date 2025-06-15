
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Crossroads Game - Working Base64 Images</title>
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

  // Simple colored rectangles as images encoded as base64 PNGs
  const roadImgSrc = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAZAAAABkCAYAAADqghb9AAAABHNCSVQICAgIfAhkiAAABq9JREFUeJzt3QeIVUeWx/HfB3bOp5HTZbAseOaYozWUpCgUg9CrsZCys0zWoLUCEylEGiKrKK3VhhpjvYISwNRqKS2ZoQ1HQGEwoQRcRcKHcizb6ZPfof+01vP35PZce+/v6nPx7d+9qYzZ95f3nrvO3tv32du1y+P3v6v6/TxS00ttzPtpv3HPY3w6POU5+2vT9Aq/CnPQT+UWxk13Tv8jwBPo6iVChK/0QoUol8JIlShR+AMJC6Ro/4IQItLpNk3i+U4x8uQvNPX2JfSNGRe7DfqvSNTXdeVITetxxAPcxrIL0d0y5E7QNvSxT+MeD8hDaCj5FoCx0EY2VseS0qsyiZUz5Ru1ziIm3uH7PcpITnxzIh93OLINRwqU0fTWFSJD7Te9zqyK+uLQ/TljO0fXPnLUQpm+zfuq/ncEm+c7JMgNLT9zTf3mG3Q9XoGpFmhVrUY6nqXZbCVHkyzN0c6Nf0jq4ntn5C3D9/Vu+6TgqXn9HtT6VytW/kd5ILxf81qPlAq7r0G1X/n8X7B9NqclOtWv1P1E4rqK9QfmlR77LcH8HTqHTqjXqwFq6XTdAjPxqet1oNmzY2L/Xmsfv/kJpuP+0f+d6ff+VZf0Xz/B5RY/Xk+rEK+lhR4m8LQYXeWv5/Ucxt9f+6fCqcvvydJ6rfvPxzzDYx7Nm4q8DHjNLnNlYq+bpHkoX1t6fqXl+tKX6aLPxx06H60/nV2NdLrgFr3hNpN57XsLL2VjKXPi17C33XsBR+ae9B2uD8+4pKzX0eWeqPq29RXzLP7XvzPM/8B+w09RZaEboFHdNdtSHWHTJDo+T0+Hxj3XzKuZr2i7V45f8rZxPyrOyr8czH/PZjV+a99B9n3rCqcr+jxxa/L7x+xuKvN0tLl1NqzX7tHlSe5iNJjti07NrMY21lT9C7wvnl7nGk8DWlWtWOdWqH++yVN+vLrN63nPcf8X5XSXdHndFhq6H1+/6R+c/r5/n+H4PPX1+3DZLRdOa7P0DNl7QbVQcAAAAASUVORK5CYII=';
  const playerImgSrc = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAZCAYAAAB89yFLAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAELUlEQVRoge2Wf2gUVRTHP2e+v8k3lEmaFQrFlJj6Fg9AmCWAUr5UTG4fYchR4icvR+ZEEH/kAKKI8UGcVLVImPgAL5wNQcSXp3VQkFZ1aIWy3HRuGyqXhU1ZEGUVgpFrtxqgIhHGIiVRBVQqUqfPa3S4vNTcczMdmbsfN+dmRmTZPznDz3v9QpZ1t79bb+90z6ZKTd7y7bbjJ89jMPlPTz/7sZdxOX7xmfrTqthnG93IZDEZjHeYZ9E4bkcdBGsUsG+cRhHLsdRLSc5R3PE9xFSc9v6lCv+EYLjsVziPN6tNPT8vHm6M06k3vF6+0Hxgh5qVRdPXxc0Ge/e6uHcE7s9a77nlb9XPzKKrS/v5xn9dhyd4U8V3HMxrve+uA7XMe7vl0zN66zTyYsEj8xa5db4/96Oz2X1Q53+9P7wLyQnrLtXMgDHfdUeA+hwJ1XMqXvXLDa+vNU06T7a4cOPbX1a+fUO7McN65kGH9z9xrM+4Tc6eqTUzc1bOq/DRXOV/Xdyv6PeqrxC7Z02/Xk4G4m/NOfWdP5k5XVyX+g0dVwc5W2rP2xqPo7rOHYvtZzDzWVaY7OFGldPTMM9m3X9z86wZ+6flQjz6yipXdMxhPprH27Z6F/Zmnnv0GzOPEftNVOn07iRr+kGZbRyD7yL8+9Nz0bPRnPntZaP6PRn9R6vlvQnf+YJm7pjV1cdNU2nvNtZ7x4bRlOx3b3GHPrxPa4xuPje0sdX8b28GrY3rfZmy5bd4mnn70+p+5dMtbsPXoTD6XqdpTeHMCcvY0aPeWlXPhc09nvkBr50ZjMe3DLy4d4VdNiPY3rb9Z1P7efTOJbh+nD0cPQvfo7tpjvHQqO5fXEd49xpORdXq+VC8/3zp1GQz3GHeW1uD51GQrMuOLZHzJzHXfge+R3XhXPUSu7L78znJ0qvLgn6aGz+9KXf86v+bZtF+RXsvF9X7fl+yVyG+hLxYcavXCyhfHpTuX0WLjlAXu6jxm8K/9Dj+0hUvC1g+Ko1Ke/fz0xlxq+jRmfG78jlXOz1v0gP/zj8fbxnpz/p+V/jXD/gVQcg9TuEQqIpFYqFQqFSqVQqFQoFQoVCoVIoVCoVC4XxPZf9wGAkgOJRYcK1zAAAAAElFTkSuQmCC';
  const enemyImgSrc = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAZCAYAAAB89yFLAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAEXklEQVRoge2XQU4UURiFZ1qwC4kbQkR1JRQIRpHQikVmoyQhURRhLsiKFxYMTUTcQkQmycsINLCJ0tCErZCwhwlObkmhnOmvRmUwsQkWlZz

