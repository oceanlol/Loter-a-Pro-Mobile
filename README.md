<!DOCTYPE html>
<html lang="en">
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lotería Pro Full</title>

<style>
body{
    margin:0;
    font-family:Arial;
    background:black;
    color:white;
    text-align:center;
    padding-top:60px;
}

.card-container{
    margin-top:20px;
    perspective:1000px;
}

.card{
    width:90%;
    max-width:320px;
    height:250px;
    margin:auto;
    position:relative;
    transform-style:preserve-3d;
    transition:0.6s;
}

.face{
    position:absolute;
    width:100%;
    height:100%;
    backface-visibility:hidden;
    border-radius:15px;
}

.front img{
    width:100%;
    height:100%;
    object-fit:cover;
    border-radius:15px;
}

.back{
    background:#111;
    display:flex;
    justify-content:center;
    align-items:center;
    font-size:30px;
    transform:rotateY(180deg);
}

#cardName{
    font-size:24px;
    margin-top:15px;
}

button{
    width:90%;
    padding:15px;
    margin:6px;
    border:none;
    border-radius:10px;
    background:#222;
    color:white;
    font-size:18px;
}

button:active{
    transform:scale(0.95);
}

.history{
    margin-top:10px;
    max-height:150px;
    overflow:auto;
    background:#111;
    border-radius:10px;
    padding:10px;
    text-align:left;
}
</style>
</head>

<body>

<h2>Lotería</h2>

<div class="card-container">
    <div class="card" id="card">
        <div class="face front">
            <img id="img" src="https://via.placeholder.com/300">
        </div>
        <div class="face back">🎴</div>
    </div>
</div>

<div id="cardName">---</div>

<button onclick="start()">Start</button>
<button onclick="next()">Next</button>
<button onclick="resetGame()">Reset</button>

<div class="history" id="history"></div>

<script>

// 🔥 FULL 54 CARD DECK
const deck = [
"El Gallo","El Diablo","La Dama","El Catrin","El Paraguas","La Sirena","La Escalera","La Botella",
"El Barril","El Arbol","El Melon","El Valiente","El Gorrito","La Muerte","La Pera","La Bandera",
"El Bandolon","El Violoncello","La Garza","El Pajaro","La Mano","La Bota","La Luna","El Cotorro",
"El Borracho","El Negrito","El Corazon","La Sandia","El Tambor","El Camaron","Las Jaras","El Musico",
"La Arana","El Soldado","La Estrella","El Cazo","El Mundo","El Apache","El Nopal","El Alacran",
"La Rosa","La Calavera","La Campana","El Cantarito","El Venado","El Sol","La Corona","La Chalupa",
"El Pino","El Pescado","La Palma","La Maceta","El Arpa","La Rana"
];

// 🔥 ALL IMAGES (YOUR LINKS)
const images = {
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

let pool = [];
let used = new Set();
let voice;

// 🎤 VOICE FIX
function loadVoices(){
    let voices = speechSynthesis.getVoices();
    voice = voices.find(v=>v.lang.includes("es")) || voices[0];
}
speechSynthesis.onvoiceschanged = loadVoices;

function speak(text){
    let msg = new SpeechSynthesisUtterance(text);
    msg.voice = voice;
    msg.rate = 0.9;
    speechSynthesis.speak(msg);
}

// START
function start(){
    speechSynthesis.speak(new SpeechSynthesisUtterance("")); // unlock
    pool = [...deck].sort(()=>Math.random()-0.5);
    used.clear();
    document.getElementById("history").innerHTML="";
}

// NEXT (NO DUPES)
function next(){
    if(pool.length===0) return;

    let card = pool.shift();
    used.add(card);

    let cardEl = document.getElementById("card");
    cardEl.style.transform="rotateY(180deg)";

    setTimeout(()=>{
        let img = document.getElementById("img");
        img.onerror=()=> img.src="https://via.placeholder.com/300?text=Error";
        img.src = images[card];

        document.getElementById("cardName").innerText = card;
        speak(card);

        let h = document.createElement("div");
        h.innerText = card;
        document.getElementById("history").prepend(h);

        cardEl.style.transform="rotateY(0deg)");
    },300);
}

// RESET
function resetGame(){
    used.clear();
    document.getElementById("history").innerHTML="";
    document.getElementById("cardName").innerText="---";
}

</script>

</body>
</html>
