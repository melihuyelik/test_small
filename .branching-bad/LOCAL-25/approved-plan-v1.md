# Plan: LOCAL-25 — Görsel Düzenleme (3D Globe)

## Goal
Add a large, 3D-looking Earth globe visual to the center of the homepage (index.html), rendered via an HTML5 Canvas using JavaScript.

## Summary
A new full-screen canvas (or large fixed canvas) will render an animated 3D-style Earth using the Canvas 2D API — drawing sphere shading with radial gradients, latitude/longitude grid lines, and slowly rotating landmass outlines approximated with bezier paths or a simplified SVG-path approach. No external libraries or image assets required — pure CSS/Canvas.

## Scope

### File: `index.html`

#### 1. Add `#globe` canvas element
Insert a new `<canvas id="globe">` element in `<body>` between the `.noise` div and the `.orb` divs (before `<div class="orb orb-1">`). This canvas renders the globe behind the `.container` but above the particle canvas.

```html
<canvas id="globe"></canvas>
```

#### 2. Add CSS for `#globe`
Insert into the `<style>` block after the `#particles` rule:

```css
#globe {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 2;
  pointer-events: none;
  opacity: 0.85;
}
```

z-index 2 places it above particles (z-index 0) and the noise (z-index 1), but below the container (z-index 10).

#### 3. Add globe rendering JavaScript
Append a new IIFE `<script>` block before the closing `</body>` tag (or inside the existing script section, after the countdown IIFE). The script:

```js
(function () {
  const canvas = document.getElementById('globe');
  const ctx = canvas.getContext('2d');
  const R = Math.min(window.innerWidth, window.innerHeight) * 0.38; // radius ~38% of smaller dimension
  const SIZE = R * 2 + 4;
  canvas.width = SIZE;
  canvas.height = SIZE;
  const cx = SIZE / 2;
  const cy = SIZE / 2;
  let angle = 0;

  // Simplified continent outlines as arrays of [lon, lat] in degrees
  // Each continent is an array of polygon points
  const continents = [
    // Africa
    [[-18,15],[10,37],[37,37],[52,12],[52,-12],[35,-35],[20,-36],[10,-18],[-18,15]],
    // Europe
    [[-10,36],[40,36],[40,60],[25,71],[5,62],[-10,50],[-10,36]],
    // Asia
    [[40,36],[130,36],[130,55],[80,72],[40,70],[40,36]],
    // North America
    [[-170,72],[-52,47],[-52,10],[-80,8],[-118,22],[-170,60],[-170,72]],
    // South America
    [[-80,8],[-34,-6],[-52,-34],[-72,-50],[-80,8]],
    // Australia
    [[114,-22],[154,-22],[154,-38],[114,-38],[114,-22]],
    // Antarctica
    [[-180,-70],[180,-70],[180,-90],[-180,-90],[-180,-70]],
  ];

  function project(lon, lat, rotY) {
    const phi = (lat * Math.PI) / 180;
    const theta = ((lon + rotY) * Math.PI) / 180;
    const x3 = Math.cos(phi) * Math.sin(theta);
    const y3 = -Math.sin(phi);
    const z3 = Math.cos(phi) * Math.cos(theta);
    return { x: cx + R * x3, y: cy + R * y3, z: z3 };
  }

  function drawGlobe(rot) {
    ctx.clearRect(0, 0, SIZE, SIZE);

    // Ocean sphere base with radial gradient
    const oceanGrad = ctx.createRadialGradient(cx - R * 0.3, cy - R * 0.3, R * 0.05, cx, cy, R);
    oceanGrad.addColorStop(0, '#1a5fa8');
    oceanGrad.addColorStop(0.5, '#0a2a5e');
    oceanGrad.addColorStop(1, '#050d1e');
    ctx.beginPath();
    ctx.arc(cx, cy, R, 0, Math.PI * 2);
    ctx.fillStyle = oceanGrad;
    ctx.fill();

    // Lat/lon grid lines
    ctx.strokeStyle = 'rgba(80,160,255,0.15)';
    ctx.lineWidth = 0.5;
    // Longitude lines every 30 degrees
    for (let lon = 0; lon < 360; lon += 30) {
      ctx.beginPath();
      let first = true;
      for (let lat = -90; lat <= 90; lat += 3) {
        const p = project(lon, lat, rot);
        if (p.z < 0) { first = true; continue; }
        if (first) { ctx.moveTo(p.x, p.y); first = false; }
        else ctx.lineTo(p.x, p.y);
      }
      ctx.stroke();
    }
    // Latitude lines every 30 degrees
    for (let lat = -60; lat <= 60; lat += 30) {
      ctx.beginPath();
      let first = true;
      for (let lon = 0; lon <= 360; lon += 3) {
        const p = project(lon, lat, rot);
        if (p.z < 0) { first = true; continue; }
        if (first) { ctx.moveTo(p.x, p.y); first = false; }
        else ctx.lineTo(p.x, p.y);
      }
      ctx.stroke();
    }

    // Continent fills (clip to visible hemisphere z>0)
    ctx.fillStyle = 'rgba(40,180,80,0.7)';
    ctx.strokeStyle = 'rgba(80,220,100,0.5)';
    ctx.lineWidth = 1;
    for (const continent of continents) {
      ctx.beginPath();
      let first = true;
      for (const [lon, lat] of continent) {
        const p = project(lon, lat, rot);
        if (p.z < 0) { first = true; continue; }
        if (first) { ctx.moveTo(p.x, p.y); first = false; }
        else ctx.lineTo(p.x, p.y);
      }
      ctx.closePath();
      ctx.fill();
      ctx.stroke();
    }

    // Atmosphere glow ring
    const atmGrad = ctx.createRadialGradient(cx, cy, R * 0.85, cx, cy, R * 1.15);
    atmGrad.addColorStop(0, 'rgba(30,100,255,0)');
    atmGrad.addColorStop(0.5, 'rgba(30,100,255,0.12)');
    atmGrad.addColorStop(1, 'rgba(30,100,255,0)');
    ctx.beginPath();
    ctx.arc(cx, cy, R * 1.15, 0, Math.PI * 2);
    ctx.fillStyle = atmGrad;
    ctx.fill();

    // Specular highlight (top-left)
    const specGrad = ctx.createRadialGradient(cx - R * 0.35, cy - R * 0.35, 0, cx - R * 0.2, cy - R * 0.2, R * 0.6);
    specGrad.addColorStop(0, 'rgba(255,255,255,0.18)');
    specGrad.addColorStop(1, 'rgba(255,255,255,0)');
    ctx.beginPath();
    ctx.arc(cx, cy, R, 0, Math.PI * 2);
    ctx.fillStyle = specGrad;
    ctx.fill();

    // Dark shadow on right side
    const shadowGrad = ctx.createRadialGradient(cx + R * 0.5, cy + R * 0.3, 0, cx + R * 0.2, cy, R);
    shadowGrad.addColorStop(0, 'rgba(0,0,0,0.45)');
    shadowGrad.addColorStop(1, 'rgba(0,0,0,0)');
    ctx.beginPath();
    ctx.arc(cx, cy, R, 0, Math.PI * 2);
    ctx.fillStyle = shadowGrad;
    ctx.fill();
  }

  function animate() {
    angle += 0.003; // slow eastward rotation
    const rotDeg = (angle * 180) / Math.PI;
    drawGlobe(rotDeg);
    requestAnimationFrame(animate);
  }

  animate();

  window.addEventListener('resize', () => {
    const newR = Math.min(window.innerWidth, window.innerHeight) * 0.38;
    const newSize = newR * 2 + 4;
    canvas.width = newSize;
    canvas.height = newSize;
    // Update closure vars — re-assign globals via Object.assign trick not needed; use module-level var
    // Simple approach: reload the animation with updated radius
    // Since R and SIZE are const inside closure, handle resize by reloading:
    location.reload(); // fallback: simple resize reload
    // Better: make R, SIZE, cx, cy mutable vars (use let instead of const in actual code)
  });
})();
```

