
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Тетрис</title>
  <link rel="stylesheet" href="styles.css">
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
  font-size: 18px;
}

#game-over {
  display: none;
  margin-top: 20px;
  font-size: 30px;
  color: red;
  font-weight: bold;
}

#next-piece-canvas {
  border: 1px solid #000;
  margin-top: 20px;
}
  </style>
</head>
<body>
  <canvas id="tetrisCanvas" width="300" height="600"></canvas>
  <div id="score">Счет: 0</div>
  <div id="level">Уровень: 1</div>
  <div id="game-over">Game Over!</div>
  <canvas id="next-piece-canvas" width="100" height="100"></canvas>
  <button id="sound-button">Звук 🔊</button>
  <button id="reset-button">начать заново 🔄</button>
  <button id="pause-resume-button">Пауза</button>
  <div id="touch-controls">
    <button id="left-button">Лево</button>
    <button id="right-button">Право</button>
    <button id="rotate-button">Вращать</button>
    <button id="down-button">Вниз</button>
    <button id="drop-button">Быстрое падение</button>
  </div>

  <script src="tetris.js"></script>
  <script>
    // Your JavaScript code
    const canvas = document.getElementById('tetrisCanvas');
    const ctx = canvas.getContext('2d');
    const blockSize = 30;
    const rows = 20;
    const columns = 10;
    let board = Array.from({ length: rows }, () => Array(columns).fill(0));
    let currentPiece = generatePiece();
    let nextPiece = generatePiece();
    let score = 0;
    let level = 1;
    let gameOver = false;
    let gameSpeed = 500; // Initial game speed in milliseconds
    let lastMoveDown = Date.now();
    let isPaused = false;

    const nextPieceCanvas = document.getElementById('next-piece-canvas');
    const nextPieceCtx = nextPieceCanvas.getContext('2d');

    // Touch events
    let touchStartX = 0;
    let touchStartY = 0;

    canvas.addEventListener('touchstart', handleTouchStart, false);
    canvas.addEventListener('touchmove', handleTouchMove, false);
    canvas.addEventListener('touchend', handleTouchEnd, false);

    function handleTouchStart(event) {
      touchStartX = event.touches[0].clientX;
      touchStartY = event.touches[0].clientY;
    }

    function handleTouchMove(event) {
      event.preventDefault();
      const touchX = event.touches[0].clientX;
      const touchY = event.touches[0].clientY;
      const deltaX = touchX - touchStartX;
      const deltaY = touchY - touchStartY;
      if (Math.abs(deltaX) > Math.abs(deltaY)) {
        if (deltaX > 0) {
          moveRight();
        } else {
          moveLeft();
        }
      } else {
        if (deltaY > 0) {
          moveDown();
        } else {
          rotate();
        }
      }
    }

    function handleTouchEnd(event) {
      touchStartX = 0;
      touchStartY = 0;
    }

    document.addEventListener('keydown', (event) => {
      if (!gameOver && !isPaused) {
        switch (event.key) {
          case 'ArrowLeft':
            moveLeft();
            break;
          case 'ArrowRight':
            moveRight();
            break;
          case 'ArrowDown':
            moveDown();
            break;
          case 'ArrowUp':
            rotate();
            break;
          case ' ':
            moveDrop();
            break;
        }
      }
    });

    const soundButton = document.getElementById('sound-button');
    soundButton.addEventListener('click', toggleSound);

    const resetButton = document.getElementById('reset-button');
    resetButton.addEventListener('click', resetGame);

    let soundEnabled = true;

    function toggleSound() {
      soundEnabled = !soundEnabled;
      const soundIcon = document.getElementById('sound-icon');
      if (soundEnabled) {
        soundIcon.textContent = '🔊';
      } else {
        soundIcon.textContent = '🔇';
      }
    }

    function updateScoreAndLevel() {
      document.getElementById('score').textContent = `Счет: ${score}`;
      document.getElementById('level').textContent = `Уровень: ${level}`;
    }

    const leftButton = document.getElementById('left-button');
    leftButton.addEventListener('click', moveLeft);

    const rightButton = document.getElementById('right-button');
    rightButton.addEventListener('click', moveRight);

    const downButton = document.getElementById('down-button');
    downButton.addEventListener('click', moveDown);

    const rotateButton = document.getElementById('rotate-button');
    rotateButton.addEventListener('click', rotate);

    const dropButton = document.getElementById('drop-button');
    dropButton.addEventListener('click', moveDrop);

    function displayGameOver() {
      if (gameOver) {
        document.getElementById('game-over').style.display = 'block';
      } else {
        document.getElementById('game-over').style.display = 'none';
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

    function drawSquare(x, y, color, context) {
      context.fillStyle = color;
      context.fillRect(x * blockSize, y * blockSize, blockSize, blockSize);
      context.strokeStyle = "#000";
      context.strokeRect(x * blockSize, y * blockSize, blockSize, blockSize);
    }

    function drawBoard() {
      for (let row = 0; row < rows; row++) {
        for (let col = 0; col < columns; col++) {
          if (board[row][col] !== 0) {
            drawSquare(col, row, board[row][col], ctx);
          }
        }
      }
    }

    function drawPiece(piece, context) {
      piece.shape.forEach((row, i) => {
        row.forEach((cell, j) => {
          if (cell !== 0) {
            drawSquare(piece.x + j, piece.y + i, piece.color, context);
          }
        });
      });
    }

    function drawNextPiece() {
      nextPieceCtx.clearRect(0, 0, nextPieceCanvas.width, nextPieceCanvas.height);
      const offsetX = (nextPieceCanvas.width - blockSize * nextPiece.shape[0].length) / 2;
      const offsetY = (nextPieceCanvas.height - blockSize * nextPiece.shape.length) / 2;

      drawPiece(nextPiece, nextPieceCtx);
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBoard();
      drawPiece(currentPiece, ctx);
      updateScoreAndLevel();
      displayGameOver();
    }

    function moveLeft() {
      if (!gameOver && isValidMove(-1, 0)) {
        currentPiece.x--;
      }
    }

    function moveRight() {
      if (!gameOver && isValidMove(1, 0)) {
        currentPiece.x++;
      }
    }

    function moveDown() {
      if (!gameOver && isValidMove(0, 1)) {
        currentPiece.y++;
      } else if (!gameOver) {
        mergePiece();
        clearLines();
        currentPiece = nextPiece;
        nextPiece = generatePiece();
        drawNextPiece();
        if (!isValidMove(0, 0)) {
          gameOver = true;
          displayGameOver();
        }
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
      }
    }

    function moveDrop() {
      while (isValidMove(0, 1)) {
        moveDown();
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
        level = Math.floor(score / 1000) + 1;
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

    function resetGame() {
      score = 0;
      level = 1;
      gameOver = false;
      gameSpeed = 500;
      clearBoard();
      currentPiece = generatePiece();
      nextPiece = generatePiece();
      updateScoreAndLevel();
      displayGameOver();
    }

    function clearBoard() {
      board = Array.from({ length: rows }, () => Array(columns).fill(0));
    }

    function checkGameOver() {
      for (let j = 0; j < columns; j++) {
        if (board[0][j] !== 0) {
          return true;
        }
      }
      return false;
    }

    const gameInterval = setInterval(gameLoop, gameSpeed);

    // Pause/Resume Button Click Handler
    const pauseResumeButton = document.getElementById('pause-resume-button');
    pauseResumeButton.addEventListener('click', togglePauseResume);

    function togglePauseResume() {
      isPaused = !isPaused;
      if (isPaused) {
        pauseGame();
      } else {
        resumeGame();
      }
    }

    function pauseGame() {
      clearInterval(gameInterval);
      pauseResumeButton.textContent = 'Продолжить';
    }

    function resumeGame() {
      gameInterval = setInterval(gameLoop, gameSpeed);
      pauseResumeButton.textContent = 'Пауза';
    }
  </script>

  <p>&copy; 2024 Разработчик Dylan933 Все права защищены. г Вяземский | <span id="companyLink"></span></p>
