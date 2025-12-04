# Merry-Christmas

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roulette Spinner with Scalable Background</title>
<style>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
    font-family: Arial, sans-serif;
    overflow: hidden;
  }

  /* Background container that scales */
  .background-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: url('https://as2.ftcdn.net/v2/jpg/06/36/18/21/1000_F_636182143_mGhPRnv13Ur0JwFbyOzpEDvKajxv7iv8.jpg') no-repeat center center;
    background-size: cover; /* scales image to cover viewport */
    z-index: -2;
  }

  /* Dim overlay to adjust brightness */
  .background-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.4); /* adjust alpha to dim more or less */
    z-index: -1;
  }

  .spinner-container {
    position: relative;
    width: 400px;
    height: 400px;
  }

  canvas {
    border-radius: 50%;
    background: #ffffffcc;
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

  /* Make spinner responsive */
  @media (max-width: 500px) {
    .spinner-container {
      width: 300px;
      height: 300px;
    }

    canvas {
      width: 100%;
      height: 100%;
    }

    .pointer {
      transform: translate(-50%, -145px); /* adjust for smaller spinner */
      border-left: 12px solid transparent;
      border-right: 12px solid transparent;
      border-bottom: 25px solid red;
    }
  }
</style>
</head>
<body>

<!-- Background -->
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
let rotation = 0;

// Responsive canvas resizing
function resizeCanvas() {
    const container = document.querySelector('.spinner-container');
    const rect = container.getBoundingClientRect();
    canvas.width = rect.width;
    canvas.height = rect.height;
    drawSpinner();
}
window.addEventListener('resize', resizeCanvas);

// Prizes
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

// Draw spinner
function drawSpinner() {
    const size = canvas.width;
    const center = size / 2;
    ctx.clearRect(0, 0, size, size);
    const wedgeAngle = (2 * Math.PI) / numWedges;

    for (let i = 0; i < numWedges; i++) {
        ctx.beginPath();
        ctx.moveTo(center, center);
        ctx.arc(center, center, center - 10, i * wedgeAngle, (i+1) * wedgeAngle);
        ctx.closePath();
        ctx.fillStyle = wedgeColors[i % wedgeColors.length];
        ctx.fill();
        ctx.strokeStyle = "#000";
        ctx.lineWidth = 2;
        ctx.stroke();

        // Add wedge text
        ctx.save();
        ctx.translate(center, center);
        ctx.rotate(i * wedgeAngle + wedgeAngle/2);
        ctx.textAlign = "right";
        ctx.fillStyle = "#000";
        ctx.font = `${Math.floor(size/25)}px Arial`;
        ctx.fillText(prizes[i].name, center - 30, 5);
        ctx.restore();
    }
}

// Initial draw and responsive
resizeCanvas();

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

// Show prize
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

// Easing
function easeOutCubic(t) {
    return (--t)*t*t+1;
}
</script>

</body>
</html>



