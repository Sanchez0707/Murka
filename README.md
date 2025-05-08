<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Типо тетрис</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"/>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0; padding: 10px;
      background: #222; color: #fafafa;
      user-select: none; touch-action: manipulation;
    }
    h1 { margin-bottom: 5px; }
    #top-bar {
      display: flex; justify-content: space-between; align-items: center;
      max-width: 480px; margin: auto; padding: 0 10px;
    }
    #score-box { font-size: 18px; }
    #restart, #music-toggle {
      padding: 5px 10px; font-size: 14px; cursor: pointer;
      background: #555; color: #fafafa; border: none; border-radius: 4px;
    }
    #speed-select {
      padding: 4px; font-size: 14px;
      background: #333; color: #fafafa; border: none; border-radius: 4px;
    }
    #game-container {
      display: flex; justify-content: center; align-items: flex-start;
      gap: 20px; flex-wrap: wrap; margin-top: 10px;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(16, 30px);
      gap: 1px; background: #444;
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
    #controls {
      margin-top: 15px;
    }
    #controls button {
      width:50px; height:50px; font-size:24px; margin:5px;
      background:#555; color:#fafafa; border:none; border-radius:4px;
      cursor:pointer;
    }
    #controls button:active { background:#777 }
    footer {
      margin-top:20px; font-size:12px; color:#888;
    }
  </style>
</head>
<body>

  <h1>Типо тетрис</h1>
  <div id="top-bar">
    <button id="restart">Заново</button>
    <div>
      Скорость:
      <select id="speed-select">
        <option value="1000">1.00×</option>
        <option value="1333">0.75×</option>
        <option value="2000">0.50×</option>
        <option value="800">1.25×</option>
        <option value="666">1.50×</option>
      </select>
    </div>
    <div id="score-box">Очки: 0</div>
    <button id="music-toggle">Музыка</button>
  </div>

  <div id="game-container">
    <div id="board"></div>
  </div>

  <div id="controls">
    <button onclick="moveLeft()">←</button>
    <button onclick="moveRight()">→</button>
    <button onclick="rotate()">↻</button>
    <button onclick="softDrop()">↓</button>
  </div>

  <footer>Я не пытаюсь кого-либо плагиатить</footer>

  <audio id="bgm" loop>
    <source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_4ac093ee98.mp3?filename=relaxing-ambient-12692.mp3" type="audio/mpeg">
  </audio>

<script>
  const COLS=10, ROWS=16;
  const boardEl=document.getElementById('board');
  const scoreBox=document.getElementById('score-box');
  const restartBtn=document.getElementById('restart');
  const speedSelect=document.getElementById('speed-select');
  const musicToggle=document.getElementById('music-toggle');
  const bgm=document.getElementById('bgm');

  let grid, current, pos, score=0, intervalId;

  const SHAPES = [
    [[1,1,1,1]],
    [[1,1],[1,1]],
    [[0,1,0],[1,1,1]],
    [[1,0,0],[1,1,1]],
    [[1,1,1]],
    [[1,1]],
    [[1,0],[1,1]]
  ];

  // Создаём поле
  for(let i=0;i<COLS*ROWS;i++){
    const c=document.createElement('div');
    c.className='cell';
    boardEl.appendChild(c);
  }
  const cells=boardEl.children;

  function resetGame(){
    clearInterval(intervalId);
    grid=Array.from({length:ROWS},()=>Array(COLS).fill(0));
    score=0; scoreBox.textContent='Очки: 0';
    spawnFigure(); draw();
    intervalId=setInterval(gameTick, parseInt(speedSelect.value));
  }

  function spawnFigure(){
    const s=SHAPES[Math.floor(Math.random()*SHAPES.length)];
    current=s.map(r=>[...r]);
    pos={ x: Math.floor((COLS-current[0].length)/2), y:0 };
    if(collide(pos.x,pos.y)){ clearInterval(intervalId); alert('Игра окончена!'); }
  }

  function collide(px,py,fig=current){
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
    grid.flat().forEach((v,i)=>cells[i].innerHTML='');
    // занятые
    grid.forEach((row,y)=>row.forEach((v,x)=>{
      if(v) cells[y*COLS+x].innerHTML='<div class="block"></div>';
    }));
    // ghost
    let gy=pos.y;
    while(!collide(pos.x,gy+1)) gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(gy+ry)*COLS+pos.x+rx].innerHTML='<div class="ghost"></div>';
    }));
    // текущая
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(pos.y+ry)*COLS+pos.x+rx].innerHTML='<div class="block"></div>';
    }));
  }

  function clearLines(){
    let lines=0;
    for(let y=ROWS-1;y>=0;y--){
      if(grid[y].every(c=>c)){
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

  function gameTick(){
    if(!collide(pos.x,pos.y+1)) pos.y++;
    else {
      current.forEach((r,ry)=>r.forEach((v,rx)=>{
        if(v) grid[pos.y+ry][pos.x+rx]=1;
      }));
      clearLines(); spawnFigure();
    }
    draw();
  }

  function moveLeft(){ if(!collide(pos.x-1,pos.y)) pos.x--; draw(); }
  function moveRight(){ if(!collide(pos.x+1,pos.y)) pos.x++; draw(); }
  function rotate(){
    const R=current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if(!collide(pos.x,pos.y,R)) current=R;
    draw();
  }
  function softDrop(){ for(let i=0;i<3;i++){ if(!collide(pos.x,pos.y+1)) pos.y++; } draw(); }

  restartBtn.onclick=e=>{ e.preventDefault(); resetGame(); };
  speedSelect.onchange=e=>{
    clearInterval(intervalId);
    intervalId=setInterval(gameTick, parseInt(speedSelect.value));
  };
  musicToggle.onclick=e=>{
    e.preventDefault();
    bgm.paused?bgm.play():bgm.pause();
  };

  // старт
  resetGame();
  bgm.play();
</script>

</body>
</html>
