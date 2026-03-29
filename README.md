<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>Lotería</title>

<style>
body {
    margin: 0;
    font-family: -apple-system, Arial;
    background: linear-gradient(#5b21b6, #4c1d95);
    color: white;
    text-align: center;
}

.header {
    padding: 12px;
    font-size: 20px;
    font-weight: bold;
}

.card-display {
    margin: 15px auto;
    width: 85%;
    max-width: 280px;
    height: 220px;
    background: #7c3aed;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

#cardName {
    font-size: 22px;
}

#countdown {
    font-size: 36px;
    font-weight: bold;
}

.play-btn {
    width: 65px;
    height: 65px;
    border-radius: 50%;
    background: #7c3aed;
    border: none;
    font-size: 26px;
    color: white;
}

.controls button {
    margin: 4px;
    padding: 8px 10px;
    border-radius: 8px;
    border: none;
    background: #6d28d9;
    color: white;
}

input {
    width: 50px;
    text-align: center;
}

#picker {
    position: fixed;
    bottom: 0;
    width: 100%;
    height: 70%;
    background: #111827;
    border-radius: 20px 20px 0 0;
    transform: translateY(100%);
    transition: transform 0.25s ease;
}

.handle {
    width: 50px;
    height: 5px;
    background: gray;
    border-radius: 10px;
    margin: 10px auto;
}

#grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 6px;
    padding: 10px;
    overflow-y: auto;
    height: calc(100% - 30px);
}

.card {
    background: #374151;
    padding: 10px;
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

<div class="header">Lotería</div>

<div class="card-display">
    <div id="cardName">---</div>
    <div id="countdown">0</div>
</div>

<button class="play-btn" onclick="startAuto()">▶</button>

<div class="controls">
    <button onclick="stopAuto()">Stop</button>
    <button onclick="resetGame()">Reset</button>
    <button onclick="openPicker()">Cards</button>
    <br>
    Speed: <input id="speed" value="4">s
</div>

<div id="picker">
    <div class="handle"></div>
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
let countdownInterval = null;
let timeLeft = 0;

// BUILD GRID
const grid = document.getElementById("grid");

deck.forEach(card => {
    const div = document.createElement("div");
    div.className = "card";
    div.innerText = card;

    div.addEventListener("click", () => {
        if (selectedSet.has(card)) {
            selectedSet.delete(card);
            div.classList.remove("selected");
        } else {
            selectedSet.add(card);
            div.classList.add("selected");
        }
    });

    grid.appendChild(div);
});

// PICKER
const picker = document.getElementById("picker");

function openPicker() {
    picker.style.transform = "translateY(0)";
}

function closePicker() {
    picker.style.transform = "translateY(100%)";
}

// DRAG
let startY = 0;

picker.addEventListener("touchstart", e => {
    startY = e.touches[0].clientY;
});

picker.addEventListener("touchend", e => {
    let endY = e.changedTouches[0].clientY;
    if (endY - startY > 80) closePicker();
});

// SETUP (FIXED)
function setup() {
    if (selectedSet.size > 0) {
        randomPool = [...selectedSet];
    } else {
        randomPool = [...deck];
    }

    randomPool = randomPool.sort(() => Math.random() - 0.5);
}

// SPEAK
function speak(text) {
    speechSynthesis.cancel();

    const msg = new SpeechSynthesisUtterance(text);
    msg.lang = "es-MX";

    const voices = speechSynthesis.getVoices();
    const spanish = voices.find(v => v.lang.includes("es"));
    if (spanish) msg.voice = spanish;

    speechSynthesis.speak(msg);
}

// NEXT CARD
function nextCard() {
    if (randomPool.length === 0) setup();

    const card = randomPool.shift();
    document.getElementById("cardName").innerText = card;

    speak(card);
    startCountdown();
}

// COUNTDOWN
function startCountdown() {
    clearInterval(countdownInterval);

    timeLeft = getSpeed();
    updateCountdown();

    countdownInterval = setInterval(() => {
        timeLeft--;
        updateCountdown();
        if (timeLeft <= 0) clearInterval(countdownInterval);
    }, 1000);
}

function updateCountdown() {
    document.getElementById("countdown").innerText = timeLeft;
}

function getSpeed() {
    return parseInt(document.getElementById("speed").value) || 4;
}

// START
function startAuto() {
    speechSynthesis.resume();

    stopAuto();
    setup();
    closePicker();

    nextCard();

    autoInterval = setInterval(() => {
        nextCard();
    }, getSpeed() * 1000);
}

// STOP
function stopAuto() {
    clearInterval(autoInterval);
    clearInterval(countdownInterval);
    autoInterval = null;
}

// RESET
function resetGame() {
    stopAuto();
    document.getElementById("cardName").innerText = "---";
    document.getElementById("countdown").innerText = "0";
}
</script>

</body>
</html>
