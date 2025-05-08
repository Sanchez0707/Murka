<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>♻ Типо тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin: 0; padding: 10px;
      font-family: Arial, sans-serif;
      background: #222; color: #fafafa;
      text-align: center;
      user-select: none; touch-action: manipulation;
      transition: background 0.3s, color 0.3s;
    }
    h1 { margin: 5px 0; }
    #controls {
      margin: 10px 0;
    }
    #controls button {
      font-size: 24px;
      padding: 8px 12px;
      margin: 0 5px;
      background: #555; color: #fafafa;
      border: none; border-radius: 4px;
      cursor: pointer;
      transition: background 0.3s;
    }
    #controls button:active { background: #777; }
    #score {
      font-size: 18px; margin: 10px 0;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(10,30px);
      grid-template-rows: repeat(16,30px);
      gap: 1px;
      margin: auto;
      background: #444;
      transition: background 0.3s;
    }
    .cell {
      width:30px; height:30px;
      background:#eee;
      position: relative;
      transition: background 0.3s;
    }
    .block {
      position:absolute; width:100%; height:100%;
      background:#333;
      transition: background 0.3s;
    }
    .ghost {
      position:absolute; width:100%; height:100%;
      background:rgba(0,0,0,0.4);
      transition: background 0.3s;
    }
    /* светлая тема */
    .light {
      background: #fafafa !important;
      color: #222 !important;
    }
    .light #board { background: #ccc !important; }
    .light .cell { background: #ddd !important; }
    .light .block { background: #444 !important; }
    .light .ghost { background: rgba(68,68,68,0.4) !important; }
    .light #controls button { background: #ccc !important; color: #222 !important; }
  </style>
</head>
<body>

  <h1>♻ Типо тетрис</h1>
  <div id="score">Очки: 0</div>
  <div id="controls">
    <button onclick="toggleTheme()">🌓 Тема</button>
    <button onclick="startGame()">↺</button>
    <button onclick="moveLeft()">←</button>
    <button onclick="moveRight()">→</button>
    <button onclick="rotate()">↻</button>
    <button onclick="softDrop()">↓</button>
  </div>
  <div id="board"></div>

<script>
  const COLS=10, ROWS=16;
  const board=document.getElementById('board');
  const scoreEl=document.getElementById('score');
  let grid, current, pos, score=0, timer;

  // все фигуры: 4 базовых + их зеркала + три доп + их зеркала
  const SHAPES = [
    [[1,1,1,1]], // I
    [[1,1],[1,1]], // O
    [[0,1,0],[1,1,1]], // T
    [[1,0,0],[1,1,1]], // L
    // зеркала базовых
    [[1],[1],[1],[1]],
    [[1,1],[1,1]], // O зеркалится само
    [[1,1,1],[0,1,0]],
    [[0,0,1],[1,1,1]],
    // доп
    [[1,1,1]], // 3
    [[1,1]], // 2
    [[1,0],[1,1]], // мал L
    // зеркала доп
    [[1,1,1]], 
    [[1,1]],
    [[0,1],[1,1]]
  ];

  // создаём клетки
  for(let i=0;i<COLS*ROWS;i++){
    const c=document.createElement('div');
    c.className='cell';
    board.appendChild(c);
  }
  const cells=board.children;

  function startGame(){
    clearInterval(timer);
    grid=Array.from({length:ROWS},()=>Array(COLS).fill(0));
    score=0; scoreEl.textContent='Очки: 0';
    spawn(); draw();
    timer=setInterval(tick,500);
  }

  function spawn(){
    current=JSON.parse(JSON.stringify(
      SHAPES[Math.floor(Math.random()*SHAPES.length)]
    ));
    pos={ x: Math.floor((COLS-current[0].length)/2), y:0 };
    if(collision(pos.x,pos.y)) {
      clearInterval(timer);
      alert('Игра окончена!');
    }
  }

  function collision(px,py, fig=current){
    for(let y=0;y<fig.length;y++)
      for(let x=0;x<fig[0].length;x++)
        if(fig[y][x]){
          const nx=px+x, ny=py+y;
          if(nx<0||nx>=COLS||ny>=ROWS||grid[ny][nx]) return true;
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
    while(!collision(pos.x,gy+1)) gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(gy+ry)*COLS+pos.x+rx].innerHTML='<div class="ghost"></div>';
    }));
    // текущая
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(pos.y+ry)*COLS+pos.x+rx].innerHTML='<div class="block"></div>';
    }));
  }

  function clearLines(){
    let cnt=0;
    for(let y=ROWS-1;y>=0;y--){
      if(grid[y].every(c=>c)){
        grid.splice(y,1);
        grid.unshift(Array(COLS).fill(0));
        cnt++; y++;
      }
    }
    if(cnt){
      let add = cnt===1?10: cnt===2?50: cnt===3?200: cnt*200;
      score+=add;
      scoreEl.textContent='Очки: '+score;
    }
  }

  function tick(){
    if(!collision(pos.x,pos.y+1)) pos.y++;
    else {
      current.forEach((r,ry)=>r.forEach((v,rx)=>{
        if(v) grid[pos.y+ry][pos.x+rx]=1;
      }));
      clearLines(); spawn();
    }
    draw();
  }

  function moveLeft(){ if(!collision(pos.x-1,pos.y)) pos.x--; draw(); }
  function moveRight(){ if(!collision(pos.x+1,pos.y)) pos.x++; draw(); }
  function rotate(){
    const R=current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if(!collision(pos.x,pos.y,R)) current=R;
    draw();
  }
  function softDrop(){
    for(let i=0;i<3;i++) if(!collision(pos.x,pos.y+1)) pos.y++;
    draw();
  }

  function toggleTheme(){
    document.body.classList.toggle('light');
  }

  // старт
  startGame();
</script>

</body>
</html>
