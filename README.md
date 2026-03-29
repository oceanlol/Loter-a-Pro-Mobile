<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lotería Pro MAX</title>

<style>
body {
    margin: 0;
    font-family: 'Segoe UI', Arial;
    background: radial-gradient(circle at top, #111, #000);
    color: white;
    overflow-x: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 30px;
}

/* Glow particles */
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

/* CARD */
.card-box {
    width: 100%;
    min-height: 260px;
    perspective: 1000px;
}

.card-inner {
    width: 100%;
    height: 100%;
    border-radius: 20px;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.6s;
    box-shadow: 0 0 25px rgba(255,255,255,0.15);
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
    backdrop-filter: blur(10px);
}

.card-front img {
    width: 90%;
    max-height: 190px;
    object-fit: contain;
    border-radius: 15px;
}

.card-back {
    background: rgba(255,255,255,0.1);
    transform: rotateY(180deg);
    font-size: 28px;
}

#cardName {
    font-size: 26px;
    margin-top: 10px;
}

#countdown {
    font-size: 34px;
}

/* BUTTONS */
button {
    background: rgba(255,255,255,0.08);
    border: none;
    color: white;
    padding: 18px;
    margin: 6px 0;
    border-radius: 14px;
    width: 100%;
    font-size: 18px;
}

button:active {
    transform: scale(0.95);
}

/* HISTORY */
.history {
    margin-top: 10px;
    max-height: 150px;
    overflow-y: auto;
    font-size: 14px;
    background: rgba(255,255,255,0.05);
    border-radius: 10px;
    padding: 8px;
}

/* ANIMATION */
@keyframes bounce {
    0%{transform:translateY(0)}
    30%{transform:translateY(-10px)}
    100%{transform:translateY(0)}
}
.bounce { animation: bounce 0.4s; }
</style>
</head>

<body>

<canvas id="particles"></canvas>

<div class="container">

<div class="card-box" id="tapZone">
    <div class="card-inner" id="cardInner">
        <div class="card-front">
            <img id="cardImage" src="">
        </div>
        <div class="card-back">Lotería</div>
    </div>
    <div id="cardName">Tap Start</div>
    <div id="countdown">0</div>
</div>

<button onclick="startGame()">▶ Start</button>
<button onclick="activateCaller()">Caller Mode</button>
<button onclick="stopAuto()">Stop</button>
<button onclick="resetGame()">Reset</button>

<input type="range" id="speed" min="1" max="10" value="4">

<div class="history" id="history"></div>

</div>

<script>
// AUDIO UNLOCK
document.body.addEventListener("click", ()=>speechSynthesis.resume());

// FULL IMAGE SET
const base = "https://loteriamx.com/images/cards/";
const names = [
"gallo","diablo","dama","catrin","paraguas","sirena","escalera","botella","barril","arbol",
"melon","valiente","gorrito","muerte","pera","bandera","bandolon","violoncello","garza","pajaro",
"mano","bota","luna","cotorro","borracho","negrito","corazon","sandia","tambor","camaron",
"jaras","musico","arana","soldado","estrella","cazo","mundo","apache","nopal","alacran",
"rosa","calavera","campana","cantarito","venado","sol","corona","chalupa","pino","pescado",
"palma","maceta","arpa","rana"
];

const deck = [
"El Gallo","El Diablo","La Dama","El Catrin","El Paraguas","La Sirena","La Escalera","La Botella","El Barril","El Arbol",
"El Melon","El Valiente","El Gorrito","La Muerte","La Pera","La Bandera","El Bandolon","El Violoncello","La Garza","El Pajaro",
"La Mano","La Bota","La Luna","El Cotorro","El Borracho","El Negrito","El Corazon","La Sandia","El Tambor","El Camaron",
"Las Jaras","El Musico","La Arana","El Soldado","La Estrella","El Cazo","El Mundo","El Apache","El Nopal","El Alacran",
"La Rosa","La Calavera","La Campana","El Cantarito","El Venado","El Sol","La Corona","La Chalupa","El Pino","El Pescado",
"La Palma","La Maceta","El Arpa","La Rana"
];

// map images
let cardImages = {};
deck.forEach((card,i)=>{
    cardImages[card] = base + names[i] + ".jpg";
});

