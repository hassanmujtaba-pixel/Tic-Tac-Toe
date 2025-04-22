<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Tic Tac Toe</title>
    <style>
        :root {
            --dark-bg: #121212;
            --darker-bg: #0a0a0a;
            --cell-bg: #1e1e1e;
            --accent-blue: #00d4ff;
            --accent-pink: #ff00aa;
            --text: rgba(255, 255, 255, 0.87);
        }
        
        body {
            background-color: var(--dark-bg);
            color: var(--text);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        
        h1 {
            margin-bottom: 30px;
            font-size: 2.5rem;
            background: linear-gradient(90deg, var(--accent-blue), var(--accent-pink));
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            text-align: center;
        }
        
        .game-container {
            background: var(--darker-bg);
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            width: 100%;
            max-width: 400px;
        }
        
        .status {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            font-size: 1.2rem;
            padding: 10px 15px;
            border-radius: 8px;
            background: rgba(255, 255, 255, 0.05);
        }
        
        .player-x {
            color: var(--accent-blue);
            font-weight: bold;
        }
        
        .player-o {
            color: var(--accent-pink);
            font-weight: bold;
        }
        
        .board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-gap: 10px;
            margin-bottom: 20px;
        }
        
        .cell {
            aspect-ratio: 1/1;
            background-color: var(--cell-bg);
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 3rem;
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }
        
        .cell:hover {
            transform: translateY(-3px);
            box-shadow: 0 5px 15px rgba(0, 212, 255, 0.2);
        }
        
        .cell.x {
            color: var(--accent-blue);
        }
        
        .cell.o {
            color: var(--accent-pink);
        }
        
        .win-strike {
            position: absolute;
            background: white;
            z-index: 10;
        }
        
        .strike-row-1 { top: 16.66%; height: 5px; width: 100%; }
        .strike-row-2 { top: 50%; height: 5px; width: 100%; }
        .strike-row-3 { top: 83.33%; height: 5px; width: 100%; }
        .strike-col-1 { left: 16.66%; width: 5px; height: 100%; }
        .strike-col-2 { left: 50%; width: 5px; height: 100%; }
        .strike-col-3 { left: 83.33%; width: 5px; height: 100%; }
        .strike-diag-1 { 
            width: 120%; 
            height: 5px; 
            top: 50%;
            left: -10%;
            transform: rotate(45deg);
        }
        .strike-diag-2 { 
            width: 120%; 
            height: 5px; 
            top: 50%;
            left: -10%;
            transform: rotate(-45deg);
        }
        
        .reset-btn {
            background: linear-gradient(90deg, var(--accent-blue), var(--accent-pink));
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s ease;
            width: 100%;
        }
        
        .reset-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(255, 0, 170, 0.3);
        }
        
        .confetti {
            position: fixed;
            width: 10px;
            height: 10px;
            background-color: #f00;
            border-radius: 50%;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <h1>Neon Tic Tac Toe</h1>
    
    <div class="game-container">
        <div class="status">
            <span>Player: </span>
            <span id="current-player" class="player-x">X</span>
        </div>
        
        <div class="board" id="board">
            <div class="cell" data-index="0"></div>
            <div class="cell" data-index="1"></div>
            <div class="cell" data-index="2"></div>
            <div class="cell" data-index="3"></div>
            <div class="cell" data-index="4"></div>
            <div class="cell" data-index="5"></div>
            <div class="cell" data-index="6"></div>
            <div class="cell" data-index="7"></div>
            <div class="cell" data-index="8"></div>
        </div>
        
        <button class="reset-btn" id="reset-btn">New Game</button>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const board = document.getElementById('board');
            const cells = document.querySelectorAll('.cell');
            const currentPlayerDisplay = document.getElementById('current-player');
            const resetButton = document.getElementById('reset-btn');
            
            let currentPlayer = 'X';
            let gameState = ['', '', '', '', '', '', '', '', ''];
            let gameActive = true;
            
            const winningConditions = [
                [0, 1, 2], [3, 4, 5], [6, 7, 8], // rows
                [0, 3, 6], [1, 4, 7], [2, 5, 8], // columns
                [0, 4, 8], [2, 4, 6]             // diagonals
            ];
            
            // Initialize game
            resetButton.addEventListener('click', resetGame);
            cells.forEach(cell => cell.addEventListener('click', handleCellClick));
            
            function handleCellClick(e) {
                const clickedCell = e.target;
                const clickedCellIndex = parseInt(clickedCell.getAttribute('data-index'));
                
                if (gameState[clickedCellIndex] !== '' || !gameActive) return;
                
                // Update game state and UI
                gameState[clickedCellIndex] = currentPlayer;
                clickedCell.classList.add(currentPlayer.toLowerCase());
                clickedCell.textContent = currentPlayer;
                
                // Check for win or draw
                checkResult();
            }
            
            function checkResult() {
                let roundWon = false;
                
                for (let i = 0; i < winningConditions.length; i++) {
                    const [a, b, c] = winningConditions[i];
                    
                    if (gameState[a] === '' || gameState[b] === '' || gameState[c] === '') continue;
                    
                    if (gameState[a] === gameState[b] && gameState[b] === gameState[c]) {
                        roundWon = true;
                        drawWinningLine(winningConditions[i]);
                        break;
                    }
                }
                
                if (roundWon) {
                    gameActive = false;
                    celebrateWin();
                    return;
                }
                
                if (!gameState.includes('')) {
                    // Draw
                    gameActive = false;
                    return;
                }
                
                // Switch player
                currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
                currentPlayerDisplay.textContent = currentPlayer;
                currentPlayerDisplay.className = currentPlayer === 'X' ? 'player-x' : 'player-o';
            }
            
            function drawWinningLine(winningCombo) {
                const [a, b, c] = winningCombo;
                const cell1 = document.querySelector(`[data-index="${a}"]`);
                const cell2 = document.querySelector(`[data-index="${b}"]`);
                const cell3 = document.querySelector(`[data-index="${c}"]`);
                
                // Determine line direction
                let strikeClass = '';
                
                // Check rows
                if (a === 0 && b === 1 && c === 2) strikeClass = 'strike-row-1';
                else if (a === 3 && b === 4 && c === 5) strikeClass = 'strike-row-2';
                else if (a === 6 && b === 7 && c === 8) strikeClass = 'strike-row-3';
                // Check columns
                else if (a === 0 && b === 3 && c === 6) strikeClass = 'strike-col-1';
                else if (a === 1 && b === 4 && c === 7) strikeClass = 'strike-col-2';
                else if (a === 2 && b === 5 && c === 8) strikeClass = 'strike-col-3';
                // Check diagonals
                else if (a === 0 && b === 4 && c === 8) strikeClass = 'strike-diag-1';
                else if (a === 2 && b === 4 && c === 6) strikeClass = 'strike-diag-2';
                
                // Create strike element
                const strike = document.createElement('div');
                strike.className = `win-strike ${strikeClass}`;
                
                // Add to middle cell
                cell2.appendChild(strike);
            }
            
            function celebrateWin() {
                // Create confetti
                for (let i = 0; i < 100; i++) {
                    setTimeout(() => {
                        const confetti = document.createElement('div');
                        confetti.className = 'confetti';
                        confetti.style.left = `${Math.random() * 100}vw`;
                        confetti.style.top = '-10px';
                        confetti.style.backgroundColor = currentPlayer === 'X' ? 
                            'var(--accent-blue)' : 'var(--accent-pink)';
                        document.body.appendChild(confetti);
                        
                        // Animate
                        const duration = Math.random() * 3 + 2;
                        confetti.style.transition = `top ${duration}s linear, opacity ${duration}s ease`;
                        
                        setTimeout(() => {
                            confetti.style.top = `${Math.random() * 100 + 100}vh`;
                            confetti.style.opacity = '0';
                        }, 10);
                        
                        // Remove after animation
                        setTimeout(() => {
                            confetti.remove();
                        }, duration * 1000 + 100);
                    }, i * 50);
                }
            }
            
            function resetGame() {
                currentPlayer = 'X';
                gameState = ['', '', '', '', '', '', '', '', ''];
                gameActive = true;
                
                currentPlayerDisplay.textContent = currentPlayer;
                currentPlayerDisplay.className = 'player-x';
                
                cells.forEach(cell => {
                    cell.textContent = '';
                    cell.className = 'cell';
                    
                    // Remove any strike lines
                    const strike = cell.querySelector('.win-strike');
                    if (strike) strike.remove();
                });
            }
        });
    </script>
</body>
</html>
