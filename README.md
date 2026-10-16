<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Login</title>
<style>
:root{
  --bg:#0a0a0c;
  --amber:255,186,102;
  --text:#e9e6e1;
  --font:"Helvetica Neue","Segoe UI",system-ui,-apple-system,Arial,sans-serif;
}
*{box-sizing:border-box;margin:0;padding:0}
html,body{min-height:100%;background:var(--bg);color:var(--text);font-family:var(--font)}
body{overflow-x:hidden}

/* Wall texture: fine noise + faint plaster mottling */
body::before{
  content:"";position:fixed;inset:0;pointer-events:none;z-index:0;
  background:
    url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='300'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='3' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 1  0 0 0 0 1  0 0 0 0 1  0 0 0 .55 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>"),
    url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='600' height='600'><filter id='m'><feTurbulence type='fractalNoise' baseFrequency='.008' numOctaves='3'/><feColorMatrix values='0 0 0 0 1  0 0 0 0 1  0 0 0 0 1  0 0 0 .9 0'/></filter><rect width='100%' height='100%' filter='url(%23m)'/></svg>");
  opacity:.055;mix-blend-mode:screen;
}
/* Vignette */
body::after{
  content:"";position:fixed;inset:0;pointer-events:none;z-index:5;
  background:radial-gradient(ellipse at 50% 30%,transparent 35%,rgba(0,0,0,.65) 100%);
}

/* Room light spreading over the wall */
.wash{position:fixed;inset:0;z-index:1;pointer-events:none;opacity:0;transition:opacity 1.6s ease-out;
  background:
    radial-gradient(ellipse 38% 30% at 50% 9%,rgba(var(--amber),.34),transparent 70%),
    radial-gradient(ellipse 75% 70% at 50% 10%,rgba(var(--amber),.13),transparent 75%);}
.flash{position:fixed;inset:0;z-index:6;pointer-events:none;opacity:0;
  background:radial-gradient(circle at 50% 9%,rgba(255,240,215,.95),rgba(var(--amber),.35) 30%,transparent 62%);}
.flash.go{animation:flash .7s ease-out}
@keyframes flash{0%{opacity:0}12%{opacity:1}100%{opacity:0}}
body.lit .wash{opacity:1}

.stage{position:relative;z-index:2;min-height:100vh;display:flex;flex-direction:column;align-items:center;padding:clamp(14px,4vh,40px) 16px 48px}

