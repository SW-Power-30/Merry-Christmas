
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Prize Wheel</title>

<style>
body {
    font-family: Arial, sans-serif;
    text-align: center;
    margin: 0;
    padding-bottom: 200px;
}

/* Title input */
#titleBox {
    margin-top: 20px;
    padding: 10px;
    font-size: 20px;
    width: 80%;
    max-width: 500px;
}

/* Wheel container */
#wheelContainer {
    margin: 40px auto;
    width: 400px;
    height: 400px;
    position: relative;
}

/* POINTER — Correct fixed position */
.pointer {
    position: absolute;
    top: -20px; /* sits above the wheel */
    left: 50%;
    transform: translateX(-50%);

    width: 0;
    height: 0;

    border-left: 20px solid transparent;
    border-right: 20px solid transparent;
    border-bottom: 40px solid yellow;

    z-index: 50;
    pointer-events: none;
}

/* Wheel canvas */
#wheelCanvas {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background: white;
}

/* Buttons */
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
    <div id="pointer" class="pointer"></div>
    <canvas id="wheelCanvas" width="400" height="400"></canvas>
</div>

<button id="spinBtn">Spin</button>
<button id="tryAgainBtn" style="display:none;">Try Again</button>
button id="acceptBtn" style="display:none;">Accept Prize</button>
<button id="resetBtn" style="display:none;">Reset</button>

<div id="resultBox"></div>

<script>
let prizes = ["Prize A", "Prize B", "Prize C", "Prize D", "Prize E", "Prize F"];
const colors = ["#90EE90", "#FF7F7F"]; // alternating green/red

let wheel = document.getElementById("wheelCanvas");
let ctx = wheel.getContext("2d");
let pointer = document.getElementById("pointer");

let spinBtn = document.getElementById("spinBtn");
let tryAgainBtn = document.getElementById("tryAgainBtn");
let acceptBtn = document.getElementById("acceptBtn");
let resetBtn = document.getElementById("resetBtn");

let selectedPrize = null;
let spinning = false;
let angle = 0;

/* ---------------- DRAW WHEEL ---------------- */
function drawWheel() {
    let count = prizes.length;
    let arc = (2 * Math.PI) / count;

    ctx.clearRect(0, 0, 400, 400);

    for (let i = 0; i < count; i++) {
        ctx.beginPath();
        ctx.fillStyle = colors[i % 2];
        ctx.moveTo(200, 200);
        ctx.arc(200, 200, 200, i * arc, (i + 1) * arc);
        ctx.lineTo(200, 200);
        ctx.fill();

        /* Text */
        ctx.save();
        ctx.translate(200, 200);
        ctx.rotate(i * arc + arc / 2);
        ctx.textAlign = "right";
        ctx.fillStyle = "black";
        ctx.font = "18px Arial";
        ctx.fillText(prizes[i], 170, 10);
        ctx.restore();
    }
}

/* --------------- SPIN WHEEL (ACCURATE STOP) --------------- */
function spinWheel() {
    if (spinning || prizes.length === 0) return;

    spinning = true;
    let count = prizes.length;
    let arc = 360 / count;

    /* Select random prize */
    let index = Math.floor(Math.random() * count);
    selectedPrize = prizes[index];

    /* Calculate exact stop angle so the wedge lands under pointer */
    let stopAngle = 360 - (index * arc + arc / 2);
    let finalAngle = stopAngle + 360 * 8; // spin 8 full turns

    let duration = 3400;
    let start = null;

    function animate(timestamp) {
        if (!start) start = timestamp;
        let progress = timestamp - start;

        /* Smooth easing */
        angle = easeOut(progress, 0, finalAngle, duration);

        wheel.style.transform = `rotate(${angle}deg)`;

        if (progress < duration) {
            requestAnimationFrame(animate);
        } else {
            angle = finalAngle % 360;
            wheel.style.transform = `rotate(${angle}deg)`;
            spinning = false;

            document.getElementById("resultBox").textContent =
                "You won: " + selectedPrize;

            spinBtn.style.display = "none";
            acceptBtn.style.display = "inline-block";
            tryAgainBtn.style.display = "inline-block";
        }
    }

    requestAnimationFrame(animate);
}

/* Ease-out cubic */
function easeOut(t, b, c, d) {
    t /= d;
    t--;
    return c * (t * t * t + 1) + b;
}

/* ---------------- BUTTON LOGIC ---------------- */

tryAgainBtn.onclick = () => {
    prizes = prizes.filter(p => p !== selectedPrize);
    drawWheel();

    selectedPrize = null;
    document.getElementById("resultBox").textContent = "";

    spinBtn.style.display = "inline-block";
    acceptBtn.style.display = "none";
    tryAgainBtn.style.display = "none";
};

acceptBtn.onclick = () => {
    acceptBtn.style.display = "none";
    tryAgainBtn.style.display = "none";
    resetBtn.style.display = "inline-block";
};

resetBtn.onclick = () => {
    prizes = ["Prize A", "Prize B", "Prize C", "Prize D", "Prize E", "Prize F"];
    angle = 0;
    wheel.style.transform = "rotate(0deg)";
    selectedPrize = null;
    drawWheel();

    resetBtn.style.display = "none";
    spinBtn.style.display = "inline-block";
    document.getElementById("resultBox").textContent = "";
};

/* ---------------- INITIAL DRAW ---------------- */
drawWheel();
spinBtn.onclick = spinWheel;
</script>
</body>
</html>


