<script>
  import { onMount, onDestroy } from 'svelte';
  import { gsap } from 'gsap';
  // === State ===
  let canvas, ctx, leftHalf = null, rightHalf = null;
  let isOpened = false;
  let animationId;
  let width, height;
  let activeTab = 'profile';
  let soundEnabled = true;
  let audioCtx;
  // === Tabs ===
  const tabs = [
    { id: 'profile', label: 'Neural Profile', icon: 'Lightning' },
    { id: 'experience', label: 'Training Data', icon: 'Brain' },
    { id: 'skills', label: 'Model Architecture', icon: 'Laptop' },
    { id: 'projects', label: 'Deployment History', icon: 'Rocket' },
    { id: 'contact', label: 'Contact Protocol', icon: 'Satellite' }
  ];
  // === Audio Setup ===
  function initAudio() {
    if (typeof window !== 'undefined' && !audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }
  }
  function unlockAudio() {
    if (audioCtx && audioCtx.state !== 'running') audioCtx.resume();
  }
  function playSound(fn) {
    if (!soundEnabled || !audioCtx) return;
    unlockAudio();
    fn(audioCtx);
  }
  // === SOUNDS ===
  function playDoorOpen() {
    playSound(ctx => {
      const hiss = ctx.createBufferSource();
      const buffer = ctx.createBuffer(1, ctx.sampleRate * 1.2, ctx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < buffer.length; i++) {
        data[i] = (Math.random() - 0.5) * 0.3 * Math.exp(-i / (ctx.sampleRate * 0.8));
      }
      hiss.buffer = buffer;
      const filter = ctx.createBiquadFilter();
      filter.type = 'lowpass';
      filter.frequency.setValueAtTime(800, ctx.currentTime);
      filter.frequency.exponentialRampToValueAtTime(200, ctx.currentTime + 1.2);
      const gain = ctx.createGain();
      gain.gain.setValueAtTime(0.4, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 1.2);
      hiss.connect(filter).connect(gain).connect(ctx.destination);
      hiss.start();
      setTimeout(() => {
        const click = ctx.createOscillator();
        const g = ctx.createGain();
        click.type = 'square';
        click.frequency.value = 600;
        g.gain.setValueAtTime(0.4, ctx.currentTime);
        g.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1);
        click.connect(g).connect(ctx.destination);
        click.start();
        click.stop(ctx.currentTime + 0.1);
      }, 300);
    });
  }
  function playDoorClose() {
    playSound(ctx => {
      const hiss = ctx.createBufferSource();
      const buffer = ctx.createBuffer(1, ctx.sampleRate * 1.0, ctx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < buffer.length; i++) {
        data[i] = (Math.random() - 0.5) * 0.25 * Math.exp(-i / (ctx.sampleRate * 0.6));
      }
      hiss.buffer = buffer;
      const filter = ctx.createBiquadFilter();
      filter.type = 'lowpass';
      filter.frequency.setValueAtTime(600, ctx.currentTime);
      filter.frequency.exponentialRampToValueAtTime(100, ctx.currentTime + 1.0);
      const gain = ctx.createGain();
      gain.gain.setValueAtTime(0.35, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 1.0);
      hiss.connect(filter).connect(gain).connect(ctx.destination);
      hiss.start();
      setTimeout(() => {
        const click = ctx.createOscillator();
        const g = ctx.createGain();
        click.type = 'square';
        click.frequency.value = 500;
        g.gain.setValueAtTime(0.35, ctx.currentTime);
        g.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.08);
        click.connect(g).connect(ctx.destination);
        click.start();
        click.stop(ctx.currentTime + 0.08);
      }, 200);
    });
  }
  function playTabSwitch() {
    playSound(ctx => {
      const click = ctx.createOscillator();
      const g = ctx.createGain();
      click.type = 'square';
      click.frequency.value = 800;
      g.gain.setValueAtTime(0.3, ctx.currentTime);
      g.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1);
      click.connect(g).connect(ctx.destination);
      click.start();
      click.stop(ctx.currentTime + 0.1);
    });
  }
  function playTick() {
    playSound(ctx => {
      const osc = ctx.createOscillator();
      const g = ctx.createGain();
      osc.type = 'square';
      osc.frequency.value = 1200;
      g.gain.setValueAtTime(0.15, ctx.currentTime);
      g.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.05);
      osc.connect(g).connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 0.05);
    });
  }
  // === Lifecycle ===
  onMount(() => {
    initAudio();
    canvas = document.getElementById('bg-canvas');
    ctx = canvas.getContext('2d');
    resize();
    window.addEventListener('resize', resize);
    animate();
    if (leftHalf && rightHalf) {
      gsap.to([leftHalf, rightHalf], {
        x: () => Math.random() * 6 - 3,
        y: () => Math.random() * 6 - 3,
        duration: 0.1,
        repeat: -1,
        yoyo: true,
        ease: "none"
      });
      gsap.to([leftHalf, rightHalf], {
        textShadow: "0 0 60px #00ffff, 0 0 100px #00ffff",
        duration: 1.2,
        repeat: -1,
        yoyo: true,
        ease: "power2.inOut"
      });
    }
  });
  onDestroy(() => {
    if (animationId) cancelAnimationFrame(animationId);
    window.removeEventListener('resize', resize);
  });
  // === Canvas ===
  function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
  }
  function animate() {
    animationId = requestAnimationFrame(animate);
    ctx.fillStyle = 'rgba(0, 0, 0, 0.05)';
    ctx.fillRect(0, 0, width, height);
  }
  // === DOOR OPEN ===
  function openGlyph() {
    if (isOpened) return;
    isOpened = true;
    playDoorOpen();
    gsap.killTweensOf([leftHalf, rightHalf]);
    gsap.to(leftHalf, { x: -width * 0.55, duration: 1.4, ease: "power2.inOut" });
    gsap.to(rightHalf, { x: width * 0.55, duration: 1.4, ease: "power2.inOut" });
    setTimeout(() => {
      gsap.fromTo(".title", { scale: 0.8, opacity: 0 }, { scale: 1, opacity: 1, duration: 1.0, ease: "elastic.out(1, 0.4)" });
      gsap.fromTo(".tab-content", { opacity: 0, y: 20 }, { opacity: 1, y: 0, duration: 0.8, stagger: 0.1, ease: "power3.out" });
    }, 1000);
  }
  function resetRitual() {
    playDoorClose();
    isOpened = false;
    gsap.to(leftHalf, { x: 0, duration: 1.2, ease: "power2.inOut" });
    gsap.to(rightHalf, { x: 0, duration: 1.2, ease: "power2.inOut" });
  }
  function switchTab(tabId) {
    if (activeTab === tabId) return;
    const oldContent = document.querySelector(`#${activeTab}`);
    const newContent = document.querySelector(`#${tabId}`);
    playTabSwitch();
    gsap.to(oldContent, { x: -100, opacity: 0, duration: 0.4, ease: "power2.in" });
    gsap.fromTo(newContent, { x: 100, opacity: 0 }, { x: 0, opacity: 1, duration: 0.4, ease: "power2.out", delay: 0.2 });
    activeTab = tabId;
  }
  function toggleSound() {
    soundEnabled = !soundEnabled;
  }
