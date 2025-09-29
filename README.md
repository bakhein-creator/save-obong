<!DOCTYPE html>
<html lang="ko">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>오봉댐을 지켜라!</title>
   <style>
       @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');
       body {
           font-family: 'Inter', sans-serif;
           margin: 0;
           padding: 0;
           background-color: #2c3e50;
           display: flex;
           justify-content: center;
           align-items: center;
           min-height: 100vh;
           text-align: center;
           color: #ecf0f1;
           flex-direction: column;
       }
       #game-container {
           position: relative;
           border-radius: 1rem;
           box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
       }
       canvas {
           display: block;
       }
       #ui-container {
           position: absolute;
           top: 0;
           left: 0;
           width: 100%;
           display: flex;
           justify-content: space-between;
           align-items: center;
           padding: 1rem;
           box-sizing: border-box;
           color: #ecf0f1;
           font-weight: bold;
           font-size: 1.2rem;
           text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
       }
       #start-screen, #shop-screen, #game-over-screen, #revival-prompt {
           position: absolute;
           top: 0;
           left: 0;
           width: 100%;
           height: 100%;
           background-color: rgba(44, 62, 80, 0.9);
           display: flex;
           flex-direction: column;
           justify-content: center;
           align-items: center;
           z-index: 10;
           max-height: 100%;
           overflow-y: auto;
       }
       h1 {
           font-size: 3rem;
           margin-bottom: 1rem;
           color: #3498db;
       }
       p {
           font-size: 1.5rem;
           margin-bottom: 2rem;
       }
       .button {
           background-color: #3498db;
           color: white;
           padding: 0.75rem 2rem;
           border-radius: 2rem;
           font-size: 1.2rem;
           font-weight: bold;
           cursor: pointer;
           transition: transform 0.2s, background-color 0.2s;
           border: 2px solid #2980b9;
           margin: 0.5rem 0;
       }
       .button:hover {
           background-color: #2980b9;
           transform: scale(1.05);
       }
       .button:active {
           transform: scale(0.95);
       }
       .quit-button {
           background-color: #e74c3c;
           border-color: #c0392b;
       }
       .quit-button:hover {
           background-color: #c0392b;
       }
       #skin-selection {
           display: flex;
           gap: 1rem;
           margin-bottom: 2rem;
       }
       .skin-button {
           width: 80px;
           height: 80px;
           border-radius: 1rem;
           cursor: pointer;
           border: 3px solid transparent;
           transition: border-color 0.2s;
           background-size: contain;
           background-repeat: no-repeat;
           background-position: center;
           position: relative;
       }
       .skin-button.selected {
           border-color: #f1c40f;
       }
       .skin-button.owned::after {
           content: '✓';
           position: absolute;
           top: 5px;
           right: 5px;
           font-size: 1.5rem;
           color: #2ecc71;
           text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
       }
       .skin-button:not(.owned) {
           opacity: 0.5;
           cursor: not-allowed;
       }
       #skin1 { background-color: #f1c40f; }
       #skin2 { background-color: #e74c3c; }
       #skin3 { background-color: #9b59b6; }
       #skin4 { background-color: #34495e; }
       #difficulty-select {
           padding: 0.5rem;
           font-size: 1rem;
           border-radius: 0.5rem;
           border: none;
           margin-bottom: 1.5rem;
           background-color: #34495e;
           color: white;
       }
       .shop-item {
           display: flex;
           flex-direction: column;
           align-items: center;
           margin-bottom: 1.5rem;
       }
       .shop-item span {
           font-size: 1rem;
           margin-bottom: 0.5rem;
       }
   </style>
</head>
<body>


