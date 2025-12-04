# Merry-Christmas

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roulette Spinner</title>

<style>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
    font-family: Arial, sans-serif;
    overflow: hidden;
  }

  /* Scalable background */
  .background-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: url('https://via.placeholder.com/1600x900') no-repeat center center;
    background-size: cover;
    z-index: -2;
  }

  /* Dim overlay */
  .background-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.4);
    z-index: -1;
  }

  .spinner-container {
    position: relative;
    width: 400px;
    height: 400px;
    margin: 30px auto;
  }

  canvas {
    border-radius: 50%;
    background: transparent;
    box-shadow: 0 0 10px rgba(0,0,0,0.3);
  }

  .pointer {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0; 
    height: 0;
    border-left: 15px solid transparent;
    border-right: 15px solid transparent;
    border-bottom: 35px solid red;
    transform: translate(-50%, -190px);
    z-index: 10;
  }

  button {
    display: block;
    margin: 10px auto 0 auto;
    padding: 12px 25px;
    font-size: 18px;
    cursor: pointer;
  }

  /* Prize popup */
  .prize-popup {
    width: 300px;
    margin: 20px auto 0 auto;
    padding: 20px;
    background: #ffd700;
    border-radius: 12px;
    font-size: 18px;
    display: none;
    text-align: center;
    box-shadow: 0 0 12px rgba(0,0,0,0.4);
  }

  .prize-popup img {
    max-width: 100%;
    border-radius: 8px;
    margin-top: 10px;
  }

  .prize-name {
    font-weight: bold;
    font-size: 22px;
  }

  .prize-description {
    font-size: 16px;
    margin-top: 8px;
  }

  /* Responsive spinner for small screens */
  @media (max-width: 500px) {
    .spinner-container {
      width: 300px;
      height: 300px;
    }
    .pointer {
      transform: translate(-50%, -145px);
      border-bottom: 28px solid red;
    }
  }
</style>
</head>
<body>

<!-- Background -->
<div class="background-container"></div>
<div class="background-overlay"></div>

<!-- Spinner -->
<div class="spinner-container">
  <canvas id="spinner"></canvas>
  <div class="pointer"></div>
</div>

<button onclick="spin()">SPIN</button>

<!-- Prize Popup -->
<div class="prize-popup" id="prizePopup">
  <div class="prize-name"></div>
  <div class="prize-description"></div>
  <img class="prize-image" src="" alt="">
</div>

<script>
const canvas = document.getElementById("spinner");
const ctx = canvas.getContext("2d");
const popup = document.getElementById("prizePopup");
const prizeNameEl = popup.querySelector(".prize-name");
const prizeDescEl = popup.querySelector(".prize-description");
const prizeImgEl = popup.querySelector(".prize-image");

// 6 PRIZES — YOU CAN EDIT THESE
const prizes = [
    { name: "Prize A", description: "Description for Prize A", image: "https://via.placeholder.com/150?text=Prize+A" },
    { name: "Prize B", description: "Description for Prize B", image: "https://via.placeholder.com/150?text=Prize+B" },
    { name: "Prize C", description: "Description for Prize C", image: "https://via.placeholder.com/150?text=Prize+C" },
    { name: "Prize D", description: "Description for Prize D", image: "https://via.placeholder.com/150?text=Prize+D" },
    { name: "Prize E", description: "Description for Prize E", image: "https://via.placeholder.com/150?text=Prize+E" },
    { name: "Prize F", description: "Description for Prize F", image: "https://via.placeholder.com/150?text=Prize+F" }
];

// Alternating colours
const wedgeColors = [
    "#90EE90", "#FF7F7F",
    "#90EE90", "#FF7F7F",
    "#90EE90", "#FF7F7F"
];

let numWedges = prizes.length;
let rotation = 0;

// Resize canvas to container
function resizeCanvas() {
    const box = document.querySelector(".spinner-container");
    canvas.width = box.clientWidth;
    canvas.height = box.clientHeight;
    drawSpinner();
}
window.addEventListener("resize", resizeCanvas);
resizeCanvas();

// Draw wheel wedges
function drawSpinner() {
    const size = canvas.width;
    const center = size / 2;
    ctx.clearRect(0, 0, size, size);

    const wedgeAngle = (2 * Math.PI) / numWedges;

    for (let i = 0; i < numWedges; i++) {
        ctx.beginPath();
        ctx.moveTo(center, center);
        ctx.arc(center, center, center - 5, i * wedgeAngle, (i+1) * wedgeAngle);
        ctx.closePath();
        ctx.fillStyle = wedgeColors[i];
        ctx.fill();
        
        ctx.strokeStyle = "#000";
        ctx.lineWidth = 2;
        ctx.stroke();

        // Text
        ctx.save();
        ctx.translate(center, center);
        ctx.rotate(i * wedgeAngle + wedgeAngle / 2);
        ctx.textAlign = "right";
        ctx.font = `${Math.floor(size / 22)}px Arial`;
        ctx.fillStyle = "#000";
        ctx.fillText(prizes[i].name, center - 25, 5);
        ctx.restore();
    }
}

// Spin animation
function spin() {
    popup.style.display = "none";

    const spins = Math.floor(Math.random() * 4) + 4;
    const randomIndex = Math.floor(Math.random() * numWedges);
    const wedgeAngle = (2 * Math.PI) / numWedges;

    const targetRotation =
        2 * Math.PI * spins +
        (randomIndex * wedgeAngle) +
        wedgeAngle / 2;

    let start = null;

    function animate(time) {
        if (!start) start = time;
        const progress = (time - start) / 4000;
        const eased = easeOutCubic(progress);

        if (progress < 1) {
            rotation = targetRotation * eased;
            canvas.style.transform = `rotate(${rotation}rad)`;
            requestAnimationFrame(animate);
        } else {
            canvas.style.transform = `rotate(${targetRotation}rad)`;
            showPrize(prizes[(numWedges - randomIndex) % numWedges]);
        }
    }
    requestAnimationFrame(animate);
}

// Display prize popup
function showPrize(prize) {
    prizeNameEl.textContent = prize.name;
    prizeDescEl.textContent = prize.description;
    prizeImgEl.src = prize.image;

    popup.style.display = "block";
}

// Ease-out function
function easeOutCubic(t) {
    return (--t) * t * t + 1;
}
</script>

</body>
</html>




