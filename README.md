<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Kahan Ghori</title>
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&family=DM+Mono:wght@300;400&display=swap" rel="stylesheet"/>
<style>
*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }

:root {
  --ink:        #0e0d0b;
  --charcoal:   #161412;
  --stone:      #1e1b17;
  --slate:      #2a2520;
  --warm-gray:  #3d3730;
  --mid:        #6b6055;
  --pale:       #b0a89e;
  --cream:      #ede8e1;
  --white:      #f5f1eb;
  --gold:       #c9a84c;
  --gold-light: #e8c878;
  --red:        #c44b2b;
}

html { scroll-behavior: smooth; }

body {
  background: var(--ink);
  color: var(--cream);
  font-family: 'DM Sans', sans-serif;
  font-weight: 300;
  overflow-x: hidden;
  cursor: none;
}

/* ── CUSTOM CURSOR ── */
.cursor {
  position: fixed; width: 8px; height: 8px;
  background: var(--gold); border-radius: 50%;
  pointer-events: none; z-index: 9999;
  transform: translate(-50%, -50%);
  transition: transform 0.1s ease;
  mix-blend-mode: difference;
}
.cursor-ring {
  position: fixed; width: 36px; height: 36px;
  border: 1px solid rgba(201,168,76,0.4); border-radius: 50%;
  pointer-events: none; z-index: 9998;
  transform: translate(-50%, -50%);
  transition: all 0.15s ease;
}

/* ── NOISE TEXTURE OVERLAY ── */
body::before {
  content: '';
  position: fixed; inset: 0; z-index: 1; pointer-events: none;
  opacity: 0.04;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
  background-size: 128px 128px;
}

/* ── LAYOUT ── */
.wrapper {
  position: relative; z-index: 2;
  max-width: 1000px; margin: 0 auto;
  padding: 0 32px 100px;
}

/* ══════════════════════════════════
   HERO
══════════════════════════════════ */
.hero {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 60px;
  padding: 80px 0;
  position: relative;
}

/* Left panel */
.hero-left { position: relative; }

.hero-eyebrow {
  font-family: 'DM Mono', monospace;
  font-size: 11px; font-weight: 300;
  letter-spacing: 4px; text-transform: uppercase;
  color: var(--gold);
  margin-bottom: 20px;
  display: flex; align-items: center; gap: 12px;
}
.hero-eyebrow::before {
  content: '';
  width: 32px; height: 1px;
  background: var(--gold);
}

.hero-name {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(72px, 10vw, 120px);
  line-height: 0.92;
  letter-spacing: -1px;
  color: var(--white);
  margin-bottom: 8px;
}
.hero-name span {
  display: block;
  color: transparent;
  -webkit-text-stroke: 1px var(--warm-gray);
}

.hero-role {
  font-size: 13px; font-weight: 300;
  color: var(--mid); letter-spacing: 1px;
  margin-top: 24px; margin-bottom: 36px;
  line-height: 1.8;
  max-width: 280px;
  border-left: 2px solid var(--slate);
  padding-left: 16px;
}

.hero-tags {
  display: flex; flex-wrap: wrap; gap: 8px;
}
.tag {
  font-family: 'DM Mono', monospace;
  font-size: 10px; letter-spacing: 2px; text-transform: uppercase;
  padding: 6px 14px;
  border: 1px solid var(--warm-gray);
  color: var(--pale);
  border-radius: 2px;
  transition: all 0.25s ease;
}
.tag:hover {
  border-color: var(--gold);
  color: var(--gold);
  background: rgba(201,168,76,0.06);
}

/* Right panel — 3D card stack */
.hero-right {
  perspective: 1000px;
  display: flex; align-items: center; justify-content: center;
}

.card-3d-scene {
  width: 300px; height: 380px;
  position: relative;
  transform-style: preserve-3d;
  animation: floatScene 6s ease-in-out infinite;
}
@keyframes floatScene {
  0%,100% { transform: rotateX(4deg) rotateY(-8deg); }
  50%      { transform: rotateX(-2deg) rotateY(8deg); }
}

.card-layer {
  position: absolute; inset: 0;
  border-radius: 6px;
  transform-style: preserve-3d;
}

