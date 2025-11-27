<div align="center">
  <h1 style="margin-bottom:6px;">🌀 MO·CHENG SIGNAL LOG</h1>
</div>

<script>
  const shaderCanvas = document.getElementById("shader-canvas");
  const particleCanvas = document.getElementById("particle-canvas");
  const badges = document.querySelector(".badges");

  const fitCanvas = (canvas) => {
    if (!canvas) return;
    const { width, height } = canvas.getBoundingClientRect();
    canvas.width = Math.round(width * window.devicePixelRatio);
    canvas.height = Math.round(height * window.devicePixelRatio);
  };

  const initShader = () => {
    if (!shaderCanvas) return;
    const gl = shaderCanvas.getContext("webgl");
    if (!gl) return;

    const vertexSrc = `
      attribute vec2 a_position;
      void main() {
        gl_Position = vec4(a_position, 0.0, 1.0);
      }
    `;
    const fragmentSrc = `
      precision mediump float;
      uniform vec2 u_resolution;
      uniform float u_time;

      float hash(vec2 p) {
        return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453123);
      }

      float noise(vec2 p){
        vec2 i = floor(p);
        vec2 f = fract(p);
        float a = hash(i);
        float b = hash(i + vec2(1.0, 0.0));
        float c = hash(i + vec2(0.0, 1.0));
        float d = hash(i + vec2(1.0, 1.0));
        vec2 u = f * f * (3.0 - 2.0 * f);
        return mix(a, b, u.x) + (c - a)* u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
      }

      float fbm(vec2 p) {
        float total = 0.0;
        float amplitude = 0.5;
        for (int i = 0; i < 5; i++) {
          total += noise(p) * amplitude;
          p *= 2.0;
          amplitude *= 0.45;
        }
        return total;
      }

      void main() {
        vec2 st = gl_FragCoord.xy / u_resolution.xy;
        st.x *= u_resolution.x / u_resolution.y;
        float t = u_time * 0.02;
        float m = fbm(st * 3.5 + vec2(t, t * 0.8));
        vec3 base = vec3(0.02, 0.05, 0.15);
        vec3 glow = vec3(0.15, 0.65, 0.54);
        vec3 aura = vec3(0.6, 0.4, 0.95);
        vec3 color = mix(base, glow, smoothstep(0.1, 0.9, m));
        color += aura * pow(m, 3.5);
        gl_FragColor = vec4(color, 0.55);
      }
    `;

    const compile = (type, src) => {
      const shader = gl.createShader(type);
      gl.shaderSource(shader, src);
      gl.compileShader(shader);
      if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        console.warn(gl.getShaderInfoLog(shader));
        gl.deleteShader(shader);
        return null;
      }
      return shader;
    };

    const vertexShader = compile(gl.VERTEX_SHADER, vertexSrc);
    const fragmentShader = compile(gl.FRAGMENT_SHADER, fragmentSrc);
    if (!vertexShader || !fragmentShader) return;

    const program = gl.createProgram();
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);
    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
      console.warn(gl.getProgramInfoLog(program));
      return;
    }
    gl.useProgram(program);

    const buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
      -1, -1,
      1, -1,
      -1, 1,
      -1, 1,
      1, -1,
      1, 1
    ]), gl.STATIC_DRAW);

    const positionLocation = gl.getAttribLocation(program, "a_position");
    gl.enableVertexAttribArray(positionLocation);
    gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);

    const uResolution = gl.getUniformLocation(program, "u_resolution");
    const uTime = gl.getUniformLocation(program, "u_time");

    const render = (time) => {
      fitCanvas(shaderCanvas);
      gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
      gl.uniform2f(uResolution, gl.drawingBufferWidth, gl.drawingBufferHeight);
      gl.uniform1f(uTime, time * 0.001);
      gl.drawArrays(gl.TRIANGLES, 0, 6);
      requestAnimationFrame(render);
    };

    requestAnimationFrame(render);
    window.addEventListener("resize", () => fitCanvas(shaderCanvas));
  };

  const initParticles = () => {
    if (!particleCanvas) return;
    const ctx = particleCanvas.getContext("2d");
    const particles = Array.from({ length: 80 }, () => ({
      x: Math.random(),
      y: Math.random(),
      r: Math.random() * 2 + 0.5,
      vx: (Math.random() - 0.5) * 0.002,
      vy: (Math.random() - 0.5) * 0.002,
      life: Math.random()
    }));

    const draw = () => {
      fitCanvas(particleCanvas);
      const w = particleCanvas.width;
      const h = particleCanvas.height;
      ctx.clearRect(0, 0, w, h);
      ctx.fillStyle = "rgba(4,251,195,0.45)";
      particles.forEach((p) => {
        p.x += p.vx;
        p.y += p.vy;
        p.life += 0.005;
        if (p.x < 0 || p.x > 1) p.vx *= -1;
        if (p.y < 0 || p.y > 1) p.vy *= -1;
        const alpha = 0.2 + Math.abs(Math.sin(p.life * Math.PI));
        ctx.beginPath();
        ctx.arc(p.x * w, p.y * h, p.r * window.devicePixelRatio, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(4,251,195,${alpha})`;
        ctx.fill();
      });
      requestAnimationFrame(draw);
    };
    requestAnimationFrame(draw);
    window.addEventListener("resize", () => fitCanvas(particleCanvas));
  };

  const initBadgesObserver = () => {
    if (!badges) return;
    if (!("IntersectionObserver" in window)) {
      badges.classList.add("active");
      return;
    }
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          badges.classList.add("active");
        }
      });
    }, { threshold: 0.3 });
    observer.observe(badges);
  };

  document.addEventListener("DOMContentLoaded", () => {
    initShader();
    initParticles();
    initBadgesObserver();
  });
</script>

<style>
  :root {
    color-scheme: dark;
    font-family: "Söhne", "Inter", "Menlo", system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
    --ink-1: #04fbc3;
    --ink-2: #a88bff;
    --ink-3: #ff6ec4;
    --card-bg: rgba(6, 10, 22, .75);
    --card-border: rgba(255,255,255,.12);
    --text-main: #f0f5ff;
    --text-muted: #bac3ff;
  }
  * {
    box-sizing: border-box;
  }
  body {
    margin: 0;
    background: #01030a;
    color: var(--text-main);
  }
  a {
    color: inherit;
  }
  .sky {
    min-height: 100vh;
    padding: clamp(32px, 5vw, 72px);
    position: relative;
    overflow: hidden;
    background: radial-gradient(circle at 10% 20%, rgba(4,251,195,.25), transparent 45%),
                radial-gradient(circle at 80% 10%, rgba(255,110,196,.2), transparent 40%),
                linear-gradient(135deg, #050b1b 0%, #0f1434 45%, #0c0a21 100%);
    display: grid;
    place-items: center;
  }
  .sky::before,
  .sky::after {
    content: "";
    position: absolute;
    inset: -40%;
    background: conic-gradient(from 120deg, rgba(255,255,255,.05), rgba(4,251,195,.03), transparent 75%);
    filter: blur(120px);
    animation: orbit 60s linear infinite;
    pointer-events: none;
  }
  .sky::after {
    animation-duration: 95s;
    animation-direction: reverse;
    mix-blend-mode: screen;
  }
  .grain,
  .halo,
  .flow {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }
  .grain {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='220' height='220'%3E%3Cfilter id='g'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.75' numOctaves='3'/%3E%3C/filter%3E%3Crect width='220' height='220' filter='url(%23g)' opacity='.22'/%3E%3C/svg%3E");
    mix-blend-mode: soft-light;
    animation: grainShift 12s steps(6) infinite;
    opacity: .35;
  }
  #shader-canvas,
  .particle-canvas {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
  }
  .particle-canvas {
    mix-blend-mode: screen;
    opacity: .55;
  }
  .flow {
    background: linear-gradient(120deg, rgba(255,255,255,.08), transparent 70%);
    mix-blend-mode: overlay;
    mask: radial-gradient(circle, transparent 40%, black 90%);
    animation: drift 55s linear infinite;
  }
  .free-card {
    width: min(960px, 92vw);
    border-radius: 40px;
    padding: clamp(28px, 4vw, 60px);
    border: 1.2px solid var(--card-border);
    background: var(--card-bg);
    box-shadow: 0 30px 120px rgba(2,5,18,.85);
    backdrop-filter: blur(36px);
    display: grid;
    gap: 40px;
    z-index: 1;
    position: relative;
  }
  .signature h2 {
    font-size: clamp(38px, 6vw, 70px);
    margin: 8px 0 18px;
    line-height: 1.05;
    letter-spacing: -.02em;
    animation: breathe 6s ease-in-out infinite;
  }
  .signature h2 .ink {
    background: linear-gradient(120deg, var(--ink-1), var(--ink-3));
    -webkit-background-clip: text;
    color: transparent;
  }
  .eyebrow {
    font-size: 14px;
    letter-spacing: .25em;
    text-transform: uppercase;
    color: var(--text-muted);
  }
  .tagline {
    font-size: 18px;
    color: rgba(255,255,255,.75);
    max-width: 60ch;
    line-height: 1.6;
  }
  .badges {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 24px;
  }
  .badges span {
    padding: 10px 16px;
    border-radius: 999px;
    border: 1px solid rgba(255,255,255,.2);
    background: rgba(255,255,255,.05);
    font-size: 13px;
    letter-spacing: .05em;
    text-transform: uppercase;
    animation: floatChip 6s ease-in-out infinite;
    animation-play-state: paused;
  }
  .badges.active span { animation-play-state: running; }
  .badges span:nth-child(2) { animation-delay: .6s; }
  .badges span:nth-child(3) { animation-delay: 1.2s; }
  .badges span:nth-child(4) { animation-delay: 1.8s; }
  .manifest {
    display: grid;
    gap: 18px;
  }
  .manifest h3 {
    margin: 0;
    font-size: 20px;
    letter-spacing: .2em;
    text-transform: uppercase;
    color: rgba(255,255,255,.6);
  }
  .manifest ul {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    gap: 14px;
  }
  .manifest li {
    position: relative;
    padding-left: 30px;
    font-size: 17px;
    line-height: 1.5;
  }
  .manifest li::before {
    content: "✶";
    position: absolute;
    left: 0;
    top: 0;
    color: var(--ink-2);
    animation: sparkle 3.8s linear infinite;
  }
  .contact {
    border-top: 1px dashed rgba(255,255,255,.18);
    padding-top: 28px;
    display: grid;
    gap: 16px;
  }
  .contact h3 {
    margin: 0;
    font-size: 16px;
    letter-spacing: .3em;
    text-transform: uppercase;
    color: rgba(255,255,255,.6);
  }
  .contact p {
    margin: 0;
    color: rgba(255,255,255,.75);
  }
  .contact-links {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
  }
  .contact-links a {
    padding: 10px 18px;
    border-radius: 999px;
    text-decoration: none;
    font-weight: 600;
    border: 1px solid transparent;
    background: rgba(255,255,255,.08);
    position: relative;
    overflow: hidden;
    transition: transform .3s ease, border .3s ease, background .3s ease;
  }
  .contact-links a::after {
    content: "";
    position: absolute;
    inset: -40% -10%;
    background: radial-gradient(circle, rgba(255,255,255,.9), transparent 60%);
    opacity: 0;
    mix-blend-mode: difference;
    transform: scale(.4);
    transition: opacity .35s ease, transform .35s ease;
  }
  .contact-links a:hover {
    transform: translateY(-4px) scale(1.02);
    border-color: rgba(255,255,255,.3);
    background: rgba(255,255,255,.16);
  }
  .contact-links a:hover::after {
    opacity: .9;
    transform: scale(1);
  }
  .footnote {
    font-size: 13px;
    color: rgba(255,255,255,.5);
    letter-spacing: .08em;
  }
  @keyframes orbit {
    to { transform: rotate(360deg); }
  }
  @keyframes drift {
    to { transform: translate3d(6%, -4%, 0) rotate(6deg); }
  }
  @keyframes grainShift {
    to { transform: translate3d(5%, -5%, 0); }
  }
  @keyframes pulse {
    0%, 100% { opacity: .25; transform: scale(.9); }
    50% { opacity: .5; transform: scale(1.1); }
  }
  @keyframes breathe {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-8px); }
  }
  @keyframes floatChip {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-5px); }
  }
  @keyframes sparkle {
    0% { opacity: .3; }
    40% { opacity: 1; }
    100% { opacity: .3; }
  }
  @media (max-width: 640px) {
    .free-card {
      border-radius: 28px;
      padding: 24px;
    }
    .badges span {
      font-size: 11px;
    }
  }
</style>

<div class="sky">
  <canvas id="shader-canvas" aria-hidden="true"></canvas>
  <canvas class="particle-canvas" id="particle-canvas" aria-hidden="true"></canvas>
  <div class="grain" aria-hidden="true"></div>
  <div class="flow" aria-hidden="true"></div>
  <main class="free-card">
    <section class="signature">
      <p class="eyebrow">freestyle frontend · audio reactive visuals</p>
      <h2>我把code当即兴现场，<span class="ink">让失控成为质感</span>。</h2>
      <p class="tagline"></p>
      <div class="badges">
        <span>Generative UI</span>
        <span>Glitch Animations</span>
        <span>Design Systems gone rogue</span>
        <span>Signal-based storytelling</span>
      </div>
    </section>
    <section class="manifest">
      <h3>ongoing signals</h3>
      <ul>
        <li>欢迎你来到我的个人主页</li>
        <li>目前仍在学习，抽时间看书</li>
        <li>我很喜欢一句话：“不积跬步，无以至千里。不积小流，无以成江海”</li>
      </ul>
    </section>
    <section class="contact">
      <h3>contact / ping me</h3>
      <p>任何合作、实验 demo、或只是分享一个奇怪的灵感，都可以直接扔信号给我。</p>
      <div class="contact-links">
        <a href="mailto:751024981@qq.com">Signal · Email</a>
        <a href="https://github.com/Mo-Cheng24" target="_blank">Noise · GitHub</a>
      </div>
      <p class="footnote">偏爱 async，对即兴通话也保持开放。</p>
    </section>
  </main>
</div>

