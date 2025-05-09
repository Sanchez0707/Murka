<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin:0; padding:0;
      background:#111; color:#fafafa;
      font-family:'Courier New', monospace;
      text-align:center;
      user-select:none; touch-action:manipulation;
    }
    h1 { margin:10px 0; color:#0ff; text-shadow: 0 0 8px #0ff; }

    /* Верхняя панель */
    #top-bar {
      display:flex; justify-content:space-between; align-items:center;
      max-width:400px; margin:auto; padding:0 10px;
    }
    #top-bar button {
      padding:5px 10px; cursor:pointer;
      background:#222; color:#0ff; border:2px solid #0ff; border-radius:4px;
      text-shadow: 0 0 4px #0ff;  
    }
    #top-bar button:active { background:#0a0a0a; }
    #score-box, #best-box {
      font-size:18px; color:#0ff; text-shadow:0 0 4px #0ff;
    }

    /* Поле */
    #board {
      display:grid;
      grid-template-columns:repeat(10,30px);
      grid-template-rows:repeat(16,30px);
      gap:1px; background:#222;
      margin:10px auto; width:calc(10*30px+9px);
      box-shadow: 0 0 8px #0ff inset;
    }
    .cell {
      position: relative; width:30px; height:30px;
      background:#000; border:1px solid #0a0a0a;
      overflow:hidden;
    }
    .block {
      position:absolute; width:100%; height:100%;
      background:#0ff; box-shadow:0 0 4px #0ff, inset 0 0 4px #0ff;
      transition: top 0.1s ease;
    }
    .ghost {
      position:absolute; width:100%; height:100%;
      background:rgba(0,255,255,0.3);
    }
    @keyframes clear-line {
      0%   { opacity: 1; }
      50%  { opacity: 0; }
      100% { opacity: 1; }
    }
    .clearing .block {
      animation: clear-line 0.3s ease-in-out 2;
    }

    /* Кнопки управления */
    #controls {
      display:flex; justify-content:center; gap:10px; margin:15px 0;
    }
    #controls button {
      width:50px; height:50px; font-size:24px;
      background:#222; color:#0ff; border:2px solid #0ff; border-radius:4px;
      cursor:pointer; text-shadow:0 0 4px #0ff;
    }
    #controls button:active { background:#0a0a0a; }

    footer {
      margin:20px 0; font-size:12px; color:#555;
    }

    /* ЭКРАН КОНЦА ИГРЫ */
    #game-over-screen {
      display: none;
      position: fixed; top:0; left:0; right:0; bottom:0;
      background: rgba(0,0,0,0.85);
      z-index:1000;
      justify-content:center; align-items:center;
    }
    #game-over-content {
      background:#000;
      border:4px solid #0ff;
      padding:30px;
      border-radius:8px;
      text-align:center;
      box-shadow:0 0 12px #0ff;
    }
    #game-over-content h2 {
      margin:0 0 15px;
      font-size:28px;
      color:#0ff;
      text-shadow:0 0 6px #0ff;
    }
    #game-over-content p {
      font-size:20px;
      margin:0 0 20px;
      color:#fff;
    }
    #play-again {
      padding:10px 20px;
      font-size:18px;
      background:#0ff; color:#000;
      border:none; border-radius:4px;
      cursor:pointer;
      box-shadow:0 0 6px #0ff;
    }
    #play-again:active { background:#0cc; }
  </style>
</head>
<body>

  <h1>Тетрис</h1>
  <div id="top-bar">
    <button id="restart">Заново</button>
    <div>
      <span id="score-box">Очки: 0</span>
      &nbsp;|&nbsp;
      <span id="best-box">Лучший: 0</span>
    </div>
    <button id="btn-music">Музыка: Вкл</button>
  </div>

  <div id="board"></div>

  <div id="controls">
    <button id="btn-left">←</button>
    <button id="btn-right">→</button>
    <button id="btn-rot">↻</button>
    <button id="btn-drop">↓</button>
  </div>

  <!-- Экран конца игры -->
  <div id="game-over-screen">
    <div id="game-over-content">
      <h2>Игра окончена!</h2>
      <p id="final-score">Вы набрали 0 очков</p>
      <button id="play-again">Играть снова</button>
    </div>
  </div>

  <footer>Я не пытаюсь кого-либо плагиатить</footer>

  <audio id="bg-music" loop preload="auto">
    <source src="Video_Game_Players_-_Tetris_Theme_48152782.mp3" type="audio/mpeg">
  </audio>

