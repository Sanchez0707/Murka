<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Пазл: Собери картинку</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      margin-top: 20px;
    }
    #board {
      width: 400px;
      height: 400px;
      margin: auto;
      display: flex;
      flex-wrap: wrap;
      border: 2px solid #000;
    }
    .piece {
      width: 200px;
      height: 200px;
      box-sizing: border-box;
      border: 1px solid #999;
      background-image: url('https://sun9-3.userapi.com/impg/vxBZgckQyhFhpkX7V5FQvNmz-2fpaWGDwrWfNg/3Zn9PEFnrn4.jpg?size=350x604&quality=96&sign=1b4adada310abcfa40aff68029028b3c&type=album');
      background-size: 400px 400px;
      background-repeat: no-repeat;
    }
  </style>
</head>
<body>
  <h2>Собери картинку</h2>
  <div id="board"></div>

  <script>
    const board = document.getElementById('board');

    // координаты частей картинки
    const positions = [
      '0px 0px',
      '-200px 0px',
      '0px -200px',
      '-200px -200px'
    ];

    // создаём и перемешиваем части
    const shuffled = positions.sort(() => Math.random() - 0.5);
    shuffled.forEach((pos, i) => {
      const div = document.createElement('div');
      div.className = 'piece';
      div.style.backgroundPosition = pos;
      div.draggable = true;
      div.dataset.position = pos;
      board.appendChild(div);
    });

    let dragged;

    board.addEventListener('dragstart', e => {
      dragged = e.target;
    });

    board.addEventListener('dragover', e => {
      e.preventDefault();
    });

    board.addEventListener('drop', e => {
      if (e.target.className === 'piece' && e.target !== dragged) {
        // обмен позициями
        const temp = dragged.style.backgroundPosition;
        dragged.style.backgroundPosition = e.target.style.backgroundPosition;
        e.target.style.backgroundPosition = temp;
      }
    });
  </script>
</body>
</html>
