import { useState, useEffect, useRef, useCallback } from "react";

/* ─────────────────────────────────────────
   GLOBAL STYLES injected once into <head>
───────────────────────────────────────── */
const GLOBAL_CSS = `
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&family=DM+Mono:wght@300;400&display=swap');

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

body::before {
  content: '';
  position: fixed; inset: 0; z-index: 1; pointer-events: none;
  opacity: 0.04;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
  background-size: 128px 128px;
}

/* ── KEYFRAMES ── */
@keyframes floatScene {
  0%,100% { transform: rotateX(4deg) rotateY(-8deg); }
  50%      { transform: rotateX(-2deg) rotateY(8deg); }
}
@keyframes shineBar {
  0%,100% { background-position: 0%; }
  50%      { background-position: 100%; }
}

/* ── CURSOR ── */
.kg-cursor {
  position: fixed; width: 8px; height: 8px;
  background: var(--gold); border-radius: 50%;
  pointer-events: none; z-index: 9999;
  transform: translate(-50%, -50%);
  transition: transform 0.1s ease;
  mix-blend-mode: difference;
}
.kg-cursor-ring {
  position: fixed; width: 36px; height: 36px;
  border: 1px solid rgba(201,168,76,0.4); border-radius: 50%;
  pointer-events: none; z-index: 9998;
  transform: translate(-50%, -50%);
  transition: all 0.15s ease;
}
.kg-cursor.hovered      { transform: translate(-50%,-50%) scale(2.5); }
.kg-cursor-ring.hovered { transform: translate(-50%,-50%) scale(1.6); border-color: rgba(201,168,76,0.6); }

/* ── WRAPPER ── */
.kg-wrapper {
  position: relative; z-index: 2;
  max-width: 1000px; margin: 0 auto;
  padding: 0 32px 100px;
}

/* ── HERO ── */
.kg-hero {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: 60px;
  padding: 80px 0;
  position: relative;
}
.kg-hero::after {
  content: '';
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--warm-gray), transparent);
}
.kg-hero-left { position: relative; }

.kg-eyebrow {
  font-family: 'DM Mono', monospace;
  font-size: 11px; font-weight: 300;
  letter-spacing: 4px; text-transform: uppercase;
  color: var(--gold);
  margin-bottom: 20px;
  display: flex; align-items: center; gap: 12px;
}
.kg-eyebrow::before {
  content: '';
  width: 32px; height: 1px;
  background: var(--gold);
}

.kg-hero-name {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(72px, 10vw, 120px);
  line-height: 0.92;
  letter-spacing: -1px;
  color: var(--white);
  margin-bottom: 8px;
}
.kg-hero-name span {
  display: block;
  color: transparent;
  -webkit-text-stroke: 1px var(--warm-gray);
}

.kg-hero-role {
  font-size: 13px; font-weight: 300;
  color: var(--mid); letter-spacing: 1px;
  margin-top: 24px; margin-bottom: 36px;
  line-height: 1.8;
  max-width: 280px;
  border-left: 2px solid var(--slate);
  padding-left: 16px;
}

.kg-tags { display: flex; flex-wrap: wrap; gap: 8px; }
.kg-tag {
  font-family: 'DM Mono', monospace;
  font-size: 10px; letter-spacing: 2px; text-transform: uppercase;
  padding: 6px 14px;
  border: 1px solid var(--warm-gray);
  color: var(--pale);
  border-radius: 2px;
  transition: all 0.25s ease;
  cursor: default;
}
.kg-tag:hover {
  border-color: var(--gold);
  color: var(--gold);
  background: rgba(201,168,76,0.06);
}

/* ── 3D CARD ── */
.kg-hero-right {
  perspective: 1000px;
  display: flex; align-items: center; justify-content: center;
}
.kg-card-scene {
  width: 300px; height: 380px;
  position: relative;
  transform-style: preserve-3d;
  animation: floatScene 6s ease-in-out infinite;
}
.kg-card-layer {
  position: absolute; inset: 0;
  border-radius: 6px;
  transform-style: preserve-3d;
}
.kg-layer-s3 {
  background: rgba(201,168,76,0.08);
  border: 1px solid rgba(201,168,76,0.15);
  transform: translateZ(-60px) translateX(18px) translateY(18px);
}
.kg-layer-s2 {
  background: var(--slate);
  border: 1px solid var(--warm-gray);
  transform: translateZ(-30px) translateX(10px) translateY(10px);
}
.kg-layer-main {
  background: linear-gradient(145deg, var(--stone) 0%, var(--charcoal) 100%);
  border: 1px solid var(--warm-gray);
  transform: translateZ(0);
  overflow: hidden;
  display: flex; flex-direction: column;
  justify-content: space-between;
  padding: 36px 32px;
  position: relative;
}
.kg-layer-main::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0; height: 3px;
  background: linear-gradient(90deg, var(--gold), var(--red), var(--gold));
  background-size: 200%;
  animation: shineBar 4s ease infinite;
}
.kg-card-initials {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 80px; line-height: 1;
  color: transparent;
  -webkit-text-stroke: 1px rgba(201,168,76,0.3);
  user-select: none;
}
.kg-card-label {
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 3px; text-transform: uppercase;
  color: var(--gold); margin-bottom: 6px;
}
.kg-card-main-text {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 28px; line-height: 1.1;
  color: var(--white);
}
.kg-card-divider { height: 1px; background: var(--warm-gray); margin: 16px 0; }
.kg-card-meta {
  font-family: 'DM Mono', monospace;
  font-size: 10px; color: var(--mid); line-height: 2;
}
.kg-card-meta strong { color: var(--pale); font-weight: 400; }
.kg-layer-shine {
  position: absolute; inset: 0; border-radius: 6px;
  background: linear-gradient(135deg, rgba(255,255,255,0.06) 0%, transparent 50%, rgba(0,0,0,0.1) 100%);
  transform: translateZ(2px);
  pointer-events: none;
}
.kg-card-float-label {
  position: absolute;
  right: -48px; top: 50%;
  transform: translateY(-50%) translateZ(20px) rotate(90deg);
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 4px; text-transform: uppercase;
  color: var(--warm-gray);
  white-space: nowrap;
}

/* ── SECTIONS ── */
.kg-section { padding: 80px 0; position: relative; }
.kg-section + .kg-section::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--slate), transparent);
}
.kg-sec-header {
  display: flex; align-items: baseline;
  justify-content: space-between;
  margin-bottom: 48px;
}
.kg-sec-number {
  font-family: 'DM Mono', monospace;
  font-size: 11px; color: var(--gold); letter-spacing: 2px;
}
.kg-sec-title {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 48px; letter-spacing: 1px;
  color: var(--white); line-height: 1;
}
.kg-sec-rule {
  flex: 1; height: 1px;
  background: var(--slate);
  margin: 0 24px;
  align-self: center;
}

/* ── CODE BLOCK ── */
.kg-code-block {
  background: var(--charcoal);
  border: 1px solid var(--slate);
  border-radius: 4px;
  overflow: hidden;
  transition: transform 0.4s ease, box-shadow 0.4s ease;
}
.kg-code-block:hover {
  transform: perspective(800px) rotateX(1deg) rotateY(-1deg) translateY(-4px);
  box-shadow: 12px 24px 60px rgba(0,0,0,0.6), 0 0 0 1px var(--warm-gray);
}
.kg-code-bar {
  background: var(--stone);
  padding: 12px 20px;
  display: flex; align-items: center; gap: 8px;
  border-bottom: 1px solid var(--slate);
}
.kg-dot { width: 10px; height: 10px; border-radius: 50%; }
.kg-dot-r { background: #c44b2b; }
.kg-dot-y { background: #c9a84c; }
.kg-dot-g { background: #4a9b6f; }
.kg-code-filename {
  font-family: 'DM Mono', monospace;
  font-size: 11px; color: var(--mid);
  margin-left: 8px; letter-spacing: 1px;
}
.kg-code-body {
  padding: 28px 32px;
  font-family: 'DM Mono', monospace;
  font-size: 13px; line-height: 2;
}
.c-dim  { color: var(--warm-gray); }
.c-key  { color: #c9a84c; }
.c-str  { color: #9bc48a; }
.c-val  { color: #7eb8d4; }
.c-fn   { color: #c44b2b; }
.c-arr  { color: var(--pale); }
.c-ind  { padding-left: 28px; display: block; }

/* ── TECH STACK ── */
.kg-shelf { margin-bottom: 36px; }
.kg-shelf-label {
  font-family: 'DM Mono', monospace;
  font-size: 10px; letter-spacing: 3px; text-transform: uppercase;
  color: var(--mid); margin-bottom: 14px;
  display: flex; align-items: center; gap: 12px;
}
.kg-shelf-label::after { content: ''; flex: 1; height: 1px; background: var(--slate); }
.kg-tech-row { display: flex; flex-wrap: wrap; gap: 10px; }
.kg-tech-pill {
  display: flex; align-items: center; gap: 8px;
  padding: 10px 18px;
  background: var(--stone);
  border: 1px solid var(--slate);
  border-radius: 3px;
  font-size: 12px; font-weight: 400;
  color: var(--pale); letter-spacing: 0.5px;
  transition: all 0.25s ease;
  position: relative; overflow: hidden;
  cursor: default;
}
.kg-tech-pill::after {
  content: '';
  position: absolute; inset: 0;
  background: linear-gradient(90deg, var(--gold), transparent);
  opacity: 0; transition: opacity 0.25s ease;
}
.kg-tech-pill:hover {
  border-color: var(--gold); color: var(--white);
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.4);
}
.kg-tech-pill:hover::after { opacity: 0.04; }
.kg-tech-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--gold); flex-shrink: 0; }

/* ── STATS ── */
.kg-stats-grid {
  display: grid; grid-template-columns: repeat(3, 1fr);
  gap: 2px; background: var(--slate);
  border: 1px solid var(--slate); border-radius: 4px; overflow: hidden;
  margin-bottom: 40px;
}
.kg-stat-panel {
  background: var(--stone); padding: 36px 28px;
  text-align: center; position: relative; overflow: hidden;
  cursor: default; transition: background 0.25s ease;
}
.kg-stat-panel:hover { background: var(--charcoal); }
.kg-stat-panel::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 2px; background: var(--gold);
  transform: scaleX(0); transition: transform 0.3s ease;
  transform-origin: left;
}
.kg-stat-panel:hover::before { transform: scaleX(1); }
.kg-stat-num {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 56px; line-height: 1; color: var(--white);
  display: block; margin-bottom: 6px; transition: color 0.25s;
}
.kg-stat-panel:hover .kg-stat-num { color: var(--gold-light); }
.kg-stat-lbl {
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 3px; text-transform: uppercase; color: var(--mid);
}
.kg-stat-images {
  display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin-bottom: 16px;
}
.kg-stat-images img {
  width: 100%; border-radius: 4px;
  border: 1px solid var(--slate);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.kg-stat-images img:hover {
  transform: perspective(600px) rotateY(-3deg) translateY(-4px);
  box-shadow: 12px 16px 40px rgba(0,0,0,0.5);
}
.kg-streak-img {
  width: 100%; border-radius: 4px; border: 1px solid var(--slate); display: block;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.kg-streak-img:hover {
  transform: perspective(600px) rotateX(-2deg) translateY(-4px);
  box-shadow: 0 16px 40px rgba(0,0,0,0.5);
}

/* ── CERTS ── */
.kg-cert-list { display: flex; flex-direction: column; gap: 2px; }
.kg-cert-item {
  display: flex; align-items: center;
  background: var(--stone); border: 1px solid var(--slate);
  padding: 24px 28px; gap: 24px;
  transition: all 0.25s ease;
  position: relative; overflow: hidden; cursor: default;
}
.kg-cert-item:first-child { border-radius: 4px 4px 0 0; }
.kg-cert-item:last-child  { border-radius: 0 0 4px 4px; }
.kg-cert-item::before {
  content: '';
  position: absolute; left: 0; top: 0; bottom: 0;
  width: 3px; background: var(--gold);
  transform: scaleY(0); transition: transform 0.3s ease; transform-origin: bottom;
}
.kg-cert-item:hover { background: var(--charcoal); padding-left: 36px; }
.kg-cert-item:hover::before { transform: scaleY(1); }
.kg-cert-num {
  font-family: 'Bebas Neue', sans-serif; font-size: 32px; color: var(--slate);
  min-width: 40px; transition: color 0.25s;
}
.kg-cert-item:hover .kg-cert-num { color: var(--warm-gray); }
.kg-cert-name { font-size: 14px; font-weight: 400; color: var(--white); margin-bottom: 4px; }
.kg-cert-org {
  font-family: 'DM Mono', monospace;
  font-size: 10px; letter-spacing: 2px; text-transform: uppercase; color: var(--gold);
}
.kg-cert-arrow {
  margin-left: auto; font-size: 18px; color: var(--slate);
  transition: color 0.25s, transform 0.25s;
}
.kg-cert-item:hover .kg-cert-arrow { color: var(--gold); transform: translateX(4px); }

/* ── CONNECT ── */
.kg-connect-grid {
  display: grid; grid-template-columns: repeat(3, 1fr);
  gap: 2px; background: var(--slate);
  border: 1px solid var(--slate); border-radius: 4px; overflow: hidden;
}
.kg-connect-item {
  background: var(--stone); padding: 32px 24px;
  text-decoration: none; display: flex; flex-direction: column; gap: 12px;
  transition: all 0.25s ease; position: relative; overflow: hidden;
}
.kg-connect-item::after {
  content: '';
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 2px; background: var(--gold);
  transform: scaleX(0); transform-origin: left; transition: transform 0.3s ease;
}
.kg-connect-item:hover { background: var(--charcoal); }
.kg-connect-item:hover::after { transform: scaleX(1); }
.kg-connect-platform {
  font-family: 'DM Mono', monospace;
  font-size: 9px; letter-spacing: 3px; text-transform: uppercase; color: var(--gold);
}
.kg-connect-handle { font-size: 13px; color: var(--pale); font-weight: 300; word-break: break-all; }
.kg-connect-arrow {
  font-size: 18px; color: var(--slate); transition: color 0.25s, transform 0.25s; margin-top: auto;
}
.kg-connect-item:hover .kg-connect-arrow { color: var(--gold); transform: translate(2px,-2px); }

/* ── FOOTER ── */
.kg-footer {
  padding: 48px 0 0; border-top: 1px solid var(--slate);
  display: flex; justify-content: space-between; align-items: flex-end;
  flex-wrap: wrap; gap: 20px;
}
.kg-footer-quote {
  font-style: italic; font-size: 13px;
  color: var(--mid); max-width: 400px;
  line-height: 1.8; font-weight: 300;
  border-left: 2px solid var(--warm-gray); padding-left: 16px;
}
.kg-footer-badge img { height: 24px; filter: grayscale(0.4); }

/* ── REVEAL ── */
.kg-reveal {
  opacity: 0; transform: translateY(30px);
  transition: opacity 0.7s ease, transform 0.7s ease;
}
.kg-reveal.visible { opacity: 1; transform: none; }

@media (max-width: 700px) {
  .kg-hero { grid-template-columns: 1fr; min-height: auto; padding: 60px 0 40px; }
  .kg-hero-right { display: none; }
  .kg-stats-grid, .kg-connect-grid { grid-template-columns: 1fr; }
  .kg-stat-images { grid-template-columns: 1fr; }
  .kg-sec-title { font-size: 36px; }
}
`;

