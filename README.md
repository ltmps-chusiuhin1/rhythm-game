<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>藍循節奏大師: 1.0正式版</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <meta name="apple-mobile-web-app-capable" content="yes">

  <!-- Firebase & Google Identity Services -->
  <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
  <script src="https://accounts.google.com/gsi/client" async defer></script>

  <style>
    :root {
      --bg-dark: #020617;
      --glass-panel: rgba(15, 25, 45, 0.35);
      --glass-border: rgba(255, 255, 255, 0.2);
      --glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.4);
      --glass-blur: blur(12px);
      --p1-color: #22d3ee;
      --p2-color: #f472b6;
      --rank-s: #22d3ee;
      --rank-a: #4ade80;
      --rank-b: #facc15;
      --rank-c: #fb923c;
      --rank-d: #f87171;
      --comp-gold: #fbbf24;
      --prog-color: #a78bfa;
      --text-primary: #f8fafc;
      --text-secondary: #94a3b8;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      background-color: var(--bg-dark);
      color: var(--text-primary);
      font-family: 'Segoe UI', Roboto, Helvetica, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
      user-select: none;
      -webkit-user-select: none;
      touch-action: none;
      -webkit-touch-callout: none;
      position: relative;
    }

    .bg-layer {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      z-index: 0;
      pointer-events: none;
      background: radial-gradient(circle at center, #0f172a 0%, #020617 100%);
    }

    .aurora-container {
      position: absolute;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      overflow: hidden;
      opacity: 0.5;
      z-index: 1;
      mix-blend-mode: screen;
    }

    .aurora-blob {
      position: absolute;
      border-radius: 50%;
      filter: blur(100px);
      animation: aurora-flow 15s infinite alternate ease-in-out;
    }

    .aurora-1 { top: 20%; left: 10%; width: 60vw; height: 60vw; background: #6366f1; animation-delay: 0s; }
    .aurora-2 { bottom: 20%; right: 10%; width: 70vw; height: 70vw; background: #ec4899; animation-delay: 5s; }
    .aurora-3 { top: 30%; left: 30%; width: 50vw; height: 50vw; background: #06b6d4; animation-delay: 10s; }

    @keyframes aurora-flow {
      0% { transform: translate(0, 0) scale(0.8); }
      33% { transform: translate(15%, 10%) scale(1.1); }
      66% { transform: translate(-10%, 20%) scale(0.9); }
      100% { transform: translate(5%, 15%) scale(1.2); }
    }

    .shooting-star {
      position: absolute;
      width: 150px;
      height: 2px;
      background: linear-gradient(90deg, rgba(255,255,255,0.8), transparent);
      border-radius: 50%;
      z-index: 2;
      opacity: 0;
      transform: rotate(-45deg);
      filter: drop-shadow(0 0 6px white);
    }

    .ss-1 { top: 5%; left: 80%; animation: shooting 4s linear infinite 0.5s; }
    .ss-2 { top: 15%; left: 100%; animation: shooting 5s linear infinite 2s; }
    .ss-3 { top: 40%; left: 120%; animation: shooting 6s linear infinite 3.5s; }
    .ss-4 { top: 10%; left: 60%; animation: shooting 4.5s linear infinite 1s; }
    .ss-5 { top: 25%; left: 90%; animation: shooting 5.5s linear infinite 4s; }

    @keyframes shooting {
      0% { transform: translate(0, 0) rotate(-45deg); opacity: 1; }
      15% { opacity: 1; }
      100% { transform: translate(-2000px, 2000px) rotate(-45deg); opacity: 0; }
    }

    .cyber-grid {
      position: absolute;
      width: 200%;
      height: 200%;
      bottom: 50%;
      left: -50%;
      background-image: linear-gradient(rgba(255,255,255,0.04) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,0.04) 1px, transparent 1px);
      background-size: 60px 60px;
      background-position: center bottom;
      transform: perspective(500px) rotateX(60deg);
      animation: gridMove 4s linear infinite;
      mask-image: linear-gradient(to top, rgba(0,0,0,1) 0%, rgba(0,0,0,0) 60%);
      -webkit-mask-image: linear-gradient(to top, rgba(0,0,0,1) 0%, rgba(0,0,0,0) 60%);
      z-index: 3;
    }

    @keyframes gridMove {
      0% { transform: perspective(500px) rotateX(60deg) translateY(0); }
      100% { transform: perspective(500px) rotateX(60deg) translateY(60px); }
    }

    .particle {
      position: absolute;
      background: white;
      border-radius: 50%;
      opacity: 0;
      animation: floatUp 5s linear infinite;
      z-index: 4;
    }

    .p-1 { left: 10%; width: 2px; height: 2px; animation-duration: 4s; animation-delay: 0s; }
    .p-2 { left: 30%; width: 3px; height: 3px; animation-duration: 7s; animation-delay: 1s; }
    .p-3 { left: 70%; width: 2px; height: 2px; animation-duration: 5s; animation-delay: 2s; }
    .p-4 { left: 90%; width: 4px; height: 4px; animation-duration: 6s; animation-delay: 0.5s; }

    @keyframes floatUp {
      0% { bottom: 10px; opacity: 0; }
      50% { opacity: 0.6; box-shadow: 0 0 10px white; }
      100% { bottom: 100%; opacity: 0; }
    }

    #game-wrapper {
      position: relative;
      width: 100%;
      max-width: 1100px;
      height: 100%;
      max-height: 680px;
      background: rgba(15, 23, 42, 0.15);
      border-radius: 24px;
      overflow: hidden;
      z-index: 10;
      backdrop-filter: blur(5px);
      border: 1px solid rgba(255,255,255,0.1);
      animation: borderPulse 4s infinite alternate;
    }

    @media (min-height: 700px) {
      #game-wrapper {
        transform: translateY(-4vh);
      }
    }

    @keyframes borderPulse {
      0% { box-shadow: 0 0 40px rgba(34, 211, 238, 0.15), inset 0 0 15px rgba(34, 211, 238, 0.1); border-color: rgba(34, 211, 238, 0.2); }
      100% { box-shadow: 0 0 60px rgba(244, 114, 182, 0.25), inset 0 8px 25px rgba(244, 114, 182, 0.15); border-color: rgba(244, 114, 182, 0.3); }
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }

    .touch-layer {
      position: absolute;
      left: 0;
      width: 100%;
      height: 50%;
      z-index: 10;
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
    }
    #touch-p1 { top: 0; }
    #touch-p2 { top: 50%; }

    #ui-layer {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      display: none;
      flex-direction: column;
      justify-content: space-between;
      padding: 25px;
      box-sizing: border-box;
      z-index: 20;
    }

    .hud-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 20px;
      font-weight: 300;
      letter-spacing: 1px;
      padding: 8px 20px;
      border-radius: 12px;
      background: rgba(0, 0, 0, 0.4);
      border: 1px solid var(--glass-border);
      backdrop-filter: blur(8px);
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    }

    .p1-hud { border-left: 4px solid var(--p1-color); color: var(--p1-color); }
    .p2-hud { border-left: 4px solid var(--p2-color); color: var(--p2-color); opacity: 0; }
    .lives { letter-spacing: 2px; font-size: 22px; min-width: 100px; text-align: right; display: flex; justify-content: flex-end; gap: 5px; }

    .hud-center {
      position: absolute;
      top: 30px;
      left: 50%;
      transform: translateX(-50%);
      color: var(--text-secondary);
      font-size: 14px;
      font-weight: 700;
      letter-spacing: 2px;
      background: rgba(0,0,0,0.6);
      padding: 5px 15px;
      border-radius: 20px;
      border: 1px solid var(--glass-border);
      backdrop-filter: blur(4px);
    }

    #center-notify {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 90px;
      font-weight: 900;
      text-transform: uppercase;
      opacity: 0;
      white-space: nowrap;
      filter: drop-shadow(0 0 20px rgba(255, 255, 255, 0.5));
      transition: all 0.2s ease-out;
      z-index: 50;
    }

    #btn-pause {
      display: none;
      position: absolute;
      bottom: 25px;
      left: 50%;
      transform: translateX(-50%);
      padding: 8px 30px;
      background: rgba(0,0,0,0.4);
      border: 1px solid var(--glass-border);
      border-radius: 12px;
      color: white;
      font-weight: bold;
      cursor: pointer;
      z-index: 60;
      backdrop-filter: blur(5px);
      transition: 0.2s;
      font-size: 16px;
      pointer-events: auto;
    }

    #btn-pause:hover {
      background: rgba(255,255,255,0.2);
      border-color: white;
      transform: translateX(-50%) scale(1.05);
    }

    .s1-slogan {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 15px;
      font-size: 48px;
      font-weight: 900;
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.5s ease-in-out;
      z-index: 15;
      font-family: "Comic Sans MS", 'Varela Round', 'Arial Rounded MT Bold', 'Segoe UI', sans-serif;
    }
    .s1-slogan.visible { opacity: 1; }

    .screen {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: none;
      flex-direction: column;
      padding: 40px;
      background: var(--glass-panel);
      backdrop-filter: var(--glass-blur);
      z-index: 30;
    }
    .screen.active { display: flex; }

    .screen-header h1 {
      font-size: 36px;
      font-weight: 800;
      margin-bottom: 10px;
    }

    .subtitle {
      color: var(--text-secondary);
      font-size: 16px;
    }

    .main-content-area {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .control-panel {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
      width: 100%;
      max-width: 600px;
    }

    .control-group {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }
    .control-group.full { grid-column: span 2; }

    .control-group label {
      font-size: 14px;
      color: var(--text-secondary);
    }

    .toggle-wrapper {
      display: flex;
      gap: 10px;
    }

    .toggle-btn, select, .btn {
      padding: 10px 16px;
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid var(--glass-border);
      border-radius: 8px;
      color: white;
      cursor: pointer;
      font-size: 14px;
      transition: all 0.2s;
    }

    .toggle-btn.active, .btn-start {
      background: var(--p1-color);
      color: #020617;
      font-weight: bold;
      border-color: var(--p1-color);
    }

    .btn-gold {
      background: var(--comp-gold);
      color: #020617;
      font-weight: bold;
      border-color: var(--comp-gold);
    }

    .screen-footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .btn-group {
      display: flex;
      gap: 15px;
      width: 100%;
      justify-content: flex-end;
    }

    .choice-card {
      background: #0f172a;
      padding: 30px;
      border-radius: 16px;
      border: 1px solid var(--glass-border);
      text-align: center;
      margin: auto;
      max-width: 400px;
      width: 100%;
    }
    .choice-title { font-size: 22px; font-weight: bold; margin-bottom: 10px; }
    .choice-desc { color: var(--text-secondary); margin-bottom: 20px; }
  </style>
</head>
<body>

  <audio id="bgm-season1" src="https://ltmps-chusiuhin.github.io/rhythm-game/30.mp3" preload="auto" style="display:none;"></audio>
  <audio id="bgmPlayer" style="display:none;"></audio>

  <div class="bg-layer">
    <div class="aurora-container">
      <div class="aurora-blob aurora-1"></div>
      <div class="aurora-blob aurora-2"></div>
      <div class="aurora-blob aurora-3"></div>
    </div>
    <div class="shooting-star ss-1"></div>
    <div class="shooting-star ss-2"></div>
    <div class="shooting-star ss-3"></div>
    <div class="shooting-star ss-4"></div>
    <div class="shooting-star ss-5"></div>
    <div class="cyber-grid"></div>
    <div class="particle p-1"></div>
    <div class="particle p-2"></div>
    <div class="particle p-3"></div>
    <div class="particle p-4"></div>
  </div>

  <div id="game-wrapper">
    <canvas id="gameCanvas" width="1100" height="680"></canvas>
    <div id="touch-p1" class="touch-layer"></div>
    <div id="touch-p2" class="touch-layer"></div>
    
    <div id="ui-layer" style="display: none;">
      <div id="season1-slogan-top" class="s1-slogan">
        <span style="--i:1; color: #ffb3ba;">藍</span>
        <span style="--i:2; color: #ffd1a9;">循</span>
        <span style="--i:3; color: #fde047;">卅</span>
        <span style="--i:4; color: #baffc9;">載</span>
        <span style="--i:5; color: #bae1ff;">傅</span>
        <span style="--i:6; color: #dcbaff;">承</span>
      </div>
      <div id="season1-slogan-bottom" class="s1-slogan">
        <span style="--i:1; color: #ffb3ba;">信</span>
        <span style="--i:2; color: #ffd1a9;">守</span>
        <span style="--i:3; color: #fde047;">真</span>
        <span style="--i:4; color: #baffc9;">理</span>
        <span style="--i:5; color: #bae1ff;">育</span>
        <span style="--i:6; color: #dcbaff;">英</span>
      </div>
      
      <div class="hud-bar p1-hud" id="hud-p1">
        <div>P1 SCORE <span id="p1-score" style="margin-left:10px; color:white;">0</span></div>
        <div id="p1-lives" class="lives"></div>
      </div>
      <div class="hud-center" id="round-info">ROUND 3/6</div>
      <div id="center-notify">PLAY</div>
      <div class="hud-bar p2-hud" id="hud-p2">
        <div>P2 SCORE <span id="p2-score" style="margin-left:10px; color:white;">0</span></div>
        <div id="p2-lives" class="lives"></div>
      </div>

      <button id="btn-pause" onclick="pauseGame()">返回主選單</button>
    </div>

    <!-- 主選單 -->
    <div id="menu-screen" class="screen active">
      <div class="screen-header">
        <h1>藍循節奏大師</h1>
        <p class="subtitle">先聆聽四拍節奏，然後擊打出相同的節奏。</p>
      </div>
      <div class="main-content-area">
        <div class="control-panel">
          <div class="control-group" id="player-group">
            <label>玩家人數</label>
            <div class="toggle-wrapper">
              <button class="toggle-btn active" onclick="setPlayers(1)">單人(1P)</button>
              <button class="toggle-btn" onclick="setPlayers(2)">雙人(2P)</button>
            </div>
          </div>
          <div class="control-group">
            <label>遊戲模式</label>
            <div class="toggle-wrapper">
              <button class="toggle-btn active" onclick="setMode('fixed')">練習</button>
              <button class="toggle-btn" onclick="setMode('season1')">賽季1</button>
            </div>
          </div>
          <div id="mode-desc" class="control-group full">
            <div><span style="color:var(--p1-color); font-weight:bold;">[1P]</span> 單人模式：全螢幕判定，使用 [空白鍵] 或點擊螢幕。</div>
          </div>
        </div>
      </div>
      <div class="screen-footer">
        <div class="btn-group">
          <button class="btn btn-secondary" id="btn-leaderboard" onclick="openLeaderboardMenu()">名人堂</button>
          <button class="btn btn-start" onclick="startGame()">開始遊戲</button>
          <button class="btn btn-gold" onclick="openCompetitionRanking()">賽季龍虎榜</button>
        </div>
      </div>
    </div>

    <!-- 名人堂 -->
    <div id="leaderboard-screen" class="screen">
      <div class="screen-header"><h1>名人堂</h1></div>
      <div class="screen-footer">
        <button class="btn btn-secondary" onclick="showMenu()">返回主選單</button>
      </div>
    </div>

    <!-- 賽季競賽 -->
    <div id="competition-screen" class="screen">
      <div class="screen-header"><h1 style="color:var(--comp-gold);">賽季龍虎榜</h1></div>
      <div class="screen-footer">
        <button class="btn btn-secondary" onclick="showMenu()">返回主選單</button>
      </div>
    </div>

    <!-- 結算畫面 -->
    <div id="result-screen" class="screen">
      <div class="screen-header"><h1>訓練結束</h1></div>
      <div class="screen-footer">
        <button class="btn btn-start" onclick="resetToMenu()">返回選單</button>
      </div>
    </div>

    <!-- 暫停視窗 -->
    <div id="pause-overlay" class="screen" style="z-index: 100; background: rgba(0,0,0,0.8);">
      <div class="choice-card">
        <div class="choice-title">暫停遊戲</div>
        <div class="choice-desc">是否確定放棄當前進度並返回主選單？</div>
        <div style="display: flex; gap:10px; justify-content: center;">
          <button class="btn btn-secondary" onclick="resumeGame()">繼續遊戲</button>
          <button class="btn btn-start" onclick="quitToMenu()">確定離開</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    const SEASON1_BPM = 75;
    const SEASON1_MUSIC_START_DELAY = 1.0;
    const SEASON1_CHART_OFFSET = 1.52;
    const SEASON1_NOTE_FLIGHT_TIME = 3.0;

    const SEASON1_MAP = "1010000010100000101000001010000010100100101000001010000011101010110000001010110011";
    const SEASON1_LYRICS = "藍天碧海春風化雨暖我心無盡旅途伴快樂一世間".replace(/\s+/g, '');

    const firebaseConfig = {
      apiKey: "AIzaSyD4rb1PHX9TCxj8VORCnZWBIk6HPDZK8NE",
      authDomain: "ltmps-rhythm-game.firebaseapp.com",
      projectId: "ltmps-rhythm-game",
      storageBucket: "ltmps-rhythm-game.appspot.com",
      messagingSenderId: "1035987577062",
      appId: "1:1035987577062:web:0c7e1625fa4c22a9aff727"
    };

    let db;
    let isLBLocked = false;
    let lockDocRef = null;
    let isGoogleLoggedIn = false;
    let googleUserInfo = null;
    let tokenClient;
    let gameState = "MENU";
    let config = { players: 1, mode: 'fixed' };
    let timeouts = [];
    let animId = null;
    let session = {};

    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;

    window.onload = function() {
      try {
        if (firebaseConfig.apiKey) {
          firebase.initializeApp(firebaseConfig);
          db = firebase.firestore();
          lockDocRef = db.collection('config').doc('leaderboard_settings');
          lockDocRef.onSnapshot((doc) => {
            if (doc.exists) {
              isLBLocked = doc.data().locked === true;
            } else {
              isLBLocked = false;
            }
          });
        }
      } catch (e) {
        console.error("Firebase Init Error:", e);
      }
    };

    function showMenu() {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById('menu-screen').classList.add('active');
    }

    function pauseGame() {
      if (gameState !== 'PLAYING') return;
      gameState = 'PAUSED';
      document.getElementById('pause-overlay').classList.add('active');
    }

    function resumeGame() {
      document.getElementById('pause-overlay').classList.remove('active');
      gameState = 'PLAYING';
    }

    function quitToMenu() {
      document.getElementById('pause-overlay').classList.remove('active');
      resetToMenu();
    }

    function resetToMenu() {
      gameState = "MENU";
      timeouts.forEach(clearTimeout);
      timeouts = [];
      if (animId) {
        cancelAnimationFrame(animId);
        animId = null;
      }
      ctx.clearRect(0, 0, width, height);
      document.getElementById('ui-layer').style.display = "none";
      showMenu();
    }

    function startGame() {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById('ui-layer').style.display = "flex";
      document.getElementById('btn-pause').style.display = "block";
      gameState = "PLAYING";
    }

    function openLeaderboardMenu() {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById('leaderboard-screen').classList.add('active');
    }

    function openCompetitionRanking() {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById('competition-screen').classList.add('active');
    }

    function setPlayers(num) {
      config.players = num;
    }

    function setMode(mode) {
      config.mode = mode;
    }
  </script>
</body>
</html>
