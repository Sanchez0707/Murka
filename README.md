<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Мини-Тетрис 10×10 (С таймером)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <style>
    body {
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 20px;
    }
    #timer {
      font-size: 20px;
      margin-bottom: 10px;
      color: #333;
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
      margin-top: 10px;

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
    .controls button:focus { outline: none; }
    .center { background: none; border: none; }
  </style>
</head>
<body>

  <h2>Мини-Тетрис 10×10</h2>
  <div id="timer">Время: 15</div>
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
    const boardEl   = document.getElementById('board');
    const timerEl   = document.getElementById('timer');
    const grid      = Array.from({ length: boardSize }, () => Array(boardSize).fill(0));
    const shapes    = [
      [[1,1],[1,1]],
      [[1,1,1]],
      [[1],[1],[1],[1]],
      [[0,1,0],[1,1,1]],
      [[1,0],[1,0],[1,1]]
    ];
    let currentFigure, pos, timerId, timeLeft;

    function createBoard() {
      boardEl.innerHTML = '';
      for (let i = 0; i < boardSize * boardSize; i++) {
        const div = document.createElement('div');
        div.classList.add('cell');
        boardEl.appendChild(div);
      }
    }

    function drawBoard() {
      const cells = boardEl.children;
      grid.flat().forEach((v, idx) => {
        cells[idx].className = v ? 'cell filled' : 'cell';
      });
      previewFigure();
    }

    function clamp(v, min, max) {
      return v < min ? min : v > max ? max : v;
    }

    function previewFigure() {
      const cells = boardEl.children;
      let conflict = false;
      currentFigure.forEach((row, dy) => {
        row.forEach((val, dx) => {
          if (!val) return;
          const x = pos.x + dx, y = pos.y + dy;
          if (x<0||x>=boardSize||y<0||y>=boardSize) { conflict = true; return; }
          const idx = y*boardSize + x;
          cells[idx].classList.add(grid[y][x] ? 'red' : 'filled');
          if (grid[y][x]) conflict = true;
        });
      });
      return !conflict;
    }

    function move(dir) {
      let {x,y} = pos;
      if (dir==='left')  x--;
      if (dir==='right') x++;
      if (dir==='up')    y--;
      if (dir==='down')  y++;
      pos.x = clamp(x, 0, boardSize-1);
      pos.y = clamp(y, 0, boardSize-1);
      drawBoard();
    }

    function rotate() {
      const rotated = currentFigure[0].map((_,i)=>currentFigure.map(r=>r[i]).reverse());
      currentFigure = rotated;
      pos.x = clamp(pos.x, 0, boardSize - currentFigure[0].length);
      pos.y = clamp(pos.y, 0, boardSize - currentFigure.length);
      drawBoard();
    }

    function placeFigure() {
      if (!previewFigure()) return;
      currentFigure.forEach((row, dy)=> row.forEach((v,dx)=>{
        if (v) grid[pos.y+dy][pos.x+dx] = 1;
      }));
      clearLines();
      spawnNew();
    }

    function clearLines() {
      for (let y=0;y<boardSize;y++){
        if (grid[y].every(v=>v)){ grid[y].fill(0); }
      }
      for (let x=0;x<boardSize;x++){
        if (grid.every(r=>r[x])) boardEl,grid.forEach(r=>r[x]=0);
      }
    }

    function spawnNew() {
      // очистка таймера
      clearTimeout(timerId);
      timeLeft = 15;
      timerEl.textContent = 'Время: ' + timeLeft;
      // новая фигура
      currentFigure = shapes[Math.floor(Math.random()*shapes.length)]
                       .map(r=>[...r]);
      pos = { x: 3, y: 3 };
      drawBoard();
      // запуск таймера
      timerId = setInterval(()=>{
        timeLeft--;
        timerEl.textContent = 'Время: ' + timeLeft;
        if (timeLeft <= 0) {
          clearInterval(timerId);
          autoPlace();
        }
      }, 1000);
    }

    function autoPlace() {
      // ищем все возможные позиции
      const possible = [];
      for (let y=0;y<boardSize;y++) for(let x=0;x<boardSize;x++){
        pos = {x,y};
        if (previewFigure()) possible.push({x,y});
      }
      if (possible.length) {
        const p = possible[Math.floor(Math.random()*possible.length)];
        pos = p;
        placeFigure();
      } else {
        spawnNew(); // если некуда поставить, просто новая
      }
    }

    // старт
    createBoard();
    spawnNew();
  </script>
</body>
</html>
