# Goal

Create a standalone animated "Coming Soon" HTML page with a dark color scheme and 2026 design aesthetics (glassmorphism, neon accents, fluid animations, brutalist typography touches).

---

## Summary

A single `index.html` file in the repository root with embedded CSS and JS. No build tools or dependencies — pure HTML/CSS/JS. The page features rich animations: particle background, glitch text effect on the headline, animated gradient orbs, a countdown timer, and smooth entrance animations.

---

## Scope

### File to create

**`/Users/melih/Documents/code/test_small/index.html`**

### Full implementation spec

#### Structure
```html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Coming Soon</title>
  <style>/* all CSS inline */</style>
</head>
<body>
  <canvas id="particles"></canvas>
  <div class="orb orb-1"></div>
  <div class="orb orb-2"></div>
  <div class="orb orb-3"></div>
  <main class="container">
    <div class="badge">2026</div>
    <h1 class="glitch" data-text="COMING SOON">COMING SOON</h1>
    <p class="subtitle">Bir şeyler geliyor. Hazır ol.</p>
    <div class="countdown">
      <div class="unit"><span id="days">00</span><label>Gün</label></div>
      <div class="unit"><span id="hours">00</span><label>Saat</label></div>
      <div class="unit"><span id="mins">00</span><label>Dakika</label></div>
      <div class="unit"><span id="secs">00</span><label>Saniye</label></div>
    </div>
    <div class="line"></div>
    <p class="tagline">© 2026 — The Future Is Now</p>
  </main>
  <script>/* all JS inline */</script>
</body>
</html>
```

#### CSS Details

