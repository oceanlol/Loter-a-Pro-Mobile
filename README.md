<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lotería Pro Mobile</title>

<style>
body {
    margin: 0;
    font-family: 'Segoe UI', Arial;
    background: black;
    color: white;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 40px;
}

#particles {
    position: fixed;
    width: 100%;
    height: 100%;
    z-index: 0;
}

.container {
    position: relative;
    z-index: 1;
    width: 90%;
    max-width: 400px;
    text-align: center;
}

.card-box {
    width: 100%;
    min-height: 300px;
    perspective: 1000px;
    margin-bottom: 15px;
    display: flex;
    flex-direction: column;
    align-items: center;
}

/* push card a bit down so name is visible */
.card-inner {
    width: 90%;
    height: 220px;
    border-radius: 20px;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.6s;
    margin-bottom: 10px;
}

.card-front, .card-back {
    position: absolute;
    width: 100%;
    height: 100%;
    border-radius: 20px;
    backface-visibility: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgba(255,255,255,0.05);
    box-shadow: 0 0 0 rgba(255,255,255,0);
    transition: box-shadow 0.3s;
}

.card-front img {
    width: 90%;
    max-height: 200px;
    object-fit: contain;
    border-radius: 15px;
}

.card-back {
    background: rgba(255,255,255,0.08);
    transform: rotateY(180deg);
    font-size: 28px;
    font-weight: bold;
}

#cardName {
    font-size: 28px;
    font-weight: bold;
    margin-top: 10px;
}
#countdown {
    font-size: 40px;
    margin-bottom: 5px;
}

button {
    background: rgba(255,255,255,0.08);
    border: none;
    color: white;
    padding: 14px;
    margin: 8px 0;
    border-radius: 12px;
    width: 100%;
    font-size: 18px;
    transition: transform 0.1s;
}

button:active {
    transform: scale(0.95);
}

.history {
    margin-top: 10px;
    max-height: 180px;
    overflow-y: auto;
    font-size: 14px;
    text-align: left;
    background: rgba(255,255,255,0.05);
    padding: 8px;
    border-radius: 10px;
    width: 100%;
}

.history div {
    padding: 4px;
    border-bottom: 1px solid rgba(255,255,255,0.1);
}

/* Bounce animation for card after flip */
@keyframes bounce {
    0% { transform: translateY(0); }
    30% { transform: translateY(-10px); }
    50% { transform: translateY(0); }
    70% { transform: translateY(-5px); }
    100% { transform: translateY(0); }
}
.bounce {
    animation: bounce 0.4s;
}

/* Glow effect while flipping */
.glow {
    box-shadow: 0 0 20px 5px rgba(255,255,255,0.7);
}
</style>
</head>

<body>

<canvas id="particles"></canvas>

<div class="container">

<div class="card-box">
    <div class="card-inner" id="cardInner">
        <div class="card-front">
            <img id="cardImage" src="https://via.placeholder.com/150?text=No+Image" alt="Card Image">
        </div>
        <div class="card-back">Lotería</div>
    </div>
    <div id="cardName">---</div>
    <div id="countdown">0</div>
</div>

<button onclick="startGame()">▶ Start</button>
<button onclick="activateCaller()">Caller Mode</button>
<button onclick="stopAuto()">Stop</button>
<button onclick="resetGame()">Reset</button>

<div>
Speed:
<input type="range" id="speed" min="1" max="10" value="4">
</div>

<div class="history" id="history"></div>

</div>

<script>
// Deck
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

let callerList = ["El Musico","El Catrin","La Dama","El Negrito"];
let randomPool = [];
let injectQueue = [];
let blendMode = false;
let callCount = 0;
let autoInterval = null;
let countdownInterval = null;
let selectedVoice = null;
let isSpeaking = false;

