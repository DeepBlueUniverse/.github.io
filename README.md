<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Wizard — Lightning & Demon Form</title>
  <style>
    html,body{height:100%;margin:0;background:linear-gradient(#0b1220,#071022);color:#eee;font-family:system-ui,-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,'Helvetica Neue',Arial}
    #gameWrap{display:flex;flex-direction:column;height:100vh}
    header{padding:10px 14px;background:rgba(255,255,255,0.03);display:flex;align-items:center;gap:12px}
    header h1{font-size:16px;margin:0}
    #canvas{background:linear-gradient(#0a1324,#07111c);flex:1;display:block;}
    #hud{padding:12px;background:rgba(0,0,0,0.25);display:flex;gap:18px;align-items:center}
    .stat{font-size:14px}
    .hint{opacity:.7;font-size:13px}
    button{background:#1b2430;border:1px solid rgba(255,255,255,0.03);color:#fff;padding:8px 10px;border-radius:8px;cursor:pointer}
    @media (max-width:600px){header h1{font-size:14px}.stat{font-size:12px}}
  </style>
</head>
<body>
  <div id="gameWrap">
    <header>
      <h1>Wizard — Lightning & Demon Form</h1>
      <div class="hint">Controls: WASD to move · Aim with mouse · E to shoot lightning · X to shield (5s) · R to demon form</div>
    </header>

    <canvas id="canvas" width="1200" height="700"></canvas>

    <div id="hud">
      <div class="stat" id="mode">Mode: Normal</div>
      <div class="stat" id="shield">Shield: Ready</div>
      <div class="stat" id="demon">Demon: Ready</div>
      <div style="flex:1"></div>
      <div class="stat">Tips: Lightning bolts pierce. Demon form increases size and emits a ring of flames on activation.</div>
    </div>
  </div>

<script>
(() => {
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  const W = canvas.width = Math.min(1400, window.innerWidth - 40);
  const H = canvas.height = Math.min(800, window.innerHeight - 160);

  // HUD elements
  const modeEl = document.getElementById('mode');
  const shieldEl = document.getElementById('shield');
  const demonEl = document.getElementById('demon');

  // Player
  const player = {
    x: W/2,
    y: H/2,
    r: 18,
    speed: 220, // px/sec
    vx:0, vy:0,
    isDemon: false
  };

  // Input state
  const keys = {};
  let mouse = {x: W/2, y: H/2};

  // Timers
  let lastTime = performance.now();
  const projectiles = [];
  const flames = [];

  // Shield
  let shield = {active:false,duration:5,remaining:0,cooldown:0};

  // Demon form
  let demon = {active:false,duration:10,remaining:0,cooldown:0};

  // Small utility
  function clamp(v,a,b){return Math.max(a,Math.min(b,v))}
  function dist(a,b){const dx=a.x-b.x,dy=a.y-b.y;return Math.hypot(dx,dy)}

  // Projectiles as lightning bolts
  function spawnLightning(x,y,tx,ty){
    const ang = Math.atan2(ty-y,tx-x);
    const speed = 750;
    projectiles.push({x,y,dx:Math.cos(ang)*speed,dy:Math.sin(ang)*speed,age:0,life:0.9});
  }

  // Flames for the demon ring
  function spawnFlameRing(cx,cy){
    const count = 28;
    for(let i=0;i<count;i++){
      const a = (i/count)*Math.PI*2 + (Math.random()-0.5)*0.2;
      const s = 160 + Math.random()*60;
      flames.push({x:cx,y:cy,ang:a,dist:10,spd:120 + Math.random()*60,life:1,ttl:1.2,rad:6+Math.random()*6});
    }
  }

  // Input listeners
  window.addEventListener('keydown', e=>{
    keys[e.key.toLowerCase()] = true;
    // handle single-press actions
    if(e.key.toLowerCase() === 'e'){
      spawnLightning(player.x, player.y, mouse.x, mouse.y);
    }
    if(e.key.toLowerCase() === 'x'){
      if(!shield.active && shield.cooldown<=0){
        shield.active = true;
        shield.remaining = shield.duration;
        shield.cooldown = 8; // seconds until shield can be reused
      }
    }
    if(e.key.toLowerCase() === 'r'){
      if(!demon.active && demon.cooldown<=0){
        demon.active = true;
        demon.remaining = demon.duration;
        demon.cooldown = 20; // seconds cooldown
        // spawn ring of flames
        spawnFlameRing(player.x, player.y);
      }
    }
  });
  window.addEventListener('keyup', e=>{keys[e.key.toLowerCase()] = false});
  canvas.addEventListener('mousemove', e=>{
    const rect = canvas.getBoundingClientRect();
    mouse.x = (e.clientX - rect.left) * (canvas.width/rect.width);
    mouse.y = (e.clientY - rect.top) * (canvas.height/rect.height);
  });
  canvas.addEventListener('mousedown', e=>{
    // optional: shoot on click
    spawnLightning(player.x, player.y, mouse.x, mouse.y);
  });

  function update(dt){
    // movement
    let mx=0,my=0;
    if(keys['w']||keys['arrowup']) my -= 1;
    if(keys['s']||keys['arrowdown']) my += 1;
    if(keys['a']||keys['arrowleft']) mx -= 1;
    if(keys['d']||keys['arrowright']) mx += 1;
    const len = Math.hypot(mx,my);
    const speed = player.speed * (demon.active?1.05:1);
    if(len>0){ mx/=len; my/=len; player.x += mx*speed*dt; player.y += my*speed*dt; }

    // clamp inside canvas
    player.x = clamp(player.x, 24, W-24);
    player.y = clamp(player.y, 24, H-24);

    // update projectiles
    for(let i=projectiles.length-1;i>=0;i--){
      const p = projectiles[i];
      p.age += dt;
      p.x += p.dx*dt; p.y += p.dy*dt;
      // lightning pierces but fades after life
      if(p.age > p.life) projectiles.splice(i,1);
    }

    // update flames
    for(let i=flames.length-1;i>=0;i--){
      const f = flames[i];
      f.life -= dt/f.ttl * f.ttl; // normalize
      f.dist += f.spd*dt;
      if(f.life<=0) flames.splice(i,1);
    }

    // shield timing
    if(shield.active){
      shield.remaining -= dt;
      if(shield.remaining <= 0){ shield.active = false; }
    }
    if(shield.cooldown>0) shield.cooldown -= dt;

    // demon timing
    if(demon.active){
      demon.remaining -= dt;
      if(demon.remaining <= 0){ demon.active = false; }
    }
    if(demon.cooldown>0) demon.cooldown -= dt;

    // HUD
    modeEl.textContent = 'Mode: ' + (demon.active? 'Demon' : 'Normal');
    shieldEl.textContent = shield.active ? `Shield: ${shield.remaining.toFixed(1)}s` : (shield.cooldown>0 ? `Shield CD: ${shield.cooldown.toFixed(1)}s` : 'Shield: Ready');
    demonEl.textContent = demon.active ? `Demon: ${demon.remaining.toFixed(1)}s` : (demon.cooldown>0 ? `Demon CD: ${Math.ceil(demon.cooldown)}s` : 'Demon: Ready');
  }

  function drawWizard(x,y){
    // draw basic wizard or demon depending on state
    ctx.save();
    ctx.translate(x,y);

    if(demon.active){
      // larger body
      const R = player.r*1.9;
      // body
      ctx.beginPath(); ctx.arc(0,6,R,0,Math.PI*2); ctx.fillStyle='rgba(58,6,6,0.95)'; ctx.fill();
      // horns
      ctx.beginPath(); ctx.moveTo(-R*0.4,-R*0.6); ctx.quadraticCurveTo(-R*0.9,-R*1.2,-R*0.6,-R*1.45); ctx.lineTo(-R*0.3,-R*0.9);
      ctx.moveTo(R*0.4,-R*0.6); ctx.quadraticCurveTo(R*0.9,-R*1.2,R*0.6,-R*1.45); ctx.lineTo(R*0.3,-R*0.9);
      ctx.fillStyle = '#2b0000'; ctx.fill();
      // head / glowing eyes
      ctx.beginPath(); ctx.ellipse(0,-R*0.15,R*0.8,R*0.6,0,0,Math.PI*2); ctx.fillStyle='#111'; ctx.fill();
      // eyes
      ctx.beginPath(); ctx.fillStyle='#ffdd33'; ctx.ellipse(-R*0.35,-R*0.25,R*0.18,R*0.08,0,0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.ellipse(R*0.35,-R*0.25,R*0.18,R*0.08,0,0,Math.PI*2); ctx.fill();
      // faint glow
      ctx.beginPath(); ctx.arc(-R*0.35,-R*0.25, R*0.6, 0, Math.PI*2); ctx.fillStyle='rgba(255,220,90,0.06)'; ctx.fill();

    } else {
      // normal wizard
      const R = player.r;
      // robe
      ctx.beginPath(); ctx.arc(0,6,R,0,Math.PI*2); ctx.fillStyle='#24407a'; ctx.fill();
      // hat
      ctx.beginPath(); ctx.moveTo(-R*0.9,-R*0.9); ctx.quadraticCurveTo(0,-R*1.9,R*0.9,-R*0.9); ctx.lineTo(0,-R*0.3); ctx.closePath(); ctx.fillStyle='#18304f'; ctx.fill();
      // brim
      ctx.beginPath(); ctx.ellipse(0,-R*0.15,R*1.1,R*0.4,0,0,Math.PI*2); ctx.fillStyle='#132539'; ctx.fill();
      // face
      ctx.beginPath(); ctx.arc(0,-R*0.25,R*0.45,0,Math.PI*2); ctx.fillStyle='#f3d6b6'; ctx.fill();
      // eyes
      ctx.beginPath(); ctx.fillStyle='#111'; ctx.ellipse(-R*0.18,-R*0.3,3,5,0,0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.ellipse(R*0.18,-R*0.3,3,5,0,0,Math.PI*2); ctx.fill();
    }

    ctx.restore();
  }

  function drawProjectile(p){
    // draw jagged lightning path as short polyline
    const len = 28;
    ctx.save();
    ctx.translate(p.x, p.y);
    const angle = Math.atan2(p.dy, p.dx);
    ctx.rotate(angle);

    // glow
    const glowAlpha = clamp(1 - p.age/p.life, 0,1);
    ctx.beginPath(); ctx.moveTo(-len,0);
    for(let i=0;i<5;i++) ctx.lineTo(-len + (i+1)*(len/5), (Math.random()-0.5)*8);
    ctx.strokeStyle = `rgba(180,220,255,${0.22*glowAlpha})`; ctx.lineWidth=10; ctx.stroke();
    ctx.beginPath(); ctx.moveTo(-len,0);
    for(let i=0;i<5;i++) ctx.lineTo(-len + (i+1)*(len/5), (Math.random()-0.5)*4);
    ctx.strokeStyle = `rgba(220,255,255,${0.95*glowAlpha})`; ctx.lineWidth=3; ctx.stroke();
    ctx.restore();
  }

  function drawFlame(f){
    const pct = f.life;
    const x = f.x + Math.cos(f.ang)*f.dist;
    const y = f.y + Math.sin(f.ang)*f.dist;
    const a = clamp(pct,0,1);
    // flame blob
    ctx.beginPath(); ctx.ellipse(x,y, f.rad*(1+a*0.6), f.rad*(0.7+a*0.8), 0, 0, Math.PI*2);
    const g = ctx.createRadialGradient(x,y,1,x,y,f.rad*1.6);
    g.addColorStop(0, `rgba(255,220,120,${0.95*a})`);
    g.addColorStop(0.5, `rgba(255,120,30,${0.6*a})`);
    g.addColorStop(1, `rgba(60,20,10,${0.06*a})`);
    ctx.fillStyle = g; ctx.fill();
  }

  function render(){
    // background
    ctx.clearRect(0,0,W,H);

    // subtle grid or ground
    ctx.save();
    const grd = ctx.createLinearGradient(0,0,0,H);
    grd.addColorStop(0,'rgba(255,255,255,0.02)');
    grd.addColorStop(1,'rgba(0,0,0,0.02)');
    ctx.fillStyle = grd; ctx.fillRect(0,0,W,H);
    ctx.restore();

    // draw flames behind player
    for(const f of flames) drawFlame(f);

    // draw projectiles
    for(const p of projectiles) drawProjectile(p);

    // draw shield
    if(shield.active){
      ctx.beginPath(); ctx.arc(player.x, player.y, player.r*3.1, 0, Math.PI*2);
      ctx.fillStyle = 'rgba(160,200,255,0.06)'; ctx.fill();
      ctx.strokeStyle = 'rgba(170,220,255,0.22)'; ctx.lineWidth = 3; ctx.stroke();
    }

    // player aura during demon form
    if(demon.active){
      const glowRad = player.r * 3.2;
      const g = ctx.createRadialGradient(player.x,player.y,1,player.x,player.y,glowRad);
      g.addColorStop(0,'rgba(255,120,60,0.08)');
      g.addColorStop(1,'rgba(0,0,0,0)');
      ctx.fillStyle = g; ctx.beginPath(); ctx.arc(player.x,player.y,glowRad,0,Math.PI*2); ctx.fill();
    }

    // draw player
    drawWizard(player.x, player.y);

    // cursor aim indicator
    ctx.beginPath(); ctx.moveTo(player.x,player.y); ctx.lineTo(mouse.x,mouse.y);
    ctx.strokeStyle='rgba(255,255,255,0.03)'; ctx.lineWidth=1; ctx.setLineDash([6,6]); ctx.stroke(); ctx.setLineDash([]);
    ctx.beginPath(); ctx.arc(mouse.x, mouse.y, 6, 0, Math.PI*2); ctx.fillStyle='rgba(255,255,255,0.02)'; ctx.fill();

    // overlay text for debug small
    ctx.font='12px system-ui'; ctx.fillStyle='rgba(255,255,255,0.06)'; ctx.fillText(`Projectiles: ${projectiles.length}`, 10, H-14);
  }

  function loop(now){
    const dt = Math.min(0.05, (now - lastTime)/1000);
    lastTime = now;
    update(dt);
    render();
    requestAnimationFrame(loop);
  }

  requestAnimationFrame(loop);

  // keep canvas responsive
  window.addEventListener('resize', ()=>{
    const rect = canvas.getBoundingClientRect();
  });

  // polish: remove old projectiles when offscreen
  setInterval(()=>{
    for(let i=projectiles.length-1;i>=0;i--){
      const p = projectiles[i];
      if(p.x<-100||p.x>W+100||p.y<-100||p.y>H+100) projectiles.splice(i,1);
    }
  }, 2000);

})();
</script>
</body>
</html>