/* ─────────────────────────────────────────
   HOOKS
───────────────────────────────────────── */
function useCursor() {
  const cursorRef = useRef(null);
  const ringRef = useRef(null);
  const mx = useRef(0), my = useRef(0);
  const rx = useRef(0), ry = useRef(0);
  const raf = useRef(null);

  useEffect(() => {
    const onMove = (e) => {
      mx.current = e.clientX; my.current = e.clientY;
      if (cursorRef.current) {
        cursorRef.current.style.left = e.clientX + "px";
        cursorRef.current.style.top  = e.clientY + "px";
      }
    };
    window.addEventListener("mousemove", onMove);
    const loop = () => {
      rx.current += (mx.current - rx.current) * 0.12;
      ry.current += (my.current - ry.current) * 0.12;
      if (ringRef.current) {
        ringRef.current.style.left = rx.current + "px";
        ringRef.current.style.top  = ry.current + "px";
      }
      raf.current = requestAnimationFrame(loop);
    };
    raf.current = requestAnimationFrame(loop);
    return () => { window.removeEventListener("mousemove", onMove); cancelAnimationFrame(raf.current); };
  }, []);

  return { cursorRef, ringRef };
}

function useReveal() {
  useEffect(() => {
    const els = document.querySelectorAll(".kg-reveal");
    const obs = new IntersectionObserver((entries) => {
      entries.forEach((e) => {
        if (e.isIntersecting) { setTimeout(() => e.target.classList.add("visible"), 80); obs.unobserve(e.target); }
      });
    }, { threshold: 0.1 });
    els.forEach((el) => obs.observe(el));
    return () => obs.disconnect();
  }, []);
}

