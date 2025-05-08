<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Типо тетрис</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0;
      padding: 10px;
      background: #f0f0f0;
    }
    h1 {
      margin-bottom: 5px;
    }
    #game-container {
      display: flex;
      justify-content: center;
      align-items: flex-start;
      gap: 20px;
      flex-wrap: wrap;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(16, 30px);
      gap: 1px;
      background: #ccc;
    }
    .cell {
      width: 30px;
      height: 30px;
      background: #eee;
    }
    .filled {
      background: #444;
    }
    .ghost {
      background: rgba(0, 0, 0, 0.2);
    }
    #sidebar {
      text-align: left;
      min-width: 150px;
    }
    #score {
      font-size: 18px;
      margin-bottom: 10px;
    }
    #controls {
      margin-top: 10px;
    }
    .btn {
      font-size: 20px;
      padding: 8px 12px;
      margin: 4px;
      border: none;
      border-radius: 5px;
      background: #ddd;
      cursor: pointer;
    }
    .btn:active {
      background: #bbb;
    }
    #footer {
      margin-top: 20px;
      font-size: 12px;
      color: #555;
    }
  </style>
</head>
<body>

  <h1>Типо тетрис</h1>
  <button class="btn" onclick="restartGame()">Заново</button>

  <div id="game-container">
    <div id="board"></div>
    <div id="sidebar">
      <div id="score">Очки: 0</div>
      <div>
        <label>Скорость:
          <select id="speedSelect" onchange="changeSpeed()">
            <option value="1000">1.00x</option>
            <option value="1333">0.75x</option>
            <option value="2000">0.50x</option>
            <option value="800">1.25x</option>
            <option value="666">1.50x</option>
          </select>
        </label>
      </div>
      <div>
        <button class="btn" onclick="toggleMusic()">Музыка: Вкл/Выкл</button>
        <audio id="bgm" loop>
          <source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_4ac093ee98.mp3" type="audio/mpeg">
        </audio>
      </div>
    </div>
  </div>

  <div id="controls">
    <button class="btn" onclick="moveLeft()">←</button>
    <button class="btn" onclick="moveRight()">→</button>
    <button class="btn" onclick="rotate()">↻</button>
    <button class="btn" onclick="softDrop()">↓</button>
  </div>

  <div id="footer">я не пытаюсь кого либо плагиатить</div>

  <script>
    const width = 10, height = 16;
    const boardEl = document.getElementById('board');
    const scoreEl = document.getElementById('score');
    const speedSelect = document.getElementById('speedSelect');
    const bgm = document.getElementById('bgm');

    let board = [], current, pos, interval, score = 0, dropDelay = 1000;

    const figures = [
      [[1,1,1,1]], // палка
      [[1,1],[1,1]], // квадрат
      [[0,1,0],[1,1,1]], // крест
      [[1,0,0],[1,1,1]], // L
      [[1,1,1]], // новая 3х1
      [[1,1]],   // новая 2х1
      [[1,0],[1,1]] // маленькая L
    ];

    function createBoard() {
      board = Array.from({ length: height }, () => Array(width).fill(0));
      boardEl.innerHTML = '';
      for (let i = 0; i < width * height; i++) {
        const cell = document.createElement('div');
        cell.className = 'cell';
        boardEl.appendChild(cell);
      }
    }

    function draw() {
      const cells = boardEl.children;
      for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
          const cell = cells[y * width + x];
          cell.className = 'cell';
          if (board[y][x]) cell.classList.add('filled');
        }
      }

      // тень
      let ghostY = pos.y;
      while (!hasCollision(pos.x, ghostY + 1, current)) ghostY++;
      for (let y = 0; y < current.length; y++) {
        for (let x = 0; x < current[0].length; x++) {
          if (current[y][x] && ghostY + y >= 0) {
            const cell = cells[(ghostY + y) * width + (pos.x + x)];
            if (cell) cell.classList.add('ghost');
          }
        }
      }

      // текущая фигура
      for (let y = 0; y < current.length; y++) {
        for (let x = 0; x < current[0].length; x++) {
          if (current[y][x] && pos.y + y >= 0) {
            const cell = cells[(pos.y + y) * width + (pos.x + x)];
            if (cell) cell.classList.add('filled');
          }
        }
      }
    }

    function hasCollision(x, y, fig) {
      for (let i = 0; i < fig.length; i++) {
        for (let j = 0; j < fig[0].length; j++) {
          if (fig[i][j]) {
            const nx = x + j, ny = y + i;
            if (nx < 0 || nx >= width || ny >= height || (ny >= 0 && board[ny][nx])) {
              return true;
            }
          }
        }
      }
      return false;
    }

    function mergeFigure() {
      for (let y = 0; y < current.length; y++) {
        for (let x = 0; x < current[0].length; x++) {
          if (current[y][x] && pos.y + y >= 0) {
            board[pos.y + y][pos.x + x] = 1;
          }
        }
      }
    }

    function clearLines() {
      let lines = 0;
      for (let y = height - 1; y >= 0; y--) {
        if (board[y].every(cell => cell)) {
          board.splice(y, 1);
          board.unshift(Array(width).fill(0));
          lines++;
        }
      }
      score += lines * 10;
      scoreEl.textContent = "Очки: " + score;
    }

    function spawnFigure() {
      current = JSON.parse(JSON.stringify(figures[Math.floor(Math.random() * figures.length)]));
      pos = { x: Math.floor((width - current[0].length) / 2), y: -1 };
      if (hasCollision(pos.x, pos.y + 1, current)) {
        alert("Игра окончена!");
        restartGame();
      }
    }

    function gameTick() {
      if (!hasCollision(pos.x, pos.y + 1, current)) {
        pos.y++;
      } else {
        mergeFigure();
        clearLines();
        spawnFigure();
      }
      draw();
    }

    function rotate() {
      const rotated = current[0].map((_, i) => current.map(r => r[i]).reverse());
      if (!hasCollision(pos.x, pos.y, rotated)) current = rotated;
      draw();
    }

    function moveLeft() {
      if (!hasCollision(pos.x - 1, pos.y, current)) pos.x--;
      draw();
    }

    function moveRight() {
      if (!hasCollision(pos.x + 1, pos.y, current)) pos.x++;
      draw();
    }

    function softDrop() {
      let steps = 3;
      while (steps-- && !hasCollision(pos.x, pos.y + 1, current)) {
        pos.y++;
      }
      draw();
    }

    function toggleMusic() {
      if (bgm.paused) bgm.play();
      else bgm.pause();
    }

    function changeSpeed() {
      dropDelay = parseInt(speedSelect.value);
      clearInterval(interval);
      interval = setInterval(gameTick, dropDelay);
    }

    function restartGame() {
      clearInterval(interval);
      score = 0;
      createBoard();
      spawnFigure();
      interval = setInterval(gameTick, dropDelay);
      draw();
    }

    // Инициализация
    createBoard();
    restartGame();
    bgm.play();
  </script>
</body>
</html>
