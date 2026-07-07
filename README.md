<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tetris</title>
    <style>
        /* ===== Minimalism: Xám + Xanh ===== */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #0d1117;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Courier New', monospace;
        }

        .container {
            background: #161b22;
            padding: 30px 40px;
            border-radius: 12px;
            box-shadow: 0 8px 24px rgba(0, 0, 0, 0.6);
            display: flex;
            gap: 40px;
            align-items: flex-start;
            border: 1px solid #30363d;
        }

        /* ===== Bảng game ===== */
        .game-area {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        canvas#tetris {
            background: #0d1117;
            border: 2px solid #4A90D9;
            border-radius: 6px;
            display: block;
        }

        .controls {
            margin-top: 16px;
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .controls button {
            background: #21262d;
            color: #c9d1d9;
            border: 1px solid #30363d;
            padding: 8px 16px;
            border-radius: 6px;
            font-size: 14px;
            font-family: inherit;
            cursor: pointer;
            transition: 0.2s;
            min-width: 60px;
        }

        .controls button:hover {
            background: #4A90D9;
            border-color: #4A90D9;
            color: #fff;
        }

        /* ===== Bảng thông tin bên phải ===== */
        .info {
            display: flex;
            flex-direction: column;
            gap: 20px;
            min-width: 140px;
        }

        .info-box {
            background: #0d1117;
            padding: 16px 20px;
            border-radius: 8px;
            border: 1px solid #30363d;
            text-align: center;
        }

        .info-box .label {
            color: #808080;
            font-size: 12px;
            text-transform: uppercase;
            letter-spacing: 1px;
            margin-bottom: 4px;
        }

        .info-box .value {
            color: #4A90D9;
            font-size: 28px;
            font-weight: bold;
        }

        #nextCanvas {
            background: #0d1117;
            border: 1px solid #30363d;
            border-radius: 6px;
            display: block;
            margin: 0 auto;
        }

        /* ===== Responsive ===== */
        @media (max-width: 700px) {
            .container {
                flex-direction: column;
                align-items: center;
                padding: 20px;
                gap: 24px;
            }
            .info {
                flex-direction: row;
                flex-wrap: wrap;
                justify-content: center;
                min-width: unset;
            }
            .info-box {
                min-width: 100px;
            }
        }
    </style>
