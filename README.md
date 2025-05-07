<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0">
  <title>Мини-Тетрис</title>
  <style>
    body {
      margin: 20px;
      font-family: Arial, sans-serif;
      text-align: center;
      -webkit-user-select: none;
      user-select: none;
    }
    #header {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 20px;
      margin-bottom: 10px;
    }
    #timer { font-size: 20px; }
    #btn-reset {
      padding: 5px 10px;
      font-size: 16px;
      cursor: pointer;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(10,30px);
      grid-template-rows: repeat(10,30px);
      gap: 1px;
      margin: auto;
      width: max-content;
    }
    .cell {
      width: 30px; height: 30px;
      background: #eee; border:1px solid #ccc;
    }
    .filled { background:#444 }

    #controls {
      position: relative;
      width: 160px; height: 160px;
      margin: 20px auto 0;
      touch-action: none;
    }
    .btn {
      position: absolute;
      width: 50px; height: 50px;
      font-size: 24px;
      background: #f0f0f0; border:none; border-radius:8px;
      box-shadow:0 2px 5px rgba(0,0,0,0.1);
      line-height:50px;
      touch-action: manipulation;
    }
    .btn:active { background:#ccc }

    #btn-up    { top: 0;    left: 55px; }
    #btn-down  { bottom: 0; left: 55px; }
    #btn-left  { top: 55px; left: 0;    }
    #btn-right { top: 55px; right: 0;   }
    #btn-place { top: 55px; left: 55px; }
    #btn-rot   { top: 0;    right: 0;   }
  </style>
</head>
<body>

  <div id="header">
    <div id="timer">Время: <span id="time">15</span></div>
    <button id="btn-reset">Заново</button>
  </div>

  <div id="board"></div>

  <div id="controls">
    <button id="btn-up"    class="btn">↑</button>
    <button id="btn-down"  class="btn">↓</button>
    <button id="btn-left"  class="btn">←</button>
    <button id="btn-right" class="btn">→</button>
    <button id="btn-place" class="btn">•</button>
    <button id="btn-rot"   class="btn">⟳</button>
  </div>

  <script>
    const boardSize = 10,
          boardEl = document.getElementById('board'),
          timeSpan = document.getElementById('time'),
          resetBtn = document.getElementById('btn-reset');

    let grid, figures, current, pos, timer, interval;

    figures = [
      [[1,1],[1,1]],
      [[1,1,1]],
      [[1],[1],[1],[1]],
      [[0,1,0],[1,1,1]],
      [[1,0],[1,0],[1,1]]
    ];

    function createBoard() {
      boardEl.innerHTML = '';
      for (let i = 0; i < boardSize * boardSize; i++) {
        const cell = document.createElement('div');
        cell.className = 'cell';
        boardEl.appendChild(cell);
      }
    }

    function initGrid() {
      grid = Array.from({ length: boardSize }, () => Array(boardSize).fill(0));
    }

    function draw() {
      const cells = boardEl.children;
      grid.flat().forEach((v, i) => {
        cells[i].className = v ? 'cell filled' : 'cell';
      });

      if (canPlaceAt(pos.x, pos.y)) {
        current.forEach((row, y) =>
          row.forEach((v, x) => {
            if (v) {
              const i = (pos.y + y) * boardSize + (pos.x + x);
              if (cells[i]) cells[i].classList.add('filled');
            }
          })
        );
      }
    }

    function clamp(v, min, max) {
      return v < min ? min : v > max ? max : v;
    }

    function canPlaceAt(px, py) {
      return current.every((row, y) =>
        row.every((v, x) =>
          !v ||
          (px + x >= 0 && px + x < boardSize &&
           py + y >= 0 && py + y < boardSize &&
           !grid[py + y][px + x])
        )
      );
    }

    function move(dir) {
      let x = pos.x, y = pos.y;
      if (dir === 'left')  x--;
      if (dir === 'right') x++;
      if (dir === 'up')    y--;
      if (dir === 'down')  y++;

      if (canPlaceAt(x, y)) {
        pos.x = x;
        pos.y = y;
        draw();
      }
    }

    function rotate() {
      const rotated = current[0].map((_, i) =>
        current.map(row => row[i]).reverse()
      );
      const newX = clamp(pos.x, 0, boardSize - rotated[0].length);
      const newY = clamp(pos.y, 0, boardSize - rotated.length);
      if (canPlaceAt(newX, newY)) {
        current = rotated;
        pos.x = newX;
        pos.y = newY;
        draw();
      }
    }

    function placeFigure() {
      if (!canPlaceAt(pos.x, pos.y)) return;
      current.forEach((row, y) =>
        row.forEach((v, x) => {
          if (v) grid[pos.y + y][pos.x + x] = 1;
        })
      );
      clearLines();
      newFigure();
    }

    function clearLines() {
      for (let y = 0; y < boardSize; y++) {
        if (grid[y].every(v => v)) grid[y].fill(0);
      }
      for (let x = 0; x < boardSize; x++) {
        if (grid.every(row => row[x])) grid.forEach(row => row[x] = 0);
      }
    }

    function newFigure() {
      clearInterval(interval);
      timer = 15;
      timeSpan.textContent = timer;
      current = figures[Math.floor(Math.random() * figures.length)].map(r => [...r]);
      pos = {
        x: Math.floor((boardSize - current[0].length) / 2),
        y: Math.floor((boardSize - current.length) / 2)
      };
      draw();
      interval = setInterval(() => {
        timer--;
        timeSpan.textContent = timer;
        if (timer <= 0) {
          clearInterval(interval);
          autoPlace();
        }
      }, 1000);
    }

    function autoPlace() {
      const opts = [];
      for (let y = 0; y <= boardSize - current.length; y++) {
        for (let x = 0; x <= boardSize - current[0].length; x++) {
          if (canPlaceAt(x, y)) opts.push({ x, y });
        }
      }
      if (opts.length) {
        const choice = opts[Math.floor(Math.random() * opts.length)];
        pos = choice;
        placeFigure();
      } else {
        newFigure();
      }
    }

    function resetGame() {
      clearInterval(interval);
      initGrid();
      newFigure();
      draw();
    }

    function setupControls() {
      const actions = {
        'btn-up': () => move('up'),
        'btn-down': () => move('down'),
        'btn-left': () => move('left'),
        'btn-right': () => move('right'),
        'btn-place': () => placeFigure(),
        'btn-rot': () => rotate()
      };
      for (let id in actions) {
        const el = document.getElementById(id);
        el.addEventListener('mousedown', e => {
          e.preventDefault();
          actions[id]();
        });
        el.addEventListener('touchstart', e => {
          e.preventDefault();
          actions[id]();
        });
      }
    }

    // Инициализация
    createBoard();
    setupControls();
    resetBtn.addEventListener('click', resetGame);
    resetGame();
  </script>

</body>
</html>
