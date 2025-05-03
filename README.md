<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Перетащи в цель</title>
  <style>
    #box {
      width: 100px;
      height: 100px;
      background-color: blue;
      margin: 20px;
      cursor: grab;
    }

    #target {
      width: 120px;
      height: 120px;
      border: 2px dashed green;
      margin: 20px auto;
    }

    #message {
      font-size: 20px;
      color: green;
      display: none;
    }
  </style>
</head>
<body>
  <div id="box" draggable="true"></div>
  <div id="target"></div>
  <p id="message">Молодец!</p>

  <script>
    const box = document.getElementById("box");
    const target = document.getElementById("target");
    const message = document.getElementById("message");

    box.addEventListener("dragstart", e => {
      e.dataTransfer.setData("text/plain", "box");
    });

    target.addEventListener("dragover", e => {
      e.preventDefault();
    });

    target.addEventListener("drop", e => {
      const data = e.dataTransfer.getData("text/plain");
      if (data === "box") {
        target.appendChild(box);
        message.style.display = "block";
      }
    });
  </script>
</body>
</html>
