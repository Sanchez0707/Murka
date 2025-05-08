<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Типо Тетрис — Murka</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no"/>
  <link rel="icon" href="https://sanchez0707.github.io/Murka/favicon.ico"/>
  <style>
    body {
      margin:0; padding:10px;
      background:#222; color:#fafafa;
      font-family:Arial, sans-serif;
      text-align:center;
      user-select:none; touch-action:manipulation;
    }
    h1 { margin:5px 0 10px; }
    #top-bar {
      display:flex; justify-content:space-between; align-items:center;
      flex-wrap:wrap; max-width:600px; margin:auto;
    }
    button, select {
      margin:5px; padding:5px 10px; font-size:14px;
      background:#555; color:#fafafa; border:none; border-radius:4px;
      cursor:pointer; transition:background .3s;
    }
    button:active, select:active { background:#777 }
    #score-box { font-size:18px; margin:5px; }
    #game-area {
      display:flex; justify-content:center; align-items:flex-start;
      gap:20px; flex-wrap:wrap; margin-top:10px;
    }
    #board {
      display:grid;
      grid-template-columns:repeat(10,30px);
      grid-template-rows:repeat(16,30px);
      gap:1px; background:#444;
      width:calc(10*30px + 9px);
    }
    .cell {
      width:30px; height:30px; background:#eee; position:relative;
    }
    .block {
      position:absolute; width:100%; height:100%; background:#333;
    }
    .ghost {
      position:absolute; width:100%; height:100%; background:rgba(0,0,0,0.4);
    }
    #sidebar {
      min-width:150px; text-align:left;
    }
    #sidebar h3 { margin:0 0 10px; }
    #sidebar button { width:100%; }
    #controls {
      margin-top:15px;
    }
    #controls button {
      width:50px; height:50px; font-size:24px; margin:5px;
    }
    footer { margin-top:20px; font-size:12px; color:#888; }
  </style>
</head>
<body>

  <h1>Типо Тетрис — Murka</h1>
  <div id="top-bar">
    <button id="restart">Заново</button>
    <div id="score-box">Очки: 0</div>
    <select id="speed-select">
      <option value="1000">1.00×</option>
      <option value="1333">0.75×</option>
      <option value="2000">0.50×</option>
      <option value="800">1.25×</option>
      <option value="666">1.50×</option>
    </select>
    <button id="theme-btn">Тема</button>
  </div>

  <div id="game-area">
    <div id="board"></div>
    <div id="sidebar">
      <h3>Функции</h3>
      <button id="hint-btn">Показать подсказку</button>
      <button id="save-btn">Сохранить рекорд</button>
      <button id="share-btn">Поделиться ссылкой</button>
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
  // Основные константы
  const COLS=10, ROWS=16;
  const boardEl=document.getElementById('board');
  const scoreBox=document.getElementById('score-box');
  const restartBtn=document.getElementById('restart');
  const speedSelect=document.getElementById('speed-select');
  const themeBtn=document.getElementById('theme-btn');
  const hintBtn=document.getElementById('hint-btn');
  const saveBtn=document.getElementById('save-btn');
  const shareBtn=document.getElementById('share-btn');

  // Темы оформления
  const themes=[
    {bg:'#222', color:'#fafafa', board:'#444', cell:'#eee', block:'#333', ghost:'rgba(0,0,0,0.4)', btn:'#555'},
    {bg:'#fdf6e3', color:'#073642', board:'#eee8d5', cell:'#eee8d5', block:'#268bd2', ghost:'rgba(38,139,210,0.3)', btn:'#93a1a1'},
    {bg:'#000', color:'#39ff14', board:'#003300', cell:'#001a00', block:'#39ff14', ghost:'rgba(57,255,20,0.3)', btn:'#005500'},
    {bg:'#2e8b57', color:'#e0ffff', board:'#3cb371', cell:'#66cdaa', block:'#20b2aa', ghost:'rgba(32,178,170,0.3)', btn:'#2e8b57'},
    {bg:'#ff7f50', color:'#4b0082', board:'#cd5c5c', cell:'#f08080', block:'#800080', ghost:'rgba(128,0,128,0.3)', btn:'#cd5c5c'}
  ];
  let themeIndex=0;

  // Поле, фигуры и логика
  let grid, current, pos, score=0, intervalId;
  const SHAPES=[
    [[1,1,1,1]], [[1,1],[1,1]],
    [[0,1,0],[1,1,1]], [[1,0,0],[1,1,1]],
    [[1,1,1]], [[1,1]], [[1,0],[1,1]]
  ];

  // Создаём клетки один раз
  for(let i=0;i<COLS*ROWS;i++){
    const c=document.createElement('div');
    c.className='cell';
    boardEl.appendChild(c);
  }
  const cells=boardEl.children;

  function applyTheme(){
    const t=themes[themeIndex];
    document.body.style.background=t.bg;
    document.body.style.color=t.color;
    boardEl.style.background=t.board;
    Array.from(cells).forEach(c=>c.style.background=t.cell);
    document.querySelectorAll('.block').forEach(b=>b.style.background=t.block);
    document.querySelectorAll('.ghost').forEach(g=>g.style.background=t.ghost);
    document.querySelectorAll('button, select').forEach(e=>e.style.background=t.btn);
  }

  function resetGame(){
    clearInterval(intervalId);
    grid=Array.from({length:ROWS},()=>Array(COLS).fill(0));
    score=0; scoreBox.textContent='Очки: 0';
    spawnFigure(); draw();
    intervalId=setInterval(gameTick,parseInt(speedSelect.value));
  }

  function spawnFigure(){
    const s=SHAPES[Math.floor(Math.random()*SHAPES.length)];
    current=s.map(r=>[...r]);
    pos={x:Math.floor((COLS-current[0].length)/2),y:0};
    if(collide(pos.x,pos.y)){ clearInterval(intervalId); alert('Игра окончена!'); }
  }

  function collide(px,py,fig=current){
    for(let y=0;y<fig.length;y++)for(let x=0;x<fig[0].length;x++)if(fig[y][x]){
      const nx=px+x,ny=py+y;
      if(nx<0||nx>=COLS||ny>=ROWS||grid[ny][nx])return true;
    }
    return false;
  }

  function draw(){
    // очистка
    grid.flat().forEach((v,i)=>cells[i].innerHTML='');
    grid.forEach((row,y)=>row.forEach((v,x)=>{ if(v)cells[y*COLS+x].innerHTML='<div class="block"></div>'; }));
    let gy=pos.y; while(!collide(pos.x,gy+1))gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{ if(v)cells[(gy+ry)*COLS+pos.x+rx].innerHTML='<div class="ghost"></div>'; }));
    current.forEach((r,ry)=>r.forEach((v,rx)=>{ if(v)cells[(pos.y+ry)*COLS+pos.x+rx].innerHTML='<div class="block"></div>'; }));
    applyTheme();
  }

  function clearLines(){
    let lines=0;
    for(let y=ROWS-1;y>=0;y--){
      if(grid[y].every(c=>c)){ grid.splice(y,1);grid.unshift(Array(COLS).fill(0));lines++;y++; }
    }
    if(lines){ score+=lines*10; scoreBox.textContent='Очки: '+score; }
  }

  function gameTick(){
    if(!collide(pos.x,pos.y+1))pos.y++;
    else{ current.forEach((r,ry)=>r.forEach((v,rx)=>{ if(v)grid[pos.y+ry][pos.x+rx]=1; })); clearLines(); spawnFigure();}
    draw();
  }

  function moveLeft(){ if(!collide(pos.x-1,pos.y))pos.x--; draw(); }
  function moveRight(){ if(!collide(pos.x+1,pos.y))pos.x++; draw(); }
  function rotate(){ const R=current[0].map((_,i)=>current.map(r=>r[i]).reverse()); if(!collide(pos.x,pos.y,R))current=R; draw(); }
  function softDrop(){ for(let i=0;i<3;i++)if(!collide(pos.x,pos.y+1))pos.y++; draw(); }

  // Разные функции справа
  hintBtn.onclick = ()=> alert('Подсказка: ищи места для длинных фигур!');
  saveBtn.onclick = ()=> alert('Рекорд сохранён: '+score);
  shareBtn.onclick = ()=> prompt('Ссылка на игру:', location.href);

  // События
  restartBtn.onclick = resetGame;
  speedSelect.onchange = ()=>{ clearInterval(intervalId); intervalId=setInterval(gameTick,parseInt(speedSelect.value)); };
  themeBtn.onclick = ()=>{ themeIndex=(themeIndex+1)%themes.length; draw(); };

  // Старт
  resetGame();
</script>

</body>
</html>
