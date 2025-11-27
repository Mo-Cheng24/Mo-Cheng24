<div align="center">
  <h1 style="margin-bottom:6px;">🌀 墨·CHENG SIGNAL LOG</h1>
  <p style="font-size:14px;color:#9cf9e9;">unpolished · alive · personal</p>
</div>

<style>
  :root {
    color-scheme: dark;
    font-family: "Söhne", "Inter", "Menlo", system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
    --ink-1: #04fbc3;
    --ink-2: #a88bff;
    --ink-3: #ff6ec4;
    --bg-1: #020617;
    --bg-2: #0d1b3f;
    --bg-3: #130b29;
    --card-bg: rgba(5, 8, 21, .78);
    --card-border: rgba(255,255,255,.18);
    --text-main: #f0f5ff;
    --text-muted: #aeb8ff;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    min-height: 100vh;
    background: var(--bg-1);
    color: var(--text-main);
  }
  a { color: inherit; }
  .canvas {
    position: relative;
    min-height: 100vh;
    padding: clamp(32px, 6vw, 80px);
    display: grid;
    place-items: center;
    overflow: hidden;
    background: radial-gradient(circle at 20% 20%, rgba(4,251,195,.28), transparent 50%),
                radial-gradient(circle at 80% 0%, rgba(255,110,196,.2), transparent 50%),
                linear-gradient(135deg, var(--bg-2), var(--bg-3));
  }
  .canvas::before,
  .canvas::after {
    content: "";
    position: absolute;
    inset: -30%;
    background: conic-gradient(from 120deg, rgba(255,255,255,.08), rgba(4,251,195,.05), transparent 65%);
    filter: blur(140px);
    animation: orbit 60s linear infinite;
    opacity: .65;
    pointer-events: none;
  }
  .canvas::after {
    animation-duration: 95s;
    animation-direction: reverse;
    mix-blend-mode: screen;
    opacity: .45;
  }
  .noise {
    position: absolute;
    inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.75' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='160' height='160' filter='url(%23n)' opacity='.25'/%3E%3C/svg%3E");
    opacity: .35;
    mix-blend-mode: soft-light;
    animation: noiseShift 18s steps(6) infinite;
    pointer-events: none;
  }
  .card {
    width: min(920px, 94vw);
    border-radius: 42px;
    padding: clamp(30px, 5vw, 64px);
    border: 1.2px solid var(--card-border);
    background: var(--card-bg);
    backdrop-filter: blur(36px);
    box-shadow: 0 35px 120px rgba(1,4,15,.85);
    display: grid;
    gap: 42px;
    z-index: 1;
  }
  .eyebrow {
    font-size: 13px;
    letter-spacing: .28em;
    text-transform: uppercase;
    color: var(--text-muted);
  }
  .signature h2 {
    font-size: clamp(36px, 6vw, 72px);
    margin: 10px 0 16px;
    line-height: 1.05;
    letter-spacing: -.02em;
    animation: breathe 6s ease-in-out infinite;
  }
  .signature h2 .ink {
    background: linear-gradient(120deg, var(--ink-1), var(--ink-3));
    -webkit-background-clip: text;
    color: transparent;
  }
  .tagline {
    font-size: 18px;
    color: rgba(255,255,255,.75);
    margin: 0 0 24px;
    max-width: 60ch;
    line-height: 1.6;
  }
  .badges {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }
  .badges span {
    padding: 9px 16px;
    border-radius: 999px;
    border: 1px solid rgba(255,255,255,.18);
    background: rgba(255,255,255,.06);
    font-size: 12px;
    text-transform: uppercase;
    letter-spacing: .08em;
    animation: floatChip 7s ease-in-out infinite;
  }
  .badges span:nth-child(2) { animation-delay: .5s; }
  .badges span:nth-child(3) { animation-delay: 1s; }
  .badges span:nth-child(4) { animation-delay: 1.5s; }
  .manifest h3,
  .contact h3 {
    margin: 0 0 12px;
    font-size: 16px;
    letter-spacing: .3em;
    text-transform: uppercase;
    color: rgba(255,255,255,.6);
  }
  .manifest ul {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    gap: 16px;
  }
  .manifest li {
    position: relative;
    padding-left: 32px;
    font-size: 17px;
    line-height: 1.6;
  }
  .manifest li::before {
    content: "✶";
    position: absolute;
    left: 0;
    color: var(--ink-2);
    animation: sparkle 3.8s linear infinite;
  }
  .contact {
    border-top: 1px dashed rgba(255,255,255,.16);
    padding-top: 28px;
    display: grid;
    gap: 16px;
  }
  .contact p {
    margin: 0;
    color: rgba(255,255,255,.78);
  }
  .contact-links {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
  }
  .contact-links a {
    padding: 12px 20px;
    border-radius: 999px;
    border: 1px solid transparent;
    background: rgba(255,255,255,.08);
    text-decoration: none;
    font-weight: 600;
    letter-spacing: .03em;
    position: relative;
    overflow: hidden;
    transition: transform .3s ease, border .3s ease, background .3s ease;
  }
  .contact-links a::after {
    content: "";
    position: absolute;
    inset: -45% -20%;
    background: radial-gradient(circle, rgba(255,255,255,.95), transparent 60%);
    opacity: 0;
    transform: scale(.4);
    mix-blend-mode: difference;
    transition: opacity .35s ease, transform .35s ease;
  }
  .contact-links a:hover {
    transform: translateY(-4px) scale(1.02);
    border-color: rgba(255,255,255,.32);
    background: rgba(255,255,255,.18);
  }
  .contact-links a:hover::after {
    opacity: .85;
    transform: scale(1);
  }
  .footnote {
    font-size: 13px;
    color: rgba(255,255,255,.5);
    letter-spacing: .1em;
  }
  @keyframes orbit {
    to { transform: rotate(360deg); }
  }
  @keyframes noiseShift {
    to { transform: translate3d(6%, -8%, 0); }
  }
  @keyframes breathe {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-8px); }
  }
  @keyframes floatChip {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-6px); }
  }
  @keyframes sparkle {
    0% { opacity: .3; }
    40% { opacity: 1; }
    100% { opacity: .3; }
  }
  @media (max-width: 640px) {
    .card { border-radius: 30px; padding: 26px; }
    .contact-links { flex-direction: column; }
  }
