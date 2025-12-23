<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
</head>
<body>

<h1>This is a Heading</h1>
<p>This is a paragraph.</p>

</body>
</html>
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>貪吃蛇遊戲 (帶加速功能)</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Arial', sans-serif;
        }

        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            background-color: #1a1a1a;
            color: #fff;
        }

        .game-container {
            position: relative;
            border: 2px solid #4CAF50;
            background-color: #000;
        }

        canvas {
            display: block;
        }

        .game-info {
            margin-bottom: 10px;
            display: flex;
            justify-content: space-between;
            width: 400px;
            font-size: 18px;
        }

        .score, .speed {
            padding: 5px 10px;
            background-color: #333;
            border-radius: 4px;
        }

        .controls {
            margin-top: 15px;
            text-align: center;
            font-size: 14px;
            color: #ccc;
        }

        .game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            padding: 20px 40px;
            border-radius: 8px;
            text-align: center;
            display: none;
        }

        .game-over h2 {
            color: #ff4444;
            margin-bottom: 10px;
        }

        .game-over p {
            margin-bottom: 15px;
        }

        .game-over button {
            padding: 8px 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }

        .game-over button:hover {
            background-color: #45a049;
        }
        
        .start-tip {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #ccc;
            font-size: 16px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="game-info">
        <div class="score">分數: <span id="score">0</span></div>
        <div class="speed">速度: <span id="speed">1</span>x</div>
    </div>
    
    <div class="game-container">
        <canvas id="gameCanvas" width="400" height="400"></canvas>
        <div class="start-tip" id="startTip">按方向鍵開始遊戲</div>
        <div class="game-over" id="gameOver">
            <h2>遊戲結束</h2>
            <p>最終分數: <span id="finalScore">0</span></p>
            <button onclick="restartGame()">重新開始</button>
        </div>
    </div>

    <div class="controls">
        <p>操作說明: 方向鍵控制蛇的移動 | 每吃5顆食物自動加速</p>
    </div>

    <script>
        // 遊戲設定
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const speedElement = document.getElementById('speed');
        const gameOverElement = document.getElementById('finalScore');
        const gameOverScreen = document.querySelector('.game-over');
        const startTip = document.getElementById('startTip');

        // 遊戲參數
        const gridSize = 20;
        const tileCount = canvas.width / gridSize;
        
        let snake = [
            { x: 10, y: 10 }
        ];
        let food = { x: 5, y: 5 };
        let dx = 0;
        let dy = 0;
        let score = 0;
        let gameSpeed = 100; // 初始速度 (毫秒)
        let speedMultiplier = 1; // 速度倍數顯示
        let gameLoop;
        let isGameOver = false;
        let isGameStarted = false;

        // 初始化遊戲
        function initGame() {
            snake = [{ x: 10, y: 10 }];
            dx = 0;
            dy = 0;
            score = 0;
            gameSpeed = 100;
            speedMultiplier = 1;
            isGameOver = false;
            isGameStarted = false;
            gameOverScreen.style.display = 'none';
            startTip.style.display = 'block';
            
            // 隨機生成食物位置
            generateFood();
            
            // 更新分數和速度顯示
            updateScore();
            updateSpeed();
            
            // 立即繪製初始畫面 (解決蛇不顯示問題)
            drawGame();
            
            // 清除舊的遊戲迴圈
            if (gameLoop) clearInterval(gameLoop);
        }

        // 開始遊戲 (第一次按方向鍵時)
        function startGame() {
            if (!isGameStarted && (dx !== 0 || dy !== 0)) {
                isGameStarted = true;
                startTip.style.display = 'none';
                gameLoop = setInterval(gameUpdate, gameSpeed);
            }
        }

        // 遊戲更新邏輯
        function gameUpdate() {
            if (isGameOver) return;
            
            // 移動蛇
            moveSnake();
            
            // 檢查碰撞
            if (checkCollision()) {
                gameOver();
                return;
            }
            
            // 檢查是否吃到食物
            if (snake[0].x === food.x && snake[0].y === food.y) {
                // 增加分數
                score += 10;
                updateScore();
                
                // 不刪除蛇尾 (蛇變長)
                generateFood();
                
                // 每吃5顆食物加速一次
                if (score % 50 === 0 && gameSpeed > 30) {
                    increaseSpeed();
                }
            } else {
                // 刪除蛇尾 (維持長度)
                snake.pop();
            }
            
            // 繪製遊戲畫面
            drawGame();
        }

        // 移動蛇
        function moveSnake() {
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            snake.unshift(head);
        }

        // 檢查碰撞 (牆壁或自身)
        function checkCollision() {
            // 牆壁碰撞
            if (
                snake[0].x < 0 || 
                snake[0].x >= tileCount || 
                snake[0].y < 0 || 
                snake[0].y >= tileCount
            ) {
                return true;
            }
            
            // 自身碰撞
            for (let i = 1; i < snake.length; i++) {
                if (snake[0].x === snake[i].x && snake[0].y === snake[i].y) {
                    return true;
                }
            }
            
            return false;
        }

        // 隨機生成食物
        function generateFood() {
            // 確保食物不會生成在蛇身上
            let validPosition = false;
            while (!validPosition) {
                food.x = Math.floor(Math.random() * tileCount);
                food.y = Math.floor(Math.random() * tileCount);
                
                // 檢查是否與蛇重疊
                validPosition = true;
                for (let segment of snake) {
                    if (segment.x === food.x && segment.y === food.y) {
                        validPosition = false;
                        break;
                    }
                }
            }
        }

        // 加速功能
        function increaseSpeed() {
            gameSpeed -= 10; // 減少延遲時間 (加快速度)
            speedMultiplier = (100 / gameSpeed).toFixed(1); // 計算速度倍數
            
            // 更新速度顯示
            updateSpeed();
            
            // 重新設定遊戲迴圈速度
            clearInterval(gameLoop);
            gameLoop = setInterval(gameUpdate, gameSpeed);
        }

        // 更新分數顯示
        function updateScore() {
            scoreElement.textContent = score;
        }

        // 更新速度顯示
        function updateSpeed() {
            speedElement.textContent = speedMultiplier;
        }

        // 遊戲結束
        function gameOver() {
            isGameOver = true;
            clearInterval(gameLoop);
            gameOverElement.textContent = score;
            gameOverScreen.style.display = 'block';
        }

        // 重新開始遊戲
        function restartGame() {
            initGame();
        }

        // 繪製遊戲畫面 (核心修正：確保蛇一開始就顯示)
        function drawGame() {
            // 清空畫布
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // 繪製食物 (紅色)
            ctx.fillStyle = '#ff4444';
            ctx.fillRect(food.x * gridSize, food.y * gridSize, gridSize - 1, gridSize - 1);
            
            // 繪製蛇 (綠色) - 確保每一節都清晰顯示
            ctx.fillStyle = '#4CAF50';
            snake.forEach((segment, index) => {
                // 蛇頭用深綠色區分
                if (index === 0) {
                    ctx.fillStyle = '#2E7D32';
                } else {
                    ctx.fillStyle = '#4CAF50';
                }
                ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize - 1, gridSize - 1);
            });
        }

        // 鍵盤控制 (優化：增加遊戲啟動邏輯)
        document.addEventListener('keydown', (e) => {
            // 防止連續按相反方向
            if (isGameOver) return;
            
            const LEFT_KEY = 37;
            const RIGHT_KEY = 39;
            const UP_KEY = 38;
            const DOWN_KEY = 40;

            const keyPressed = e.keyCode;
            
            // 向左
            if (keyPressed === LEFT_KEY && dx !== 1) {
                dx = -1;
                dy = 0;
                startGame(); // 嘗試啟動遊戲
            }
            
            // 向右
            if (keyPressed === RIGHT_KEY && dx !== -1) {
                dx = 1;
                dy = 0;
                startGame(); // 嘗試啟動遊戲
            }
            
            // 向上
            if (keyPressed === UP_KEY && dy !== 1) {
                dx = 0;
                dy = -1;
                startGame(); // 嘗試啟動遊戲
            }
            
            // 向下
            if (keyPressed === DOWN_KEY && dy !== -1) {
                dx = 0;
                dy = 1;
                startGame(); // 嘗試啟動遊戲
            }
            
            // 立即重新繪製畫面
            drawGame();
        });

        // 啟動遊戲
        window.onload = initGame;
    </script>
</body>
</html>