<div id="game-container">
   <canvas id="gameCanvas"></canvas>


   <div id="ui-container">
       <div id="water-level">🌊 오봉댐 수위: 0%</div>
       <div id="ammo-count">🔫 0</div>
       <div id="score">점수: 0</div>
       <div id="coin-count">💰 0</div>
       <div id="in-game-items">
           <button id="useRemoverBtn" class="button" style="display: none;">방해꾼 제거 (0)</button>
           <button id="quitButton" class="button quit-button" style="display: none;">게임 종료</button>
       </div>
   </div>


   <div id="start-screen">
       <h1>오봉댐을 지켜라!</h1>
       <p>방향키(←, →)로 플레이어를 조종하세요.<br>스페이스바를 눌러 물방울을 발사하세요.<br>빗방울을 모아 총알을 충전하고,<br>댐을 마르게 하는 구름들을 막아 댐 수위를 지키세요!</p>
       <div id="skin-selection">
           <div class="skin-button" id="skin1" data-skin="default"></div>
           <div class="skin-button" id="skin2" data-skin="lava"></div>
           <div class="skin-button" id="skin3" data-skin="purple"></div>
           <div class="skin-button" id="skin4" data-skin="ocean"></div>
       </div>
       <select id="difficulty-select">
           <option value="easy">쉬움</option>
           <option value="normal" selected>보통</option>
           <option value="hard">어려움</option>
       </select>
       <button id="startButton" class="button">게임 시작</button>
       <button id="shopButton" class="button">상점</button>
   </div>


   <div id="shop-screen" style="display: none;">
       <h1>상점</h1>
       <p id="shopCoinCount">💰 내 코인: 0</p>
       <div class="shop-item">
           <span>부활권</span>
           <button id="buyRevival" class="button">구매 (15 코인)</button>
       </div>
       <div class="shop-item">
           <span>초강력 총</span>
           <button id="buySuperGun" class="button">구매 (30 코인)</button>
       </div>
       <div class="shop-item">
           <span>방해꾼 제거권 (3회)</span>
           <button id="buyRemover" class="button">구매 (15 코인)</button>
       </div>
       <div class="shop-item">
           <span>랜덤 스킨 상자</span>
           <button id="buyRandomBox" class="button">구매 (3 코인)</button>
       </div>
       <button id="backButton" class="button">돌아가기</button>
   </div>


   <div id="game-over-screen" style="display: none;">
       <h1>게임 오버!</h1>
       <p id="finalScore"></p>
       <button id="restartButton" class="button">다시 시작</button>
   </div>


   <div id="revival-prompt" style="display: none;">
       <h1>부활권을 사용하시겠습니까?</h1>
       <p id="revivalCountDisplay"></p>
       <button id="useRevivalBtn" class="button">사용</button>
       <button id="noRevivalBtn" class="button">게임 오버</button>
   </div>
</div>


