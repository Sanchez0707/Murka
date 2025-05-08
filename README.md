<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Типо тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin:0; padding:10px;
      font-family: Arial, sans-serif;
      text-align: center;
      user-select: none; touch-action: manipulation;
      transition: background 0.5s, color 0.5s;
    }
    h1 { margin-bottom:5px; }
    #top-bar {
      display:flex; justify-content:space-between; align-items:center;
      flex-wrap:wrap; max-width:480px; margin:auto; padding:0 10px;
    }
    #score-box { font-size:18px; margin:5px; }
    #restart, #theme-btn {
      padding:5px 10px; font-size:14px; cursor:pointer;
      border:none; border-radius:4px; margin:5px;
      transition: background 0.5s, color 0.5s;
    }
    #speed-select {
      padding:4px; font-size:14px; margin:5px;
      border:none; border-radius:4px;
      transition: background 0.5s, color 0.5s;
    }
    #board {
      display:grid;
      grid-template-columns:repeat(10,30px);
      grid-template-rows:repeat(16,30px);
      gap:1px; margin:10px auto;
      transition: background 0.5s;
    }
    .cell {
      width:30px; height:30px;
      background:#eee; position:relative;
      transition: background 0.5s;
    }
    .block {
      position:absolute; width:100%; height:100%;
      transition: background 0.5s;
    }
    .ghost {
      position:absolute; width:100%; height:100%;
      transition: background 0.5s;
    }
    #controls {
      margin-top:15px;
    }
    #controls button {
      width:50px; height:50px; font-size:24px; margin:5px;
      border:none; border-radius:4px; cursor:pointer;
      transition: background 0.5s, color 0.5s;
    }
    footer {
      margin-top:20px; font-size:12px; transition: color 0.5s;
    }
  </style>
