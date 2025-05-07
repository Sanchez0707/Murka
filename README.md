<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Murka</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 20px;
    }

    h1 {
      color: #007bff;
    }

    .container {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 40px;
      flex-wrap: wrap;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(10, 30px);
      gap: 1px;
      margin-top: 20px;
    }

    .cell {
      width: 30px;
      height: 30px;
      background-color: #eee;
      border: 1px solid #ccc;
    }

    .active {
      background-color: #333;
    }

    .controls {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 10px;
    }

    .control-row {
      display: flex;
      gap: 10px;
    }

    .control-btn {
      width: 50px;
      height: 50px;
      font-size: 24px;
      border: none;
      border-radius: 10px;
      background-color: #f0f0f0;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      transition: background-color 0.2s ease;
    }

    .control-btn:hover {
      background-color: #e0e0e0;
    }

    .control-btn:active {
      background-color: #ccc;
    }
  </style>
</head>
<body>
  <h1>Murka</h1>
  <h2>Мини-Тетрис 10×10</h2>
  <div>Время: <span id="timer">15</span></div>

  <div class="container">
    <div id="grid" class="grid"></div>

    <div class="controls">
      <div class="control-row">
        <button class="control-btn" onclick="move('up')">↑</button>
        <button class="control-btn" onclick="rotate()">⟳</button>
      </div>
      <div class="control-row">
        <button class="control-btn" onclick="move('left')">←</button>
        <button class="control-btn" disabled style="opacity:0;">•</button>
        <button class="control-btn" onclick="move('right')">→</button>
      </div>
      <div>
        <button class="control-btn" onclick="move('down')">↓</button>
      </div>
    </div>
  </div>

  <script>
    const grid = document.getElementById("grid");
    const timerEl = document.getElementById("timer");
    const gridSize = 10;
    const cells = [];

    for (let i = 0; i < gridSize * gridSize; i++) {
      const cell = document.createElement("div");
      cell.classList.add("cell");
      grid.appendChild(cell);
      cells.push(cell);
    }

    let currentBlock = [];
    let timer = 15;
    let timerInterval;

    const figures = [
      [0, 1, 10, 11], // квадрат
      [0, 1, 2],      // палка
      [0, 10, 20, 30],// палка вертикальная
      [1, 10, 11, 12],// крест
      [0, 1, 11, 21], // L-образная
    ];

    function getRandomFigure() {
      const fig = figures[Math.floor(Math.random() * figures.length)];
      const offset = Math.floor(Math.random() * (gridSize * (gridSize - 3)));
      return fig.map(i => i + offset);
    }

    function drawBlock(block) {
      block.forEach(i => {
        if (cells[i]) cells[i].classList.add("active");
      });
    }

    function clearBlock(block) {
      block.forEach(i => {
        if (cells[i]) cells[i].classList.remove("active");
      });
    }

    function placeBlock() {
      clearBlock(currentBlock);
      currentBlock.forEach(i => {
        if (cells[i]) cells[i].classList.add("active");
      });
      clearInterval(timerInterval);
      startTimer(); // Запускаем таймер заново
      currentBlock = getRandomFigure();
      drawBlock(currentBlock);
    }

    function move(direction) {
      clearBlock(currentBlock);
      let offset = 0;

      switch (direction) {
        case 'up': offset = -gridSize; break;
        case 'down': offset = gridSize; break;
        case 'left': offset = -1; break;
        case 'right': offset = 1; break;
      }

      const newBlock = currentBlock.map(i => i + offset);
      if (newBlock.every(i => i >= 0 && i < gridSize * gridSize)) {
        currentBlock = newBlock;
      }

      drawBlock(currentBlock);
    }

    function rotate() {
      // простая псевдо-ротация: новая фигура
      placeBlock();
    }

    function startTimer() {
      timer = 15;
      timerEl.textContent = timer;
      timerInterval = setInterval(() => {
        timer--;
        timerEl.textContent = timer;
        if (timer <= 0) {
          clearInterval(timerInterval);
          autoPlace();
        }
      }, 1000);
    }

    function autoPlace() {
      clearBlock(currentBlock);
      currentBlock = getRandomFigure();
      drawBlock(currentBlock);
      startTimer();
    }

    // старт игры
    currentBlock = getRandomFigure();
    drawBlock(currentBlock);
    startTimer();
  </script>
</body>
</html>
