<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tic Tac Toe Online</title>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet" />
    <style>
        /* General Page Setup */
        body {
            font-family: 'Press Start 2P', cursive;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: url('background.jpg') no-repeat center center fixed;
            background-size: cover;
            color: lightblue;
            position: relative;
        }

        /* Watermark Text */
        .made-with {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 12px;
            color: #ff6b6b;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
        }

        /* Main Container */
        .container {
            text-align: center;
            background: rgba(0, 0, 0, 0.6);
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
            max-width: 400px;
            width: 100%;
            position: relative;
        }

        /* Decorative background */
        .interface-bg {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: url('interface1.png') no-repeat center center;
            background-size: cover;
            opacity: 0.25;
            border-radius: 15px;
            z-index: -1;
        }

        /* Board layout */
        .board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin: 20px 0;
        }

        .cell {
            width: 90px;
            height: 90px;
            background: rgba(255, 255, 255, 0.1);
            border: 2px solid lightblue;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2em;
            color: white;
            cursor: pointer;
            border-radius: 8px;
            transition: 0.2s;
        }

        .cell:hover {
            filter: brightness(1.3);
            background: rgba(255, 255, 255, 0.2);
        }

        button {
            padding: 12px 25px;
            margin: 8px;
            background: #4ecdc4;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 12px;
            font-family: 'Press Start 2P', cursive;
        }

        button:hover {
            background: #45b7d1;
        }

        input {
            padding: 10px;
            margin: 10px;
            border-radius: 5px;
            border: none;
            font-family: 'Press Start 2P', cursive;
            font-size: 10px;
            text-align: center;
        }

        #status {
            margin: 20px 0;
            color: #fff;
        }
    </style>
</head>
<body>
    <div class="made-with">Made with ❤️ by Krishna</div>
    <div class="container">
        <div class="interface-bg"></div>
        <h1>Tic Tac Toe</h1>
        <div id="menu">
            <button id="createGame">Create New Game</button><br>
            <input type="text" id="gameCode" placeholder="Enter Game Code" /><br>
            <button id="joinGame">Join Game</button>
        </div>

        <div id="game" style="display: none;">
            <div id="status">Waiting for opponent...</div>
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
            <button id="reset" style="display: none;">Play Again</button>
        </div>
    </div>

    <!-- Firebase Libraries -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js"></script>
    <script>
        // ✅ Replace this with your Firebase config from the Firebase Console
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
            databaseURL: "https://YOUR_PROJECT_ID-default-rtdb.firebaseio.com",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_PROJECT_ID.appspot.com",
            messagingSenderId: "YOUR_SENDER_ID",
            appId: "YOUR_APP_ID"
        };

        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        let gameRef;
        let player = '';
        let currentPlayer = 'X';
        let board = ['', '', '', '', '', '', '', '', ''];
        let gameActive = true;

        const menu = document.getElementById('menu');
        const game = document.getElementById('game');
        const status = document.getElementById('status');
        const cells = document.querySelectorAll('.cell');
        const createGameBtn = document.getElementById('createGame');
        const joinGameBtn = document.getElementById('joinGame');
        const gameCodeInput = document.getElementById('gameCode');
        const resetBtn = document.getElementById('reset');

        createGameBtn.addEventListener('click', createGame);
        joinGameBtn.addEventListener('click', joinGame);
        resetBtn.addEventListener('click', resetGame);
        cells.forEach(cell => cell.addEventListener('click', handleCellClick));

        function generateCode() {
            return Math.random().toString(36).substr(2, 8).toUpperCase();
        }

        function createGame() {
            const code = generateCode();
            gameRef = database.ref('games/' + code);
            player = 'X';
            gameRef.set({
                board: board,
                currentPlayer: currentPlayer,
                players: { X: true, O: false },
                winner: null
            });
            menu.style.display = 'none';
            game.style.display = 'block';
            status.textContent = `Game Code: ${code}. Waiting for opponent...`;
            listenForUpdates();
        }

        function joinGame() {
            const code = gameCodeInput.value.toUpperCase();
            if (!code) return alert('Enter a game code');
            gameRef = database.ref('games/' + code);
            gameRef.once('value', snapshot => {
                const data = snapshot.val();
                if (!data) return alert('Game not found');
                if (data.players.O) return alert('Game full');
                player = 'O';
                gameRef.update({ players: { X: true, O: true } });
                menu.style.display = 'none';
                game.style.display = 'block';
                status.textContent = 'You are O. Game started!';
                listenForUpdates();
            });
        }

        function listenForUpdates() {
            gameRef.on('value', snapshot => {
                const data = snapshot.val();
                if (!data) return;
                board = data.board;
                currentPlayer = data.currentPlayer;
                updateBoard();
                if (data.winner) {
                    gameActive = false;
                    status.textContent = data.winner === 'draw' ? "It's a draw!" : `${data.winner} wins!`;
                    resetBtn.style.display = 'block';
                } else if (data.players.X && data.players.O) {
                    status.textContent = currentPlayer === player ? 'Your turn' : "Opponent's turn";
                }
            });
        }

        function handleCellClick(e) {
            const index = e.target.dataset.index;
            if (!gameActive || board[index] || currentPlayer !== player) return;
            board[index] = player;
            const winner = checkWinner();
            gameRef.update({
                board: board,
                currentPlayer: currentPlayer === 'X' ? 'O' : 'X',
                winner: winner
            });
        }

        function updateBoard() {
            cells.forEach((cell, index) => {
                cell.textContent = board[index];
            });
        }

        function checkWinner() {
            const winPatterns = [
                [0, 1, 2],
                [3, 4, 5],
                [6, 7, 8],
                [0, 3, 6],
                [1, 4, 7],
                [2, 5, 8],
                [0, 4, 8],
                [2, 4, 6]
            ];
            for (let pattern of winPatterns) {
                const [a, b, c] = pattern;
                if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                    return board[a];
                }
            }
            return board.includes('') ? null : 'draw';
        }

        function resetGame() {
            board = ['', '', '', '', '', '', '', '', ''];
            currentPlayer = 'X';
            gameActive = true;
            gameRef.update({
                board: board,
                currentPlayer: currentPlayer,
                winner: null
            });
            resetBtn.style.display = 'none';
            status.textContent = 'Game reset!';
        }
    </script>
</body>
</html>

