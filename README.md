<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Игра-Блоки</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      background: #111;
      color: white;
      font-family: sans-serif;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(20, 30px);
      gap: 1px;
      margin-top: 20px;
      background: #444;
    }
    .cell {
      width: 30px;
      height: 30px;
      background: #222;
      border: 1px solid #333;
    }
    .figures {
      margin-top: 20px;
      display: flex;
      gap: 10px;
    }
    .figure {
      display: grid;
      grid-template-columns: repeat(2, 30px);
      grid-template-rows: repeat(2, 30px);
      gap: 1px;
      cursor: grab;
    }
    .block {
      background: red;
      width: 30px;
      height: 30px;
    }
  </style>
</head>
<body>

  <h1>Игровое поле</h1>
  <div class="grid" id="gameGrid"></div>

  <h2>Фигуры</h2>
  <div class="figures">
    <div class="figure" draggable="true" id="square">
      <div class="block"></div>
      <div class="block"></div>
      <div class="block"></div>
      <div class="block"></div>
    </div>
  </div>

  <script>
    // Создание поля
    const grid = document.getElementById('gameGrid');
    for (let i = 0; i < 200; i++) {
      const cell = document.createElement('div');
      cell.classList.add('cell');
      grid.appendChild(cell);
    }

    // Перетаскивание
    const square = document.getElementById('square');
    square.addEventListener('dragstart', (e) => {
      e.dataTransfer.setData("text/plain", "square");
    });

    grid.addEventListener('dragover', (e) => {
      e.preventDefault();
    });

    grid.addEventListener('drop', (e) => {
      e.preventDefault();
      const figure = document.getElementById('square').cloneNode(true);
      figure.style.position = 'absolute';
      figure.style.left = `${e.pageX - 30}px`;
      figure.style.top = `${e.pageY - 30}px`;
      document.body.appendChild(figure);
    });
  </script>

</body>
</html>
