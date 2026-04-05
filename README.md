<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Lotería Pro MAX</title>

<style>
  :root {
    --bg: #0a0a0c;
    --panel: #1a1a1f;
    --accent: #00f2fe;
    --card-white: #ffffff;
    --btn-text: #ffffff;
  }

  * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }

  body {
    margin: 0; padding: 0;
    font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
    background: var(--bg);
    color: white;
    display: flex;
    justify-content: center;
    height: 100vh;
    overflow: hidden;
  }

  .container {
    width: 100%;
    max-width: 400px;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    justify-content: space-between;
  }

  .stats {
    width: 100%;
    display: flex;
    justify-content: space-between;
    font-size: 0.75rem;
    font-weight: 800;
    color: rgba(255,255,255,0.3);
    text-transform: uppercase;
    letter-spacing: 1.5px;
  }

  /* Card Stage */
  .stage {
    position: relative;
    width: 260px;
    height: 380px;
    perspective: 1000px;
    margin: 10px 0;
  }

  .card-wrap {
    width: 100%;
    height: 100%;
    background: var(--card-white);
    border-radius: 22px;
    border: 8px solid white;
    box-shadow: 0 30px 60px rgba(0,0,0,0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.5s cubic-bezier(0.23, 1, 0.32, 1);
  }

  .card-wrap img {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }

  /* Animation States */
  .slide-out { opacity: 0; transform: translateX(120px) rotate(8deg) scale(0.9); }
  .slide-in { opacity: 1; transform: translateX(0) rotate(0deg) scale(1); }

  #cardName {
    font-size: 2.2rem;
    font-weight: 900;
    margin: 0;
    text-align: center;
    background: linear-gradient(180deg, #fff 0%, #bbb 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    height: 45px;
  }

  /* History Dock */
  .history-bar {
    display: flex;
    gap: 8px;
    overflow-x: auto;
    width: 100%;
    padding: 10px 0;
    scrollbar-width: none;
  }
  .history-bar::-webkit-scrollbar { display: none; }

  .hist-card {
    width: 45px; height: 65px;
    background: white;
    border-radius: 6px;
    flex-shrink: 0;
    border: 2px solid white;
    overflow: hidden;
    box-shadow: 0 4px 10px rgba(0,0,0,0.4);
  }
  .hist-card img { width: 100%; height: 100%; object-fit: contain; }

  /* Controls */
  .btn-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    width: 100%;
  }

  button {
    background: var(--panel);
    border: 1px solid rgba(255,255,255,0.05);
    color: var(--btn-text);
    padding: 16px;
    border-radius: 16px;
    font-weight: 700;
    font-size: 0.85rem;
    cursor: pointer;
    transition: 0.2s;
  }

  button:active { transform: scale(0.95); background: #2a2a30; }

  .btn-start {
    grid-column: span 2;
    background: white;
    color: black;
    font-size: 1.1rem;
    border: none;
  }

  input[type=range] {
    width: 100%;
    accent-color: #fff;
    margin-top: 10px;
  }
</style>
</head>
<body>

<div class="container">
  <div class="stats">
    <span>LOTERÍA PRO MAX</span>
    <span id="deckLeft">54 CARTAS</span>
  </div>

  <div class="stage" onclick="manualNext()">
    <div id="cardEl" class="card-wrap slide-out">
      <img id="cardImg" src="">
    </div>
  </div>

  <h1 id="cardName">LISTOS</h1>

  <div class="history-bar" id="history"></div>

  <div style="width:100%; text-align:center;">
    <span style="font-size:0.7rem; opacity:0.5;">VELOCIDAD: <span id="speedVal">4</span>s</span>
    <input type="range" id="speed" min="2" max="10" value="4" oninput="document.getElementById('speedVal').innerText=this.value">
  </div>

  <div class="btn-grid">
    <button class="btn-start" onclick="startGame()">▶ INICIAR JUEGO</button>
    <button onclick="activateCaller()">🎭 CALLER MODE</button>
    <button onclick="stopGame()">⏸ PAUSA</button>
    <button onclick="resetGame()" style="grid-column: span 2;">🔄 REINICIAR</button>
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

const imgs = {};
names.forEach((name, i) => imgs[name] = `https://raw.githubusercontent.com/oceanlol/winning/main/card%20${i+1}.jpg`);

let pool = [];
let rigQueue = [];
let drawn = [];
let loop = null;

function say(t) {
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
    // Logic to ensure NO DUPES:
    pool = pool.filter(c => c !== card);
  } else if (pool.length > 0) {
    card = pool.shift();
  }

  if (card) {
    drawn.push(card);
    render(card);
  } else {
    stopGame();
    document.getElementById('cardName').innerText = "FIN";
  }
}

function render(name) {
  const el = document.getElementById('cardEl');
  const img = document.getElementById('cardImg');
  const title = document.getElementById('cardName');
  
  el.classList.add('slide-out');
  el.classList.remove('slide-in');

  setTimeout(() => {
    img.src = imgs[name];
    title.innerText = name;
    document.getElementById('deckLeft').innerText = `${pool.length} RESTANTES`;

    // Visual History
    const div = document.createElement('div');
    div.className = 'hist-card';
    div.innerHTML = `<img src="${imgs[name]}">`;
    document.getElementById('history').prepend(div);

    el.classList.remove('slide-out');
    el.classList.add('slide-in');
    say(name);
  }, 400);
}

function startGame() {
  stopGame();
  if (drawn.length === 0) pool = [...names].sort(() => Math.random() - 0.5);
  say("¡Corre y se va!");
  setTimeout(() => {
    next();
    loop = setInterval(next, document.getElementById('speed').value * 1000);
  }, 1200);
}

function stopGame() { clearInterval(loop); }

function manualNext() {
  stopGame();
  next();
}

function resetGame() {
  stopGame();
  pool = [...names].sort(() => Math.random() - 0.5);
  drawn = [];
  rigQueue = [];
  document.getElementById('history').innerHTML = '';
  document.getElementById('cardEl').classList.add('slide-out');
  document.getElementById('cardName').innerText = "LISTOS";
  document.getElementById('deckLeft').innerText = "54 RESTANTES";
}

function activateCaller() {
  // Silent Rig - No alerts.
  // Set your winning sequence here
  rigQueue = ["La Chalupa", "El Borracho", "La Sirena", "El Corazon"];
}

window.speechSynthesis.onvoiceschanged = () => { speechSynthesis.getVoices(); };
</script>
</body>
</html>
