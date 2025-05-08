<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Типо Тетрис</title>
  <style>
    body {
      margin: 0;
      font-family: sans-serif;
      background-color: #111;
      color: white;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    #game-container {
      display: flex;
      justify-content: center;
      margin-top: 20px;
    }

    #game-board {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(16, 30px);
      gap: 1px;
      background-color: #222;
    }

    .cell {
      width: 30px;
      height: 30px;
      background-color: #333;
      border: 1px solid #444;
    }

    .filled {
      background-color: #f00;
    }

    #controls {
      margin-top: 20px;
      display: flex;
      gap: 10px;
    }

    button {
      padding: 10px 15px;
      font-size: 16px;
      cursor: pointer;
    }

    .light-theme {
      background-color: #eee;
      color: black;
    }

    .light-theme #game-board {
      background-color: #ddd;
    }

    .light-theme .cell {
      background-color: #ccc;
      border: 1px solid #bbb;
    }

    .light-theme .filled {
      background-color: #00f;
    }

  </style>
</head>
<body>
  <h1>Типо Тетрис</h1>
  <div id="game-container">
    <div id="game-board"></div>
  </div>
  <div id="controls">
    <button id="restart">↺</button>
    <button id="theme-toggle">Тема</button>
  </div>
  <script>
    const board = document.getElementById("game-board");

    function createBoard(rows = 16, cols = 10) {
      board.innerHTML = '';
      for (let i = 0; i < rows * cols; i++) {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        board.appendChild(cell);
      }
    }

    document.getElementById("restart").addEventListener("click", () => {
      createBoard();
    });

    let darkMode = true;
    document.getElementById("theme-toggle").addEventListener("click", () => {
      document.body.classList.toggle("light-theme");
      darkMode = !darkMode;
    });

    createBoard();
  </script>
</body>
</html>