/* Lamp */
.lamp{position:relative;width:140px;height:170px;border:0;background:none;cursor:pointer;flex:none;-webkit-tap-highlight-color:transparent}
.lamp:focus-visible{outline:2px solid rgba(var(--amber),.7);outline-offset:6px;border-radius:12px}
.lamp svg{width:100%;height:100%;overflow:visible;display:block}
.halo{opacity:0;transition:opacity .08s}
.glass{fill:#16161a;transition:fill .08s}
.fil{stroke:#3a2a1c;stroke-width:1.6;fill:none;transition:stroke .08s,filter .08s}
.lamp.on .halo{opacity:1}
.lamp.on .glass{fill:url(#bulbOn)}
.lamp.on .fil{stroke:#fff6e0;filter:drop-shadow(0 0 4px #ffd08a) drop-shadow(0 0 10px #ff9d3c)}
.shadeGlow{opacity:0;transition:opacity .4s}
.lamp.on .shadeGlow{opacity:1}

/* Beam */
.beam{position:absolute;left:50%;top:112px;width:min(560px,130vw);height:560px;transform:translateX(-50%);
  background:linear-gradient(to bottom,rgba(255,230,190,.55),rgba(var(--amber),.2) 35%,rgba(var(--amber),.05) 75%,transparent);
  clip-path:polygon(47% 0,53% 0,100% 100%,0 100%);filter:blur(7px);
  opacity:0;transition:opacity 1.2s ease-out;pointer-events:none;z-index:1;mix-blend-mode:screen}
body.lit .beam{opacity:1}

/* Click cue */
.cue{position:absolute;top:178px;left:50%;transform:translateX(-50%);display:flex;flex-direction:column;align-items:center;gap:8px;
  background:none;border:0;color:rgba(233,230,225,.5);font:300 13px var(--font);letter-spacing:.14em;cursor:pointer;transition:opacity .5s}
.cue svg{width:22px;height:22px;animation:bob 1.8s ease-in-out infinite}
.cue span{animation:fade 1.8s ease-in-out infinite}
@keyframes bob{0%,100%{transform:translateY(5px);opacity:.35}50%{transform:translateY(-3px);opacity:1}}
@keyframes fade{0%,100%{opacity:.4}50%{opacity:.9}}
body.lit .cue{opacity:0;pointer-events:none}

/* Glass card */
.card{position:relative;z-index:2;width:min(380px,100%);margin-top:70px;padding:38px 32px 32px;border-radius:26px;
  opacity:0;pointer-events:none;transform:translateY(18px) scale(.97);
  transition:opacity 1.3s ease-out .35s,transform 1.3s cubic-bezier(.2,.8,.2,1) .35s;
  background:linear-gradient(155deg,rgba(255,255,255,.11),rgba(255,255,255,.03) 45%,rgba(20,20,26,.35));
  -webkit-backdrop-filter:blur(22px) saturate(1.3);backdrop-filter:blur(22px) saturate(1.3);
  border:1px solid rgba(255,255,255,.1);
  box-shadow:
    inset 0 1px 0 rgba(255,255,255,.28),
    inset 0 0 0 1px rgba(255,255,255,.05),
    inset 0 -18px 30px rgba(0,0,0,.35),
    0 30px 60px -10px rgba(0,0,0,.8),
    0 0 80px -20px rgba(var(--amber),.25)}
body.lit .card{opacity:1;pointer-events:auto;transform:none}
.card::before{content:"";position:absolute;inset:0;border-radius:inherit;pointer-events:none;
  background:linear-gradient(180deg,rgba(255,225,180,.16),transparent 38%)}

h1{position:relative;text-align:center;font-size:44px;font-weight:700;letter-spacing:.02em;margin-bottom:28px;
  background:linear-gradient(180deg,#fff 0%,#cfcac2 38%,#7d7a76 52%,#d9d4cc 70%,#f6f2ea 100%);
  -webkit-background-clip:text;background-clip:text;color:transparent;-webkit-text-fill-color:transparent;
  filter:drop-shadow(0 -1px 0 rgba(255,255,255,.35)) drop-shadow(0 2px 1px rgba(0,0,0,.9)) drop-shadow(0 6px 10px rgba(0,0,0,.55))}

.field{position:relative;margin-bottom:16px}
.field svg{position:absolute;left:16px;top:50%;width:19px;height:19px;transform:translateY(-50%);stroke:rgba(233,230,225,.55);fill:none;stroke-width:1.7;stroke-linecap:round;stroke-linejoin:round;transition:stroke .25s}
.field input{width:100%;height:52px;padding:0 16px 0 46px;border-radius:14px;color:var(--text);font:400 15px var(--font);outline:none;
  background:linear-gradient(180deg,rgba(0,0,0,.42),rgba(0,0,0,.22));
  border:1px solid rgba(255,255,255,.08);border-bottom-color:rgba(255,255,255,.16);
  box-shadow:inset 0 3px 8px rgba(0,0,0,.65),inset 0 -1px 0 rgba(255,255,255,.05),0 1px 0 rgba(255,255,255,.07);
  transition:border-color .25s,box-shadow .25s}
.field input::placeholder{color:rgba(233,230,225,.38)}
.field input:focus{border-color:rgba(var(--amber),.55);box-shadow:inset 0 3px 8px rgba(0,0,0,.65),0 0 0 3px rgba(var(--amber),.14)}
.field:focus-within svg{stroke:rgb(var(--amber))}

.btn{display:block;width:100%;height:54px;margin-top:24px;border:0;border-radius:999px;cursor:pointer;
  font:600 16px var(--font);letter-spacing:.06em;color:#241707;text-shadow:0 1px 0 rgba(255,255,255,.45);
  background:linear-gradient(180deg,#ffe3b3 0%,#f3b45f 52%,#d98f34 100%);
  box-shadow:inset 0 1px 0 rgba(255,255,255,.75),inset 0 -3px 6px rgba(120,60,0,.35),0 2px 0 #9a5f1c,0 10px 18px rgba(0,0,0,.55),0 0 24px rgba(var(--amber),.2);
  transition:transform .15s,box-shadow .15s,filter .2s}
.btn:hover{filter:brightness(1.08) saturate(1.05);transform:translateY(-2px);
  box-shadow:inset 0 1px 0 rgba(255,255,255,.85),inset 0 -3px 6px rgba(120,60,0,.35),0 4px 0 #9a5f1c,0 16px 26px rgba(0,0,0,.6),0 0 34px rgba(var(--amber),.35)}
.btn:active{transform:translateY(2px);box-shadow:inset 0 2px 6px rgba(120,60,0,.5),0 0 0 #9a5f1c,0 3px 8px rgba(0,0,0,.5)}
.btn:focus-visible{outline:2px solid #fff;outline-offset:3px}
.msg{min-height:20px;margin-top:14px;text-align:center;font-size:13px;color:rgba(233,230,225,.7)}

@media (prefers-reduced-motion:reduce){
  .cue svg,.cue span{animation:none}.flash.go{animation:none}
  .card,.wash,.beam{transition-duration:.01s;transition-delay:0s}
}
</style>
</head>
<body>
<div class="wash"></div>
<div class="flash" id="flash"></div>

<main class="stage">
  <button class="lamp" id="lamp" aria-label="Turn on the light" aria-pressed="false">
    <div class="beam"></div>
    <svg viewBox="0 0 140 170" aria-hidden="true">
      <defs>
        <linearGradient id="metal" x1="0" x2="1">
          <stop offset="0" stop-color="#0d0d10"/><stop offset=".28" stop-color="#3b3b42"/>
          <stop offset=".5" stop-color="#5a5a63"/><stop offset=".72" stop-color="#2a2a30"/><stop offset="1" stop-color="#0b0b0e"/>
        </linearGradient>
        <linearGradient id="brass" x1="0" x2="1">
          <stop offset="0" stop-color="#5b4425"/><stop offset=".45" stop-color="#c9a263"/><stop offset="1" stop-color="#4a361c"/>
        </linearGradient>
        <radialGradient id="bulbOn" cx=".5" cy=".5" r=".5">
          <stop offset="0" stop-color="#fffdf2"/><stop offset=".55" stop-color="#ffe2a8"/><stop offset="1" stop-color="#ff9d3c"/>
        </radialGradient>
        <radialGradient id="haloG" cx=".5" cy=".5" r=".5">
          <stop offset="0" stop-color="#fff0cf" stop-opacity=".95"/><stop offset=".35" stop-color="#ffb866" stop-opacity=".5"/><stop offset="1" stop-color="#ff9d3c" stop-opacity="0"/>
        </radialGradient>
        <radialGradient id="shadeInner" cx=".5" cy="1" r=".8">
          <stop offset="0" stop-color="#ffcf8a" stop-opacity=".7"/><stop offset="1" stop-color="#ffcf8a" stop-opacity="0"/>
        </radialGradient>
        <linearGradient id="plate" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0" stop-color="#4a4a52"/><stop offset="1" stop-color="#16161a"/>
        </linearGradient>
      </defs>
      <rect x="38" y="-6" width="64" height="20" rx="4" fill="url(#plate)" stroke="#000" stroke-opacity=".6"/>
      <rect x="64" y="14" width="12" height="26" fill="url(#brass)"/>
      <path d="M24 100 Q24 40 70 40 Q116 40 116 100 Z" fill="url(#metal)"/>
      <path d="M24 100 Q24 40 70 40 Q116 40 116 100 Z" fill="url(#shadeInner)" class="shadeGlow"/>
      <path d="M44 92 Q46 56 66 47" stroke="#fff" stroke-opacity=".22" stroke-width="3" fill="none" stroke-linecap="round"/>
      <rect x="20" y="98" width="100" height="7" rx="3.5" fill="url(#brass)"/>
      <circle class="halo" cx="70" cy="114" r="52" fill="url(#haloG)"/>
      <circle class="glass" cx="70" cy="114" r="16" stroke="#fff" stroke-opacity=".18"/>
      <path class="fil" d="M64 124 L64 112 L67 106 L70 112 L73 106 L76 112 L76 124"/>
      <ellipse cx="63" cy="107" rx="4" ry="2.5" fill="#fff" fill-opacity=".25" transform="rotate(-35 63 107)"/>
    </svg>
    <span class="cue" id="cue" role="presentation">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M12 19V5M5 11l7-7 7 7"/></svg>
      <span>Click to Light</span>
    </span>
  </button>

  <section class="card" aria-label="Login">
    <h1>Login</h1>
    <form id="form" novalidate>
      <label class="field">
        <svg viewBox="0 0 24 24"><circle cx="12" cy="8" r="4"/><path d="M4 21c0-4.4 3.6-7 8-7s8 2.6 8 7"/></svg>
        <input type="text" id="user" placeholder="Username" autocomplete="username" aria-label="Username">
      </label>
      <label class="field">
        <svg viewBox="0 0 24 24"><rect x="4" y="11" width="16" height="10" rx="2.5"/><path d="M8 11V8a4 4 0 0 1 8 0v3"/></svg>
        <input type="password" id="pass" placeholder="Password" autocomplete="current-password" aria-label="Password">
      </label>
      <button class="btn" type="submit">Login</button>
      <p class="msg" id="msg" role="status"></p>
    </form>
  </section>
</main>

<script>
(function(){
  var lamp=document.getElementById('lamp'), flash=document.getElementById('flash'),
      msg=document.getElementById('msg'), timers=[];
  var reduce=matchMedia('(prefers-reduced-motion: reduce)').matches;
  function later(fn,ms){timers.push(setTimeout(fn,ms));}
  function bulb(on){lamp.classList.toggle('on',on);}

  // Sounds: synthesized with Web Audio, so no audio files are needed
  var actx;
  function audio(){
    if(!actx){var AC=window.AudioContext||window.webkitAudioContext;if(!AC)return null;actx=new AC();}
    if(actx.state==='suspended')actx.resume();
    return actx;
  }
  function click(vol){ // switch "clack"
    var a=audio();if(!a)return;var t=a.currentTime;
    var len=Math.floor(a.sampleRate*.04),buf=a.createBuffer(1,len,a.sampleRate),d=buf.getChannelData(0);
    for(var i=0;i<len;i++)d[i]=(Math.random()*2-1)*Math.pow(1-i/len,3);
    var n=a.createBufferSource();n.buffer=buf;
    var bp=a.createBiquadFilter();bp.type='bandpass';bp.frequency.value=2800;bp.Q.value=.8;
    var g=a.createGain();g.gain.value=vol;
    n.connect(bp);bp.connect(g);g.connect(a.destination);n.start(t);
    var o=a.createOscillator(),og=a.createGain();
    o.type='sine';o.frequency.setValueAtTime(220,t);o.frequency.exponentialRampToValueAtTime(70,t+.06);
    og.gain.setValueAtTime(vol*.8,t);og.gain.exponentialRampToValueAtTime(.001,t+.09);
    o.connect(og);og.connect(a.destination);o.start(t);o.stop(t+.1);
  }
  function buzz(){ // bulb strike: soft electric hum fading out, plus a glass ping
    var a=audio();if(!a)return;var t=a.currentTime;
    var o=a.createOscillator(),lp=a.createBiquadFilter(),g=a.createGain();
    o.type='sawtooth';o.frequency.value=120;lp.type='lowpass';lp.frequency.value=700;
    g.gain.setValueAtTime(.0001,t);g.gain.exponentialRampToValueAtTime(.05,t+.03);
    g.gain.exponentialRampToValueAtTime(.0001,t+1.5);
    o.connect(lp);lp.connect(g);g.connect(a.destination);o.start(t);o.stop(t+1.6);
    var p=a.createOscillator(),pg=a.createGain();
    p.type='sine';p.frequency.value=2400;
    pg.gain.setValueAtTime(.0001,t);pg.gain.exponentialRampToValueAtTime(.05,t+.01);
    pg.gain.exponentialRampToValueAtTime(.0001,t+.6);
    p.connect(pg);pg.connect(a.destination);p.start(t);p.stop(t+.7);
  }

  function turnOn(){
    timers.forEach(clearTimeout);timers=[];
    lamp.setAttribute('aria-pressed','true');
    lamp.setAttribute('aria-label','Turn off the light');
    click(.5);
    if(reduce){bulb(true);buzz();document.body.classList.add('lit');return;}
    // Filament strike: quick flicker, then a hard flash and full light
    bulb(true);later(function(){bulb(false)},90);
    later(function(){bulb(true);click(.12)},170);later(function(){bulb(false)},240);
    later(function(){
      bulb(true);buzz();
      flash.classList.remove('go');void flash.offsetWidth;flash.classList.add('go');
      document.body.classList.add('lit');
    },330);
  }
  function turnOff(){
    timers.forEach(clearTimeout);timers=[];
    lamp.setAttribute('aria-pressed','false');
    lamp.setAttribute('aria-label','Turn on the light');
    click(.4);
    bulb(false);document.body.classList.remove('lit');msg.textContent='';
  }
  lamp.addEventListener('click',function(){
    document.body.classList.contains('lit')?turnOff():turnOn();
  });

  document.getElementById('form').addEventListener('submit',function(e){
    e.preventDefault();
    var u=document.getElementById('user').value.trim(), p=document.getElementById('pass').value;
    msg.textContent=(!u||!p)?'Enter your username and password.':'Welcome back, '+u+'. (Demo only: no server attached.)';
  });
})();
</script>
