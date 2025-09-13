<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>2048 - Bitta HTML fayl</title>
  <style>
    :root{
      --bg:#faf8ef;
      --board:#bbada0;
      --tile-2:#eee4da;
      --tile-4:#ede0c8;
      --tile-8:#f2b179;
      --tile-16:#f59563;
      --tile-32:#f67c5f;
      --tile-64:#f65e3b;
      --tile-128:#edcf72;
      --tile-256:#edcc61;
      --tile-512:#edc850;
      --tile-1024:#edc53f;
      --tile-2048:#edc22e;
      --text-dark:#776e65;
      --accent:#8f7a66;
    }
    *{box-sizing:border-box}
    body{font-family: 'Segoe UI', Roboto, Arial; background:var(--bg); color:var(--text-dark); display:flex;align-items:center;justify-content:center;height:100vh;margin:0}
    .container{width:420px;max-width:96vw}
    header{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px}
    header h1{font-size:22px;margin:0}
    .controls{display:flex;gap:10px;align-items:center}
    .score, .best{background:#bbada0;color:white;padding:8px 12px;border-radius:6px;font-weight:700}
    .buttons{display:flex;gap:8px}
    button{background:#8f7a66;color:white;border:none;padding:8px 12px;border-radius:6px;cursor:pointer}
    button.secondary{background:transparent;color:var(--accent);border:2px solid var(--accent)}

    .board{background:var(--board);padding:15px;border-radius:10px;position:relative;user-select:none;cursor:grab}
    .board:active{cursor:grabbing}

    .grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
    .cell{width:80px;height:80px;border-radius:6px;background:rgba(238,228,218,0.35);display:flex;align-items:center;justify-content:center;font-weight:700;font-size:24px}

    .tiles{position:absolute;inset:15px;display:grid;grid-template-columns:repeat(4,1fr);gap:12px;pointer-events:none}
    .tile{position:relative;display:flex;align-items:center;justify-content:center;border-radius:6px;font-weight:700;transition:transform 120ms ease, background 200ms linear, opacity 120ms linear}
    .tile .num{font-size:22px;line-height:1}
    .tile.new{animation:pop 140ms ease}
    .tile.merge{animation:merge 160ms ease}
    @keyframes pop{0%{transform:scale(.2)}100%{transform:scale(1)}}
    @keyframes merge{0%{transform:scale(1)}50%{transform:scale(1.15)}100%{transform:scale(1)}}

    .tile[data-val="2"]{background:var(--tile-2);color:var(--text-dark)}
    .tile[data-val="4"]{background:var(--tile-4);color:var(--text-dark)}
    .tile[data-val="8"]{background:var(--tile-8);color:white}
    .tile[data-val="16"]{background:var(--tile-16);color:white}
    .tile[data-val="32"]{background:var(--tile-32);color:white}
    .tile[data-val="64"]{background:var(--tile-64);color:white}
    .tile[data-val="128"]{background:var(--tile-128);color:white}
    .tile[data-val="256"]{background:var(--tile-256);color:white}
    .tile[data-val="512"]{background:var(--tile-512);color:white}
    .tile[data-val="1024"]{background:var(--tile-1024);color:white}
    .tile[data-val="2048"]{background:var(--tile-2048);color:white}

    .overlay{position:absolute;inset:0;background:rgba(255,255,255,0.6);display:flex;align-items:center;justify-content:center;font-size:28px;font-weight:700;border-radius:10px}

    footer{margin-top:12px;font-size:13px;color:#7a6b5b}

    @media (max-width:420px){.cell{width:64px;height:64px;font-size:18px}.tiles{gap:8px}.tile .num{font-size:18px}}
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>2048 — Bitta HTML</h1>
      <div class="controls">
        <div class="score">Score: <span id="score">0</span></div>
        <div class="best">Best: <span id="best">0</span></div>
        <div class="buttons">
          <button id="newBtn">Yangi</button>
          <button id="undoBtn" class="secondary">Ortga</button>
        </div>
      </div>
    </header>

    <div class="board" id="board">
      <div class="grid" id="grid"></div>
      <div class="tiles" id="tiles"></div>
      <div class="overlay" id="overlay" style="display:none">GAME OVER</div>
    </div>
    <footer>Klaviatura: arrow yoki WASD. Sichqoncha: bosib suring (drag). Mobil: suring (swipe).</footer>
  </div>

  <script>
    (function(){
      const SIZE = 4;
      let board = createEmpty();
      let score = 0;
      let best = parseInt(localStorage.getItem('2048_best')||0,10);
      let history = [];

      const gridEl = document.getElementById('grid');
      const tilesEl = document.getElementById('tiles');
      const scoreEl = document.getElementById('score');
      const bestEl = document.getElementById('best');
      const overlay = document.getElementById('overlay');
      const boardEl = document.getElementById('board');

      function buildGrid(){
        gridEl.innerHTML='';
        for(let i=0;i<SIZE*SIZE;i++){
          const c = document.createElement('div');
          c.className='cell';
          gridEl.appendChild(c);
        }
      }

      function createEmpty(){
        return Array.from({length:SIZE},()=>Array(SIZE).fill(0));
      }

      function cloneBoard(b){ return b.map(r=>r.slice()); }
      function saveHistory(){ history.push({board:cloneBoard(board),score}); if(history.length>20) history.shift(); }
      function undo(){ if(history.length===0) return; const prev = history.pop(); board=cloneBoard(prev.board); score=prev.score; updateScore(); render(); overlay.style.display='none'; }

      function spawnRandom(){
        const empties=[];
        for(let r=0;r<SIZE;r++)for(let c=0;c<SIZE;c++)if(board[r][c]===0)empties.push([r,c]);
        if(empties.length===0) return false;
        const [r,c]=empties[Math.floor(Math.random()*empties.length)];
        board[r][c]=Math.random()<0.9?2:4;
        return {r,c,val:board[r][c]};
      }

      function updateScore(){ scoreEl.textContent=score; if(score>best){best=score;localStorage.setItem('2048_best',best);} bestEl.textContent=best; }
      function canMove(){
        for(let r=0;r<SIZE;r++)for(let c=0;c<SIZE;c++){
          if(board[r][c]===0) return true;
          if(c<SIZE-1 && board[r][c]===board[r][c+1]) return true;
          if(r<SIZE-1 && board[r][c]===board[r+1][c]) return true;
        }
        return false;
      }

      function move(dir){
        saveHistory();
        let moved=false;
        function slideRow(row){
          const arr=row.filter(v=>v!==0);
          for(let i=0;i<arr.length-1;i++){
            if(arr[i]===arr[i+1]){arr[i]*=2;score+=arr[i];arr.splice(i+1,1);}
          }
          while(arr.length<SIZE)arr.push(0);
          return arr;
        }
        if(dir==='left'){
          for(let r=0;r<SIZE;r++){const before=board[r].join(',');board[r]=slideRow(board[r]);if(board[r].join(',')!==before)moved=true;}
        }else if(dir==='right'){
          for(let r=0;r<SIZE;r++){const before=board[r].join(',');board[r]=slideRow(board[r].slice().reverse()).reverse();if(board[r].join(',')!==before)moved=true;}
        }else if(dir==='up'){
          for(let c=0;c<SIZE;c++){const col=[];for(let r=0;r<SIZE;r++)col.push(board[r][c]);const before=col.join(',');const slid=slideRow(col);for(let r=0;r<SIZE;r++)board[r][c]=slid[r];if(col.join(',')!==before)moved=true;}
        }else if(dir==='down'){
          for(let c=0;c<SIZE;c++){const col=[];for(let r=0;r<SIZE;r++)col.push(board[r][c]);const before=col.join(',');const slid=slideRow(col.slice().reverse()).reverse();for(let r=0;r<SIZE;r++)board[r][c]=slid[r];if(col.join(',')!==before)moved=true;}
        }
        if(moved){const spawn=spawnRandom();updateScore();render(spawn);if(!canMove()){overlay.textContent='GAME OVER';overlay.style.display='flex';}}else{history.pop();}
      }

      function render(newTile){
        tilesEl.innerHTML='';
        for(let r=0;r<SIZE;r++)for(let c=0;c<SIZE;c++){
          const val=board[r][c];
          if(val===0)continue;
          const t=document.createElement('div');
          t.className='tile';t.dataset.val=val;
          t.style.gridRowStart=r+1;t.style.gridColumnStart=c+1;
          const n=document.createElement('div');n.className='num';n.textContent=val;t.appendChild(n);
          tilesEl.appendChild(t);
          if(newTile&&newTile.r===r&&newTile.c===c&&newTile.val===val){t.classList.add('new');setTimeout(()=>t.classList.remove('new'),400);}
        }
      }

      document.getElementById('newBtn').addEventListener('click',startNew);
      document.getElementById('undoBtn').addEventListener('click',undo);
      function startNew(){board=createEmpty();score=0;history=[];overlay.style.display='none';spawnRandom();spawnRandom();updateScore();render();}

      window.addEventListener('keydown',e=>{
        const k=e.key;
        if(k==='ArrowLeft'||k==='a'||k==='A')move('left');
        else if(k==='ArrowRight'||k==='d'||k==='D')move('right');
        else if(k==='ArrowUp'||k==='w'||k==='W')move('up');
        else if(k==='ArrowDown'||k==='s'||k==='S')move('down');
      });

      // Touch swipe
      let touchStartX=0,touchStartY=0;
      window.addEventListener('touchstart',e=>{const t=e.touches[0];touchStartX=t.clientX;touchStartY=t.clientY;},{passive:true});
      window.addEventListener('touchend',e=>{const t=e.changedTouches[0];const dx=t.clientX-touchStartX;const dy=t.clientY-touchStartY;if(Math.abs(dx)>30||Math.abs(dy)>30){if(Math.abs(dx)>Math.abs(dy)){if(dx>0)move('right');else move('left');}else{if(dy>0)move('down');else move('up');}}});

      // Mouse drag swipe
      let dragStartX=0,dragStartY=0,dragging=false;
      boardEl.addEventListener('mousedown',e=>{dragging=true;dragStartX=e.clientX;dragStartY=e.clientY;});
      window.addEventListener('mouseup',e=>{
        if(!dragging)return;dragging=false;
        const dx=e.clientX-dragStartX,dy=e.clientY-dragStartY;
        if(Math.abs(dx)>30||Math.abs(dy)>30){
          if(Math.abs(dx)>Math.abs(dy)){if(dx>0)move('right');else move('left');}
          else{if(dy>0)move('down');else move('up');}
        }
      });

      buildGrid();startNew();
    })();
  </script>
</body>
</html>
