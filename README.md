<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Bug Floating in Beer</title>
  <style>
    :root{
      --glass-size: min(86vmin, 760px);
      --beer-color: linear-gradient(180deg,#ffd86b,#f2a82a 60%,#d07b10);
      --bug-size: 48px;
      --avoid-radius: 120px;
      --max-speed: 8;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:#0b0e12;display:flex;align-items:center;justify-content:center;font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial;color:#fff}

    .viewport{width:var(--glass-size);height:var(--glass-size);display:grid;place-items:center}

    .glass{
      width:100%;height:100%;border-radius:50%;background:radial-gradient(circle at 30% 20%, rgba(255,255,255,0.12), transparent 15%), rgba(255,255,255,0.04);box-shadow:0 30px 80px rgba(0,0,0,0.6), inset 0 6px 18px rgba(255,255,255,0.03);display:flex;align-items:center;justify-content:center;position:relative;overflow:hidden;border:6px solid rgba(255,255,255,0.06)
    }

    .beer-surface{
      width:92%;height:92%;border-radius:50%;background:var(--beer-color);position:relative;overflow:hidden;box-shadow:inset 0 18px 48px rgba(0,0,0,0.28);
    }

    .foam-ring{position:absolute;inset:0;border-radius:50%;pointer-events:none;box-shadow:inset 0 18px 36px rgba(255,255,255,0.12);}

    .bug{
      position:absolute;width:var(--bug-size);height:var(--bug-size);left:50%;top:50%;transform:translate(-50%,-50%);will-change:transform;filter:drop-shadow(0 6px 8px rgba(0,0,0,0.45));
    }

    .bug-inner{width:100%;height:100%;display:block;}
    @keyframes bob {0%{transform:translateY(-5px)}50%{transform:translateY(5px)}100%{transform:translateY(-5px)}}
    .bug-inner.bobbing{animation:bob 3.6s ease-in-out infinite}

    .bug svg{width:100%;height:100%;display:block}

    .ripple{
      position:absolute;
      border-radius:50%;
      border:2px solid rgba(255,255,255,0.4);
      transform:translate(-50%,-50%) scale(0);
      animation:ripple 1.5s ease-out forwards;
      pointer-events:none;
    }

    @keyframes ripple{
      to{transform:translate(-50%,-50%) scale(6);opacity:0;}
    }

    .spawn-button{
      position:absolute;
      bottom:10px;
      left:50%;
      transform:translateX(-50%);
      padding:6px 12px;
      border:none;
      border-radius:6px;
      background:#222;
      color:#fff;
      cursor:pointer;
      font-size:14px;
    }
  </style>
</head>
<body>
  <div class="viewport">
    <div class="glass">
      <div class="beer-surface" id="beer">
        <div class="foam-ring"></div>
        <div class="bug" id="bug">
          <div class="bug-inner bobbing">
            <svg viewBox="0 0 100 100" aria-hidden="true">
              <ellipse cx="50" cy="52" rx="14" ry="18" fill="#2b2b2b" />
              <rect x="46" y="40" width="8" height="6" rx="2" fill="#1f1f1f" />
              <circle cx="50" cy="32" r="7" fill="#222" />
              <g stroke="#1a1a1a" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
                <path d="M34 44 q-8 -6 -12 -4"/>
                <path d="M36 58 q-10 8 -12 12"/>
                <path d="M66 44 q8 -6 12 -4"/>
                <path d="M64 58 q10 8 12 12"/>
                <path d="M50 26 q-5 -6 -8 -10"/>
                <path d="M50 26 q5 -6 8 -10"/>
              </g>
            </svg>
          </div>
        </div>
        <button class="spawn-button" id="spawnBtn">New Bug</button>
      </div>
    </div>
  </div>

  <script>
    const beer = document.getElementById('beer');
    const bugEl = document.getElementById('bug');
    const spawnBtn = document.getElementById('spawnBtn');

    function createBug(x, y) {
      const bug = document.createElement('div');
      bug.className = 'bug';
      bug.style.left = x + 'px';
      bug.style.top = y + 'px';
      bug.innerHTML = `
        <div class="bug-inner bobbing">
          <svg viewBox="0 0 100 100" aria-hidden="true">
            <ellipse cx="50" cy="52" rx="14" ry="18" fill="#2b2b2b" />
            <rect x="46" y="40" width="8" height="6" rx="2" fill="#1f1f1f" />
            <circle cx="50" cy="32" r="7" fill="#222" />
            <g stroke="#1a1a1a" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
              <path d="M34 44 q-8 -6 -12 -4"/>
              <path d="M36 58 q-10 8 -12 12"/>
              <path d="M66 44 q8 -6 12 -4"/>
              <path d="M64 58 q10 8 12 12"/>
              <path d="M50 26 q-5 -6 -8 -10"/>
              <path d="M50 26 q5 -6 8 -10"/>
            </g>
          </svg>
        </div>`;
      bug.addEventListener('click', () => makeRipple(bug));
      beer.appendChild(bug);
    }

    function makeRipple(bug) {
      const rect = beer.getBoundingClientRect();
      const bugRect = bug.getBoundingClientRect();
      const ripple = document.createElement('div');
      ripple.className = 'ripple';
      ripple.style.width = ripple.style.height = bugRect.width + 'px';
      ripple.style.left = (bugRect.left - rect.left + bugRect.width/2) + 'px';
      ripple.style.top = (bugRect.top - rect.top + bugRect.height/2) + 'px';
      beer.appendChild(ripple);
      ripple.addEventListener('animationend', () => ripple.remove());
    }

    bugEl.addEventListener('click', () => makeRipple(bugEl));

    spawnBtn.addEventListener('click', () => {
      const rect = beer.getBoundingClientRect();
      const x = Math.random() * rect.width;
      const y = Math.random() * rect.height;
      createBug(x, y);
    });

    document.addEventListener('keydown', (e) => {
      if (e.key === 'e') {
        const rect = beer.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;
        createBug(x, y);
      }
    });
  </script>
</body>
</html>