// Card images
const cardImages = {
"El Gallo":"https://i.imgur.com/z1nEFgO.jpeg",
"El Diablo":"https://i.imgur.com/rxWv6vK.jpeg",
"La Dama":"https://i.imgur.com/XgDqiNl.jpeg",
"El Catrin":"https://i.imgur.com/PpondFd.jpeg",
"El Paraguas":"https://i.imgur.com/uRI9iNC.jpeg",
"La Sirena":"https://i.imgur.com/8YJMkhu.jpeg",
"La Escalera":"https://i.imgur.com/2uBrCdL.jpeg",
"La Botella":"https://i.imgur.com/HyX7qfV.jpeg",
"El Barril":"https://i.imgur.com/r3b4IY2.jpeg",
"El Arbol":"https://i.imgur.com/3MLbB7C.jpeg",
"El Melon":"https://i.imgur.com/frSRTj6.jpeg",
"El Valiente":"https://i.imgur.com/WxwS83V.jpeg",
"El Gorrito":"https://i.imgur.com/4CqxyVO.jpeg",
"La Muerte":"https://i.imgur.com/izrshro.jpeg",
"La Pera":"https://i.imgur.com/3bUxEox.jpeg",
"La Bandera":"https://i.imgur.com/pQP8NNO.jpeg",
"El Bandolon":"https://i.imgur.com/dxitjQN.jpeg",
"El Violoncello":"https://i.imgur.com/ogjNjnA.jpeg",
"La Garza":"https://i.imgur.com/vMKaIdv.jpeg",
"El Pajaro":"https://i.imgur.com/uEiQBFO.jpeg",
"La Mano":"https://i.imgur.com/VkKLlqr.jpeg",
"La Bota":"https://i.imgur.com/NDBe5OH.jpeg",
"La Luna":"https://i.imgur.com/e1fPz9R.jpeg",
"El Cotorro":"https://i.imgur.com/gFq9O8O.jpeg",
"El Borracho":"https://i.imgur.com/Y2TjiRl.jpeg",
"El Negrito":"https://i.imgur.com/dCe9v1p.jpeg",
"El Corazon":"https://i.imgur.com/q41EefM.jpeg",
"La Sandia":"https://i.imgur.com/NDqOGAp.jpeg",
"El Tambor":"https://i.imgur.com/vP4xdAG.jpeg",
"El Camaron":"https://i.imgur.com/51Y9ucg.jpeg",
"Las Jaras":"https://i.imgur.com/6j4KzKP.jpeg",
"El Musico":"https://i.imgur.com/Dkp9VKW.jpeg",
"La Arana":"https://i.imgur.com/b7iKPIi.jpeg",
"El Soldado":"https://i.imgur.com/Q8G5bt3.jpeg",
"La Estrella":"https://i.imgur.com/U5RrzEG.jpeg",
"El Cazo":"https://i.imgur.com/aGCKp5q.jpeg",
"El Mundo":"https://i.imgur.com/tPYzCG9.jpeg",
"El Apache":"https://i.imgur.com/UB3jKbQ.jpeg",
"El Nopal":"https://i.imgur.com/ZJvkH3F.jpeg",
"El Alacran":"https://i.imgur.com/AVh1tlK.jpeg",
"La Rosa":"https://i.imgur.com/Z35ozqa.jpeg",
"La Calavera":"https://i.imgur.com/gsQI9gG.jpeg",
"La Campana":"https://i.imgur.com/54E5hs5.jpeg",
"El Cantarito":"https://i.imgur.com/92ajsYs.jpeg",
"El Venado":"https://i.imgur.com/Ty0a9fh.jpeg",
"El Sol":"https://i.imgur.com/0pWa6PF.jpeg",
"La Corona":"https://i.imgur.com/vKPQeWp.jpeg",
"La Chalupa":"https://i.imgur.com/WRjLeQ9.jpeg",
"El Pino":"https://i.imgur.com/3g3mZ1j.jpeg",
"El Pescado":"https://i.imgur.com/bjpjmtT.jpeg",
"La Palma":"https://i.imgur.com/tIXKSL6.jpeg",
"La Maceta":"https://i.imgur.com/XIXd4vn.jpeg",
"El Arpa":"https://i.imgur.com/kCE4eI4.jpeg",
"La Rana":"https://i.imgur.com/FrLUeDC.jpeg"
};

// SPEECH
function loadVoice() {
    let voices = speechSynthesis.getVoices();
    selectedVoice =
        voices.find(v => v.lang === "es-MX") ||
        voices.find(v => v.lang.startsWith("es")) ||
        voices[0];
}
speechSynthesis.onvoiceschanged = loadVoice;

