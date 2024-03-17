
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tetris</title>
<style>
    body {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        margin: 0;
        background-color: #f0f0f0;
        font-family: Arial, sans-serif;
    }

    .grid {
        display: grid;
        grid-template-columns: repeat(10, 30px);
        grid-template-rows: repeat(20, 30px);
        border: 1px solid #999;
        position: relative;
    }

    .cell {
        width: 30px;
        height: 30px;
        border: 1px solid #ccc;
        background-color: #fff;
    }

    .tetromino {
        background-color: #000;
    }

    .score {
        text-align: center;
        font-size: 24px;
        margin-bottom: 20px;
    }

    .level {
        text-align: center;
        font-size: 18px;
    }

    .game-over {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 36px;
        color: red;
        display: none;
    }

    .pause {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 36px;
        color: blue;
        display: none;
    }
</style>
</head>
<body>
<div class="score">Score: 0</div>
<div class="level">Level: 1</div>
<div class="game-over">Game Over</div>
<div class="pause">Paused</div>
<div class="grid"></div>

<audio id="move-sound" src="move.mp3"></audio>
<audio id="clear-sound" src="clear.mp3"></audio>

<script>
    const grid = document.querySelector('.grid');
    const scoreDisplay = document.querySelector('.score');
    const levelDisplay = document.querySelector('.level');
    const gameOverDisplay = document.querySelector('.game-over');
    const pauseDisplay = document.querySelector('.pause');
    const moveSound = document.getElementById('move-sound');
    const clearSound = document.getElementById('clear-sound');

    let squares = Array.from(Array(200).keys());
    let timerId;
    let score = 0;
    let level = 1;
    let isPaused = false;

    const tetrominoes = [
        [[1, 11, 21, 31], [1, 2, 3, 4]], // I
        [[1, 2, 11, 12]], // O
        [[1, 2, 11, 21], [2, 11, 12, 13], [1, 11, 12, 21], [1, 10, 11, 12]], // T
        [[1, 2, 12, 22], [0, 10, 11, 12], [1, 11, 21, 22], [2, 10, 11, 12]], // L
        [[0, 1, 11, 12], [1, 11, 12, 22], [10, 11, 21, 22], [1, 10, 11, 21]], // J
        [[0, 10, 11, 21], [1, 11, 12, 22]], // S
        [[1, 11, 10, 20], [0, 10, 11, 21]] // Z
    ];

    let currentPosition = 4;
    let currentRotation = 0;
    let currentTetromino = tetrominoes[0][0];

    function draw() {
        for (let i = 0; i < squares.length; i++) {
            const square = document.createElement('div');
            square.classList.add('cell');
            grid.appendChild(square);
        }
    }

    function drawTetromino() {
        currentTetromino.forEach(index => {
            squares[currentPosition + index].classList.add('tetromino');
        });
    }

    function undrawTetromino() {
        currentTetromino.forEach(index => {
            squares[currentPosition + index].classList.remove('tetromino');
        });
    }

    draw();
    drawTetromino();

    function moveDown() {
        undrawTetromino();
        currentPosition += 10;
        drawTetromino();
        freeze();
    }

    function moveLeft() {
        undrawTetromino();
        const isAtLeftEdge = currentTetromino.some(index => (currentPosition + index) % 10 === 0);
        if (!isAtLeftEdge) currentPosition -= 1;
        if (currentTetromino.some(index => squares[currentPosition + index].classList.contains('taken'))) {
            currentPosition += 1;
        }
        drawTetromino();
    }

    function moveRight() {
        undrawTetromino();
        const isAtRightEdge = currentTetromino.some(index => (currentPosition + index) % 10 === 9);
        if (!isAtRightEdge) currentPosition += 1;
        if (currentTetromino.some(index => squares[currentPosition + index].classList.contains('taken'))) {
            currentPosition -= 1;
        }
        drawTetromino();
    }

    function rotate() {
        undrawTetromino();
        currentRotation++;
        if (currentRotation === currentTetromino.length) {
            currentRotation = 0;
        }
        currentTetromino = tetrominoes[0][currentRotation];
        drawTetromino();
    }

    function control(e) {
        if (e.keyCode === 37) {
            moveLeft();
        } else if (e.keyCode === 38) {
            rotate();
        } else if (e.keyCode === 39) {
            moveRight();
        } else if (e.keyCode === 40) {
            moveDown();
        } else if (e.keyCode === 80) { // "P" key for pause/resume
            togglePause();
        }
    }

    document.addEventListener('keydown', control);

    function togglePause() {
        if (!isPaused) {
            clearInterval(timerId);
            isPaused = true;
            pauseDisplay.style.display = 'block';
        } else {
            timerId = setInterval(moveDown, 1000 / level);
            isPaused = false;
            pauseDisplay.style.display = 'none';
        }
    }

    function playSound(sound) {
        sound.currentTime = 0;
        sound.play();
    }

    function freeze() {
        if (currentTetromino.some(index => squares[currentPosition + index + 10].classList.contains('taken'))) {
            currentTetromino.forEach(index => squares[currentPosition + index].classList.add('taken'));
            playSound(moveSound);
            // Increase score
            score += 10;
            scoreDisplay.textContent = `Score: ${score}`;
            // Check level
            if (score % 100 === 0) {
                level++;
                levelDisplay.textContent = `Level: ${level}`;
                clearInterval(timerId);
                timerId = setInterval(moveDown, 1000 / level);
            }
            // Check game over
            if (currentTetromino.some(index => squares[currentPosition + index].classList.contains('taken') && currentPosition < 10)) {
                gameOver();
            }
            // Start a new tetromino falling
            currentPosition = 4;
            currentTetromino = tetrominoes[Math.floor(Math.random() * tetrominoes.length)][0];
            drawTetromino();
            // Check for line clear
            checkForLineClear();
        }
    }

    // Check for row completion
    function checkForLineClear() {
        for (let i = 0; i < 199; i += 10) {
            const row = [i, i + 1, i + 2, i + 3, i + 4, i + 5, i + 6, i + 7, i + 8, i + 9];
            if (row.every(index => squares[index].classList.contains('taken'))) {
                playSound(clearSound);
                row.forEach(index => {
                    squares[index].classList.remove('taken');
                    squares[index].classList.remove('tetromino');
                });
                const squaresRemoved = squares.splice(i, 10);
                squares = squaresRemoved.concat(squares);
                squares.forEach(cell => grid.appendChild(cell));
            }
        }
    }

    // Game over
    function gameOver() {
        clearInterval(timerId);
        document.removeEventListener('keydown', control);
        gameOverDisplay.style.display = 'block';
    }

    timerId = setInterval(moveDown, 1000 / level);
</script>


