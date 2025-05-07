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
      touch-action: manipulation;
      user-select: none;
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

    #controls {
      position: relative;
      width: 160px; height: 160px;
      margin: 30px auto 0;
    }
    .btn {
      position: absolute;
      width: 50px; height: 50px;
      font-size: 24px;
      background: #f0f0f0;
      border: none; border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      line-height: 50px;
    }
    .btn:active { background: #ccc }
    #btn-up    { top: 0;    left: 55px; }
    #btn-down  { bottom:0; left: 55px; }
    #btn-left  { top: 55px; left: 0;    }
    #btn-right { top: 55px; right:0;    }
    #btn-place { top: 55px; left: 55px; }
    #btn-rot   { top: 0;    right:0;    }

    #top-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      max-width: 360px;
      margin: 10px auto;
    }
    #score, #timer {
      font-size: 18px;
    }
    #restart {
      padding: 5px 10px;
      font-size: 14px;
      cursor: pointer;
    }
  </style>
</head>
<body>

<h1>Мини‑Тетрис 10×10</h1>
<div id="top-bar">
  <div id="score">Очки: 0</div>
  <button id="restart">Заново</button>
  <div id="timer">Время: <span id="time">15</span></div>
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
  const boardSize = 10;
  const boardEl    = document.getElementById('board');
  const timeEl     = document.getElementById('time');
  const scoreEl    = document.getElementById('score');
  const restartBtn = document.getElementById('restart');

  let grid, current, pos, timer, interval, score = 0;
  const figures = [
    [[1,1],[1,1]],
    [[1,1,1]],
    [[1],[1],[1],[1]],
    [[0,1,0],[1,1,1]],
    [[1,0],[1,0],[1,1]]
  ];

  // Построение поля
  for (let i=0; i<boardSize*boardSize; i++){
    const cell = document.createElement('div');
    cell.className = 'cell';
    boardEl.appendChild(cell);
  }
  const cells = boardEl.children;

  function clamp(v,min,max){ return v<min?min:(v>max?max:v); }

  // Проверка коллизии для фигуры fig на позиции p
  function canPlaceAt(p, fig = current) {
    for (let y=0; y<fig.length; y++){
      for (let x=0; x<fig[0].length; x++){
        if (fig[y][x]) {
          const px = p.x + x, py = p.y + y;
          if (px<0||px>=boardSize||py<0||py>=boardSize||grid[py][px]) {
            return false;
          }
        }
      }
    }
    return true;
  }

  // Отрисовка сетки и превью
  function draw(){
    grid.flat().forEach((v,i)=>{
      cells[i].className = v ? 'cell filled' : 'cell';
    });
    // рисуем превью только если валидно
    if (canPlaceAt(pos, current)) {
      current.forEach((row,y)=>{
        row.forEach((v,x)=>{
          if (v){
            const idx = (pos.y+y)*boardSize + (pos.x+x);
            cells[idx].classList.add('filled');
          }
        });
      });
    }
  }

  // Перемещение
  function move(dir){
    const delta = { left:{x:-1,y:0}, right:{x:1,y:0}, up:{x:0,y:-1}, down:{x:0,y:1} }[dir];
    const newPos = { x: clamp(pos.x+delta.x, 0, boardSize-current[0].length),
                     y: clamp(pos.y+delta.y, 0, boardSize-current.length) };
    if (canPlaceAt(newPos)) {
      pos = newPos;
      draw();
    }
  }

  // Поворот с wall-kick
  function rotate(){
    const R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    for (let dx of [0,-1,1]){
      for (let dy of [0,-1,1]){
        const test = { x: clamp(pos.x+dx, 0, boardSize-R[0].length),
                       y: clamp(pos.y+dy, 0, boardSize-R.length) };
        if (canPlaceAt(test, R)) {
          current = R;
          pos = test;
          draw();
          return;
        }
      }
    }
  }

  // Установка фигуры
  function placeFigure(){
    if (!canPlaceAt(pos)) return;
    current.forEach((row,y)=>row.forEach((v,x)=>{
      if (v) grid[pos.y+y][pos.x+x] = 1;
    }));
    clearLines();
    newFigure();
  }

  // Очистка заполненных линий
  function clearLines(){
    for (let y=0; y<boardSize; y++){
      if (grid[y].every(v=>v)){
        grid[y].fill(0);
        score += 10;
      }
    }
    for (let x=0; x<boardSize; x++){
      if (grid.every(r=>r[x])){
        grid.forEach(r=>r[x]=0);
        score += 10;
      }
    }
    scoreEl.textContent = `Очки: ${score}`;
  }

  // Новый спавн фигуры (поиск любой валидной позиции)
  function newFigure(){
    clearInterval(interval);
    timer = 15;
    timeEl.textContent = timer;
    current = figures[Math.floor(Math.random()*figures.length)].map(r=>[...r]);

    let found=false;
    for (let y=0; y<=boardSize-current.length; y++){
      for (let x=0; x<=boardSize-current[0].length; x++){
        const p={x,y};
        if (canPlaceAt(p)){
          pos = p; found=true; break;
        }
      }
      if(found) break;
    }
    if (!found){
      alert("Игра окончена!");
      startGame();
      return;
    }

    draw();
    interval = setInterval(()=>{
      timer--;
      timeEl.textContent = timer;
      if (timer<=0){
        clearInterval(interval);
        autoPlace();
      }
    },1000);
  }

  // Автопостановка
  function autoPlace(){
    for (let y=0; y<=boardSize-current.length; y++){
      for (let x=0; x<=boardSize-current[0].length; x++){
        const p={x,y};
        if (canPlaceAt(p)){
          pos = p;
          placeFigure();
          return;
        }
      }
    }
    newFigure();
  }

  // Сброс игры
  function startGame(){
    clearInterval(interval);
    grid = Array.from({length:boardSize}, ()=>Array(boardSize).fill(0));
    score = 0; scoreEl.textContent = `Очки: ${score}`;
    newFigure();
  }

  // Настройка кнопок
  ['up','down','left','right','place','rot'].forEach(key=>{
    const id = key==='place'?'btn-place': key==='rot'?'btn-rot': 'btn-'+key;
    document.getElementById(id)
      .addEventListener('mousedown', ()=> key==='place'?placeFigure(): key==='rot'?rotate(): move(key));
  });
  restartBtn.addEventListener('click', startGame);

  // Старт
  startGame();
</script>

</body>
</html>
