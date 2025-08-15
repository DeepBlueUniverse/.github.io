# .github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>My Cart — Demo</title>
  <style>
    :root {
      --bg: #0f1221;
      --card: #171a2e;
      --muted: #a7b0c0;
      --text: #e9edf5;
      --accent: #7c5cff;
      --accent-2: #00d3a7;
      --danger: #ff5c7c;
      --ring: rgba(124,92,255,.35);
    }
    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
      background: radial-gradient(1200px 600px at 70% -10%, #1b1f3b 0%, var(--bg) 60%);
      color: var(--text);
    }
    .container {
      max-width: 980px;
      margin: 40px auto;
      padding: 0 20px;
    }
    .header {
      display: flex; align-items: center; justify-content: space-between;
      gap: 16px; margin-bottom: 20px;
    }
    h1 { font-size: clamp(24px, 2.8vw, 36px); margin: 0; letter-spacing: .5px; }
    .cart {
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0));
      border: 1px solid rgba(255,255,255,0.08);
      border-radius: 16px; padding: 16px;
      box-shadow: 0 10px 30px rgba(0,0,0,.35), inset 0 1px 0 rgba(255,255,255,.03);
    }
    .cart-item {
      display: grid; grid-template-columns: 96px 1fr auto; gap: 16px; align-items: center;
      padding: 14px; border-radius: 12px;
      border: 1px solid rgba(255,255,255,0.06);
      background: rgba(255,255,255,0.02);
    }
    .cart-item + .cart-item { margin-top: 12px; }
    .thumb {
      width: 96px; height: 96px; border-radius: 10px; background: linear-gradient(135deg, #2a2f55, #202545);
      display: grid; place-items: center; position: relative; overflow: hidden;
    }
    .thumb svg { width: 56px; height: 56px; opacity: .9; }
    .info h3 { margin: 0 0 4px; font-size: 18px; }
    .info p { margin: 0; color: var(--muted); font-size: 14px; }
    .price { font-weight: 700; margin-left: 8px; }
    .actions { display: flex; gap: 8px; flex-wrap: wrap; }

    .btn { cursor: pointer; border: 1px solid rgba(255,255,255,.12); color: var(--text);
      background: rgba(255,255,255,.03); padding: 10px 14px; border-radius: 10px; font-weight: 600;
      letter-spacing: .2px; transition: transform .08s ease, box-shadow .2s ease, border-color .2s ease;
    }
    .btn:hover { transform: translateY(-1px); border-color: var(--ring); box-shadow: 0 8px 24px rgba(124,92,255,.12); }
    .btn:focus { outline: 3px solid var(--ring); outline-offset: 2px; }
    .btn.primary { background: linear-gradient(180deg, var(--accent), #5a3cff); border-color: transparent; }
    .btn.ghost { background: linear-gradient(180deg, rgba(255,255,255,.05), rgba(255,255,255,.02)); }
    .btn.danger { background: linear-gradient(180deg, #ff7a94, var(--danger)); border-color: transparent; }

    .summary { display: flex; align-items: center; justify-content: space-between; margin-top: 16px; padding-top: 16px; border-top: 1px dashed rgba(255,255,255,.14); }
    .chip { padding: 6px 10px; border-radius: 999px; background: rgba(0,211,167,.15); color: #9ef0dd; font-weight: 700; border: 1px solid rgba(0,211,167,.35); }

    /* Modal */
    .modal-backdrop { position: fixed; inset: 0; background: rgba(5,8,22,.7); backdrop-filter: blur(6px); display: none; align-items: center; justify-content: center; z-index: 50; }
    .modal-backdrop[aria-hidden="false"] { display: flex; }
    .modal { position: relative; width: min(560px, 92vw); border-radius: 16px; padding: 0; overflow: hidden; border: 1px solid rgba(255,255,255,.12); background: radial-gradient(1000px 600px at -20% -10%, rgba(124,92,255,.4), transparent 50%), linear-gradient(180deg, #151936, #0f1330); box-shadow: 0 20px 60px rgba(0,0,0,.45); }
    .modal-header { padding: 18px 18px 0 18px; position: relative; }
    .modal-body { padding: 24px 18px 26px 18px; text-align: center; }
    .modal h2 { margin: 0 0 8px; font-size: clamp(20px, 2.4vw, 28px); }
    .modal p { margin: 0; color: var(--muted); }
    .modal .ok { margin-top: 18px; }

    /* Confetti canvas sits on top of modal */
    #confettiCanvas { position: absolute; inset: 0; width: 100%; height: 100%; pointer-events: none; }

    /* Tiny badge for free shipping */
    .free { color: #9ef0dd; font-weight: 800; letter-spacing: .4px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>🛒 Your Cart</h1>
      <div class="chip" aria-live="polite">Free shipping active</div>
    </div>

    <div class="cart" role="region" aria-label="Cart items">
      <!-- Item 1 -->
      <div class="cart-item">
        <div class="thumb" aria-hidden="true"> 
          <!-- Decorative SVG icon -->
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.4" aria-hidden="true">
            <path d="M12 2l3 6 6 .9-4.5 4.3 1 6-5.5-3-5.5 3 1-6L3 8.9 9 8l3-6z" />
          </svg>
        </div>
        <div class="info">
          <h3>Stellar Hoodie</h3>
          <p>Color: Night Sky · Size: M</p>
        </div>
        <div class="actions">
          <button class="btn ghost" data-view>View order</button>
          <button class="btn danger" data-cancel>Cancel shipping</button>
        </div>
      </div>

      <!-- Item 2 -->
      <div class="cart-item">
        <div class="thumb" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.4" aria-hidden="true">
            <circle cx="12" cy="12" r="9"/>
            <path d="M12 7v5l3 3"/>
          </svg>
        </div>
        <div class="info">
          <h3>Neon Wristwatch</h3>
          <p>Edition: Aurora</p>
        </div>
        <div class="actions">
          <button class="btn ghost" data-view>View order</button>
          <button class="btn danger" data-cancel>Cancel shipping</button>
        </div>
      </div>

      <!-- Item 3 -->
      <div class="cart-item">
        <div class="thumb" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.4" aria-hidden="true">
            <rect x="4" y="6" width="16" height="12" rx="2"/>
            <path d="M4 10h16"/>
          </svg>
        </div>
        <div class="info">
          <h3>Holographic Laptop Sleeve</h3>
          <p>13–14" · Water‑resistant</p>
        </div>
        <div class="actions">
          <button class="btn ghost" data-view>View order</button>
          <button class="btn danger" data-cancel>Cancel shipping</button>
        </div>
      </div>

      <div class="summary">
        <div>
          Subtotal <span class="price">$238.00</span>
        </div>
        <div class="free" aria-live="polite">Shipping: FREE</div>
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modal-backdrop" id="modal" aria-hidden="true" role="dialog" aria-modal="true" aria-labelledby="modalTitle" aria-describedby="modalDesc">
    <div class="modal">
      <canvas id="confettiCanvas"></canvas>
      <div class="modal-header"></div>
      <div class="modal-body">
        <h2 id="modalTitle">🎉 Congratulations!</h2>
        <p id="modalDesc">Your shipping is still free!</p>
        <button class="btn primary ok" id="okBtn">OK</button>
      </div>
    </div>
  </div>

  <script>
    // ------- Modal logic with focus handling -------
    const modal = document.getElementById('modal');
    const okBtn = document.getElementById('okBtn');
    let lastFocused = null;

    function openModal() {
      lastFocused = document.activeElement;
      modal.setAttribute('aria-hidden', 'false');
      document.body.style.overflow = 'hidden';
      startConfetti();
      // Focus the OK button for accessibility
      setTimeout(() => okBtn.focus(), 0);
    }

    function closeModal() {
      modal.setAttribute('aria-hidden', 'true');
      document.body.style.overflow = '';
      stopConfetti();
      if (lastFocused && typeof lastFocused.focus === 'function') lastFocused.focus();
    }

    // Close on ESC
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape' && modal.getAttribute('aria-hidden') === 'false') {
        closeModal();
      }
    });
    // Close on backdrop click
    modal.addEventListener('click', (e) => {
      if (e.target === modal) closeModal();
    });
    okBtn.addEventListener('click', closeModal);

    // Bind buttons
    document.querySelectorAll('[data-cancel]').forEach(btn => {
      btn.addEventListener('click', openModal);
    });
    document.querySelectorAll('[data-view]').forEach(btn => {
      btn.addEventListener('click', () => {
        // Lightweight demo action
        alert('Order details are not implemented in this demo.');
      });
    });

    // ------- Minimal, pretty confetti -------
    const canvas = document.getElementById('confettiCanvas');
    const ctx = canvas.getContext('2d');
    let confetti = [];
    let rafId = null;
    let running = false;

    const COLORS = [
      '#ffffff','#e9edf5','#7c5cff','#00d3a7','#ff7a94','#ffd166','#8ecae6','#90be6d'
    ];

    function resizeCanvas() {
      const rect = canvas.parentElement.getBoundingClientRect();
      canvas.width = Math.floor(rect.width * devicePixelRatio);
      canvas.height = Math.floor(rect.height * devicePixelRatio);
      ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
    }
    const onResize = () => { if (running) resizeCanvas(); };

    function spawnBurst(n = 200) {
      confetti = [];
      const { width, height } = canvas;
      for (let i = 0; i < n; i++) {
        const angle = Math.random() * Math.PI - Math.PI / 2; // left to right
        const speed = 4 + Math.random() * 6;
        confetti.push({
          x: (canvas.width / devicePixelRatio) / 2,
          y: (canvas.height / devicePixelRatio) / 3,
          vx: Math.cos(angle) * speed,
          vy: Math.sin(angle) * speed - 2,
          g: 0.12 + Math.random() * 0.18,
          w: 6 + Math.random() * 8,
          h: 10 + Math.random() * 16,
          r: Math.random() * Math.PI,
          vr: (-0.25 + Math.random() * 0.5),
          color: COLORS[(Math.random() * COLORS.length) | 0],
          shape: Math.random() < 0.6 ? 'rect' : 'circle',
          life: 180 + Math.random() * 60
        });
      }
    }

    function drawConfetti() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (const p of confetti) {
        p.vy += p.g;
        p.x += p.vx;
        p.y += p.vy;
        p.r += p.vr;
        p.life -= 1;
        if (p.shape === 'rect') {
          ctx.save();
          ctx.translate(p.x, p.y);
          ctx.rotate(p.r);
          ctx.fillStyle = p.color;
          ctx.fillRect(-p.w/2, -p.h/2, p.w, p.h);
          ctx.restore();
        } else {
          ctx.beginPath();
          ctx.arc(p.x, p.y, p.w * 0.45, 0, Math.PI * 2);
          ctx.fillStyle = p.color;
          ctx.fill();
        }
      }
      // fade out old particles
      confetti = confetti.filter(p => p.life > 0 && p.y < (canvas.height / devicePixelRatio) + 40);
      if (confetti.length === 0) {
        // soft loop stop when particles are gone
        stopConfetti();
        return;
      }
      rafId = requestAnimationFrame(drawConfetti);
    }

    function startConfetti() {
      if (running) {
        cancelAnimationFrame(rafId);
      }
      running = true;
      resizeCanvas();
      spawnBurst(220);
      drawConfetti();
      window.addEventListener('resize', onResize);
    }

    function stopConfetti() {
      running = false;
      cancelAnimationFrame(rafId);
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      window.removeEventListener('resize', onResize);
    }
  </script>
</body>
</html>