/* Shadow layers behind */
.layer-shadow-3 {
  background: rgba(201,168,76,0.08);
  border: 1px solid rgba(201,168,76,0.15);
  transform: translateZ(-60px) translateX(18px) translateY(18px);
}
.layer-shadow-2 {
  background: var(--slate);
  border: 1px solid var(--warm-gray);
  transform: translateZ(-30px) translateX(10px) translateY(10px);
}
.layer-main {
  background: linear-gradient(145deg, var(--stone) 0%, var(--charcoal) 100%);
  border: 1px solid var(--warm-gray);
  transform: translateZ(0);
  overflow: hidden;
  display: flex; flex-direction: column;
  justify-content: space-between;
  padding: 36px 32px;
}
.layer-main::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0; height: 3px;
  background: linear-gradient(90deg, var(--gold), var(--red), var(--gold));
  background-size: 200%;
  animation: shineBar 4s ease infinite;
}
@keyframes shineBar {
  0%,100% { background-position: 0%; }
  50%      { background-position: 100%; }
}

/* Front card content */
.card-initials {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 80px; line-height: 1;
  color: transparent;
  -webkit-text-stroke: 1px rgba(201,168,76,0.3);
  user-select: none;
}
.card-title-block {}
.card-label {
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 3px; text-transform: uppercase;
  color: var(--gold); margin-bottom: 6px;
}
.card-main-text {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 28px; line-height: 1.1;
  color: var(--white);
}
.card-divider {
  height: 1px; background: var(--warm-gray); margin: 16px 0;
}
.card-meta {
  font-family: 'DM Mono', monospace;
  font-size: 10px; color: var(--mid); line-height: 2;
}
.card-meta strong { color: var(--pale); font-weight: 400; }

/* Shine effect on card */
.layer-shine {
  position: absolute; inset: 0; border-radius: 6px;
  background: linear-gradient(135deg,
    rgba(255,255,255,0.06) 0%,
    transparent 50%,
    rgba(0,0,0,0.1) 100%
  );
  transform: translateZ(2px);
  pointer-events: none;
}

/* Vertical text label floating */
.card-float-label {
  position: absolute;
  right: -48px; top: 50%;
  transform: translateY(-50%) translateZ(20px) rotate(90deg);
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 4px; text-transform: uppercase;
  color: var(--warm-gray);
  white-space: nowrap;
}

/* Hero divider line */
.hero::after {
  content: '';
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--warm-gray), transparent);
}

/* ══════════════════════════════════
   SECTION STRUCTURE
══════════════════════════════════ */
section { padding: 80px 0; position: relative; }
section + section::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--slate), transparent);
}

.sec-header {
  display: flex; align-items: baseline;
  justify-content: space-between;
  margin-bottom: 48px;
}
.sec-number {
  font-family: 'DM Mono', monospace;
  font-size: 11px; color: var(--gold);
  letter-spacing: 2px;
}
.sec-title {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 48px; letter-spacing: 1px;
  color: var(--white);
  line-height: 1;
}
.sec-rule {
  flex: 1; height: 1px;
  background: var(--slate);
  margin: 0 24px;
  align-self: center;
}

/* ══════════════════════════════════
   ABOUT — CODE BLOCK
══════════════════════════════════ */
.code-block {
  background: var(--charcoal);
  border: 1px solid var(--slate);
  border-radius: 4px;
  overflow: hidden;
  transform-style: preserve-3d;
  transition: transform 0.4s ease, box-shadow 0.4s ease;
}
.code-block:hover {
  transform: perspective(800px) rotateX(1deg) rotateY(-1deg) translateY(-4px);
  box-shadow: 12px 24px 60px rgba(0,0,0,0.6), 0 0 0 1px var(--warm-gray);
}

