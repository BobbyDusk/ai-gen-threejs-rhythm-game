# Copilot Instructions for 4K Rhythm Game

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a 4K rhythm game built with Vite, Svelte, Tailwind CSS, and Three.js. The game features:
- 4-lane rhythm gameplay with ASDF key controls
- 3D graphics powered by Three.js
- Dynamic audio generation and visualization
- Neon cyberpunk aesthetic
- Score tracking with combo system
- Real-time accuracy calculation

## Technical Stack
- **Frontend Framework**: Svelte
- **Build Tool**: Vite
- **Styling**: Tailwind CSS with custom cyberpunk theme
- **3D Graphics**: Three.js
- **Audio**: Web Audio API for procedural sound generation
- **Language**: JavaScript (ES6+)

## Key Components
1. **RhythmGame.svelte**: Main game engine with Three.js scene management
2. **GameUI.svelte**: HUD displaying score, combo, accuracy, and controls
3. **AudioManager.svelte**: Audio synthesis and music generation
4. **app.css**: Global styles with Tailwind utilities and cyberpunk theme

## Development Guidelines
- Use modern JavaScript features (ES6+, async/await)
- Follow Svelte best practices for reactivity
- Maintain the cyberpunk aesthetic with neon colors
- Use Three.js efficiently for 3D rendering
- Keep audio processing lightweight for performance
- Ensure responsive design for different screen sizes

## Color Scheme
- Background: #0a0a0a (dark-bg)
- Secondary: #1a1a1a (dark-secondary)
- Neon Blue: #00ffff
- Neon Pink: #ff00ff
- Neon Green: #00ff00
- Neon Yellow: #ffff00
- Orange: #ff6600

## Game Mechanics
- 4 lanes controlled by A, S, D, F keys
- Notes fall from top to bottom
- Hit timing affects score and accuracy
- Combo system for consecutive hits
- Miss penalty resets combo
- Progressive difficulty scaling

## Deployment
- Configured for GitHub Pages deployment via GitHub Actions
- Automatic deployment on push to main branch
- Optimized production builds with asset optimization
- Manual deployment available via `npm run deploy`