</style>

<div class="canvas">
  <div class="noise" aria-hidden="true"></div>
  <main class="card">
    <section class="signature">
      <p class="eyebrow">freestyle frontend · audio reactive visuals</p>
      <h2>我把 code 当即兴现场，<span class="ink">让失控成为质感</span>。</h2>
      <p class="tagline">这不是典型的个人介绍，而是一个保持实验状态的发射台：用噪点、模糊、渐变和奇怪的留白，记录我与 Web </p>
      <div class="badges">
        <span>Generative UI</span>
        <span>Glitch Animations</span>
        <span>Design Systems gone rogue</span>
        <span>Signal storytelling</span>
      </div>
    </section>
    <section class="manifest">
      <h3>ongoing signals</h3>
      <ul>
        <li>欢迎你闯入这个半成品工作室</li>
        <li>目前仍在学习，抽时间读书、写代码。</li>
        <li>很爱一句话“不积跬步，无以至千里；不积小流，无以成江海。” </li>
      </ul>
    </section>
    <section class="contact">
      <h3>contact / ping me</h3>
      <p>任何合作、实验 demo，或只是分享一个奇怪的灵感，都可以直接扔信号给我。</p>
      <div class="contact-links">
        <a href="mailto:751024981@qq.com">Signal · Email</a>
        <a href="https://github.com/Mo-Cheng24" target="_blank">Noise · GitHub</a>
      </div>
      <p class="footnote">偏爱 async，不过若你想来一场即兴对话，我也乐意。</p>
    </section>
  </main>
</div>

