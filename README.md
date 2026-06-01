<!DOCTYPE html>
<html>
<head>
<title>Bazi Taha</title>
<style>
    body { 
        display: flex; 
        align-items: center; /* Center horizontally */
        min-height: 100vh; 
        background-color: #282c34; 
        color: #61dafb; 
        font-family: Arial, sans-serif; 
        margin: 0; 
        position: relative; 
        flex-direction: column; /* Arranges items vertically: h1, then game-container */
        text-align: center; 
        justify-content: flex-start; /* Start content from the top */
        padding-top: 20px; /* Add padding at the top for the title */
    }
    h1 {
        margin-top: 0; /* Remove default top margin for h1 */
        padding-top: 20px; /* Add some padding at the top for the title */
    }
    .game-container { 
        display: flex; /* Arrange children (board and score) horizontally */
        align-items: center; /* Center items vertically within the container */
        gap: 30px; 
        margin-top: 30px; /* Spacing below the title */
        width: auto; /* Allow container to size based on content */
        justify-content: center; /* Center the flex items (board and score) horizontally */
        flex-wrap: wrap; /* Allow items to wrap on smaller screens */
    }
    .game-board { 
        display: grid; 
        grid-template-columns: repeat(8, 30px); 
        grid-template-rows: repeat(8, 30px);    
        gap: 2px; 
        border: 2px solid #61dafb; 
        padding: 4px; 
        background-color: #20232a; 
        border-radius: 8px; 
        order: 1; /* Primary element */
    }
    .square { width: 30px; height: 30px; 
        background-color: #7f838d; 
        border-radius: 5px; 
        display: flex; 
        justify-content: center; 
        align-items: center; 
        font-size: 16px; 
        cursor: pointer; 
        transition: background-color 0.2s, transform 0.2s, border 0.1s; 
        border: 2px solid transparent; 
    }
    .square.selected { border: 2px solid #ffffff; transform: scale(1.05); box-shadow: 0 0 10px #61dafb; }
    .square.matched { background-color: #28a745; animation: match-effect 0.5s forwards; }
    .square.swapping { transform: scale(1.1); }
    .score-display { 
        display: flex; 
        flex-direction: column; 
        align-items: center; 
        background-color: #20232a; 
        padding: 20px; 
        border-radius: 8px; 
        border: 1px solid #61dafb; 
        min-width: 150px; 
        height: fit-content;
        order: 2; /* Ensure score display appears after the board if wrapped */
    }
    .score-display p, .moves-display p { margin: 5px 0; font-size: 18px; }
    .message { margin-top: 15px; font-size: 20px; font-weight: bold; color: #ffc107; text-align: center; width: 100%; }
    @keyframes match-effect { 0% { transform: scale(1.1); opacity: 1; } 100% { transform: scale(0.5); opacity: 0; } }

    /* Overlay Styles */
    .overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.8);
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        z-index: 1000;
        color: white;
        font-size: 24px;
    }
    .overlay-content {
        background-color: #20232a;
        padding: 40px;
        border-radius: 10px;
        text-align: center;
        border: 2px solid #61dafb;
    }
    .overlay-content h2 {
        margin-bottom: 20px;
        color: #61dafb;
    }
    .overlay-content button {
        padding: 10px 20px;
        font-size: 18px;
        margin: 10px;
        cursor: pointer;
        border-radius: 5px;
        border: none;
        transition: background-color 0.3s;
    }
    .start-button { background-color: #28a745; color: white; }
    .start-button:hover { background-color: #218838; }
    .restart-button { background-color: #ffc107; color: black; }
    .restart-button:hover { background-color: #e0a800; }
    .final-score { font-weight: bold; color: #ffc107; margin-top: 15px; }
    
    /* Footer Styles */
    .footer {
        position: absolute;
        bottom: 10px; 
        width: 100%;
        text-align: center;
        font-size: 14px;
        color: #aaa;
    }
</style>
</head>
<body>

<div id="start-screen" class="overlay">
    <div class="overlay-content">
        <h2>King_Taha</h2>
        <p>آماده‌اید؟</p>
        <button id="start-button" class="start-button">شروع بازی</button>
    </div>
</div>

<div id="game-area" style="display: none; width: 100%;"> <!-- Added width 100% to game-area -->
    <h1>Bazi Taha</h1>

    <div class="game-container">
        <div class="game-board" id="game-board"></div>
        <div class="score-display">
            <p>امتیاز: <span id="score">0</span></p>
            <p>حرکت باقی‌مانده: <span id="moves">0</span></p>
            <p id="high-score">بهترین رکورد: <span id="high-score-value">0</span></p>
        </div>
    </div>
    <div id="game-message" class="message"></div>
</div>

<div id="end-screen" class="overlay" style="display: none;">
    <div class="overlay-content">
        <h2>بازی تمام شد!</h2>
        <p class="final-score">امتیاز نهایی شما: <span id="final-score-value">0</span></p>
        <p id="game-over-message"></p>
        <button id="restart-button" class="restart-button">بازی مجدد</button>
    </div>
</div>

<div class="footer">
    ساخته شده توسط طاها
</div>

<script>
    const BOARD_SIZE = 8;
    const NUM_COLORS = 5;
    const MAX_MOVES_INITIAL = 20; 
    let board = [];
    let selectedSquare = null;
    let isProcessing = false;
    let score = 0;
    let moves = MAX_MOVES_INITIAL;
    let highScore = 0;
    let squares_on_board = [];
    let maxMoves = MAX_MOVES_INITIAL; 

    const colors = ['#ff6347', '#4682b4', '#32cd32', '#ffd700', '#9370db'];

    const startScreen = document.getElementById('start-screen');
    const gameArea = document.getElementById('game-area');
    const endScreen = document.getElementById('end-screen');
    const startButton = document.getElementById('start-button');
    const restartButton = document.getElementById('restart-button');
    const gameBoardElement = document.getElementById('game-board');
    const scoreElement = document.getElementById('score');
    const movesElement = document.getElementById('moves');
    const highScoreValueElement = document.getElementById('high-score-value');
    const finalScoreValueElement = document.getElementById('final-score-value');
    const gameMessageElement = document.getElementById('game-message');
    const gameOverMessageElement = document.getElementById('game-over-message');


    function loadHighScore() {
        const savedScore = localStorage.getItem('match3HighScore');
        if (savedScore) {
            highScore = parseInt(savedScore);
        }
        highScoreValueElement.textContent = highScore;
    }

    function saveHighScore() {
        if (score > highScore) {
            highScore = score;
            localStorage.setItem('match3HighScore', highScore);
            highScoreValueElement.textContent = highScore;
        }
    }

    function getRandomColor() {
        return colors[Math.floor(Math.random() * NUM_COLORS)];
    }

    function setupBoard() {
        board = Array(BOARD_SIZE).fill(0).map(() => Array(BOARD_SIZE).fill(null));
        for (let r = 0; r < BOARD_SIZE; r++) {
            for (let c = 0; c < BOARD_SIZE; c++) {
                let color;
                do {
                    color = getRandomColor();
                } while (
                    (c >= 2 && board[r][c-1] && board[r][c-1].color === color && board[r][c-2] && board[r][c-2].color === color) ||
                    (r >= 2 && board[r-1][c] && board[r-1][c].color === color && board[r-2][c] && board[r-2][c].color === color)
                );
                board[r][c] = { color: color, row: r, col: c };
            }
        }
    }

    function renderBoard() {
        gameBoardElement.innerHTML = '';
        squares_on_board = [];

        for (let r = 0; r < BOARD_SIZE; r++) {
            for (let c = 0; c < BOARD_SIZE; c++) {
                const square = document.createElement('div');
                square.className = 'square';
                square.style.backgroundColor = board[r][c].color;
                square.dataset.row = r;
                square.dataset.col = c;
                square.addEventListener('click', handleCellClick);
                gameBoardElement.appendChild(square);
                squares_on_board.push(square);
            }
        }
    }

    function getSquareElement(row, col) {
        return document.querySelector(`.square[data-row='${row}'][data-col='${col}']`);
    }

    function isAdjacent(r1, c1, r2, c2) {
        return (Math.abs(r1 - r2) === 1 && c1 === c2) || (Math.abs(c1 - c2) === 1 && r1 === r2);
    }

    function updateDisplay() {
        scoreElement.textContent = score;
        movesElement.textContent = moves;
    }

    function checkGameOver() {
        if (moves === 0 && !isProcessing) {
            endGame();
            return true;
        }
        return false;
    }

    function endGame() {
        gameArea.style.display = 'none';
        endScreen.style.display = 'flex';
        finalScoreValueElement.textContent = score;
        gameOverMessageElement.textContent = `بهترین رکورد شما: ${highScore}`;
        saveHighScore(); 
    }

    function handleCellClick(event) {
        if (isProcessing || moves === 0) return;

        const clickedRow = parseInt(event.target.dataset.row);
        const clickedCol = parseInt(event.target.dataset.col);

        if (selectedSquare) {
            const selRow = selectedSquare.row;
            const selCol = selectedSquare.col;

            if (clickedRow === selRow && clickedCol === selCol) {
                selectedSquare.element.classList.remove('selected');
                selectedSquare = null;
                return;
            }

            if (isAdjacent(selRow, selCol, clickedRow, clickedCol)) {
                moves--;
                updateDisplay();

                isProcessing = true;

                const square1Data = board[selRow][selCol];
                const square2Data = board[clickedRow][clickedCol];

                const el1 = getSquareElement(selRow, selCol);
                const el2 = getSquareElement(clickedRow, clickedCol);

                el1.classList.add('swapping');
                el2.classList.add('swapping');

                board[selRow][selCol] = square2Data;
                board[clickedRow][clickedCol] = square1Data;
                board[selRow][selCol].row = selRow;
                board[selRow][selCol].col = selCol;
                board[clickedRow][clickedCol].row = clickedRow;
                board[clickedRow][clickedCol].col = clickedCol;

                el1.dataset.row = selRow; el1.dataset.col = selCol;
                el2.dataset.row = clickedRow; el2.dataset.col = clickedCol;
                el1.style.backgroundColor = board[selRow][selCol].color;
                el2.style.backgroundColor = board[clickedRow][clickedCol].color;

                const matchesFoundCount = removeMatchesAndRefill();

                if (matchesFoundCount === 0) {
                    board[selRow][selCol] = square1Data; 
                    board[clickedRow][clickedCol] = square2Data;
                    board[selRow][selCol].row = selRow; board[selRow][selCol].col = selCol;
                    board[clickedRow][clickedCol].row = clickedRow; board[clickedRow][clickedCol].col = clickedCol;

                    el1.dataset.row = selRow; el1.dataset.col = selCol; 
                    el2.dataset.row = clickedRow; el2.dataset.col = clickedCol;
                    el1.style.backgroundColor = board[selRow][selCol].color; 
                    el2.style.backgroundColor = board[clickedRow][clickedCol].color;

                    gameMessageElement.textContent = "حرکت نامعتبر بود. تلاشی برای تطابق پیدا نشد.";
                    
                } else {
                    score += matchesFoundCount; 
                    gameMessageElement.textContent = `تطابق یافت شد! امتیاز: ${score}`;
                }

                el1.classList.remove('swapping');
                el2.classList.remove('swapping');
                if (selectedSquare) {
                    selectedSquare.element.classList.remove('selected');
                }
                selectedSquare = null;
                isProcessing = false;

                updateDisplay(); 

                if (matchesFoundCount === 0) { 
                    checkGameOver();
                } else { 
                    setTimeout(checkGameOver, 500); 
                }

            } else {
                selectedSquare.element.classList.remove('selected');
                selectedSquare = { element: event.target, color: board[clickedRow][clickedCol].color, row: clickedRow, col: clickedCol };
                event.target.classList.add('selected');
                gameMessageElement.textContent = "";
            }
        } else {
            selectedSquare = { element: event.target, color: board[clickedRow][clickedCol].color, row: clickedRow, col: clickedCol };
            event.target.classList.add('selected');
            gameMessageElement.textContent = "";
        }
    }

    function getMatches() {
        const matchSet = new Set();

        for (let r = 0; r < BOARD_SIZE; r++) {
            for (let c = 0; c < BOARD_SIZE - 2; c++) {
                if (board[r][c] && board[r][c+1] && board[r][c+2] &&
                    board[r][c].color === board[r][c+1].color &&
                    board[r][c].color === board[r][c+2].color) {
                    matchSet.add(JSON.stringify({r, c}));
                    matchSet.add(JSON.stringify({r, c: c + 1}));
                    matchSet.add(JSON.stringify({r, c: c + 2}));
                }
            }
        }

        for (let c = 0; c < BOARD_SIZE; c++) {
            for (let r = 0; r < BOARD_SIZE - 2; r++) {
                if (board[r][c] && board[r+1][c] && board[r+2][c] &&
                    board[r][c].color === board[r+1][c].color &&
                    board[r][c].color === board[r+2][c].color) {
                    matchSet.add(JSON.stringify({r, c}));
                    matchSet.add(JSON.stringify({r: r + 1, c}));
                    matchSet.add(JSON.stringify({r: r + 2, c}));
                }
            }
        }
        return Array.from(matchSet).map(str => JSON.parse(str));
    }

    function refillBoard() {
        let newSquaresAdded = false;
        for (let c = 0; c < BOARD_SIZE; c++) {
            for (let r = BOARD_SIZE - 1; r >= 0; r--) {
                if (board[r][c] === null) { 
                    let foundAbove = false;
                    for (let rAbove = r - 1; rAbove >= 0; rAbove--) {
                        if (board[rAbove][c] !== null) {
                            board[r][c] = { ...board[rAbove][c], row: r, col: c };
                            board[rAbove][c] = null; 
                            foundAbove = true;
                            newSquaresAdded = true;
                            break; 
                        }
                    }
                    if (!foundAbove) {
                        board[r][c] = { color: getRandomColor(), row: r, col: c };
                        newSquaresAdded = true;
                    }
                }
            }
        }
        return newSquaresAdded;
    }

    function removeMatchesAndRefill() {
        let totalMatchesCount = 0;
        let animationDelay = 0;
        
        while (true) {
            const matches = getMatches();
            if (matches.length === 0) break; 

            totalMatchesCount += matches.length;
            
            matches.forEach(({r, c}, index) => {
                const element = getSquareElement(r, c);
                if (element) {
                    element.classList.add('matched');
                    board[r][c] = null; 
                    animationDelay = Math.max(animationDelay, index * 50); 
                }
            });

            setTimeout(() => {
                matches.forEach(({r, c}) => {
                    const elementToRemove = gameBoardElement.querySelector(`.square[data-row='${r}'][data-col='${c}']`);
                    if (elementToRemove && elementToRemove.classList.contains('matched')) {
                         elementToRemove.remove();
                    }
                });

                if (refillBoard()) {
                    renderBoard(); 
                }
            }, animationDelay + 100); 
        }
        
        if (totalMatchesCount > 0) {
             if (refillBoard()) {
                setTimeout(renderBoard, animationDelay + 200); 
            }
        }

        return totalMatchesCount;
    }


    function startGame() {
        startScreen.style.display = 'none';
        gameArea.style.display = 'flex';
        endScreen.style.display = 'none';

        score = 0;
        moves = maxMoves; 
        updateDisplay();
        gameMessageElement.textContent = "";
        
        setupBoard();
        renderBoard();
        loadHighScore(); 
        isProcessing = false; 
    }

    function restartGame() {
        endScreen.style.display = 'none';
        gameArea.style.display = 'flex';
        score = 0;
        moves = maxMoves; 
        updateDisplay();
        gameMessageElement.textContent = "";
        setupBoard();
        renderBoard();
        isProcessing = false;
    }

    startButton.addEventListener('click', startGame);
    restartButton.addEventListener('click', restartGame);

    loadHighScore(); 

</script>

</body>
</html>
