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
    }
    #timer {
      font-size: 20px;
      margin-bottom: 10px;
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
    .red    { background:red!important }
    /* Управление */
    #controls {
      position: relative;
      width: 160px;  /* ширина контейнера */
      height: 160px; /* высота контейнера */
      margin: 20px auto 0;
    }
    .btn {
      position: absolute;
      width: 50px; height: 50px;
      font-size: 24px;
      background: #f0f0f0;
      border: none; border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      line-height: 50px;
      user-select: none;
      touch-action: manipulation;
    }
    .btn:active { background: #ccc }
    /* Расположение */
    #btn-up    { top: 0;   left: 55px; }
    #btn-down  { bottom:0; left: 55px; }
    #btn-left  { top: 55px; left: 0;   }
    #btn-right { top: 55px; right:0;   }
    #btn-place { top: 55px; left: 55px; width:50px; height:50px; line-height:50px; }
    #btn-rot   { top: 0;    right:0;   }
  </style>
</head>
<body>

  <h1>Мини‑Тетрис 10×10</h1>
  <div id="timer">Время: <span id="time">15</span></div>
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
          boardEl = document.getElementById('board'),
          timeEl  = document.getElementById('time');
    let grid = Array.from({length:boardSize}, ()=>Array(boardSize).fill(0)),
        figures = [
          [[1,1],[1,1]],
          [[1,1,1]],
          [[1],[1],[1],[1]],
          [[0,1,0],[1,1,1]],
          [[1,0],[1,0],[1,1]]
        ],
        current, pos, timer, interval;

    // Создаём поле
    for(let i=0;i<boardSize*boardSize;i++){
      let c = document.createElement('div');
      c.className='cell';
      boardEl.appendChild(c);
    }
    const cells = boardEl.children;

    function draw(){
      // очистка
      grid.flat().forEach((v,i)=>{
        cells[i].className = v?'cell filled':'cell';
      });
      // превью
      current.forEach((row,y)=>{
        row.forEach((v,x)=>{
          if(v){
            let ix = (pos.y+y)*boardSize + (pos.x+x);
            if(ix>=0 && ix<cells.length){
              cells[ix].classList.add(grid[pos.y+y][pos.x+x]?'red':'filled');
            }
          }
        })
      });
    }
    function clamp(v,m,M){return v<m?m:(v>M?M:v)}

    function move(dir){
      pos = {x: pos.x + (dir=='left'? -1: dir=='right'?1:0),
             y: pos.y + (dir=='up'? -1: dir=='down'?1:0)};
      pos.x = clamp(pos.x,0,boardSize - current[0].length);
      pos.y = clamp(pos.y,0,boardSize - current.length);
      draw();
    }
    function rotate(){
      let R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
      current = R;
      pos.x = clamp(pos.x,0,boardSize - current[0].length);
      pos.y = clamp(pos.y,0,boardSize - current.length);
      draw();
    }
    function placeFigure(){
      // если конфликт, не ставить
      let ok = true;
      current.forEach((row,y)=> row.forEach((v,x)=>{
        if(v && grid[pos.y+y][pos.x+x]) ok=false;
      }));
      if(!ok) return;
      current.forEach((row,y)=> row.forEach((v,x)=>{
        if(v) grid[pos.y+y][pos.x+x]=1;
      }));
      clearLines(); newFigure();
    }
    function clearLines(){
      // горизонтали
      for(let y=0;y<boardSize;y++){
        if(grid[y].every(c=>c)){
          grid[y].fill(0);
        }
      }
      // вертикали
      for(let x=0;x<boardSize;x++){
        if(grid.every(r=>r[x])){
          grid.forEach(r=>r[x]=0);
        }
      }
    }
    function newFigure(){
      clearInterval(interval);
      timer=15; timeEl.textContent=timer;
      current = figures[Math.floor(Math.random()*figures.length)]
        .map(r=>[...r]);
      pos = {x:Math.floor((boardSize-current[0].length)/2), y:Math.floor((boardSize-current.length)/2)};
      draw();
      interval = setInterval(()=>{
        if(--timer<=0){
          clearInterval(interval);
          placeRandom();
        }
        timeEl.textContent=timer;
      },1000);
    }
    function placeRandom(){
      // искать все возможные
      let opts=[];
      for(let y=0;y<=boardSize-current.length;y++){
        for(let x=0;x<=boardSize-current[0].length;x++){
          pos={x,y};
          let clash=false;
          current.forEach((r,dy)=>r.forEach((v,dx)=>{
            if(v && grid[y+dy][x+dx]) clash=true;
          }));
          if(!clash) opts.push({x,y});
        }
      }
      if(opts.length){
        let s=opts[Math.floor(Math.random()*opts.length)];
        pos=s; placeFigure();
      } else newFigure();
    }

    // старт
    newFigure();
  </script>
</body>
</html>
