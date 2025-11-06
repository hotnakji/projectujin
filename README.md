<!DOCTYPE html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Beat Clicker+</title>
<style>
body {
  margin: 0;
  overflow: hidden;
  background: #111;
  color: #fff;
  font-family: sans-serif;
  text-align: center;
}
canvas {
  display: none;
  width: 100vw;
  height: 100vh;
  background: #222;
}
#ui {
  position: fixed;
  top: 10px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-between;
  padding: 0 10px;
  font-size: 18px;
}
#center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
button {
  background: #ff4081;
  color: #fff;
  border: 0;
  padding: 10px 20px;
  border-radius: 10px;
  font-size: 18px;
  cursor: pointer;
  margin: 5px;
}
button:hover { background: #f50057; }
</style>
<body>
<div id="center">
  <h1>🎵 Beat Clicker+</h1>
  <p>Choose your level and press SPACE or CLICK when notes hit the pink line!</p>
  <button data-level="easy">Easy</button>
  <button data-level="normal">Normal</button>
  <button data-level="hard">Hard</button>
  <button data-level="insane">Insane</button>
</div>

<canvas id="c"></canvas>
<div id="ui" style="display:none">
  <span id="info">Score:0 | Time:0s</span>
  <span id="mute" style="cursor:pointer">🔊</span>
</div>

<script>
const c = document.getElementById("c"), x = c.getContext("2d");
c.width = innerWidth; c.height = innerHeight;

let notes = [], particles = [], score = 0, high = +localStorage.hs || 0,
    speed = 2, lives = 3, play = 0, mute = 0, last = 0, startTime = 0,
    level = "normal", spawnRate = 1000;

const hitS = new Audio("https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"),
      missS = new Audio("https://actions.google.com/sounds/v1/cartoon/slide_whistle_to_drum_hit.ogg"),
      deathS = new Audio("https://actions.google.com/sounds/v1/cartoon/concussive_drum_hit.ogg");

const bgmList = {
  easy:  "https://files.catbox.moe/xl2r9f.mp3",
  normal:"https://files.catbox.moe/2a9d1p.mp3",
  hard:  "https://files.catbox.moe/olsh4y.mp3",
  insane:"https://files.catbox.moe/vqz6o8.mp3"
};
let bgm = new Audio();
bgm.loop = true; bgm.volume = 0.4;

function snd(s){ if(!mute){ s.currentTime=0; s.play(); } }
function toggleMute(){
  mute ^= 1;
  document.getElementById("mute").textContent = mute ? '🔇' : '🔊';
  if(mute) bgm.pause(); else if(play) bgm.play();
}

function spawn(){ notes.push({x:Math.random()*(c.width-40)+20,y:-20}); }

function makeParticles(xp, yp){
  for(let i=0;i<10;i++)
    particles.push({x:xp,y:yp,vx:(Math.random()-0.5)*4,vy:(Math.random()-1)*-4,life:30});
}

function drawParticles(){
  for(let p of particles){
    x.fillStyle=`hsl(${Math.random()*360},100%,70%)`;
    x.beginPath();
    x.arc(p.x,p.y,4,0,7);
    x.fill();
    p.x+=p.vx; p.y+=p.vy; p.vy+=0.2; p.life--;
  }
  particles = particles.filter(p=>p.life>0);
}

function draw(){
  x.clearRect(0,0,c.width,c.height);
  // background pulse
  x.fillStyle = "#222";
  x.fillRect(0,0,c.width,c.height);
  
  x.fillStyle="#f48"; x.fillRect(0,c.height-100,c.width,5);
  for(let n of notes){
    x.fillStyle="#0ff";
    x.beginPath(); x.arc(n.x,n.y,20,0,7); x.fill();
  }
  drawParticles();
  let t=Math.floor((performance.now()-startTime)/1000);
  document.getElementById("info").textContent = 
    `Score:${score} | High:${high} | Lives:${lives} | Time:${t}s | Level:${level.toUpperCase()}`;
}

function update(){
  for(let n of notes) n.y+=speed;
  notes=notes.filter(n=>n.y<c.height+20);
}

function hitCheck(){
  for(let n of notes){
    if(Math.abs(n.y-(c.height-100))<25 && !n.hit){
      n.hit=1; score++; snd(hitS);
      makeParticles(n.x, c.height-100);
      return;
    }
  }
  lives--; snd(missS);
  if(lives<=0) end();
}

function loop(t){
  if(!play) return;
  let elapsed=(t-startTime)/1000;
  update(); draw();
  if(!last || t-last>spawnRate) { spawn(); last=t; }
  requestAnimationFrame(loop);
}

function startGame(lv){
  level = lv;
  const settings = {
    easy:{speed:2,spawn:1000},
    normal:{speed:3,spawn:800},
    hard:{speed:4.5,spawn:650},
    insane:{speed:6,spawn:500}
  }[lv];
  speed=settings.speed; spawnRate=settings.spawn;

  document.getElementById("center").style.display="none";
  c.style.display="block";
  document.getElementById("ui").style.display="flex";
  score=0; lives=3; notes=[]; particles=[]; play=1; last=0;
  startTime=performance.now();

  bgm.src = bgmList[lv];
  if(!mute) bgm.play();
  loop(startTime);
}

function end(){
  play=0; bgm.pause(); snd(deathS);
  let time=Math.floor((performance.now()-startTime)/1000);
  c.style.display="none";
  document.getElementById("ui").style.display="none";
  if(score>high) localStorage.hs=high=score;
  document.getElementById("center").innerHTML=
    `<h1>💥 Game Over</h1>
     <p>Score: <b>${score}</b> | High: ${high}</p>
     <p>You survived for <b>${time}</b> seconds!</p>
     <button data-level="easy">Easy</button>
     <button data-level="normal">Normal</button>
     <button data-level="hard">Hard</button>
     <button data-level="insane">Insane</button>`;
  document.getElementById("center").style.display="block";
  setupMenu();
}

function setupMenu(){
  document.querySelectorAll("[data-level]").forEach(btn=>{
    btn.onclick=()=>startGame(btn.dataset.level);
  });
}

window.onresize=()=>{c.width=innerWidth;c.height=innerHeight};
document.body.onkeydown=e=>e.code=="Space"&&play&&hitCheck();
document.body.onclick=()=>play&&hitCheck();
document.getElementById("mute").onclick=toggleMute;

setupMenu();
</script>
</body>
</html>
