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
    overflow-x: hidden;
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
    min-height: 250px;
    perspective: 1000px;
    margin-bottom: 15px;
}

.card-inner {
    width: 100%;
    height: 100%;
    border-radius: 20px;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.8s;
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
}

.card-front img {
    width: 90%;
    max-height: 180px;
    object-fit: contain;
    border-radius: 15px;
}

.card-back {
    background: rgba(255,255,255,0.08);
    transform: rotateY(180deg);
    font-size: 28px;
    font-weight: bold;
}

#cardName { font-size: 24px; font-weight: bold; margin-top: 10px; }
#countdown { font-size: 36px; margin-bottom: 5px; }

button {
    background: rgba(255,255,255,0.08);
    border: none;
    color: white;
    padding: 18px;
    margin: 8px 0;
    border-radius: 12px;
    width: 100%;
    font-size: 20px;
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
</style>
</head>

<body>

<canvas id="particles"></canvas>

<div class="container">

<div class="card-box">
    <div class="card-inner" id="cardInner">
        <div class="card-front">
            <img id="cardImage" src="https://via.placeholder.com/150?text=No+Image">
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
// 🔥 UNLOCK AUDIO ON MOBILE
document.body.addEventListener("click", () => {
    speechSynthesis.resume();
});

// DECK
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

let randomPool = [];
let autoInterval = null;
let countdownInterval = null;
let isSpeaking = false;

// SPEAK FIXED
function speak(text, callback) {
    if (!window.speechSynthesis) return;

    let msg = new SpeechSynthesisUtterance(text);
    let voices = speechSynthesis.getVoices();

    let voice =
        voices.find(v => v.lang === "es-MX") ||
        voices.find(v => v.lang.startsWith("es")) ||
        voices[0];

    if (voice) msg.voice = voice;

    msg.rate = 0.85;

    msg.onend = () => {
        isSpeaking = false;
        if (callback) callback();
    };

    isSpeaking = true;
    speechSynthesis.cancel();
    speechSynthesis.speak(msg);
}

// SETUP
function setup() {
    randomPool = [...deck].sort(() => Math.random() - 0.5);
}

// DISPLAY
function displayCard(card) {
    const inner = document.getElementById("cardInner");
    const img = document.getElementById("cardImage");

    inner.style.transform = "rotateY(180deg)";

    setTimeout(() => {
        img.src = "https://via.placeholder.com/150?text=" + card;
        document.getElementById("cardName").innerText = card;

        speak(card);
        inner.style.transform = "rotateY(0deg)";
        inner.classList.add("bounce");

        setTimeout(()=>inner.classList.remove("bounce"),400);
    }, 400);

    startCountdown();
}

// NEXT CARD
function nextCard() {
    if (isSpeaking) return;
    if (randomPool.length === 0) return;

    let card = randomPool.shift();
    displayCard(card);
}

// START
function startGame() {
    speechSynthesis.resume();

    stopAuto();
    setup();

    let count = 1;

    let intro = setInterval(() => {
        if (count <= 3) {
            document.getElementById("cardName").innerText = count;
            speak(String(count));
            count++;
        } else {
            clearInterval(intro);

            speak("Bienvenidos", () => {
                nextCard();
                autoInterval = setInterval(nextCard, getSpeed()*1000);
            });
        }
    }, 1000);
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
}

// SPEED
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

function resizeCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
}
window.addEventListener("resize", resizeCanvas);
resizeCanvas();

let particles = [];
for(let i=0;i<60;i++){
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
        if(p.y>canvas.height){
            p.y=0;
            p.x=Math.random()*canvas.width;
        }
    });

    requestAnimationFrame(drawParticles);
}
drawParticles();
</script>

</body>
</html>
