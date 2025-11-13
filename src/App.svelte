<script>
  import { onMount, onDestroy } from 'svelte';
  import { gsap } from 'gsap';

  // === DOM & State ===
  let canvas, ctx, leftHalf = null, rightHalf = null;
  let isOpened = false;
  let animationId;
  let particles = [];
  let width, height;
  let activeTab = 'profile';
  let soundEnabled = true;
  let audioCtx;

  // === Tabs (icons only) ===
  const tabs = [
    { id: 'profile', label: 'Neural Profile', icon: 'Lightning' },
    { id: 'experience', label: 'Training Data', icon: 'Brain' },
    { id: 'skills', label: 'Model Architecture', icon: 'Laptop' },
    { id: 'projects', label: 'Deployment History', icon: 'Rocket' },
    { id: 'contact', label: 'Contact Protocol', icon: 'Satellite' }
  ];

  // === Web Audio Setup ===
  function initAudio() {
    if (typeof window !== 'undefined' && !audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }
  }

  function unlockAudio() {
    if (audioCtx && audioCtx.state !== 'running') {
      audioCtx.resume();
    }
  }

  function playSound(fn) {
    if (!soundEnabled || !audioCtx) return;
    unlockAudio();
    fn(audioCtx);
  }

  // === SOUNDS ===
  function playUplink() {
    playSound(ctx => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(60, ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(20, ctx.currentTime + 0.3);
      gain.gain.setValueAtTime(1.2, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.8);
      osc.connect(gain).connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 0.8);

      setTimeout(() => {
        const chime = ctx.createOscillator();
        const g = ctx.createGain();
        chime.type = 'square';
        chime.frequency.setValueAtTime(1200, ctx.currentTime);
        chime.frequency.exponentialRampToValueAtTime(800, ctx.currentTime + 0.2);
        g.gain.setValueAtTime(0.6, ctx.currentTime);
        g.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.4);
        chime.connect(g).connect(ctx.destination);
        chime.start();
        chime.stop(ctx.currentTime + 0.4);
      }, 200);
    });
  }

  function playFlash() {
    playSound(ctx => {
      const noise = ctx.createBufferSource();
      const buffer = ctx.createBuffer(1, ctx.sampleRate * 0.8, ctx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < buffer.length; i++) {
        data[i] = Math.random() * 2 - 1;
      }
      noise.buffer = buffer;

      const filter = ctx.createBiquadFilter();
      filter.type = 'bandpass';
      filter.frequency.setValueAtTime(800, ctx.currentTime);
      filter.frequency.exponentialRampToValueAtTime(200, ctx.currentTime + 0.8);

      const gain = ctx.createGain();
      gain.gain.setValueAtTime(0.4, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.8);

      noise.connect(filter).connect(gain).connect(ctx.destination);
      noise.start();
    });
  }

  function playUnlock() {
    playSound(ctx => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(1000, ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(1500, ctx.currentTime + 0.2);
      gain.gain.setValueAtTime(0.3, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3);
      osc.connect(gain).connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 0.3);
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

      setTimeout(() => {
        const sweep = ctx.createOscillator();
        const gg = ctx.createGain();
        sweep.type = 'sawtooth';
        sweep.frequency.setValueAtTime(300, ctx.currentTime);
        sweep.frequency.exponentialRampToValueAtTime(600, ctx.currentTime + 0.2);
        gg.gain.setValueAtTime(0.15, ctx.currentTime);
        gg.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.2);
        sweep.connect(gg).connect(ctx.destination);
        sweep.start();
        sweep.stop(ctx.currentTime + 0.2);
      }, 100);
    });
  }

  function playReset() {
    playSound(ctx => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(20, ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(60, ctx.currentTime + 0.5);
      gain.gain.setValueAtTime(0.8, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 1);
      osc.connect(gain).connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 1);
    });
  }

  let ambienceNode = null;
  function startAmbience() {
    if (!soundEnabled || ambienceNode) return;
    playSound(ctx => {
      const bufferSize = ctx.sampleRate * 45;
      const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
      const data = buffer.getChannelData(0);

      for (let i = 0; i < bufferSize; i++) {
        const t = i / ctx.sampleRate;
        const hum = Math.sin(t * 120 * Math.PI) * 0.03;
        const buzz = (Math.random() > 0.99 ? 1 : 0) * 0.08;
        const chirp = Math.sin(t * 2000 * Math.PI) * (Math.random() > 0.995 ? 0.1 : 0);
        data[i] = hum + buzz + chirp;
      }

      ambienceNode = ctx.createBufferSource();
      ambienceNode.buffer = buffer;
      ambienceNode.loop = true;

      const gain = ctx.createGain();
      gain.gain.value = 0.12;

      ambienceNode.connect(gain).connect(ctx.destination);
      ambienceNode.start();
    });
  }

  function stopAmbience() {
    if (ambienceNode) {
      ambienceNode.stop();
      ambienceNode = null;
    }
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

  function playGong() {
    playSound(ctx => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(80, ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(40, ctx.currentTime + 1.5);
      gain.gain.setValueAtTime(0.6, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 2);
      osc.connect(gain).connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 2);
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

    document.body.addEventListener('click', startAmbience, { once: true });

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
    stopAmbience();
  });

  // === Canvas ===
  function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
  }

  function animate() {
    animationId = requestAnimationFrame(animate);
    ctx.fillStyle = 'rgba(0, 0, 0, 0.07)';
    ctx.fillRect(0, 0, width, height);

    for (let i = particles.length - 1; i >= 0; i--) {
      const p = particles[i];
      p.x += p.vx;
      p.y += p.vy;
      p.vy += 0.08;
      p.life -= p.decay;
      if (p.life <= 0) {
        particles.splice(i, 1);
        continue;
      }
      ctx.globalAlpha = p.life;
      ctx.shadowColor = p.color;
      ctx.shadowBlur = 30;
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
      ctx.fill();
      ctx.globalAlpha = 1;
      ctx.shadowBlur = 0;
    }
  }

  // === Interactions ===
  function openGlyph() {
    if (isOpened) return;
    isOpened = true;

    for (let i = 0; i < 60; i++) {
      particles.push({
        x: width / 2,
        y: height / 2,
        vx: (Math.random() - 0.5) * 10,
        vy: (Math.random() - 0.5) * 10,
        size: Math.random() * 4 + 2,
        color: `hsl(${180 + Math.random() * 60}, 100%, 70%)`,
        life: 1,
        decay: 0.015 + Math.random() * 0.01
      });
    }

    playUplink();
    setTimeout(playFlash, 100);

    gsap.killTweensOf([leftHalf, rightHalf]);

    const light = document.createElement('div');
    light.style.cssText = `position:fixed;top:50%;left:50%;width:10px;height:10px;background:#fff;border-radius:50%;transform:translate(-50%,-50%);box-shadow:0 0 100px 50px #fff;z-index:999;`;
    document.body.appendChild(light);

    gsap.to(light, { scale: 200, opacity: 0, duration: 0.8, ease: "power2.out", onComplete: () => light.remove() });
    gsap.to(leftHalf, { x: -width * 0.6, duration: 1.2, ease: "power2.inOut" });
    gsap.to(rightHalf, { x: width * 0.6, duration: 1.2, ease: "power2.inOut" });

    setTimeout(() => {
      gsap.fromTo(".title", { scale: 0.5, opacity: 0, rotation: -15 }, { scale: 1, opacity: 1, rotation: 0, duration: 1.2, ease: "elastic.out(1, 0.3)" });
      gsap.fromTo(".tab-content", { opacity: 0, y: 30 }, { opacity: 1, y: 0, duration: 0.8, stagger: 0.1, ease: "power3.out", onStart: playUnlock });
    }, 800);
  }

  function resetRitual() {
    particles = [];
    isOpened = false;
    playReset();
    gsap.to(leftHalf, { x: 0, duration: 1, ease: "elastic.out(1, 0.5)" });
    gsap.to(rightHalf, { x: 0, duration: 1, ease: "elastic.out(1, 0.5)" });
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
    if (!soundEnabled) stopAmbience();
    else startAmbience();
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
      <div class="glyph-subtitle">Tim B.C.</div>
    </div>
  {:else}
    <div class="content-wrapper">
      <h1 class="title">TIMOTHY CHAPPELL • AI ENGINEER</h1>

      <div class="tabs">
        {#each tabs as tab}
          <button class="tab-btn" class:active={activeTab === tab.id} on:click={() => switchTab(tab.id)} on:mouseenter={playTick}>
            {tab.icon} {tab.label}
          </button>
        {/each}
      </div>

      <!-- NEURAL PROFILE -->
      <div class="tab-content" id="profile" style="display: {activeTab === 'profile' ? 'block' : 'none'}">
        <section class="resume-section">
          <h2>Lightning NEURAL PROFILE</h2>
          <p class="intro">AI Prompt Engineer & Creative Technologist specializing in generative systems, music production AI, and multi-model optimization. 3+ years architecting prompts across GPT, Mistral, Grok, DeepSeek, and OpenRouter platforms.</p>
          <div class="highlight-box">
            <p><strong>Core Competency:</strong> Translating creative vision into precise AI instructions that generate professional-grade outputs across audio, visual, and text domains.</p>
          </div>
          <p class="tagline">"Building the bridge between human creativity and machine intelligence."</p>
        </section>
      </div>

      <!-- [Other tabs — same as before] -->
      <!-- TRAINING DATA, SKILLS, PROJECTS, CONTACT — unchanged -->

      <div class="footer-glyph" on:click={resetRitual} on:mouseenter={playGong}>
        ⊰ΨΩ≋⊱
        <small>Tim B.C. - Reset Portal</small>
      </div>
    </div>
  {/if}
</main>

<style>
  :global(body) { margin:0; background:#000; font-family:'Courier New',monospace; color:#00ffff; overflow-x:hidden; }
  main { position:relative; min-height:100vh; z-index:2; }

  .sound-toggle {
    position:fixed; top:20px; right:20px; z-index:1000;
    background:none; border:1px solid #00ffff; color:#00ffff;
    padding:8px 12px; font-size:1.5rem; cursor:pointer;
    transition:all 0.3s; border-radius:4px;
  }
  .sound-toggle:hover { background:rgba(0,255,255,0.2); }

  .glyph-container { position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); display:flex; font-size:8rem; cursor:pointer; user-select:none; z-index:10; }
  .glyph-half { color:#00ffff; text-shadow:0 0 30px #00ffff,0 0 60px #00ffff; }
  .left { margin-right:-1rem; }
  .right { margin-left:-1rem; }
  .glyph-subtitle { position:absolute; bottom:-40px; left:50%; transform:translateX(-50%); font-size:1.2rem; color:#00ffff; letter-spacing:0.2rem; text-shadow:0 0 10px #00ffff; }

  .content-wrapper { max-width:900px; margin:0 auto; padding:4rem 2rem; }
  .title { font-size:2.2rem; text-align:center; margin-bottom:2rem; letter-spacing:0.3rem; text-shadow:0 0 20px #00ffff; }
  .tabs { display:flex; flex-wrap:wrap; gap:1rem; justify-content:center; margin:2rem 0; }
  .tab-btn { background:none; border:1px solid #00ffff; color:#00ffff; padding:0.8rem 1.2rem; font-family:'Courier New'; cursor:pointer; text-transform:uppercase; font-size:0.9rem; transition:all 0.3s ease; }
  .tab-btn:hover { background:rgba(0,255,255,0.1); }
  .tab-btn.active { background:#00ffff; color:#000; }
  .tab-content { display:none; }

  .resume-section { margin:2rem 0; padding:2rem; border:2px solid #00ffff; border-radius:10px; background:rgba(0,255,255,0.05); }
  .resume-section h2 { font-size:1.8rem; text-align:center; margin-bottom:1.5rem; color:#00ffff; text-shadow:0 0 15px #00ffff; }
  .highlight-box { background:rgba(0,255,255,0.1); border-left:4px solid #00ffff; padding:1rem; margin:1.5rem 0; }
  .tagline { text-align:center; font-style:italic; color:#0ff; margin-top:1.5rem; font-size:1.1rem; }

  .footer-glyph { text-align:center; font-size:3rem; margin-top:4rem; cursor:pointer; animation:pulse 2s infinite; color:#00ffff; }
  .footer-glyph small { font-size:0.8rem; display:block; margin-top:0.5rem; color:#0ff; }
  @keyframes pulse { 0%,100% { opacity:0.7; } 50% { opacity:1; } }
</style>