</script>

<!-- MUTE TOGGLE -->
<button class="sound-toggle" on:click={toggleSound}>
  {soundEnabled ? 'Speaker' : 'Speaker with Slash'}
</button>
<canvas id="bg-canvas" style="position:fixed;top:0;left:0;z-index:1;"></canvas>
<main>
  {#if !isOpened}
    <div class="glyph-container">
      <div bind:this={leftHalf} class="glyph-half left" on:click={openGlyph} on:mouseenter={playTick}>
        ⊰ΨΩ
      </div>
      <div bind:this={rightHalf} class="glyph-half right" on:click={openGlyph} on:mouseenter={playTick}>
        ≋⊱
      </div>
    </div>
  {:else}
    <div class="content-wrapper">
      <h1 class="title">TIMOTHY CHAPPELL • AI ENGINEER</h1>
      <div class="tabs">
        {#each tabs as tab}
          <button
            class="tab-btn"
            class:active={activeTab === tab.id}
            on:click={() => switchTab(tab.id)}
            on:mouseenter={playTick}
          >
            {tab.label.toUpperCase()}
          </button>
        {/each}
      </div>
      <!-- NEURAL PROFILE -->
      <div class="tab-content" id="profile" style="display: {activeTab === 'profile' ? 'block' : 'none'}">
        <section class="resume-panel">
          <h2>NEURAL PROFILE</h2>
          <p class="intro">
            AI Prompt Engineer & Creative Technologist specializing in generative systems, music production AI, and multi-model optimization. 3+ years architecting prompts across GPT, Mistral, Grok, DeepSeek, and OpenRouter platforms.
          </p>
          <div class="highlight-box">
            <p><strong>Core Competency:</strong> Translating creative vision into precise AI instructions that generate professional-grade outputs across audio, visual, and text domains.</p>
          </div>
          <p class="tagline">"Building the bridge between human creativity and machine intelligence."</p>
        </section>
      </div>
      <!-- TRAINING DATA -->
      <div class="tab-content" id="experience" style="display: {activeTab === 'experience' ? 'block' : 'none'}">
        <section class="resume-panel">
          <h2>TRAINING DATA</h2>
          <div class="experience-item">
            <h3>AI Prompt Engineer (Independent)</h3>
            <p class="date">2021 - Present</p>
            <ul>
              <li>Architected prompts for GPT-3.5 through GPT-4, Claude, Mistral, Grok, DeepSeek, and OpenRouter integrations</li>
              <li>Specialized in Suno AI music generation - mastering style fusion, lyrical coherence, and audio quality optimization</li>
              <li>Developed systematic approaches to prompt engineering: iterative refinement, A/B testing, and model-specific tuning</li>
              <li>Created production-ready outputs across multiple domains: music, copywriting, visual art direction, and technical documentation</li>
            </ul>
          </div>
          <div class="experience-item">
            <h3>Music Producer & Audio Engineer</h3>
            <p class="date">2018 - Present</p>
            <ul>
              <li>Produced electronic, industrial, and experimental music with focus on sound design and atmosphere</li>
              <li>Expert in DAW operation (BandLab, audio mastering tools), MIDI composition, and audio engineering workflows</li>
              <li>Integrated AI-generated elements with traditional production techniques</li>
              <li>Published music across streaming platforms with professional artwork and branding</li>
            </ul>
          </div>
          <div class="experience-item">
            <h3>Creative Technologist</h3>
            <p class="date">2020 - Present</p>
            <ul>
              <li>Developed digital products: custom stickers, templates, and visual assets</li>
              <li>Web deployment experience with Vercel, GitHub, and version control</li>
              <li>Active learner in Python, JavaScript, and web development frameworks</li>
            </ul>
          </div>
        </section>
      </div>
      <!-- MODEL ARCHITECTURE -->
      <div class="tab-content" id="skills" style="display: {activeTab === 'skills' ? 'block' : 'none'}">
        <section class="resume-panel">
          <h2>MODEL ARCHITECTURE</h2>
          <table class="skills-table">
            <thead>
              <tr>
                <th>AI Platforms</th>
                <th>Technical Skills</th>
                <th>Creative Expertise</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>GPT-4, GPT-3.5, Claude</td>
                <td>Prompt Engineering & Optimization</td>
                <td>Music Production (Electronic/Industrial)</td>
              </tr>
              <tr>
                <td>Mistral AI</td>
                <td>Audio Production & Mastering</td>
                <td>Digital Art & Design</td>
              </tr>
              <tr>
                <td>Grok, DeepSeek</td>
                <td>Sound Design & MIDI</td>
                <td>Creative Writing & Storytelling</td>
              </tr>
              <tr>
                <td>OpenRouter integrations</td>
                <td>GitHub & Version Control</td>
                <td>Branding & Visual Identity</td>
              </tr>
              <tr>
                <td>Suno (expert-level)</td>
                <td>Python (learning)</td>
                <td>Content Strategy</td>
              </tr>
              <tr>
                <td>Midjourney</td>
                <td>HTML/CSS/JavaScript</td>
                <td></td>
              </tr>
              <tr>
                <td></td>
                <td>Web Deployment (Vercel, Netlify)</td>
                <td></td>
              </tr>
            </tbody>
          </table>
        </section>
      </div>
      <!-- DEPLOYMENT HISTORY -->
      <div class="tab-content" id="projects" style="display: {activeTab === 'projects' ? 'block' : 'none'}">
        <section class="resume-panel">
          <h2>DEPLOYMENT HISTORY</h2>
          <div class="project-item">
            <h3>Suno AI Music Production System</h3>
            <p>Developed comprehensive prompting methodology for generating professional-quality music across genres. Mastered techniques for style fusion, tonal control, and lyrical coherence that consistently produce release-ready tracks.</p>
          </div>
          <div class="project-item">
            <h3>Multi-Model AI Workflow</h3>
            <p>Created integrated workflows leveraging strengths of multiple AI platforms simultaneously - using GPT for lyrical generation, Suno for audio, Midjourney for artwork, and BandLab for final mastering.</p>
          </div>
          <div class="project-item">
            <h3>Digital Product Creation</h3>
            <p>Designed and published custom stickers, templates, and visual assets. Managed full pipeline from concept to marketplace deployment.</p>
          </div>
          <div class="project-item">
            <h3>Web Development Projects</h3>
            <p>Built and deployed websites using GitHub, Vercel, and modern web frameworks. This resume site is a live example of cyberpunk-themed web design with GSAP animations.</p>
          </div>
        </section>
      </div>
      <!-- CONTACT PROTOCOL -->
      <div class="tab-content" id="contact" style="display: {activeTab === 'contact' ? 'block' : 'none'}">
        <section class="resume-panel contact-section">
          <h2>CONTACT PROTOCOL</h2>
          <div class="contact-grid">
            <div class="contact-item">
              <div>
                <p class="contact-label">Email</p>
                <a href="mailto:tbchappell803@gmail.com" class="contact-link">tbchappell803@gmail.com</a>
              </div>
            </div>
            <div class="contact-item">
              <div>
                <p class="contact-label">Phone</p>
                <a href="tel:8033314504" class="contact-link">803-331-4504</a>
              </div>
            </div>
            <div class="contact-item">
              <div>
                <p class="contact-label">LinkedIn</p>
                <a href="https://www.linkedin.com/in/timothy-chappell-bb311b390" target="_blank" class="contact-link">
                  timothy-chappell-bb311b390
                </a>
              </div>
            </div>
          </div>
          <div class="availability">
            <p><strong>Remote Work:</strong> Available for remote opportunities worldwide</p>
            <p><strong>Availability:</strong> Immediate start</p>
          </div>
        </section>
      </div>
      <div class="footer-glyph" on:click={resetRitual}>
        ⊰ΨΩ≋⊱
      </div>
    </div>
  {/if}
</main>
<style>
  :global(body) {
    margin: 0;
    background: #000;
    font-family: 'Courier New', monospace;
    color: #00ffff;
    overflow-x: hidden;
  }
  main { position: relative; min-height: 100vh; z-index: 2; }
  .sound-toggle {
    position: fixed; top: 20px; right: 20px; z-index: 1000;
    background: none; border: 1px solid #00ffff; color: #00ffff;
    padding: 8px 12px; font-size: 1.5rem; cursor: pointer;
    border-radius: 4px;
  }
  .glyph-container {
    position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
    display: flex; font-size: 8rem; cursor: pointer; user-select: none; z-index: 10;
  }
  .glyph-half { color: #00ffff; text-shadow: 0 0 30px #00ffff, 0 0 60px #00ffff; }
  .left { margin-right: -1rem; }
  .right { margin-left: -1rem; }
  .content-wrapper { max-width: 900px; margin: 0 auto; padding: 4rem 2rem; }
  .title { font-size: 2.2rem; text-align: center; margin-bottom: 2rem; letter-spacing: 0.3rem; text-shadow: 0 0 20px #00ffff; text-transform: uppercase; }
  .tabs { display: flex; flex-wrap: wrap; gap: 1rem; justify-content: center; margin: 2rem 0; }
  .tab-btn {
    background: none; border: 2px solid #00ffff; color: #00ffff;
    padding: 0.8rem 1.5rem; font-family: 'Courier New'; cursor: pointer;
    text-transform: uppercase; font-size: 0.9rem; border-radius: 50px;
    transition: all 0.3s ease; min-width: 180px;
  }
  .tab-btn:hover { background: rgba(0,255,255,0.1); box-shadow: 0 0 15px rgba(0,255,255,0.5); }
  .tab-btn.active { background: #00ffff; color: #000; box-shadow: 0 0 20px #00ffff, 0 0 40px #00ffff; }
  .tab-content { display: none; margin-top: 2rem; }
  .resume-panel {
    background: rgba(0, 10, 15, 0.8);
    border: 2px solid #00ffff;
    border-radius: 15px;
    padding: 2.5rem;
    margin: 2rem 0;
    box-shadow: 0 0 20px rgba(0,255,255,0.3), 0 0 40px rgba(0,255,255,0.1), inset 0 0 20px rgba(0,255,255,0.05);
  }
  .resume-panel h2 { font-size: 1.8rem; text-align: center; margin-bottom: 1.5rem; color: #00ffff; text-shadow: 0 0 15px #00ffff; text-transform: uppercase; }
  .intro { font-size: 1.1rem; line-height: 1.8; margin-bottom: 1.5rem; text-align: center; }
  .highlight-box {
    background: rgba(0,255,255,0.1);
    border-left: 4px solid #00ffff;
    padding: 1.2rem;
    margin: 1.5rem 0;
    border-radius: 0 8px 8px 0;
    box-shadow: 0 0 10px rgba(0,255,255,0.2);
  }
  .tagline { text-align: center; font-style: italic; color: #0ff; margin-top: 2rem; font-size: 1.1rem; text-shadow: 0 0 10px #00ffff; }
  .experience-item { margin: 2rem 0; padding: 1.5rem; background: rgba(0,255,255,0.03); border-left: 3px solid #00ffff; }
  .experience-item h3 { color: #00ffff; margin-bottom: 0.5rem; }
  .date { color: #0ff; font-size: 0.9rem; margin-bottom: 1rem; opacity: 0.8; }
  .experience-item ul { margin-left: 1.5rem; }
  .experience-item li { margin: 0.8rem 0; line-height: 1.6; }
  .skills-table { width: 100%; border-collapse: collapse; margin-top: 2rem; }
  .skills-table th { border-bottom: 2px solid #00ffff; padding: 1rem; text-align: left; color: #00ffff; text-shadow: 0 0 10px #00ffff; }
  .skills-table td { padding: 0.5rem; vertical-align: top; }
  .project-item { margin: 1.5rem 0; padding: 1.5rem; background: rgba(0,255,255,0.05); border: 1px solid #00ffff; border-radius: 5px; }
  .project-item h3 { color: #00ffff; margin-bottom: 0.8rem; }
  .contact-section { text-align: center; }
  .contact-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 2rem; margin: 2rem 0; text-align: left; }
  
  /* --- MODIFIED CODE START --- */
  .contact-item { 
    display: flex; 
    align-items: center; 
    gap: 1rem; 
    padding: 1rem; 
    background: rgba(0,255,255,0.05); 
    border: 1px solid #00ffff; 
    border-radius: 5px; 
    overflow: hidden; /* Prevents content from spilling out */
  }
  .contact-label { font-size: 0.9rem; color: #0ff; opacity: 0.8; margin-bottom: 0.3rem; }
  .contact-link { 
    color: #00ffff; 
    text-decoration: none; 
    font-weight: bold; 
    word-wrap: break-word; /* Ensures long text like emails/URLs wrap */
    overflow-wrap: break-word; /* Modern standard for word-wrap */
  }
  /* --- MODIFIED CODE END --- */

  .contact-link:hover { text-decoration: underline; }
  .availability { margin-top: 2rem; padding: 1.5rem; background: rgba(0,255,255,0.1); border: 2px solid #00ffff; border-radius: 5px; }
  .availability p { margin: 0.8rem 0; font-size: 1.1rem; }
  .footer-glyph { text-align: center; font-size: 4rem; margin-top: 5rem; cursor: pointer; color: #00ffff; text-shadow: 0 0 30px #00ffff; }
</style>
