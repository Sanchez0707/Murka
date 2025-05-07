<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8"/>
  <title>Тетрис</title>
  <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: Arial, sans-serif;
      text-align: center;
      user-select: none;
      touch-action: manipulation;
      background: #222;
      color: #fafafa;
    }
    h1 {
      margin: 10px 0;
    }
    #top-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      max-width: 400px;
      margin: auto;
      padding: 0 10px;
    }
    #score-box {
      font-size: 18px;
    }
    #restart {
      padding: 5px 10px;
      font-size: 14px;
      cursor: pointer;
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
      gap: 1px;
      background: #444;
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
      background: #333;
    }
    .ghost {
      position: absolute;
      width: 100%; height: 100%;
      background: rgba(200,200,200,0.3);
    }
    #controls {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 10px;
      margin: 15px 0;
    }
    #controls button {
      width: 50px; height: 50px;
      font-size: 24px;
      cursor: pointer;
      border: none;
      border-radius: 4px;
      background: #555;
      color: #fafafa;
    }
    #controls button:active {
      background: #777;
    }
    footer {
      margin: 20px 0;
      font-size: 12px;
      color: #888;
    }
  </style>
</head>
<body>

  <h1>Тетрис</h1>
  <div id="top-bar">
    <button id="restart">Заново</button>
    <div id="score-box">Очки: 0</div>
  </div>

  <div id="game-container">
    <div id="board"></div>
  </div>

  <div id="controls">
    <button id="btn-left">←</button>
    <button id="btn-rot">↻</button>
    <button id="btn-right">→</button>
    <button id="btn-drop">↓</button>
  </div>

  <footer>Я не пытаюсь кого-либо плагиатить</footer>

<script>
  // Константы
  const COLS = 10, ROWS = 20, SPEED = 500;
  const boardEl = document.getElementById('board');
  const scoreBox = document.getElementById('score-box');
  const restartBtn = document.getElementById('restart');
  let grid, current, pos, score, dropInterval;

  // Фигуры: I, O, T, L
  const SHAPES = [
    [[1,1,1,1]],           // I
    [[1,1],[1,1]],         // O
    [[0,1,0],[1,1,1]],     // T
    [[1,0,0],[1,1,1]]      // L
  ];

  // Построение поля
  for (let i = 0; i < COLS * ROWS; i++) {
    const c = document.createElement('div');
    c.className = 'cell';
    boardEl.appendChild(c);
  }
  const cells = boardEl.children;

  // Инициализация
  function resetGame() {
    clearInterval(dropInterval);
    grid = Array.from({length: ROWS}, ()=>Array(COLS).fill(0));
    score = 0; scoreBox.textContent = 'Очки: 0';
    spawn();
    draw();
    dropInterval = setInterval(drop, SPEED);
  }

  // Спавн новой фигуры в центре
  function spawn() {
    const shape = SHAPES[Math.floor(Math.random() * SHAPES.length)];
    current = shape.map(r=>[...r]);
    pos = {
      x: Math.floor((COLS - current[0].length) / 2),
      y: 0
    };
    if (collide(pos.x,pos.y)) {
      // Конец игры
      clearInterval(dropInterval);
      alert('Игра окончена!');
    }
  }

  // Проверка коллизии
  function collide(px, py, fig = current) {
    for (let y = 0; y < fig.length; y++) {
      for (let x = 0; x < fig[0].length; x++) {
        if (fig[y][x]) {
          const nx = px + x, ny = py + y;
          if (nx < 0 || nx >= COLS || ny >= ROWS || grid[ny][nx]) {
            return true;
          }
        }
      }
    }
    return false;
  }

  // Отрисовка
  function draw() {
    // Очистить
    grid.flat().forEach((v,i)=>{
      cells[i].innerHTML = '';
    });
    // Рисуем занятые
    grid.forEach((row,y)=>{
      row.forEach((v,x)=>{
        if (v) {
          const idx = y*COLS + x;
          cells[idx].innerHTML = '<div class="block"></div>';
        }
      });
    });
    // Ghost (призрак)
    let gy = pos.y;
    while (!collide(pos.x, gy+1)) gy++;
    current.forEach((r,ry)=>{
      r.forEach((v,rx)=>{
        if (v) {
          const idx = (gy+ry)*COLS + (pos.x+rx);
          cells[idx].innerHTML = '<div class="ghost"></div>';
        }
      });
    });
    // Рисуем текущую
    current.forEach((r,ry)=>{
      r.forEach((v,rx)=>{
        if (v) {
          const idx = (pos.y+ry)*COLS + (pos.x+rx);
          cells[idx].innerHTML = '<div class="block"></div>';
        }
      });
    });
  }

  // Очистка линий
  function clearLines() {
    let lines = 0;
    for (let y = ROWS-1; y >= 0; y--) {
      if (grid[y].every(v=>v)) {
        grid.splice(y,1);
        grid.unshift(Array(COLS).fill(0));
        lines++;
        y++;
      }
    }
    if (lines) {
      score += lines * 10;
      scoreBox.textContent = 'Очки: ' + score;
    }
  }

  // Падение вниз
  function drop() {
    if (!collide(pos.x, pos.y+1)) {
      pos.y++;
    } else {
      // Зафиксировать
      current.forEach((r,ry)=>{
        r.forEach((v,rx)=>{
          if (v) grid[pos.y+ry][pos.x+rx] = 1;
        });
      });
      clearLines();
      spawn();
    }
    draw();
  }

  // Управление
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
  document.getElementById('btn-drop').addEventListener('mousedown', e=>{
    e.preventDefault();
    drop();
  });
  document.getElementById('btn-rot').addEventListener('mousedown', e=>{
    e.preventDefault();
    // Поворот
    const R = current[0].map((_,i)=>current.map(r=>r[i]).reverse());
    if (!collide(pos.x,pos.y,R)) {
      current = R;
    }
    draw();
  });
  restartBtn.addEventListener('mousedown', e=>{
    e.preventDefault();
    resetGame();
  });

  // Запуск
  resetGame();
</script>

</body>
</html>
