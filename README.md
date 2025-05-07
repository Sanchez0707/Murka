<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Мини‑Тетрис 10×10</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin: 20px;
      font-family: Arial, sans-serif;
      text-align: center;
      user-select: none;
      touch-action: manipulation;
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
    /* Управление */
    #controls {
      position: relative;
      width: 160px; height: 160px;
      margin: 20px auto 0;
    }
    .btn {
      position: absolute;
      width: 50px; height: 50px;
      font-size: 24px;
      background: #f0f0f0; border:none; border-radius:8px;
      box-shadow:0 2px 5px rgba(0,0,0,0.1);
      line-height:50px;
    }
    .btn:active { background:#ccc }
    /* Расположение */
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
    <button id="btn-reset" onclick="resetGame()">Заново</button>
  </div>

  <div id="board"></div>

  <div id="controls">
    <button id="btn-up"    class="btn" onclick="move('up')">↑</button>
    <button id="btn-down"  class="btn" onclick="move('down')">↓</button>
    <button id="btn-left"  class="btn" onclick="move('left')">←</button>
    <button id="btn-right" class="btn" onclick="move('right')">→</button>
    <button id="btn-place" class="btn" onclick="placeFigure()">•</button>
    <button id="btn-rot"   class="btn" onclick="rotate()">⟳</button>
  </div>

  <script>
    const boardSize = 10,
          boardEl    = document.getElementById('board'),
          timeEl     = document.getElementById('time');

    let grid, figures, current, pos, timer, interval;

    figures = [
      [[1,1],[1,1]],
      [[1,1,1]],
      [[1],[1],[1],[1]],
      [[0,1,0],[1,1,1]],
      [[1,0],[1,0],[1,1]]
    ];

    function initGrid() {
      grid = Array.from({length:boardSize}, ()=>Array(boardSize).fill(0));
      boardEl.innerHTML = '';
      for(let i=0;i<boardSize*boardSize;i++){
        let cell = document.createElement('div');
        cell.className = 'cell';
        boardEl.appendChild(cell);
      }
      resetGame();
    }

    function draw() {
      const cells = boardEl.children;
      // отрисовка занятых
      grid.flat().forEach((v,i)=>{
        cells[i].className = v ? 'cell filled' : 'cell';
      });
      // отрисовка превью только если валидно
      if (canPlaceAt(pos.x, pos.y)) {
        current.forEach((row, y) => {
          row.forEach((v, x) => {
            if (v) {
              let idx = (pos.y + y)*boardSize + (pos.x + x);
              cells[idx].classList.add('filled');
            }
          });
        });
      }
    }

    function clamp(v,min,max) { return v<min?min:(v>max?max:v); }

    function canPlaceAt(px, py) {
      return current.every((row,y) =>
        row.every((v,x)=>
          !v || (px+x>=0 && px+x<boardSize && py+y>=0 && py+y<boardSize && !grid[py+y][px+x])
        )
      );
    }

    function move(dir) {
      let {x,y} = pos;
      if (dir==='left') x--;
      if (dir==='right') x++;
      if (dir==='up') y--;
      if (dir==='down') y++;
      pos.x = clamp(x, 0, boardSize-current[0].length);
      pos.y = clamp(y, 0, boardSize-current.length);
      draw();
    }

    function rotate() {
      let R = current[0].map((_,i)=> current.map(r=>r[i]).reverse());
      current = R;
      pos.x = clamp(pos.x, 0, boardSize-current[0].length);
      pos.y = clamp(pos.y, 0, boardSize-current.length);
      draw();
    }

    function placeFigure() {
      if (!canPlaceAt(pos.x,pos.y)) return;
      current.forEach((row,y)=>
        row.forEach((v,x)=>{ if (v) grid[pos.y+y][pos.x+x]=1; })
      );
      clearLines();
      newFigure();
    }

    function clearLines() {
      for(let y=0;y<boardSize;y++){
        if(grid[y].every(v=>v)) grid[y].fill(0);
      }
      for(let x=0;x<boardSize;x++){
        if(grid.every(r=>r[x])) grid.forEach(r=>r[x]=0);
      }
    }

    function newFigure() {
      clearInterval(interval);
      timer = 15; timeEl.textContent = timer;
      current = figures[Math.floor(Math.random()*figures.length)].map(r=>[...r]);
      pos = {
        x: Math.floor((boardSize - current[0].length)/2),
        y: Math.floor((boardSize - current.length)/2)
      };
      draw();
      interval = setInterval(()=>{
        timerEl();
      },1000);
    }

    function timerEl() {
      if (--timer <= 0) {
        clearInterval(interval);
        autoPlace();
      }
      timeEl.textContent = timer;
    }

    function autoPlace() {
      // ищем валидные
      let opts=[];
      for(let y=0;y<=boardSize-current.length;y++){
        for(let x=0;x<=boardSize-current[0].length;x++){
          if (canPlaceAt(x,y)) opts.push({x,y});
        }
      }
      if(opts.length){
        let p = opts[Math.floor(Math.random()*opts.length)];
        pos = p;
      }
      placeFigure();
    }

    function resetGame() {
      clearInterval(interval);
      initGrid();
    }

    // старт
    initGrid();
  </script>
</body>
</html>
