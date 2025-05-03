<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Мини-Тетрис</title>
  <style>
    body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; margin: 0; }
    canvas { border: 2px solid #444; margin-top: 10px; }
    .controls, .pieces { margin-top: 10px; display: flex; gap: 10px; flex-wrap: wrap; justify-content: center; }
    button { padding: 10px; font-size: 16px; }
  </style>
</head>
<body>
  <h2>Мини-Тетрис 10×10</h2>
  <canvas id="game" width="300" height="300"></canvas>

  <div class="controls">
    <button onclick="moveLeft()">←</button>
    <button onclick="rotate()">⟳</button>
    <button onclick="moveRight()">→</button>
    <button onclick="place()">Поставить</button>
  </div>

  <div class="pieces">
    <div id="preview1"></div>
    <div id="preview2"></div>
    <div id="preview3"></div>
  </div>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");
    const cellSize = 30;
    const rows = 10, cols = 10;
    const board = Array.from({ length: rows }, () => Array(cols).fill(0));

    const pieces = [
      [[1,1],[1,1]], // квадрат
      [[1],[1],[1],[1]], // палка 4 вертик
      [[1,1,1,1]], // палка 4 гориз
      [[1],[1],[1]], // палка 3 вертик
      [[1,1,1]], // палка 3 гориз
      [[0,1,0],[1,1,1]], // крест
      [[1,0],[1,0],[1,1]], // L-образная
    ];

    let previews = [null, null, null];
    let currentIndex = 0;
    let currentPiece = null;
    let pieceX = 3, pieceY = 0;

    function drawBoard() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
          if (board[y][x]) drawCell(x, y, "#444");
        }
      }
      drawPiece(currentPiece, pieceX, pieceY, "#0a0");
    }

    function drawCell(x, y, color) {
      ctx.fillStyle = color;
      ctx.fillRect(x * cellSize, y * cellSize, cellSize - 2, cellSize - 2);
    }

    function drawPiece(piece, x, y, color) {
      if (!piece) return;
      for (let dy = 0; dy < piece.length; dy++) {
        for (let dx = 0; dx < piece[dy].length; dx++) {
          if (piece[dy][dx]) drawCell(x + dx, y + dy, color);
        }
      }
    }

    function moveLeft() {
      if (canPlace(currentPiece, pieceX - 1, pieceY)) pieceX--;
      drawBoard();
    }

    function moveRight() {
      if (canPlace(currentPiece, pieceX + 1, pieceY)) pieceX++;
      drawBoard();
    }

    function rotate() {
      const rotated = rotateMatrix(currentPiece);
      if (canPlace(rotated, pieceX, pieceY)) currentPiece = rotated;
      drawBoard();
    }

    function place() {
      if (!canPlace(currentPiece, pieceX, pieceY)) return;
      for (let dy = 0; dy < currentPiece.length; dy++) {
        for (let dx = 0; dx < currentPiece[dy].length; dx++) {
          if (currentPiece[dy][dx]) board[pieceY + dy][pieceX + dx] = 1;
        }
      }
      clearLines();
      loadNextPiece();
      drawBoard();
    }

    function canPlace(piece, x, y) {
      for (let dy = 0; dy < piece.length; dy++) {
        for (let dx = 0; dx < piece[dy].length; dx++) {
          if (piece[dy][dx]) {
            let nx = x + dx, ny = y + dy;
            if (nx < 0 || nx >= cols || ny < 0 || ny >= rows || board[ny][nx]) return false;
          }
        }
      }
      return true;
    }

    function rotateMatrix(matrix) {
      return matrix[0].map((_, i) => matrix.map(row => row[i]).reverse());
    }

    function clearLines() {
      for (let y = 0; y < rows; y++) {
        if (board[y].every(cell => cell)) {
          board.splice(y, 1);
          board.unshift(Array(cols).fill(0));
        }
      }
    }

    function loadNextPiece() {
      currentPiece = previews[currentIndex];
      pieceX = 3;
      pieceY = 0;
      previews[currentIndex] = getRandomPiece();
      renderPreviews();
      currentIndex = (currentIndex + 1) % 3;
    }

    function getRandomPiece() {
      return JSON.parse(JSON.stringify(pieces[Math.floor(Math.random() * pieces.length)]));
    }

    function renderPreviews() {
      for (let i = 0; i < 3; i++) {
        const el = document.getElementById(`preview${i+1}`);
        el.innerHTML = previews[i].map(row => row.map(cell => cell ? "■" : "□").join("")).join("<br>");
      }
    }

    // Инициализация
    for (let i = 0; i < 3; i++) previews[i] = getRandomPiece();
    loadNextPiece();
    drawBoard();
  </script>
</body>
</html>