**Reset & base:**
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body {
  background: #050508;
  color: #e0e0ff;
  font-family: 'Segoe UI', system-ui, sans-serif;
  min-height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

**Canvas (particle layer):**
```css
#particles {
  position: fixed;
  inset: 0;
  z-index: 0;
  opacity: 0.6;
}
```

**Animated gradient orbs (blurred, absolute positioned):**
```css
.orb {
  position: fixed;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.25;
  animation: drift 12s ease-in-out infinite alternate;
  z-index: 0;
}
.orb-1 { width: 600px; height: 600px; background: #6600ff; top: -200px; left: -150px; animation-delay: 0s; }
.orb-2 { width: 500px; height: 500px; background: #00c8ff; bottom: -150px; right: -100px; animation-delay: -4s; }
.orb-3 { width: 400px; height: 400px; background: #ff0080; top: 50%; left: 50%; transform: translate(-50%,-50%); animation-delay: -8s; }
@keyframes drift {
  from { transform: translate(0,0) scale(1); }
  to   { transform: translate(40px, 60px) scale(1.15); }
}
```

**Container:**
```css
.container {
  position: relative;
  z-index: 10;
  text-align: center;
  padding: 3rem 2rem;
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 24px;
  backdrop-filter: blur(20px);
  max-width: 720px;
  width: 90%;
  animation: fadeUp 1.2s cubic-bezier(0.16,1,0.3,1) both;
}
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(60px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

**Badge:**
```css
.badge {
  display: inline-block;
  padding: 4px 16px;
  border: 1px solid #6600ff;
  color: #bb88ff;
  border-radius: 999px;
  font-size: 0.75rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  margin-bottom: 2rem;
  animation: pulse-border 3s ease-in-out infinite;
}
@keyframes pulse-border {
  0%,100% { box-shadow: 0 0 0 0 rgba(102,0,255,0.4); }
  50%      { box-shadow: 0 0 0 8px rgba(102,0,255,0); }
}
```

**Glitch heading:**
```css
.glitch {
  font-size: clamp(2.5rem, 8vw, 6rem);
  font-weight: 900;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  position: relative;
  color: #ffffff;
  text-shadow: 0 0 30px rgba(102,0,255,0.8), 0 0 60px rgba(0,200,255,0.4);
  animation: glitch-skew 4s infinite linear alternate-reverse;
}
.glitch::before, .glitch::after {
  content: attr(data-text);
  position: absolute;
  inset: 0;
  clip-path: polygon(0 0, 100% 0, 100% 35%, 0 35%);
}
.glitch::before {
  color: #00c8ff;
  animation: glitch-anim 3s infinite linear alternate-reverse;
  left: 2px;
}
.glitch::after {
  color: #ff0080;
  animation: glitch-anim2 2.5s infinite linear alternate-reverse;
  left: -2px;
  clip-path: polygon(0 65%, 100% 65%, 100% 100%, 0 100%);
}
@keyframes glitch-anim {
  0%   { transform: skew(0deg); opacity: 1; }
  20%  { transform: skew(-2deg) translateX(3px); opacity: 0.8; }
  40%  { transform: skew(0deg); opacity: 1; }
  60%  { transform: skew(1deg) translateX(-2px); opacity: 0.9; }
  80%  { transform: skew(-1deg); opacity: 1; }
  100% { transform: skew(0deg); }
}
@keyframes glitch-anim2 {
  0%   { transform: skew(0deg); }
  25%  { transform: skew(2deg) translateX(-3px); opacity: 0.7; }
  50%  { transform: skew(0deg); opacity: 1; }
  75%  { transform: skew(-2deg) translateX(2px); }
  100% { transform: skew(0deg); }
}
@keyframes glitch-skew {
  0%,90%,100% { transform: skew(0deg); }
  92% { transform: skew(-1.5deg); }
  95% { transform: skew(1deg); }
}
```

**Subtitle:**
```css
.subtitle {
  margin-top: 1rem;
  font-size: 1rem;
  color: rgba(224,224,255,0.5);
  letter-spacing: 0.2em;
  text-transform: uppercase;
  animation: fadeUp 1.2s 0.3s cubic-bezier(0.16,1,0.3,1) both;
}
```

**Countdown:**
```css
.countdown {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  margin: 2.5rem 0;
  animation: fadeUp 1.2s 0.6s cubic-bezier(0.16,1,0.3,1) both;
}
.unit {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
}
.unit span {
  font-size: clamp(2rem, 5vw, 3.5rem);
  font-weight: 800;
  font-variant-numeric: tabular-nums;
  background: linear-gradient(135deg, #ffffff, #bb88ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  line-height: 1;
}
.unit label {
  font-size: 0.65rem;
  letter-spacing: 0.25em;
  text-transform: uppercase;
  color: rgba(224,224,255,0.4);
}
```

**Decorative line:**
```css
.line {
  height: 1px;
  background: linear-gradient(90deg, transparent, #6600ff, #00c8ff, transparent);
  margin: 0 auto 1.5rem;
  width: 80%;
  animation: shimmer 3s linear infinite;
  background-size: 200% 100%;
}
@keyframes shimmer {
  from { background-position: 200% center; }
  to   { background-position: -200% center; }
}
.tagline {
  font-size: 0.7rem;
  color: rgba(224,224,255,0.25);
  letter-spacing: 0.3em;
  text-transform: uppercase;
  animation: fadeUp 1.2s 0.9s cubic-bezier(0.16,1,0.3,1) both;
}
```

#### JavaScript Details

**Particle system (canvas-based):**
- On load, grab `canvas#particles`, set width/height to `window.innerWidth/Height`.
- Create array of ~120 particle objects: `{ x, y, vx, vy, radius: 1–3, alpha: 0.2–0.8, color: random from ['#6600ff','#00c8ff','#ff0080','#ffffff'] }`.
- `requestAnimationFrame` loop: clear canvas, update each particle position (wrap edges), draw filled circles with `ctx.globalAlpha = particle.alpha`.
- Resize listener to update canvas dimensions.

**Countdown timer (target: 2027-01-01T00:00:00):**
```js
const target = new Date('2027-01-01T00:00:00').getTime();
function updateCountdown() {
  const diff = target - Date.now();
  if (diff <= 0) { /* show zeros */ return; }
  const days  = Math.floor(diff / 86400000);
  const hours = Math.floor((diff % 86400000) / 3600000);
  const mins  = Math.floor((diff % 3600000) / 60000);
  const secs  = Math.floor((diff % 60000) / 1000);
  document.getElementById('days').textContent  = String(days).padStart(2,'0');
  document.getElementById('hours').textContent = String(hours).padStart(2,'0');
  document.getElementById('mins').textContent  = String(mins).padStart(2,'0');
  document.getElementById('secs').textContent  = String(secs).padStart(2,'0');
}
setInterval(updateCountdown, 1000);
updateCountdown();
```

---

## Risks

- None significant — pure static HTML, no dependencies, no build step.
- `backdrop-filter` may not work in some older browsers (graceful degradation: panel remains visible without blur).

---

## Verification

- Open `index.html` directly in a browser — no server required.
- Confirm: dark background renders, orbs animate, glitch effect fires on heading, countdown ticks, particles move.
- No build step needed.

---

## Acceptance Criteria

- [ ] Single file: `index.html` at repo root.
- [ ] "COMING SOON" heading with glitch animation.
- [ ] Animated background: floating gradient orbs + canvas particle field.
- [ ] Countdown timer counting down to 2027-01-01.
- [ ] Dark color palette: primary background `#050508`, accents purple/cyan/pink.
- [ ] Glassmorphism card container.
- [ ] All animations run smoothly on load with no external requests.