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

    button {
      margin-top: 20px;
      font-size: 16px;
      padding: 8px 16px;
    }
  </style>
</head>
<body>
  <canvas id="tetrisCanvas" width="300" height="600"></canvas>
  <div id="score">Score: 0</div>
  <div id="level">Level: 1</div>
  <div id="game-over">Game Over!</div>
  <canvas id="next-piece-canvas" width="100" height="100"></canvas>
  <button id="restart-button">Restart Game</button>

  <script>
    const canvas = document.getElementById('tetrisCanvas');
    const ctx = canvas.getContext('2d');
    const nextPieceCanvas = document.getElementById('next-piece-canvas');
    const nextPieceCtx = nextPieceCanvas.getContext('2d');
    const scoreElement = document.getElementById('score');
    const levelElement = document.getElementById('level');
    const restartButton = document.getElementById('restart-button');

    const blockSize = 30;
    const rows = 20;
    const columns = 10;
    let board = Array.from({ length: rows }, () => Array(columns).fill(0));
    let currentPiece = generatePiece();
    let nextPiece = generatePiece();
    let holdPiece = null;
    let canHold = true;
    let score = 0;
    let level = 1;
    let gameOver = false;
    let gameSpeed = 500; // Initial game speed in milliseconds
    let lastMoveDown = Date.now();
    let isPaused = false;
    let touchStartX = 0;
    let touchStartY = 0;
    let touchMoveTimer = null;

    // Define block colors
    const blockColors = {
      'cyan': '#00FFFF',
      'blue': '#0000FF',
      'orange': '#FFA500',
      'yellow': '#FFFF00',
      'red': '#FF0000',
      'green': '#00FF00',
      'purple': '#800080'
    };

    // Sounds
    const lineClearSound = new Audio('line_clear_sound.mp3'); // Replace with actual sound file
    const gameOverSound = new Audio('game_over_sound.mp3'); // Replace with actual sound file

    // Keyboard Controls
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
            // "C" key for holding piece
            holdCurrentPiece();
            break;
          case 'z':
            // "Z" key for clockwise rotation
            rotateClockwise();
            break;
          case 'Escape':
            // Escape key to toggle pause/resume
            isPaused ? resumeGame() : pauseGame();
            break;
          default:
            break;
        }
      }
    });

    // Touch Controls
    canvas.addEventListener('touchstart', handleTouchStart, false);
    canvas.addEventListener('touchmove', handleTouchMove, false);
    canvas.addEventListener('touchend', handleTouchEnd, false);

    function handleTouchStart(event) {
      event.preventDefault();
      const touch = event.touches[0];
      touchStartX = touch.clientX;
      touchStartY = touch.clientY;
      touchMoveTimer = setTimeout(() => {
        handleLongPress();
      }, 500); // Adjust as needed for long press duration
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
        if (deltaY > 10)
          moveDown();
          touchStartY = touchY;
        } else if (deltaY < -10) {
          rotate();
          touchStartY = touchY;
        }
      }
    }

    function handleTouchEnd(event) {
      event.preventDefault();
      clearTimeout(touchMoveTimer);
    }

    function handleLongPress() {
      // Handle long press event, for example, pause/resume the game
      isPaused ? resumeGame() : pauseGame();
    }

    // Resize canvas on window resize
    window.addEventListener('resize', resizeCanvas);

    function resizeCanvas() {
      const maxWidth = window.innerWidth - 20; // Adjust margin
      const maxHeight = window.innerHeight - 20; // Adjust margin
      const idealWidth = columns * blockSize;
      const idealHeight = rows * blockSize;
      let scale = 1;
      if (idealWidth > maxWidth || idealHeight > maxHeight) {
        scale = Math.min(maxWidth / idealWidth, maxHeight / idealHeight);
      }
      canvas.width = idealWidth * scale;
      canvas.height = idealHeight * scale;
      canvas.style.width = `${canvas.width}px`;
      canvas.style.height = `${canvas.height}px`;
    }

    // Draw a square on the canvas
    function drawSquare(x, y, color, context) {
      context.fillStyle = color;
      context.fillRect(x * blockSize, y * blockSize, blockSize, blockSize);
      context.strokeStyle = "#000";
      context.strokeRect(x * blockSize, y * blockSize, blockSize, blockSize);
    }

    // Draw the game board
    function drawBoard() {
      for (let row = 0; row < rows; row++) {
        for (let col = 0; col < columns; col++) {
          if (board[row][col] !== 0) {
            drawSquare(col, row, blockColors[board[row][col]], ctx);
          }
        }
      }
    }

    // Draw a Tetris piece on the canvas
    function drawPiece(piece, context, isGhost = false) {
      piece.shape.forEach((row, i) => {
        row.forEach((cell, j) => {
          if (cell !== 0) {
            const x = piece.x + j;
            const y = piece.y + i;
            const color = isGhost ? 'rgba(255, 255, 255, 0.5)' : blockColors[piece.color];
            drawSquare(x, y, color, context);
          }
        });
      });
    }

    // Draw the next piece preview
    function drawNextPiece() {
      nextPieceCtx.clearRect(0, 0, nextPieceCanvas.width, nextPieceCanvas.height);
      const offsetX = (nextPieceCanvas.width - blockSize * nextPiece.shape[0].length) / 2;
      const offsetY = (nextPieceCanvas.height - blockSize * nextPiece.shape.length) / 2;

      drawPiece(nextPiece, nextPieceCtx);
    }

    // Draw the ghost piece
    function drawGhostPiece() {
      let ghostPiece = {
        ...currentPiece,
        y: currentPiece.y
      };
      while (isValidMove(0, 1, ghostPiece)) {
        ghostPiece.y++;
      }
      ghostPiece.y--;
      drawPiece(ghostPiece, ctx, true); // Draw semi-transparent ghost piece
    }

    // Update the score display
    function updateScore() {
      scoreElement.textContent = `Score: ${score}`;
    }

    // Update the level display
    function updateLevel() {
      levelElement.textContent = `Level: ${level}`;
    }

    // Generate a random Tetris piece
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

    // Check if a move is valid
    function isValidMove(offsetX, offsetY, piece = currentPiece) {
      for (let i = 0; i < piece.shape.length; i++) {
        for (let j = 0; j < piece.shape[i].length; j++) {
          if (
            piece.shape[i][j] !== 0 &&
            (board[piece.y + i + offsetY] && board[piece.y + i + offsetY][piece.x +
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
      event.preventDefault();
      clearTimeout(touchMoveTimer);
    }

    function handleLongPress() {
      // Handle long press event, for example, pause/resume the game
      isPaused ? resumeGame() : pauseGame();
    }

    // Pause and Resume Game
    function pauseGame() {
      isPaused = true;
    }

    function resumeGame() {
      isPaused = false;
      lastMoveDown = Date.now();
    }

    // Resize Canvas
    window.addEventListener('resize', resizeCanvas);

    function resizeCanvas() {
      const maxWidth = window.innerWidth - 20; // Adjust margin
      const maxHeight = window.innerHeight - 20; // Adjust margin
      const idealWidth = columns * blockSize;
      const idealHeight = rows * blockSize;
      let scale = 1;
      if (idealWidth > maxWidth || idealHeight > maxHeight) {
        scale = Math.min(maxWidth / idealWidth, maxHeight / idealHeight);
      }
      canvas.width = idealWidth * scale;
      canvas.height = idealHeight * scale;
      canvas.style.width = `${canvas.width}px`;
      canvas.style.height = `${canvas.height}px`;
    }

    // Draw Functions
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
            drawSquare(col, row, blockColors[board[row][col]], ctx);
          }
        }
      }
    }

    function drawPiece(piece, context, isGhost = false) {
      piece.shape.forEach((row, i) => {
        row.forEach((cell, j) => {
          if (cell !== 0) {
            const x = piece.x + j;
            const y = piece.y + i;
            const color = isGhost ? 'rgba(255, 255, 255, 0.5)' : blockColors[piece.color];
            drawSquare(x, y, color, context);
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

    // Game Functions
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
        if (isGameOver()) {
          gameOver = true;
          playGameOverSound();
        }
        canHold = true; // Allow holding piece after new piece is generated
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

    function moveDrop() {
      while (isValidMove(0, 1)) {
        moveDown();
     
        {
          moveDown();
          touchStartY = touchY;
        } else if (deltaY < -10) {
          rotate();
          touchStartY = touchY;
        }
      }
    }

    function handleLongPress() {
      isPaused ? resumeGame() : pauseGame();
    }

    function handleTouchEnd(event) {
      event.preventDefault();
      clearTimeout(touchMoveTimer);
    }

    // Game Loop
    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    // Start the game loop
    gameLoop();

    // Functions for controlling the game
    function pauseGame() {
      isPaused = true;
    }

    function resumeGame() {
      isPaused = false;
      lastMoveDown = Date.now();
    }

    function restartGame() {
      board = Array.from({ length: rows }, () => Array(columns).fill(0));
      currentPiece = generatePiece();
      nextPiece = generatePiece();
      holdPiece = null;
      canHold = true;
      score = 0;
      level = 1;
      gameOver = false;
      gameSpeed = 500;
      lastMoveDown = Date.now();
      isPaused = false;
      updateScore();
    }

    function gameOver() {
      // Game over logic
    }

    function update() {
      const currentTime = Date.now();
      if (!isPaused && currentTime - lastMoveDown > gameSpeed) {
        moveDown();
        lastMoveDown = currentTime;
      }
    }

    function draw() {
      // Drawing logic
    }

    function generatePiece() {
      // Piece generation logic
    }

    function drawSquare(x, y, color, context) {
      // Drawing a square
    }

    function drawBoard() {
      // Drawing the game board
    }

    function drawPiece(piece, context) {
      // Drawing a piece
    }

    function clearLines() {
      // Line clearing logic
    }

    function moveLeft() {
      // Move piece left
    }

    function moveRight() {
      // Move piece right
    }

    function moveDown() {
      // Move piece down
    }

    function rotate() {
      // Rotate piece
    }

    function rotateClockwise() {
      // Rotate piece clockwise
    }

    function moveDrop() {
      // Drop piece
    }

    function isValidMove(offsetX, offsetY, piece = currentPiece) {
      // Check if move is valid
    }

    function mergePiece() {
      // Merge piece with board
    }

    function holdCurrentPiece() {
      // Hold current piece
    }

    function updateScore() {
      // Update score on the UI
    }
  </script>
<p>&copy; 2024 Разработчик  Dylan933 Все права защищены. | <span id="companyLink"></span></p>

