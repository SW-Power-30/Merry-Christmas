
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Spinner Wheel</title>

<style>
body {
    font-family: Arial, sans-serif;
    text-align: center;
    padding-bottom: 200px;
}

#titleBox {
    margin-top: 20px;
    padding: 10px;
    font-size: 20px;
    width: 80%;
}

#wheelContainer {
    position: relative;
    width: 400px;
    height: 400px;
    margin: 40px auto;
}

#wheelCanvas {
    width: 100%;
    height: 100%;
    border-radius: 50%;
}

.pointer {
    position: absolute;
    width: 0;
    height: 0;

    border-left: 15px solid transparent;
    border-right: 15px solid transparent;
    border-bottom: 60px solid red;

    top: 50%;
    left: 50%;

    /* FIX: true physical attachment to center */
    transform: translate(-50%, -100%);
    transform-origin: 50% 100%;

    pointer-events: none;
}

button {
    margin: 10px;
    padding: 12px 25px;
    font-size: 18px;
}

#resultBox {
    margin-top: 20px;
    font-size: 24px;
    min-height: 30px;
}
</style>
</head>
<body>

<h2>Enter Title</h2>
<input id="titleBox" placeholder="Enter custom title..." />

<div id="wheelContainer">
    <canvas id="wheelCanvas" width="400" height="400"></canvas>
    <div id="pointer" class="pointer"></div>
</div>

<button id="spinBtn">Spin</button>
<button id="tryAgainBtn" style="display:none;">Try Again</button>
<button id="acceptBtn" style="display:none;">Accept Prize</button>
<button id="resetBtn" style="display:none;">Reset</button>

<div id="resultBox"></div>

<script>
let prizes = ["Prize A", "Prize B", "Prize C", "Prize D", "Prize E", "Prize F"];
let colors = ["#90EE90", "#FF7F7F"];
let wheelCanvas = document.getElementById("wheelCanvas");
let ctx = wheelCanvas.getContext("2d");
let pointer = document.getElementById("pointer");

let angle = 0;
let spinning = false;
let selectedPrize = null;

function drawWheel() {
    let total = prizes.length;
    let arc = (2 * Math.PI) / total;

    ctx.clearRect(0, 0, 400, 400);

    for (let i = 0; i < total; i++) {
        ctx.beginPath();
        ctx.fillStyle = colors[i % 2];
        ctx.moveTo(200, 200);
        ctx.arc(200, 200, 200, arc * i, arc * (i + 1));
        ctx.lineTo(200, 200);
        ctx.fill();

        ctx.save();
        ctx.translate(200, 200);
        ctx.rotate(arc * (i + 0.5));
        ctx.textAlign = "right";
        ctx.fillStyle = "black";
        ctx.font = "18px Arial";
        ctx.fillText(prizes[i], 180, 10);
        ctx.restore();
    }
}

function spinWheel() {
    if (spinning || prizes.length === 0) return;

    spinning = true;

    let total = prizes.length;
    let arcDeg = 360 / total;

    let index = Math.floor(Math.random() * total);
    selectedPrize = prizes[index];

    let stopAngle = 360 - (index * arcDeg + arcDeg / 2);
    let finalAngle = stopAngle + 360 * 5;
    let duration = 3000;
    let start = null;

    function animate(timestamp) {
        if (!start) start = timestamp;
        let progress = timestamp - start;

        angle = easeOut(progress, 0, finalAngle, duration);

        wheelCanvas.style.transform = `rotate(${angle}deg)`;
        pointer.style.transform = `translate(-50%, -100%) rotate(${angle}deg)`;  // FIX

        if (progress < duration) {
            requestAnimationFrame(animate);
        } else {
            angle = finalAngle % 360;
            spinning = false;

            wheelCanvas.style.transform = `rotate(${angle}deg)`;
            pointer.style.transform = `translate(-50%, -100%) rotate(${angle}deg)`;

            document.getElementById("resultBox").textContent =
                "You won: " + selectedPrize;

            spinBtn.style.display = "none";
            acceptBtn.style.display = "inline-block";
            tryAgainBtn.style.display = "inline-block";
        }
    }

    requestAnimationFrame(animate);
}

function easeOut(t, b, c, d) {
    t /= d;
    t--;
    return c * (t * t * t + 1) + b;
}

spinBtn.onclick = spinWheel;

tryAgainBtn.onclick = () => {
    prizes = prizes.filter(p => p !== selectedPrize);
    drawWheel();

    selectedPrize = null;
    document.getElementById("resultBox").textContent = "";

    spinBtn.style.display = "inline-block";
    tryAgainBtn.style.display = "none";
    acceptBtn.style.display = "none";

    if (prizes.length === 1) {
        tryAgainBtn.style.display = "none";
    }
};

acceptBtn.onclick = () => {
    tryAgainBtn.style.display = "none";
    acceptBtn.style.display = "none";
    resetBtn.style.display = "inline-block";
};

resetBtn.onclick = () => {
    prizes = ["Prize A", "Prize B", "Prize C", "Prize D", "Prize E", "Prize F"];
    angle = 0;
    wheelCanvas.style.transform = "rotate(0deg)";
    pointer.style.transform = "translate(-50%, -100%) rotate(0deg)";
    drawWheel();

    selectedPrize = null;
    document.getElementById("resultBox").textContent = "";

    resetBtn.style.display = "none";
    spinBtn.style.display = "inline-block";
};

drawWheel();
</script>

</body>
</html>

