<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin: 0; padding: 0;
      font-family: Arial, sans-serif;
      text-align: center;
      user-select: none; touch-action: manipulation;
      background: #222; color: #fafafa;
    }
    h1 { margin: 10px 0; }
    #top-bar {
      display: flex; justify-content: space-between; align-items: center;
      max-width: 400px; margin: auto; padding: 0 10px;
    }
    #score-box { font-size: 18px; }
    #restart { padding: 5px 10px; font-size: 14px; cursor: pointer; }
    #speed-select {
      padding: 4px; font-size: 14px;
    }
    #game-container {
      position: relative;
      width: calc(10 * 30px + 9px);
      margin: 10px auto;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(10,30px);
      grid-template-rows: repeat(20,30px);
      gap: 1px; background: #444;
    }
    .cell {
      width: 30px; height: 30px;
      background: #eee; border:1px solid #ccc;
      box-sizing: border-box; position: relative;
    }
    .block {
      position: absolute; width:100%; height:100%;
      background: #333;
    }
    .ghost {
      position: absolute; width:100%; height:100%;
      background: rgba(50,50,50,0.6);
    }
    #controls {
      display: flex; justify-content: center; align-items: center;
      gap: 10px; margin: 15px 0;
    }
    #controls button {
      width: 50px; height: 50px; font-size: 24px;
      cursor: pointer; border:none; border-radius:4px;
      background: #555; color: #fafafa;
    }
    #controls button:active { background: #777; }
    footer { margin: 20px 0; font-size:12px; color:#888; }
  </style>
</head>
<body>

  <h1>Тетрис</h1>
  <div id="top-bar">
    <button id="restart">Заново</button>
    <div>
      Скорость: 
      <select id="speed-select">
        <option value="0.5">0.50×</option>
        <option value="0.75">0.75×</option>
        <option value="1" selected>1×</option>
        <option value="1.25">1.25×</option>
        <option value="1.5">1.50×</option>
      </select>
    </div>
    <div id="score-box">Очки: 0</div>
  </div>

  <div id="game-container">
    <div id="board"></div>
  </div>

  <div id="controls">
    <button id="btn-rot">↻</button>
    <button id="btn-left">←</button>
    <button id="btn-right">→</button>
    <button id="btn-drop">↓</button>
  </div>

  <footer>Я не пытаюсь кого-либо плагиатить</footer>

<script>
  const COLS = 10, ROWS = 20;
  let speed = 1, dropIntervalId, score=0;
  const boardEl = document.getElementById('board');
  const scoreBox = document.getElementById('score-box');
  const restartBtn = document.getElementById('restart');
  const speedSelect = document.getElementById('speed-select');

  // Фигуры: I, O, T, L
  const SHAPES = [
    [[1,1,1,1]],
    [[1,1],[1,1]],
    [[0,1,0],[1,1,1]],
    [[1,0,0],[1,1,1]]
  ];

  let grid, current, pos;

  // Построение поля
  for (let i=0;i<COLS*ROWS;i++){
    const c=document.createElement('div');
    c.className='cell';
    boardEl.appendChild(c);
  }
  const cells = boardEl.children;

  function resetGame(){
    clearInterval(dropIntervalId);
    grid = Array.from({length:ROWS},()=>Array(COLS).fill(0));
    score=0; scoreBox.textContent='Очки: 0';
    spawn();
    draw();
    dropIntervalId = setInterval(drop, 500/speed);
  }

  function spawn(){
    const s=SHAPES[Math.floor(Math.random()*SHAPES.length)];
    current = s.map(r=>[...r]);
    pos = {x: Math.floor((COLS-current[0].length)/2), y:0};
    if(collide(pos.x,pos.y)){
      clearInterval(dropIntervalId);
      alert('Игра окончена!');
    }
  }

  function collide(px,py, fig=current){
    for(let y=0;y<fig.length;y++){
      for(let x=0;x<fig[0].length;x++){
        if(fig[y][x]){
          const nx=px+x, ny=py+y;
          if(nx<0||nx>=COLS||ny>=ROWS||grid[ny][nx]) return true;
        }
      }
    }
    return false;
  }

  function draw(){
    // очистка
    grid.flat().forEach((v,i)=>{
      cells[i].innerHTML='';
    });
    // занятые
    grid.forEach((row,y)=>row.forEach((v,x)=>{
      if(v) cells[y*COLS+x].innerHTML='<div class="block"></div>';
    }));
    // ghost
    let gy=pos.y;
    while(!collide(pos.x,gy+1)) gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v){
        cells[(gy+ry)*COLS + (pos.x+rx)].innerHTML='<div class="ghost"></div>';
      }
    }));
    // current
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v){
        cells[(pos.y+ry)*COLS + (pos.x+rx)].innerHTML='<div class="block"></div>';
      }
    }));
  }

  function clearLines(){
    let lines=0;
    for(let y=ROWS-1;y>=0;y--){
      if(grid[y].every(v=>v)){
        grid.splice(y,1);
        grid.unshift(Array(COLS).fill(0));
        lines++; y++;
      }
    }
    if(lines){
      score+=lines*10;
      scoreBox.textContent='Очки: '+score;
    }
  }

  function drop(){
    if(!collide(pos.x,pos.y+1)) pos.y++;
    else {
      // зафиксировать
      current.forEach((r,ry)=>r.forEach((v,rx)=>{
        if(v) grid[pos.y+ry][pos.x+rx]=1;
      }));
      clearLines();
      spawn();
    }
    draw();
  }

  // Управления
  document.getElementById('btn-left').addEventListener('mousedown',e=>{
    e.preventDefault();
    if(!collide(pos.x-1,pos.y)) pos.x--;
    draw();
  });
  document.getElementById('btn-right').addEventListener('mousedown',e=>{
    e.preventDefault();
    if(!collide(pos.x+1,pos.y)) pos.x++;
    draw();
  });
  document.getElementById('btn-rot').addEventListener('mousedown',e=>{
    e.preventDefault();
    const R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if(!collide(pos.x,pos.y,R)) current=R;
    draw();
  });
  document.getElementById('btn-drop').addEventListener('mousedown',e=>{
    e.preventDefault();
    // падение на 3 клетки
    for(let i=0;i<3;i++){
      if(!collide(pos.x,pos.y+1)) pos.y++;
    }
    draw();
  });
  restartBtn.addEventListener('mousedown',e=>{
    e.preventDefault(); resetGame();
  });
  speedSelect.addEventListener('change',()=>{
    speed = parseFloat(speedSelect.value);
    clearInterval(dropIntervalId);
    dropIntervalId = setInterval(drop, 500/speed);
  });

  // Старт
  resetGame();
</script>

</body>
</html>
