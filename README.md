# Merry-Christmas

<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Roulette Spinner — Accurate Pointer</title>
<style>
  html,body{
    margin:0; padding:0; height:100%; width:100%; font-family:Arial,Helvetica,sans-serif; overflow:hidden;
  }

  /* Scalable background */
  .background-container{
    position:fixed; inset:0;
    background:url('https://cdn.stocksnap.io/img-thumbs/960w/seasonal-backgrounds_QQA4QVER48.jpg') no-repeat center center/cover;
    z-index:-2;
  }
  .background-overlay{
    position:fixed; inset:0;
    background:rgba(0,0,0,0.4);
    z-index:-1;
  }

  .spinner-container{
    position:relative;
    width:400px;
    height:400px;
    margin:30px auto;
  }

  canvas{
    width:100%;
    height:100%;
    display:block;
    border-radius:50%;
    background:transparent;
    box-shadow:0 0 10px rgba(0,0,0,0.3);
    transform-origin:50% 50%;
  }

  .pointer{
    position:absolute;
    left:50%;
    top:50%;
    transform:translate(-50%,-190px); /* sits above wheel */
    width:0; height:0;
    border-left:15px solid transparent;
    border-right:15px solid transparent;
    border-bottom:35px solid #e53935; /* red pointer */
    z-index:10;
    filter:drop-shadow(0 2px 4px rgba(0,0,0,0.4));
  }

  button{
    display:block;
    margin:12px auto 0;
    padding:12px 24px;
    font-size:18px;
    cursor:pointer;
  }

  .prize-popup{
    width:320px;
    margin:18px auto;
    padding:18px;
    background:#ffd700;
    border-radius:12px;
    text-align:center;
    display:none;
    box-shadow:0 6px 20px rgba(0,0,0,0.25);
  }
  .prize-name{ font-size:20px; font-weight:700; }
  .prize-description{ margin-top:8px; font-size:15px; font-weight:400; }
  .prize-popup img{ margin-top:12px; max-width:100%; border-radius:8px; }

  /* Responsive */
  @media (max-width:520px){
    .spinner-container{ width:300px; height:300px; }
    .pointer{ transform:translate(-50%,-145px); border-bottom:28px solid #e53935; }
    .prize-popup{ width:90%; }
  }
</style>
</head>
<body>

<div class="background-container"></div>
<div class="background-overlay"></div>

<div class="spinner-container">
  <canvas id="spinner"></canvas>
  <div class="pointer" aria-hidden="true"></div>
</div>

<button id="spinBtn">SPIN</button>

<div class="prize-popup" id="prizePopup" role="status" aria-live="polite">
  <div class="prize-name"></div>
  <div class="prize-description"></div>
  <img class="prize-image" src="" alt="">
</div>

<script>
/* ---------- Config: prizes & alternating colors ---------- */
const prizes = [
  { name: "Prize A", description: "Description for Prize A", image: "https://via.placeholder.com/300?text=Prize+A" },
  { name: "Prize B", description: "Description for Prize B", image: "https://via.placeholder.com/300?text=Prize+B" },
  { name: "Prize C", description: "Description for Prize C", image: "https://via.placeholder.com/300?text=Prize+C" },
  { name: "Prize D", description: "Description for Prize D", image: "https://via.placeholder.com/300?text=Prize+D" },
  { name: "Prize E", description: "Description for Prize E", image: "https://via.placeholder.com/300?text=Prize+E" },
  { name: "Prize F", description: "Description for Prize F", image: "https://via.placeholder.com/300?text=Prize+F" }
];

const wedgeColors = ["#90EE90","#FF7F7F","#90EE90","#FF7F7F","#90EE90","#FF7F7F"]; // alternating

/* ---------- Elements & canvas setup ---------- */
const canvas = document.getElementById("spinner");
const ctx = canvas.getContext("2d");
const container = document.querySelector(".spinner-container");
const spinBtn = document.getElementById("spinBtn");
const popup = document.getElementById("prizePopup");
const prizeNameEl = popup.querySelector(".prize-name");
const prizeDescEl = popup.querySelector(".prize-description");
const prizeImgEl = popup.querySelector(".prize-image");

let numWedges = prizes.length;
let currentRotation = 0; // radians currently applied (used for visual continuity)

/* Responsive canvas sizing + keep drawing accurate for devicePixelRatio */
function resizeCanvas(){
  const rect = container.getBoundingClientRect();
  const dpr = window.devicePixelRatio || 1;
  canvas.width = Math.round(rect.width * dpr);
  canvas.height = Math.round(rect.height * dpr);
  canvas.style.width = rect.width + "px";
  canvas.style.height = rect.height + "px";
  ctx.setTransform(dpr,0,0,dpr,0,0); // scale drawing to CSS pixels
  drawWheel(); // redraw at new size
}
window.addEventListener("resize", resizeCanvas);
resizeCanvas();

/* ---------- Drawing the wheel ---------- */
function drawWheel(){
  const w = canvas.width / (window.devicePixelRatio || 1);
  const h = canvas.height / (window.devicePixelRatio || 1);
  const center = { x: w/2, y: h/2 };
  const radius = Math.min(w,h)/2 - 6;
  ctx.clearRect(0,0,w,h);

  const wedgeAngle = (2 * Math.PI) / numWedges;

  for(let i=0;i<numWedges;i++){
    const start = i * wedgeAngle;
    const end = start + wedgeAngle;

    // wedge fill
    ctx.beginPath();
    ctx.moveTo(center.x, center.y);
    ctx.arc(center.x, center.y, radius, start, end);
    ctx.closePath();
    ctx.fillStyle = wedgeColors[i % wedgeColors.length] || "#ddd";
    ctx.fill();

    // wedge border
    ctx.strokeStyle = "#000";
    ctx.lineWidth = 2;
    ctx.stroke();

    // label (we draw upright relative to wheel; wheel rotates visually via CSS transform)
    ctx.save();
    ctx.translate(center.x, center.y);
    ctx.rotate(start + wedgeAngle/2); // rotate to wedge center
    ctx.textAlign = "right";
    ctx.fillStyle = "#000";
    ctx.font = `${Math.max(12, Math.floor(radius/6))}px Arial`;
    ctx.fillText(prizes[i].name, radius - 18, 6);
    ctx.restore();
  }
}

/* ---------- Spin logic (accurate alignment) ---------- */
/*
 Explanation:
  - wedgeCenterAngle = index*wedgeAngle + wedgeAngle/2
  - We want wedgeCenterAngle + rotation = angleAtPointer (top) (mod 2π)
  - angleAtPointer (top) is -π/2 (equivalently 3π/2)
  - So rotation = -π/2 - wedgeCenterAngle  (choose additional full spins to make rotation positive and animate nicely)
  - We build targetRotation = fullRotations*2π + (-π/2 - wedgeCenterAngle)
  - To keep a positive increasing rotation we add fullRotations*2π (fullRotations >= 3)
*/

function spin(){
  popup.style.display = "none";
  spinBtn.disabled = true;

  const wedgeAngle = (2*Math.PI)/numWedges;
  const chosenIndex = Math.floor(Math.random()*numWedges);

  // choose number of extra full rotations for visual effect
  const extraSpins = Math.floor(Math.random()*3) + 4; // 4..6 full spins

  const wedgeCenter = chosenIndex * wedgeAngle + wedgeAngle/2;
  // we want wedgeCenter + R = -Math.PI/2  (top). So R = -Math.PI/2 - wedgeCenter
  // make positive by adding extra spins * 2π
  const targetRotation = extraSpins * 2*Math.PI + (-Math.PI/2 - wedgeCenter);

  // animate from currentRotation to currentRotation + targetRotation
  const startRotation = currentRotation;
  const endRotation = currentRotation + targetRotation;

  const duration = 4200; // ms
  const startTime = performance.now();

  function animate(now){
    const elapsed = now - startTime;
    const t = Math.min(1, elapsed / duration);
    const eased = easeOutCubic(t);
    const rot = startRotation + (endRotation - startRotation) * eased;

    // apply rotation (CSS transform uses radians)
    canvas.style.transform = `rotate(${rot}rad)`;

    if(t < 1){
      requestAnimationFrame(animate);
    } else {
      // finalize
      currentRotation = endRotation % (2*Math.PI); // keep rotation normalized
      // show chosen prize
      showPrize(prizes[chosenIndex]);
      spinBtn.disabled = false;
    }
  }
  requestAnimationFrame(animate);
}

/* ---------- Show prize popup ---------- */
function showPrize(prize){
  prizeNameEl.textContent = prize.name || "";
  prizeDescEl.textContent = prize.description || "";
  if(prize.image){
    prizeImgEl.src = prize.image;
    prizeImgEl.style.display = "block";
  } else {
    prizeImgEl.style.display = "none";
  }
  popup.style.display = "block";
}

/* ---------- easing ---------- */
function easeOutCubic(t){ return (--t)*t*t + 1; }

/* ---------- hook up button ---------- */
spinBtn.addEventListener("click", spin);

/* draw initial */
drawWheel();

</script>
</body>
</html>



