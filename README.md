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

    #level {
      margin-top: 10px;
      font-size: 16px;
    }

    #highScore {
      margin-top: 10px;
      font-size: 16px;
    }

    #game-over {
      display: none;
      margin-top: 20px;
      font-size: 30px;
      color: red;
      font-weight: bold;
    }
    body {
      font-family: 'Arial', sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      background-color: #f0f0f0;
    }

    canvas {
      border: 1px solid #000;
    }

    #score,
    #level,
    #highScore,
    #game-over {
      margin-top: 20px;
      font-size: 20px;
      text-align: center;
    }

    #game-over {
      display: none;
      color: red;
      font-weight: bold;
    }

    .controls {
      margin-top: 20px;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
    }

    .controls button {
      margin: 5px;
      padding: 10px 20px;
      font-size: 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .controls button:hover {
      background-color: #45a049;
    }

    #nextPiecePreview {
      margin-top: 20px;
      border: 1px solid #000;
      width: 80px;
      height: 80px;
    }
  </style>
</head>
<body>
  <canvas id="tetrisCanvas" width="300" height="600"></canvas>
  <div id="score">Score: 0</div>
  <div id="level">Level: 1</div>
  <div id="highScore">High Score: 0</div>
  <div id="game-over">Game Over!</div>
  <div id="nextPiecePreview"></div>

   <audio id="moveSound" src="MrLololoshka_Roman_ilchenkov_-_Velikoe_plamya_Trinadcat_Ognejj_77359424.mp3"></audio>
  <script>
    const canvas = document.getElementById('tetrisCanvas');
    const ctx = canvas.getContext('2d');
    const blockSize = 30;
    const rows = 20;
    const columns = 10;
    let board = Array.from({ length: rows }, () => Array(columns).fill(0));
    let currentPiece = generatePiece();
    let score = 0;
    let level = 1;
    let highScore = localStorage.getItem('highScore') || 0;
    let gameOver = false;
    let gameSpeed = 500; // Initial game speed in milliseconds
    let lastMoveDown = Date.now();
    let linesPerLevel = 10;
    let linesClearedTotal = 0;
    let isPaused = false;
    let animationId; // To store requestAnimationFrame ID

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

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBoard();
      drawPiece();
      document.getElementById('score').textContent = `Score: ${score}`;
      document.getElementById('level').textContent = `Level: ${level}`;
      document.getElementById('highScore').textContent = `High Score: ${highScore}`;

      if (gameOver) {
        document.getElementById('game-over').style.display = 'block';
      }
    }

    function generatePiece() {
      const pieces = [
        { shape: [[1, 1, 1, 1]], color: 'cyan' },
        { shape: [[1, 1, 1], [1]], color: 'blue' },
        { shape: [[1, 1, 1], [0, 0, 1]], color: 'orange' },
        { shape: [[1, 1, 1], [1, 0]], color: 'yellow' },
        { shape: [[1, 1], [1, 1]], color: 'red' },
        { shape: [[1, 1, 0], [0, 1, 1]], color: 'green' },
        { shape: [[0, 1, 1], [1, 1]], color: 'purple' },
      ];
      const randomIndex = Math.floor(Math.random() * pieces.length);
      const piece = pieces[randomIndex];
      return {
        shape: piece.shape,
        color: piece.color,
        x: Math.floor((columns - piece.shape[0].length) / 2),
        y: 0,
      };
    }

    function moveDown() {
      if (!gameOver && isValidMove(0, 1)) {
        currentPiece.y++;
        lastMoveDown = Date.now(); // Update last move down time
      } else if (!gameOver) {
        mergePiece();
        clearLines();
        currentPiece = generatePiece();
        if (!isValidMove(0, 0)) {
          gameOver = true;
          document.getElementById('gameOverSound').play();
        }
      }
    }

    function moveLeft() {
      if (!gameOver && isValidMove(-1, 0)) {
        currentPiece.x--;
        document.getElementById('moveSound').play();
      }
    }

    function moveRight() {
      if (!gameOver && isValidMove(1, 0)) {
        currentPiece.x++;
        document.getElementById('moveSound').play();
      }
    }

    function rotate() {
      const rotatedPiece = {
        shape: currentPiece.shape.map((_, i) => currentPiece.shape.map(row => row[i])).reverse(),
        color: currentPiece.color,
        x: currentPiece.x,
        y: currentPiece.y,
      };

      if (!gameOver && isValidMove(0, 0, rotatedPiece)) {
        currentPiece.shape = rotatedPiece.shape;
        document.getElementById('moveSound').play();
      }
    }

    function rotateClockwise() {
      const rotatedPiece = {
        shape: currentPiece.shape[0].map((_, i) => currentPiece.shape.map(row => row[i])).reverse(),
        color: currentPiece.color,
        x: currentPiece.x,
        y: currentPiece.y,
      };

      if (!gameOver && isValidMove(0, 0, rotatedPiece)) {
        currentPiece.shape = rotatedPiece.shape;
        document.getElementById('moveSound').play();
      }
    }

    function moveDrop() {
      while (isValidMove(0, 1)) {
           moveDown();
      }
    }

    function moveUp() {
      if (!gameOver && isValidMove(0, -1)) {
        currentPiece.y--;
        document.getElementById('moveSound').play();
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
          linesClearedTotal++;
        }
      }
      if (linesCleared > 0) {
        score += linesCleared * 100;
        // Increase game speed after clearing lines
        gameSpeed = Math.max(100, gameSpeed - linesCleared * 10);
        // Check if it's time to level up
        if (linesClearedTotal >= linesPerLevel * level) {
          level++;
          document.getElementById('level').textContent = `Level: ${level}`;
        }
        // Update high score if current score is higher
        if (score > highScore) {
          highScore = score;
          document.getElementById('highScore').textContent = `High Score: ${highScore}`;
          // Save high score to local storage
          localStorage.setItem('highScore', highScore);
        }
        // Play line clear sound effect
        document.getElementById('lineClearSound').play();
      }
    }

    function update() {
      const currentTime = Date.now();
      if (!isPaused && currentTime - lastMoveDown > gameSpeed) {
        moveDown();
      }
    }

    function gameLoop() {
      update();
      draw();
      animationId = requestAnimationFrame(gameLoop);
    }

    document.addEventListener('keydown', (event) => {
      if (event.key === 'ArrowLeft') {
        moveLeft();
      } else if (event.key === 'ArrowRight') {
        moveRight();
      } else if (event.key === 'ArrowDown') {
        moveDown();
      } else if (event.key === 'ArrowUp') {
        rotate();
      } else if (event.key === 'x') {
        // "X" key for toggling pause/resume
        isPaused = !isPaused;
        if (isPaused) {
          cancelAnimationFrame(animationId); // Stop game loop when paused
        } else {
          gameLoop(); // Resume game loop when unpaused
        }
      } else if (event.key === 'c') {
        // "C" key for changing the position of the piece
        moveUp();
      } else if (event.key === ' ') {
        moveDrop();
      } else if (event.key === 'z') {
        // "Z" key for clockwise rotation
        rotateClockwise();
      }
    });

    gameLoop();
     
  </script>
  <p>&copy; 2024 Разработчик  Dylan933 Все права защищены. | <span id="companyLink"></span></p>
<
