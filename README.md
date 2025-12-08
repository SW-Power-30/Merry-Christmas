
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Prize Wheel</title>
<style>
    body {
        margin: 0;
        padding: 0;
        font-family: Arial, sans-serif;
        background-image: url('your-background.jpg');
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        background-attachment: scroll;
    }

    /* Title Area */
    #titleBox {
        text-align: center;
        font-size: 34px;
        font-weight: bold;
        color: white;
        margin-top: 20px;
        padding: 10px;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.6);
    }

    #titleInput {
        display: block;
        margin: 10px auto 25px auto;
        font-size: 18px;
        padding: 6px 10px;
        width: 60%;
        text-align: center;
        border-radius: 6px;
        border: 1px solid #ccc;
    }

    /* Wheel Wrapper */
    #wheelWrapper {
        position: relative;
        width: 500px;
        height: 540px;
        margin: 0 auto;
    }

    /* Pointer */
    #pointer {
        position: absolute;
        top: 0;
        left: 50%;
        transform: translateX(-50%);
        width: 0;
        height: 0;
        border-left: 22px solid transparent;
        border-right: 22px solid transparent;
        border-bottom: 35px solid yellow;
        z-index: 10;
    }

    /* Canvas */
    canvas {
        position: absolute;
        top: 40px;
        left: 0;
        right: 0;
        margin: auto;
        width: 100%;
    }

    /* Buttons */
    #controls {
        text-align: center;
        margin-top: 30px;
    }

    button {
        padding: 14px 24px;
        font-size: 18px;
        margin: 10px;
        cursor: pointer;
        border-radius: 8px;
        border: none;
        background-color: #333;
        color: white;
    }

    button:hover {
        background-color: #111;
    }

    /* Dim background */
    #popupBg {
        display: none;
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        backdrop-filter: blur(4px);
        background: rgba(0,0,0,0.5);
        z-index: 999;
    }

    /* Popup */
    #popup {
        display: none;
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: white;
        padding: 25px;
        border-radius: 14px;
        width: 330px;
        text-align: center;
        z-index: 1000;
        box-shadow: 0 0 25px rgba(0,0,0,0.4);
    }

    #popup h2 {
        margin-top: 0;
        font-size: 24px;
        color: #333;
    }
</style>
</head>
<body>

<div id="titleBox">Your Custom Title Here</div>
<input type="text" id="titleInput" placeholder="Enter custom title...">

<div id="wheelWrapper">
    <div id="pointer"></div>
    <canvas id="wheelCanvas" width="500" height="500"></canvas>
</div>

<div id="controls">
    <button id="spinBtn">SPIN</button>
    <button id="tryAgainBtn" style="display:none;">TRY AGAIN</button>
    <button id="acceptBtn" style="display:none;">ACCEPT PRIZE</button>
    <button id="resetBtn" style="display:none;">RESET</button>
</div>

<div id="popupBg"></div>

<div id="popup">
    <h2 id="popupTitle"></h2>
    <p id="popupDesc"></p>
    <button id="closePopupBtn">Close</button>
</div>

<script>
/* ------------------ TITLE UPDATE ------------------ */
document.getElementById("titleInput").addEventListener("input", function () {
    document.getElementById("titleBox").textContent = this.value;
});

/* ------------------ PRIZES ------------------ */
let originalPrizes = [
    { text: "Prize 1", desc: "Detailed description for Prize 1." },
    { text: "Prize 2", desc: "Detailed description for Prize 2." },
    { text: "Prize 3", desc: "Detailed description for Prize 3." },
    { text: "Prize 4", desc: "Detailed description for Prize 4." },
    { text: "Prize 5", desc: "Detailed description for Prize 5." },
    { text: "Prize 6", desc: "Detailed description for Prize 6." }
];

let prizes = JSON.parse(JSON.stringify(originalPrizes));

const colors = ["#90EE90", "#FF7F7F"];

/* ------------------ CANVAS SETUP ------------------ */
const canvas = document.getElementById("wheelCanvas");
const ctx = canvas.getContext("2d");