function useCounter(target, suffix, duration, triggerRef) {
  const [val, setVal] = useState("0");
  const triggered = useRef(false);

  useEffect(() => {
    if (!triggerRef?.current) return;
    const obs = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting && !triggered.current) {
        triggered.current = true;
        let startTs = null;
        const step = (ts) => {
          if (!startTs) startTs = ts;
          const p = Math.min((ts - startTs) / duration, 1);
          const ease = 1 - Math.pow(1 - p, 3);
          setVal(Math.floor(ease * target) + (p >= 1 ? suffix : ""));
          if (p < 1) requestAnimationFrame(step);
        };
        requestAnimationFrame(step);
        obs.disconnect();
      }
    }, { threshold: 0.3 });
    obs.observe(triggerRef.current);
    return () => obs.disconnect();
  }, [target, suffix, duration, triggerRef]);

  return val;
}

function useCardParallax(sceneRef) {
  useEffect(() => {
    const onMove = (e) => {
      if (!sceneRef.current) return;
      const cx = window.innerWidth / 2, cy = window.innerHeight / 2;
      const dx = (e.clientX - cx) / cx, dy = (e.clientY - cy) / cy;
      sceneRef.current.style.animation = "none";
      sceneRef.current.style.transform = `rotateX(${-dy * 10}deg) rotateY(${dx * 12}deg)`;
    };
    const onLeave = () => {
      if (!sceneRef.current) return;
      sceneRef.current.style.animation = "floatScene 6s ease-in-out infinite";
      sceneRef.current.style.transform = "";
    };
    window.addEventListener("mousemove", onMove);
    window.addEventListener("mouseleave", onLeave);
    return () => { window.removeEventListener("mousemove", onMove); window.removeEventListener("mouseleave", onLeave); };
  }, [sceneRef]);
}

