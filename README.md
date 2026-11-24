# Tgplay
Tg mini app 
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>GameOn — игры с друзьями</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
:root{
  --bg:#17212b; --bg2:#232e3c; --card:#2b5278; --accent:#5288c1;
  --text:#f5f5f5; --hint:#7f91a4; --btn:#5288c1; --btnText:#ffffff;
  --green:#31b545; --danger:#e53935; --gold:#fdd835;
}
*{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent}
body{background:var(--bg);color:var(--text);font-family:-apple-system,"Segoe UI",Roboto,Arial,sans-serif;min-height:100vh;padding:16px;padding-bottom:90px}
h1{font-size:22px;margin-bottom:4px}
.sub{color:var(--hint);font-size:13px;margin-bottom:16px}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.card{background:var(--bg2);border-radius:16px;padding:18px 14px;display:flex;flex-direction:column;align-items:center;gap:8px;cursor:pointer;border:1px solid transparent;transition:.15s;user-select:none}
.card:active{transform:scale(.96);border-color:var(--accent)}
.card .ico{font-size:44px;line-height:1}
.card .nm{font-weight:600;font-size:15px}
.card .st{font-size:11px;color:var(--hint)}
.badge{font-size:10px;padding:2px 8px;border-radius:10px;background:var(--green);color:#fff}
.badge.soon{background:var(--hint)}
.screen{display:none}
.screen.on{display:block;animation:fade .2s}
@keyframes fade{from{opacity:0;transform:translateY(6px)}to{opacity:1}}
.desc{font-size:13px;color:var(--hint);line-height:1.5;margin:10px 0 14px}
.btn{display:block;width:100%;padding:14px;border:none;border-radius:12px;background:var(--btn);color:var(--btnText);font-size:15px;font-weight:600;cursor:pointer;margin-bottom:10px}
.btn:active{opacity:.8}
.btn.ghost{background:var(--bg2);color:var(--text)}
.btn.green{background:var(--green)}
.btn:disabled{opacity:.4}
h2{font-size:18px;margin-bottom:6px}
input[type=text]{width:100%;padding:13px;border-radius:12px;border:1px solid var(--accent);background:var(--bg2);color:var(--text);font-size:16px;text-transform:uppercase;letter-spacing:2px;margin-bottom:10px;outline:none}
.lobby{background:var(--bg2);border-radius:14px;padding:16px;margin-top:14px;text-align:center}
.code{font-size:32px;font-weight:800;letter-spacing:6px;margin:8px 0;color:var(--accent)}
canvas{display:block;margin:14px auto;background:#0e1621;border-radius:14px;max-width:100%}
.row{display:flex;gap:10px;align-items:center;margin:12px 0}
input[type=range]{flex:1}
.stat{font-size:13px;color:var(--hint);text-align:center;margin-top:8px}
.online{font-size:11px;color:var(--green);margin-bottom:10px}
.online.off{color:var(--danger)}
.joinbox{display:flex;gap:8px;margin-bottom:14px}
.joinbox input{margin:0;flex:1}
.joinbox .btn{width:auto;padding:13px 18px;margin:0}
.vsbar{display:flex;justify-content:space-around;background:var(--bg2);border-radius:12px;padding:10px;margin-top:12px;font-size:14px}
.vsbar b{font-size:18px}
.turn{margin-top:8px;text-align:center;font-size:13px;color:var(--gold)}
</style>
</head>
<body>

<div id="home" class="screen on">
  <h1>🎮 GameOn</h1>
  <div class="sub">Играй с друзьями прямо в Telegram</div>
  <div class="online" id="netStatus">🟢 онлайн — подключено</div>
  <div class="joinbox">
    <input type="text" id="joinCode" placeholder="КОД" maxlength="5">
    <button class="btn" onclick="joinByCode()">Войти</button>
  </div>
  <div class="grid">
    <div class="card" onclick="openGame('chess')"><div class="ico">♟️</div><div class="nm">Шахматы</div><span class="badge soon">скоро</span></div>
    <div class="card" onclick="openGame('cars')"><div class="ico">🏎️</div><div class="nm">Машинки</div><span class="badge">онлайн</span></div>
    <div class="card" onclick="openGame('volley')"><div class="ico">🏐</div><div class="nm">Волейбол</div><span class="badge soon">скоро</span></div>
    <div class="card" onclick="openGame('box')"><div class="ico">🥊</div><div class="nm">Бокс</div><span class="badge soon">скоро</span></div>
    <div class="card" onclick="openGame('basket')"><div class="ico">🏀</div><div class="nm">Баскетбол</div><span class="badge">онлайн</span></div>
  </div>
</div>

<div id="game" class="screen">
  <h2 id="gTitle"></h2>
  <div class="desc" id="gDesc"></div>
  <button class="btn green" id="btnLobby" onclick="createLobby()">⚔️ Создать лобби</button>
  <button class="btn" onclick="quickMatch()">⚡ Быстрый матч</button>
  <button class="btn ghost" onclick="inviteFriend()">📨 Пригласить друга</button>
  <div class="lobby" id="lobby" style="display:none">
    <div style="font-size:13px;color:var(--hint)">Код комнаты — скажи его другу:</div>
    <div class="code" id="roomCode">-----</div>
    <div class="stat" id="lobbyStatus">⏳ Ожидаем второго игрока…</div>
    <button class="btn green" id="btnStart" style="margin-top:12px;display:none" onclick="startGame()">▶️ Начать игру</button>
  </div>
  <div id="demo" style="display:none"></div>
  <button class="btn ghost" style="margin-top:16px" onclick="goHome()">← Назад</button>
</div>

<script>
/* ================= Telegram ================= */
const tg = window.Telegram && window.Telegram.WebApp ? window.Telegram.WebApp : null;
if (tg) { tg.ready(); tg.expand(); try{ tg.setHeaderColor('#17212b'); tg.setBackgroundColor('#17212b'); }catch(e){} tg.BackButton.onClick(goHome); }
function haptic(t){ try{ tg && tg.HapticFeedback && tg.HapticFeedback.impactOccurred(t||'light'); }catch(e){} }

/* ================= Сеть ================= */
const WS_URL = location.origin.replace(/^http/,'ws');
let ws=null, myRoom=null, isHost=false, peerHere=false;
function connect(){
  ws = new WebSocket(WS_URL);
  ws.onopen = ()=>{ setNet(true); if (localStorage.goc) { sendW({t:'join',code:localStorage.goc}); } };
  ws.onclose = ()=>{ setNet(false); myRoom=null; setTimeout(connect, 1500); };
  ws.onmessage = ev => { let m; try{ m = JSON.parse(ev.data); }catch(e){ return; } onMsg(m); };
}
function setNet(ok){
  const el=document.getElementById('netStatus');
  el.textContent = ok ? '🟢 онлайн — подключено' : '🔴 офлайн — переподключение…';
  el.className = 'online'+(ok?'':' off');
}
function sendW(o){ if (ws && ws.readyState===1) ws.send(JSON.stringify(o)); }
connect();

/* ================= Состояние ================= */
const GAMES = {
  chess:  {t:'♟️ Шахматы', d:'Классическая партия с другом. Онлайн-дуэль: создай лобби или введи код комнаты.'},
  cars:   {t:'🏎️ Машинки', d:'Онлайн-гонка: кто дольше уклоняется от трафика — тот победил! Можно и в соло-режиме.'},
  volley: {t:'🏐 Волейбол', d:'Пляжный волейбол 1×1. Лобби уже работает — сама игра скоро.'},
  box:    {t:'🥊 Бокс', d:'Дуэль в ринге. Лобби уже работает — сама игра скоро.'},
  basket: {t:'🏀 Баскетбол', d:'По очереди бросайте штрафные: у каждого 5 бросков, кто точнее — тот и победил!'},
};
let current=null, versus=false, versusStarted=false;

function openGame(id){
  current=id; haptic('medium');
  document.getElementById('home').classList.remove('on');
  document.getElementById('game').classList.add('on');
  const g=GAMES[id];
  document.getElementById('gTitle').textContent=g.t;
  document.getElementById('gDesc').textContent=g.d;
  document.getElementById('lobby').style.display='none';
  document.getElementById('demo').style.display='none';
  document.getElementById('demo').innerHTML='';
  versus=false; versusStarted=false;
  if (tg) tg.BackButton.show();
  if (id==='cars') initCars(); else if (id==='basket') initBasket();
}
function goHome(){
  stopLoops(); if (myRoom){ sendW({t:'leave'}); myRoom=null; localStorage.removeItem('goc'); }
  document.getElementById('game').classList.remove('on');
  document.getElementById('home').classList.add('on');
  if (tg) tg.BackButton.hide();
}

/* ================= Лобби ================= */
function createLobby(){
  haptic('medium'); sendW({t:'create'});
}
function joinByCode(){
  const c=document.getElementById('joinCode').value.trim().toUpperCase();
  if (!c) return; haptic('medium'); sendW({t:'join', code:c});
}
function quickMatch(){ haptic('medium'); sendW({t:'quick'}); showLobby('?????','⚡ Ищем соперника…'); }
function inviteFriend(){
  haptic('medium');
  const url='https://t.me/share/url?url='+encodeURIComponent('https://t.me/ТВОЙ_БОТ?startapp=join_'+ (myRoom||''))+'&text='+encodeURIComponent('Сыграй со мной в '+GAMES[current].t+'!');
  if (tg && tg.openTelegramLink) tg.openTelegramLink(url); else window.open(url,'_blank');
}
function showLobby(code, status){
  document.getElementById('lobby').style.display='block';
  document.getElementById('roomCode').textContent=code;
  document.getElementById('lobbyStatus').textContent=status;
}
function startGame(){ sendW({t:'start'}); }

/* ================= Обработка сообщений ================= */
function onMsg(m){
  switch(m.t){
    case 'created': myRoom=m.code; isHost=true; peerHere=false; localStorage.goc=m.code;
      showLobby(m.code,'⏳ Ожидаем второго игрока…'); document.getElementById('btnStart').style.display='none'; break;
    case 'joined': myRoom=m.code; isHost=false; peerHere=false; localStorage.goc=m.code;
      showLobby(m.code,'✅ Вы в комнате! Ждём старта от хоста…'); document.getElementById('btnStart').style.display='none'; break;
    case 'waiting': showLobby('⚡','Поиск соперника…'); break;
    case 'peer_joined': peerHere=true;
      document.getElementById('lobbyStatus').textContent='✅ Друг подключился!';
      document.getElementById('btnStart').style.display = isHost ? 'block' : 'none';
      if (!isHost) document.getElementById('lobbyStatus').textContent='✅ Друг подключился! Ждём старта…';
      haptic('heavy'); break;
    case 'start_game': versus=true; versusStarted=true;
      document.getElementById('lobbyStatus').textContent='🏁 Игра началась!';
      document.getElementById('btnStart').style.display='none';
      haptic('heavy');
      if (current==='cars') initCars(); else if (current==='basket') initBasket();
      else document.getElementById('lobbyStatus').textContent='🏁 Игра скоро будет доступна в этой игре!';
      break;
    case 'peer_left': peerHere=false; versusStarted=false;
      document.getElementById('lobbyStatus').textContent='😞 Соперник вышел';
      haptic('medium'); break;
    case 'error': document.getElementById('netStatus').textContent='⚠️ '+m.msg; break;
    case 'connected': case 'left': break;
    default: onGameMsg(m); // игровые события
  }
}

/* ================= Движок ================= */
let raf=null;
function stopLoops(){ if(raf) cancelAnimationFrame(raf); raf=null; }

/* ---------- Машинки (онлайн-гонка) ---------- */
function initCars(){
  stopLoops();
  const d=document.getElementById('demo'); d.style.display='block';
  d.innerHTML=`<div class="vsbar"><span>Я: <b id="myS">0</b></span><span>🏎️ ГОНКА 🏎️</span><span>Соперник: <b id="opS">0</b></span></div>
    <canvas id="cv" width="320" height="400"></canvas><div class="stat" id="cstat">👆 Тап — сменить полосу</div>`;
  const cv=document.getElementById('cv'), ctx=cv.getContext('2d');
  let lane=1, cars=[], score=0, speed=2.2, over=false, finished=false;
  let spawnT=0;
  function finish(win, reason){
    if (finished) return; finished=true;
    document.getElementById('cstat').textContent = win===null ? '🤝 Ничья! '+reason : (win?'🏆 ПОБЕДА! ':'😔 Поражение. ')+reason;
    haptic(win?'heavy':'medium');
  }
  function loop(){
    if(over) return;
    ctx.fillStyle='#0e1621'; ctx.fillRect(0,0,320,400);
    ctx.strokeStyle='#3a4a5c'; ctx.setLineDash([18,18]); ctx.lineWidth=4;
    for(let i=1;i<3;i++){ ctx.beginPath(); ctx.moveTo(i*106,0); ctx.lineTo(i*106,400); ctx.stroke(); }
    ctx.setLineDash([]);
    spawnT++; if(spawnT>Math.max(35,90-speed*10)){ cars.push({lane:Math.floor(Math.random()*3),y:-40,c:['#e53935','#fdd835','#8e24aa'][Math.floor(Math.random()*3)]}); spawnT=0; }
    const px=lane*106+53;
    cars.forEach(c=>{ c.y+=speed; ctx.fillStyle=c.c; ctx.beginPath(); ctx.roundRect(c.lane*106+18,c.y,70,36,8); ctx.fill(); });
    cars=cars.filter(c=>c.y<440);
    ctx.fillStyle='#5288c1'; ctx.beginPath(); ctx.roundRect(px-35,360,70,36,8); ctx.fill();
    ctx.font='20px serif'; ctx.fillText('🏎️',px-14,387);
    if(cars.some(c=>c.lane===lane && c.y+36>360 && c.y<396)){
      over=true; haptic('heavy');
      sendW({t:'crash', score});
      if (versus && versusStarted) finish(false,'Разбился на '+score+'. Ждём соперника…');
      else { document.getElementById('cstat').textContent='💥 Авария! Счёт: '+score+'. Тапни для рестарта.'; cv.onclick=()=>{haptic('light');initCars();}; }
      return;
    }
    score++; speed+=0.002;
    document.getElementById('myS').textContent=score;
    raf=requestAnimationFrame(loop);
  }
  cv.onclick=()=>{ if(!over){ lane=(lane+1)%3; haptic('light'); } };
  window._carsMsg = m => {
    if (m.t==='crash'){ document.getElementById('opS').textContent=m.score;
      if (versus && !over) finish(true,'Соперник разбился на '+m.score+' — ты едешь дальше!'); }
  };
  loop();
}

/* ---------- Баскетбол (по очереди) ---------- */
function initBasket(){
  stopLoops();
  const d=document.getElementById('demo'); d.style.display='block';
  d.innerHTML=`<div class="vsbar"><span>Я: <b id="myP">0</b>/5</span><span>🏀</span><span>Соперник: <b id="opP">0</b>/5</span></div>
    <div class="turn" id="turnInfo"></div>
    <canvas id="cv" width="320" height="380"></canvas>
    <div class="row"><span>🎯 Угол</span><input type="range" id="ang" min="-60" max="60" value="20"></div>
    <div class="row"><span>💪 Сила</span><input type="range" id="pow" min="30" max="100" value="65"></div>
    <button class="btn green" id="shoot">Бросить!</button>
    <div class="stat" id="bstat"></div>`;
  const cv=document.getElementById('cv'), ctx=cv.getContext('2d');
  let pts=0, opPts=0, throws=0, opThrows=0, ball=null, myTurn=true, done=false;
  function turnText(){
    const el=document.getElementById('turnInfo');
    if (versus){ el.textContent = done ? '' : (myTurn?'🟢 Твой бросок':'⏳ Бросок соперника…'); }
    else el.textContent='Соло-режим: набери как можно больше!';
    document.getElementById('shoot').disabled = versus && (!myTurn || done);
  }
  function endGame(){
    done=true; turnText();
    const res = pts===opPts ? '🤝 Ничья: '+pts+':'+opPts : (pts>opPts?'🏆 ПОБЕДА ':'😔 Поражение ')+pts+':'+opPts;
    document.getElementById('bstat').textContent=res; haptic(pts>=opPts?'heavy':'medium');
  }
  function draw(){
    ctx.fillStyle='#0e1621'; ctx.fillRect(0,0,320,380);
    ctx.fillStyle='#5d4037'; ctx.fillRect(120,60,80,10);
    ctx.strokeStyle='#ff9800'; ctx.lineWidth=3;
    ctx.beginPath(); ctx.ellipse(160,90,26,7,0,0,Math.PI*2); ctx.stroke();
    if(ball){
      ball.t+=0.016; const t=ball.t,a=ball.a*Math.PI/180,v=ball.v;
      ball.x=160+Math.cos(a)*v*t*60; ball.y=340-Math.sin(a)*v*t*60+0.5*9.8*t*t*60;
      if(ball.y<98 && ball.y>76 && Math.abs(ball.x-160)<22){
        pts++; throws++; haptic('heavy');
        document.getElementById('myP').textContent=pts;
        document.getElementById('bstat').textContent='🏀 Попал! Очки: '+pts;
        if (versus){ sendW({t:'shot', hit:true}); afterThrow(); }
        ball=null; return;
      }
      if(ball.y>390){
        throws++; haptic('medium'); document.getElementById('bstat').textContent='Мимо. Очки: '+pts;
        if (versus){ sendW({t:'shot', hit:false}); afterThrow(); }
        ball=null; return;
      }
      ctx.fillStyle='#ff7043'; ctx.beginPath(); ctx.arc(ball.x,ball.y,10,0,Math.PI*2); ctx.fill();
    } else { ctx.fillStyle='#ff7043'; ctx.beginPath(); ctx.arc(160,340,10,0,Math.PI*2); ctx.fill(); }
    raf=requestAnimationFrame(draw);
  }
  function afterThrow(){
    if (throws>=5 && opThrows>=5) endGame();
    else { myTurn=false; turnText(); }
  }
  document.getElementById('shoot').onclick=()=>{
    if (versus && (!myTurn || done)) return;
    haptic('medium');
    ball={x:160,y:340,t:0,a:parseFloat(document.getElementById('ang').value),v:parseFloat(document.getElementById('pow').value)/14};
  };
  window._basketMsg = m => {
    if (m.t==='shot'){ opThrows++; if(m.hit){ opPts++; document.getElementById('opP').textContent=opPts; document.getElementById('bstat').textContent='Соперник попал! '+pts+':'+opPts; }
      else document.getElementById('bstat').textContent='Соперник промахнулся. '+pts+':'+opPts;
      if (throws>=5 && opThrows>=5) endGame(); else { myTurn=true; turnText(); } }
  };
  turnText(); draw();
}

function onGameMsg(m){
  if (current==='cars' && window._carsMsg) window._carsMsg(m);
  if (current==='basket' && window._basketMsg) window._basketMsg(m);
}
</script>
</body>
</html>

// GameOn сервер: статика + WebSocket-лобби (Node.js, без внешних зависимостей)
const http = require('http');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const PORT = process.env.PORT || 3000;
const server = http.createServer((req, res) => {
  const file = path.join(__dirname, 'index.html');
  fs.readFile(file, (err, data) => {
    if (err) { res.writeHead(500); res.end('error'); return; }
    res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
    res.end(data);
  });
});

// ---- минимальный WebSocket (RFC6455) ----
const GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
function wsAccept(key){ return crypto.createHash('sha1').update(key+GUID).digest('base64'); }
function encodeFrame(str){
  const payload = Buffer.from(str), len = payload.length;
  let header;
  if (len < 126) { header = Buffer.from([0x81, len]); }
  else if (len < 65536) { header = Buffer.alloc(4); header[0]=0x81; header[1]=126; header.writeUInt16BE(len,2); }
  else { header = Buffer.alloc(10); header[0]=0x81; header[1]=127; header.writeBigUInt64BE(BigInt(len),2); }
  return Buffer.concat([header, payload]);
}
function decodeFrames(buf){
  const frames=[]; let off=0;
  while (off+2 <= buf.length) {
    const opcode = buf[off] & 0x0f;
    let len = buf[off+1] & 0x7f, hlen=2;
    if (len===126){ if(off+4>buf.length) break; len=buf.readUInt16BE(off+2); hlen=4; }
    else if (len===127){ if(off+10>buf.length) break; len=Number(buf.readBigUInt64BE(off+2)); hlen=10; }
    if (off+hlen+len > buf.length) break;
    frames.push({opcode, payload: buf.slice(off+hlen, off+hlen+len)});
    off += hlen+len;
  }
  return frames;
}

const rooms = new Map(); // code -> {players:[sockA,sockB], meta:{}, started:bool}
let waiting = null; // очередь быстрого матча

function send(sock, obj){ try{ sock.write(encodeFrame(JSON.stringify(obj))); }catch(e){} }
function broadcast(room, obj, except){ room.players.forEach(p=>{ if(p && p!==except) send(p, obj); }); }
function roomOf(sock){ return sock._room ? rooms.get(sock._room) : null; }

server.on('upgrade', (req, sock) => {
  const key = req.headers['sec-websocket-key'];
  if (!key) { sock.destroy(); return; }
  sock.write('HTTP/1.1 101 Switching Protocols\r\nUpgrade: websocket\r\nConnection: Upgrade\r\n'+
             'Sec-WebSocket-Accept: '+wsAccept(key)+'\r\n\r\n');
  sock.setNoDelay(true);
  send(sock, {t:'connected'});

  let buf = Buffer.alloc(0);
  sock.on('data', d => {
    buf = Buffer.concat([buf, d]);
    decodeFrames(buf).forEach(fr => {
      if (fr.opcode === 8) { cleanup(sock); sock.end(); return; }
      if (fr.opcode !== 1) return;
      buf = Buffer.alloc(0);
      let m; try { m = JSON.parse(fr.payload.toString()); } catch(e){ return; }
      handle(sock, m);
    });
  });
  sock.on('close', ()=>cleanup(sock));
  sock.on('error', ()=>cleanup(sock));
});

function leaveRoom(sock){
  const code = sock._room; if (!code) return;
  const room = rooms.get(code);
  if (room){
    room.players = room.players.filter(p=>p!==sock);
    broadcast(room, {t:'peer_left'});
    if (room.players.length===0) rooms.delete(code);
  }
  sock._room = null;
}
function cleanup(sock){ if (sock._room) leaveRoom(sock); if (waiting===sock) waiting=null; }

function handle(sock, m){
  const me = () => {
    const r = roomOf(sock);
    return r ? {room:r, idx:r.players.indexOf(sock)} : null;
  };
  switch(m.t){
    case 'create': {
      leaveRoom(sock);
      const code = crypto.randomBytes(3).toString('hex').toUpperCase().slice(0,5);
      rooms.set(code, {players:[sock], started:false});
      sock._room = code;
      send(sock, {t:'created', code});
      break;
    }
    case 'join': {
      leaveRoom(sock);
      const code = String(m.code||'').toUpperCase();
      const room = rooms.get(code);
      if (!room) { send(sock, {t:'error', msg:'Комната не найдена'}); break; }
      if (room.players.length >= 2) { send(sock, {t:'error', msg:'Комната заполнена'}); break; }
      room.players.push(sock); sock._room = code;
      send(sock, {t:'joined', code});
      broadcast(room, {t:'peer_joined'});
      break;
    }
    case 'leave': { leaveRoom(sock); send(sock, {t:'left'}); break; }
    case 'quick': { // быстрый матч
      leaveRoom(sock);
      if (waiting && waiting !== sock && waiting._room==null) {
        const code = crypto.randomBytes(3).toString('hex').toUpperCase().slice(0,5);
        rooms.set(code, {players:[waiting, sock], started:false});
        waiting._room = code; sock._room = code; waiting = null;
        send(sock, {t:'joined', code}); broadcast(rooms.get(code), {t:'peer_joined'});
      } else { waiting = sock; send(sock, {t:'waiting'}); }
      break;
    }
    case 'start': { // хост жмёт «начать», обоим уходит сигнал старта
      const ctx = me(); if (!ctx) break;
      ctx.room.started = true;
      broadcast(ctx.room, {t:'start_game', host: ctx.idx===0?1:0});
      break;
    }
    default: { // всё остальное — релей врагу: msg, state, score, result...
      const ctx = me(); if (!ctx) break;
      m._from = ctx.idx;
      broadcast(ctx.room, m, sock);
    }
  }
}

server.listen(PORT, ()=>console.log('GameOn на порту '+PORT));
