<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Felyfit Live Goals Overlay</title>
<style>
/* --- Body y contenedor --- */
body {
    margin:0;
    padding:0;
    background-color: transparent;
    display:flex;
    justify-content:center;
    align-items:center;
    height:100vh;
    flex-direction:column;
    font-family: 'Arial Black', sans-serif;
}

/* --- Contenedor de círculos --- */
.circle-container {
    position:relative;
    width:280px;
    height:280px;
}

svg {
    transform: rotate(-90deg);
    width:100%;
    height:100%;
}

circle {
    fill:none;
    stroke-width:18;
    stroke-linecap:round;
    transition: stroke-dashoffset 0.5s ease, stroke 0.3s ease;
}

.background { stroke: #F5F5DC; } /* huesito */
.likes { stroke:#FF6F00; stroke-dasharray:534; stroke-dashoffset:534; } /* naranja fitness */
.shares { stroke:#CCFF66; stroke-dasharray:408; stroke-dashoffset:408; } /* amarillo matcha */

.flash { filter: drop-shadow(0 0 20px #fff); }

/* --- Texto central --- */
.text-center {
    position:absolute;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
    text-align:center;
    font-weight:bold;
    text-shadow: 0 0 6px #000;
}
.text-center div { font-size:1.4rem; margin:5px 0; }
.likes-text { color:#FF6F00; }
.shares-text { color:#CCFF66; }

/* --- Botones de prueba --- */
button {
    margin-top:20px;
    padding:10px 20px;
    font-size:1rem;
    cursor:pointer;
    border-radius:8px;
    border:none;
    background:#FF6F00;
    color:white;
    font-weight:bold;
    margin-right:10px;
}

/* --- Efectos de destello --- */
.pulse {
    animation: pulse 0.5s ease-out;
}

@keyframes pulse {
    0% { transform: scale(1); opacity:1; }
    50% { transform: scale(1.1); opacity:0.7; }
    100% { transform: scale(1); opacity:1; }
}
</style>
</head>
<body>

<div class="circle-container">
    <svg>
        <circle class="background" cx="140" cy="140" r="85"/>
        <circle class="background" cx="140" cy="140" r="65"/>
        <circle class="likes" cx="140" cy="140" r="85"/>
        <circle class="shares" cx="140" cy="140" r="65"/>
    </svg>
    <div class="text-center">
        <div class="likes-text">❤️ <span id="likes">0</span></div>
        <div class="shares-text">🔁 <span id="shares">0</span></div>
    </div>
</div>

<div>
    <button onclick="setMeta(0)">Meta 1</button>
    <button onclick="setMeta(1)">Meta 2</button>
    <button onclick="setMeta(2)">Meta 3</button>
</div>

<script>
// --- Variables ---
let totalLikes = 0, totalShares = 0;
const likesText = document.getElementById("likes");
const sharesText = document.getElementById("shares");
const likesCircle = document.querySelector(".likes");
const sharesCircle = document.querySelector(".shares");

// --- Metas ---
const likeGoals = [10000, 50000, 100000];
const shareGoals = [100, 150, 200];
let currentGoalIndex = 0;

// --- Funciones de metas ---
function getCurrentLikeGoal() { return likeGoals[currentGoalIndex] || likeGoals[likeGoals.length-1]; }
function getPrevLikeGoal() { return likeGoals[currentGoalIndex-1] || 0; }
function getCurrentShareGoal() { return shareGoals[currentGoalIndex] || shareGoals[shareGoals.length-1]; }
function getPrevShareGoal() { return shareGoals[currentGoalIndex-1] || 0; }

// --- Actualizar círculos ---
function updateCircle(circle, total, prevGoal, currGoal){
    const radius = circle.r.baseVal.value;
    const circumference = 2 * Math.PI * radius;
    const progress = Math.min(Math.max((total - prevGoal)/(currGoal - prevGoal),0),1);
    const offset = circumference * (1 - progress);
    circle.style.strokeDashoffset = offset;
}

// --- Pulse al alcanzar meta ---
function pulseCircle(circle){
    circle.classList.add("pulse");
    setTimeout(()=>circle.classList.remove("pulse"),500);
}

// --- Verificar metas ---
function checkGoals(){
    if(totalLikes >= getCurrentLikeGoal() && totalShares >= getCurrentShareGoal()){
        pulseCircle(likesCircle);
        pulseCircle(sharesCircle);
        currentGoalIndex++;
        console.log(🎉 Meta alcanzada! Nueva meta: Likes ${getCurrentLikeGoal()}, Shares ${getCurrentShareGoal()});
    }
}

// --- Cambiar meta manual ---
function setMeta(index){
    if(index>=0 && index<likeGoals.length){
        currentGoalIndex = index;
        console.log(🔧 Meta cambiada manualmente a: Likes ${getCurrentLikeGoal()}, Shares ${getCurrentShareGoal()});
        updateCircle(likesCircle, totalLikes, getPrevLikeGoal(), getCurrentLikeGoal());
        updateCircle(sharesCircle, totalShares, getPrevShareGoal(), getCurrentShareGoal());
    }
}

// --- Conectar WebSocket ---
let socket = new WebSocket("ws://localhost:8765");
socket.onmessage = function(event){
    try{
        const data = JSON.parse(event.data);
        totalLikes = data.likes || totalLikes;
        totalShares = data.shares || totalShares;
        likesText.innerText = totalLikes;
        sharesText.innerText = totalShares;

        updateCircle(likesCircle, totalLikes, getPrevLikeGoal(), getCurrentLikeGoal());
        updateCircle(sharesCircle, totalShares, getPrevShareGoal(), getCurrentShareGoal());
        checkGoals();
    } catch(e){ console.error("Error parsing WebSocket data:", e); }
};

// --- Simulación si no hay WebSocket ---
/*
setInterval(()=>{
    totalLikes += Math.floor(Math.random()*500);
    totalShares += Math.floor(Math.random()*5);
    likesText.innerText = totalLikes;
    sharesText.innerText = totalShares;
    updateCircle(likesCircle, totalLikes, getPrevLikeGoal(), getCurrentLikeGoal());
    updateCircle(sharesCircle, totalShares, getPrevShareGoal(), getCurrentShareGoal());
    checkGoals();
},1000);
*/
</script>
</body>
</html>
