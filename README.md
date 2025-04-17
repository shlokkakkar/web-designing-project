<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>shlok Pac-Man</title>
  <style>
    body {
      margin: 0;
      background: black;
      font-family: sans-serif;
      color: yellow;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    h1 {
      margin-top: 10px;
    }

    #score {
      font-size: 20px;
    }

    #startBtn {
      padding: 10px 20px;
      margin-top: 10px;
      font-size: 16px;
      cursor: pointer;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(15, 30px);
      grid-template-rows: repeat(15, 30px);
      gap: 2px;
      margin-top: 20px;
    }

    .cell {
      width: 30px;
      height: 30px;
      background: #111;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .wall {
      background: #222;
    }

    .dot {
      background: yellow;
      border-radius: 50%;
      width: 6px;
      height: 6px;
    }

    .pacman {
      background: cyan;
      border-radius: 50%;
      animation: chomp 0.3s infinite alternate;
      width: 100%;
      height: 100%;
    }

    @keyframes chomp {
      0% { transform: scaleY(1); }
      100% { transform: scaleY(0.8); }
    }

    .ghost {
      border-radius: 50% 50% 0 0;
      position: relative;
      width: 100%;
      height: 100%;
    }

    .ghost::after {
      content: '';
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 50%;
      border-radius: 0 0 10px 10px;
    }

    .ghost1 { background: red; }
    .ghost1::after { background: red; }
    .ghost2 { background: orange; }
    .ghost2::after { background: orange; }
    .ghost3 { background: pink; }
    .ghost3::after { background: pink; }
  </style>
</head>
<body>
  <h1>shlok pac-Man </h1>
  <div id="score">Score: 0</div>
  <button id="startBtn">Start Game</button>
  <div class="grid" id="grid"></div>

  <script>
    const grid = document.getElementById('grid');
    const width = 15;
    let interval, moveInterval;

    const layout = [
      1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,
      1,0,0,0,1,0,0,0,0,1,0,0,0,0,1,
      1,0,1,0,1,0,1,1,0,1,0,1,0,1,1,
      1,0,1,0,0,0,0,1,0,0,0,1,0,0,1,
      1,0,1,1,1,1,0,1,1,1,0,1,1,0,1,
      1,0,0,0,0,1,0,0,0,1,0,0,1,0,1,
      1,1,1,1,0,1,1,1,0,1,1,1,1,0,1,
      1,0,0,0,0,0,0,0,0,0,0,0,0,0,1,
      1,0,1,1,1,1,1,1,1,1,1,1,1,0,1,
      1,0,1,0,0,0,0,0,0,0,0,0,1,0,1,
      1,0,1,0,1,1,1,1,1,1,1,0,1,0,1,
      1,0,0,0,0,0,0,0,0,0,0,0,0,0,1,
      1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
    ];

    let pacmanIndex = 16;
    let ghost1Index = 208, ghost2Index = 46, ghost3Index = 152;
    let direction = null, score = 0;
    const sound = new Audio('https://www.myinstants.com/media/sounds/pacman.mp3');

    function draw() {
      grid.innerHTML = '';
      layout.forEach((cell, i) => {
        const div = document.createElement('div');
        div.classList.add('cell');
        if (cell === 1) div.classList.add('wall');
        if (cell === 0) {
          const dot = document.createElement('div');
          dot.classList.add('dot');
          div.appendChild(dot);
        }
        if (i === pacmanIndex) div.classList.add('pacman');
        if (i === ghost1Index) div.classList.add('ghost', 'ghost1');
        if (i === ghost2Index) div.classList.add('ghost', 'ghost2');
        if (i === ghost3Index) div.classList.add('ghost', 'ghost3');
        grid.appendChild(div);
      });
    }

    function updateGame() {
      if (layout[pacmanIndex] === 0) {
        layout[pacmanIndex] = 2;
        score++;
        document.getElementById('score').innerText = 'Score: ' + score;
        sound.play();
      }
      draw();
      if ([ghost1Index, ghost2Index, ghost3Index].includes(pacmanIndex)) {
        clearInterval(interval);
        clearInterval(moveInterval);
        alert("Game Over!");
      }
    }

    function autoMove() {
      if (!direction) return;
      const left = pacmanIndex % width !== 0;
      const right = pacmanIndex % width !== width - 1;
      const up = pacmanIndex - width >= 0;
      const down = pacmanIndex + width < layout.length;

      if (direction === 'ArrowRight' && right && layout[pacmanIndex + 1] !== 1) pacmanIndex++;
      else if (direction === 'ArrowLeft' && left && layout[pacmanIndex - 1] !== 1) pacmanIndex--;
      else if (direction === 'ArrowUp' && up && layout[pacmanIndex - width] !== 1) pacmanIndex -= width;
      else if (direction === 'ArrowDown' && down && layout[pacmanIndex + width] !== 1) pacmanIndex += width;
      updateGame();
    }

    function moveGhost(index, setIndex) {
      const directions = [-1, 1, -width, width];
      let best = index, shortest = Infinity;
      directions.forEach(dir => {
        const next = index + dir;
        if (layout[next] !== 1 && next >= 0 && next < layout.length) {
          const dist = Math.abs(next % width - pacmanIndex % width) + Math.abs(Math.floor(next / width) - Math.floor(pacmanIndex / width));
          if (dist < shortest) {
            shortest = dist;
            best = next;
          }
        }
      });
      setIndex(best);
    }

    document.addEventListener('keydown', e => direction = e.key);

    document.getElementById('startBtn').addEventListener('click', () => {
      interval = setInterval(() => {
        moveGhost(ghost1Index, newPos => ghost1Index = newPos);
        moveGhost(ghost2Index, newPos => ghost2Index = newPos);
        moveGhost(ghost3Index, newPos => ghost3Index = newPos);
        draw();
      }, 500);
      moveInterval = setInterval(autoMove, 200);
    });

    draw();
  </script>
</body>
</html>
# web-designing-project