function speak(text, callback) {
    let msg = new SpeechSynthesisUtterance(text);
    if (!selectedVoice) loadVoice();
    if (selectedVoice) msg.voice = selectedVoice;
    msg.rate = 0.8;
    msg.pitch = 1;
    isSpeaking = true;
    msg.onend = () => {
        isSpeaking = false;
        if (callback) callback();
    };
    speechSynthesis.speak(msg);
}

// SETUP
function setup() {
    randomPool = [...deck].sort(() => Math.random() - 0.5);
}

// HISTORY
let calledSet = new Set();
function addToHistory(card) {
    if (calledSet.has(card)) return;
    calledSet.add(card);
    let history = document.getElementById("history");
    let item = document.createElement("div");
    item.innerText = card;
    history.prepend(item);
}

// DISPLAY CARD WITH FLIP + BOUNCE + GLOW
function displayCard(card) {
    const inner = document.getElementById("cardInner");
    const img = document.getElementById("cardImage");

    inner.classList.add("glow");
    inner.style.transform = "rotateY(180deg)";

    setTimeout(() => {
        img.src = cardImages[card] || "https://via.placeholder.com/150?text=No+Image";
        img.alt = card;
        document.getElementById("cardName").innerText = card;
        speak(card);
        addToHistory(card);

        inner.style.transform = "rotateY(0deg)";
        inner.classList.add("bounce");
        setTimeout(() => { inner.classList.remove("bounce"); inner.classList.remove("glow"); }, 400);
    }, 400);

    startCountdown();
}

// NEXT CARD
function nextCard() {
    if (isSpeaking) return;
    let card;
    if (blendMode && injectQueue.length > 0 && callCount % 2 === 1) {
        card = injectQueue.shift();
    } else {
        if (randomPool.length === 0) return;
        card = randomPool.shift();
    }
    displayCard(card);
    callCount++;
}

// START GAME
function startGame() {
    stopAuto();
    setup();
    callCount = 0;
    calledSet.clear();
    document.getElementById("history").innerHTML = "";
    let count = 1;
    let intro = setInterval(() => {
        if (count <= 3) {
            document.getElementById("cardName").innerText = count;
            speak(String(count));
            count++;
        } else {
            clearInterval(intro);
            speak("Bienvenidos a la Lotería", () => {
                nextCard();
                autoInterval = setInterval(nextCard, getSpeed()*1000);
            });
        }
    }, 1000);
}

// CALLER MODE
function activateCaller() {
    injectQueue = [...callerList].sort(() => Math.random() - 0.5);
    blendMode = true;
}

// CONTROLS
function stopAuto() {
    clearInterval(autoInterval);
    clearInterval(countdownInterval);
}

function resetGame() {
    stopAuto();
    document.getElementById("cardName").innerText = "---";
    document.getElementById("countdown").innerText = "0";
    document.getElementById("history").innerHTML = "";
    const img = document.getElementById("cardImage");
    img.src = "https://via.placeholder.com/150?text=No+Image";
    const inner = document.getElementById("cardInner");
    inner.style.transform = "rotateY(0deg)";
    calledSet.clear();
}

function getSpeed() {
    return parseInt(document.getElementById("speed").value);
}

// TIMER
function startCountdown() {
    clearInterval(countdownInterval);
    let time = getSpeed();
    document.getElementById("countdown").innerText = time;
    countdownInterval = setInterval(() => {
        time--;
        document.getElementById("countdown").innerText = time;
        if (time <= 0) clearInterval(countdownInterval);
    }, 1000);
}

// PARTICLES
const canvas = document.getElementById("particles");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
let particles = [];
for(let i=0;i<80;i++){
    particles.push({x:Math.random()*canvas.width, y:Math.random()*canvas.height, r:Math.random()*2, d:Math.random()*1});
}
function drawParticles(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.fillStyle = "white";
    particles.forEach(p=>{
        ctx.beginPath();
        ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
        ctx.fill();
        p.y += p.d;
        if(p.y>canvas.height){p.y=0;p.x=Math.random()*canvas.width;}
    });
    requestAnimationFrame(drawParticles);
}
drawParticles();
</script>

</body>
</html>
