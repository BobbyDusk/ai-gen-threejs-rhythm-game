<script>
  import { onMount } from 'svelte';
  
  let audioContext;
  let masterGain;
  let musicSource;
  let isInitialized = false;
  let isPlaying = false;
  
  // Audio effects
  let hitSound;
  let missSound;
  let comboSound;
  
  onMount(() => {
    initializeAudio();
  });
  
  function initializeAudio() {
    try {
      audioContext = new (window.AudioContext || window.webkitAudioContext)();
      masterGain = audioContext.createGain();
      masterGain.connect(audioContext.destination);
      masterGain.gain.value = 0.7;
      
      createSoundEffects();
      generateBackgroundMusic();
      isInitialized = true;
    } catch (error) {
      console.error('Failed to initialize audio:', error);
    }
  }
  
  function createSoundEffects() {
    // Create hit sound effect
    hitSound = createSynthSound(800, 0.1, 'sine');
    
    // Create miss sound effect
    missSound = createSynthSound(200, 0.2, 'square');
    
    // Create combo sound effect
    comboSound = createSynthSound(1200, 0.15, 'triangle');
  }
  
  function createSynthSound(frequency, duration, type = 'sine') {
    return {
      play: () => {
        if (!audioContext) return;
        
        const oscillator = audioContext.createOscillator();
        const gainNode = audioContext.createGain();
        
        oscillator.type = type;
        oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);
        
        gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
        gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + duration);
        
        oscillator.connect(gainNode);
        gainNode.connect(masterGain);
        
        oscillator.start(audioContext.currentTime);
        oscillator.stop(audioContext.currentTime + duration);
      }
    };
  }
  
  function generateBackgroundMusic() {
    // Create a simple electronic beat pattern
    const bpm = 120;
    const beatInterval = 60 / bpm; // seconds per beat
    
    // Bass pattern
    const bassPattern = [1, 0, 1, 0, 1, 0, 1, 0];
    
    // Synth pattern
    const synthPattern = [0, 1, 0, 1, 0, 1, 1, 0];
    
    function playBassNote(time, frequency) {
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      const filter = audioContext.createBiquadFilter();
      
      oscillator.type = 'sawtooth';
      oscillator.frequency.setValueAtTime(frequency, time);
      
      filter.type = 'lowpass';
      filter.frequency.setValueAtTime(400, time);
      
      gainNode.gain.setValueAtTime(0.4, time);
      gainNode.gain.exponentialRampToValueAtTime(0.01, time + 0.3);
      
      oscillator.connect(filter);
      filter.connect(gainNode);
      gainNode.connect(masterGain);
      
      oscillator.start(time);
      oscillator.stop(time + 0.3);
    }
    
    function playSynthNote(time, frequency) {
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      
      oscillator.type = 'triangle';
      oscillator.frequency.setValueAtTime(frequency, time);
      
      gainNode.gain.setValueAtTime(0.2, time);
      gainNode.gain.exponentialRampToValueAtTime(0.01, time + 0.2);
      
      oscillator.connect(gainNode);
      gainNode.connect(masterGain);
      
      oscillator.start(time);
      oscillator.stop(time + 0.2);
    }
    
    // Start the music loop
    function scheduleMusicLoop() {
      if (!isPlaying) return;
      
      const currentTime = audioContext.currentTime;
      const lookAhead = 0.1; // seconds
      
      for (let i = 0; i < 8; i++) {
        const beatTime = currentTime + i * beatInterval;
        
        if (bassPattern[i % bassPattern.length]) {
          playBassNote(beatTime, 55); // A1
        }
        
        if (synthPattern[i % synthPattern.length]) {
          playSynthNote(beatTime, 220); // A3
        }
      }
      
      // Schedule next loop
      setTimeout(scheduleMusicLoop, 100);
    }
    
    return { start: scheduleMusicLoop };
  }
  
  export function startGame() {
    if (!isInitialized) return;
    
    // Resume audio context if suspended
    if (audioContext.state === 'suspended') {
      audioContext.resume();
    }
    
    isPlaying = true;
    generateBackgroundMusic().start();
  }
  
  export function pauseGame() {
    isPlaying = false;
    if (audioContext) {
      audioContext.suspend();
    }
  }
  
  export function resetGame() {
    isPlaying = false;
    if (audioContext) {
      audioContext.suspend();
    }
  }
  
  export function playHitSound() {
    if (hitSound) {
      hitSound.play();
    }
  }
  
  export function playMissSound() {
    if (missSound) {
      missSound.play();
    }
  }
  
  export function playComboSound() {
    if (comboSound) {
      comboSound.play();
    }
  }
  
  export function setVolume(volume) {
    if (masterGain) {
      masterGain.gain.value = Math.max(0, Math.min(1, volume));
    }
  }
</script>

<!-- Audio visualization -->
<div class="audio-visualizer">
  <div class="audio-status">
    <div class="status-indicator" class:active={isPlaying}>
      <div class="pulse"></div>
    </div>
    <span class="status-text">
      {isPlaying ? 'PLAYING' : 'PAUSED'}
    </span>
  </div>
  
  <div class="volume-controls">
    <label for="volume">Volume</label>
    <input 
      type="range" 
      id="volume" 
      min="0" 
      max="1" 
      step="0.1" 
      value="0.7"
      on:input={(e) => setVolume(e.target.value)}
    />
  </div>
</div>

<style>
  .audio-visualizer {
    position: absolute;
    bottom: 20px;
    right: 20px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    font-family: 'Orbitron', monospace;
  }
  
  .audio-status {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .status-indicator {
    width: 20px;
    height: 20px;
    border: 2px solid #333;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.7);
  }
  
  .status-indicator.active {
    border-color: #00ff00;
    box-shadow: 0 0 10px #00ff00;
  }
  
  .pulse {
    width: 8px;
    height: 8px;
    background: #00ff00;
    border-radius: 50%;
    opacity: 0.6;
  }
  
  .status-indicator.active .pulse {
    animation: pulse 1s ease-in-out infinite;
  }
  
  .status-text {
    font-size: 12px;
    color: #888;
    font-weight: 700;
  }
  
  .volume-controls {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .volume-controls label {
    font-size: 12px;
    color: #888;
    font-weight: 700;
  }
  
  .volume-controls input[type="range"] {
    width: 100px;
    height: 4px;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 2px;
    outline: none;
    -webkit-appearance: none;
  }
  
  .volume-controls input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 12px;
    height: 12px;
    background: #00ffff;
    border-radius: 50%;
    cursor: pointer;
    box-shadow: 0 0 5px #00ffff;
  }
  
  .volume-controls input[type="range"]::-moz-range-thumb {
    width: 12px;
    height: 12px;
    background: #00ffff;
    border-radius: 50%;
    cursor: pointer;
    border: none;
    box-shadow: 0 0 5px #00ffff;
  }
  
  @keyframes pulse {
    0%, 100% {
      transform: scale(1);
      opacity: 0.6;
    }
    50% {
      transform: scale(1.5);
      opacity: 1;
    }
  }
</style>
