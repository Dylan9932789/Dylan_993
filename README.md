
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tetris</title>
  <style>
    body {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      font-family: 'Arial', sans-serif;
    }

    canvas {
      border: 1px solid #000;
    }

    #score {
      margin-top: 20px;
      font-size: 20px;
    }

    #game-controls {
      margin-bottom: 20px;
    }

    #game-controls button {
      margin-right: 10px;
    }

    #game-over {
      display: none;
      margin-top: 20px;
      font-size: 30px;
      color: red;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div id="game-controls">
    <button id="start-pause-btn">Start / Pause</button>
    <button id="restart-btn">Restart</button>
    <button id="sound-toggle-btn">Toggle Sound</button>
  </div>
  <canvas id="tetrisCanvas" width="300" height="600"></canvas>
  <div id="score">Score: 0</div>
  <div id="game-over">Game Over!</div>

  <script src="https://cdn.lordicon.com/lordicon.js"></script>
  <lord-icon
    src="https://cdn.lordicon.com/bzqvamqv.json"
    trigger="hover"
    style="width:100px;height:100px">
  </lord-icon>

  <script>
    const canvas = document.getElementById('tetrisCanvas');
    const ctx = canvas.getContext('2d');
    const blockSize = 30;
    const rows = 20;
    const columns = 10;
    const initialGameSpeed = 500;
    const pieceQueue = [];
    const pieceColors = ['cyan', 'blue', 'orange', 'yellow', 'red', 'green', 'purple'];
    let board, currentPiece, nextPiece, score, gameOver, gameSpeed, lastMoveDown, isPaused;

    function initialize() {
      board = Array.from({ length: rows }, () => Array(columns).fill(0));
      score = 0;
      gameOver = false;
      gameSpeed = initialGameSpeed;
      lastMoveDown = Date.now();
      isPaused = false;
      generateNewPiece();
    }

    function generateNewPiece() {
      if (pieceQueue.length === 0) {
        pieceColors.sort(() => Math.random() - 0.5);
        pieceColors.forEach(color => pieceQueue.push({ color }));
      }

      const nextPieceInfo = pieceQueue.shift();
      nextPiece = {
        shape: getRandomPieceShape(),
        color: nextPieceInfo.color,
        x: Math.floor((columns - nextPieceInfo.shape[0].length) / 2),
        y: 0,
      };
    }

    function getRandomPieceShape() {
      const pieces = [
        [[1, 1, 1, 1]],          // I
        [[1, 1, 1], [1]],        // J
        [[1, 1, 1], [0, 0, 1]],  // L
        [[1, 1, 1], [1, 0]],     // O
        [[1, 1], [1, 1]],        // S
        [[0, 1, 1], [1, 1]],     // T
        [[1, 1, 0], [0, 1, 1]]   // Z
      ];
      return pieces[Math.floor(Math.random() * pieces.length)];
    }

    function drawSquare(x, y, color) {
      ctx.fillStyle = color;
      ctx.fillRect(x * blockSize, y * blockSize, blockSize, blockSize);
      ctx.strokeStyle = "#000";
      ctx.strokeRect(x * blockSize, y * blockSize, blockSize, blockSize);
    }

    function drawBoard() {
      for (let row = 0; row < rows; row++) {
        for (let col = 0; col < columns; col++) {
          if (board[row][col] !== 0) {
            drawSquare(col, row, board[row][col]);
          }
        }
      }
    }

    function drawPiece() {
      currentPiece.shape.forEach((row, i) => {
        row.forEach((cell, j) => {
          if (cell !== 0) {
            drawSquare(currentPiece.x + j, currentPiece.y + i, currentPiece.color);
          }
        });
      });
    }

    function drawNextPiecePreview() {
      // Your code to draw the next piece preview (optional)
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBoard();
      drawPiece();
      drawNextPiecePreview();
      document.getElementById('score').textContent = `Score: ${score}`;

      if (gameOver) {
        document.getElementById('game-over').style.display = 'block';
      }
    }

    function moveDown() {
      if (!gameOver && isValidMove(0, 1)) {
        currentPiece.y++;
      } else if (!gameOver) {
        mergePiece();
        clearLines();
        currentPiece = nextPiece;
        generateNewPiece();
        if (!isValidMove(0, 0)) {
          gameOver = true;
          handleGameOver();
        }
      }
    }

    function isValidMove(offsetX, offsetY, piece = currentPiece) {
      for (let i = 0; i < piece.shape.length; i++) {
        for (let j = 0; j < piece.shape[i].length; j++) {
          if (
            piece.shape[i][j] !== 0 &&
            (board[piece.y + i + offsetY] && board[piece.y + i + offsetY][piece.x + j + offsetX]) !== 0
          ) {
            return false;
          }
        }
      }
      return true;
    }

    function mergePiece() {
      currentPiece.shape.forEach((row, i) => {
        row.forEach((cell, j) => {
          if (cell !== 0) {
            board[currentPiece.y + i][currentPiece.x + j] = currentPiece.color;
          }
        });
      });
    }

    function clearLines() {
      let linesCleared = 0;
      for (let row = rows - 1; row >= 0; row--) {
        if (board[row].every(cell => cell !== 0)) {
          board.splice(row, 1);
          board.unshift(Array(columns).fill(0));
          linesCleared++;
        }
      }
      if (linesCleared > 0) {
        score += linesCleared * 100;
        gameSpeed = Math.max(100, gameSpeed - linesCleared * 10);
      }
    }

    function update() {
      const currentTime = Date.now();
      if (!isPaused && currentTime - lastMoveDown > gameSpeed) {
        moveDown();
        lastMoveDown = currentTime;
      }
    }

    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    function handleGameOver() {
      // Your code to handle game over (e.g., stop game loop, display score, etc.)
    }

    function restartGame() {
      initialize();
      requestAnimationFrame(gameLoop);
    }

    function startPauseGame() {
      isPaused = !isPaused;
      document.getElementById('start-pause-btn').textContent = isPaused ? 'Resume' : 'Pause';
      if (!gameOver) {
        requestAnimationFrame(gameLoop);
      }
    }

    function handleSoundToggleClick() {
      // Your code to toggle sound effects (optional)
    }

    document.getElementById('start-pause-btn').addEventListener('click', startPauseGame);
    document.getElementById('restart-btn').addEventListener('click', restartGame);
    document.getElementById('sound-toggle-btn').addEventListener('click', handleSoundToggleClick);

    initialize();
    gameLoop();
  </script>


