<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lotería</title>

<style>
body {
    margin: 0;
    font-family: Arial;
    background: linear-gradient(#5b21b6, #4c1d95);
    color: white;
    text-align: center;
}

/* HEADER */
.header {
    padding: 15px;
    font-size: 22px;
    font-weight: bold;
}

/* CARD DISPLAY */
.card-display {
    margin: 20px auto;
    width: 240px;
    height: 320px;
    background: #7c3aed;
    border-radius: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 22px;
    box-shadow: 0 10px 25px rgba(0,0,0,0.4);
}

/* PLAY BUTTON */
.play-btn {
    width: 70px;
    height: 70px;
    border-radius: 50%;
    background: #7c3aed;
    border: none;
    font-size: 28px;
    color: white;
    cursor: pointer;
}

/* CONTROLS */
.controls {
    margin-top: 15px;
}

button.small {
    padding: 8px 12px;
    margin: 5px;
    border-radius: 10px;
    border: none;
    background: #6d28d9;
    color: white;
}

/* CARD PICKER PANEL */
#picker {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    background: #111827;
    border-top-left-radius: 15px;
    border-top-right-radius: 15px;
    max-height: 0;
    overflow: hidden;
    transition: 0.3s;
}

/* GRID */
#grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 6px;
    padding: 10px;
}

/* MOBILE STACK */
@media (max-width: 500px) {
    #grid {
        grid-template-columns: repeat(3, 1fr);
    }
}

.card {
    background: rgba(255,255,255,0.1);
    padding: 8px;
    border-radius: 8px;
    font-size: 11px;
}

.selected {
    background: #22c55e;
    color: black;
}
</style>
</head>

<body>

<div class="header">Baraja de Lotería</div>

<div id="current" class="card-display">---</div>

<button class="play-btn" onclick="startAuto()">▶</button>

<div class="controls">
    <button class="small" onclick="stopAuto()">Stop</button>
    <button class="small" onclick="resetGame()">Reset</button>
    <button class="small" onclick="togglePicker()">Select Cards</button>
</div>

<!-- PICKER PANEL -->
<div id="picker">
    <div id="grid"></div>
</div>

<script>
const deck = [
"El Gallo","El Diablo","La Dama","El Catrin","El Paraguas",
"La Sirena","La Escalera","La Botella","El Barril","El Arbol",
"El Melon","El Valiente","El Gorrito","La Muerte","La Pera",
"La Bandera","El Bandolon","El Violoncello","La Garza","El Pajaro",
"La Mano","La Bota","La Luna","El Cotorro","El Borracho",
"El Negrito","El Corazon","La Sandia","El Tambor","El Camaron",
"Las Jaras","El Musico","La Arana","El Soldado","La Estrella",
"El Cazo","El Mundo","El Apache","El Nopal","El Alacran",
"La Rosa","La Calavera","La Campana","El Cantarito","El Venado",
"El Sol","La Corona","La Chalupa","El Pino","El Pescado",
"La Palma","La Maceta","El Arpa","La Rana"
];

let selectedSet = new Set();
let randomPool = [];
let autoInterval = null;
let callCount = 0;
let pickerOpen = false;

// BUILD GRID
const grid = document.getElementById("grid");

deck.forEach(card => {
    const div = document.createElement("div");
    div.className = "card";
    div.innerText = card;

    div.onclick = () => {
        if (selectedSet.has(card)) {
            selectedSet.delete(card);
            div.classList.remove("selected");
        } else {
            selectedSet.add(card);
            div.classList.add("selected");
        }
    };

    grid.appendChild(div);
});

// TOGGLE PICKER
function togglePicker() {
    const picker = document.getElementById("picker");
    pickerOpen = !pickerOpen;

    picker.style.maxHeight = pickerOpen ? "60%" : "0";
}

// SETUP
function setupPools() {
    randomPool = [...deck].sort(() => Math.random() - 0.5);
}

// SPEAK
function speak(text) {
    speechSynthesis.cancel();
    const msg = new SpeechSynthesisUtterance(text);
    msg.lang = "es-MX";
    speechSynthesis.speak(msg);
}

// NEXT CARD
function nextCard() {
    if (randomPool.length === 0) setupPools();

    const card = randomPool.shift();

    document.getElementById("current").innerText = card;
    speak(card);
}

// AUTO PLAY
function startAuto() {
    if (autoInterval) return;

    setupPools();
    nextCard();

    autoInterval = setInterval(nextCard, 4000);
}

function stopAuto() {
    clearInterval(autoInterval);
    autoInterval = null;
}

function resetGame() {
    stopAuto();
    document.getElementById("current").innerText = "---";
}
</script>

</body>
</html>
