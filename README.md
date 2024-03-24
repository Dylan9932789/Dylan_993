
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

      // Determine the direction of the movement
      if (Math.abs(deltaX) > Math.abs(deltaY)) {
        // Horizontal movement
        if (deltaX > 0) {
          moveRight();
        } else {
          moveLeft();
        }
      } else {
        // Vertical movement
        if (deltaY > 0) {
          moveDown();
        } else {
          rotate();
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

    // Добавляем обработчик события для нажатия на фигуру
    const rotateCurrentPiece = () => {
      rotate();
    };

    canvas.addEventListener('click', rotateCurrentPiece);

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
      } else if (!gameOver) {
        mergePiece();
        clearLines();
        currentPiece = nextPiece;
        nextPiece = generatePiece();
        if (!isValidMove(0, 0)) {
          gameOver = true;
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

    function moveUp() {
      if (!gameOver && isValidMove(0, -1)) {
        currentPiece.y--;
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
        level = Math.floor(score / 1000) + 1; // Update level
        // Increase game speed after clearing lines
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

    gameLoop();
    function update() {
  const currentTime = Date.now();
  if (!isPaused && currentTime - lastMoveDown > gameSpeed) {
    moveDown();
    lastMoveDown = currentTime;
  }
}

function pauseGame() {
  isPaused = true;
}

function resumeGame() {
  isPaused = false;
}

document.addEventListener('keydown', (event) => {
  if (event.key === 'Escape') { // Press Escape key to toggle pause/resume
    isPaused ? resumeGame() : pauseGame();
  }
});

function gameLoop() {
  update();
  draw();
  drawNextPiece(); // Add this to update the next piece display
  requestAnimationFrame(gameLoop);
}

gameLoop();
// Update handleTouchMove function to handle continuous touch movement
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

// Add game over detection
function isGameOver() {
  // Check if the current piece can be placed at the top of the board
  return !isValidMove(0, 0);
}

// Update moveDown function to check for game over
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
    }
  }
}

// Update draw function to display game over message
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

// Update gameLoop function to stop when game over
function gameLoop() {
  if (!gameOver) {
    update();
    draw();
    drawNextPiece();
    requestAnimationFrame(gameLoop);
  }
}

// Call gameLoop to start the game
gameLoop();
// Add scoring and leveling up
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
    score += linesCleared * 100 * level; // Increase score based on level
    level = Math.floor(score / 1000) + 1; // Update level
    // Increase game speed after clearing lines
    gameSpeed = Math.max(100, gameSpeed - linesCleared * 10);
  }
}

// Update draw function to display game over message with final score
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBoard();
  drawPiece(currentPiece, ctx);
  document.getElementById('score').textContent = `Score: ${score}`;
  document.getElementById('level').textContent = `Level: ${level}`;

  if (gameOver) {
    ctx.fillStyle = "rgba(255, 255, 255, 0.5)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.font = "30px Arial";
    ctx.fillStyle = "red";
    ctx.textAlign = "center";
    ctx.fillText("Game Over!", canvas.width / 2, canvas.height / 2 - 30);
    ctx.fillText(`Final Score: ${score}`, canvas.width / 2, canvas.height / 2 + 10);
  }
}

// Update gameLoop function to stop when game over
function gameLoop() {
  if (!gameOver) {
    update();
    draw();
    drawNextPiece();
    requestAnimationFrame(gameLoop);
  }
}

// Call gameLoop to start the game
gameLoop();

  </script>

 <p>&copy; 2024 Разработчик  Dylan933 Все права защищены. | <span id="companyLink"></span></p>
