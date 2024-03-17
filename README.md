<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>4 in a Row</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
            height: 100vh;
            overflow: hidden;
        }
        .board {
            display: inline-block;
            border-collapse: collapse;
            border: 2px solid black;
            width: auto;
            max-width: 600px;
        }
        .board td {
            width: 50px;
            height: 50px;
            border: 1px solid black;
            border-radius: 50%;
        }
        .board .empty {
            background-color: white;
        }
        .board .player1 {
            background-color: blue;
        }
        .board .player2 {
            background-color: red;
        }
        .board .player3 {
            background-color: #ffcc00;
        }
        .column-numbers, .player-buttons {
            margin-top: 10px;
        }
        .column-numbers button, .player-buttons button {
            font-size: 20px;
            margin: 0 5px;
            padding: 10px 15px;
            cursor: pointer;
            border: none;
            background-color: #4CAF50;
            color: white;
            border-radius: 5px;
        }
        .restart-button, .play-again-button {
            margin-top: 20px;
            font-size: 20px;
            padding: 10px 15px;
            cursor: pointer;
            border: none;
            background-color: #4CAF50;
            color: white;
            border-radius: 5px;
        }
        .turn-indicator {
            margin-top: 20px;
            font-size: 24px;
        }
        .modal {
            display: none;
            align-items: center;
            justify-content: center;
            position: fixed;
            z-index: 1;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.7);
        }
        .modal-content {
            background-color: #fefefe;
            padding: 20px;
            border: 1px solid #888;
            text-align: center;
            border-radius: 10px;
        }
        .modal-content h2 {
            margin-top: 0;
        }
        .btn-restart {
            margin-top: 20px;
            font-size: 20px;
            padding: 10px 15px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div id="startModal" class="modal">
        <div class="modal-content">
            <h2>Welcome to 4 in a Row</h2>
            <div class="player-buttons">
                <button class="btn-mode" onclick="selectMode(2)">2 Player</button>
                <button class="btn-mode" onclick="selectMode(3)">3 Player</button>
            </div>
            <div id="player-names" style="display: none;">
                <input type="text" id="player1-name" placeholder="Player 1 Name">
                <input type="text" id="player2-name" placeholder="Player 2 Name">
                <input type="text" id="player3-name" placeholder="Player 3 Name" style="display: none;">
                <button onclick="startGame()">Start Game</button>
                <button onclick="skipNames()">Skip</button>
                <button onclick="goBack()">Back</button>
            </div>
        </div>
    </div>
    <div id="winModal" class="modal">
        <div class="modal-content">
            <h2 id="win-text"></h2>
            <button class="btn-restart" onclick="restartGame()">Restart</button>
            <button class="play-again-button" onclick="playAgain()">Play Again</button>
        </div>
    </div>
    <h1>4 in a Row</h1>
    <table class="board"></table>
    <div class="column-numbers">
        <button onclick="handleClick(0)">1</button>
        <button onclick="handleClick(1)">2</button>
        <button onclick="handleClick(2)">3</button>
        <button onclick="handleClick(3)">4</button>
        <button onclick="handleClick(4)">5</button>
        <button onclick="handleClick(5)">6</button>
        <button onclick="handleClick(6)">7</button>
        <button id="col-7" style="display: none;" onclick="handleClick(7)">8</button>
        <button id="col-8" style="display: none;" onclick="handleClick(8)">9</button>
    </div>
    <button class="restart-button" onclick="restartGame()">Restart</button>
    <div class="turn-indicator">
        <span id="turn-text">Player 1's Turn</span>
    </div>
    <!-- Player scores section -->
    <div id="player-scores" style="display: none;">
        <div id="player1-score"></div>
        <div id="player2-score"></div>
        <div id="player3-score"></div>
    </div>
    <script>
        let numPlayers = 2;
        let rows = 6;
        let cols = 7;
        let board = [];
        let currentPlayer = 'player1';
        let gameEnded = false;

        let player1NameGlobal = '';
        let player2NameGlobal = '';
        let player3NameGlobal = '';
        let player1Score = 0;
        let player2Score = 0;
        let player3Score = 0;

        function createBoard() {
            const table = document.querySelector('.board');
            table.innerHTML = '';
            for (let i = 0; i < rows; i++) {
                const row = table.insertRow();
                board.push([]);
                for (let j = 0; j < cols; j++) {
                    const cell = row.insertCell();
                    cell.className = 'empty';
                    board[i].push('empty');
                }
            }
        }

        function updateBoard() {
            const table = document.querySelector('.board');
            for (let i = 0; i < rows; i++) {
                for (let j = 0; j < cols; j++) {
                    const cell = table.rows[i].cells[j];
                    cell.className = board[i][j];
                }
            }
        }

        function dropPiece(col) {
            for (let i = rows - 1; i >= 0; i--) {
                if (board[i][col] === 'empty') {
                    board[i][col] = currentPlayer;
                    updateBoard();
                    return;
                }
            }
        }

        function checkWin() {
            for (let i = 0; i < rows; i++) {
                for (let j = 0; j < cols; j++) {
                    if (board[i][j] !== 'empty') {
                        if (j <= cols - 4 && board[i][j] === board[i][j+1] && board[i][j] === board[i][j+2] && board[i][j] === board[i][j+3]) {
                            return true;
                        }
                        if (i <= rows - 4 && board[i][j] === board[i+1][j] && board[i][j] === board[i+2][j] && board[i][j] === board[i+3][j]) {
                            return true;
                        }
                        if (i <= rows - 4 && j <= cols - 4) {
                            if (board[i][j] === board[i+1][j+1] && board[i][j] === board[i+2][j+2] && board[i][j] === board[i+3][j+3]) {
                                return true;
                            }
                        }
                        if (i >= 3 && j <= cols - 4) {
                            if (board[i][j] === board[i-1][j+1] && board[i][j] === board[i-2][j+2] && board[i][j] === board[i-3][j+3]) {
                                return true;
                            }
                        }
                    }
                }
            }
            return false;
        }

        function handleClick(col) {
            if (gameEnded || board[0][col] !== 'empty') {
                return;
            }
            dropPiece(col);
            if (checkWin()) {
                if (numPlayers === 2) {
                    document.getElementById('win-text').innerText = currentPlayer === 'player1' ? `${player1NameGlobal} wins!` : `${player2NameGlobal} wins!`;
                    if (currentPlayer === 'player1') {
                        player1Score++;
                    } else {
                        player2Score++;
                    }
                } else {
                    document.getElementById('win-text').innerText = currentPlayer === 'player1' ? `${player1NameGlobal} wins!` : (currentPlayer === 'player2' ? `${player2NameGlobal} wins!` : `${player3NameGlobal} wins!`);
                    if (currentPlayer === 'player1') {
                        player1Score++;
                    } else if (currentPlayer === 'player2') {
                        player2Score++;
                    } else {
                        player3Score++;
                    }
                }
                document.getElementById('winModal').style.display = 'flex';
                gameEnded = true;
                return;
            }
            if (numPlayers === 2) {
                currentPlayer = currentPlayer === 'player1' ? 'player2' : 'player1';
                document.getElementById('turn-text').innerText = currentPlayer === 'player1' ? `${player1NameGlobal}'s Turn` : `${player2NameGlobal}'s Turn`;
            } else {
                currentPlayer = currentPlayer === 'player1' ? 'player2' : (currentPlayer === 'player2' ? 'player3' : 'player1');
                document.getElementById('turn-text').innerText = currentPlayer === 'player1' ? `${player1NameGlobal}'s Turn` : (currentPlayer === 'player2' ? `${player2NameGlobal}'s Turn` : `${player3NameGlobal}'s Turn`);
            }
        }

        function resetGame() {
            board = Array.from({length: rows}, () => Array.from({length: cols}, () => 'empty'));
            updateBoard();
            currentPlayer = 'player1';
            gameEnded = false;
            document.getElementById('turn-text').innerText = `${player1NameGlobal}'s Turn`;
            document.getElementById('winModal').style.display = 'none';
        }

        function restartGame() {
            resetGame();
            document.getElementById('startModal').style.display = 'flex';
        }

        function selectMode(players) {
            numPlayers = players;
            document.querySelector('.player-buttons').style.display = 'none';
            if (numPlayers === 2) {
                rows = 6;
                cols = 7;
                document.getElementById('col-7').style.display = 'none';
                document.getElementById('col-8').style.display = 'none';
                document.getElementById('player3-name').style.display = 'none';
                document.getElementById('player3-score').style.display = 'none'; // Hide third player's score
            } else if (numPlayers === 3) {
                rows = 8;
                cols = 9;
                document.getElementById('col-7').style.display = 'inline-block';
                document.getElementById('col-8').style.display = 'inline-block';
                document.getElementById('player3-name').style.display = 'inline-block';
                document.getElementById('player3-score').style.display = 'block'; // Show third player's score
            }
            document.getElementById('player-names').style.display = 'block';
        }

        function startGame() {
            const player1Name = document.getElementById('player1-name').value;
            const player2Name = document.getElementById('player2-name').value;
            const player3Name = document.getElementById('player3-name').value;

            const defaultNames = ["Player 1", "Player 2", "Player 3"];
            const names = [player1Name || defaultNames[0], player2Name || defaultNames[1], player3Name || defaultNames[2]];

            const playerNames = document.getElementById('player-names');
            const turnText = document.getElementById('turn-text');

            if (numPlayers === 2) {
                turnText.innerHTML = `${names[0]}'s Turn`;
            } else if (numPlayers === 3) {
                turnText.innerHTML = `${names[0]}'s Turn`;
            }

            player1NameGlobal = names[0];
            player2NameGlobal = names[1];
            player3NameGlobal = names[2];

            playerNames.style.display = 'none';
            createBoard();
            updateBoard();
            document.getElementById('startModal').style.display = 'none';
        }

        function skipNames() {
            document.getElementById('player1-name').value = 'Player 1';
            document.getElementById('player2-name').value = 'Player 2';
            document.getElementById('player3-name').value = 'Player 3';
            startGame();
        }

        function goBack() {
            document.getElementById('startModal').style.display = 'flex';
            document.getElementById('player-names').style.display = 'none';
            document.querySelector('.player-buttons').style.display = 'block';
        }

        function playAgain() {
            resetGame();
            document.getElementById('winModal').style.display = 'none';
            document.getElementById('player-scores').style.display = 'block';

            document.getElementById('player1-score').innerText = `${player1NameGlobal}: ${player1Score}`;
            document.getElementById('player2-score').innerText = `${player2NameGlobal}: ${player2Score}`;
            document.getElementById('player3-score').innerText = `${player3NameGlobal}: ${player3Score}`;
        }

        createBoard();
        updateBoard();
        document.getElementById('startModal').style.display = 'flex';
    </script>
</body>
</html>
