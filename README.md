<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Фолл‑Тетрис 10×10</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin: 20px;
      font-family: sans-serif;
      text-align: center;
      user-select: none;
    }
    #top-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      max-width: 380px;
      margin: auto 0 10px;
    }
    #score-box {
      width: 60px; height: 50px;
      border: 1px solid #333;
      display: flex; align-items: center; justify-content: center;
      font-size: 18px;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(10,30px);
      grid-template-rows: repeat(10,30px);
      gap: 1px;
      margin: auto;
      width: max-content;
      background: #555;
    }
    .cell {
      width: 30px; height: 30px;
      background: #eee; border:1px solid #ccc;
      box-sizing: border-box;
      position: relative;
    }
    .block {
      position: absolute;
      width: 100%; height: 100%;
      background: #444;
    }
    .ghost {
      position: absolute;
      width: 100%; height: 100%;
      background: rgba(100,100,100,0.3);
    }
    #controls {
      margin-top: 15px;
    }
    #controls button {
      width: 50px; height: 50px;
      font-size: 24px; margin: 5px;
      cursor: pointer;
    }
  </style>
</head>
<body>

  <h1>Фолл‑Тетрис 10×10</h1>
  <div id="top-bar">
    <div id="score-box">0</div>
    <button onclick="startGame()">Заново</button>
  </div>

  <div id="board"></div>

  <div id="controls">
    <button id="up">/\\</button>
    <button id="left">&lt;</button>
    <button id="down">\\/</button>
    <button id="right">&gt;</button>
  </div>

<script>
  const cols = 10, rows = 10, boardEl = document.getElementById('board'),
        scoreBox = document.getElementById('score-box');
  const shapes = [
    [[1,1,1,1]],               // I
    [[1,1],[1,1]],             // O
    [[0,1,0],[1,1,1]],         // T
    [[1,0,0],[1,1,1]],         // J
    [[0,0,1],[1,1,1]],         // L
    [[1,1,0],[0,1,1]],         // S
    [[0,1,1],[1,1,0]]          // Z
  ];
  let grid, current, pos, dropInterval, dropSpeed=500, score;

  // Создаем клетки
  for(let i=0;i<cols*rows;i++){
    const c=document.createElement('div');
    c.className='cell';
    boardEl.appendChild(c);
  }
  const cells = boardEl.children;

  function resetGrid(){
    grid=Array(rows).fill().map(()=>Array(cols).fill(0));
  }

  function drawGrid(){
    for(let y=0;y<rows;y++){
      for(let x=0;x<cols;x++){
        const idx=y*cols+x;
        cells[idx].innerHTML = grid[y][x]?'<div class="block"></div>':'';
      }
    }
  }

  function drawCurrent(){
    // ghost (призрак)
    let ghostY=pos.y;
    while(!collide(current, pos.x, ghostY+1)) ghostY++;
    current.forEach((r,dy)=>{
      r.forEach((v,dx)=>{
        if(v){
          const idx=(ghostY+dy)*cols + (pos.x+dx);
          cells[idx].innerHTML = '<div class="ghost"></div>';
        }
      });
    });
    // current
    current.forEach((r,dy)=>{
      r.forEach((v,dx)=>{
        if(v){
          const idx=(pos.y+dy)*cols + (pos.x+dx);
          cells[idx].innerHTML = '<div class="block"></div>';
        }
      });
    });
  }

  function collide(shape, x, y){
    for(let dy=0;dy<shape.length;dy++){
      for(let dx=0;dx<shape[0].length;dx++){
        if(shape[dy][dx]){
          const ny=y+dy, nx=x+dx;
          if(nx<0||nx>=cols||ny>=rows||grid[ny][nx]) return true;
        }
      }
    }
    return false;
  }

  function rotateShape(){
    const R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if(!collide(R,pos.x,pos.y)) current=R;
  }

  function place(){
    current.forEach((r,dy)=>r.forEach((v,dx)=>{
      if(v) grid[pos.y+dy][pos.x+dx]=1;
    }));
    clearLines();
    spawn();
  }

  function clearLines(){
    let lines=0;
    for(let y=rows-1;y>=0;y--){
      if(grid[y].every(v=>v)){
        grid.splice(y,1);
        grid.unshift(Array(cols).fill(0));
        lines++;
        y++;
      }
    }
    if(lines){
      score+=lines*10;
      scoreBox.textContent=score;
    }
  }

  function spawn(){
    const s=shapes[Math.floor(Math.random()*shapes.length)];
    current=s.map(r=>[...r]);
    pos={x:Math.floor((cols-current[0].length)/2), y:0};
    if(collide(current,pos.x,pos.y)){
      // game over
      clearInterval(dropInterval);
      alert('Игра окончена!');
    }
  }

  function drop(){
    if(!collide(current,pos.x,pos.y+1)){
      pos.y++;
    } else {
      place();
    }
    render();
  }

  function render(){
    drawGrid();
    drawCurrent();
  }

  function startGame(){
    clearInterval(dropInterval);
    score=0; scoreBox.textContent=score;
    resetGrid();
    spawn();
    render();
    dropInterval = setInterval(drop, dropSpeed);
  }

  // Управление
  document.getElementById('left').onclick = ()=>{ if(!collide(current,pos.x-1,pos.y)) pos.x--; render(); };
  document.getElementById('right').onclick = ()=>{ if(!collide(current,pos.x+1,pos.y)) pos.x++; render(); };
  document.getElementById('down').onclick = ()=>{ drop(); };
  document.getElementById('up').onclick = ()=>{ rotateShape(); render(); };

  // Запуск
  startGame();
</script>
</body>
</html>
