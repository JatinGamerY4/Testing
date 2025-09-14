<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Stickman Table Shop — $5 to spend</title>
<style>
  :root {
    --bg: #f0f7ff;
    --panel: #ffffffcc;
    --accent: #2b8cff;
    --bad: #e64b4b;
    --good: #3cb371;
  }
  html,body{height:100%;margin:0;font-family:Inter,Arial,sans-serif;background:linear-gradient(#e8f3ff,#f8fbff);display:flex;align-items:center;justify-content:center}
  #game {
    width: 920px;
    max-width:96%;
    min-height:560px;
    background:var(--panel);
    box-shadow: 0 14px 40px rgba(18,38,63,0.12);
    border-radius:12px;
    overflow:hidden;
    display:grid;
    grid-template-columns: 1fr 360px;
  }
  /* canvas area */
  #world {
    position:relative;
    background: linear-gradient(180deg, #8ad, #5aa);
    overflow:hidden;
  }
  canvas { display:block; width:100%; height:100%; }

  /* HUD / shop panel */
  #shop {
    padding:18px;
    box-sizing:border-box;
    background:linear-gradient(180deg,#ffffffcc,#f7fbffcc);
    border-left:1px solid rgba(0,0,0,0.04);
  }
  h2 { margin:0 0 12px 0; font-size:20px; color:#123; }
  .money {
    font-size:20px; font-weight:700; margin-bottom:12px;
    display:flex; align-items:center; gap:10px;
  }
  .money .amount { color:var(--accent); font-size:22px; }
  .message {
    height:28px; margin-bottom:12px; color:#222; font-size:14px;
  }

  .items {
    display:grid;
    grid-template-columns: 1fr;
    gap:8px;
    margin-bottom:12px;
  }

  .item {
    display:flex; gap:10px; align-items:center; padding:8px; border-radius:8px;
    background:linear-gradient(180deg,#fff,#fafcff); box-shadow:0 2px 6px rgba(10,20,40,0.04);
    border:1px solid rgba(10,20,40,0.03);
  }
  .icon {
    width:56px; height:56px; border-radius:8px; display:flex;align-items:center;justify-content:center;
    font-size:26px; background:linear-gradient(180deg,#f2f6ff,#e9f2ff); color:#111;
    box-shadow:inset 0 -6px 14px rgba(0,0,0,0.02);
  }
  .meta { flex:1 }
  .meta .name { font-weight:700; color:#123; }
  .meta .price { color:#666; font-size:13px }
  .buy {
    display:flex; flex-direction:column; gap:6px; align-items:flex-end;
  }
  .btn {
    background:var(--accent); color:white; border:none; padding:8px 12px; border-radius:8px; cursor:pointer;
    font-weight:700; box-shadow:0 6px 12px rgba(43,140,255,0.18);
  }
  .btn:active{ transform: translateY(1px) }
  .btn.small { padding:6px 10px; font-size:13px; font-weight:600; background:#5a5a5a; box-shadow:none; }
  .btn:disabled { opacity:0.5; cursor:not-allowed; transform:none }

  .inventory {
    margin-top:12px; font-size:14px;
  }
  .inv-list { margin:6px 0 0 0; padding:0; list-style:none; max-height:150px; overflow:auto; }
  .inv-list li { padding:6px 8px; border-radius:6px; background:#fff; margin-bottom:6px; border:1px solid rgba(0,0,0,0.03); display:flex;justify-content:space-between; align-items:center; font-weight:600; color:#333 }

  /* HUD overlays */
  #hudPrompt {
    position:absolute; left:16px; top:14px; background:rgba(255,255,255,0.85); padding:8px 12px; border-radius:10px; font-weight:700; color:#123;
    box-shadow:0 6px 14px rgba(0,0,0,0.08);
  }

  /* feedback text when purchase fails */
  .flash {
    animation: flashRed 0.9s ease;
  }
  @keyframes flashRed {
    0%{background:var(--bad); color:#fff}
    100%{background:transparent; color:inherit}
  }

  /* small footer */
  footer { font-size:12px; color:#456; margin-top:8px; opacity:0.9 }
</style>
</head>
<body>

<div id="game" role="application" aria-label="Stickman Table Shop">
  <div id="world">
    <div id="hudPrompt">Click items to buy — stickman has <strong>$5</strong></div>
    <canvas id="canvas"></canvas>
  </div>

  <aside id="shop" aria-label="Shop and inventory">
    <h2>Shop — Spend wisely</h2>
    <div class="money">Money: <span class="amount" id="moneyDisplay">$5.00</span></div>
    <div class="message" id="message">Welcome! Tap an item to buy.</div>

    <div class="items" id="itemsList">
      <!-- Items populated by JS -->
    </div>

    <div class="inventory">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <strong>Inventory</strong>
        <button class="btn small" id="resetBtn">Reset</button>
      </div>
      <ul class="inv-list" id="inventoryList"></ul>
      <footer>Tip: Not enough money? hit <strong>Reset</strong> to get $5 again.</footer>
    </div>
  </aside>
</div>

<!-- Audio: short success & fail beeps (synthesized) generated on demand -->
<script>
/*
  Stickman Table Shop
  - Canvas draws a simple table and a stickman sitting on a stool.
  - Player starts with $5.00.
  - Shop has several items (name, price, emoji icon).
  - Buying reduces money and adds to inventory; stickman animates to hold item.
  - If insufficient funds, UI flashes and stickman shakes head.
  - Reset button restores $5 and clears inventory.

  No external assets required — everything is drawn procedurally.
*/

(() => {
  // ------------ Configuration ------------
  const START_MONEY = 5.00;

  // define shop items (emoji icons used to avoid external images)
  const SHOP_ITEMS = [
    { id: 'apple', name: 'Apple', price: 1.00, icon: '🍎', desc: 'A tasty snack.' },
    { id: 'coffee', name: 'Coffee', price: 2.00, icon: '☕', desc: 'Wake up!' },
    { id: 'hat', name: 'Hat', price: 3.00, icon: '🎩', desc: 'Stylish.' },
    { id: 'book', name: 'Book', price: 4.00, icon: '📘', desc: 'Learn stuff.' },
    { id: 'toycar', name: 'Toy Car', price: 5.00, icon: '🚗', desc: 'Fun to play with.' },
  ];

  // ------------ State ------------
  let money = START_MONEY;
  const inventory = []; // list of item ids
  let heldItem = null; // item id stickman is currently holding (displayed near hand)
  let lastActionTimestamp = 0;

  // ------------ DOM refs ------------
  const moneyDisplay = document.getElementById('moneyDisplay');
  const itemsList = document.getElementById('itemsList');
  const inventoryList = document.getElementById('inventoryList');
  const messageEl = document.getElementById('message');
  const resetBtn = document.getElementById('resetBtn');
  const hudPrompt = document.getElementById('hudPrompt');

  // ------------ Canvas setup ------------
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d', { alpha: false });
  function resizeCanvas() {
    const rect = canvas.getBoundingClientRect();
    canvas.width = Math.floor(rect.width * devicePixelRatio);
    canvas.height = Math.floor(rect.height * devicePixelRatio);
    ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  }
  // initial sizing: observe container
  window.addEventListener('resize', () => resizeCanvas());
  setTimeout(resizeCanvas, 50);

  // ------------ Audio helpers ------------
  let audioCtx = null;
  function beep(frequency = 440, duration = 0.12, type='sine', gain=0.14) {
    try {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      const t0 = audioCtx.currentTime;
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.type = type;
      o.frequency.setValueAtTime(frequency, t0);
      g.gain.setValueAtTime(gain * 0.0001, t0);
      g.gain.exponentialRampToValueAtTime(gain, t0 + 0.01);
      g.gain.exponentialRampToValueAtTime(0.0001, t0 + duration);
      o.connect(g); g.connect(audioCtx.destination);
      o.start(t0);
      o.stop(t0 + duration + 0.02);
    } catch (e) { /* ignore */ }
  }

  function successSound(){ beep(880, 0.09, 'sine', 0.08); beep(1100, 0.06, 'triangle', 0.06); }
  function failSound(){ beep(220, 0.14, 'sawtooth', 0.12); }

  // ------------ UI rendering ------------
  function formatMoney(v){ return '$' + v.toFixed(2); }
  function updateMoneyDisplay(){
    moneyDisplay.textContent = formatMoney(money);
    // disable buttons if we cannot buy
    document.querySelectorAll('.btn.buy-btn').forEach(btn => {
      const price = parseFloat(btn.dataset.price);
      btn.disabled = (price > money);
    });
  }

  function showMessage(text, kind='normal') {
    messageEl.textContent = text;
    messageEl.classList.remove('flash');
    if (kind === 'bad') {
      messageEl.style.color = getComputedStyle(document.documentElement).getPropertyValue('--bad') || '#e64b4b';
      // flash background quickly
      messageEl.classList.add('flash');
      failSound();
    } else {
      messageEl.style.color = '';
      if (kind === 'good') successSound();
    }
  }

  // populate shop
  function buildShop(){
    itemsList.innerHTML = '';
    SHOP_ITEMS.forEach(it => {
      const el = document.createElement('div');
      el.className = 'item';
      el.innerHTML = `
        <div class="icon" aria-hidden="true">${it.icon}</div>
        <div class="meta">
          <div class="name">${it.name}</div>
          <div class="price">${it.desc} — <strong>${formatMoney(it.price)}</strong></div>
        </div>
        <div class="buy">
          <button class="btn buy-btn" data-id="${it.id}" data-price="${it.price}">Buy</button>
        </div>
      `;
      itemsList.appendChild(el);
    });

    // attach buy handlers
    document.querySelectorAll('.buy-btn').forEach(btn => {
      btn.addEventListener('click', (ev) => {
        const id = btn.dataset.id;
        const price = parseFloat(btn.dataset.price);
        attemptBuy(id, price);
      });
    });
    updateMoneyDisplay();
  }

  // inventory UI
  function rebuildInventory(){
    inventoryList.innerHTML = '';
    if (inventory.length === 0) {
      const li = document.createElement('li');
      li.textContent = 'Empty';
      li.style.opacity = 0.6;
      inventoryList.appendChild(li);
      return;
    }
    inventory.forEach((id, idx) => {
      const it = SHOP_ITEMS.find(x=>x.id===id);
      const li = document.createElement('li');
      li.innerHTML = `<span>${it.icon} ${it.name}</span><span style="opacity:0.7;font-weight:600">${formatMoney(it.price)}</span>`;
      inventoryList.appendChild(li);
    });
  }

  // ------------ Buy logic ------------
  function attemptBuy(itemId, price) {
    const now = Date.now();
    // basic rate limit to prevent spamming
    if (now - lastActionTimestamp < 120) return;
    lastActionTimestamp = now;

    const item = SHOP_ITEMS.find(x => x.id === itemId);
    if (!item) return;

    if (price > money + 1e-9) {
      // not enough
      showMessage(`Not enough money for ${item.name}!`, 'bad');
      triggerStickmanShake();
      return;
    }
    // perform purchase
    money = +(money - price).toFixed(2);
    inventory.push(itemId);
    heldItem = itemId; // make stickman hold the new item (visual)
    showMessage(`You bought ${item.name} for ${formatMoney(price)}.`, 'good');
    updateMoneyDisplay();
    rebuildInventory();
    triggerStickmanReach(item.icon);
  }

  resetBtn.addEventListener('click', ()=>{
    money = START_MONEY;
    inventory.length = 0;
    heldItem = null;
    rebuildInventory();
    updateMoneyDisplay();
    showMessage('Reset: you have $5.00 again.');
  });

  // ------------ Stickman animation & drawing ------------
  const state = {
    // joint angles & animation timers
    headShake: 0,         // -1..1 for left-right head shake
    reachProgress: 0,     // 0..1 progress of reach-to-hand animation
    reachIcon: null,      // emoji icon to draw while holding/reaching
    reachTimer: 0,
    shakeTimer: 0
  };

  function triggerStickmanShake() {
    state.shakeTimer = 400; // ms
  }
  function triggerStickmanReach(icon) {
    state.reachIcon = icon || null;
    state.reachTimer = 900; // ms
    state.reachProgress = 0;
  }

  // Draw helper functions (2D stickman sitting on stool at table)
  function drawScene(dt) {
    const W = canvas.width / devicePixelRatio;
    const H = canvas.height / devicePixelRatio;
    ctx.clearRect(0,0,W,H);

    // Background gradient (simple sky -> floor)
    const g = ctx.createLinearGradient(0,0,0,H);
    g.addColorStop(0, '#86c3ff');
    g.addColorStop(0.6, '#bfe0ff');
    g.addColorStop(1, '#dff5ff');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,W,H);

    // table position
    const tableY = Math.round(H * 0.62);
    const tableTopHeight = 24;
    const tableLeft = Math.round(W * 0.06);
    const tableRight = Math.round(W * 0.9);
    const tableWidth = tableRight - tableLeft;

    // draw table top
    ctx.fillStyle = '#6b4a2b';
    roundRect(ctx, tableLeft, tableY - tableTopHeight, tableWidth, tableTopHeight, 8, true, false);
    // table leg
    ctx.fillStyle = '#55381f';
    const legW = 20;
    ctx.fillRect(W*0.5 - legW/2, tableY - tableTopHeight, legW, H - tableY + tableTopHeight);

    // stool (left)
    const stoolX = W*0.5 - 170;
    const stoolY = tableY + 26;
    drawStool(stoolX, stoolY);

    // stickman position (sits near center-left of table)
    const manX = W*0.5 - 110;
    const manY = tableY - tableTopHeight; // table top y
    drawStickman(manX, manY, state, dt);

    // draw any held item on table (if inventory not null, show small icons on table near him)
    drawTableItems(W, tableY, inventory);

    // if reach animation active, draw floating icon near his hand
    if (state.reachTimer > 0 && state.reachIcon) {
      const t = 1 - (state.reachTimer / 900);
      const px = manX + 40 + t*40;
      const py = manY - 30 - Math.sin(t * Math.PI) * 22;
      ctx.font = `${20 + Math.round(t*6)}px serif`;
      ctx.fillText(state.reachIcon, px, py);
    }
  }

  // stool drawing
  function drawStool(x,y){
    ctx.save();
    ctx.translate(x,y);
    ctx.fillStyle = '#4b3a2f';
    ctx.fillRect(-36,0,72,8); // seat
    ctx.fillStyle = '#2c1f1a';
    ctx.fillRect(-22,8,12,44);
    ctx.fillRect(10,8,12,44);
    ctx.restore();
  }

  function drawTableItems(W, tableY, inv){
    const startX = W*0.08;
    const gap = 36;
    ctx.font = '20px serif';
    for (let i=0;i<Math.min(inv.length,6);i++){
      const it = SHOP_ITEMS.find(x=>x.id===inv[i]);
      const x = startX + i * gap;
      const y = tableY - 10;
      ctx.fillText(it.icon, x, y);
    }
  }

  // draw stickman
  function drawStickman(cx, tableTopY, s, dt) {
    // Basic proportions
    const headR = 14;
    const bodyLen = 46;
    const armLen = 36;
    const legLen = 44;

    // compute timers
    if (s.reachTimer > 0) { s.reachTimer -= dt; if (s.reachTimer < 0) s.reachTimer = 0; }
    if (s.shakeTimer > 0) { s.shakeTimer -= dt; if (s.shakeTimer < 0) s.shakeTimer = 0; }

    // head shake amount
    const shakeAmt = s.shakeTimer > 0 ? Math.sin((s.shakeTimer/60) * 0.04 * Math.PI * 8) * 0.9 : 0;

    // body base
    const baseX = cx;
    const baseY = tableTopY - 6;

    // draw legs (sitting on stool)
    const leftHipX = baseX - 6;
    const leftHipY = baseY + 6;
    const rightHipX = baseX + 6;
    const rightHipY = baseY + 6;
    ctx.strokeStyle = '#111';
    ctx.lineWidth = 3.2;
    ctx.lineCap = 'round';

    ctx.beginPath();
    // left thigh downward and forward (sitting)
    ctx.moveTo(leftHipX, leftHipY);
    ctx.lineTo(leftHipX - 4, leftHipY + legLen);
    // right thigh => down
    ctx.moveTo(rightHipX, rightHipY);
    ctx.lineTo(rightHipX + 6, rightHipY + legLen);
    ctx.stroke();

    // draw body trunk
    ctx.beginPath();
    ctx.moveTo(baseX, baseY);
    ctx.lineTo(baseX, baseY - bodyLen);
    ctx.stroke();

    // head (on top of body)
    const headX = baseX + (shakeAmt * 2.5);
    const headY = baseY - bodyLen - headR - 2;
    ctx.beginPath();
    ctx.fillStyle = '#fff';
    ctx.strokeStyle = '#111';
    ctx.lineWidth = 3;
    ctx.arc(headX, headY, headR, 0, Math.PI*2);
    ctx.fill();
    ctx.stroke();

    // eyes
    ctx.fillStyle = '#111';
    ctx.beginPath();
    ctx.arc(headX - 5 + shakeAmt*1.8, headY - 2, 2.2, 0, Math.PI*2);
    ctx.arc(headX + 5 + shakeAmt*1.8, headY - 2, 2.2, 0, Math.PI*2);
    ctx.fill();

    // simple mouth (change when happy? not needed)
    ctx.beginPath();
    ctx.lineWidth = 2;
    ctx.moveTo(headX - 4, headY + 6);
    ctx.lineTo(headX + 4, headY + 6);
    ctx.stroke();

    // arms: left rests on table, right reaches
    // left arm (on table)
    ctx.lineWidth = 3.4;
    ctx.beginPath();
    ctx.moveTo(baseX - 2, baseY - 20);
    ctx.lineTo(baseX - 70, baseY - 8); // rests on table surface
    ctx.stroke();

    // right arm: depending on reachTimer animate forward/up
    const reachActive = s.reachTimer > 0;
    const reachT = reachActive ? (1 - s.reachTimer/900) : 0;
    // compute eased progress
    const ease = reachT < 0.5 ? (2*reachT*reachT) : (-1 + (4 - 2*reachT)*reachT); // rough ease
    const armStartX = baseX + 2;
    const armStartY = baseY - 18;
    const handTargetX = baseX + 36 + (reachActive ? ease * 40 : 0);
    const handTargetY = baseY - 24 - (reachActive ? Math.sin(ease * Math.PI) * 18 : 0);

    ctx.beginPath();
    ctx.moveTo(armStartX, armStartY);
    ctx.lineTo(handTargetX, handTargetY);
    ctx.stroke();

    // draw held item near hand if any
    if (heldItem) {
      const it = SHOP_ITEMS.find(x=>x.id===heldItem);
      if (it) {
        ctx.font = '26px serif';
        ctx.fillText(it.icon, handTargetX + 6, handTargetY + 8);
      }
    }

    // tiny shadow under stool/man
    ctx.fillStyle = 'rgba(0,0,0,0.12)';
    ctx.beginPath();
    ctx.ellipse(baseX, baseY + legLen + 12, 60, 12, 0, 0, Math.PI*2);
    ctx.fill();
  }

  function roundRect(ctx,x,y,w,h,r,fill,stroke){
    if (typeof stroke === 'undefined') stroke = true;
    if (typeof r === 'undefined') r = 5;
    ctx.beginPath();
    ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    ctx.closePath();
    if (fill) ctx.fill();
    if (stroke) ctx.stroke();
  }

  // main loop
  let lastTime = performance.now();
  function tick(now) {
    const dt = now - lastTime;
    lastTime = now;

    // Update state timers
    if (state.reachTimer > 0) {
      // when complete, keep held item for a little while
      if (state.reachTimer < 120) {
        // after reach ends, hold item for a while
        // set heldItem preserved (already set), nothing else
      }
    }

    // draw
    drawScene(dt);

    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);

  // ------------ Input helpers ------------
  // Clicking item in the world could also buy (optional). For now shop buttons handle it.
  // But we can also click the canvas to make stickman wave!
  canvas.addEventListener('click', () => {
    // small wave: toggle head shake a bit to show interaction
    state.shakeTimer = 240;
    showMessage('You tapped the scene. Try buying something!', '');
  });

  // ---- initialize UI ----
  buildShop();
  rebuildInventory();
  updateMoneyDisplay();
  showMessage('Welcome! You have $5.00.');

  // Accessibility: announce money changes via aria-live (optional)
  // (Could create a hidden live region; omitted for brevity)

  // Expose a small test API in console for debugging:
  window._shop = { money, inventory, attemptBuy, reset: ()=>resetBtn.click() };

})();
</script>
</body>
</html>
