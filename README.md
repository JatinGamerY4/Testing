<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1"/>
<title>Portrait Car Driving Game</title>
<style>
  html,body{
    margin:0; height:100%; background:#111; overflow:hidden;
    font-family:sans-serif;
  }
  #game {
    position:relative; width:100%; height:100%;
  }
  canvas {
    display:block; width:100%; height:100%;
    background:linear-gradient(#87ceeb 0%, #7ec0ff 30%, #2b7 100%);
  }

  /* HUD */
  .hud {
    position:absolute; top:10px; left:50%; transform:translateX(-50%);
    background:rgba(0,0,0,0.5); padding:6px 12px; border-radius:8px;
    color:#fff; font-size:16px; text-align:center;
  }

  /* Controls for portrait */
  .controls {
    position:absolute; bottom:12px; left:50%; transform:translateX(-50%);
    display:flex; flex-wrap:wrap; gap:10px; justify-content:center;
  }
  button.control {
    background:rgba(255,255,255,0.1);
    border:1px solid rgba(255,255,255,0.3);
    border-radius:10px; color:#fff; font-size:20px;
    padding:18px 24px; min-width:60px;
  }

  /* Car overlay (looks like a car interior/hood) */
  .car {
    position:absolute; bottom:0; left:50%; transform:translateX(-50%);
    width:100%; height:40%;
    pointer-events:none;
  }
  .car svg { width:100%; height:100%; }
</style>
</head>
<body>
<div id="game">
  <canvas id="c"></canvas>

  <div class="hud">
    <div id="speed">0 km/h</div>
  </div>

  <div class="controls">
    <button class="control" id="btnLeft">◀</button>
    <button class="control" id="btnAccelerate">▲</button>
    <button class="control" id="btnBrake">▼</button>
    <button class="control" id="btnRight">▶</button>
  </div>

  <!-- Car overlay -->
  <div class="car">
    <svg viewBox="0 0 100 60" preserveAspectRatio="xMidYMax meet">
      <!-- Hood -->
      <path d="M0,60 L20,20 L80,20 L100,60 Z" fill="#e63939"/>
      <!-- Windshield -->
      <rect x="25" y="10" width="50" height="12" rx="3" fill="#cfe8ff" stroke="#222" stroke-width="1"/>
      <!-- Dashboard -->
      <rect x="0" y="40" width="100" height="20" fill="#111"/>
      <!-- Steering wheel -->
      <circle cx="50" cy="45" r="7" fill="#333" stroke="#666" stroke-width="2"/>
    </svg>
  </div>
</div>

<!-- Sound effects -->
<audio id="engine" src="engine-loop.mp3" loop preload="auto"></audio>
<audio id="brake" src="brake.mp3" preload="auto"></audio>
<audio id="click" src="click.mp3" preload="auto"></audio>

<script>
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
function resize(){
  canvas.width = canvas.clientWidth;
  canvas.height = canvas.clientHeight;
}
window.addEventListener('resize', resize);
resize();

// Simple player state
const state = { speed:0, pos:0, input:{left:false,right:false,accel:false,brake:false} };

const engine = document.getElementById('engine');
const brakeSound = document.getElementById('brake');
const clickSound = document.getElementById('click');

// Attach controls
function controlBtn(id, key){
  const el=document.getElementById(id);
  el.addEventListener('mousedown', ()=>{ state.input[key]=true; clickSound.play(); });
  el.addEventListener('mouseup', ()=>{ state.input[key]=false; });
  el.addEventListener('touchstart', e=>{ e.preventDefault(); state.input[key]=true; clickSound.play(); });
  el.addEventListener('touchend', ()=>{ state.input[key]=false; });
}
controlBtn('btnAccelerate','accel');
controlBtn('btnBrake','brake');
controlBtn('btnLeft','left');
controlBtn('btnRight','right');

// Keyboard
document.addEventListener('keydown',e=>{
  if(e.key==='ArrowUp') state.input.accel=true;
  if(e.key==='ArrowDown') state.input.brake=true;
  if(e.key==='ArrowLeft') state.input.left=true;
  if(e.key==='ArrowRight') state.input.right=true;
});
document.addEventListener('keyup',e=>{
  if(e.key==='ArrowUp') state.input.accel=false;
  if(e.key==='ArrowDown') state.input.brake=false;
  if(e.key==='ArrowLeft') state.input.left=false;
  if(e.key==='ArrowRight') state.input.right=false;
});

function update(dt){
  if(state.input.accel){ state.speed += 200*dt; if(engine.paused) engine.play(); }
  else { state.speed -= 100*dt; if(state.speed<=0 && !engine.paused){ engine.pause(); engine.currentTime=0; } }
  if(state.input.brake){ state.speed -= 300*dt; brakeSound.play(); }
  state.speed = Math.max(0, Math.min(400, state.speed));
  state.pos += state.speed*dt;
  document.getElementById('speed').textContent = Math.floor(state.speed*0.6)+" km/h";
}

function draw(){
  const W=canvas.width,H=canvas.height;
  ctx.fillStyle='#87ceeb'; ctx.fillRect(0,0,W,H/2);
  ctx.fillStyle='#2b7'; ctx.fillRect(0,H/2,W,H/2);
  // road trapezoid
  ctx.fillStyle='#333';
  ctx.beginPath();
  ctx.moveTo(W*0.25,H*0.5);
  ctx.lineTo(W*0.75,H*0.5);
  ctx.lineTo(W*0.9,H);
  ctx.lineTo(W*0.1,H);
  ctx.closePath();
  ctx.fill();
  // center line
  ctx.strokeStyle='#ff0'; ctx.lineWidth=4;
  ctx.setLineDash([20,20]);
  ctx.beginPath();
  ctx.moveTo(W/2,H*0.5);
  ctx.lineTo(W/2,H);
  ctx.stroke();
  ctx.setLineDash([]);
}

let last=performance.now();
function loop(now){
  const dt=(now-last)/1000; last=now;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
</script>
</body>
</html>