/* ─────────────────────────────────────────
   CURSOR HOVER helper
───────────────────────────────────────── */
function useCursorHover(cursorRef, ringRef) {
  const enter = useCallback(() => {
    cursorRef.current?.classList.add("hovered");
    ringRef.current?.classList.add("hovered");
  }, [cursorRef, ringRef]);
  const leave = useCallback(() => {
    cursorRef.current?.classList.remove("hovered");
    ringRef.current?.classList.remove("hovered");
  }, [cursorRef, ringRef]);
  return { onMouseEnter: enter, onMouseLeave: leave };
}

/* ─────────────────────────────────────────
   SUB-COMPONENTS
───────────────────────────────────────── */
function SectionHeader({ number, title }) {
  return (
    <div className="kg-sec-header">
      <span className="kg-sec-number">{number}</span>
      <div className="kg-sec-rule" />
      <h2 className="kg-sec-title">{title}</h2>
    </div>
  );
}

function TechShelf({ label, items }) {
  return (
    <div className="kg-shelf">
      <div className="kg-shelf-label">{label}</div>
      <div className="kg-tech-row">
        {items.map((t) => (
          <div className="kg-tech-pill" key={t}>
            <div className="kg-tech-dot" />
            {t}
          </div>
        ))}
      </div>
    </div>
  );
}

