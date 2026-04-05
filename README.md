<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Lotería Pro Dashboard</title>

<style>
  :root {
    --bg: #050506;
    --panel: #141417;
    --accent: #00f2fe;
    --card-white: #ffffff;
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
    grid-template-columns: 1fr 1.5fr 1fr; 
    width: 100%;
    height: 100%;
    align-items: center;
    padding: 20px;
    gap: 20px;
  }

  .side-panel {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 80%;
    justify-content: center;
    opacity: 0.5;
  }

  .history-wall {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    justify-content: center;
    overflow-y: auto;
    max-height: 100%;
    padding: 10px;
    scrollbar-width: none;
  }

  .side-card {
    width: 100px;
    height: 150px;
    background: white;
    border-radius: 8px;
    border: 3px solid white;
    box-shadow: 0 5px 15px rgba(0,0,0,0.5);
    flex-shrink: 0;
  }
  .side-card img { width: 100%; height: 100%; object-fit: contain; }

  .center-stage {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 15px;
  }

  .main-card-area {
    width: 280px;
    height: 420px;
  }

  .card-wrap {
    width: 100%;
    height: 100%;
    background: white;
    border-radius: 20px;
    border: 8px solid white;
    box-shadow: 0 30px 60px rgba(0,0,0,0.8);
    transition: all 0.4s cubic-bezier(0.23, 1, 0.32, 1);
  }
  .card-wrap img { width: 100%; height: 100%; object-fit: contain; }

  .slide-out { opacity: 0; transform: translateY(40px) scale(0.9); }
  .slide-in { opacity: 1; transform: translateY(0) scale(1); }

  .title { font-size: 0.8rem; letter-spacing: 3px; color: var(--accent); margin-bottom: 5px; text-transform: uppercase; }
  .label-text { font-size: 2.2rem; font-weight: 900; margin: 5px 0; }
  .deck-count { font-size: 0.9rem; opacity: 0.6; }

  .controls {
    background: var(--panel);
    padding: 20px;
    border-radius: 24px;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    width: 340px;
    border: 1px solid rgba(255,255,255,0.05);
  }

  button {
    background: rgba(255,255,255,0.05);
    border: 1px solid rgba(255,255,255,0.1);
    color: white;
    padding: 14px;
    border-radius: 14px;
    font-weight: 700;
    cursor: pointer;
    font-size: 0.85rem;
  }

  .btn-start { grid-column: span 2; background: white; color: black; font-size: 1rem; border: none; }
  .btn-win { background: linear-gradient(45deg, #f9d423, #ff4e50); color: black; border: none; }

  @media (max-width: 1000px) {
    .app-grid { grid-template-columns: 1fr; }
    .side-panel { display: none; }
  }
</style>
</head>
<body>

<div class="app-grid">
  <div class="side-panel">
    <div class="title">Anteriores</div>
    <div id="prevList" class="history-wall"></div>
  </div>

  <div class="center-stage">
    <div class="title">Loteria Pro Max</div>
    <div class="main-card-area" onclick="manualNext()">
      <div id="mainCard" class="card-wrap slide-out">
        <img id="mainImg" src="">
      </div>
    </div>
    <h1 class="label-text" id="mainLabel">LISTOS</h1>
    <div class="deck-count" id="deckCount">54 RESTANTES</div>

    <div class="controls">
      <button class="btn-start" onclick="startGame()">▶ INICIAR JUEGO</button>
      <button onclick="activateCaller()">🎭 CALLER MODE</button>
      <button class="btn-win" onclick="triggerWinner()">🏆 LOTERÍA!</button>
      <button onclick="stopGame()">⏸ PAUSA</button>
      <button onclick="resetGame()">🔄 REINICIAR</button>
      <div style="grid-column: span 2; padding: 10px 0;">
        <input type="range" id="speed" min="2" max="10" value="4" style="width:100%; accent-color: white;">
      </div>
    </div>
  </div>

  <div class="side-panel">
    <div class="title">Historial Completo</div>
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

// The Locked Cards
const reserved = ["El Musico", "El Catrin", "La Dama", "El Negrito"];

const cardImgs = {};
names.forEach((n, i) => cardImgs[n] = `https://raw.githubusercontent.com/oceanlol/winning/main/card%20${i+1}.jpg`);

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
    document.getElementById('mainLabel').innerText = "FIN";
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
    document.getElementById('deckCount').innerText = `${pool.length + rigQueue.length} RESTANTES`;
    wrap.classList.remove('slide-out');
    wrap.classList.add('slide-in');
    talk(name);
  }, 400);
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

function startGame() {
  stopGame();
  if (history.length === 0) {
    // PREVENT NORMAL CALLS: Remove reserved cards from the pool entirely
    pool = names.filter(name => !reserved.includes(name)).sort(() => Math.random() - 0.5);
  }
  talk("Corre y se va");
  setTimeout(() => {
    next();
    gameLoop = setInterval(next, document.getElementById('speed').value * 1000);
  }, 1000);
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
  document.getElementById('mainLabel').innerText = "LISTOS";
  document.getElementById('deckCount').innerText = "50 RESTANTES";
}

function activateCaller() {
  // ONLY way to call these 4 cards is through this button
  if (rigQueue.length === 0 && !history.some(r => reserved.includes(r))) {
    rigQueue = [...reserved];
  }
}

function triggerWinner() {
  stopGame();
  talk("¡Lotería!");
}

window.speechSynthesis.onvoiceschanged = () => { speechSynthesis.getVoices(); };
</script>
</body>
</html>
