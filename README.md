# Merry-Christmas

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Circle Spinner</title>
  <link rel="stylesheet" href="spinner.css" />
</head>
<body>
  <main>
    <h1>Circle Spinner</h1>

    <div class="spinner-wrapper">
      <svg id="spinner-svg" viewBox="0 0 400 400" width="400" height="400" aria-label="Prize wheel">
        <!-- Wheel group (wedges will be injected here) -->
        <g id="wheel-group" transform="translate(200,200)"></g>

        <!-- Marker (fixed) -->
        <g id="marker" transform="translate(200,20)">
          <polygon points="-12,0 12,0 0,24" class="marker-triangle" />
        </g>
      </svg>
    </div>

    <div class="controls">
      <button id="spin-random">Spin Random</button>
      <button id="spin-index">Spin to index 2</button>
      <div id="result" aria-live="polite"></div>
    </div>
  </main>

  <script src="spinner.js"></script>
  <script>
    // Example configuration
    const config = {
      size: 400,
      radius: 180,
      wedges: [
        { label: "A", color: "#FF6B6B" },
        { label: "B", color: "#FFD93D" },
        { label: "C", color: "#6BCB77" },
        { label: "D", color: "#4D96FF" },
        { label: "E", color: "#9D4EDD" },
        { label: "F", color: "#FF8AC2" }
      ],
      textColor: "#222",
      spins: 6, // default full spins during animation
      duration: 4200 // ms
    };

    const spinner = new Spinner("spinner-svg", config);

    document.getElementById("spin-random").addEventListener("click", () => {
      document.getElementById("result").textContent = "Spinning...";
      spinner.spinRandom().then(index => {
        document.getElementById("result").textContent = `Landed on wedge ${index} (${config.wedges[index].label})`;
      });
    });

    document.getElementById("spin-index").addEventListener("click", () => {
      const idx = 2;
      document.getElementById("result").textContent = `Spinning to index ${idx}...`;
      spinner.spinTo(idx).then(index => {
        document.getElementById("result").textContent = `Landed on wedge ${index} (${config.wedges[index].label})`;
      });
    });
  </script>
</body>
</html>

