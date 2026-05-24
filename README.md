<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MnemOS</title>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  html, body {
    height: 100%;
    background: #000;
    font-family: 'Courier New', Courier, monospace;
    color: #fff;
    overflow-x: hidden;
  }

  nav {
    position: fixed;
    top: 0; left: 0; right: 0;
    z-index: 100;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 56px;
    height: 60px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
    background: #000;
  }

  .nav-logo {
    font-size: 13px;
    color: rgba(255,255,255,0.5);
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }

  .nav-links {
    display: flex;
    gap: 36px;
    list-style: none;
  }

  .nav-links a {
    font-size: 12px;
    color: rgba(255,255,255,0.35);
    text-decoration: none;
    letter-spacing: 0.1em;
    transition: color 0.2s;
  }
  .nav-links a:hover { color: #fff; }

  .nav-btn {
    font-family: 'Courier New', monospace;
    font-size: 12px;
    color: #000;
    background: #fff;
    border: none;
    padding: 9px 22px;
    cursor: pointer;
    letter-spacing: 0.1em;
    transition: opacity 0.2s;
  }
  .nav-btn:hover { opacity: 0.82; }

  hero {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    padding: 60px 48px 0;
    text-align: center;
    position: relative;
    overflow: hidden;
  }

  .grid-bg {
    position: absolute;
    inset: 0;
    background-image:
      linear-gradient(rgba(255,255,255,0.025) 1px, transparent 1px),
      linear-gradient(90deg, rgba(255,255,255,0.025) 1px, transparent 1px);
    background-size: 48px 48px;
    pointer-events: none;
  }

  .badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    font-size: 11px;
    color: rgba(255,255,255,0.4);
    border: 1px solid rgba(255,255,255,0.1);
    padding: 5px 16px;
    letter-spacing: 0.12em;
    margin-bottom: 36px;
    position: relative;
    z-index: 1;
  }

  .badge-dot {
    width: 5px; height: 5px;
    background: #fff;
    flex-shrink: 0;
    animation: blink 1.2s step-end infinite;
  }

  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }

  canvas#hero-logo {
    display: block;
    width: 100%;
    height: auto;
    image-rendering: pixelated;
    position: relative;
    z-index: 1;
    margin-bottom: 36px;
  }

  .hero-sub {
    font-size: 14px;
    color: rgba(255,255,255,0.35);
    letter-spacing: 0.07em;
    line-height: 2;
    max-width: 520px;
    margin: 0 auto 40px;
    position: relative;
    z-index: 1;
  }

  .cta-row {
    display: flex;
    justify-content: center;
    gap: 12px;
    margin-bottom: 36px;
    flex-wrap: wrap;
    position: relative;
    z-index: 1;
  }

  .btn-primary {
    font-family: 'Courier New', monospace;
    font-size: 12px;
    letter-spacing: 0.12em;
    color: #000;
    background: #fff;
    border: 1px solid #fff;
    padding: 13px 32px;
    cursor: pointer;
    transition: opacity 0.2s;
  }
  .btn-primary:hover { opacity: 0.85; }

  .btn-secondary {
    font-family: 'Courier New', monospace;
    font-size: 12px;
    letter-spacing: 0.12em;
    color: rgba(255,255,255,0.55);
    background: transparent;
    border: 1px solid rgba(255,255,255,0.16);
    padding: 13px 32px;
    cursor: pointer;
    transition: border-color 0.2s, color 0.2s;
  }
  .btn-secondary:hover { border-color: rgba(255,255,255,0.5); color: #fff; }

  .pills {
    display: flex;
    justify-content: center;
    gap: 8px;
    flex-wrap: wrap;
    position: relative;
    z-index: 1;
    margin-bottom: 52px;
  }

  .pill {
    font-size: 10px;
    letter-spacing: 0.1em;
    color: rgba(255,255,255,0.28);
    border: 1px solid rgba(255,255,255,0.07);
    padding: 5px 14px;
  }

  .terminal-preview {
    width: 100%;
    max-width: 600px;
    border: 1px solid rgba(255,255,255,0.1);
    position: relative;
    z-index: 1;
    text-align: left;
    margin-bottom: 0;
  }

  .term-bar {
    background: rgba(255,255,255,0.05);
    padding: 9px 16px;
    display: flex;
    align-items: center;
    gap: 6px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
  }

  .term-dot { width: 8px; height: 8px; border: 1px solid rgba(255,255,255,0.18); }

  .term-title {
    font-size: 10px;
    color: rgba(255,255,255,0.22);
    letter-spacing: 0.12em;
    margin-left: 6px;
  }

  .term-body { padding: 20px 24px 24px; font-size: 12px; line-height: 2.1; }

  .term-line { display: flex; gap: 10px; }
  .term-prompt { color: rgba(255,255,255,0.18); }
  .term-cmd { color: rgba(255,255,255,0.65); }
  .term-out { color: rgba(255,255,255,0.28); padding-left: 20px; }
  .term-ok { color: rgba(255,255,255,0.4); margin-right: 6px; }
  .term-cursor {
    display: inline-block;
    width: 8px; height: 14px;
    background: #fff;
    vertical-align: middle;
    animation: blink 1s step-end infinite;
    margin-left: 2px;
  }

  .scroll-hint {
    position: relative;
    z-index: 1;
    margin-top: 32px;
    margin-bottom: 32px;
    font-size: 10px;
    letter-spacing: 0.18em;
    color: rgba(255,255,255,0.15);
    text-transform: uppercase;
  }

  .features {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    border-top: 1px solid rgba(255,255,255,0.06);
  }

  .feat {
    padding: 44px 36px;
    border-right: 1px solid rgba(255,255,255,0.06);
  }
  .feat:last-child { border-right: none; }

  .feat-tag {
    font-size: 9px;
    letter-spacing: 0.18em;
    color: rgba(255,255,255,0.2);
    text-transform: uppercase;
    margin-bottom: 14px;
  }

  .feat-title {
    font-size: 14px;
    color: rgba(255,255,255,0.75);
    margin-bottom: 10px;
    letter-spacing: 0.04em;
  }

  .feat-desc {
    font-size: 11px;
    color: rgba(255,255,255,0.26);
    line-height: 1.9;
    letter-spacing: 0.03em;
  }

  footer {
    padding: 24px 56px;
    border-top: 1px solid rgba(255,255,255,0.06);
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 10px;
  }

  .footer-txt {
    font-size: 10px;
    color: rgba(255,255,255,0.18);
    letter-spacing: 0.1em;
  }
</style>
</head>
<body>

<nav>
  <span class="nav-logo">MnemOS</span>
  <ul class="nav-links">
    <li><a href="#">docs</a></li>
    <li><a href="#">github</a></li>
    <li><a href="#">demo</a></li>
  </ul>
  <button class="nav-btn">get started</button>
</nav>

<hero>
  <div class="grid-bg"></div>

  <div class="badge">
    <div class="badge-dot"></div>
    hackathon build &mdash; agents under pressure
  </div>

  <canvas id="hero-logo"></canvas>

  <p class="hero-sub">
    drag. drop. deploy. build ai agents that browse the web,<br>
    extract data, make decisions &mdash; and remember everything.
  </p>

  <div class="cta-row">
    <button class="btn-primary">[ quick start ]</button>
    <button class="btn-secondary">[ view on github ]</button>
  </div>

  <div class="pills">
    <span class="pill">no code</span>
    <span class="pill">persistent memory</span>
    <span class="pill">auto-recover</span>
    <span class="pill">cron scheduler</span>
    <span class="pill">real browser</span>
    <span class="pill">HydraDB + Groq</span>
  </div>

  <div class="terminal-preview">
    <div class="term-bar">
      <div class="term-dot"></div>
      <div class="term-dot"></div>
      <div class="term-dot"></div>
      <span class="term-title">terminal</span>
    </div>
    <div class="term-body">
      <div class="term-line"><span class="term-prompt">$</span><span class="term-cmd">git clone https://github.com/yranjan06/MnemOS</span></div>
      <div class="term-line"><span class="term-prompt">$</span><span class="term-cmd">cp .env.example .env &amp;&amp; docker compose up --build</span></div>
      <div class="term-out"><span class="term-ok">&gt;&gt;</span> memory core initialized</div>
      <div class="term-out"><span class="term-ok">&gt;&gt;</span> agent runtime ready &mdash; localhost:3000</div>
      <div class="term-line"><span class="term-prompt">$</span><span class="term-cursor"></span></div>
    </div>
  </div>

  <p class="scroll-hint">scroll to explore</p>
</hero>

<div class="features">
  <div class="feat">
    <div class="feat-tag">memory</div>
    <div class="feat-title">remember node</div>
    <div class="feat-desc">saves observations, prices, outcomes to HydraDB. persists across every run and container restart.</div>
  </div>
  <div class="feat">
    <div class="feat-tag">memory</div>
    <div class="feat-title">recall node</div>
    <div class="feat-desc">fetches relevant past memory via graph-enhanced search. finds useful context, not just similar text.</div>
  </div>
  <div class="feat">
    <div class="feat-tag">recovery</div>
    <div class="feat-title">recover node</div>
    <div class="feat-desc">wraps any action in auto error handling. checks memory for past failures, asks llm for a fix, retries.</div>
  </div>
  <div class="feat">
    <div class="feat-tag">adaptation</div>
    <div class="feat-title">plan node</div>
    <div class="feat-desc">reads memory context, llm picks best strategy from your options. different decision every run.</div>
  </div>
</div>

<footer>
  <span class="footer-txt">powered by HydraDB &middot; Groq &middot; Playwright</span>
  <span class="footer-txt">agents that remember. workflows that adapt. zero code.</span>
</footer>

<script>
const cv = document.getElementById('hero-logo');
const ctx = cv.getContext('2d');

const GLYPHS={
  'M':[[1,0,0,0,1],[1,1,0,1,1],[1,0,1,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1]],
  'n':[[0,0,0,0],[0,0,0,0],[1,1,1,0],[1,0,0,1],[1,0,0,1],[1,0,0,1],[1,0,0,1]],
  'e':[[0,0,0,0],[0,0,0,0],[0,1,1,0],[1,0,0,1],[1,1,1,1],[1,0,0,0],[0,1,1,1]],
  'm':[[0,0,0,0,0],[0,0,0,0,0],[1,1,0,1,1],[1,0,1,0,1],[1,0,1,0,1],[1,0,0,0,1],[1,0,0,0,1]],
  'O':[[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
  'S':[[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,0],[0,1,1,1,0],[0,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
};

const TEXT='MnemOS';
const TAGLINE='build agents that never forget';

let P,G,S,LG,CW,CH,bigW,bigX,bigY,tagY,tagFontSize;
let revealed=0,tagChars=0,phase=0,frame=0,blink=true,blinkT=0;
let animStarted=false;

function gw(ch){const g=GLYPHS[ch];return g?g[0].length:0;}
function getLogoColCount(){
  let cols=0;
  for(let i=0;i<TEXT.length;i++){cols+=gw(TEXT[i]);if(i<TEXT.length-1)cols+=1.8;}
  return cols;
}
function totalBigW(){let w=0;for(let i=0;i<TEXT.length;i++){w+=gw(TEXT[i])*S;if(i<TEXT.length-1)w+=LG;}return w;}
function letterX(idx){let x=0;for(let i=0;i<idx;i++)x+=gw(TEXT[i])*S+LG;return x;}
function drawGlyph(g,x,y){ctx.fillStyle='#fff';for(let r=0;r<g.length;r++)for(let c=0;c<g[r].length;c++)if(g[r][c])ctx.fillRect(x+c*S,y+r*S,P,P);}

function setup(){
  CW = cv.offsetWidth || window.innerWidth;
  const targetLogoW = CW * 0.88;
  const letterCols = 28;
  const letterGaps = 5;
  S = Math.floor(targetLogoW / (letterCols + letterGaps * 2.0));
  P = Math.max(2, Math.floor(S * 0.78));
  G = S - P;
  LG = Math.floor(S * 2.0);
  bigW = totalBigW();
  bigX = Math.floor((CW - bigW) / 2);
  bigY = Math.floor(S * 1.2);
  tagFontSize = Math.max(11, Math.floor(S * 0.85));
  tagY = bigY + 7*S + Math.floor(S * 1.8);
  CH = tagY + tagFontSize + Math.floor(S * 1.5);
  cv.width = CW;
  cv.height = CH;
}

function draw(){
  ctx.fillStyle='#000';
  ctx.fillRect(0,0,CW,CH);
  for(let i=0;i<Math.min(revealed,TEXT.length);i++){const g=GLYPHS[TEXT[i]];if(g)drawGlyph(g,bigX+letterX(i),bigY);}
  blinkT++;
  if(blinkT%16===0)blink=!blink;
  if(phase===0&&blink&&revealed<TEXT.length){ctx.fillStyle='#fff';ctx.fillRect(bigX+letterX(revealed),bigY,P,7*S-G);}
  if(phase>=1){
    const tag=TAGLINE.slice(0,tagChars);
    ctx.font=`bold ${tagFontSize}px "Courier New",monospace`;
    ctx.fillStyle='rgba(255,255,255,0.42)';
    ctx.textBaseline='top';
    const tw=ctx.measureText(TAGLINE).width;
    const tx=Math.floor((CW-tw)/2);
    ctx.fillText(tag,tx,tagY);
    if(phase===1&&tagChars<TAGLINE.length&&blink){ctx.fillStyle='rgba(255,255,255,0.42)';ctx.fillRect(tx+ctx.measureText(tag).width+2,tagY,Math.floor(tagFontSize*0.55),tagFontSize);}
  }
  frame++;
  if(phase===0){if(frame%10===0&&revealed<TEXT.length)revealed++;if(revealed>=TEXT.length){phase=1;frame=0;}}
  else if(phase===1){if(frame%3===0&&tagChars<TAGLINE.length)tagChars++;if(tagChars>=TAGLINE.length)phase=2;}
  requestAnimationFrame(draw);
}

function init(){
  setup();
  if(!animStarted){animStarted=true;draw();}
}

window.addEventListener('resize',()=>{setup();});
init();
</script>
</body>
</html>
