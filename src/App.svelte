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
    { id: 'profile', label: 'Neural Profile', icon: '' },
    { id: 'experience', label: 'Training Data', icon: '' },
    { id: 'skills', label: 'Model Architecture', icon: '' },
    { id: 'projects', label: 'Deployment History', icon: '' },
    { id: 'contact', label: 'Contact Protocol', icon: '' }
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
          <p class="intro
