# Merry-Christmas

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roulette Spinner with Background</title>
<style>
  body {
    margin: 0;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-family: Arial, sans-serif;
    background: #000; /* fallback background color */
  }

  /* Background container */
  .background-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: url('https://via.placeholder.com/800x600') no-repeat center center;
    background-size: cover;
    z-index: -2;
  }

  /* Dim overlay to control brightness */
  .background-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.4); /* Change alpha to dim more or less */
    z-index: -1;
  }

  .spinner-container {
    position: relative;
    width: 400px;
    height: 400px;
  }

  canvas {
    border-radius: 50%;
    background: #ffffffcc; /* slightly transparent spinner background */
    box-shadow: 0 0 10px rgba(0,0,0,0.2);
    transform: rotate(0deg);
  }

  .pointer {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0; 
    height: 0;
    border-left: 15px solid transparent;
    border-right: 15px solid transparent;
    border-bottom: 30px solid red;
    transform: translate(-50%, -190px);
    z-index: 2;
  }

  button {
    margin-top: 20px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }

  .prize-popup {
    margin-top: 20px;
    padding: 20px;
    background: #ffd700;
    border-radius: 10px;
    font-size: 18px;
    font-weight: bold;
    display: none;
    box-shadow: 0 0 10px rgba(0,0,0,0.3);
    text-align: center;
    max-width: 300px;
  }

  .prize-popup img {
    max-width: 100%;
    border-radius: 8px;
    margin-top: 10px;
  }

  .prize-description {
    font-weight: normal;
    font-size: 16px;
    margin-top: 5px;
  }
</style>
</head>
<body>

<!-- Background and overlay -->
<div class="background-container"></div>
<div class="background-overlay"></div>

<div class="spinner-container">
  <canvas id="spinner" width="400" height="400"></canvas>
  <div class="pointer"></div>
</div>
<button onclick="spin()">Spin</button>

<div class="prize-popup" id="prizePopup">
  <div class="prize-name"></div>
  <div class="prize-description"></div>
  <img class="prize-image" src="" alt="" />
</div>

<script>
const canvas = document.getElementById('spinner');
const ctx = canvas.getContext('2d');
const popup = document.getElementById('prizePopup');
const prizeNameEl = popup.querySelector('.prize-name');
const prizeDescEl = popup.querySelector('.prize-description');
const prizeImgEl = popup.querySelector('.prize-image');
const size = canvas.width;
const center = size / 2;

// Define rich prize details
const prizes = [
    {name: "Prize A", description: "A $50 gift card", image: "https://via.placeholder.com/150?text=Prize+A"},
    {name: "Prize B", description: "A new headphones set", image: "https://via.placeholder.com/150?text=Prize+B"},
    {name: "Prize C", description: "A smartwatch", image: "https://via.placeholder.com/150?text=Prize+C"},
    {name: "Prize D", description: "A gaming mouse", image: "https://via.placeholder.com/150?text=Prize+D"},
    {name: "Prize E", description: "A coffee maker", image: "https://via.placeholder.com/150?text=Prize+E"},
    {name: "Prize F", description: "A book bundle", image: "https://via.placeholder.com/150?text=Prize+F"},
    {name: "Prize G", description: "A backpack", image: "https://via.placeholder.com/150?text=Prize+G"},
    {name: "Prize H", description: "A mystery box", image: "https://via.placeholder.com/150?text=Prize+H"}
];

const wedgeColors = ["#FF5733", "#33FF57", "#3357FF", "#F3FF33", "#FF33F3", "#33FFF3", "#F39C12", "#8E44AD"];
const numWedges = prizes.length;
let rotation = 0;

// Draw spinner
function drawSpinner() {
    ctx.clearRect(0,0,size,size);
    const wedgeAngle = (2 * Math.PI) / numWedges;
    
    for (let i = 0; i < numWedges; i++) {
        ctx.beginPath();
        ctx.moveTo(center, center);
        ctx.arc(center, center, center - 10, i * wedgeAngle, (i+1) * wedgeAngle);
        ctx.closePath();
        ctx.fillStyle = wedgeColors[i % wedgeColors.length];
        ctx.fill();
        ctx.strokeStyle = "#000000";
        ctx.lineWidth = 2;
        ctx.stroke();

        // Add wedge text
        ctx.save();
        ctx.translate(center, center);
        ctx.rotate(i * wedgeAngle + wedgeAngle/2);
        ctx.textAlign = "right";
        ctx.fillStyle = "#000";
        ctx.font = "16px Arial";
        ctx.fillText(prizes[i].name, center - 30, 5);
        ctx.restore();
    }
}

drawSpinner();

// Spin function
function spin() {
    popup.style.display = "none";
    const spins = Math.floor(Math.random() * 4) + 4; 
    const randomWedge = Math.floor(Math.random() * numWedges);
    const wedgeAngle = 2 * Math.PI / numWedges;
    const targetRotation = 2 * Math.PI * spins + randomWedge * wedgeAngle + wedgeAngle/2;

    let start = null;
    function animate(timestamp) {
        if (!start) start = timestamp;
        const progress = timestamp - start;
        const duration = 4000; 
        const easedProgress = easeOutCubic(progress / duration);
        rotation = targetRotation * easedProgress;
        canvas.style.transform = `rotate(${rotation}rad)`;
        if (progress < duration) {
            requestAnimationFrame(animate);
        } else {
            rotation = targetRotation;
            canvas.style.transform = `rotate(${rotation}rad)`;
            const landedIndex = (numWedges - randomWedge) % numWedges;
            showPrize(prizes[landedIndex]);
        }
    }
    requestAnimationFrame(animate);
}

// Show rich prize details
function showPrize(prize) {
    prizeNameEl.textContent = prize.name;
    prizeDescEl.textContent = prize.description || "";
    if (prize.image) {
        prizeImgEl.src = prize.image;
        prizeImgEl.style.display = "block";
    } else {
        prizeImgEl.style.display = "none";
    }
    popup.style.display = "block";
}

// Easing function
function easeOutCubic(t) {
    return (--t)*t*t+1;
}
</script>

</body>
</html>



