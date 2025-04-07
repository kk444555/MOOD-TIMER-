<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mode Tracker</title>
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500&display=swap" rel="stylesheet">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Orbitron', sans-serif;
      background: linear-gradient(135deg, #1f1c2c, #928dab);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: #fff;
      padding: 20px;
    }
    h1 {
      font-size: 2.5rem;
      margin-bottom: 30px;
      text-shadow: 0 0 10px #fff;
      animation: fadeIn 1s ease-in-out;
    }
    .mode {
      backdrop-filter: blur(10px);
      background: rgba(255, 255, 255, 0.1);
      border-radius: 20px;
      border: 1px solid rgba(255, 255, 255, 0.3);
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.37);
      padding: 30px;
      width: 90%;
      max-width: 400px;
      margin: 20px auto;
      transition: transform 0.4s ease, background 0.4s ease;
      transform: scale(1);
      animation: fadeInUp 0.6s ease forwards;
      text-align: center;
    }
    .mode.active {
      background: rgba(0, 255, 150, 0.2);
      transform: scale(1.03);
    }
    .mode h2 {
      margin-bottom: 15px;
    }
    .toggle-switch {
      position: relative;
      display: inline-block;
      width: 60px;
      height: 34px;
    }
    .toggle-switch input { display: none; }
    .slider {
      position: absolute;
      cursor: pointer;
      background-color: #ccc;
      border-radius: 34px;
      top: 0; left: 0; right: 0; bottom: 0;
      transition: 0.4s;
    }
    .slider:before {
      position: absolute;
      content: "";
      height: 26px;
      width: 26px;
      left: 4px;
      bottom: 4px;
      background-color: white;
      border-radius: 50%;
      transition: 0.4s;
    }
    input:checked + .slider {
      background-color: #00ffae;
    }
    input:checked + .slider:before {
      transform: translateX(26px);
    }
    .timer {
      margin-top: 15px;
      font-size: 18px;
      color: #e0ffe0;
      text-shadow: 0 0 5px #00ffae;
    }

    /* Animated Restart Button */
    .restart-all {
      margin-top: 30px;
      padding: 14px 28px;
      background: linear-gradient(45deg, #ff416c, #ff4b2b);
      border: none;
      border-radius: 50px;
      color: white;
      font-size: 18px;
      font-weight: bold;
      cursor: pointer;
      box-shadow: 0 8px 20px rgba(255, 0, 100, 0.4);
      transition: all 0.3s ease;
      animation: pulseGlow 2s infinite;
    }
    .restart-all:hover {
      transform: scale(1.05);
      box-shadow: 0 0 25px #ff4b2b;
    }

    @keyframes pulseGlow {
      0% { box-shadow: 0 0 15px #ff4b2b; }
      50% { box-shadow: 0 0 30px #ff416c; }
      100% { box-shadow: 0 0 15px #ff4b2b; }
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: scale(0.95); }
      to { opacity: 1; transform: scale(1); }
    }

    @keyframes fadeInUp {
      from { transform: translateY(50px); opacity: 0; }
      to { transform: translateY(0); opacity: 1; }
    }
  </style>
</head>
<body>

<h1>Mode Toggle System</h1>

<div class="mode" id="study">
  <h2>Study Mode</h2>
  <label class="toggle-switch">
    <input type="checkbox" onchange="toggleMode('study')">
    <span class="slider"></span>
  </label>
  <div class="timer" id="study-timer">Time: 00:00:000</div>
</div>

<div class="mode" id="game">
  <h2>Game Mode</h2>
  <label class="toggle-switch">
    <input type="checkbox" onchange="toggleMode('game')">
    <span class="slider"></span>
  </label>
  <div class="timer" id="game-timer">Time: 00:00:000</div>
</div>

<div class="mode" id="notes">
  <h2>Notes Mode</h2>
  <label class="toggle-switch">
    <input type="checkbox" onchange="toggleMode('notes')">
    <span class="slider"></span>
  </label>
  <div class="timer" id="notes-timer">Time: 00:00:000</div>
</div>

<!-- Restart Button with Animation -->
<button class="restart-all" onclick="resetAll()">⟳ Restart All</button>

<script>
  let intervals = { study: null, game: null, notes: null };
  let currentMode = null;
  let startTimes = {};
  let elapsedOffsets = {
    study: parseInt(localStorage.getItem('study') || '0'),
    game: parseInt(localStorage.getItem('game') || '0'),
    notes: parseInt(localStorage.getItem('notes') || '0')
  };

  function formatTime(ms) {
    let minutes = String(Math.floor(ms / 60000)).padStart(2, '0');
    let seconds = String(Math.floor((ms % 60000) / 1000)).padStart(2, '0');
    let milliseconds = String(ms % 1000).padStart(3, '0');
    return `Time: ${minutes}:${seconds}:${milliseconds}`;
  }

  function updateAllTimers() {
    ['study', 'game', 'notes'].forEach(mode => {
      document.getElementById(`${mode}-timer`).innerText = formatTime(elapsedOffsets[mode]);
    });
  }

  updateAllTimers();

  function toggleMode(modeId) {
    const modes = ['study', 'game', 'notes'];

    if (currentMode === modeId) {
      clearInterval(intervals[modeId]);
      document.getElementById(modeId).classList.remove('active');
      document.querySelector(`#${modeId} input`).checked = false;
      elapsedOffsets[modeId] += Date.now() - startTimes[modeId];
      localStorage.setItem(modeId, elapsedOffsets[modeId]);
      currentMode = null;
      return;
    }

    modes.forEach(mode => {
      clearInterval(intervals[mode]);
      document.getElementById(mode).classList.remove('active');
      document.querySelector(`#${mode} input`).checked = false;
      if (mode === currentMode) {
        elapsedOffsets[mode] += Date.now() - startTimes[mode];
        localStorage.setItem(mode, elapsedOffsets[mode]);
      }
    });

    currentMode = modeId;
    startTimes[modeId] = Date.now();
    document.getElementById(modeId).classList.add('active');
    document.querySelector(`#${modeId} input`).checked = true;

    intervals[modeId] = setInterval(() => {
      let totalElapsed = Date.now() - startTimes[modeId] + elapsedOffsets[modeId];
      document.getElementById(`${modeId}-timer`).innerText = formatTime(totalElapsed);
    }, 100);
  }

  function resetAll() {
    const modes = ['study', 'game', 'notes'];
    modes.forEach(mode => {
      clearInterval(intervals[mode]);
      elapsedOffsets[mode] = 0;
      localStorage.setItem(mode, 0);
      document.getElementById(`${mode}-timer`).innerText = formatTime(0);
      document.querySelector(`#${mode} input`).checked = false;
      document.getElementById(mode).classList.remove('active');
    });
    currentMode = null;
  }
</script>

</body>
</html>
