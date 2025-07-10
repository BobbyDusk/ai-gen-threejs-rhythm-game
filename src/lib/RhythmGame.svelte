<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import GameUI from './GameUI.svelte';
  import AudioManager from './AudioManager.svelte';
  
  let gameContainer;
  let scene, camera, renderer, gameWorld;
  let gameState = {
    score: 0,
    combo: 0,
    accuracy: 100,
    isPlaying: false,
    currentTime: 0,
    totalTime: 0,
    level: 1
  };
  
  let notes = [];
  let trackPositions = [-1.5, -0.5, 0.5, 1.5]; // 4 tracks
  let hitZoneY = -3;
  let gameSpeed = 2;
  let audioManager;
  
  // Game timing
  let startTime = 0;
  let lastFrameTime = 0;
  // Enhanced beat pattern with more variety
  let beatPattern = [
    // Opening sequence
    { time: 1000, track: 0, type: 'normal' },
    { time: 1500, track: 1, type: 'normal' },
    { time: 2000, track: 2, type: 'normal' },
    { time: 2500, track: 3, type: 'normal' },
    
    // Build up
    { time: 3000, track: 1, type: 'normal' },
    { time: 3250, track: 2, type: 'normal' },
    { time: 3500, track: 0, type: 'normal' },
    { time: 3750, track: 3, type: 'normal' },
    
    // Rapid sequence
    { time: 4000, track: 0, type: 'normal' },
    { time: 4125, track: 1, type: 'normal' },
    { time: 4250, track: 2, type: 'normal' },
    { time: 4375, track: 3, type: 'normal' },
    
    // Pattern variation
    { time: 5000, track: 0, type: 'normal' },
    { time: 5000, track: 2, type: 'normal' }, // Simultaneous notes
    { time: 5500, track: 1, type: 'normal' },
    { time: 5500, track: 3, type: 'normal' },
    
    // Complex rhythm
    { time: 6000, track: 0, type: 'normal' },
    { time: 6250, track: 1, type: 'normal' },
    { time: 6375, track: 2, type: 'normal' },
    { time: 6500, track: 3, type: 'normal' },
    { time: 6750, track: 1, type: 'normal' },
    
    // Finale
    { time: 7000, track: 0, type: 'normal' },
    { time: 7100, track: 1, type: 'normal' },
    { time: 7200, track: 2, type: 'normal' },
    { time: 7300, track: 3, type: 'normal' },
    { time: 7400, track: 0, type: 'normal' },
    { time: 7500, track: 1, type: 'normal' },
    { time: 7600, track: 2, type: 'normal' },
    { time: 7700, track: 3, type: 'normal' },
    
    // Cool down
    { time: 8000, track: 0, type: 'normal' },
    { time: 8500, track: 2, type: 'normal' },
    { time: 9000, track: 1, type: 'normal' },
    { time: 9500, track: 3, type: 'normal' },
    { time: 10000, track: 0, type: 'normal' },
  ];
  
  let nextNoteIndex = 0;
  
  class Note {
    constructor(track, type = 'normal') {
      this.track = track;
      this.type = type;
      this.hit = false;
      this.missed = false;
      
      // Create 3D note object
      const geometry = new THREE.BoxGeometry(0.3, 0.2, 0.1);
      const material = new THREE.MeshBasicMaterial({ 
        color: type === 'normal' ? 0x00ffff : 0xff00ff,
        transparent: true,
        opacity: 0.8
      });
      this.mesh = new THREE.Mesh(geometry, material);
      this.mesh.position.set(trackPositions[track], 4, 0);
      
      // Add glow effect
      const glowGeometry = new THREE.BoxGeometry(0.4, 0.3, 0.2);
      const glowMaterial = new THREE.MeshBasicMaterial({
        color: type === 'normal' ? 0x00ffff : 0xff00ff,
        transparent: true,
        opacity: 0.3
      });
      this.glow = new THREE.Mesh(glowGeometry, glowMaterial);
      this.glow.position.copy(this.mesh.position);
      
      scene.add(this.mesh);
      scene.add(this.glow);
    }
    
    update(deltaTime) {
      if (!this.hit && !this.missed) {
        this.mesh.position.y -= gameSpeed * deltaTime;
        this.glow.position.y = this.mesh.position.y;
        
        // Check if note is missed
        if (this.mesh.position.y < hitZoneY - 0.5) {
          this.missed = true;
          this.cleanup();
          return false;
        }
      }
      return true;
    }
    
    checkHit(track) {
      if (this.track === track && !this.hit && !this.missed) {
        const distance = Math.abs(this.mesh.position.y - hitZoneY);
        if (distance < 0.3) {
          this.hit = true;
          this.cleanup();
          
          // Calculate score based on accuracy
          let hitAccuracy = Math.max(0, 100 - (distance * 100));
          let points = Math.floor(hitAccuracy * 10);
          
          if (hitAccuracy > 90) {
            gameState.combo += 1;
            points *= (1 + gameState.combo * 0.1);
          } else {
            gameState.combo = 0;
          }
          
          gameState.score += points;
          return { hit: true, accuracy: hitAccuracy, points };
        }
      }
      return { hit: false };
    }
    
    cleanup() {
      scene.remove(this.mesh);
      scene.remove(this.glow);
    }
  }
  
  onMount(() => {
    initThreeJS();
    createGameWorld();
    setupControls();
    startGameLoop();
    
    return () => {
      if (renderer) {
        renderer.dispose();
      }
    };
  });
  
  function initThreeJS() {
    // Scene setup
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0a0a0a);
    
    // Camera setup
    camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 0, 5);
    
    // Renderer setup
    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    
    gameContainer.appendChild(renderer.domElement);
    
    // Handle window resize
    window.addEventListener('resize', onWindowResize);
  }
  
  function createGameWorld() {
    // Add ambient light
    const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
    scene.add(ambientLight);
    
    // Add directional light
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
    directionalLight.position.set(0, 10, 5);
    directionalLight.castShadow = true;
    scene.add(directionalLight);
    
    // Create track lanes
    for (let i = 0; i < 4; i++) {
      const laneGeometry = new THREE.PlaneGeometry(0.5, 10);
      const laneMaterial = new THREE.MeshBasicMaterial({ 
        color: 0x333333, 
        transparent: true, 
        opacity: 0.3 
      });
      const lane = new THREE.Mesh(laneGeometry, laneMaterial);
      lane.position.set(trackPositions[i], 0, -0.1);
      scene.add(lane);
    }
    
    // Create hit zone
    const hitZoneGeometry = new THREE.PlaneGeometry(3, 0.3);
    const hitZoneMaterial = new THREE.MeshBasicMaterial({ 
      color: 0x00ff00, 
      transparent: true, 
      opacity: 0.5 
    });
    const hitZone = new THREE.Mesh(hitZoneGeometry, hitZoneMaterial);
    hitZone.position.set(0, hitZoneY, 0);
    scene.add(hitZone);
    
    // Create particle system for effects
    createParticleSystem();
  }
  
  function createParticleSystem() {
    const particleCount = 100;
    const particles = new THREE.BufferGeometry();
    const positions = new Float32Array(particleCount * 3);
    
    for (let i = 0; i < particleCount * 3; i++) {
      positions[i] = (Math.random() - 0.5) * 10;
    }
    
    particles.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    
    const particleMaterial = new THREE.PointsMaterial({
      color: 0x00ffff,
      size: 0.02,
      transparent: true,
      opacity: 0.6
    });
    
    const particleSystem = new THREE.Points(particles, particleMaterial);
    scene.add(particleSystem);
  }
  
  function setupControls() {
    // Keyboard controls
    document.addEventListener('keydown', (event) => {
      if (!gameState.isPlaying) return;
      
      let track = -1;
      switch(event.key) {
        case 'a':
        case 'A':
          track = 0;
          break;
        case 's':
        case 'S':
          track = 1;
          break;
        case 'd':
        case 'D':
          track = 2;
          break;
        case 'f':
        case 'F':
          track = 3;
          break;
      }
      
      if (track !== -1) {
        handleNoteHit(track);
      }
    });
  }
  
  function handleNoteHit(track) {
    // Find the closest note in the track
    let closestNote = null;
    let closestDistance = Infinity;
    
    for (let note of notes) {
      if (note.track === track && !note.hit && !note.missed) {
        const distance = Math.abs(note.mesh.position.y - hitZoneY);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestNote = note;
        }
      }
    }
    
    if (closestNote) {
      const result = closestNote.checkHit(track);
      if (result.hit) {
        // Create hit effect
        createHitEffect(trackPositions[track], hitZoneY);
      }
    }
  }
  
  function createHitEffect(x, y) {
    const effectGeometry = new THREE.RingGeometry(0.1, 0.3, 8);
    const effectMaterial = new THREE.MeshBasicMaterial({ 
      color: 0x00ff00,
      transparent: true,
      opacity: 1
    });
    const effect = new THREE.Mesh(effectGeometry, effectMaterial);
    effect.position.set(x, y, 0.1);
    scene.add(effect);
    
    // Animate effect
    const startTime = performance.now();
    const animate = () => {
      const elapsed = performance.now() - startTime;
      const progress = elapsed / 500; // 500ms duration
      
      if (progress < 1) {
        effect.scale.setScalar(1 + progress * 2);
        effect.material.opacity = 1 - progress;
        requestAnimationFrame(animate);
      } else {
        scene.remove(effect);
      }
    };
    animate();
  }
  
  function startGameLoop() {
    startTime = performance.now();
    lastFrameTime = startTime;
    animate();
  }
  
  function animate() {
    requestAnimationFrame(animate);
    
    const currentTime = performance.now();
    const deltaTime = (currentTime - lastFrameTime) / 1000;
    lastFrameTime = currentTime;
    
    if (gameState.isPlaying) {
      gameState.currentTime = currentTime - startTime;
      
      // Spawn new notes
      spawnNotes();
      
      // Update notes
      notes = notes.filter(note => note.update(deltaTime));
      
      // Update game state
      updateGameState();
    }
    
    renderer.render(scene, camera);
  }
  
  function spawnNotes() {
    while (nextNoteIndex < beatPattern.length && 
           beatPattern[nextNoteIndex].time <= gameState.currentTime) {
      const noteData = beatPattern[nextNoteIndex];
      const note = new Note(noteData.track, noteData.type);
      notes.push(note);
      nextNoteIndex++;
    }
  }
  
  function updateGameState() {
    // Update accuracy
    const totalNotes = nextNoteIndex;
    const hitNotes = notes.filter(n => n.hit).length;
    if (totalNotes > 0) {
      gameState.accuracy = Math.round((hitNotes / totalNotes) * 100);
    }
  }
  
  function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }
  
  function startGame() {
    gameState.isPlaying = true;
    gameState.score = 0;
    gameState.combo = 0;
    gameState.accuracy = 100;
    notes = [];
    nextNoteIndex = 0;
    startTime = performance.now();
    
    // Start audio if available
    if (audioManager) {
      audioManager.startGame();
    }
  }
  
  function pauseGame() {
    gameState.isPlaying = false;
    if (audioManager) {
      audioManager.pauseGame();
    }
  }
  
  function resetGame() {
    gameState.isPlaying = false;
    gameState.score = 0;
    gameState.combo = 0;
    gameState.accuracy = 100;
    gameState.currentTime = 0;
    nextNoteIndex = 0;
    
    // Clear all notes
    notes.forEach(note => note.cleanup());
    notes = [];
    
    if (audioManager) {
      audioManager.resetGame();
    }
  }
