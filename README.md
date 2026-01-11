<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>Crash Doma – لعبة الطيارة</title>

<meta name="description" content="لعبة Crash بالطائرة للتجربة فقط – اسحب قبل ما الطيارة تتحطم">
<meta name="keywords" content="crash game, لعبة كراش, لعبة طيارة, crash doma">
<meta name="robots" content="index, follow">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body { font-family: Arial, sans-serif; text-align: center; background:#eee; }
canvas { border:2px solid black; background: skyblue; }
#controls {
    background: gold;
    padding: 10px;
    border-radius: 10px;
    margin: 10px auto;
    width: 320px;
}
button { margin: 5px; padding: 10px; font-size: 16px; }
#betInput, #okBtn { display:none; }
#countdown {
    font-size: 48px;
    color: red;
    display: none;
    animation: spin 1s linear infinite;
}
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}
</style>
</head>

<body>

<h1>Crash Doma ✈️</h1>
<p>للتجربة فقط – لا يوجد ربح حقيقي</p>

<canvas id="gameCanvas" width="400" height="600"></canvas>

<div id="controls">
<p>الرصيد: <span id="balance">1000</span> جنيه</p>

<input type="number" id="betInput" placeholder="المبلغ" min="1">
<br>

<button id="placeBet">Place Bet</button>
<button id="okBtn">OK</button>
<button id="cashOut" disabled>Cash Out</button>

<p>المضاعف: <span id="multiplier">1.00</span>x</p>
<p id="countdown"></p>
<p id="message"></p>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const balanceEl = document.getElementById("balance");
const betInput = document.getElementById("betInput");
const placeBetBtn = document.getElementById("placeBet");
const okBtn = document.getElementById("okBtn");
const cashOutBtn = document.getElementById("cashOut");
const multiplierEl = document.getElementById("multiplier");
const countdownEl = document.getElementById("countdown");
const messageEl = document.getElementById("message");

let balance = parseFloat(localStorage.getItem("balance")) || 1000;
let bet = 0;
let multiplier = 1;
let isPlaying = false;
let hasCashedOut = false;
let planeY = canvas.height - 50;
let crashPoint = 0;
let countdown = 5;

function updateBalance(){
    balanceEl.textContent = balance.toFixed(2);
    localStorage.setItem("balance", balance);
}
updateBalance();

function drawPlane(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.font = "30px Arial";
    ctx.fillText("✈️",185,planeY);
}

function drawExplosion(){
    ctx.fillStyle="red";
    ctx.beginPath();
    ctx.arc(200,planeY,30,0,Math.PI*2);
    ctx.fill();
    ctx.fillText("💥",190,planeY);
}

function gameLoop(){
    if(isPlaying){
        planeY -= 1.5;
        multiplier += 0.01;
        multiplierEl.textContent = multiplier.toFixed(2);
        drawPlane();

        if(multiplier >= crashPoint){
            drawExplosion();
            messageEl.textContent="الطيارة اتحطمت";
            isPlaying=false;
            cashOutBtn.disabled=true;
            placeBetBtn.disabled=false;
            hasCashedOut=false;
            bet=0;
        }
    }
    requestAnimationFrame(gameLoop);
}
gameLoop();

placeBetBtn.onclick=()=>{
    betInput.style.display="inline";
    okBtn.style.display="inline";
    placeBetBtn.disabled=true;
};

okBtn.onclick=()=>{
    const amount=parseFloat(betInput.value);
    if(amount>0 && amount<=balance){
        bet=amount;
        balance-=bet;
        updateBalance();
        betInput.style.display="none";
        okBtn.style.display="none";

        countdown=5;
        countdownEl.style.display="block";
        countdownEl.textContent=countdown;

        const timer=setInterval(()=>{
            countdown--;
            countdownEl.textContent=countdown;
            if(countdown===0){
                clearInterval(timer);
                countdownEl.style.display="none";
                planeY=canvas.height-50;
                multiplier=1;
                crashPoint=Math.random()*9+1;
                isPlaying=true;
                hasCashedOut=false;
                cashOutBtn.disabled=false;
                messageEl.textContent="اللعبة بدأت";
            }
        },1000);
    }
};

cashOutBtn.onclick=()=>{
    if(isPlaying && !hasCashedOut){
        const win=bet*multiplier;
        balance+=win;
        updateBalance();
        messageEl.textContent=`سحبت عند ${multiplier.toFixed(2)}x`;
        hasCashedOut=true;
        cashOutBtn.disabled=true;
        bet=0;
    }
};
</script>

</body>
</html>
