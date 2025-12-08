
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Prize Wheel</title>
<style>
    body {
        margin: 0;
        padding: 0;
        font-family: Arial, sans-serif;
        background-image: url('https://cdn.stocksnap.io/img-thumbs/960w/wooden-wall_XQV3531O4I.jpg');
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        background-attachment: scroll; /* allows page scrolling */
    }

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

    #wheelContainer {
        width: 100%;
        text-align: center;
        margin-top: 20px;
        margin-bottom: 100px; /* allows room for popup visibility */
    }

    canvas {
        margin-top: 20px;
        max-width: 90vw;
        height: auto;
    }

    #pointer {
        width: 0; 
        height: 0; 
        border-left: 20px solid transparent;
        border-right: 20px solid transparent;
        border-bottom: 30px solid yellow;
        margin: 0 auto 10px auto;
    }

    #controls {
        text-align: center;
        margin-top: 20px;
    }

    button {
        padding: 12px 22px;
        font-size: 18px;
        margin: 10px;
        cursor: pointer;
        border-radius: 8px;
        border: none;
        background-color: #444;
        color: white;
    }

    button:hover {
        background-color: #222;
    }

    /* Popup overlay background */
    #popupBg {
        display: none;
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        backdrop-filter: blur(3px);
        background: rgba(0,0,0,0.55);
        z-index: 999;
    }

    /* Prize popup */
    #popup {
        display: none;
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: white;
        padding: 25px;
        border-radius: 12px;
        width: 320px;
        text-align: center;
        z-index: 1000;
        box-shadow: 0 0 18px rgba(0,0,0,0.4);
    }

    #popup h2 {
        margin-top: 0;
        color: #333;
        font-size: 22px;
    }

</style>
</head>
<body>

<div id="titleBox">Game Time, Lets have fun to determine your present!</div>


<div id="wheelContainer">
    <div id="pointer"></div>
    <canvas id="wheelCanvas" width="500" height="500"></canvas>
</div>

<div id="controls">
    <button id="spinBtn">SPIN</button>
    <button id="tryAgainBtn" style="display:none;">TRY AGAIN</button>
    <button id="acceptBtn" style="display:none;">ACCEPT PRIZE</button>
    <button id="resetBtn" style="display:none;">RESET</button>
</div>

<!-- Popup -->
<div id="popupBg"></div>

<div id="popup">
    <h2 id="popupTitle"></h2>
    <p id="popupDesc"></p>
    <button id="closePopupBtn">Close</button>
</div>

<script>
/* --------------------- TITLE BOX BEHAVIOUR --------------------- */
document.getElementById("titleInput").addEventListener("input", function () {
    document.getElementById("titleBox").textContent = this.value;
});

/* --------------------- PRIZE SETUP --------------------- */
let prizes = [
    { text: "Prize 1", desc: "This is the detailed description for Prize 1." },
    { text: "Prize 2", desc: "More details about Prize 2." },
    { text: "Prize 3", desc: "This explains Prize 3 in detail." },
    { text: "Prize 4", desc: "Prize 4 description info." },
    { text: "Prize 5", desc: "Information about Prize 5." },
    { text: "Prize 6", desc: "Prize 6 details displayed here." }
];

const colors = ["#90EE90", "#FF7F7F"];

/* --------------------- CANVAS + WHEEL DRAW --------------------- */
const canvas = document.getElementById("wheelCanvas");
const ctx = canvas.getContext("2d");
let angle = 0;
let spinning = false;

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

/* --------------------- SPIN LOGIC --------------------- */
let spinBtn = document.getElementById("spinBtn");
let tryAgainBtn = document.getElementById("tryAgainBtn");
let acceptBtn = document.getElementById("acceptBtn");
let resetBtn = document.getElementById("resetBtn");

let selectedPrize = null;
let spinCount = 0;

spinBtn.onclick = () => spinWheel();

function spinWheel() {
    if (spinning || prizes.length === 0) return;

    spinning = true;
    spinBtn.style.display = "none";

    let num = prizes.length;
    let arc = (2 * Math.PI) / num;
    let targetIndex = Math.floor(Math.random() * num);
    selectedPrize = prizes[targetIndex];

    let finalAngle = (Math.PI / 2) - (arc * targetIndex) + (2 * Math.PI * 5);

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
            angle = finalAngle;
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

/* --------------------- POPUP HANDLING --------------------- */
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

/* --------------------- BUTTON LOGIC --------------------- */
tryAgainBtn.onclick = () => {
    prizes = prizes.filter(p => p !== selectedPrize);

    tryAgainBtn.style.display = "none";
    acceptBtn.style.display = "none";

    spinBtn.style.display = "inline-block";

    document.getElementById("popupBg").style.display = "none";
    document.getElementById("popup").style.display = "none";

    drawWheel();
};

acceptBtn.onclick = () => {
    tryAgainBtn.style.display = "none";
    acceptBtn.style.display = "none";
    resetBtn.style.display = "inline-block";
};

resetBtn.onclick = () => {
    prizes = [
        { text: "Prize 1", desc: "This is the detailed description for Prize 1." },
        { text: "Prize 2", desc: "More details about Prize 2." },
        { text: "Prize 3", desc: "This explains Prize 3 in detail." },
        { text: "Prize 4", desc: "Prize 4 description info." },
        { text: "Prize 5", desc: "Information about Prize 5." },
        { text: "Prize 6", desc: "Prize 6 details displayed here." }
    ];

    spinCount = 0;
    selectedPrize = null;

    resetBtn.style.display = "none";
    spinBtn.style.display = "inline-block";

    document.getElementById("popupBg").style.display = "none";
    document.getElementById("popup").style.display = "none";

    drawWheel();
};

</script>
</body>
</html>