</script>

<div class="game-container" bind:this={gameContainer}>
  <GameUI bind:gameState />
  <AudioManager bind:this={audioManager} />
</div>

<div class="game-controls">
  <button class="control-btn" on:click={startGame} disabled={gameState.isPlaying}>
    Start Game
  </button>
  <button class="control-btn" on:click={pauseGame} disabled={!gameState.isPlaying}>
    Pause
  </button>
  <button class="control-btn" on:click={resetGame}>
    Reset
  </button>
</div>

<!-- AI Disclaimer -->
<div class="ai-disclaimer">
  <p>This game was generated by AI model Claude Sonnet 4, with the following prompt: "Create a 4k rhythm game using vite, tailwind, svelte and three.js."</p>
</div>

<!-- Instructions -->
<div class="instructions">
  <h3>How to Play</h3>
  <p>Press <kbd>A</kbd>, <kbd>S</kbd>, <kbd>D</kbd>, <kbd>F</kbd> keys when notes reach the green hit zone!</p>
  <p>Hit notes precisely for higher scores and build up combos!</p>
</div>

<style>
  .game-container {
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }
  
  .game-controls {
    position: absolute;
    top: 20px;
    left: 20px;
    z-index: 100;
    display: flex;
    gap: 10px;
  }
  
  .control-btn {
    padding: 10px 20px;
    background: rgba(0, 255, 255, 0.2);
    border: 2px solid #00ffff;
    color: #00ffff;
    cursor: pointer;
    border-radius: 5px;
    font-family: 'Orbitron', monospace;
    font-weight: bold;
    transition: all 0.3s ease;
  }
  
  .control-btn:hover {
    background: rgba(0, 255, 255, 0.4);
    box-shadow: 0 0 20px #00ffff;
  }
  
  .control-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  
  .ai-disclaimer {
    position: absolute;
    bottom: 20px;
    right: 20px;
    max-width: 300px;
    color: #888;
    font-family: 'Orbitron', monospace;
    font-size: 10px;
    z-index: 100;
    background: rgba(0, 0, 0, 0.7);
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #333;
  }
  
  .ai-disclaimer p {
    margin: 0;
    line-height: 1.4;
  }
  
  .instructions {
    position: absolute;
    bottom: 20px;
    left: 20px;
    color: #fff;
    font-family: 'Orbitron', monospace;
    z-index: 100;
  }
  
  .instructions h3 {
    margin: 0 0 10px 0;
    color: #00ffff;
  }
  
  .instructions p {
    margin: 5px 0;
    font-size: 14px;
  }
  
  .instructions kbd {
    background: #333;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: monospace;
    font-weight: bold;
    color: #00ffff;
  }
</style>
