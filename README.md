<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WASD Wizard</title>
  <style>
    :root {
      --game-w: 800px;
      --game-h: 460px;
      --bg1: #0d0f1a;
      --bg2: #141a2a;
      --grid: rgba(255,255,255,0.06);
      --accent: #7aa2ff;
    }
    html, body {
      height: 100%;
      margin: 0;
      background: linear-gradient(135deg, var(--bg1), var(--bg2));
      color: #e7e7f0;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, 'Helvetica Neue', Arial, Noto Sans, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol';
    }
    .wrap {
      min-height: 100%;
      display: grid;
      place-items: center;
      padding: 24px;
      box-sizing: border-box;
    }
    #game {
      position: relative;
      width: var(--game-w);
      height: var(--game-h);
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 20px 60px rgba(0,0,0,.5), 0 0 0 1px rgba(255,255,255,0.08) inset;
      outline: none;
      background:
        linear-gradient(0deg, transparent 24%, var(--grid) 25%, var(--grid) 26%, transparent 27%, transparent 74%, var(--grid) 75%, var(--grid) 76%, transparent 77%),
        linear-gradient(90deg, transparent 24%, var(--grid) 25%, var(--grid) 26%, transparent 27%, transparent 74%, var(--grid) 75%, var(--grid) 76%, transparent 77%),
        radial-gradient(1200px 400px at 50% 110%, rgba(0,210,255,0.15), transparent 50%),
        radial-gradient(600px 300px at 10% -10%, rgba(170,140,255,0.18), transparent 60%),
        linear-gradient(135deg, #0b0f17, #111827);
      background-size: 50px 50px, 50px 50px, auto, auto, auto;
      user-select: none;
    }
    .hud {
      position: absolute;
      left: 12px; top: 12px;
      font-size: 14px;
      line-height: 1.25;
      background: rgba(0,0,0,0.35);
      padding: 8px 10px;
      border-radius: 10px;
      backdrop-filter: blur(4px);
      border: 1px solid rgba(255,255,255,0.12);
    }
    .hud kbd {
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
      background: rgba(255,255,255,0.08);
      padding: 1px 6px; border-radius: 6px;
      border: 1px solid rgba(255,255,255,0.15);
    }

    /* Wizard sprite container */
    #wizard {
      position: absolute;
      left: 0; top: 0;
      width: 56px; height: 56px;
      transform: translate(372px, 202px); /* start roughly center */
      will-change: transform;
    }
    /* Subtle shadow under wizard */
    #shadow {
      position: absolute; left: 50%; top: 50%;
      width: 42px; height: 14px;
      background: radial-gradient(ellipse at center, rgba(0,0,0,0.45), rgba(0,0,0,0) 70%);
      transform: translate(-50%, 16px);
      pointer-events: none;
      filter: blur(1px);
    }
    /* Flip the sprite when moving left */
    .face-left svg { transform: scaleX(-1); }

    /* Make it keyboard-focusable hint */
    .hint {
      position: absolute; bottom: 12px; right: 12px;
      font-size: 12px; opacity: .75;
      background: rgba(0,0,0,.25);
      padding: 6px 8px; border-radius: 8px;
      border: 1px solid rgba(255,255,255,0.1);
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div id="game" tabindex="0" aria-label="WASD Wizard playfield (focus to move)">
      <div class="hud">Move with <kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd> · Hold <kbd>Shift</kbd> to sprint · <kbd>R</kbd> to recenter</div>
      <div id="wizard" aria-label="Wizard sprite">
        <div id="shadow"></div>
        <!-- Inline SVG wizard sprite (simple, cute) -->
        <svg width="56" height="56" viewBox="0 0 56 56" fill="none" xmlns="http://www.w3.org/2000/svg" style="display:block;filter: drop-shadow(0 4px 6px rgba(0,0,0,.35));">
          <!-- Hat -->
          <path d="M10 26 L28 6 L46 26 Z" fill="#3b5cff" stroke="#cdd8ff" stroke-width="2"/>
          <circle cx="22" cy="18" r="2" fill="#cdd8ff"/>
          <circle cx="34" cy="14" r="2" fill="#cdd8ff"/>
          <!-- Head -->
          <circle cx="28" cy="30" r="10" fill="#f0d6b6" stroke="#c39c75" stroke-width="1.5"/>
          <!-- Beard -->
          <path d="M20 30 C20 40, 36 40, 36 30 C34 32, 22 32, 20 30 Z" fill="#e7edf7" stroke="#b9c4d3" stroke-width="1.5"/>
          <!-- Body / Robe -->
          <path d="M16 40 L28 52 L40 40 L36 36 L20 36 Z" fill="#445eea" stroke="#a8b4ff" stroke-width="1.5"/>
          <!-- Staff -->
          <path d="M44 14 C46 16, 46 21, 44 24 L43 24 L43 44" stroke="#8b5a2b" stroke-width="3" stroke-linecap="round"/>
          <circle cx="44" cy="14" r="4" fill="#ffe081" stroke="#f3d373" stroke-width="2"/>
          <!-- Eyes -->
          <circle cx="24.5" cy="28.5" r="1.8" fill="#1a1a1a"/>
          <circle cx="31.5" cy="28.5" r="1.8" fill="#1a1a1a"/>
        </svg>
      </div>
      <div class="hint">Click the playfield if keys don't work</div>
    </div>
  </div>

  <script>
    (function(){
      const game = document.getElementById('game');
      const wizard = document.getElementById('wizard');

      // Position state
      let pos = { x: 372, y: 202 };
      const size = { w: 56, h: 56 };

      // Movement state
      const keys = new Set();
      let last = performance.now();

      const BASE_SPEED = 200; // px per second
      const SPRINT_MULT = 1.8;

      function clamp(v, min, max){ return Math.max(min, Math.min(max, v)); }
      function setTransform(){ wizard.style.transform = `translate(${pos.x}px, ${pos.y}px)`; }

      function loop(now){
        const dt = Math.min(0.05, (now - last) / 1000); // cap big dt spikes
        last = now;

        let dx = 0, dy = 0;
        if(keys.has('KeyW')) dy -= 1;
        if(keys.has('KeyS')) dy += 1;
        if(keys.has('KeyA')) dx -= 1;
        if(keys.has('KeyD')) dx += 1;

        // Normalize diagonal movement
        if(dx !== 0 || dy !== 0){
          const mag = Math.hypot(dx, dy);
          dx /= mag; dy /= mag;

          const sprinting = keys.has('ShiftLeft') || keys.has('ShiftRight');
          const speed = BASE_SPEED * (sprinting ? SPRINT_MULT : 1);
          pos.x += dx * speed * dt;
          pos.y += dy * speed * dt;

          // Face left or right by flipping the SVG
          if(dx < 0) wizard.classList.add('face-left');
          else if(dx > 0) wizard.classList.remove('face-left');
        }

        // Clamp within bounds
        const bounds = game.getBoundingClientRect();
        const maxX = bounds.width - size.w;
        const maxY = bounds.height - size.h;
        pos.x = clamp(pos.x, 0, maxX);
        pos.y = clamp(pos.y, 0, maxY);

        setTransform();
        requestAnimationFrame(loop);
      }

      // Input handling
      function onDown(e){
        // Prevent page scrolling for WASD/arrow/space while focused
        if(['KeyW','KeyA','KeyS','KeyD','ArrowUp','ArrowLeft','ArrowDown','ArrowRight','Space'].includes(e.code)){
          e.preventDefault();
        }
        keys.add(e.code);
        // Arrow keys also map to movement
        if(e.code === 'ArrowUp') keys.add('KeyW');
        if(e.code === 'ArrowDown') keys.add('KeyS');
        if(e.code === 'ArrowLeft') keys.add('KeyA');
        if(e.code === 'ArrowRight') keys.add('KeyD');

        // Recenter with R
        if(e.code === 'KeyR'){
          pos.x = (game.clientWidth - size.w)/2;
          pos.y = (game.clientHeight - size.h)/2;
          setTransform();
        }
      }
      function onUp(e){
        keys.delete(e.code);
        if(e.code === 'ArrowUp') keys.delete('KeyW');
        if(e.code === 'ArrowDown') keys.delete('KeyS');
        if(e.code === 'ArrowLeft') keys.delete('KeyA');
        if(e.code === 'ArrowRight') keys.delete('KeyD');
      }

      // Focus game on click so keys work
      game.addEventListener('click', () => game.focus());
      game.addEventListener('keydown', onDown);
      game.addEventListener('keyup', onUp);

      // Start centered
      pos.x = (game.clientWidth - size.w)/2;
      pos.y = (game.clientHeight - size.h)/2;
      setTransform();

      // Start the loop
      requestAnimationFrame(loop);

      // Auto-focus shortly after load to enable immediate key input
      setTimeout(() => game.focus(), 100);

      // Resize handling keeps the wizard within bounds
      window.addEventListener('resize', () => {
        const maxX = game.clientWidth - size.w;
        const maxY = game.clientHeight - size.h;
        pos.x = clamp(pos.x, 0, maxX);
        pos.y = clamp(pos.y, 0, maxY);
        setTransform();
      });
    })();
  </script>
</body>
</html>
