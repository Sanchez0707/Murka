<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
  <style>
    body { margin:0; padding:0; background:#222; color:#fafafa;
      font-family:Arial,sans-serif; text-align:center;
      user-select:none; touch-action:manipulation;
    }
    h1 { margin:10px 0; }
    #top-bar {
      display:flex; justify-content:space-between; align-items:center;
      max-width:400px; margin:auto; padding:0 10px;
    }
    #score-box { font-size:18px }
    #restart { padding:5px 10px; cursor:pointer }
    #board {
      display:grid;
      grid-template-columns:repeat(10,30px);
      grid-template-rows:repeat(16,30px);
      gap:1px; background:#444;
      margin:10px auto; width:calc(10*30px+9px);
    }
    .cell {
      position: relative; width: 30px; height: 30px;
      background: #eee; border: 1px solid #ccc;
      overflow: hidden;
    }
    /* Плавное появление/движение блока */
    .block {
      position: absolute; width:100%; height:100%;
      background: #333;
      transition: top 0.1s ease;
    }
    .ghost {
      position:absolute; width:100%; height:100%;
      background: rgba(50,50,50,0.75);
    }
    /* Анимация очистки линии */
    @keyframes clear-line {
      0%   { opacity: 1; }
      50%  { opacity: 0; }
      100% { opacity: 1; }
    }
    .clearing .block {
      animation: clear-line 0.3s ease-in-out 2;
    }
    #controls {
      display:flex; justify-content:center; gap:10px; margin:15px 0;
    }
    #controls button {
      width:50px; height:50px; font-size:24px;
      background:#555; color:#fafafa; border:none; border-radius:4px;
      cursor:pointer;
    }
    #controls button:active { background:#777 }
    footer { margin:20px 0; font-size:12px; color:#888 }
  </style>
</head>
<body>

  <h1>Тетрис</h1>
  <div id="top-bar">
    <button id="restart">Заново</button>
    <div id="score-box">Очки: 0</div>
  </div>

  <div id="board"></div>

  <div id="controls">
    <button id="btn-left">←</button>
    <button id="btn-right">→</button>
    <button id="btn-rot">↻</button>
    <button id="btn-drop">↓</button>
  </div>

  <footer>Я не пытаюсь кого-либо плагиатить</footer>

<script>
  const COLS = 10, ROWS = 16;
  let dropInt, score = 0;

  const board = document.getElementById('board');
  const scoreBox = document.getElementById('score-box');
  const restart = document.getElementById('restart');

  // создаём ячейки
  for (let i = 0; i < COLS * ROWS; i++) {
    const c = document.createElement('div');
    c.className = 'cell';
    board.appendChild(c);
  }
  const cells = board.children;

  // Фигуры: I, O, T, L, зеркальный L, 3‑блочный, 2‑блочный, маленькое L
  const SHAPES = [
    [[1,1,1,1]],         // I
    [[1,1],[1,1]],       // O
    [[0,1,0],[1,1,1]],   // T
    [[1,0,0],[1,1,1]],   // L
    [[0,0,1],[1,1,1]],   // зеркальный L
    [[1,1,1]],           // длина 3
    [[1,1]],             // длина 2
    [[1,0],[1,1]]        // маленькое L
  ];

  let grid, current, pos;

  function resetGame() {
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
    if (collide(pos.x,pos.y)) {
      clearInterval(dropInt);
      alert('Игра окончена!');
    }
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
    // очистка
    grid.flat().forEach((v,i)=> cells[i].innerHTML='');
    // занятые
    grid.forEach((row,y)=>row.forEach((v,x)=>{
      if (v) {
        const div = document.createElement('div');
        div.className = 'block';
        // позиционируем для плавного перехода
        div.style.top = '0px';
        cells[y*COLS+x].appendChild(div);
      }
    }));
    // ghost
    let gy = pos.y;
    while(!collide(pos.x,gy+1)) gy++;
    current.forEach((r,ry)=>r.forEach((v,rx)=>{
      if(v) cells[(gy+ry)*COLS + pos.x+rx].innerHTML='<div class="ghost"></div>';
    }));
    // current
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
    // находим полные линии
    for (let y = ROWS - 1; y >= 0; y--) {
      if (grid[y].every(v => v)) {
        lines.push(y);
      }
    }
    if (!lines.length) return;

    // мигаем
    lines.forEach(y => {
      for (let x = 0; x < COLS; x++) {
        const idx = y * COLS + x;
        cells[idx].classList.add('clearing');
      }
    });

    setTimeout(() => {
      // удаляем линии и считаем очки
      const count = lines.length;
      lines.forEach(y => {
        grid.splice(y, 1);
        grid.unshift(Array(COLS).fill(0));
      });
      if (count === 1) score += 10;
      else if (count === 2) score += 50;
      else if (count === 3) score += 200;
      else score += 200;
      scoreBox.textContent = 'Очки: ' + score;

      // убираем класс и перерисовываем
      cells.forEach(c => c.classList.remove('clearing'));
      draw();
    }, 350);
  }

  function drop() {
    if (!collide(pos.x,pos.y+1)) {
      pos.y++;
    } else {
      current.forEach((r,ry)=>r.forEach((v,rx)=>{
        if (v) grid[pos.y+ry][pos.x+rx] = 1;
      }));
      clearLines();
      spawn();
    }
    draw();
  }

  // управление
  document.getElementById('btn-left').addEventListener('mousedown', e=>{
    e.preventDefault();
    if (!collide(pos.x-1,pos.y)) pos.x--;
    draw();
  });
  document.getElementById('btn-right').addEventListener('mousedown', e=>{
    e.preventDefault();
    if (!collide(pos.x+1,pos.y)) pos.x++;
    draw();
  });
  document.getElementById('btn-rot').addEventListener('mousedown', e=>{
    e.preventDefault();
    const R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if (!collide(pos.x,pos.y,R)) current = R;
    draw();
  });
  document.getElementById('btn-drop').addEventListener('mousedown', e=>{
    e.preventDefault();
    for (let i=0; i<3; i++){
      if (!collide(pos.x,pos.y+1)) pos.y++;
    }
    draw();
  });
  restart.addEventListener('mousedown', ()=> resetGame());

  resetGame();
</script>

</body>
</html>
