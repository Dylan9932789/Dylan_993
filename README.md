<!DOCTYPE html>
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
  <div id="score">Score: 0</div>
  <div id="level">Level: 1</div>
  <div id="game-over">Game Over!</div>
  <canvas id="next-piece-canvas" width="100" height="100"></canvas>

  <script>
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
      // Calculate the distance moved
      const touchX = event.touches[0].clientX;
      const touchY = event.touches[0].clientY;
      const deltaX = touchX - touchStartX;
      const deltaY = touchY - touchStartY;

      if (Math.abs(deltaX) > Math.abs(deltaY)) {
        // Horizontal movement
        if (deltaX > 10) { // Adjust threshold as needed for smoother controls
          moveRight();
          touchStartX = touchX;
        } else if (deltaX < -10) {
          moveLeft();
          touchStartX = touchX;
        }
      } else {
        // Vertical movement
        if (deltaY > 10) { // Adjust threshold as needed for smoother controls
          moveDown();
          touchStartY = touchY;
        } else if (deltaY < -10) {
          rotate();
          touchStartY = touchY;
        }
      }
    }

    function handleTouchEnd(event) {
      // Reset touch coordinates
      touchStartX = 0;
      touchStartY = 0;
    }

    document.addEventListener('keydown', (event) => {
      if (!gameOver && !isPaused) {
        switch (event.key) {
          case 'ArrowLeft':
          case 'a':
            moveLeft();
            break;
          case 'ArrowRight':
          case 'd':
            moveRight();
            break;
          case 'ArrowDown':
          case 's':
            moveDown();
            break;
          case 'ArrowUp':
          case 'w':
            rotate();
            break;
          case ' ':
            moveDrop();
            break;
          case 'x':
            // "X" key for toggling pause/resume
            isPaused = !isPaused;
            break;
          case 'c':
            // "C" key for changing the position of the piece
            moveUp();
            break;
          case 'z':
            // "Z" key for clockwise rotation
            rotateClockwise();
            break;
          default:
            break;
        }
      }
    });

    // Add game over detection
    function isGameOver() {
      return !isValidMove(0, 0);
    }

    // Add preview of the next piece
    function drawNextPiece() {
      nextPieceCtx.clearRect(0, 0, nextPieceCanvas.width, nextPieceCanvas.height);
      const offsetX = (nextPieceCanvas.width - blockSize * nextPiece.shape[0].length) / 2;
      const offsetY = (nextPieceCanvas.height - blockSize * nextPiece.shape.length) / 2;

      drawPiece(nextPiece, nextPieceCtx);
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

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBoard();
      drawPiece(currentPiece, ctx);
      document.getElementById('score').textContent = `Score: ${score}`;
      document.getElementById('level').textContent = `Level: ${level}`;

      if (gameOver) {
        document.getElementById('game-over').style.display = 'block';
      }
    }

    function generatePiece() {
      const pieces = [
        { shape: [[1, 1, 1, 1]], color: 'cyan' },
        { shape: [[1, 1, 1], [1]], color: 'blue' },
        { shape: [[1, 1, 1], [0, 0, 1]], color: 'orange' },
        { shape: [[1, 1, 1], [1 fill(0, 0, 3)], color: 'yellow' }, // Добавлено fill(0, 0, 3) для заполнения массива до нужного размера
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
        y: 0
      };
    }

    function moveDown() {
      if (!gameOver && isValidMove(0, 1)) {
        currentPiece.y++;
      } else if (!gameOver) {
        mergePiece();
        clearLines();
        currentPiece = nextPiece;
        nextPiece = generatePiece();
        if (isGameOver()) {
          gameOver = true;
          playGameOverSound(); // В случае окончания игры воспроизводим звуковой эффект
        }
      }
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

    function moveUp() {
      if (!gameOver && isValidMove(0, -1)) {
        currentPiece.y--;
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

    function rotateClockwise() {
      const rotatedPiece = {
        shape: currentPiece.shape[0].map((_, i) => currentPiece.shape.map(row => row[i])).reverse(),
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
          animateLineClear(row); // Анимация удаления линии
          board.splice(row, 1);
          board.unshift(Array(columns).fill(0));
          linesCleared++;
        }
      }
      if (linesCleared > 0) {
        score += linesCleared * 100 * level;
        level = Math.floor(score / 1000) + 1;
        gameSpeed = Math.max(100, gameSpeed - linesCleared * 10);
        playLineClearSound(); // Воспроизводим звуковой эффект
        updateHighScore(); // Обновляем высший счет
      }
    }

    // Функция анимации удаления линии
    function animateLineClear(row) {
      for (let col = 0; col < columns; col++) {
        setTimeout(() => {
          board[row][col] = 0;
          drawBoard();
          drawPiece(currentPiece, ctx);
        }, col * 50); // Скорость анимации, можно изменить по желанию
      }
    }

    function update() {
      const currentTime = Date.now();
      if (!isPaused && currentTime - lastMoveDown > gameSpeed) {
        moveDown();
        lastMoveDown = currentTime;
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBoard();
      drawPiece(currentPiece, ctx);
      document.getElementById('score').textContent = `Score: ${score}`;
      document.getElementById('level').textContent = `Level: ${level}`;

      if (gameOver) {
        gameOverAnimation();
      } else {
        restartButton.style.display = 'none';
      }
    }

    function gameOverAnimation() {
      const gameOverText = "Game Over!";
      ctx.fillStyle = "rgba(255, 255, 255, 0.5)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.font = "30px Arial";
      ctx.fillStyle = "red";
      ctx.textAlign = "center";

      let currentIndex = 0;
      function animate() {
        if (currentIndex < gameOverText.length) {
          ctx.fillText(gameOverText[currentIndex], canvas.width / 2, canvas.height / 2);
          currentIndex++;
          requestAnimationFrame(animate);
        } else {
          restartButton.style.display = 'block';
        }
      }

      animate();
    }

    // Вызов функции resizeCanvas для инициализации размера холста
    resizeCanvas();

    // Вызов gameLoop для начала игры
    gameLoop();
  </script>

  <p>&copy; 2024 Разработчик Dylan933 Все права защищены. | <span id="companyLink"></span></p>
</body>
</html>