let angle = 0;
let spinning = false;
let spinCount = 0;
let selectedPrize = null;

/* ------------------ DRAW WHEEL ------------------ */
function drawWheel() {
    let num = prizes.length;
    let arc = (2 * Math.PI) / num;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    for (let i = 0; i < num; i++) {
        ctx.beginPath();
        ctx.fillStyle = colors[i % 2];
        ctx.moveTo(250, 250);
        ctx.arc(250, 250, 240, arc * i + angle, arc * (i + 1) + angle);
        ctx.fill();

        ctx.save();
        ctx.translate(250, 250);
        ctx.rotate(arc * i + arc / 2 + angle);
        ctx.textAlign = "right";
        ctx.fillStyle = "black";
        ctx.font = "20px Arial";
        ctx.fillText(prizes[i].text, 220, 10);
        ctx.restore();
    }
}

drawWheel();

/* ------------------ SPIN LOGIC ------------------ */
let spinBtn = document.getElementById("spinBtn");
let tryAgainBtn = document.getElementById("tryAgainBtn");
let acceptBtn = document.getElementById("acceptBtn");
let resetBtn = document.getElementById("resetBtn");

spinBtn.onclick = () => spinWheel();

function spinWheel() {
    if (spinning || prizes.length === 0) return;

    spinning = true;

    spinBtn.style.display = "none";

    let num = prizes.length;
    let arc = (2 * Math.PI) / num;

    let targetIndex = Math.floor(Math.random() * num);
    selectedPrize = prizes[targetIndex];

    /* ------------------ PERFECT POINTER ALIGNMENT ------------------ */
    let targetAngle = (Math.PI / 2) - (arc * (targetIndex + 0.5));
    let finalAngle = targetAngle + (Math.PI * 8);

    let duration = 3000;
    let start = null;

    function animate(timestamp) {
        if (!start) start = timestamp;
        let progress = timestamp - start;

        angle = finalAngle * (progress / duration);

        drawWheel();

        if (progress < duration) {
            requestAnimationFrame(animate);
        } else {
            angle = finalAngle % (2 * Math.PI);
            drawWheel();
            spinning = false;

            showPopup(selectedPrize);

            spinCount++;

            if (spinCount === 1) {
                tryAgainBtn.style.display = "inline-block";
                acceptBtn.style.display = "inline-block";
            } else {
                resetBtn.style.display = "inline-block";
            }
        }
    }

    requestAnimationFrame(animate);
}

/* ------------------ POPUP ------------------ */
function showPopup(prize) {
    document.getElementById("popupTitle").textContent = prize.text;
    document.getElementById("popupDesc").textContent = prize.desc;

    document.getElementById("popupBg").style.display = "block";
    document.getElementById("popup").style.display = "block";
}

document.getElementById("closePopupBtn").onclick = () => {
    document.getElementById("popupBg").style.display = "none";
    document.getElementById("popup").style.display = "none";
};

/* ------------------ TRY AGAIN ------------------ */
tryAgainBtn.onclick = () => {
    prizes = prizes.filter(p => p !== selectedPrize);

    tryAgainBtn.style.display = "none";
    acceptBtn.style.display = "none";

    document.getElementById("popupBg").style.display = "none";
    document.getElementById("popup").style.display = "none";

    drawWheel();

    spinBtn.style.display = "inline-block";
};

/* ------------------ ACCEPT PRIZE ------------------ */
acceptBtn.onclick = () => {
    tryAgainBtn.style.display = "none";
    acceptBtn.style.display = "none";
    resetBtn.style.display = "inline-block";
};

/* ------------------ RESET ------------------ */
resetBtn.onclick = () => {
    prizes = JSON.parse(JSON.stringify(originalPrizes));
    spinCount = 0;
    angle = 0;

    resetBtn.style.display = "none";
    spinBtn.style.display = "inline-block";

    document.getElementById("popupBg").style.display = "none";
    document.getElementById("popup").style.display = "none";

    drawWheel();
};
</script>

</body>
</html>


