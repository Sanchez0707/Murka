<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Мини-Тетрис</title>
  <style>
    body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; }
    #board { display: grid; grid-template-columns: repeat(10, 30px); grid-template-rows: repeat(10, 30px); gap: 1px; margin: 20px 0; }
    .cell { width: 30px; height: 30px; background: #eee; border: 1px solid #ccc; }
    .filled { background: #444; }
    .preview { display: flex; gap: 10px; }
    .figure { display: grid; grid-template-columns: repeat(4, 20px); grid-template-rows: repeat(4, 20px); }
    .block { width: 20px; height: 20px; background: #00f; }
    .red { background: red !important; }
    .controls { margin-top: 10px; display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; }
    button { padding: 8px 16px; }
  </style>
</head>
<body>
  <h2>Мини-Тетрис 10x10</h2>
  <div id="board"></div>
  <div class="preview" id="figures"></div>
  <div class="controls">
    <button onclick="move('up')">↑</button>
    <button onclick="move('left')">←</button>
    <button onclick="move('down')">↓</button>
    <button onclick="move('right')">→</button>
    <button onclick="rotate()">Повернуть</button>
    <button onclick="placeFigure()">Поставить</button>
  </div>

  <script>
    const boardSize = 10;
    const board = [];
    const boardEl = document.getElementById('board');
    const figuresEl = document.getElementById('figures');
    let currentFigure = null;
    let figures = [];
    let pos = { x: 0, y: 0 };

    const shapes = [
      [[1, 1], [1, 1]], // Квадрат
      [[1, 1, 1]],      // Палка 3 (гор)
      [[1], [1], [1], [1]], // Палка 4 (верт)
      [[0, 1, 0], [1, 1, 1]], // Крест
      [[1, 0], [1, 0], [1, 1]] // L-образная
    ];

    function createBoard() {
      for (let i = 0; i < boardSize * boardSize; i++) {
        const cell = document.createElement('div');
        cell.classList.add('cell');
        board.push(cell);
        boardEl.appendChild(cell);
      }
    }

    function drawBoard() {
      board.forEach((cell, i) => {
        const x = i % boardSize;
        const y = Math.floor(i / boardSize);
        cell.className = 'cell';
        if (grid[y][x]) cell.classList.add('filled');
      });
    }

    function generateFigures() {
      figures = [];
      figuresEl.innerHTML = '';
      for (let i = 0; i < 3; i++) {
        const shape = JSON.parse(JSON.stringify(shapes[Math.floor(Math.random() * shapes.length)]));
        figures.push(shape);
        const preview = document.createElement('div');
        preview.classList.add('figure');
        shape.flat().forEach(val => {
          const b = document.createElement('div');
          b.className = 'cell';
          if (val) b.classList.add('block');
          preview.appendChild(b);
        });
        figuresEl.appendChild(preview);
      }
      selectFigure(0);
    }

    function selectFigure(i) {
      currentFigure = figures[i];
      pos = { x: 0, y: 0 };
      drawBoard();
      previewFigure();
    }

    function rotate() {
      const newFig = currentFigure[0].map((_, i) => currentFigure.map(r => r[i]).reverse());
      currentFigure = newFig;
      previewFigure();
    }

    function move(dir) {
      const delta = { left: -1, right: 1, up: -1, down: 1 };
      if (dir === 'left' || dir === 'right') pos.x += delta[dir];
      if (dir === 'up' || dir === 'down') pos.y += delta[dir];
      previewFigure();
    }

    const grid = Array(boardSize).fill(0).map(() => Array(boardSize).fill(0));

    function previewFigure() {
      drawBoard();
      let conflict = false;
      currentFigure.forEach((row, dy) => {
        row.forEach((val, dx) => {
          const x = pos.x + dx;
          const y = pos.y + dy;
          if (val && y >= 0 && y < boardSize && x >= 0 && x < boardSize) {
            const index = y * boardSize + x;
            board[index].classList.add(grid[y][x] ? 'red' : 'filled');
            if (grid[y][x]) conflict = true;
          }
        });
      });
    }

    function placeFigure() {
      let valid = true;
      currentFigure.forEach((row, dy) => {
        row.forEach((val, dx) => {
          const x = pos.x + dx;
          const y = pos.y + dy;
          if (val && (x < 0 || x >= boardSize || y < 0 || y >= boardSize || grid[y][x])) {
            valid = false;
          }
        });
      });
      if (!valid) return;

      currentFigure.forEach((row, dy) => {
        row.forEach((val, dx) => {
          const x = pos.x + dx;
          const y = pos.y + dy;
          if (val) grid[y][x] = 1;
        });
      });

      clearLines();
      generateFigures();
      drawBoard();
    }

    function clearLines() {
      for (let y = 0; y < boardSize; y++) {
        if (grid[y].every(cell => cell === 1)) {
          grid[y] = Array(boardSize).fill(0);
        }
      }

      for (let x = 0; x < boardSize; x++) {
        let col = grid.map(row => row[x]);
        if (col.every(val => val === 1)) {
          for (let y = 0; y < boardSize; y++) {
            grid[y][x] = 0;
          }
        }
      }
    }

    createBoard();
    generateFigures();
    drawBoard();
  </script>
</body>
</html>
