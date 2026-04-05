<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Lotería Dual Dashboard</title>

<style>
  :root {
    --bg: #050506;
    --panel: #141417;
    --accent: #00f2fe;
    --card-white: #ffffff;
    --win-gold: #f9d423;
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
    min-height: 100vh;
    overflow: hidden;
  }

  /* Dashboard Layout */
  .dashboard {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: row; /* Desktop/Landscape mode */
    justify-content: space-around;
    align-items: center;
    padding: 20px;
    gap: 20px;
  }

  /* Vertical Stack for Mobile */
  @media (max-width: 800px) {
    .dashboard { flex-direction: column; overflow-y: auto; height: auto; padding: 10px; }
    .stage { width: 220px !important; height: 330px !important; }
  }

  /* Deck Sections */
  .deck-section {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
  }

  .stats {
    font-size: 0.7rem;
    font-weight: 800;
    color: var(--accent);
    letter-spacing: 2px;
    text-transform: uppercase;
  }

  /* Card Display */
  .stage {
    position: relative;
    width: 280px;
    height: 420px;
    perspective: 1000px;
  }

  .card-wrap {
    width: 100%;
    height: 100%;
    background: var(--card-white);
    border-radius: 18px;
    border: 6px solid white;
    box-shadow: 0 20px 50px rgba(0,0,0,0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.4s cubic-bezier(0.23, 1, 0.32, 1);
  }

  .card-wrap img { width: 100%; height: 100%; object-fit: contain; }

  .slide-out { opacity: 0; transform: translateY(50px) scale(0.9); }
  .slide-in { opacity: 1; transform: translateY(0) scale(1); }

  .card-label {
    font-size: 1.8rem;
    font-weight: 900;
    height: 40px;
    text-align: center;
    margin-top: 10px;
  }

  /* Center Controls */
  .controls-panel {
    width: 320px;
    background: var(--panel);
    padding: 25px;
    border-radius: 24px;
    display: flex;
    flex-direction: column;
    gap: 15px;
    border: 1px solid rgba(255,255,255,0.05);
    box-shadow: 0 10px 30px rgba(0,0,0,0.5);
  }

  .btn-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
  }

  button {
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.1);
    color: white;
    padding: 14px;
    border-radius: 12px;
    font-weight: 700;
    font-size: 0.8rem;
    cursor: pointer;
    transition: 0.2s;
  }

  button:active { transform: scale(0.95); }

  .btn-start {
    grid-column: span 2;
    background: white;
    color: black;
    font-size: 1rem;
    border: none;
  }

  .btn-win { background: linear-gradient(45deg, #f9d423, #ff4e50); border: none; color: black; }
  .btn-caller { background: #1a1a1f; border: 1px solid var(--accent); color: var(--accent); }

  .history-line {
    display: flex;
    gap: 5px;
    overflow-x: auto;
    width: 100%;
    scrollbar-width: none;
    padding: 5px 0;
  }
  .h-thumb { width: 30px; height: 45px; background: #fff; border-radius: 3px; flex-shrink: 0; overflow: hidden; }
  .h-thumb img { width: 100%; height: 100%; object-fit: contain; }

  .overlay {
    position: fixed; top:0; left:0; width:100%; height:100%;
    background: rgba(0,0,0,0.95);
    display: none; justify-content: center; align-items: center; z-index: 100;
  }
</style>
</head>
<body>

<div class="dashboard">
  
  <div class="deck-section">
    <div class="stats">DECK A: <span id="countA">54</span></div>
    <div class="stage">
      <div id="cardA" class="card-wrap slide-out">
        <img id="imgA" src="">
      </div>
    </div>
    <div class="card-label" id="labelA">LISTO</div>
    <div class="history-line" id="histA"></div>
  </div>

  <div class="controls-panel">
    <div style="text-align: center; color: var(--accent); font-weight: 900; letter-spacing: 2px;">DASHBOARD</div>
    
    <div style="text-align:center;">
      <span style="font-size:0.7rem; opacity:0.5;">INTERVALO: <span id="speedVal">4</span>s</span>
      <input type="range" id="speed" min="2" max="10" value="4" oninput="document.getElementById('speedVal').innerText=this.value" style="width:100%; accent-color: white;">
    </div>

    <div class="btn-grid">
      <button class="btn-start" onclick="startGame()">▶ INICIAR AMBOS</button>
      <button class="btn-caller" onclick="activateCaller()">🎭 CALLER MODE</button>
      <button class="btn-win" onclick="triggerWinner()">🏆 LOTERÍA!</button>
      <button onclick="stopGame()">⏸ PAUSA</button>
      <button onclick="resetGame()" style="grid-column: span 2;">🔄 REINICIAR DASHBOARD</button>
    </div>
  </div>

  <div class="deck-section">
    <div class="stats">DECK B: <span id="countB">54</span></div>
    <div class="stage">
      <div id="cardB" class="card-wrap slide-out">
        <img id="imgB" src="">
      </div>
    </div>
    <div class="card-label" id="labelB">LISTO</div>
    <div class="history-line" id="histB"></div>
  </div>

</div>

<div class="overlay" id="winOverlay">
  <div style="text-align:center;">
    <h1 style="color:var(--win-gold); font-size:4rem; margin:0;">¡BUENAS!</h1>
    <button onclick="closeWin()" style="background:white; color:black; padding: 20px 40px; border-radius: 10px; border:none; font-weight:900;">CERRAR</button>
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

let poolA = [], poolB = [];
let rigQueue = [];
let loop = null;

function say(t) {
  speechSynthesis.cancel();
  const m = new SpeechSynthesisUtterance(t);
  m.lang = 'es-MX';
  m.rate = 1.0;
  speechSynthesis.speak(m);
}

function next() {
  // Logic for Deck A
  let cardA = (rigQueue.length > 0) ? rigQueue.shift() : poolA.shift();
  if(cardA) {
    poolA = poolA.filter(c => c !== cardA);
    render('A', cardA);
  }

  // Logic for Deck B (Randomly delayed so they don't talk over each other perfectly)
  setTimeout(() => {
    let cardB = poolB.shift();
    if(cardB) {
        poolB = poolB.filter(c => c !== cardB);
        render('B', cardB);
    }
  }, 1500);
}

function render(id, name) {
  const wrap = document.getElementById(`card${id}`);
  const img = document.getElementById(`img${id}`);
  const label = document.getElementById(`label${id}`);
  const count = document.getElementById(`count${id}`);
  
  wrap.classList.add('slide-out');
  setTimeout(() => {
    img.src = imgs[name];
    label.innerText = name;
    count.innerText = (id === 'A' ? poolA.length : poolB.length);
    
    // History
    const h = document.createElement('div');
    h.className = 'h-thumb';
    h.innerHTML = `<img src="${imgs[name]}">`;
    document.getElementById(`hist${id}`).prepend(h);

    wrap.classList.remove('slide-out');
    wrap.classList.add('slide-in');
    say(name);
  }, 300);
}

function startGame() {
  stopGame();
  if(poolA.length === 0) poolA = [...names].sort(() => Math.random() - 0.5);
  if(poolB.length === 0) poolB = [...names].sort(() => Math.random() - 0.5);
  
  say("¡Corre y se va con los dos mazos!");
  next();
  loop = setInterval(next, document.getElementById('speed').value * 1000);
}

function stopGame() { clearInterval(loop); }

function resetGame() {
  stopGame();
  poolA = [...names].sort(() => Math.random() - 0.5);
  poolB = [...names].sort(() => Math.random() - 0.5);
  rigQueue = [];
  document.getElementById('histA').innerHTML = '';
  document.getElementById('histB').innerHTML = '';
  document.getElementById('cardA').classList.add('slide-out');
  document.getElementById('cardB').classList.add('slide-out');
}

function activateCaller() {
  // EL MUSICO, EL CATRIN, LA DAMA, EL NEGRITO
  rigQueue = ["El Musico", "El Catrin", "La Dama", "El Negrito"];
}

function triggerWinner() {
  stopGame();
  say("¡Lotería! ¡Buenas!");
  document.getElementById('winOverlay').style.display = 'flex';
}

function closeWin() { document.getElementById('winOverlay').style.display = 'none'; }
</script>
</body>
</html>