</head>
<body>

  <h1>Типо тетрис</h1>
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

  <div id="board"></div>

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
  const scoreBox=document.getElementById('score-box');
  const restartBtn=document.getElementById('restart');
  const speedSelect=document.getElementById('speed-select');
  const themeBtn=document.getElementById('theme-btn');
  const cells=[];

  // Цветовые темы
  const themes = [
    { // классическая
      bodyBg: '#222', text: '#fafafa',
      boardBg: '#444', cellBg: '#eee',
      block: '#333', ghost: 'rgba(0,0,0,0.4)',
      btnBg: '#555', btnText: '#fafafa'
    },
    { // светлая пастель
      bodyBg: '#fdf6e3', text: '#073642',
      boardBg: '#eee8d5', cellBg: '#eee8d5',
      block: '#268bd2', ghost: 'rgba(38,139,210,0.3)',
      btnBg: '#93a1a1', btnText: '#073642'
    },
    { // неоновая
      bodyBg: '#000000', text: '#39ff14',
      boardBg: '#003300', cellBg: '#001a00',
      block: '#39ff14', ghost: 'rgba(57,255,20,0.3)',
      btnBg: '#005500', btnText: '#39ff14'
    },
    { // морская
      bodyBg: '#2e8b57', text: '#e0ffff',
      boardBg: '#3cb371', cellBg: '#66cdaa',
      block: '#20b2aa', ghost: 'rgba(32,178,170,0.3)',
      btnBg: '#2e8b57', btnText: '#e0ffff'
    },
    { // закат
      bodyBg: '#ff7f50', text: '#4b0082',
      boardBg: '#cd5c5c', cellBg: '#f08080',
      block: '#800080', ghost: 'rgba(128,0,128,0.3)',
      btnBg: '#cd5c5c', btnText: '#4b0082'
    }
  ];
  let themeIndex=0;

  function applyTheme(i) {
    const t = themes[i];
    document.body.style.background = t.bodyBg;
    document.body.style.color = t.text;
    boardEl.style.background = t.boardBg;
    cells.forEach(c=> c.style.background = t.cellBg);
    document.querySelectorAll('.block').forEach(b=> b.style.background = t.block);
    document.querySelectorAll('.ghost').forEach(g=> g.style.background = t.ghost);
    document.querySelectorAll('button, select').forEach(e=>{
      e.style.background = t.btnBg;
      e.style.color = t.btnText;
    });
  }

  themeBtn.addEventListener('click', ()=>{
    themeIndex = (themeIndex + 1) % themes.length;
    applyTheme(themeIndex);
  });

  // --- Игра (без изменений) ---
  let grid, current, pos, score=0, intervalId;

  const SHAPES=[
    [[1,1,1,1]], [[1,1],[1,1]],
    [[0,1,0],[1,1,1]], [[1,0,0],[1,1,1]],
    [[1,1,1]], [[1,1]], [[1,0],[1,1]]
  ];

  // создаём клетки
  for(let i=0;i<COLS*ROWS;i++){
    const c=document.createElement('div');
    c.className='cell';
    boardEl.appendChild(c);
    cells.push(c);
  }

  function resetGame(){
    clearInterval(intervalId);
    grid = Array.from({length:ROWS}, ()=>Array(COLS).fill(0));
    score = 0; scoreBox.textContent = 'Очки: 0';
    spawnFigure(); draw();
    intervalId = setInterval(gameTick, parseInt(speedSelect.value));
  }

  function spawnFigure(){
    const s=SHAPES[Math.floor(Math.random()*SHAPES.length)];
    current=s.map(r=>[...r]);
    pos={ x:Math.floor((COLS-current[0].length)/2), y:0 };
    if(collide(pos.x,pos.y)){ clearInterval(intervalId); alert('Игра окончена!'); }
  }

  function collide(px,py, fig=current){
    for(let y=0;y<fig.length;y++)
     for(let x=0;x<fig[0].length;x++)
      if(fig[y][x]){
        const nx=px+x, ny=py+y;
        if(nx<0||nx>=COLS||ny>=ROWS||grid[ny][nx]) return true;
      }
    return false;
  }

  function draw(){
    grid.flat().forEach((v,i)=>cells[i].innerHTML='');
    grid.forEach((row,y)=>row.forEach((v,x)=>{
      if(v) cells[y*COLS+x].innerHTML='<div class="block"></div>';
    }));
    let gy=pos.y;
    while(!collide(pos.x,gy+1)) gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(gy+ry)*COLS+pos.x+rx].innerHTML='<div class="ghost"></div>';
    }));
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(pos.y+ry)*COLS+pos.x+rx].innerHTML='<div class="block"></div>';
    }));
    applyTheme(themeIndex); // обновить цвета блоков/тени
  }

  function clearLines(){
    let lines=0;
    for(let y=ROWS-1;y>=0;y--){
      if(grid[y].every(c=>c)){
        grid.splice(y,1); grid.unshift(Array(COLS).fill(0));
        lines++; y++;
      }
    }
    if(lines){
      score+=lines*10; scoreBox.textContent='Очки: '+score;
    }
  }

  function gameTick(){
    if(!collide(pos.x,pos.y+1)) pos.y++;
    else{
      current.forEach((r,ry)=>r.forEach((v,rx)=>{
        if(v) grid[pos.y+ry][pos.x+rx]=1;
      }));
      clearLines(); spawnFigure();
    }
    draw();
  }

  function moveLeft(){ if(!collide(pos.x-1,pos.y)) pos.x--; draw();}
  function moveRight(){ if(!collide(pos.x+1,pos.y)) pos.x++; draw();}
  function rotate(){ 
    const R=current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if(!collide(pos.x,pos.y,R)) current=R; draw();
  }
  function softDrop(){ for(let i=0;i<3;i++) if(!collide(pos.x,pos.y+1)) pos.y++; draw(); }

  restartBtn.addEventListener('click', resetGame);
  speedSelect.addEventListener('change', ()=>{
    clearInterval(intervalId);
    intervalId=setInterval(gameTick, parseInt(speedSelect.value));
  });

  // старт
  resetGame();
  applyTheme(themeIndex);
</script>

</body>
</html>
