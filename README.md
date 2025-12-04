# Merry-Christmas

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roulette Spinner</title>
<style>
  body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background: #f5f5f5;
    font-family: Arial, sans-serif;
  }

  .spinner-container {
    position: relative;
    width: 400px;
    height: 400px;
  }

  canvas {
    border-radius: 50%;
    background: #ffffffcc; /* Slightly transparent background */
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
    transform: translate(-50%, -190px); /* Position at top of spinner */
    z-index: 2;
  }

  button {
    margin-top: 20px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }
</style>
</head>
<body>

<div class="spinner-container">
  <canvas id="spinner" width="400" height="400"></canvas>
  <div class="pointer"></div>
</div>
<button onclick="spin()">Spin</button>

<script>
const canvas = document.getElementById('spinner');
const ctx = canvas.getContext('2d');
const size = canvas.width;
const center = size / 2;
const numWedges = 8;
const wedgeColors = ["#FF5733", "#33FF57", "#3357FF", "#F3FF33", "#FF33F3", "#33FFF3", "#F39C12", "#8E44AD"];
const wedgeLabels = ["1","2","3","4","5","6","7","8"];
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
        ctx.strokeStyle = "#000000"; // wedge lines
        ctx.lineWidth = 2;
        ctx.stroke();

        // Add label
        ctx.save();
        ctx.translate(center, center);
        ctx.rotate(i * wedgeAngle + wedgeAngle/2);
        ctx.textAlign = "right";
        ctx.fillStyle = "#000";
        ctx.font = "16px Arial";
        ctx.fillText(wedgeLabels[i], center - 30, 5);
        ctx.restore();
    }
}

// Initial draw
drawSpinner();

// Spin function
function spin() {
    const spins = Math.floor(Math.random() * 4) + 4; // 4-7 full rotations
    const randomWedge = Math.floor(Math.random() * numWedges);
    const wedgeAngle = 2 * Math.PI / numWedges;
    const targetRotation = 2 * Math.PI * spins + randomWedge * wedgeAngle + wedgeAngle / 2;

    let start = null;
    function animate(timestamp) {
        if (!start) start = timestamp;
        const progress = timestamp - start;
        const duration = 4000; // 4 seconds
        const easedProgress = easeOutCubic(progress / duration);
        rotation = targetRotation * easedProgress;
        canvas.style.transform = `rotate(${rotation}rad)`;
        if (progress < duration) {
            requestAnimationFrame(animate);
        } else {
            rotation = targetRotation;
            canvas.style.transform = `rotate(${rotation}rad)`;
            const landedWedge = (numWedges - randomWedge) % numWedges;
            alert("Landed on wedge: " + wedgeLabels[landedWedge]);
        }
    }
    requestAnimationFrame(animate);
}

// Easing function
function easeOutCubic(t) {
    return (--t)*t*t+1;
}
</script>

</body>
</html>