function CertItem({ num, name, org }) {
  return (
    <div className="kg-cert-item">
      <span className="kg-cert-num">{num}</span>
      <div>
        <div className="kg-cert-name">{name}</div>
        <div className="kg-cert-org">{org}</div>
      </div>
      <span className="kg-cert-arrow">→</span>
    </div>
  );
}

function ConnectItem({ platform, handle, href }) {
  return (
    <a className="kg-connect-item" href={href} target={href.startsWith("mailto") ? undefined : "_blank"} rel="noreferrer">
      <span className="kg-connect-platform">{platform}</span>
      <span className="kg-connect-handle">{handle}</span>
      <span className="kg-connect-arrow">↗</span>
    </a>
  );
}

/* ─────────────────────────────────────────
   MAIN COMPONENT
───────────────────────────────────────── */
export default function KahanProfile() {
  const { cursorRef, ringRef } = useCursor();
  const hover = useCursorHover(cursorRef, ringRef);
  const sceneRef = useRef(null);
  const statsGridRef = useRef(null);

  useReveal();
  useCardParallax(sceneRef);

  const c1 = useCounter(120, "+", 1800, statsGridRef);
  const c2 = useCounter(15,  "+", 1400, statsGridRef);
  const c3 = useCounter(30,  "+", 1200, statsGridRef);

  /* Inject global CSS once */
  useEffect(() => {
    const id = "kg-global-styles";
    if (!document.getElementById(id)) {
      const style = document.createElement("style");
      style.id = id;
      style.textContent = GLOBAL_CSS;
      document.head.appendChild(style);
    }
  }, []);

  return (
    <>
      {/* CURSOR */}
      <div className="kg-cursor" ref={cursorRef} />
      <div className="kg-cursor-ring" ref={ringRef} />

      <div className="kg-wrapper">

        {/* ══ HERO ══ */}
        <div className="kg-hero">
          <div className="kg-hero-left kg-reveal">
            <div className="kg-eyebrow">Software Developer</div>
            <h1 className="kg-hero-name">
              Kahan
              <span>Ghori</span>
            </h1>
            <p className="kg-hero-role">
              MERN Stack Enthusiast<br />
              Backend &amp; API Development<br />
              Building scalable digital solutions
            </p>
            <div className="kg-tags">
              {["Node.js","React","MongoDB","Express","REST APIs","C++"].map((t) => (
                <span className="kg-tag" key={t}>{t}</span>
              ))}
            </div>
          </div>

          <div className="kg-hero-right kg-reveal" style={{ transitionDelay: "0.2s" }}>
            <div className="kg-card-scene" ref={sceneRef}>
              <div className="kg-card-layer kg-layer-s3" />
              <div className="kg-card-layer kg-layer-s2" />
              <div className="kg-card-layer kg-layer-main">
                <div className="kg-card-initials">KG</div>
                <div>
                  <div className="kg-card-label">Profile</div>
                  <div className="kg-card-main-text">Aspiring<br/>Software<br/>Developer</div>
                  <div className="kg-card-divider" />
                  <div className="kg-card-meta">
                    <div><strong>Stack</strong> — MERN</div>
                    <div><strong>Focus</strong> — Backend</div>
                    <div><strong>Status</strong> — Open to Work</div>
                    <div><strong>Base</strong> — India 🇮🇳</div>
                  </div>
                </div>
              </div>
              <div className="kg-card-layer kg-layer-shine" />
              <div className="kg-card-float-label">ghorikahan · github</div>
            </div>
          </div>
        </div>

        {/* ══ ABOUT ══ */}
        <section className="kg-section kg-reveal">
          <SectionHeader number="01" title="About" />
          <div className="kg-code-block" {...hover}>
            <div className="kg-code-bar">
              <div className="kg-dot kg-dot-r" /><div className="kg-dot kg-dot-y" /><div className="kg-dot kg-dot-g" />
              <span className="kg-code-filename">kahan.js</span>
            </div>
            <div className="kg-code-body">
              <span className="c-dim">{"// Kahan Ghori — Developer Profile"}</span><br/><br/>
              <span className="c-key">const</span> <span className="c-val"> developer</span> <span className="c-dim"> =</span> {"{"}<br/>
              <span className="c-ind"><span className="c-key">name</span><span className="c-dim">:</span>       <span className="c-str">"Kahan Ghori"</span>,</span>
              <span className="c-ind"><span className="c-key">education</span><span className="c-dim">:</span>  <span className="c-str">"B.Tech @ CodingGita"</span>,</span>
              <span className="c-ind"><span className="c-key">location</span><span className="c-dim">:</span>   <span className="c-str">"India 🇮🇳"</span>,</span>
              <span className="c-ind"><span className="c-key">passion</span><span className="c-dim">:</span>    <span className="c-str">"Scalable &amp; efficient digital solutions"</span>,</span>
              <span className="c-ind"><span className="c-key">learning</span><span className="c-dim">:</span>   <span className="c-str">"Full Stack MERN Development"</span>,</span>
              <span className="c-ind"><span className="c-key">available</span><span className="c-dim">:</span>  [<span className="c-str">"Internships"</span>, <span className="c-str">"Collaborations"</span>, <span className="c-str">"Entry-level"</span>],</span>
              <span className="c-ind"><span className="c-fn">greet</span><span className="c-dim">:</span>      {"() "}<span className="c-dim">{"=>"}</span> <span className="c-str">"Let's build something great together."</span></span>
              {"};"}
              <br/><br/>
              <span className="c-dim">{"// → "}</span><span className="c-val">developer</span>.<span className="c-fn">greet</span>()<br/>
              <span className="c-str">"Let's build something great together."</span>
            </div>
          </div>
        </section>

        {/* ══ TECH ══ */}
        <section className="kg-section kg-reveal">
          <SectionHeader number="02" title="Stack" />
          <TechShelf label="Frontend"          items={["React","JavaScript","HTML5","CSS3"]} />
          <TechShelf label="Backend"           items={["Node.js","Express.js","REST APIs","C++"]} />
          <TechShelf label="Database & Tools"  items={["MongoDB","Git","GitHub","Figma","VS Code"]} />
        </section>

        {/* ══ STATS ══ */}
        <section className="kg-section kg-reveal">
          <SectionHeader number="03" title="Stats" />
          <div className="kg-stats-grid" ref={statsGridRef}>
            {[["c1",c1,"Commits"],["c2",c2,"Repositories"],["c3",c3,"Day Streak"]].map(([id,val,lbl])=>(
              <div className="kg-stat-panel" key={id} {...hover}>
                <span className="kg-stat-num">{val}</span>
                <span className="kg-stat-lbl">{lbl}</span>
              </div>
            ))}
          </div>
          <div className="kg-stat-images">
            <img src="https://github-readme-stats.vercel.app/api?username=ghorikahan&show_icons=true&hide_border=true&count_private=true&bg_color=161412&title_color=c9a84c&icon_color=c9a84c&text_color=b0a89e&border_radius=3" alt="GitHub Stats"/>
            <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=ghorikahan&layout=compact&hide_border=true&bg_color=161412&title_color=c9a84c&text_color=b0a89e&border_radius=3" alt="Top Languages"/>
          </div>
          <img className="kg-streak-img" src="https://github-readme-streak-stats-eight.vercel.app/?user=ghorikahan&hide_border=true&background=161412&ring=c9a84c&fire=c44b2b&currStreakLabel=c9a84c&dates=6b6055&border_radius=3" alt="GitHub Streak"/>
        </section>

        {/* ══ CERTS ══ */}
        <section className="kg-section kg-reveal">
          <SectionHeader number="04" title="Credentials" />
          <div className="kg-cert-list">
            <CertItem num="01" name="Software Engineering Job Simulation" org="JPMorgan Chase & Co." />
            <CertItem num="02" name="GitHub Copilot Certification"        org="Microsoft" />
          </div>
        </section>

        {/* ══ CONNECT ══ */}
        <section className="kg-section kg-reveal">
          <SectionHeader number="05" title="Connect" />
          <div className="kg-connect-grid">
            <ConnectItem platform="LinkedIn" handle="kahan-ghori-157487394"   href="https://linkedin.com/in/kahan-ghori-157487394" />
            <ConnectItem platform="Email"    handle="ghorikahan@gmail.com"     href="mailto:ghorikahan@gmail.com" />
            <ConnectItem platform="GitHub"   handle="github.com/ghorikahan"    href="https://github.com/ghorikahan" />
          </div>
        </section>

        {/* ══ FOOTER ══ */}
        <footer className="kg-footer kg-reveal">
          <p className="kg-footer-quote">
            "Consistent learning. Hands-on practice.<br/>
            Staying curious. Building impact."
          </p>
          <div className="kg-footer-badge">
            <img src="https://komarev.com/ghpvc/?username=ghorikahan&label=Profile+Views&color=c9a84c&style=flat" alt="Profile Views"/>
          </div>
        </footer>

      </div>
    </>
  );
}
