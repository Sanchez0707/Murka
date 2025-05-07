<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Мини-Тетрис 10×10 (Без зума)</title>
  <!-- Отключаем масштабирование на мобильных -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <style>
    body {
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 20px;
    }

    #board {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(10, 30px);
      gap: 1px;
      margin-bottom: 15px;
    }
    .cell {
      width: 30px;
      height: 30px;
      background: #eee;
      border: 1px solid #ccc;
    }
    .filled { background: #444; }
    .red    { background: red !important; }

    .controls {
      display: grid;
      grid-template-columns: 30px 30px 30px;
      grid-template-rows: repeat(3, 30px);
      gap: 5px;
      margin-bottom: 10px;

      /* Отключаем лишние эффекты при касании */
      touch-action: manipulation;
      -webkit-tap-highlight-color: transparent;
    }
    .controls button {
      padding: 0;
      font-size: 18px;
      cursor: pointer;

      user-select: none;
      touch-action: manipulation;
      -webkit-tap-highlight-color: transparent;
    }
    .controls button:focus {
      outline: none;
    }
    .center {
      background: none;
      border: none;
    }
  </style>
</head>
<body>

  <h2>Мини-Тетрис 10×10</h2>
  <div id="board"></div>

  <div class="controls">
    <button onclick="move('up')">↑</button>
    <div class="center"></div>
    <button onclick="rotate()">⟳</button>

    <button onclick="move('left')">←</button>
    <button onclick="placeFigure()">•</button>
    <button onclick="move('right')">→</button>

    <button class="center"></button>
    <button onclick="move('down')">↓</button>
    <button class="center"></button>
  </div>

  <script>
    const boardSize = 10;
    const boardEl = document.getElementById('board');
    const grid = Array.from({ length: boardSize }, () => Array(boardSize).fill(0));
    let currentFigure;
    let pos = { x: 3, y: 3 };  // стартовая позиция (центр)

    const shapes = [
      [[1,1],[1,1]],               // квадрат
      [[1,1,1]],                   // палка 3 гор.
      [[1],[1],[1],[1]],           // палка 4 верт.
      [[0,1,0],[1,1,1]],           // крест
      [[1,0],[1,0],[1,1]]          // L-образная
    ];

    function createBoard() {
      for (let i = 0; i < boardSize * boardSize; i++) {
        const div = document.createElement('div');
        div.classList.add('cell');
        boardEl.appendChild(div);
      }
    }

    function drawBoard() {
      const cells = boardEl.children;
      grid.flat().forEach((val, idx) => {
        cells[idx].className = val ? 'cell filled' : 'cell';
      });
      previewFigure();
    }

    function clamp(val, min, max) {
      return Math.max(min, Math.min(max, val));
    }

    function previewFigure() {
      const cells = boardEl.children;
      const { x, y } = pos;
      let conflict = false;

      currentFigure.forEach((row, dy) => {
        row.forEach((val, dx) => {
          if (!val) return;
          const nx = x + dx;
          const ny = y + dy;
          if (nx < 0 || nx >= boardSize || ny < 0 || ny >= boardSize) {
            conflict = true;
            return;
          }
          const idx = ny * boardSize + nx;
          cells[idx].classList.add(grid[ny][nx] ? 'red' : 'filled');
          if (grid[ny][nx]) conflict = true;
        });
      });

      return !conflict;
    }

    function move(dir) {
      let { x, y } = pos;
      if (dir === 'left') x--;
      if (dir === 'right') x++;
      if (dir === 'up') y--;
      if (dir === 'down') y++;
      pos.x = clamp(x, 0, boardSize - 1);
      pos.y = clamp(y, 0, boardSize - 1);
      drawBoard();
    }

    function rotate() {
      const rotated = currentFigure[0].map((_, i) =>
        currentFigure.map(r => r[i]).reverse()
      );
      currentFigure = rotated;
      pos.x = clamp(pos.x, 0, boardSize - currentFigure[0].length);
      pos.y = clamp(pos.y, 0, boardSize - currentFigure.length);
      drawBoard();
    }

    function placeFigure() {
      if (!previewFigure()) return;
      currentFigure.forEach((row, dy) => {
        row.forEach((val, dx) => {
          if (val) grid[pos.y + dy][pos.x + dx] = 1;
        });
      });
      clearLines();
      spawnNew();
      drawBoard();
    }

    function clearLines() {
      for (let y = 0; y < boardSize; y++) {
        if (grid[y].every(v => v)) {
          grid[y].fill(0);
        }
      }
      for (let x = 0; x < boardSize; x++) {
        if (grid.every(row => row[x])) {
          grid.forEach(row => row[x] = 0);
        }
      }
    }

    function spawnNew() {
      currentFigure = shapes[Math.floor(Math.random() * shapes.length)]
        .map(row => [...row]);
      pos = { x: 3, y: 3 };
    }

    createBoard();
    spawnNew();
    drawBoard();
  </script>
</body>
</html>
