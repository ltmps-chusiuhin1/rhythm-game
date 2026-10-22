<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>藍循節奏大師 Rhythm Game</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }
    body {
      background-color: #030305;
      color: #ffffff;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      overflow: hidden;
    }
    .bg-layer {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: -1;
    }
    #game-wrapper {
      position: relative;
      width: 1100px;
      height: 680px;
      background: rgba(0, 0, 0, 0.8);
      border: 2px solid rgba(255, 255, 255, 0.2);
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 0 30px rgba(0, 242, 254, 0.3);
    }
    #gameCanvas {
      width: 100%;
      height: 100%;
      display: block;
    }
    .touch-layer {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
    }
    .screen {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: none;
    }
    .screen.active {
      display: flex;
      flex-direction: column;
    }
    .btn {
      padding: 10px 20px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-weight: bold;
    }
    .btn-start {
      background-color: #00f2fe;
      color: #000;
    }
    .btn-secondary {
      background-color: #333;
      color: #fff;
    }
    .btn-gold {
      background-color: #facc15;
      color: #000;
    }
    .disabled {
      opacity: 0.5;
      pointer-events: none;
    }
  </style>
  <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-firestore.js"></script>
  <script src="https://accounts.google.com/gsi/client" async defer></script>
</head>
<body>

  <!-- 背景音樂 -->
  <audio id="bgm-season1" src="https://ltmps-chusiuhin.github.io/rhythm-game/30.mp3" preload="auto" style="display:none;"></audio>
  <audio id="bgmPlayer" style="display:none;"></audio>

  <!-- 背景視覺與特效特效 -->
  <div class="bg-layer">
    <!-- 動態極光流光特效 -->
    <div class="aurora-container">
      <div class="aurora-blob aurora-1"></div>
      <div class="aurora-blob aurora-2"></div>
      <div class="aurora-blob aurora-3"></div>
    </div>
    <!-- 流星雨特效 -->
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
      <!-- Slogan 與吉祥物 -->
      <div id="season1-slogan-top" class="sl-slogan"></div>
      <div id="season1-slogan-bottom" class="sl-slogan"></div>
      <img id="mascot-left" src="left.png" class="mascot" alt="Mascot Left">
      <img id="mascot-right" src="right.png" class="mascot" alt="Mascot Right">
      
      <div class="hud-bar p1-hud" id="hud-p1"></div>
      <div class="hud-center" id="round-info">ROUND 3/6</div>
      <div id="center-notify" style="color: rgb(250, 204, 21); opacity: 0; top: 50%; transform: translate(-50%, -50%) scale(1);">PLAY</div>
      <div class="hud-bar p2-hud" id="hud-p2" style="margin-top: auto; opacity: 0;"></div>
      
      <!-- 返回主選單/暫停按鈕 -->
      <button id="btn-pause" onclick="pauseGame()" style="display: none;">返回選單</button>
    </div>

    <!-- 主選單畫面 -->
    <div id="menu-screen" class="screen active">
      <div class="screen-header">
        <h1>藍循節奏大師</h1>
        <p class="subtitle">先聆聽四拍節奏，然後擊打出相同的節奏。</p>
        <div class="season-banner"></div>
      </div>
      <div class="main-content-area">
        <div class="control-panel">
          <div class="control-group" id="player-group"></div>
          <div class="control-group"></div>
          <div id="mode-desc" class="mode-description"></div>
          <div class="control-group" id="round-group"></div>
          <div class="control-group"></div>
          <div class="control-group full"></div>
        </div>
      </div>
      <div class="screen-footer">
        <div class="btn-group">
          <div class="leaderboard-btn-wrapper"></div>
          <button class="btn btn-start" onclick="startGame()">開始遊戲</button>
          <button id="btn-leaderboard" class="btn btn-secondary btn-gold" onclick="openCompetitionRanking()">龍虎榜</button>
          <button id="btn-lb-lock" onclick="toggleLeaderboardLock()" style="margin-left: 5px;">🔓</button>
        </div>
      </div>
    </div>

    <!-- 名人堂/排行榜畫面 -->
    <div id="leaderboard-screen" class="screen">
      <div class="screen-header">
        <h1>名人堂</h1>
      </div>
      <div class="scroll-content">
        <div class="leaderboard-section active"></div>
        <div class="screen-footer">
          <button class="btn btn-secondary" style="width:auto; padding: 12px 30px;" onclick="showMenu()">返回主選單</button>
        </div>
      </div>
    </div>

    <!-- 競賽畫面 -->
    <div id="competition-screen" class="screen">
      <div class="screen-header"></div>
      <div class="scroll-content"></div>
      <div class="screen-footer"></div>
    </div>

    <!-- 結算結果畫面 -->
    <div id="result-screen" class="screen">
      <div class="screen-header"></div>
      <div class="scroll-content" id="result-scroll-container"></div>
      <div class="screen-footer">
        <button class="btn btn-start" onclick="resetToMenu()">返回選單</button>
      </div>
    </div>

    <!-- 暫停確認視窗 -->
    <div id="pause-overlay" class="screen" style="z-index: 100;">
      <div class="choice-card">
        <div class="choice-title">暫停遊戲</div>
        <div class="choice-desc">是否確定放棄當前進度並返回主選單？</div>
        <div style="display: flex; gap:10px; margin-top:20px; justify-content: center;">
          <button class="btn btn-secondary" onclick="resumeGame()">繼續遊戲</button>
          <button class="btn btn-start" onclick="confirmQuit()">確定離開</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    // 1: 自定義音樂與譜面參數設定
    const SEASON1_BPM = 75; // 歌曲速度 (BPM)
    const SEASON1_MUSIC_START_DELAY = 1.0; // 按下開始後，等待多少秒才開始播放音樂 (秒)
    const SEASON1_CHART_OFFSET = 1.57; // 譜面偏移 (秒)

    // 譜面設定：每個字元代表一個「十六分音符」的長度 (1=打擊, 0=不打擊)
    const SEASON1_MAP = "10000010100000101000001010000010100000101000001010010010100000101000001010000011101010110000001010110011100100101001001010000000000000101000001010000011101010110000001010000010100000101010101100000010100000101000001110101011000000101011001110010010101010110000001110101011000000111010101100000011101010110000001110010010100000000010101010010011101010100010000000101010100100101010101010110000001010101001001110101010100100101000001010110010100100101000000000000000";

    // 歌詞設定：系統會自動去除空格，並依序填入音符中
    const SEASON1_LYRICS = "藍天碧海春風化雨暖我心無盡 旅途伴快樂一世間 恩典早已充滿我心 勤學 勤道效主愛人樂與學在藍循我在長廊裡禱告你在球場上躍步帶著疑 與苦 也會為我引路我結伴同行 向天際遇星河大海內奮鬥得到知識再破巨浪你我在藍循 那美夢全然綻放用新世界的光去共創".replace(/\s+/g, '');

    // Firebase 配置
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

    const AudioEngine = {
      ctx: new (window.AudioContext || window.webkitAudioContext)()
    };

    window.onload = function () {
      if (window.google) {
        tokenClient = google.accounts.oauth2.initTokenClient({
          client_id: '573911894747-8c28hfr0bmkjnn2pkm2msg6sjal1thqe.apps.googleusercontent.com',
          scope: 'email profile',
          callback: (tokenResponse) => {
            if (tokenResponse && tokenResponse.access_token) {
              fetch("https://www.googleapis.com/oauth2/v3/userinfo", {
                headers: { Authorization: `Bearer ${tokenResponse.access_token}` }
              }).then(res => res.json()).then(data => {
                isGoogleLoggedIn = true;
                googleUserInfo = data;
                alert("Google 登入成功！歡迎 " + data.name);
                setModeInternal("season1");
              }).catch(err => {
                console.error("Error fetching user info:", err);
                alert("獲取用戶資訊失敗，請重試。");
              });
            }
          }
        });
      }

      try {
        if (firebaseConfig.apiKey && firebaseConfig.apiKey !== "您的_API_KEY") {
          firebase.initializeApp(firebaseConfig);
          db = firebase.firestore();
          lockDocRef = db.collection('config').doc('leaderboard_settings');
          lockDocRef.onSnapshot((doc) => {
            if (doc.exists) {
              isLBLocked = doc.data().locked === true;
            } else {
              isLBLocked = false;
            }
            updateLockUI();
          });
        }
      } catch (e) {
        console.error("Firebase Init Error:", e);
      }
    };

    function toggleLeaderboardLock() {
      let actionStr = isLBLocked ? "解鎖" : "上鎖";
      let pwd = prompt(`請輸入密碼以進行「${actionStr}」:`);
      if (pwd === "Ltmps") {
        setLockState(!isLBLocked);
      } else if (pwd !== null && pwd !== "") {
        alert("密碼錯誤！");
      }
    }

    function setLockState(locked) {
      if (lockDocRef) {
        lockDocRef.set({ locked: locked }, { merge: true }).then(() => {
          isLBLocked = locked;
          updateLockUI();
          alert(locked ? "名人堂已成功上鎖。" : "名人堂已開放。");
        }).catch(err => {
          console.error("無法更新雲端狀態", err);
          alert("更新狀態失敗！請檢查 Firebase 資料庫的寫入權限。");
        });
      } else {
        isLBLocked = locked;
        updateLockUI();
      }
    }

    function updateLockUI() {
      const btn = document.getElementById('btn-leaderboard');
      const lockBtn = document.getElementById("btn-lb-lock");
      if (btn && lockBtn) {
        if (isLBLocked) {
          lockBtn.innerText = "🔒";
          btn.innerText = "名人堂 (已上鎖)";
          btn.classList.add('disabled');
        } else {
          lockBtn.innerText = "🔓";
          btn.innerText = "龍虎榜";
          btn.classList.remove('disabled');
        }
      }
    }

    const DESCRIPTIONS = {
      players: {
        1: "單人模式：全螢幕判定，使用 [空白鍵]、[A] 或點擊螢幕。",
        2: "雙人對戰：P1 點擊上半屏/ [空白鍵/A]，P2 點擊下半屏/ [1/4]。"
      },
      modes: {
        "fixed": "練習模式：標準玩法，適合體驗不同的速度及難度。",
        "vanishing": "消失模式：音符會在途中消失，考驗你對節奏的記憶力。",
        "survival": "生存模式：無盡挑戰直至出現多次失誤，挑戰你的集中力。",
        "vanishing-survival": "生存消失：消失與生存的混合模式，大師級的峰值挑戰。",
        "progressive": "漸進模式：無盡挑戰直至出現5次失誤，速度將逐漸提升。",
        "season1": "賽季1：欣賞三十周年主題曲一邊擊打節奏！（需先登入 Google 帳戶）"
      }
    };

    function showMenu() {
      document.querySelectorAll(".screen").forEach(s => s.classList.remove('active'));
      document.getElementById("menu-screen").classList.add("active");
      updateDescription();
    }

    function pauseGame() {
      if (gameState !== "PLAYING") return;
      gameState = "PAUSED";
      if (AudioEngine.ctx) AudioEngine.ctx.suspend();
      const bgm = document.getElementById('bgm-season1');
      if (bgm && !bgm.paused) {
        bgm.pause();
        session.bgmWasPlaying = true;
      } else {
        session.bgmWasPlaying = false;
      }
      document.getElementById("pause-overlay").classList.add("active");
    }

    function resumeGame() {
      document.getElementById("pause-overlay").classList.remove('active');
      gameState = "PLAYING";
      if (AudioEngine.ctx) AudioEngine.ctx.resume();
      const bgm = document.getElementById('bgm-season1');
      if (bgm && session.bgmWasPlaying) {
        bgm.play();
      }
    }

    function confirmQuit() {
      document.getElementById("pause-overlay").classList.remove('active');
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

      if (session.fadeInterval) {
        clearInterval(session.fadeInterval);
        session.fadeInterval = null;
      }

      const bgmSeason = document.getElementById("bgm-season1");
      if (bgmSeason) {
        bgmSeason.pause();
        bgmSeason.currentTime = 0;
        bgmSeason.volume = 1.0;
      }

      if (AudioEngine.ctx && AudioEngine.ctx.state === "running") {
        AudioEngine.ctx.suspend();
      }

      document.getElementById("ui-layer").style.display = "none";
      document.getElementById('season1-slogan-top').classList.remove('visible');
      document.getElementById('season1-slogan-bottom').classList.remove('visible');
      document.getElementById('mascot-left').classList.remove('visible');
      document.getElementById('mascot-right').classList.remove('visible');
      document.getElementById('btn-pause').style.display = "none";
      
      showMenu();
    }

    function updateDescription() {
      const pText = DESCRIPTIONS.players[config.players] || "";
      const mText = DESCRIPTIONS.modes[config.mode] || "";
      const el = document.getElementById('mode-desc');
      
      let pColor = config.players === 1 ? "var(--p1-color, #00f2fe)" : "var(--p2-color, #ff007f)";
      let mColor = "white";

      if (config.mode.includes('vanishing')) mColor = '#d946ef';
      if (config.mode.includes('survival')) mColor = '#facc15';
      if (config.mode === 'vanishing-survival') mColor = '#187171';
      if (config.mode === 'progressive') mColor = 'var(--prog-color, #3b82f6)';
      if (config.mode === 'season1') mColor = 'var(--comp-gold, #f59e0b)';

      if (el) {
        el.innerHTML = `<div><span style="color:${pColor}; font-weight:bold;">[${config.players}P]</span> ${pText}</div>` +
                       `<div style="margin-top:3px;"><span style="color:${mColor}; font-weight:bold;">[${getModeName(config.mode)}]</span> ${mText}</div>`;
      }
    }

    function getModeName(m) {
      const names = {
        'fixed': '練習',
        'vanishing': '消失',
        'survival': '生存',
        'vanishing-survival': '極限',
        'progressive': '漸進',
        'season1': '賽季1'
      };
      return names[m] || m;
    }

    function startGame() {
      alert("遊戲開始！");
      document.querySelectorAll(".screen").forEach(s => s.classList.remove('active'));
      document.getElementById("ui-layer").style.display = "block";
      document.getElementById("btn-pause").style.display = "block";
      gameState = "PLAYING";
    }

    function openCompetitionRanking() {
      if (isLBLocked) {
        alert("名人堂目前已上鎖！");
        return;
      }
      document.querySelectorAll(".screen").forEach(s => s.classList.remove('active'));
      document.getElementById("leaderboard-screen").classList.add("active");
    }

    function setModeInternal(mode) {
      config.mode = mode;
      updateDescription();
    }
  </script>
</body>
</html>