.code-bar {
  background: var(--stone);
  padding: 12px 20px;
  display: flex; align-items: center; gap: 8px;
  border-bottom: 1px solid var(--slate);
}
.dot { width: 10px; height: 10px; border-radius: 50%; }
.dot-r { background: #c44b2b; }
.dot-y { background: #c9a84c; }
.dot-g { background: #4a9b6f; }
.code-filename {
  font-family: 'DM Mono', monospace;
  font-size: 11px; color: var(--mid);
  margin-left: 8px; letter-spacing: 1px;
}

.code-body {
  padding: 28px 32px;
  font-family: 'DM Mono', monospace;
  font-size: 13px; line-height: 2;
}
.c-dim    { color: var(--warm-gray); }
.c-key    { color: #c9a84c; }
.c-str    { color: #9bc48a; }
.c-val    { color: #7eb8d4; }
.c-arr    { color: var(--pale); }
.c-fn     { color: #c44b2b; }
.indent   { padding-left: 28px; display: block; }

/* ══════════════════════════════════
   TECH STACK — HORIZONTAL SHELVES
══════════════════════════════════ */
.shelf {
  margin-bottom: 36px;
}
.shelf-label {
  font-family: 'DM Mono', monospace;
  font-size: 10px; letter-spacing: 3px; text-transform: uppercase;
  color: var(--mid); margin-bottom: 14px;
  display: flex; align-items: center; gap: 12px;
}
.shelf-label::after {
  content: ''; flex: 1; height: 1px;
  background: var(--slate);
}

.tech-row {
  display: flex; flex-wrap: wrap; gap: 10px;
}

.tech-pill {
  display: flex; align-items: center; gap: 8px;
  padding: 10px 18px;
  background: var(--stone);
  border: 1px solid var(--slate);
  border-radius: 3px;
  font-size: 12px; font-weight: 400;
  color: var(--pale);
  letter-spacing: 0.5px;
  transition: all 0.25s ease;
  position: relative; overflow: hidden;
  cursor: default;
}
.tech-pill::after {
  content: '';
  position: absolute; inset: 0;
  background: linear-gradient(90deg, var(--gold), transparent);
  opacity: 0;
  transition: opacity 0.25s ease;
}
.tech-pill:hover {
  border-color: var(--gold);
  color: var(--white);
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.4);
}
.tech-pill:hover::after { opacity: 0.04; }
.tech-dot {
  width: 6px; height: 6px; border-radius: 50%;
  background: var(--gold); flex-shrink: 0;
}

/* ══════════════════════════════════
   STATS — 3D ELEVATED PANELS
══════════════════════════════════ */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2px;
  background: var(--slate);
  border: 1px solid var(--slate);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 40px;
}
.stat-panel {
  background: var(--stone);
  padding: 36px 28px;
  text-align: center;
  position: relative;
  overflow: hidden;
  cursor: default;
  transition: background 0.25s ease;
}
.stat-panel:hover { background: var(--charcoal); }
.stat-panel::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 2px; background: var(--gold);
  transform: scaleX(0);
  transition: transform 0.3s ease;
  transform-origin: left;
}
.stat-panel:hover::before { transform: scaleX(1); }
.stat-num {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 56px; line-height: 1;
  color: var(--white);
  display: block; margin-bottom: 6px;
  transition: color 0.25s;
}
.stat-panel:hover .stat-num { color: var(--gold-light); }
.stat-lbl {
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 3px; text-transform: uppercase;
  color: var(--mid);
}

/* GitHub stat images */
.stat-images {
  display: grid; grid-template-columns: 1fr 1fr; gap: 16px;
  margin-bottom: 16px;
}
.stat-images img {
  width: 100%; border-radius: 4px;
  border: 1px solid var(--slate);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.stat-images img:hover {
  transform: perspective(600px) rotateY(-3deg) translateY(-4px);
  box-shadow: 12px 16px 40px rgba(0,0,0,0.5);
}
.streak-img {
  width: 100%; border-radius: 4px;
  border: 1px solid var(--slate);
  display: block;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.streak-img:hover {
  transform: perspective(600px) rotateX(-2deg) translateY(-4px);
  box-shadow: 0 16px 40px rgba(0,0,0,0.5);
}

/* ══════════════════════════════════
   CERTIFICATIONS
══════════════════════════════════ */
.cert-list { display: flex; flex-direction: column; gap: 2px; }

.cert-item {
  display: flex; align-items: center;
  background: var(--stone);
  border: 1px solid var(--slate);
  padding: 24px 28px;
  gap: 24px;
  transition: all 0.25s ease;
  position: relative; overflow: hidden;
  cursor: default;
}
.cert-item:first-child { border-radius: 4px 4px 0 0; }
.cert-item:last-child  { border-radius: 0 0 4px 4px; }

.cert-item::before {
  content: '';
  position: absolute; left: 0; top: 0; bottom: 0;
  width: 3px; background: var(--gold);
  transform: scaleY(0);
  transition: transform 0.3s ease;
  transform-origin: bottom;
}
.cert-item:hover { background: var(--charcoal); padding-left: 36px; }
.cert-item:hover::before { transform: scaleY(1); }

.cert-num {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 32px; color: var(--slate);
  min-width: 40px;
  transition: color 0.25s;
}
.cert-item:hover .cert-num { color: var(--warm-gray); }
.cert-body {}
.cert-name {
  font-size: 14px; font-weight: 400;
  color: var(--white); margin-bottom: 4px;
}
.cert-org {
  font-family: 'DM Mono', monospace;
  font-size: 10px; letter-spacing: 2px; text-transform: uppercase;
  color: var(--gold);
}
.cert-arrow {
  margin-left: auto; font-size: 18px; color: var(--slate);
  transition: color 0.25s, transform 0.25s;
}
.cert-item:hover .cert-arrow { color: var(--gold); transform: translateX(4px); }

/* ══════════════════════════════════
   CONNECT
══════════════════════════════════ */
.connect-grid {
  display: grid; grid-template-columns: repeat(3, 1fr);
  gap: 2px; background: var(--slate);
  border: 1px solid var(--slate); border-radius: 4px; overflow: hidden;
}
.connect-item {
  background: var(--stone);
  padding: 32px 24px;
  text-decoration: none;
  display: flex; flex-direction: column;
  gap: 12px;
  transition: all 0.25s ease;
  position: relative; overflow: hidden;
}
.connect-item::after {
  content: '';
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 2px; background: var(--gold);
  transform: scaleX(0); transform-origin: left;
  transition: transform 0.3s ease;
}
.connect-item:hover { background: var(--charcoal); }
.connect-item:hover::after { transform: scaleX(1); }

.connect-platform {
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 3px; text-transform: uppercase;
  color: var(--gold);
}
.connect-handle {
  font-size: 13px; color: var(--pale); font-weight: 300;
  word-break: break-all;
}
.connect-arrow {
  font-size: 18px; color: var(--slate);
  transition: color 0.25s, transform 0.25s;
  margin-top: auto;
}
.connect-item:hover .connect-arrow { color: var(--gold); transform: translate(2px,-2px); }

/* ══════════════════════════════════
   FOOTER
══════════════════════════════════ */
.footer {
  padding: 48px 0 0;
  border-top: 1px solid var(--slate);
  display: flex; justify-content: space-between; align-items: flex-end;
  flex-wrap: wrap; gap: 20px;
}
.footer-quote {
  font-style: italic; font-size: 13px;
  color: var(--mid); max-width: 400px;
  line-height: 1.8; font-weight: 300;
  border-left: 2px solid var(--warm-gray); padding-left: 16px;
}
.footer-badge img { height: 24px; filter: grayscale(0.4); }

/* ══════════════════════════════════
   ANIMATIONS — ENTER
══════════════════════════════════ */
.reveal {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.7s ease, transform 0.7s ease;
}
.reveal.visible {
  opacity: 1;
  transform: none;
}

@media (max-width: 700px) {
  .hero { grid-template-columns: 1fr; min-height: auto; padding: 60px 0 40px; }
  .hero-right { display: none; }
  .stats-grid, .connect-grid { grid-template-columns: 1fr; }
  .stat-images { grid-template-columns: 1fr; }
  .sec-title { font-size: 36px; }
}
</style>
</head>
<body>

<div class="cursor" id="cursor"></div>
<div class="cursor-ring" id="cursorRing"></div>

<div class="wrapper">

  <!-- ══ HERO ══ -->
  <div class="hero">
    <div class="hero-left reveal">
      <div class="hero-eyebrow">Software Developer</div>
      <h1 class="hero-name">
        Kahan
        <span>Ghori</span>
      </h1>
      <p class="hero-role">
        MERN Stack Enthusiast<br/>
        Backend &amp; API Development<br/>
        Building scalable digital solutions
      </p>
      <div class="hero-tags">
        <span class="tag">Node.js</span>
        <span class="tag">React</span>
        <span class="tag">MongoDB</span>
        <span class="tag">Express</span>
        <span class="tag">REST APIs</span>
        <span class="tag">C++</span>
      </div>
    </div>

    <div class="hero-right reveal" style="transition-delay:0.2s">
      <div class="card-3d-scene">
        <div class="card-layer layer-shadow-3"></div>
        <div class="card-layer layer-shadow-2"></div>
        <div class="card-layer layer-main">
          <div class="card-initials">KG</div>
          <div>
            <div class="card-label">Profile</div>
            <div class="card-main-text">Aspiring<br/>Software<br/>Developer</div>
            <div class="card-divider"></div>
            <div class="card-meta">
              <div><strong>Stack</strong> — MERN</div>
              <div><strong>Focus</strong> — Backend</div>
              <div><strong>Status</strong> — Open to Work</div>
              <div><strong>Base</strong> — India 🇮🇳</div>
            </div>
          </div>
        </div>
        <div class="card-layer layer-shine"></div>
        <div class="card-float-label">ghorikahan · github</div>
      </div>
    </div>
  </div>

  <!-- ══ ABOUT ══ -->
  <section class="reveal">
    <div class="sec-header">
      <span class="sec-number">01</span>
      <div class="sec-rule"></div>
      <h2 class="sec-title">About</h2>
    </div>
    <div class="code-block">
      <div class="code-bar">
        <div class="dot dot-r"></div>
        <div class="dot dot-y"></div>
        <div class="dot dot-g"></div>
        <span class="code-filename">kahan.js</span>
      </div>
      <div class="code-body">
        <span class="c-dim">// Kahan Ghori — Developer Profile</span><br/>
        <br/>
        <span class="c-key">const</span> <span class="c-val">developer</span> <span class="c-dim">=</span> {<br/>
        <span class="indent"><span class="c-key">name</span><span class="c-dim">:</span>       <span class="c-str">"Kahan Ghori"</span>,</span>
        <span class="indent"><span class="c-key">education</span><span class="c-dim">:</span>  <span class="c-str">"B.Tech @ CodingGita"</span>,</span>
        <span class="indent"><span class="c-key">location</span><span class="c-dim">:</span>   <span class="c-str">"India 🇮🇳"</span>,</span>
        <span class="indent"><span class="c-key">passion</span><span class="c-dim">:</span>    <span class="c-str">"Scalable &amp; efficient digital solutions"</span>,</span>
        <span class="indent"><span class="c-key">learning</span><span class="c-dim">:</span>   <span class="c-str">"Full Stack MERN Development"</span>,</span>
        <span class="indent"><span class="c-key">available</span><span class="c-dim">:</span>  [<span class="c-str">"Internships"</span>, <span class="c-str">"Collaborations"</span>, <span class="c-str">"Entry-level"</span>],</span>
        <span class="indent"><span class="c-fn">greet</span><span class="c-dim">:</span>      () <span class="c-dim">=&gt;</span> <span class="c-str">"Let's build something great together."</span></span>
        };<br/>
        <br/>
        <span class="c-dim">// → </span><span class="c-val">developer</span>.<span class="c-fn">greet</span>()<br/>
        <span class="c-str">"Let's build something great together."</span>
      </div>
    </div>
  </section>

  <!-- ══ TECH ══ -->
  <section class="reveal">
    <div class="sec-header">
      <span class="sec-number">02</span>
      <div class="sec-rule"></div>
      <h2 class="sec-title">Stack</h2>
    </div>

    <div class="shelf">
      <div class="shelf-label">Frontend</div>
      <div class="tech-row">
        <div class="tech-pill"><div class="tech-dot"></div>React</div>
        <div class="tech-pill"><div class="tech-dot"></div>JavaScript</div>
        <div class="tech-pill"><div class="tech-dot"></div>HTML5</div>
        <div class="tech-pill"><div class="tech-dot"></div>CSS3</div>
      </div>
    </div>

    <div class="shelf">
      <div class="shelf-label">Backend</div>
      <div class="tech-row">
        <div class="tech-pill"><div class="tech-dot"></div>Node.js</div>
        <div class="tech-pill"><div class="tech-dot"></div>Express.js</div>
        <div class="tech-pill"><div class="tech-dot"></div>REST APIs</div>
        <div class="tech-pill"><div class="tech-dot"></div>C++</div>
      </div>
    </div>

    <div class="shelf">
      <div class="shelf-label">Database &amp; Tools</div>
      <div class="tech-row">
        <div class="tech-pill"><div class="tech-dot"></div>MongoDB</div>
        <div class="tech-pill"><div class="tech-dot"></div>Git</div>
        <div class="tech-pill"><div class="tech-dot"></div>GitHub</div>
        <div class="tech-pill"><div class="tech-dot"></div>Figma</div>
        <div class="tech-pill"><div class="tech-dot"></div>VS Code</div>
      </div>
    </div>
  </section>

  <!-- ══ STATS ══ -->
  <section class="reveal">
    <div class="sec-header">
      <span class="sec-number">03</span>
      <div class="sec-rule"></div>
      <h2 class="sec-title">Stats</h2>
    </div>

    <div class="stats-grid">
      <div class="stat-panel">
        <span class="stat-num" id="c1">0</span>
        <span class="stat-lbl">Commits</span>
      </div>
      <div class="stat-panel">
        <span class="stat-num" id="c2">0</span>
        <span class="stat-lbl">Repositories</span>
      </div>
      <div class="stat-panel">
        <span class="stat-num" id="c3">0</span>
        <span class="stat-lbl">Day Streak</span>
      </div>
    </div>

    <div class="stat-images">
      <img src="https://github-readme-stats.vercel.app/api?username=ghorikahan&show_icons=true&hide_border=true&count_private=true&bg_color=161412&title_color=c9a84c&icon_color=c9a84c&text_color=b0a89e&border_radius=3" alt="GitHub Stats"/>
      <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=ghorikahan&layout=compact&hide_border=true&bg_color=161412&title_color=c9a84c&text_color=b0a89e&border_radius=3" alt="Top Languages"/>
    </div>

    <img class="streak-img" src="https://github-readme-streak-stats-eight.vercel.app/?user=ghorikahan&hide_border=true&background=161412&ring=c9a84c&fire=c44b2b&currStreakLabel=c9a84c&dates=6b6055&border_radius=3" alt="GitHub Streak"/>
  </section>

  <!-- ══ CERTS ══ -->
  <section class="reveal">
    <div class="sec-header">
      <span class="sec-number">04</span>
      <div class="sec-rule"></div>
      <h2 class="sec-title">Credentials</h2>
    </div>
    <div class="cert-list">
      <div class="cert-item">
        <span class="cert-num">01</span>
        <div class="cert-body">
          <div class="cert-name">Software Engineering Job Simulation</div>
          <div class="cert-org">JPMorgan Chase &amp; Co.</div>
        </div>
        <span class="cert-arrow">→</span>
      </div>
      <div class="cert-item">
        <span class="cert-num">02</span>
        <div class="cert-body">
          <div class="cert-name">GitHub Copilot Certification</div>
          <div class="cert-org">Microsoft</div>
        </div>
        <span class="cert-arrow">→</span>
      </div>
    </div>
  </section>

  <!-- ══ CONNECT ══ -->
  <section class="reveal">
    <div class="sec-header">
      <span class="sec-number">05</span>
      <div class="sec-rule"></div>
      <h2 class="sec-title">Connect</h2>
    </div>
    <div class="connect-grid">
      <a class="connect-item" href="https://linkedin.com/in/kahan-ghori-157487394" target="_blank">
        <span class="connect-platform">LinkedIn</span>
        <span class="connect-handle">kahan-ghori-157487394</span>
        <span class="connect-arrow">↗</span>
      </a>
      <a class="connect-item" href="mailto:ghorikahan@gmail.com">
        <span class="connect-platform">Email</span>
        <span class="connect-handle">ghorikahan@gmail.com</span>
        <span class="connect-arrow">↗</span>
      </a>
      <a class="connect-item" href="https://github.com/ghorikahan" target="_blank">
        <span class="connect-platform">GitHub</span>
        <span class="connect-handle">github.com/ghorikahan</span>
        <span class="connect-arrow">↗</span>
      </a>
    </div>
  </section>

  <!-- ══ FOOTER ══ -->
  <footer class="footer reveal">
    <p class="footer-quote">
      "Consistent learning. Hands-on practice.<br/>
      Staying curious. Building impact."
    </p>
    <div class="footer-badge">
      <img src="https://komarev.com/ghpvc/?username=ghorikahan&label=Profile+Views&color=c9a84c&style=flat" alt="Views"/>
    </div>
  </footer>

</div>

<script>
// Custom cursor
const cursor = document.getElementById('cursor');
const ring = document.getElementById('cursorRing');
let mx = 0, my = 0, rx = 0, ry = 0;

document.addEventListener('mousemove', e => {
  mx = e.clientX; my = e.clientY;
  cursor.style.left = mx + 'px';
  cursor.style.top  = my + 'px';
});

// Lag ring
(function loop() {
  rx += (mx - rx) * 0.12;
  ry += (my - ry) * 0.12;
  ring.style.left = rx + 'px';
  ring.style.top  = ry + 'px';
  requestAnimationFrame(loop);
})();

// Cursor scale on hover
document.querySelectorAll('a, .tech-pill, .cert-item, .connect-item, .stat-panel, .code-block').forEach(el => {
  el.addEventListener('mouseenter', () => {
    cursor.style.transform = 'translate(-50%,-50%) scale(2.5)';
    ring.style.transform = 'translate(-50%,-50%) scale(1.6)';
    ring.style.borderColor = 'rgba(201,168,76,0.6)';
  });
  el.addEventListener('mouseleave', () => {
    cursor.style.transform = 'translate(-50%,-50%) scale(1)';
    ring.style.transform = 'translate(-50%,-50%) scale(1)';
    ring.style.borderColor = 'rgba(201,168,76,0.4)';
  });
});

// Scroll reveal
const observer = new IntersectionObserver(entries => {
  entries.forEach((e, i) => {
    if (e.isIntersecting) {
      setTimeout(() => e.target.classList.add('visible'), 80);
      observer.unobserve(e.target);
    }
  });
}, { threshold: 0.1 });
document.querySelectorAll('.reveal').forEach(el => observer.observe(el));

// Counter animation
function count(id, target, suffix, dur) {
  const el = document.getElementById(id);
  if (!el) return;
  let start = null;
  function step(ts) {
    if (!start) start = ts;
    const p = Math.min((ts - start) / dur, 1);
    const ease = 1 - Math.pow(1 - p, 3);
    el.textContent = Math.floor(ease * target) + (p >= 1 ? suffix : '');
    if (p < 1) requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
}

// Trigger counters when stats section is visible
const statsSection = document.querySelector('.stats-grid');
const statsObs = new IntersectionObserver(entries => {
  if (entries[0].isIntersecting) {
    count('c1', 120, '+', 1800);
    count('c2', 15,  '+', 1400);
    count('c3', 30,  '+', 1200);
    statsObs.disconnect();
  }
}, { threshold: 0.3 });
if (statsSection) statsObs.observe(statsSection);

// Mouse parallax on hero card
const scene = document.querySelector('.card-3d-scene');
if (scene) {
  document.addEventListener('mousemove', e => {
    const cx = window.innerWidth / 2;
    const cy = window.innerHeight / 2;
    const dx = (e.clientX - cx) / cx;
    const dy = (e.clientY - cy) / cy;
    scene.style.animation = 'none';
    scene.style.transform = `rotateX(${-dy * 10}deg) rotateY(${dx * 12}deg)`;
  });
  document.addEventListener('mouseleave', () => {
    scene.style.animation = 'floatScene 6s ease-in-out infinite';
  });
}
</script>
</body>
</html>
