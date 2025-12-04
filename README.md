# ChristmasSpin
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Spinning Wheel Prize</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 40px;
    }
    #wheel {
      width: 400px;
      height: 400px;
      border-radius: 50%;
      border: 8px solid #333;
      position: relative;
      overflow: hidden;
      transition: transform 4s cubic-bezier(0.25, 0.1, 0.25, 1);
    }
    .slice {
      position: absolute;
      width: 50%;
      height: 50%;
      top: 50%;
      left: 50%;
      transform-origin: 0% 0%;
      background: #eee;
      border: 1px solid #ccc;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 14px;
      text-align: center;
      padding: 4px;
    }
    #pointer {
      width: 0;
      height: 0;
      border-left: 20px solid transparent;
      border-right: 20px solid transparent;
      border-bottom: 30px solid red;
      margin-bottom: 20px;
    }
    #spinBtn {
      padding: 12px 20px;
      font-size: 16px;
      cursor: pointer;
    }
    #result {
      margin-top: 30px;
      font-size: 18px;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div id="pointer"></div>
  <div id="wheel"></div>
  <button id="spinBtn">SPIN</button>
  <div id="result"></div>

  <script>
    const prizes = [
      { label: "Prize A", details: "You win Prize A: A $20 Gift Card." },
      { label: "Prize B", details: "Prize B: Free movie tickets!" },
      { label: "Prize C", details: "Prize C: A bonus day off work!" },
      { label: "Prize D", details: "Prize D: A mystery box surprise." },
      { label: "Prize E", details: "Prize E: A coffee voucher." },
      { label: "Prize F", details: "Prize F: A tech gadget!" }
    ];

    const wheel = document.getElementById("wheel");
    const result = document.getElementById("result");

    const numSlices = prizes.length;
    const sliceAngle = 360 / numSlices;

    prizes.forEach((prize, index) => {
      const slice = document.createElement("div");
      slice.className = "slice";
      slice.style.transform = `rotate(${sliceAngle * index}deg) skewY(${90 - sliceAngle}deg)`;
      slice.textContent = prize.label;
      wheel.appendChild(slice);
    });

    let currentRotation = 0;

    document.getElementById("spinBtn").addEventListener("click", () => {
      const spinAmount = Math.floor(Math.random() * 360) + 720; // spin at least 2 full rotations
      currentRotation += spinAmount;
      wheel.style.transform = `rotate(${currentRotation}deg)`;

      setTimeout(() => {
        const normalized = currentRotation % 360;
        const index = Math.floor((numSlices - normalized / sliceAngle) % numSlices);
        const selectedPrize = prizes[index];
        result.textContent = `Result: ${selectedPrize.details}`;
      }, 4200);
    });
  </script>
</body>
</html>
