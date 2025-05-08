<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Типо тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
  <style>
    body { margin:0; padding:10px; background:#222; color:#fafafa;
      font-family:Arial,sans-serif; text-align:center;
      user-select:none; touch-action:manipulation;
      transition: background .5s, color .5s;
    }
    h1 { margin-bottom:5px; }
    #top-bar {
      display:flex; justify-content:space-between; align-items:center;
      flex-wrap:wrap; max-width:500px; margin:auto; padding:0 10px;
    }
    #score-box, #lines-box { font-size:18px; margin:5px; }
    button, select { margin:5px; border:none; border-radius:4px; cursor:pointer; transition: background .5s, color .5s; }
    #restart, #theme-btn, #music-toggle {
      padding:5px 10px; font-size:14px; background:#555; color:#fafafa;
    }
    #speed-select { padding:4px; font-size:14px; background:#333; color:#fafafa; }
    #game-container {
      display:flex; justify-content:center; align-items:flex-start;
      gap:20px; flex-wrap:wrap; margin-top:10px;
    }
    #board {
      display:grid;
      grid-template-columns:repeat(10,30px);
      grid-template-rows:repeat(16,30px);
      gap:1px; background:#444;
      transition: background .5s;
    }
    .cell {
      width:30px; height:30px; background:#eee; position:relative;
      transition: background .5s;
    }
    .block { position:absolute; width:100%; height:100%; background:#333; transition: background .5s; }
    .ghost { position:absolute; width:100%; height:100%; background:rgba(0,0,0,0.4); transition: background .5s; }

    /* Превью next */
    #sidebar { text-align:left; min-width:120px; }
    #next-title { margin-top:10px; }
    #next-grid { display:grid; grid-template:repeat(4,20px)/repeat(4,20px); gap:1px; margin:auto; }
    .next-cell { width:20px; height:20px; background:#eee; }
    .next-block { width:20px; height:20px; background:#333; position:absolute; }

    #controls { margin-top:15px; }
    #controls button { width:50px; height:50px; font-size:24px; background:#555; color:#fafafa; }
    #controls button:active { background:#777; }
    footer { margin-top:20px; font-size:12px; color:#888; transition: color .5s; }
  </style>
</head>
<body>

  <h1>Типо тетрис</h1>

  <div id="top-bar">
    <button id="restart">Заново</button>
    <div id="score-box">Очки: 0</div>
    <div id="lines-box">Линий: 0</div>
    <select id="speed-select">
      <option value="1000">1.00×</option>
      <option value="1333">0.75×</option>
      <option value="2000">0.50×</option>
      <option value="800">1.25×</option>
      <option value="666">1.50×</option>
    </select>
    <button id="theme-btn">Тема</button>
  </div>

  <div id="game-container">
    <div id="board"></div>
    <div id="sidebar">
      <div id="next-title">Следующая:</div>
      <div id="next-grid"></div>
    </div>
  </div>

  <div id="controls">
    <button onclick="moveLeft()">←</button>
    <button onclick="moveRight()">→</button>
    <button onclick="rotate()">↻</button>
    <button onclick="softDrop()">↓</button>
  </div>

  <footer>Я не пытаюсь кого-либо плагиатить</footer>

<script>
  const COLS=10, ROWS=16;
  const boardEl=document.getElementById('board');
  const nextGrid=document.getElementById('next-grid');
  const scoreBox=document.getElementById('score-box');
  const linesBox=document.getElementById('lines-box');
  const restartBtn=document.getElementById('restart');
  const speedSelect=document.getElementById('speed-select');
  const themeBtn=document.getElementById('theme-btn');

  let grid, current, next, pos, score=0, lines=0, intervalId, themeIndex=0;

  // Цветовые темы (как до этого)
  const themes = [ /* ... тот же массив themes ... */ ];
  function applyTheme(i){ /* ... тот же applyTheme ... */ }

  // Фигуры
  const SHAPES=[
    [[1,1,1,1]], [[1,1],[1,1]],
    [[0,1,0],[1,1,1]], [[1,0,0],[1,1,1]],
    [[1,1,1]], [[1,1]], [[1,0],[1,1]]
  ];

  // Построение полей
  for(let i=0;i<COLS*ROWS;i++){
    const c=document.createElement('div'); c.className='cell';
    boardEl.appendChild(c);
  }
  for(let i=0;i<16;i++){ // 4×4 grid for next
    const c=document.createElement('div'); c.className='next-cell';
    nextGrid.appendChild(c);
  }
  const cells=boardEl.children, nextCells=nextGrid.children;

  function resetGame(){
    clearInterval(intervalId);
    grid=Array.from({length:ROWS},()=>Array(COLS).fill(0));
    score=0; lines=0;
    scoreBox.textContent='Очки: 0';
    linesBox.textContent='Линий: 0';
    spawn();
    draw(); drawNext();
    intervalId=setInterval(gameTick, parseInt(speedSelect.value));
  }

  function spawn(){
    current = next || SHAPES[Math.floor(Math.random()*SHAPES.length)];
    next = SHAPES[Math.floor(Math.random()*SHAPES.length)];
    current = current.map(r=>[...r]);
    pos={ x:Math.floor((COLS-current[0].length)/2), y:0 };
    if(collide(pos.x,pos.y)){ clearInterval(intervalId); alert('Игра окончена!'); }
    drawNext();
  }

  function collide(px,py,fig=current){
    for(let y=0;y<fig.length;y++)for(let x=0;x<fig[0].length;x++){
      if(fig[y][x]){
        const nx=px+x, ny=py+y;
        if(nx<0||nx>=COLS||ny>=ROWS||grid[ny][nx])return true;
      }
    }
    return false;
  }

  function draw(){
    grid.flat().forEach((v,i)=>cells[i].innerHTML='');
    grid.forEach((row,y)=>row.forEach((v,x)=>{
      if(v) cells[y*COLS+x].innerHTML='<div class="block"></div>';
    }));
    let gy=pos.y; while(!collide(pos.x,gy+1))gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) nextCells?0:0; // noop
      cells[(gy+ry)*COLS+pos.x+rx].innerHTML='<div class="ghost"></div>';
    }));
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(pos.y+ry)*COLS+pos.x+rx].innerHTML='<div class="block"></div>';
    }));
    applyTheme(themeIndex);
  }

  function drawNext(){
    nextCells.forEach(c=>c.innerHTML='');
    for(let y=0;y<next.length;y++)for(let x=0;x<next[0].length;x++){
      if(next[y][x]){
        nextCells[y*4+x].innerHTML='<div class="block"></div>';
      }
    }
  }

  function clearLines(){
    let cnt=0;
    for(let y=ROWS-1;y>=0;y--){
      if(grid[y].every(c=>c)){
        grid.splice(y,1); grid.unshift(Array(COLS).fill(0));
        cnt++; y++;
      }
    }
    if(cnt){
      lines+=cnt; score+=cnt*10;
      linesBox.textContent='Линий: '+lines;
      scoreBox.textContent='Очки: '+score;
    }
  }

  function gameTick(){
    if(!collide(pos.x,pos.y+1))pos.y++;
    else{ current.forEach((r,ry)=>r.forEach((v,rx)=>{
            if(v)grid[pos.y+ry][pos.x+rx]=1;
          }));
          clearLines(); spawn();
    }
    draw();
  }

  function moveLeft(){ if(!collide(pos.x-1,pos.y))pos.x--; draw(); }
  function moveRight(){ if(!collide(pos.x+1,pos.y))pos.x++; draw(); }
  function rotate(){
    const R=current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if(!collide(pos.x,pos.y,R)) current=R; draw();
  }
  function softDrop(){for(let i=0;i<3;i++)if(!collide(pos.x,pos.y+1))pos.y++; draw();}

  restartBtn.addEventListener('click',resetGame);
  speedSelect.addEventListener('change',()=>{
    clearInterval(intervalId);
    intervalId=setInterval(gameTick,parseInt(speedSelect.value));
  });
  themeBtn.addEventListener('click',()=>{
    themeIndex=(themeIndex+1)%themes.length;
    applyTheme(themeIndex);
    draw(); drawNext();
  });

  // старт
  resetGame();
  applyTheme(themeIndex);
</script>
</body>
</html>
