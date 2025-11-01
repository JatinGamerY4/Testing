<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Offline Activation App</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--txt:#e6eef8;--muted:#9fb0c8}
    html,body{height:100%;margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;color:var(--txt);background:linear-gradient(180deg,#071026 0%, #052036 100%);}
    .center{display:flex;align-items:center;justify-content:center;height:100vh}
    .card{background:var(--card);padding:28px;border-radius:12px;box-shadow:0 8px 30px rgba(2,6,23,.6);width:360px;max-width:92%;text-align:center}
    h1{margin:0 0 8px;font-size:20px}
    p{margin:0 0 18px;color:var(--muted)}
    .row{display:flex;gap:8px;justify-content:center}
    button{background:#1f6feb;border:none;color:white;padding:10px 14px;border-radius:8px;font-weight:600;cursor:pointer}
    button.ghost{background:transparent;border:1px solid rgba(255,255,255,.08)}
    input{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,.06);background:transparent;color:var(--txt);margin-bottom:10px}
    .top-left{position:fixed;left:14px;top:12px}
    .info-btn{width:44px;height:44px;border-radius:10px;background:rgba(255,255,255,.03);display:flex;align-items:center;justify-content:center;font-weight:700;cursor:pointer}
    .small{font-size:13px;color:var(--muted)}
    .hidden{display:none}
    .notice{padding:8px;border-radius:8px;background:rgba(255,255,255,.03);margin-bottom:10px}
    .ok{color:#8ef29a}
    .bad{color:#ff8b8b}
    footer{margin-top:12px;font-size:12px;color:var(--muted)}
  </style>
</head>
<body>
  <div class="top-left">
    <div class="info-btn" id="infoBtn">i</div>
  </div>  <div class="center" id="root">
    <div class="card" id="authCard">
      <h1>Welcome</h1>
      <p class="small">Sign up or Login to continue (all data saved locally).</p>
      <div class="row" style="margin-bottom:12px">
        <button id="btnSignup">Sign Up</button>
        <button class="ghost" id="btnLogin">Login</button>
      </div>
      <div id="message" class="small"></div>
      <footer>Offline only -- data stored on this device.</footer>
    </div><div class="card hidden" id="signupCard">
  <h1>Create Account</h1>
  <p class="small">Enter your name and a valid email to create an account.</p>
  <div id="signupMsg" class="small"></div>
  <input type="text" id="su_name" placeholder="Your name" />
  <input type="email" id="su_email" placeholder="Email address" />
  <div class="row">
    <button id="createBtn">Create ID</button>
    <button class="ghost" id="su_back">Back</button>
  </div>
</div>

<div class="card hidden" id="loginCard">
  <h1>Login</h1>
  <p class="small">Type your registered email to login.</p>
  <div id="loginMsg" class="small"></div>
  <input type="email" id="li_email" placeholder="Registered email" />
  <div class="row">
    <button id="loginBtn">Login</button>
    <button class="ghost" id="li_back">Back</button>
  </div>
</div>

<div class="card hidden" id="mainCard">
  <h1 id="mainTitle">Main</h1>
  <p class="small" id="mainSub">Center option: Enter Activation Code</p>
  <div style="margin:16px 0" id="activationSection">
    <button id="enterCode">Enter Activation Code</button>
  </div>
  <div id="activationMsg" class="small"></div>
  <div id="welcomeBox" class="notice hidden"><strong>Welcome to app</strong></div>
  <div style="margin-top:12px" id="logoutSection">
    <button class="ghost" id="logout">Logout</button>
  </div>
</div>

  </div>  <div class="card hidden" id="infoCard" style="position:fixed;left:20px;top:70px;z-index:40">
    <h1>Account Info</h1>
    <p class="small">See your saved account details.</p>
    <div id="infoContent" class="small"></div>
    <div style="margin-top:8px" class="row">
      <button id="closeInfo" class="ghost">Close</button>
      <button id="adminBtn">Admin</button>
    </div>
  </div>  <div class="card hidden" id="adminCard" style="position:fixed;left:20px;top:70px;z-index:50">
    <h1>Admin -- Add Activation Code</h1>
    <p class="small">Add a code and link it to a specific App ID. Codes are single-use.</p>
    <div id="adminMsg" class="small"></div>
    <input id="adm_code" placeholder="Activation code (e.g. 1826122511)" />
    <input id="adm_appid" placeholder="Target App ID (10 digits)" />
    <div class="row">
      <button id="addCode">Add Code</button>
      <button class="ghost" id="closeAdmin">Close</button>
    </div>
    <div style="margin-top:10px" class="small">Existing codes:</div>
    <pre id="codesList" class="small" style="max-height:140px;overflow:auto;background:transparent;border:1px dashed rgba(255,255,255,.03);padding:8px;border-radius:8px"></pre>
  </div>  <script>
    const qs = s => document.querySelector(s);
    const rand10 = () => Math.floor(1000000000 + Math.random()*9000000000).toString();
    const emailValid = em => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(em);

    const USERS_KEY = 'offline_app_users_v1';
    const CODES_KEY = 'offline_app_codes_v1';
    const CURR_KEY = 'offline_app_current_email_v1';

    function loadUsers(){ try{ return JSON.parse(localStorage.getItem(USERS_KEY)||'[]') }catch(e){return[]} }
    function saveUsers(u){ localStorage.setItem(USERS_KEY, JSON.stringify(u)) }
    function loadCodes(){ try{ return JSON.parse(localStorage.getItem(CODES_KEY)||'{}') }catch(e){return{}} }
    function saveCodes(c){ localStorage.setItem(CODES_KEY, JSON.stringify(c)) }

    const authCard = qs('#authCard');
    const signupCard = qs('#signupCard');
    const loginCard = qs('#loginCard');
    const mainCard = qs('#mainCard');
    const infoCard = qs('#infoCard');
    const adminCard = qs('#adminCard');

    function show(el){[authCard,signupCard,loginCard,mainCard,infoCard,adminCard].forEach(x=>x.classList.add('hidden'));el.classList.remove('hidden')}

    qs('#btnSignup').onclick = ()=> show(signupCard);
    qs('#btnLogin').onclick = ()=> show(loginCard);
    qs('#su_back').onclick = ()=> show(authCard);
    qs('#li_back').onclick = ()=> show(authCard);

    qs('#createBtn').onclick = ()=>{
      const name = qs('#su_name').value.trim();
      const email = qs('#su_email').value.trim().toLowerCase();
      const msg = qs('#signupMsg'); msg.textContent = '';
      if(!name){ msg.textContent='Enter name'; return }
      if(!emailValid(email)){ msg.textContent='Enter valid email'; return }
      const users = loadUsers();
      if(users.find(u=>u.email===email)){ msg.textContent='Already Signed Up with this email'; return }
      const appId = rand10();
      const user = {name, email, appId, activated:false, activatedAt:null};
      users.push(user); saveUsers(users);
      localStorage.setItem(CURR_KEY, email);
      openMainFor(user);
    }

    qs('#loginBtn').onclick = ()=>{
      const email = qs('#li_email').value.trim().toLowerCase();
      const msg = qs('#loginMsg'); msg.textContent='';
      if(!emailValid(email)){ msg.textContent='Enter valid email'; return }
      const users = loadUsers();
      const user = users.find(u=>u.email===email);
      if(!user){ msg.textContent='No account for this email'; return }
      localStorage.setItem(CURR_KEY, email);
      openMainFor(user);
    }

    function openMainFor(user){
      show(mainCard);
      qs('#mainTitle').textContent = `Hello, ${user.name}`;
      qs('#mainSub').textContent = user.activated? 'Activated -- Enjoy the app' : 'Enter Activation Code';
      qs('#welcomeBox').classList.toggle('hidden', !user.activated);
      qs('#activationMsg').textContent = '';
      qs('#activationSection').classList.toggle('hidden', user.activated);
    }

    qs('#infoBtn').onclick = ()=>{
      const email = localStorage.getItem(CURR_KEY);
      if(!email){ alert('No user logged in -- login or signup first.'); return }
      const users = loadUsers(); const user = users.find(u=>u.email===email);
      if(!user){ alert('User not found'); return }
      show(infoCard);
      qs('#infoContent').innerHTML = `<div><strong>Name:</strong> ${user.name}</div><div><strong>Email:</strong> ${user.email}</div><div><strong>App ID:</strong> ${user.appId}</div><div><strong>Activated:</strong> ${user.activated? 'Yes' : 'No'}</div>`;
    }
    qs('#closeInfo').onclick = ()=> show(mainCard);

    qs('#logout').onclick = ()=>{ localStorage.removeItem(CURR_KEY); show(authCard); }

    qs('#enterCode').onclick = ()=>{
      const code = prompt('Enter activation code:'); if(!code) return;
      const email = localStorage.getItem(CURR_KEY);
      if(!email){ alert('No user logged in'); return }
      const users = loadUsers(); const user = users.find(u=>u.email===email);
      if(!user){ alert('User not found'); return }
      const codes = loadCodes();
      if(!codes[code]){ qs('#activationMsg').textContent = 'Not Active For You'; qs('#activationMsg').className='small bad'; return }
      const entry = codes[code];
      if(entry.used){ qs('#activationMsg').textContent = 'This activation code was already used'; qs('#activationMsg').className='small bad'; return }
      if(entry.appId !== user.appId){ qs('#activationMsg').textContent = 'Not Active For You'; qs('#activationMsg').className='small bad'; return }
      entry.used = true; entry.usedBy = user.email; entry.usedAt = new Date().toISOString(); codes[code]=entry; saveCodes(codes);
      user.activated = true; user.activatedAt = new Date().toISOString();
      const all = loadUsers().map(u=> u.email===user.email?user:u); saveUsers(all);
      qs('#activationMsg').textContent='Activated! Welcome.'; qs('#activationMsg').className='small ok';
      qs('#welcomeBox').classList.remove('hidden'); qs('#mainSub').textContent='Activated -- Enjoy the app';
      qs('#activationSection').classList.add('hidden');
    }

    qs('#adminBtn').onclick = ()=>{
      const p = prompt('Enter admin password to add codes:');
      if(p!== 'admin123'){ alert('Wrong password'); return }
      show(adminCard); refreshCodesList();
    }
    qs('#closeAdmin').onclick = ()=> show(infoCard);

    qs('#addCode').onclick = ()=>{
      const code = qs('#adm_code').value.trim(); const appid = qs('#adm_appid').value.trim();
      const msg = qs('#adminMsg'); msg.textContent='';
      if(!code || !appid){ msg.textContent='Enter both code and App ID'; return }
      if(!/^\d{10}$/.test(appid)){ msg.textContent='App ID must be 10 digits'; return }
      const codes = loadCodes();
      if(codes[code]){ msg.textContent='This activation code already exists'; return }
      codes[code] = {appId:appid, used:false}; saveCodes(codes);
      qs('#adm_code').value=''; qs('#adm_appid').value=''; msg.textContent='Added'; refreshCodesList();
    }

    function refreshCodesList(){ const codes = loadCodes(); let out = '';
      for(const k in codes){ out += `${k} => AppID:${codes[k].appId} | used:${codes[k].used ? 'yes' : 'no'}\n` }
      qs('#codesList').textContent = out || 'No codes yet';
    }

    window.addEventListener('load', ()=>{
      const cur = localStorage.getItem(CURR_KEY);
      if(cur){ const users = loadUsers(); const user = users.find(u=>u.email===cur); if(user) openMainFor(user); }
    });
  </script></body>
</html>
