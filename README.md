<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Lotería Pro Dashboard</title>

<style>
  :root {
    --bg: #050506;
    --panel: #0e0e10;
    --accent: #00f2fe;
    --card-white: #ffffff;
    --panel-border: rgba(255, 255, 255, 0.08);
  }

  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }

  body {
    margin: 0; padding: 0;
    font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
    background: var(--bg);
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    width: 100vw;
    overflow: hidden;
  }

  .app-grid {
    display: grid;
    grid-template-columns: 200px 1.5fr 350px; /* Adjusted widths for more history breathing room */
    width: 100%;
    height: 100%;
    align-items: center;
    padding: 25px;
    gap: 25px;
  }

  /* Side Panels */
  .side-panel {
    display: flex;
    flex-direction: column;
    height: 90vh;
    background: var(--panel);
    border-radius: 24px;
    border: 1px solid var(--panel-border);
    padding: 20px;
    box-shadow: inset 0 0 20px rgba(0,0,0,0.5);
  }

  .history-wall {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 3 columns to make them smaller and less clumped */
    gap: 8px;
    overflow-y: auto;
    flex-grow: 1;
    padding: 10px 5px;
    scrollbar-width: none;
  }

  /* Smaller History Cards */
  .side-card {
    width: 100%;
    aspect-ratio: 2/3;
    background: white;
    border-radius: 6px;
    border: 1.5px solid white;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
    overflow: hidden;
    transition: transform 0.2s;
  }
  .side-card img { width: 100%; height: 100%; object-fit: contain; }
  
  /* Highlight most recent in history */
  #fullHistory .side-card:first-child { 
    border-color: var(--accent); 
    box-shadow: 0 0 10px var(--accent); 
  }

  /* Center Stage */
  .center-stage {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  .main-card-area {
    width: 300px;
    height: 450px;
  }

  .card-wrap {
    width: 100%;
    height: 100%;
    background: white;
    border-radius: 20px;
    border: 8px solid white;
    box-shadow: 0 30px 60px rgba(0,0,0,0.8);
    transition: transform 0.3s ease-out, opacity 0.3s ease-out;
  }
  .card-wrap img { width: 100%; height: 100%; object-fit: contain; }

  .slide-out { opacity: 0; transform: translateY(20px) scale(0.95); }
  .slide-in { opacity: 1; transform: translateY(0) scale(1); }
  
  .shuffling { animation: shuffleEffect 0.1s infinite; }
  @keyframes shuffleEffect {
    0% { transform: translate(2px, 2px); }
    50% { transform: translate(-2px, -2px); }
    100% { transform: translate(0, 0); }
  }

  .title-header { 
    font-size: 0.7rem; 
    letter-spacing: 4px; 
    color: var(--accent); 
    margin-bottom: 15px; 
    text-transform: uppercase; 
    text-align: center;
    border-bottom: 1px solid var(--panel-border);
    padding-bottom: 10px;
  }

  .label-text { font-size: 2.5rem; font-weight: 900; margin: 10px 0; text-shadow: 0 0 20px rgba(0,242,254,0.3); }
  .deck-count { font-size: 0.9rem; opacity: 0.6; font-weight: 700; }

  /* Controls */
  .controls {
    background: var(--panel);
    padding: 20px;
    border-radius: 24px;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    width: 360px;
    border: 1px solid var(--panel-border);
  }

  button {
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.08);
    color: white;
    padding: 15px;
    border-radius: 14px;
    font-weight: 700;
    cursor: pointer;
    font-size: 0.8rem;
    transition: 0.2s;
  }
  button:hover { background: rgba(255,255,255,0.08); }

  .btn-start { grid-column: span 2; background: white; color: black; font-size: 1rem; border: none; }
  .btn-win { background: linear-gradient(45deg, #f9d423, #ff4e50); color: black; border: none; }

  @media (max-width: 1100px) {
    .app-grid { grid-template-columns: 1fr; overflow-y: auto; height: auto; }
    .side-panel { display: none; }
  }
</style>
</head>
<body>

<div class="app-grid">
  <div class="side-panel">
    <div class="title-header">Previous</div>
    <div id="prevList" class="history-wall" style="grid-template-columns: 1fr; gap: 15px;"></div>
  </div>

  <div class="center-stage">
    <div style="text-align: center;">
        <div class="title-header" style="border:none; margin:0;">Loteria Pro Max</div>
        <div class="deck-count" id="deckCount">54 REMAINING</div>
    </div>
    
    <div class="main-card-area" onclick="manualNext()">
      <div id="mainCard" class="card-wrap slide-out">
        <img id="mainImg" src="">
      </div>
    </div>

    <h1 class="label-text" id="mainLabel">READY</h1>

    <div class="controls">
      <button class="btn-start" id="startBtn" onclick="startGame()">▶ START GAME</button>
      <button onclick="activateCaller()">🎭 CALLER MODE</button>
      <button class="btn-win" onclick="triggerWinner()">🏆 LOTERÍA!</button>
      <button onclick="stopGame()">⏸ PAUSA</button>
      <button onclick="resetGame()">🔄 RESET</button>
      <button onclick="shuffleDeck()" style="grid-column: span 2;">🔀 SHUFFLE DECK</button>
      <div style="grid-column: span 2; padding: 10px 0; text-align: center;">
        <input type="range" id="speed" min="2" max="10" value="4" style="width:100%; accent-color: white;">
      </div>
    </div>
  </div>

  <div class="side-panel">
    <div class="title-header">Full History</div>
    <div id="fullHistory" class="history-wall"></div>
  </div>
</div>

<script>
const names = [
  "El Gallo","El Diablo","La Dama","El Catrin","El Paraguas","La Sirena","La Escalera","La Botella","El Barril","El Arbol",
  "El Melon","El Valiente","El Gorrito","La Muerte","La Pera","La Bandera","El Bandolon","El Violoncello","La Garza","El Pajaro",
  "La Mano","La Bota","La Luna","El Cotorro","El Borracho","El Negrito","El Corazon","La Sandia","El Tambor","El Camaron",
  "Las Jaras","El Musico","La Arana","El Soldado","La Estrella","El Cazo","El Mundo","El Apache","El Nopal","El Alacran",
  "La Rosa","La Calavera","La Campana","El Cantarito","El Venado","El Sol","La Corona","La Chalupa","El Pino","El Pescado",
  "La Palma","La Maceta","El Arpa","La Rana"
];

const reserved = ["El Musico", "El Catrin", "La Dama", "El Negrito"];

const cardImgs = {};
names.forEach((n, i) => {
  if (n === "El Catrin") {
    cardImgs[n] = `https://raw.githubusercontent.com/oceanlol/Loter-a-Pro-Mobile/main/CARD%204.jpg`;
  } else if (n === "El Paraguas") {
    cardImgs[n] = `https://raw.githubusercontent.com/oceanlol/Loter-a-Pro-Mobile/main/CARD%205.jpg`;
  } else if (n === "Las Jaras") {
    cardImgs[n] = `https://raw.githubusercontent.com/oceanlol/Loter-a-Pro-Mobile/main/card%20%2031.jpg`;
  } else {
    cardImgs[n] = `https://raw.githubusercontent.com/oceanlol/winning/main/card%20${i+1}.jpg`;
  }
});

let pool = [];
let rigQueue = [];
let history = [];
let gameLoop = null;

function talk(t) {
  speechSynthesis.cancel();
  const m = new SpeechSynthesisUtterance(t);
  m.lang = 'es-MX';
  m.rate = 0.95;
  speechSynthesis.speak(m);
}

function next() {
  let card = null;
  if (rigQueue.length > 0) {
    card = rigQueue.shift();
  } else if (pool.length > 0) {
    card = pool.shift();
  }

  if (card) {
    if (history.length > 0) updateSides(history[history.length - 1]);
    history.push(card);
    renderMain(card);
  } else {
    stopGame();
    document.getElementById('mainLabel').innerText = "FINISH";
  }
}

function renderMain(name) {
  const wrap = document.getElementById('mainCard');
  const img = document.getElementById('mainImg');
  const label = document.getElementById('mainLabel');
  
  wrap.classList.add('slide-out');
  
  setTimeout(() => {
    img.src = cardImgs[name];
    label.innerText = name;
    document.getElementById('deckCount').innerText = `${pool.length + rigQueue.length} REMAINING`;
    wrap.classList.remove('slide-out');
    wrap.classList.add('slide-in');
    talk(name);
  }, 150);
}

function updateSides(lastCard) {
  const left = document.getElementById('prevList');
  const cardDiv = document.createElement('div');
  cardDiv.className = 'side-card';
  cardDiv.innerHTML = `<img src="${cardImgs[lastCard]}">`;
  left.prepend(cardDiv);
  if (left.children.length > 4) left.removeChild(left.lastChild);

  const right = document.getElementById('fullHistory');
  const cardDiv2 = cardDiv.cloneNode(true);
  right.prepend(cardDiv2);
}

function shuffleDeck() {
  stopGame();
  const wrap = document.getElementById('mainCard');
  wrap.classList.add('shuffling');
  
  setTimeout(() => {
    wrap.classList.remove('shuffling');
    pool = names.filter(name => !reserved.includes(name)).sort(() => Math.random() - 0.5);
    history = [];
    document.getElementById('prevList').innerHTML = '';
    document.getElementById('fullHistory').innerHTML = '';
    document.getElementById('mainLabel').innerText = "READY";
    document.getElementById('deckCount').innerText = "50 REMAINING";
  }, 800);
}

function startGame() {
  stopGame();
  if (history.length === 0 && pool.length === 0) {
    pool = names.filter(name => !reserved.includes(name)).sort(() => Math.random() - 0.5);
  }
  talk("¡Corre y se va!");
  setTimeout(() => {
    next();
    gameLoop = setInterval(next, document.getElementById('speed').value * 1000);
  }, 800);
}

function stopGame() { clearInterval(gameLoop); }
function manualNext() { stopGame(); next(); }

function resetGame() {
  stopGame();
  pool = names.filter(name => !reserved.includes(name)).sort(() => Math.random() - 0.5);
  history = []; rigQueue = [];
  document.getElementById('prevList').innerHTML = '';
  document.getElementById('fullHistory').innerHTML = '';
  document.getElementById('mainCard').classList.add('slide-out');
  document.getElementById('mainLabel').innerText = "READY";
  document.getElementById('deckCount').innerText = "50 REMAINING";
}

function activateCaller() {
  if (rigQueue.length === 0 && !history.some(r => reserved.includes(r))) {
    rigQueue = [...reserved];
  }
}

function triggerWinner() {
  stopGame();
  talk("¡Lotería! ¡Buenas!");
  document.getElementById('mainLabel').innerText = "¡LOTERÍA!";
}

window.speechSynthesis.onvoiceschanged = () => { speechSynthesis.getVoices(); };
</script>
</body>
</html>
