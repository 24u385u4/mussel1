# mussel1
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sticky Mussel 2048</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }

        body {
            background-color: #8fafac;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }

        h1 {
            font-size: 40px;
            color: #29303d;
            margin-bottom: 20px;
        }

        .game-container {
            position: relative;
            padding: 15px;
            background-color: #29303d;
            border-radius: 10px;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(4, 100px);
            grid-template-rows: repeat(4, 100px);
            gap: 15px;
        }

        .cell {
            width: 100px;
            height: 100px;
            background-color: #c0dedf;
            border-radius: 5px;
        }

        .tile {
            position: absolute;
            width: 100px;
            height: 100px;
            border-radius: 5px;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.7s ease;
            z-index: 10;
        }

        .tile img {
            width: 100px;
            height: 100px;
            object-fit: contain;
        }

        .score-container {
            background-color: #deb57b;
            color: #29303d;
            padding: 10px 20px;
            border-radius: 5px;
            font-weight: bold;
            margin-bottom: 20px;
        }

        .game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(238, 228, 218, 0.73);
            display: none;
            align-items: center;
            justify-content: center;
            flex-direction: column;
            font-size: 25px;
            color: #29303d;
            z-index: 100;
        }

        .game-over button {
            padding: 10px 20px;
            background-color: #deb57b;
            color: 29303d;
            border: none;
            border-radius: 5px;
            font-size: 20px;
            margin-top: 20px;
            cursor: pointer;
        }

        .win-message {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(146, 204, 112, 0.5);
            display: none;
            align-items: center;
            justify-content: center;
            flex-direction: column;
            font-size: 25px;
            color: #29303d;
            z-index: 90;
        }

        .win-message button {
            padding: 10px 20px;
            background-color: #deb57b;
            color: 29303d;
            border: none;
            border-radius: 5px;
            font-size: 20px;
            margin-top: 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Sticky Mussel 2048</h1>
    <div class="score-container">Score: <span id="score">0</span></div>
    <div class="game-container">
        <div class="grid" id="grid"></div>
        <div class="win-message" id="winMessage">
            <p>Congrats! You just got the Ultimate Mussel!</p >
            <button onclick="continueGame()">Hooray!!!</button>
        </div>
        <div class="game-over" id="gameOver">
            <p>Game over. You slipped off the rock!</p >
            <button onclick="restartGame()">Restart</button>
        </div>
    </div>

    <script>
        // 图片路径（按等级从低到高，对应2、4、8...2048）
        const images = [
            'https://i.imgs.ovh/2025/07/11/8IZ6q.jpeg',       // 2
            'https://i.imgs.ovh/2025/07/11/8IalC.jpeg',      // 4
            'https://i.imgs.ovh/2025/07/11/8IfAA.jpeg',      // 8
            'https://i.imgs.ovh/2025/07/11/83vOd.jpeg',  // 16
            'https://i.imgs.ovh/2025/07/11/8Iq1O.jpeg',      // 32
            'https://i.imgs.ovh/2025/07/12/s4TiH.jpeg',       // 64
            'https://i.imgs.ovh/2025/07/12/s4VbU.jpeg',        // 128
            'https://i.imgs.ovh/2025/07/12/sIzCq.jpeg',        // 256
            'https://i.imgs.ovh/2025/07/12/sIRJ4.jpeg',     // 512
            'https://i.imgs.ovh/2025/07/13/9yBXd.jpeg',    // 1024
            'https://i.imgs.ovh/2025/07/11/8IXfQ.jpeg'        // 2048
        ];

        // 游戏变量
        let grid;
        let score = 0;
        let gameOver = false;
        let hasWon = false;

        // 初始化游戏
        function initGame() {
            grid = Array(4).fill().map(() => Array(4).fill(0));
            score = 0;
            gameOver = false;
            hasWon = false;
            document.getElementById('score').textContent = '0';
            document.getElementById('gameOver').style.display = 'none';
            document.getElementById('winMessage').style.display = 'none';
            
            // 初始化两个数字
            addRandomTile();
            addRandomTile();
            
            // 渲染网格
            renderGrid();
        }

        // 在随机位置添加数字
        function addRandomTile() {
            // 找出所有空位置
            const emptyCells = [];
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    if (grid[i][j] === 0) {
                        emptyCells.push({x: i, y: j});
                    }
                }
            }
            
            // 如果没有空位置，检查游戏结束
            if (emptyCells.length === 0) {
                checkGameOver();
                return;
            }
            
            // 随机选择一个空位置
            const randomCell = emptyCells[Math.floor(Math.random() * emptyCells.length)];
            // 90%概率生成等级1(2)，10%概率生成等级2(4)
            grid[randomCell.x][randomCell.y] = Math.random() < 0.9 ? 1 : 2;
        }

        // 渲染网格
        function renderGrid() {
            const gridElement = document.getElementById('grid');
            gridElement.innerHTML = '';
            
            // 创建单元格
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    gridElement.appendChild(cell);
                    
                    // 如果有数字，添加对应图片
                    if (grid[i][j] > 0) {
                        const tile = document.createElement('div');
                        tile.className = 'tile';
                        tile.style.left = `${j * 115 + 15}px`;  // 100px单元格 + 15px间距
                        tile.style.top = `${i * 115 + 15}px`;
                        
                        const img = document.createElement('img');
                        // 根据等级获取对应的图片
                        const imageIndex = Math.min(grid[i][j] - 1, images.length - 1);
                        img.src = images[imageIndex];
                        img.alt = `等级 ${grid[i][j]}`;
                        
                        tile.appendChild(img);
                        gridElement.appendChild(tile);
                    }
                }
            }

            // 检查是否获胜（达到2048）
            if (!hasWon) {
                checkWin();
            }
        }

        // 处理移动
        function handleMove(direction) {
            if (gameOver || hasWon) return;
            
            let moved = false;
            let newGrid = JSON.parse(JSON.stringify(grid)); // 复制当前网格
            
            switch(direction) {
                case 'up':
                    moved = moveUp(newGrid);
                    break;
                case 'down':
                    moved = moveDown(newGrid);
                    break;
                case 'left':
                    moved = moveLeft(newGrid);
                    break;
                case 'right':
                    moved = moveRight(newGrid);
                    break;
            }
            
            // 如果有移动，更新网格并添加新数字
            if (moved) {
                grid = newGrid;
                addRandomTile();
                renderGrid();
            }
            
            checkGameOver();
        }

        // 向上移动
        function moveUp(grid) {
            let moved = false;
            
            for (let j = 0; j < 4; j++) {
                for (let i = 1; i < 4; i++) {
                    if (grid[i][j] !== 0) {
                        let row = i;
                        // 向上移动直到碰到其他数字或边界
                        while (row > 0 && grid[row - 1][j] === 0) {
                            grid[row - 1][j] = grid[row][j];
                            grid[row][j] = 0;
                            row--;
                            moved = true;
                        }
                        
                        // 检查是否可以合并
                        if (row > 0 && grid[row - 1][j] === grid[row][j]) {
                            grid[row - 1][j]++; // 等级提升
                            grid[row][j] = 0;
                            // 分数增加（2的n次方，n为等级）
                            score += Math.pow(2, grid[row - 1][j]);
                            document.getElementById('score').textContent = score;
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }

        // 向下移动
        function moveDown(grid) {
            let moved = false;
            
            for (let j = 0; j < 4; j++) {
                for (let i = 2; i >= 0; i--) {
                    if (grid[i][j] !== 0) {
                        let row = i;
                        // 向下移动直到碰到其他数字或边界
                        while (row < 3 && grid[row + 1][j] === 0) {
                            grid[row + 1][j] = grid[row][j];
                            grid[row][j] = 0;
                            row++;
                            moved = true;
                        }
                        
                        // 检查是否可以合并
                        if (row < 3 && grid[row + 1][j] === grid[row][j]) {
                            grid[row + 1][j]++; // 等级提升
                            grid[row][j] = 0;
                            score += Math.pow(2, grid[row + 1][j]);
                            document.getElementById('score').textContent = score;
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }

        // 向左移动
        function moveLeft(grid) {
            let moved = false;
            
            for (let i = 0; i < 4; i++) {
                for (let j = 1; j < 4; j++) {
                    if (grid[i][j] !== 0) {
                        let col = j;
                        // 向左移动直到碰到其他数字或边界
                        while (col > 0 && grid[i][col - 1] === 0) {
                            grid[i][col - 1] = grid[i][col];
                            grid[i][col] = 0;
                            col--;
                            moved = true;
                        }
                        
                        // 检查是否可以合并
                        if (col > 0 && grid[i][col - 1] === grid[i][col]) {
                            grid[i][col - 1]++; // 等级提升
                            grid[i][col] = 0;
                            score += Math.pow(2, grid[i][col - 1]);
                            document.getElementById('score').textContent = score;
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }

        // 向右移动
        function moveRight(grid) {
            let moved = false;
            
            for (let i = 0; i < 4; i++) {
                for (let j = 2; j >= 0; j--) {
                    if (grid[i][j] !== 0) {
                        let col = j;
                        // 向右移动直到碰到其他数字或边界
                        while (col < 3 && grid[i][col + 1] === 0) {
                            grid[i][col + 1] = grid[i][col];
                            grid[i][col] = 0;
                            col++;
                            moved = true;
                        }
                        
                        // 检查是否可以合并
                        if (col < 3 && grid[i][col + 1] === grid[i][col]) {
                            grid[i][col + 1]++; // 等级提升
                            grid[i][col] = 0;
                            score += Math.pow(2, grid[i][col + 1]);
                            document.getElementById('score').textContent = score;
                            moved = true;
                        }
                    }
                }
            }
            
            return moved;
        }

        // 检查是否获胜（达到2048）
        function checkWin() {
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    if (grid[i][j] === 11) { // 等级11对应2048
                        hasWon = true;
                        document.getElementById('winMessage').style.display = 'flex';
                        return true;
                    }
                }
            }
            return false;
        }

        // 检查游戏是否结束
        function checkGameOver() {
            // 检查是否还有空位置
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    if (grid[i][j] === 0) {
                        return false;
                    }
                }
            }
            
            // 检查是否还能合并
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    if (
                        (i < 3 && grid[i][j] === grid[i + 1][j]) ||
                        (j < 3 && grid[i][j] === grid[i][j + 1])
                    ) {
                        return false;
                    }
                }
            }
            
            // 游戏结束
            gameOver = true;
            document.getElementById('gameOver').style.display = 'flex';
            return true;
        }

        // 继续游戏（获胜后）
        function continueGame() {
            document.getElementById('winMessage').style.display = 'none';
        }

        // 重新开始游戏
        function restartGame() {
            initGame();
        }

        // 键盘控制
        document.addEventListener('keydown', (e) => {
            switch(e.key) {
                case 'ArrowUp':
                    handleMove('up');
                    break;
                case 'ArrowDown':
                    handleMove('down');
                    break;
                case 'ArrowLeft':
                    handleMove('left');
                    break;
                case 'ArrowRight':
                    handleMove('right');
                    break;
            }
        });

        // 触摸控制（移动设备）
        let touchStartX = 0;
        let touchStartY = 0;

        document.addEventListener('touchstart', (e) => {
            touchStartX = e.touches[0].clientX;
            touchStartY = e.touches[0].clientY;
        });

        document.addEventListener('touchend', (e) => {
            if (!gameOver) {
                const touchEndX = e.changedTouches[0].clientX;
                const touchEndY = e.changedTouches[0].clientY;
                
                const diffX = touchEndX - touchStartX;
                const diffY = touchEndY - touchStartY;
                
                // 判断滑动方向（水平或垂直）
                if (Math.abs(diffX) > Math.abs(diffY)) {
                    // 水平滑动
                    if (Math.abs(diffX) > 20) { // 最小滑动距离
                        diffX > 0 ? handleMove('right') : handleMove('left');
                    }
                } else {
                    // 垂直滑动
                    if (Math.abs(diffY) > 20) { // 最小滑动距离
                        diffY > 0 ? handleMove('down') : handleMove('up');
                    }
                }
            }
        });

        // 初始化游戏
        initGame();
    </script>
</body>
</html>