**Important note for executor**: In the actual implementation, declare `R`, `SIZE`, `cx`, `cy` with `let` (not `const`) so they can be updated on resize without reloading. Update the resize handler to recalculate these values and call `canvas.width = newSize; canvas.height = newSize;` and update the mutable vars.

#### 4. Adjust `.container` z-index and styling
The existing `.container` has `z-index: 10` which already places it above the globe (z-index 2). No change needed.

Optionally reduce `.container` background opacity slightly so the globe shows through subtly:
- Change `background: rgba(255, 255, 255, 0.02)` to `rgba(255, 255, 255, 0.04)` for slightly more visible card.

## Risks
- **Continent polygon simplification**: The simplified bounding-box polygons for continents are approximate; they convey shape but won't be geographically perfect. This is acceptable for a decorative visual.
- **Resize handling**: Using `location.reload()` on resize is a poor UX fallback. Use mutable `let` vars for R/SIZE/cx/cy instead.
- **Performance**: Running three canvas animations (particles + globe) simultaneously. The globe renders ~60fps but is a single canvas with moderate complexity. Should be smooth on modern hardware.
- **z-index layering**: Globe at z-index 2 sits above noise (z-index 1) and particles (z-index 0), below container (z-index 10). The orb divs are z-index 0 with blur. Globe may obscure some orb glow — this is acceptable and visually desirable.

## Verification
Open `index.html` directly in a browser (no build step required — pure HTML/CSS/JS). Verify:
1. Globe canvas appears centered on page.
2. Globe rotates slowly.
3. Grid lines, continent shapes, atmosphere glow, and specular highlight are visible.
4. Countdown and text container renders above the globe.
5. No JS console errors.

## Acceptance Criteria
- A large 3D-style globe is centered on the homepage.
- The globe rotates continuously.
- The globe displays ocean coloring, landmass shapes, grid lines, atmosphere glow, and a specular highlight to create a 3D appearance.
- Existing content (badge, title, countdown, footer) remains visible and functional above the globe.
- No external resources (images, CDN libraries) are required.