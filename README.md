<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tetris</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
    }
    canvas {
      border: 1px solid #000;
      display: block;
      margin: 0 auto;
    }
    .cell {
      width: 30px;
      height: 30px;
      border: 1px solid #ddd;
      box-sizing: border-box;
    }
  </style>
</head>
<body>
  <canvas id="tetris" width="300" height="600"></canvas>
  <br>
  <button onclick="playerMove(-1)">Move Left</button>
  <button onclick="playerMove(1)">Move Right</button>
  <button onclick="playerDrop()">Drop</button>
  <button onclick="rotatePlayer()">Rotate</button>
  <button onclick="togglePause()">Pause</button>
  <script>
    const canvas = document.getElementById('tetris');
    const context = canvas.getContext('2d');

    const ROWS = 20;
    const COLS = 10;
    const BLOCK_SIZE = 30;

    const arena = createMatrix(COLS, ROWS);

    let lastTime = 0;
    let dropCounter = 0;
    let dropInterval = 1000;
    let player = {
      pos: { x: 0, y: 0 },
      matrix: null,
      score: 0,
      isPaused: false
    };

    document.addEventListener('keydown', event => {
      if (!player.isPaused) {
        if (event.keyCode === 37) {
          playerMove(-1);
        } else if (event.keyCode === 39) {
          playerMove(1);
        } else if (event.keyCode === 40) {
          playerDrop();
        } else if (event.keyCode === 38) {
          rotatePlayer();
        }
      }
      if (event.keyCode === 32) {
        togglePause();
      }
    });

    function togglePause() {
      player.isPaused = !player.isPaused;
    }

    function rotatePlayer() {
      rotate(player.matrix);
    }

    function rotate(matrix) {
      for (let y = 0; y < matrix.length; ++y) {
        for (let x = 0; x < y; ++x) {
          [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
        }
      }
      matrix.forEach(row => row.reverse());
    }

    function playerDrop() {
      player.pos.y++;
      if (collide(arena, player)) {
        player.pos.y--;
        merge(arena, player);
        playerReset();
        arenaSweep();
        updateScore();
      }
      dropCounter = 0;
    }

    function playerMove(offset) {
      player.pos.x += offset;
      if (collide(arena, player)) {
        player.pos.x -= offset;
      }
    }

    function createMatrix(width, height) {
      const matrix = [];
      while (height--) {
        matrix.push(new Array(width).fill(0));
      }
      return matrix;
    }

    function collide(arena, player) {
      const [m, o] = [player.matrix, player.pos];
      for (let y = 0; y < m.length; ++y) {
        for (let x = 0; x < m[y].length; ++x) {
          if (m[y][x] !== 0 && (arena[y + o.y] && arena[y + o.y][x + o.x]) !== 0) {
            return true;
          }
        }
      }
      return false;
    }

    function merge(arena, player) {
      player.matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            arena[y + player.pos.y][x + player.pos.x] = value;
          }
        });
      });
    }

    function playerReset() {
      const pieces = 'TJLOSZI';
      player.matrix = createPiece(pieces[pieces.length * Math.random() | 0]);
      player.pos.y = 0;
      player.pos.x = (COLS / 2 | 0) - (player.matrix[0].length / 2 | 0);
      if (collide(arena, player)) {
        arena.forEach(row => row.fill(0));
        player.score = 0;
        updateScore();
      }
    }

    function createPiece(type) {
      if (type === 'T') {
        return [
          [0, 0, 0],
          [1, 1, 1],
          [0, 1, 0],
        ];
      } else if (type === 'J') {
        return [
          [0, 0, 0],
          [2, 2, 2],
          [0, 0, 2],
        ];
      } else if (type === 'L') {
        return [
          [0, 0, 0],
          [3, 3, 3],
          [3, 0, 0],
        ];
      } else if (type === 'O') {
        return [
          [4, 4],
          [4, 4],
        ];
      } else if (type === 'S') {
        return [
          [0, 0, 0],
          [0, 5, 5],
          [5, 5, 0],
        ];
      } else if (type === 'Z') {
        return [
          [0, 0, 0],
          [6, 6, 0],
          [0, 6, 6],
        ];
      } else if (type === 'I') {
        return [
          [0, 0, 0, 0],
          [7, 7, 7, 7],
          [0, 0, 0, 0],
          [0, 0, 0, 0],
        ];
      }
    }

    function draw() {
      context.fillStyle = '#000';
      context.fillRect(0, 0, canvas.width, canvas.height);

      drawMatrix(arena, { x: 0, y: 0 });
      drawMatrix(player.matrix, player.pos);
    }

    function drawMatrix(matrix, offset) {
      matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            context.fillStyle = colors[value];
            context.fillRect(x * BLOCK_SIZE + offset.x, y * BLOCK_SIZE + offset.y, BLOCK_SIZE, BLOCK_SIZE);
            context.strokeStyle = '#fff';
            context.strokeRect(x * BLOCK_SIZE + offset.x, y * BLOCK_SIZE + offset.y, BLOCK_SIZE, BLOCK_SIZE);
          }
        });
      });
    }

    function arenaSweep() {
      let rowCount = 1;
      outer: for (let y = arena.length - 1; y > 0; --y) {
        for (let x = 0; x < arena[y].length; ++x) {
          if (arena[y][x] === 0) {
            continue outer;
          }
        }
        const row = arena.splice(y, 1)[0].fill(0);
        arena.unshift(row);
        ++y;

        player.score += rowCount * 10;
        rowCount *= 2;
      }
    }

    function updateScore() {
      document.getElementById('score').innerText = player.score;
    }

    const colors = [
      null,
      '#FF0D72',
      '#0DC2FF',
      '#0DFF72',
      '#F538FF',
      '#FF8E0D',
      '#FFE138',
      '#3877FF',
    ];

    function update(time = 0) {
      if (!player.isPaused) {
        const deltaTime = time - lastTime;
        lastTime = time;

        dropCounter += deltaTime;
        if (dropCounter > dropInterval) {
          playerDrop();
        }
      }

      draw();
      requestAnimationFrame(update);
    }

    update();
  </script>
  <div>Score: <span id="score">0</span></div>
</body>
</html>