<script>
  const COLS = 10, ROWS = 16;
  let dropInt, score = 0;
  const board = document.getElementById('board');
  const scoreBox = document.getElementById('score-box');
  const bestBox = document.getElementById('best-box');
  const restart = document.getElementById('restart');
  const musicBtn = document.getElementById('btn-music');
  const bgMusic = document.getElementById('bg-music');
  const gameOverScreen = document.getElementById('game-over-screen');
  const finalScore = document.getElementById('final-score');
  const playAgain = document.getElementById('play-again');

  // Построение поля
  for (let i = 0; i < COLS * ROWS; i++) {
    const c = document.createElement('div');
    c.className = 'cell';
    board.appendChild(c);
  }
  const cells = board.children;

  // Фигуры
  const SHAPES = [
    [[1,1,1,1]],
    [[1,1],[1,1]],
    [[0,1,0],[1,1,1]],
    [[1,0,0],[1,1,1]],
    [[0,0,1],[1,1,1]],
    [[1,1,1]],
    [[1,1]],
    [[1,0],[1,1]]
  ];

  let grid, current, pos;
  let best = parseInt(localStorage.getItem('tetrisBest')) || 0;
  bestBox.textContent = 'Лучший: ' + best;

  function resetGame() {
    gameOverScreen.style.display = 'none';
    clearInterval(dropInt);
    grid = Array.from({length:ROWS}, ()=>Array(COLS).fill(0));
    score = 0;
    scoreBox.textContent = 'Очки: 0';
    spawn(); draw();
    dropInt = setInterval(drop, 500);
  }

  function spawn() {
    const s = SHAPES[Math.floor(Math.random()*SHAPES.length)];
    current = s.map(r=>[...r]);
    pos = { x: Math.floor((COLS - current[0].length)/2), y: 0 };
    if (collide(pos.x,pos.y)) endGame();
  }

  function collide(px,py, fig=current) {
    for (let y=0; y<fig.length; y++){
      for (let x=0; x<fig[0].length; x++){
        if (fig[y][x]) {
          const nx=px+x, ny=py+y;
          if (nx<0||nx>=COLS||ny>=ROWS||grid[ny][nx]) return true;
        }
      }
    }
    return false;
  }

  function draw() {
    Array.from(cells).forEach(c => c.classList.remove('clearing'));
    grid.flat().forEach((v,i)=> cells[i].innerHTML='');
    grid.forEach((row,y)=>row.forEach((v,x)=>{
      if (v) {
        const div = document.createElement('div');
        div.className = 'block';
        div.style.top = '0px';
        cells[y*COLS+x].appendChild(div);
      }
    }));
    let gy = pos.y;
    while(!collide(pos.x,gy+1)) gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(gy+ry)*COLS + pos.x+rx].innerHTML = '<div class="ghost"></div>';
    }));
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) {
        const div = document.createElement('div');
        div.className = 'block';
        div.style.top = '0px';
        cells[(pos.y+ry)*COLS + pos.x+rx].appendChild(div);
      }
    }));
  }

  function clearLines() {
    const lines = [];
    for (let y = ROWS - 1; y >= 0; y--) {
      if (grid[y].every(v => v)) lines.push(y);
    }
    if (!lines.length) return;
    lines.forEach(y => {
      for (let x = 0; x < COLS; x++) {
        cells[y*COLS + x].classList.add('clearing');
      }
    });
    setTimeout(() => {
      const count = lines.length;
      lines.sort((a,b)=>a-b).forEach((y,i) => {
        grid.splice(y-i, 1);
        grid.unshift(Array(COLS).fill(0));
      });
      if (count === 1) score += 10;
      else if (count === 2) score += 50;
      else if (count === 3) score += 200;
      else score += 200;
      scoreBox.textContent = 'Очки: ' + score;
      draw();
    }, 350);
  }

  function drop() {
    if (!collide(pos.x,pos.y+1)) pos.y++;
    else {
      current.forEach((r,ry)=>r.forEach((v,rx)=>{
        if (v) grid[pos.y+ry][pos.x+rx] = 1;
      }));
      clearLines();
      spawn();
    }
    draw();
  }

  function endGame() {
    clearInterval(dropInt);
    if (score > best) {
      best = score;
      localStorage.setItem('tetrisBest', best);
      bestBox.textContent = 'Лучший: ' + best;
    }
    finalScore.textContent = 'Вы набрали ' + score + ' очков';
    gameOverScreen.style.display = 'flex';
  }

  // Управление
  document.getElementById('btn-left').addEventListener('mousedown', e=>{
    e.preventDefault(); if (!collide(pos.x-1,pos.y)) pos.x--; draw();
  });
  document.getElementById('btn-right').addEventListener('mousedown', e=>{
    e.preventDefault(); if (!collide(pos.x+1,pos.y)) pos.x++; draw();
  });
  document.getElementById('btn-rot').addEventListener('mousedown', e=>{
    e.preventDefault();
    const R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if (!collide(pos.x,pos.y,R)) current = R; draw();
  });
  document.getElementById('btn-drop').addEventListener('mousedown', e=>{
    e.preventDefault();
    for (let i=0; i<3; i++) if (!collide(pos.x,pos.y+1)) pos.y++;
    draw();
  });
  restart.addEventListener('mousedown', ()=> resetGame());

  // Музыка
  let musicOn = true;
  bgMusic.load();
  musicBtn.addEventListener('click', () => {
    musicOn = !musicOn;
    if (musicOn) bgMusic.play().catch(()=>{}), musicBtn.textContent = 'Музыка: Вкл';
    else bgMusic.pause(), musicBtn.textContent = 'Музыка: Выкл';
  });
  document.body.addEventListener('click', () => {
    if (musicOn) bgMusic.play().catch(()=>{});
  }, { once: true });

  // Старт
  resetGame();
</script>

</body>
</html>
