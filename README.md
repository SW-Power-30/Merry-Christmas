# Merry-Christmas

<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 40px;
      background-image: url('https://clipart-library.com/newimages/christmas-images-28.jpg');
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
    }
    body::before {
      content: "";
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-image: inherit;
      background-size: inherit;
      background-position: inherit;
      background-repeat: inherit;
      opacity: 0.1;
      z-index: -1;
    }
    #wheel-container {
      position: relative;
      width: 400px;
      height: 400px;
      margin-bottom: 20px;
    }
    #wheel {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      border: 8px solid #333;
      position: absolute;
      top: 0;
      left: 0;
      transition: transform 4s cubic-bezier(0.25, 0.1, 0.25, 1);
      overflow: hidden;
      box-shadow: inset 0 0 0 1px #333;
    }
    .slice {
      position: absolute;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      border-radius: 50%;
      clip-path: polygon(50% 50%, 100% 0%, 100% 100%);
      transform-origin: 50% 50%;
      background-color: white;
      display: flex;
      align-items: center;
      justify-content: flex-start;
      padding-left: 50%;
      font-size: 14px;
      text-align: center;
      box-sizing: border-box;
      border-right: 2px solid black; /* line to define each wedge */
    }
    #pointer {
      width: 0;
      height: 0;
      border-left: 15px solid transparent;
      border-right: 15px solid transparent;
      border-bottom: 40px solid red;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%) rotate(0deg);
      transform-origin: 50% 150%;
      z-index: 10;
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
  <div id="wheel-container">
    <div id="wheel"></div>
    <div id="pointer"></div>
  </div>
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
    const pointer = document.getElementById("pointer");
    const result = document.getElementById("result");

    const numSlices = prizes.length;
    const sliceAngle = 360 / numSlices;

    prizes.forEach((prize, index) => {
      const slice = document.createElement("div");
      slice.className = "slice";
      slice.style.transform = `rotate(${sliceAngle * index}deg)`;
      slice.textContent = prize.label;
      wheel.appendChild(slice);
    });

    let currentRotation = 0;

    document.getElementById("spinBtn").addEventListener("click", () => {
      const spinAmount = Math.floor(Math.random() * 360) + 720;
      currentRotation += spinAmount;
      wheel.style.transform = `rotate(${currentRotation}deg)`;
      pointer.style.transform = `translate(-50%, -50%) rotate(${-currentRotation}deg)`;

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