</head>
<body>

    <div class="container">

        <!-- Bên trái: Game -->
        <div class="game-area">
            <canvas id="tetris" width="300" height="600"></canvas>
            <div class="controls">
                <button id="btnLeft">◄</button>
                <button id="btnRight">►</button>
                <button id="btnRotate">↻</button>
                <button id="btnDrop">↓</button>
                <button id="btnPause">⏸</button>
                <button id="btnReset">⟳</button>
            </div>
        </div>

        <!-- Bên phải: Thông tin -->
        <div class="info">
            <div class="info-box">
                <div class="label">Điểm</div>
                <div class="value" id="scoreDisplay">0</div>
            </div>
            <div class="info-box">
                <div class="label">Cấp độ</div>
                <div class="value" id="levelDisplay">1</div>
            </div>
            <div class="info-box">
                <div class="label">Tiếp theo</div>
                <canvas id="nextCanvas" width="120" height="120"></canvas>
            </div>
        </div>

    </div>

    <script>
        // ============================================================
        //  TETRIS – Minimalism (Xám + Xanh)
        // ============================================================

        // --- Kích thước ---
        const COLS = 10;
        const ROWS = 20;
        const BLOCK_SIZE = 30;

        // --- Canvas chính ---
        const canvas = document.getElementById('tetris');
        const ctx = canvas.getContext('2d');

        // --- Canvas "Next" ---
        const nextCanvas = document.getElementById('nextCanvas');
        const nextCtx = nextCanvas.getContext('2d');

        // --- DOM hiển thị ---
        const scoreSpan = document.getElementById('scoreDisplay');
        const levelSpan = document.getElementById('levelDisplay');

        // --- Màu sắc (xám + xanh) ---
        const COLORS = {
            I: '#4A90D9', // xanh chủ đạo
            O: '#808080', // xám
            T: '#5A9FD9',
            S: '#6B9FD9',
            Z: '#3A7FBF',
            L: '#4A90D9',
            J: '#2A6FBF',
            ghost: 'rgba(74, 144, 217, 0.15)',
            grid: '#21262d',
            background: '#0d1117',
        };

        // --- Định nghĩa các khối ---
        const SHAPES = {
            I: { matrix: [
                    [0, 0, 0, 0],
                    [1, 1, 1, 1],
                    [0, 0, 0, 0],
                    [0, 0, 0, 0]
                ], color: 'I' },
            O: { matrix: [
                    [1, 1],
                    [1, 1]
                ], color: 'O' },
            T: { matrix: [
                    [0, 1, 0],
                    [1, 1, 1],
                    [0, 0, 0]
                ], color: 'T' },
            S: { matrix: [
                    [0, 1, 1],
                    [1, 1, 0],
                    [0, 0, 0]
                ], color: 'S' },
            Z: { matrix: [
                    [1, 1, 0],
                    [0, 1, 1],
                    [0, 0, 0]
                ], color: 'Z' },
            L: { matrix: [
                    [1, 0, 0],
                    [1, 1, 1],
                    [0, 0, 0]
                ], color: 'L' },
            J: { matrix: [
                    [0, 0, 1],
                    [1, 1, 1],
                    [0, 0, 0]
                ], color: 'J' },
        };

        const PIECE_NAMES = ['I', 'O', 'T', 'S', 'Z', 'L', 'J'];

        // --- Trạng thái game ---
        let board = [];
        let currentPiece = null;
        let nextPiece = null;
        let score = 0;
        let level = 1;
        let gameOver = false;
        let paused = false;
        let dropInterval = null;
        const BASE_INTERVAL = 500; // ms

        // ============================================================
        //  HÀM KHỞI TẠO
        // ============================================================

        function createBoard() {
            board = [];
            for (let r = 0; r < ROWS; r++) {
                board.push(new Array(COLS).fill(0));
            }
        }

        function randomPiece() {
            const name = PIECE_NAMES[Math.floor(Math.random() * PIECE_NAMES.length)];
            const shape = SHAPES[name];
            return {
                name: name,
                matrix: shape.matrix.map(row => [...row]),
                color: shape.color,
                row: 0,
                col: Math.floor((COLS - shape.matrix[0].length) / 2),
            };
        }

        function spawnNewPiece() {
            if (!nextPiece) {
                nextPiece = randomPiece();
            }
            // piece hiện tại = next, rồi tạo next mới
            currentPiece = {
                name: nextPiece.name,
                matrix: nextPiece.matrix.map(row => [...row]),
                color: nextPiece.color,
                row: 0,
                col: Math.floor((COLS - nextPiece.matrix[0].length) / 2),
            };
            nextPiece = randomPiece();

            // Kiểm tra game over
            if (collision(currentPiece.matrix, currentPiece.row, currentPiece.col)) {
                gameOver = true;
                clearInterval(dropInterval);
                dropInterval = null;
            }

            drawNext();
            draw();
        }

        // ============================================================
        //  XỬ LÝ VA CHẠM
        // ============================================================

        function collision(matrix, row, col) {
            for (let r = 0; r < matrix.length; r++) {
                for (let c = 0; c < matrix[0].length; c++) {
                    if (matrix[r][c] !== 0) {
                        const boardR = row + r;
                        const boardC = col + c;
                        if (boardR >= ROWS || boardC < 0 || boardC >= COLS || boardR < 0) {
                            return true;
                        }
                        if (boardR >= 0 && board[boardR][boardC] !== 0) {
                            return true;
                        }
                    }
                }
            }
            return false;
        }

        // ============================================================
        //  GHÉP PIECE VÀO BOARD
        // ============================================================

        function mergePiece() {
            if (!currentPiece) return;
            const { matrix, row, col, color } = currentPiece;
            for (let r = 0; r < matrix.length; r++) {
                for (let c = 0; c < matrix[0].length; c++) {
                    if (matrix[r][c] !== 0) {
                        const boardR = row + r;
                        const boardC = col + c;
                        if (boardR >= 0 && boardR < ROWS && boardC >= 0 && boardC < COLS) {
                            board[boardR][boardC] = color;
                        }
                    }
                }
            }
        }

        // ============================================================
        //  XÓA HÀNG & TÍNH ĐIỂM
        // ============================================================

        function clearRows() {
            let cleared = 0;
            for (let r = ROWS - 1; r >= 0; ) {
                let full = true;
                for (let c = 0; c < COLS; c++) {
                    if (board[r][c] === 0) {
                        full = false;
                        break;
                    }
                }
                if (full) {
                    // Xoá hàng
                    for (let r2 = r; r2 > 0; r2--) {
                        board[r2] = [...board[r2 - 1]];
                    }
                    board[0] = new Array(COLS).fill(0);
                    cleared++;
                    // Không tăng r, kiểm tra lại hàng hiện tại
                } else {
                    r--;
                }
            }

            if (cleared > 0) {
                // Tính điểm: 1 hàng = 100 * level, 2 = 300, 3 = 500, 4 = 800
                const points = [0, 100, 300, 500, 800];
                const addScore = points[cleared] * level;
                score += addScore;
                level = Math.floor(score / 1000) + 1;
                updateInfo();
                resetTimer();
            }
        }

        // ============================================================
        //  DI CHUYỂN / XOAY
        // ============================================================

        function movePiece(dCol, dRow) {
            if (!currentPiece || gameOver || paused) return false;
            const { matrix, row, col } = currentPiece;
            if (!collision(matrix, row + dRow, col + dCol)) {
                currentPiece.row += dRow;
                currentPiece.col += dCol;
                draw();
                return true;
            }
            // Nếu di chuyển xuống thất bại => ghép và spawn mới
            if (dRow === 1) {
                lockPiece();
            }
            return false;
        }

        function rotatePiece() {
            if (!currentPiece || gameOver || paused) return;
            const oldMatrix = currentPiece.matrix;
            const n = oldMatrix.length;
            const rotated = [];
            for (let r = 0; r < n; r++) {
                rotated.push([]);
                for (let c = 0; c < n; c++) {
                    rotated[r].push(oldMatrix[n - 1 - c][r]);
                }
            }
            const { row, col } = currentPiece;
            if (!collision(rotated, row, col)) {
                currentPiece.matrix = rotated;
                draw();
            } else {
                // Thử shift sang trái/phải nếu xoay bị vướng
                if (!collision(rotated, row, col - 1)) {
                    currentPiece.matrix = rotated;
                    currentPiece.col -= 1;
                    draw();
                } else if (!collision(rotated, row, col + 1)) {
                    currentPiece.matrix = rotated;
                    currentPiece.col += 1;
                    draw();
                }
            }
        }

        function hardDrop() {
            if (!currentPiece || gameOver || paused) return;
            while (!collision(currentPiece.matrix, currentPiece.row + 1, currentPiece.col)) {
                currentPiece.row++;
            }
            lockPiece();
        }

        function lockPiece() {
            if (!currentPiece) return;
            mergePiece();
            clearRows();
            spawnNewPiece();
            draw();
            if (gameOver) {
                clearInterval(dropInterval);
                dropInterval = null;
                draw();
            }
        }

        // ============================================================
        //  TIMER
        // ============================================================

        function resetTimer() {
            if (dropInterval) {
                clearInterval(dropInterval);
                dropInterval = null;
            }
            if (!gameOver && !paused) {
                const interval = Math.max(100, BASE_INTERVAL - (level - 1) * 40);
                dropInterval = setInterval(() => {
                    if (!paused && !gameOver && currentPiece) {
                        movePiece(0, 1);
                    }
                }, interval);
            }
        }

        // ============================================================
        //  VẼ
        // ============================================================

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Vẽ board
            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    const color = board[r][c];
                    if (color !== 0) {
                        ctx.fillStyle = COLORS[color] || '#4A90D9';
                        ctx.fillRect(c * BLOCK_SIZE, r * BLOCK_SIZE, BLOCK_SIZE - 1, BLOCK_SIZE - 1);
                    } else {
                        // Vẽ grid nhẹ
                        ctx.strokeStyle = '#21262d';
                        ctx.lineWidth = 0.5;
                        ctx.strokeRect(c * BLOCK_SIZE, r * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
                    }
                }
            }

            // Vẽ ghost (bóng)
            if (currentPiece && !gameOver) {
                let ghostRow = currentPiece.row;
                while (!collision(currentPiece.matrix, ghostRow + 1, currentPiece.col)) {
                    ghostRow++;
                }
                const { matrix, col, color } = currentPiece;
                for (let r = 0; r < matrix.length; r++) {
                    for (let c = 0; c < matrix[0].length; c++) {
                        if (matrix[r][c] !== 0) {
                            const x = (col + c) * BLOCK_SIZE;
                            const y = (ghostRow + r) * BLOCK_SIZE;
                            ctx.fillStyle = COLORS.ghost;
                            ctx.fillRect(x, y, BLOCK_SIZE - 1, BLOCK_SIZE - 1);
                        }
                    }
                }
            }

            // Vẽ current piece
            if (currentPiece && !gameOver) {
                const { matrix, row, col, color } = currentPiece;
                for (let r = 0; r < matrix.length; r++) {
                    for (let c = 0; c < matrix[0].length; c++) {
                        if (matrix[r][c] !== 0) {
                            const x = (col + c) * BLOCK_SIZE;
                            const y = (row + r) * BLOCK_SIZE;
                            ctx.fillStyle = COLORS[color] || '#4A90D9';
                            ctx.fillRect(x, y, BLOCK_SIZE - 1, BLOCK_SIZE - 1);
                        }
                    }
                }
            }

            // Vẽ game over
            if (gameOver) {
                ctx.fillStyle = 'rgba(0,0,0,0.7)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#4A90D9';
                ctx.font = 'bold 28px "Courier New", monospace';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('GAME OVER', canvas.width / 2, canvas.height / 2 - 10);
                ctx.fillStyle = '#808080';
                ctx.font = '16px "Courier New", monospace';
                ctx.fillText('Nhấn ⟳ để chơi lại', canvas.width / 2, canvas.height / 2 + 40);
            } else if (paused) {
                ctx.fillStyle = 'rgba(0,0,0,0.6)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#808080';
                ctx.font = 'bold 28px "Courier New", monospace';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('⏸ PAUSED', canvas.width / 2, canvas.height / 2);
            }
        }

        function drawNext() {
            nextCtx.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            if (!nextPiece) return;
            const matrix = nextPiece.matrix;
            const color = nextPiece.color;
            const block = 30;
            const offsetX = (nextCanvas.width - matrix[0].length * block) / 2;
            const offsetY = (nextCanvas.height - matrix.length * block) / 2;
            for (let r = 0; r < matrix.length; r++) {
                for (let c = 0; c < matrix[0].length; c++) {
                    if (matrix[r][c] !== 0) {
                        nextCtx.fillStyle = COLORS[color] || '#4A90D9';
                        nextCtx.fillRect(offsetX + c * block, offsetY + r * block, block - 1, block - 1);
                    }
                }
            }
        }

        function updateInfo() {
            scoreSpan.textContent = score;
            levelSpan.textContent = level;
        }

        // ============================================================
        //  RESET GAME
        // ============================================================

        function resetGame() {
            if (dropInterval) {
                clearInterval(dropInterval);
                dropInterval = null;
            }
            createBoard();
            score = 0;
            level = 1;
            gameOver = false;
            paused = false;
            document.getElementById('btnPause').textContent = '⏸';
            updateInfo();
            nextPiece = randomPiece();
            spawnNewPiece();
            resetTimer();
            draw();
        }

        // ============================================================
        //  PAUSE
        // ============================================================

        function togglePause() {
            if (gameOver) return;
            paused = !paused;
            document.getElementById('btnPause').textContent = paused ? '▶' : '⏸';
            if (paused) {
                if (dropInterval) {
                    clearInterval(dropInterval);
                    dropInterval = null;
                }
            } else {
                resetTimer();
            }
            draw();
        }

        // ============================================================
        //  KHỞI TẠO & SỰ KIỆN PHÍM
        // ============================================================

        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') { e.preventDefault();
                movePiece(-1, 0); }
            if (e.key === 'ArrowRight') { e.preventDefault();
                movePiece(1, 0); }
            if (e.key === 'ArrowDown') { e.preventDefault();
                movePiece(0, 1); }
            if (e.key === 'ArrowUp') { e.preventDefault();
                rotatePiece(); }
            if (e.key === ' ') { e.preventDefault();
                hardDrop(); }
            if (e.key === 'p' || e.key === 'P') { e.preventDefault();
                togglePause(); }
            if (e.key === 'r' || e.key === 'R') { e.preventDefault();
                resetGame(); }
        });

        // Nút bấm
        document.getElementById('btnLeft').addEventListener('click', () => movePiece(-1, 0));
        document.getElementById('btnRight').addEventListener('click', () => movePiece(1, 0));
        document.getElementById('btnRotate').addEventListener('click', rotatePiece);
        document.getElementById('btnDrop').addEventListener('click', hardDrop);
        document.getElementById('btnPause').addEventListener('click', togglePause);
        document.getElementById('btnReset').addEventListener('click', resetGame);

        // ============================================================
        //  BẮT ĐẦU GAME
        // ============================================================

        resetGame();
    </script>

</body>
</html>