<script>
   document.addEventListener('DOMContentLoaded', () => {
       const canvas = document.getElementById('gameCanvas');
       const ctx = canvas.getContext('2d');
       const waterLevelEl = document.getElementById('water-level');
       const ammoCountEl = document.getElementById('ammo-count');
       const scoreEl = document.getElementById('score');
       const coinCountEl = document.getElementById('coin-count');
       const startScreen = document.getElementById('start-screen');
       const shopScreen = document.getElementById('shop-screen');
       const gameOverScreen = document.getElementById('game-over-screen');
       const startButton = document.getElementById('startButton');
       const shopButton = document.getElementById('shopButton');
       const backButton = document.getElementById('backButton');
       const restartButton = document.getElementById('restartButton');
       const finalScoreEl = document.getElementById('finalScore');
       const skinButtons = document.querySelectorAll('.skin-button');
       const difficultySelect = document.getElementById('difficulty-select');
       const shopCoinCountEl = document.getElementById('shopCoinCount');
       const buyRevivalBtn = document.getElementById('buyRevival');
       const buySuperGunBtn = document.getElementById('buySuperGun');
       const buyRemoverBtn = document.getElementById('buyRemover');
       const buyRandomBoxBtn = document.getElementById('buyRandomBox');
       const useRemoverBtn = document.getElementById('useRemoverBtn');
      
       const revivalPrompt = document.getElementById('revival-prompt');
       const revivalCountDisplay = document.getElementById('revivalCountDisplay');
       const useRevivalBtn = document.getElementById('useRevivalBtn');
       const noRevivalBtn = document.getElementById('noRevivalBtn');
       const quitButton = document.getElementById('quitButton'); // 게임 종료 버튼 추가


       // Game state variables
       let gameActive = false;
       let score = 0;
       let waterLevel = 50;
       let playerAmmo = 10;
       let selectedSkin = 'default';
       let ownedSkins = new Set(['default']);
       let isPoweredUp = false;
       let powerUpTimer = 0;
       const powerUpDuration = 5000;
       let isSuperPoweredUp = false;
       let superPowerUpTimer = 0;
       const superPowerUpDuration = 10000;
       let difficultyLevel = 1;
       let nextDifficultyThreshold;
       let coins = 0;
       let lastCoinScore = 0;
       let revivalTickets = 0;
       let superGuns = 0;
       let thiefRemovers = 0;
       let collectEffects = [];
       let player = {
           x: 0,
           y: 0,
           width: 70,
           height: 70,
           speed: 5
       };
       let bullets = [];
       let raindrops = [];
       let enemies = [];
       let items = [];
       let rainParticles = [];
       let keys = {};
       let lastUpdateTime = 0;


       // Game settings
       const maxWaterLevel = 100;
       const maxPlayerAmmo = 100;
       const baseBulletSpeed = 7;
       const basePowerfulBulletSpeed = 10;
       const powerfulBulletSize = 10;
       const superPowerfulBulletSpeed = 15;
       const superPowerfulBulletSize = 15;
       const baseEnemySpeed = 1;
       const baseSuperEnemySpeedMultiplier = 1.5;
       const baseRaindropSpeed = 2;
       const ammoPerRaindrop = 10;
       const waterLossPerEnemy = 10;
       const scorePerNormalKill = 10;
       const scorePerSuperKill = 50;
       const waterCostPerShot = 1;
       const itemAmmoBoost = 20;
       const itemWaterBoost = 20;
       const passiveWaterIncreaseRate = 0.05;


       // Shop Costs
       const revivalCost = 15;
       const superGunCost = 30;
       const removerCost = 15;
       const randomBoxCost = 3;
       const allSkins = ['default', 'lava', 'purple', 'ocean'];
       const thiefAmmoLoss = {
           'easy': 10,
           'normal': 25,
           'hard': 50
       };


       function resizeCanvas() {
           canvas.width = window.innerWidth > 600 ? 500 : window.innerWidth * 0.9;
           canvas.height = window.innerHeight > 800 ? 700 : window.innerHeight * 0.9;
           player.x = canvas.width / 2;
           player.y = canvas.height - 80;
       }


       window.addEventListener('resize', resizeCanvas);
       resizeCanvas();


       function updateSkinButtons() {
           skinButtons.forEach(button => {
               const skin = button.dataset.skin;
               if (ownedSkins.has(skin)) {
                   button.classList.add('owned');
                   button.classList.add('selected');
                   button.style.opacity = 1;
                   button.style.cursor = 'pointer';
               } else {
                   button.classList.remove('owned');
                   button.classList.remove('selected');
                   button.style.opacity = 0.5;
                   button.style.cursor = 'not-allowed';
               }
               if (skin === selectedSkin) {
                    button.classList.add('selected');
               }
           });
       }
       updateSkinButtons();




       skinButtons.forEach(button => {
           button.addEventListener('click', () => {
               const skin = button.dataset.skin;
               if (ownedSkins.has(skin)) {
                   skinButtons.forEach(btn => btn.classList.remove('selected'));
                   button.classList.add('selected');
                   selectedSkin = skin;
               }
           });
       });
      
       document.addEventListener('keydown', (e) => {
           keys[e.key] = true;
           if (e.key === ' ' && gameActive) {
               shoot();
           }
       });


       document.addEventListener('keyup', (e) => {
           keys[e.key] = false;
       });


       startButton.addEventListener('click', startGame);
       restartButton.addEventListener('click', startGame);
       shopButton.addEventListener('click', showShop);
       backButton.addEventListener('click', showStartScreen);
       useRemoverBtn.addEventListener('click', useThiefRemover);
       quitButton.addEventListener('click', quitGame); // 게임 종료 버튼 이벤트 리스너 추가


       buyRevivalBtn.addEventListener('click', () => { buyItem('revival'); });
       buySuperGunBtn.addEventListener('click', () => { buyItem('superGun'); });
       buyRemoverBtn.addEventListener('click', () => { buyItem('remover'); });
       buyRandomBoxBtn.addEventListener('click', () => { buyItem('randomBox'); });
       useRevivalBtn.addEventListener('click', useRevivalTicket);
       noRevivalBtn.addEventListener('click', showGameOverScreen);
      
       function showStartScreen() {
           shopScreen.style.display = 'none';
           revivalPrompt.style.display = 'none';
           gameOverScreen.style.display = 'none'; // 게임 오버 화면 숨김
           startScreen.style.display = 'flex';
           quitButton.style.display = 'none'; // 게임 종료 버튼 숨김
           useRemoverBtn.style.display = 'none';
           updateUI();
           updateSkinButtons();
       }


       function showShop() {
           startScreen.style.display = 'none';
           shopScreen.style.display = 'flex';
           shopCoinCountEl.textContent = `💰 내 코인: ${coins}`;
       }


       function buyItem(item) {
           if (item === 'revival' && coins >= revivalCost) {
               coins -= revivalCost;
               revivalTickets++;
           } else if (item === 'superGun' && coins >= superGunCost) {
               coins -= superGunCost;
               isSuperPoweredUp = true;
               superPowerUpTimer = Date.now();
           } else if (item === 'remover' && coins >= removerCost) {
               coins -= removerCost;
               thiefRemovers++;
           } else if (item === 'randomBox' && coins >= randomBoxCost) {
               coins -= randomBoxCost;
               const newSkin = allSkins[Math.floor(Math.random() * allSkins.length)];
               ownedSkins.add(newSkin);
               updateSkinButtons();
           } else {
               return;
           }
           shopCoinCountEl.textContent = `💰 내 코인: ${coins}`;
           updateUI();
       }


       function useThiefRemover() {
           if (thiefRemovers > 0) {
               thiefRemovers--;
               enemies = enemies.filter(enemy => enemy.type !== 'thief');
               updateUI();
           }
       }
      
       function useRevivalTicket() {
           revivalTickets--;
           waterLevel = maxWaterLevel / 2;
           revivalPrompt.style.display = 'none';
           gameActive = true;
           updateUI();
           gameLoop();
       }


       function showRevivalPrompt() {
           gameActive = false;
           revivalCountDisplay.textContent = `남은 횟수: ${revivalTickets}회`;
           revivalPrompt.style.display = 'flex';
       }


       function showGameOverScreen() {
           revivalPrompt.style.display = 'none';
           gameActive = false;
           finalScoreEl.textContent = `최종 점수: ${score}`;
           gameOverScreen.style.display = 'flex';
           quitButton.style.display = 'none';
           useRemoverBtn.style.display = 'none';
       }
      
       function quitGame() {
           // 게임 종료 시 상태 초기화 (코인, 스킨은 유지)
           gameActive = false;
           score = 0;
           waterLevel = 50;
           playerAmmo = 10;
           bullets = [];
           raindrops = [];
           enemies = [];
           items = [];
           collectEffects = [];
           isPoweredUp = false;
           isSuperPoweredUp = false;
           showStartScreen(); // 시작 화면으로 돌아가기
       }


       function startGame() {
           const difficulty = difficultySelect.value;
           switch (difficulty) {
               case 'easy':
                   waterLevel = 75;
                   playerAmmo = 20;
                   nextDifficultyThreshold = 500;
                   break;
               case 'hard':
                   waterLevel = 25;
                   playerAmmo = 5;
                   nextDifficultyThreshold = 200;
                   break;
               case 'normal':
               default:
                   waterLevel = 50;
                   playerAmmo = 10;
                   nextDifficultyThreshold = 300;
                   break;
           }


           startScreen.style.display = 'none';
           gameOverScreen.style.display = 'none';
           gameActive = true;
           score = 0;
           isPoweredUp = false;
           isSuperPoweredUp = false;
           difficultyLevel = 1;
           bullets = [];
           raindrops = [];
           enemies = [];
           items = [];
           collectEffects = [];
           quitButton.style.display = 'block'; // 게임 시작 시 게임 종료 버튼 표시
           updateUI();
           gameLoop();
       }


       function gameOver() {
           if (revivalTickets > 0) {
               showRevivalPrompt();
           } else {
               showGameOverScreen();
           }
       }


       function updateUI() {
           waterLevelEl.textContent = `🌊 오봉댐 수위: ${waterLevel.toFixed(1)}%`;
           ammoCountEl.textContent = `🔫 ${playerAmmo}/${maxPlayerAmmo}`;
           scoreEl.textContent = `점수: ${score}`;
           coinCountEl.textContent = `💰 ${coins}`;
           useRemoverBtn.textContent = `방해꾼 제거 (${thiefRemovers})`;
           useRemoverBtn.style.display = thiefRemovers > 0 && gameActive ? 'block' : 'none';
       }


       function drawBackground() {
           const skyGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
           skyGradient.addColorStop(0, '#4a698c');
           skyGradient.addColorStop(1, '#6a89a8');
           ctx.fillStyle = skyGradient;
           ctx.fillRect(0, 0, canvas.width, canvas.height);
       }


       function drawPlayer() {
           ctx.fillStyle = '#2c3e50';
           ctx.beginPath();
           ctx.arc(player.x, player.y + 10, 20, 0, Math.PI * 2);
           ctx.fill();


           // 총 디자인
           switch (selectedSkin) {
               case 'lava':
                   // 용암 총 디자인
                   ctx.fillStyle = '#e74c3c';
                   ctx.beginPath();
                   ctx.moveTo(player.x, player.y - 30);
                   ctx.lineTo(player.x - 10, player.y);
                   ctx.lineTo(player.x + 10, player.y);
                   ctx.closePath();
                   ctx.fill();


                   ctx.fillStyle = '#f1c40f';
                   ctx.fillRect(player.x - 20, player.y - 15, 40, 15);


                   ctx.fillStyle = '#c0392b';
                   ctx.beginPath();
                   ctx.arc(player.x - 15, player.y + 15, 8, 0, Math.PI * 2);
                   ctx.arc(player.x + 15, player.y + 15, 8, 0, Math.PI * 2);
                   ctx.fill();
                   break;
               case 'purple':
                   // 보라색 총 디자인
                   ctx.fillStyle = '#9b59b6';
                   ctx.fillRect(player.x - 25, player.y - 5, 50, 15);


                   ctx.fillStyle = '#8e44ad';
                   ctx.fillRect(player.x - 5, player.y - 20, 10, 15);


                   ctx.fillStyle = '#ecf0f1';
                   ctx.fillRect(player.x - 20, player.y - 3, 5, 11);
                   ctx.fillRect(player.x + 15, player.y - 3, 5, 11);


                   ctx.beginPath();
                   ctx.arc(player.x, player.y + 10, 10, 0, Math.PI * 2);
                   ctx.fill();
                   break;
               case 'ocean':
                   // 오션 스킨 디자인
                   ctx.fillStyle = '#2c92b2';
                   ctx.fillRect(player.x - 25, player.y - 5, 50, 15);


                   ctx.fillStyle = '#1c718a';
                   ctx.fillRect(player.x - 5, player.y - 20, 10, 15);


                   ctx.fillStyle = '#b6d7e0';
                   ctx.beginPath();
                   ctx.moveTo(player.x - 25, player.y);
                   ctx.quadraticCurveTo(player.x - 10, player.y - 5, player.x, player.y);
                   ctx.quadraticCurveTo(player.x + 10, player.y + 5, player.x + 25, player.y);
                   ctx.fill();


                   ctx.beginPath();
                   ctx.arc(player.x, player.y + 10, 10, 0, Math.PI * 2);
                   ctx.fill();
                   break;
               case 'default':
               default:
                   // 기본 총 디자인
                   ctx.fillStyle = '#f1c40f';
                   ctx.fillRect(player.x - 20, player.y, 40, 15);


                   ctx.fillStyle = '#e67e22';
                   ctx.fillRect(player.x - 5, player.y + 15, 10, 20);


                   ctx.fillStyle = '#7f8c8d';
                   ctx.beginPath();
                   ctx.arc(player.x, player.y - 5, 8, 0, Math.PI * 2);
                   ctx.fill();
                  
                   break;
           }
       }


       function drawBullets() {
           bullets.forEach(bullet => {
               ctx.fillStyle = isSuperPoweredUp ? '#3498db' : (isPoweredUp ? '#f1c40f' : '#e74c3c');
               ctx.beginPath();
               ctx.arc(bullet.x, bullet.y, bullet.size, 0, Math.PI * 2);
               ctx.fill();
           });
       }


       function drawRaindrops() {
           ctx.fillStyle = '#85c1e9';
           raindrops.forEach(drop => {
               ctx.beginPath();
               ctx.moveTo(drop.x, drop.y);
               ctx.bezierCurveTo(drop.x + 15, drop.y + 40, drop.x - 15, drop.y + 40, drop.x, drop.y + 60);
               ctx.fill();
           });
       }


       function drawEnemies() {
           enemies.forEach(enemy => {
               if (enemy.type === 'thief') {
                   // 방해꾼(도둑) 디자인
                   ctx.fillStyle = '#555';
                   ctx.beginPath();
                   ctx.arc(enemy.x, enemy.y, 20, 0, Math.PI * 2);
                   ctx.fill();
                   ctx.fillStyle = '#333';
                   ctx.fillRect(enemy.x - 10, enemy.y - 10, 20, 15);
                   ctx.fillStyle = 'white';
                   ctx.font = '20px Arial';
                   ctx.fillText('💧', enemy.x, enemy.y + 5);
               } else if (enemy.isSuper) {
                   const gradient = ctx.createLinearGradient(enemy.x - 30, enemy.y, enemy.x + 30, enemy.y);
                   gradient.addColorStop(0, '#e74c3c');
                   gradient.addColorStop(0.2, '#f1c40f');
                   gradient.addColorStop(0.4, '#2ecc71');
                   gradient.addColorStop(0.6, '#3498db');
                   gradient.addColorStop(0.8, '#9b59b6');
                   gradient.addColorStop(1, '#e74c3c');
                   ctx.fillStyle = gradient;
                   ctx.beginPath();
                   ctx.arc(enemy.x, enemy.y, 25, 0, Math.PI * 2);
                   ctx.arc(enemy.x + 30, enemy.y + 15, 25, 0, Math.PI * 2);
                   ctx.arc(enemy.x - 25, enemy.y + 15, 25, 0, Math.PI * 2);
                   ctx.fill();
               } else {
                   ctx.fillStyle = '#bdc3c7';
                   ctx.beginPath();
                   ctx.arc(enemy.x, enemy.y, 25, 0, Math.PI * 2);
                   ctx.arc(enemy.x + 30, enemy.y + 15, 25, 0, Math.PI * 2);
                   ctx.arc(enemy.x - 25, enemy.y + 15, 25, 0, Math.PI * 2);
                   ctx.fill();
               }
           });
       }
      
       function drawItems() {
           items.forEach(item => {
               if (item.type === 'ammo_boost') {
                   ctx.fillStyle = '#f1c40f';
                   const size = 25;
                   ctx.beginPath();
                   ctx.moveTo(item.x, item.y - size);
                   ctx.lineTo(item.x + size/2.5, item.y + size/2.5);
                   ctx.lineTo(item.x - size/2.5, item.y - size/2.5);
                   ctx.lineTo(item.x + size/2.5, item.y - size/2.5);
                   ctx.lineTo(item.x - size/2.5, item.y + size/2.5);
                   ctx.closePath();
                   ctx.fill();
               } else if (item.type === 'water_boost') {
                   ctx.font = '30px Arial';
                   ctx.textAlign = 'center';
                   ctx.fillText('❤️', item.x, item.y);
               }
           });
       }


       function drawCollectEffects() {
           for (let i = collectEffects.length - 1; i >= 0; i--) {
               const effect = collectEffects[i];
               ctx.beginPath();
               ctx.arc(effect.x, effect.y, effect.size, 0, Math.PI * 2);
               ctx.fillStyle = `rgba(52, 152, 219, ${effect.alpha})`;
               ctx.fill();
              
               effect.size += 1;
               effect.alpha -= 0.02;
               if (effect.alpha <= 0) {
                   collectEffects.splice(i, 1);
               }
           }
       }




       function drawDam() {
           ctx.fillStyle = '#7f8c8d';
           ctx.fillRect(0, canvas.height - 30, canvas.width, 30);


           const damHeight = (waterLevel / maxWaterLevel) * (canvas.height - 30);
           ctx.fillStyle = '#2e86c1';
           ctx.fillRect(0, canvas.height - 30 - damHeight, canvas.width, damHeight);
       }


       function drawRainEffect() {
           if (Math.random() < 0.2) {
               rainParticles.push({
                   x: Math.random() * canvas.width,
                   y: -10,
                   length: 10 + Math.random() * 15,
                   speed: 5 + Math.random() * 3
               });
           }


           for (let i = rainParticles.length - 1; i >= 0; i--) {
               const p = rainParticles[i];
               p.y += p.speed;


               if (p.y > canvas.height) {
                   rainParticles.splice(i, 1);
               }
           }


           ctx.strokeStyle = 'rgba(133, 193, 233, 0.7)';
           ctx.lineWidth = 2;
           ctx.beginPath();
           rainParticles.forEach(p => {
               ctx.moveTo(p.x, p.y);
               ctx.lineTo(p.x, p.y + p.length);
           });
           ctx.stroke();
       }


       function shoot() {
           if (playerAmmo > 0) {
               playerAmmo -= waterCostPerShot;
               let bulletSpeed = baseBulletSpeed;
               let bulletSize = 5;
               if (isSuperPoweredUp) {
                   bulletSpeed = superPowerfulBulletSpeed;
                   bulletSize = superPowerfulBulletSize;
               } else if (isPoweredUp) {
                   bulletSpeed = basePowerfulBulletSpeed;
                   bulletSize = powerfulBulletSize;
               }
               bullets.push({
                   x: player.x,
                   y: player.y - 60,
                   speed: bulletSpeed,
                   size: bulletSize
               });
               updateUI();
           }
       }


       function updateGameObjects() {
           if (score >= lastCoinScore + 100) {
               coins++;
               lastCoinScore += 100;
               updateUI();
           }


           if (isPoweredUp && Date.now() - powerUpTimer > powerUpDuration) {
               isPoweredUp = false;
           }
           if (isSuperPoweredUp && Date.now() - superPowerUpTimer > superPowerUpDuration) {
               isSuperPoweredUp = false;
           }


           if (waterLevel < maxWaterLevel) {
               waterLevel += passiveWaterIncreaseRate;
           }


           if (keys['ArrowLeft'] && player.x > 30) {
               player.x -= player.speed;
           }
           if (keys['ArrowRight'] && player.x < canvas.width - 30) {
               player.x += player.speed;
           }


           // 빗방울 생성
           const raindropSpawnRate = 0.05 + (difficultyLevel - 1) * 0.005;
           if (Math.random() < raindropSpawnRate) {
               raindrops.push({
                   x: Math.random() * canvas.width,
                   y: -20,
                   speed: baseRaindropSpeed + (difficultyLevel - 1) * 0.4
               });
           }


           // 적 생성
           const enemySpawnRate = 0.015 + (difficultyLevel - 1) * 0.005;
           if (Math.random() < enemySpawnRate) {
               const isSuper = Math.random() < 0.1;
               let enemyType = 'normal';
               if (difficultySelect.value === 'hard' && Math.random() < 0.2 && enemies.filter(e => e.type === 'thief').length === 0) {
                   enemyType = 'thief';
               }
               enemies.push({
                   x: Math.random() * canvas.width,
                   y: -50,
                   speed: (baseEnemySpeed + (isSuper ? baseSuperEnemySpeedMultiplier : 0) + Math.random() * 0.5) * (1 + (difficultyLevel - 1) * 0.3),
                   isSuper: isSuper,
                   type: enemyType,
                   spawnTime: Date.now()
               });
           }


           // 아이템 생성
           if (Math.random() < 0.005) {
               items.push({ x: Math.random() * canvas.width, y: -20, speed: baseRaindropSpeed + 0.5, type: 'ammo_boost' });
           }
           if (Math.random() < 0.001) {
               items.push({ x: Math.random() * canvas.width, y: -20, speed: baseRaindropSpeed + 0.5, type: 'water_boost' });
           }


           // 빗방울 업데이트
           raindrops = raindrops.filter(drop => {
               drop.y += drop.speed;
               const isColliding = drop.y > player.y - 15 && drop.x > player.x - player.width / 2 - 10 && drop.x < player.x + player.width / 2 + 10;
               if (isColliding && playerAmmo < maxPlayerAmmo) {
                   playerAmmo = Math.min(maxPlayerAmmo, playerAmmo + ammoPerRaindrop);
                   collectEffects.push({ x: player.x, y: player.y, size: 5, alpha: 1 });
                   updateUI();
                   return false;
               }
               return drop.y < canvas.height;
           });


           // 아이템 업데이트
           items = items.filter(item => {
               item.y += item.speed;
               const isColliding = item.y > player.y - 15 && item.x > player.x - player.width / 2 && item.x < player.x + player.width / 2;
               if (isColliding) {
                   if (item.type === 'ammo_boost') {
                       playerAmmo = Math.min(maxPlayerAmmo, playerAmmo + itemAmmoBoost);
                       isPoweredUp = true;
                       powerUpTimer = Date.now();
                   } else if (item.type === 'water_boost') {
                       waterLevel = Math.min(maxWaterLevel, waterLevel + itemWaterBoost);
                   }
                   updateUI();
                   return false;
               }
               return item.y < canvas.height;
           });


           // 총알 업데이트
           bullets = bullets.filter(bullet => {
               bullet.y -= bullet.speed;
               for (let i = 0; i < enemies.length; i++) {
                   const enemy = enemies[i];
                   if (enemy.type === 'thief') continue; // 방해꾼은 총알에 맞지 않음
                   const dist = Math.sqrt(Math.pow(bullet.x - enemy.x, 2) + Math.pow(bullet.y - enemy.y, 2));
                   if (dist < 30) {
                       if (enemy.isSuper) {
                           score += scorePerSuperKill;
                       } else {
                           score += scorePerNormalKill;
                       }
                       enemies.splice(i, 1);
                       updateUI();
                       return false;
                   }
               }
               return bullet.y > 0;
           });


           // 적 업데이트
           enemies = enemies.filter(enemy => {
               enemy.y += enemy.speed;
               if (enemy.type === 'thief') {
                   if (Date.now() - enemy.spawnTime > 10000) {
                       const difficulty = difficultySelect.value;
                       playerAmmo = Math.max(0, playerAmmo - thiefAmmoLoss[difficulty]);
                       updateUI();
                       return false;
                   }
               } else if (enemy.y > canvas.height - 30) {
                   waterLevel -= waterLossPerEnemy;
                   if (waterLevel <= 0) {
                       gameOver();
                   }
                   updateUI();
                   return false;
               }
               return enemy.y < canvas.height;
           });
       }


       function gameLoop(timestamp) {
           if (!gameActive) return;


           const deltaTime = timestamp - lastUpdateTime;
           if (deltaTime < 1000 / 60) {
                requestAnimationFrame(gameLoop);
                return;
           }
           lastUpdateTime = timestamp;


           ctx.clearRect(0, 0, canvas.width, canvas.height);
           drawBackground();
           drawDam();
           updateGameObjects();
           drawRainEffect();
           drawRaindrops();
           drawItems();
           drawBullets();
           drawEnemies();
           drawPlayer();
           drawCollectEffects();


           requestAnimationFrame(gameLoop);
       }
   });
</script>


</body>
</html>