// CALLER MODE (UNCHANGED)
let callerList = ["El Musico","El Catrin","La Dama","El Negrito"];
let injectQueue = [];
let blendMode = false;
let callCount = 0;

// STATE
let randomPool = [];
let autoInterval = null;
let countdownInterval = null;
let isSpeaking = false;
let calledSet = new Set();

// PRELOAD IMAGES
Object.values(cardImages).forEach(src=>{
    const img = new Image();
    img.src = src;
});

// SPEAK
function speak(text, callback){
    let msg = new SpeechSynthesisUtterance(text);
    let v = speechSynthesis.getVoices().find(v=>v.lang.includes("es")) || speechSynthesis.getVoices()[0];
    if(v) msg.voice = v;

    msg.onend = ()=>{isSpeaking=false; if(callback)callback();}
    isSpeaking=true;
    speechSynthesis.cancel();
    speechSynthesis.speak(msg);
}

// SETUP
function setup(){
    randomPool=[...deck].sort(()=>Math.random()-0.5);
}

// DISPLAY
function displayCard(card){
    const inner=document.getElementById("cardInner");
    const img=document.getElementById("cardImage");

    inner.style.transform="rotateY(180deg)";

    setTimeout(()=>{
        img.src=cardImages[card];
        document.getElementById("cardName").innerText=card;
        speak(card);
        addToHistory(card);

        inner.style.transform="rotateY(0deg)";
        inner.classList.add("bounce");
        setTimeout(()=>inner.classList.remove("bounce"),400);
    },300);

    startCountdown();
}

// NEXT
function nextCard(){
    if(isSpeaking)return;

    let card;
    if(blendMode && injectQueue.length>0 && callCount%2===1){
        card=injectQueue.shift();
    } else {
        if(randomPool.length===0)return;
        card=randomPool.shift();
    }

    displayCard(card);
    callCount++;
}

// START
function startGame(){
    speechSynthesis.resume();
    stopAuto();
    setup();
    callCount=0;
    calledSet.clear();
    document.getElementById("history").innerHTML="";

    nextCard();
    autoInterval=setInterval(nextCard,getSpeed()*1000);
}

// CALLER
function activateCaller(){
    injectQueue=[...callerList].sort(()=>Math.random()-0.5);
    blendMode=true;
}

// HISTORY
function addToHistory(card){
    if(calledSet.has(card))return;
    calledSet.add(card);
    let d=document.createElement("div");
    d.innerText=card;
    history.prepend(d);
}

// CONTROLS
function stopAuto(){
    clearInterval(autoInterval);
    clearInterval(countdownInterval);
}

function resetGame(){
    stopAuto();
    blendMode=false;
    injectQueue=[];
    document.getElementById("cardName").innerText="---";
    document.getElementById("countdown").innerText="0";
    document.getElementById("history").innerHTML="";
    calledSet.clear();
}

// SPEED
function getSpeed(){
    return parseInt(document.getElementById("speed").value);
}

// TIMER
function startCountdown(){
    clearInterval(countdownInterval);
    let t=getSpeed();
    countdown.innerText=t;
    countdownInterval=setInterval(()=>{
        t--;
        countdown.innerText=t;
        if(t<=0)clearInterval(countdownInterval);
    },1000);
}

// TAP + SWIPE
let startX=0;
document.getElementById("tapZone").addEventListener("touchstart",e=>{
    startX=e.touches[0].clientX;
});
document.getElementById("tapZone").addEventListener("touchend",e=>{
    let endX=e.changedTouches[0].clientX;
    if(Math.abs(endX-startX)>50){
        nextCard();
    } else {
        nextCard();
    }
});

// PARTICLES
const canvas=document.getElementById("particles");
const ctx=canvas.getContext("2d");

function resizeCanvas(){
    canvas.width=window.innerWidth;
    canvas.height=window.innerHeight;
}
window.addEventListener("resize",resizeCanvas);
resizeCanvas();

let particles=[];
for(let i=0;i<70;i++){
    particles.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,r:Math.random()*2,d:Math.random()*1});
}

function drawParticles(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.fillStyle="white";
    particles.forEach(p=>{
        ctx.beginPath();
        ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
        ctx.fill();
        p.y+=p.d;
        if(p.y>canvas.height){p.y=0;p.x=Math.random()*canvas.width;}
    });
    requestAnimationFrame(drawParticles);
}
drawParticles();
</script>

</body>
</html>
