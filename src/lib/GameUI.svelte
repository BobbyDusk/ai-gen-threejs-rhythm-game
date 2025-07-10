<script>
  export let gameState;
  
  $: scoreString = gameState.score.toString().padStart(8, '0');
  $: comboString = gameState.combo > 0 ? `${gameState.combo}x` : '';
  $: progressPercent = gameState.totalTime > 0 ? (gameState.currentTime / gameState.totalTime) * 100 : 0;
  
  function formatTime(ms) {
    const seconds = Math.floor(ms / 1000);
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = seconds % 60;
    return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`;
  }
</script>

<div class="game-ui">
  <!-- Score Display -->
  <div class="score-panel">
    <div class="score-display">
      {scoreString}
    </div>
    <div class="score-label">SCORE</div>
  </div>
  
  <!-- Combo Display -->
  {#if gameState.combo > 0}
    <div class="combo-panel">
      <div class="combo-display">
        {comboString}
      </div>
      <div class="combo-label">COMBO</div>
    </div>
  {/if}
  
  <!-- Accuracy Display -->
  <div class="accuracy-panel">
    <div class="accuracy-display">
      {gameState.accuracy}%
    </div>
    <div class="accuracy-label">ACCURACY</div>
  </div>
  
  <!-- Time Display -->
  <div class="time-panel">
    <div class="time-display">
      {formatTime(gameState.currentTime)}
    </div>
    <div class="time-label">TIME</div>
  </div>
  
  <!-- Level Display -->
  <div class="level-panel">
    <div class="level-display">
      {gameState.level}
    </div>
    <div class="level-label">LEVEL</div>
  </div>
  
  <!-- Progress Bar -->
  <div class="progress-container">
    <div class="progress-bar">
      <div class="progress-fill" style="width: {progressPercent}%"></div>
    </div>
  </div>
  
  <!-- Beat Visualizer -->
  <div class="beat-visualizer">
    {#each Array(4) as _, i}
      <div class="beat-lane">
        <div class="beat-indicator" class:active={gameState.isPlaying}>
          <div class="beat-pulse"></div>
        </div>
        <div class="lane-key">{['A', 'S', 'D', 'F'][i]}</div>
      </div>
    {/each}
  </div>
</div>

<style>
  .game-ui {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 10;
    font-family: 'Orbitron', monospace;
  }
  
  .score-panel {
    position: absolute;
    top: 20px;
    right: 20px;
    text-align: right;
  }
  
  .score-display {
    font-size: 48px;
    font-weight: 900;
    color: #00ffff;
    text-shadow: 0 0 20px #00ffff;
    letter-spacing: 4px;
  }
  
  .score-label {
    font-size: 12px;
    color: #888;
    margin-top: 5px;
  }
  
  .combo-panel {
    position: absolute;
    top: 120px;
    right: 20px;
    text-align: right;
  }
  
  .combo-display {
    font-size: 36px;
    font-weight: 700;
    color: #ff00ff;
    text-shadow: 0 0 20px #ff00ff;
    animation: combo-pulse 0.5s ease-in-out infinite;
  }
  
  .combo-label {
    font-size: 12px;
    color: #888;
    margin-top: 5px;
  }
  
  .accuracy-panel {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
  }
  
  .accuracy-display {
    font-size: 32px;
    font-weight: 700;
    color: #00ff00;
    text-shadow: 0 0 15px #00ff00;
  }
  
  .accuracy-label {
    font-size: 12px;
    color: #888;
    margin-top: 5px;
  }
  
  .time-panel {
    position: absolute;
    top: 80px;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
  }
  
  .time-display {
    font-size: 24px;
    font-weight: 700;
    color: #ffff00;
    text-shadow: 0 0 15px #ffff00;
  }
  
  .time-label {
    font-size: 12px;
    color: #888;
    margin-top: 5px;
  }
  
  .level-panel {
    position: absolute;
    top: 20px;
    left: 20px;
    text-align: left;
  }
  
  .level-display {
    font-size: 32px;
    font-weight: 700;
    color: #ff6600;
    text-shadow: 0 0 15px #ff6600;
  }
  
  .level-label {
    font-size: 12px;
    color: #888;
    margin-top: 5px;
  }
  
  .progress-container {
    position: absolute;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    width: 300px;
  }
  
  .progress-bar {
    width: 100%;
    height: 4px;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 2px;
    overflow: hidden;
  }
  
  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #00ffff, #ff00ff);
    transition: width 0.1s ease;
    box-shadow: 0 0 10px currentColor;
  }
  
  .beat-visualizer {
    position: absolute;
    bottom: 80px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 20px;
  }
  
  .beat-lane {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
  }
  
  .beat-indicator {
    width: 60px;
    height: 60px;
    border: 3px solid #333;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    background: rgba(0, 0, 0, 0.7);
  }
  
  .beat-indicator.active {
    border-color: #00ffff;
    box-shadow: 0 0 20px #00ffff;
  }
  
  .beat-pulse {
    width: 20px;
    height: 20px;
    background: #00ffff;
    border-radius: 50%;
    opacity: 0.6;
  }
  
  .beat-indicator.active .beat-pulse {
    animation: beat-pulse 0.5s ease-in-out infinite;
  }
  
  .lane-key {
    font-size: 18px;
    font-weight: 700;
    color: #fff;
    text-shadow: 0 0 10px #fff;
  }
  
  @keyframes combo-pulse {
    0%, 100% {
      transform: scale(1);
    }
    50% {
      transform: scale(1.1);
    }
  }
  
  @keyframes beat-pulse {
    0%, 100% {
      transform: scale(1);
      opacity: 0.6;
    }
    50% {
      transform: scale(1.3);
      opacity: 1;
    }
  }
</style>
