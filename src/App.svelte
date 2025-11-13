<script>
  import { onMount, onDestroy } from 'svelte';
  import { gsap } from 'gsap';

  let canvas;
  let ctx;
  let leftHalf;
  let rightHalf;
  let isOpened = false;
  let animationId;
  let particles = [];
  let width, height;

  // Resume tabs
  let activeTab = 'profile';
  const tabs = [
    { id: 'profile', label: 'Neural Profile', icon: '⚡' },
    { id: 'experience', label: 'Training Data', icon: '🧠' },
    { id: 'skills', label: 'Model Architecture', icon: '💻' },
    { id: 'projects', label: 'Deployment History', icon: '🚀' },
    { id: 'contact', label: 'Contact Protocol', icon: '📡' }
  ];

  onMount(() => {
    canvas = document.getElementById('bg-canvas');
    ctx = canvas.getContext('2d');
    resize();
    animate();

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
  });

  onDestroy(() => {
    if (animationId) cancelAnimationFrame(animationId);
  });

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

  function openGlyph() {
    if (isOpened) return;
    isOpened = true;

    gsap.killTweensOf([leftHalf, rightHalf]);

    const light = document.createElement('div');
    light.style.cssText = `
      position: fixed; top: 50%; left: 50%; width: 10px; height: 10px;
      background: #fff; border-radius: 50%; transform: translate(-50%, -50%);
      box-shadow: 0 0 100px 50px #fff; z-index: 999;
    `;
    document.body.appendChild(light);

    gsap.to(light, {
      scale: 200,
      opacity: 0,
      duration: 0.8,
      ease: "power2.out",
      onComplete: () => light.remove()
    });

    gsap.to(leftHalf, { x: -width * 0.6, duration: 1.2, ease: "power2.inOut" });
    gsap.to(rightHalf, { x: width * 0.6, duration: 1.2, ease: "power2.inOut" });

    setTimeout(() => {
      gsap.fromTo(".title", 
        { scale: 0.5, opacity: 0, rotation: -15 }, 
        { scale: 1, opacity: 1, rotation: 0, duration: 1.2, ease: "elastic.out(1, 0.3)" }
      );
      gsap.fromTo(".tab-content", 
        { opacity: 0, y: 30 }, 
        { opacity: 1, y: 0, duration: 0.8, stagger: 0.1, ease: "power3.out" }
      );
    }, 800);
  }

  function resetRitual() {
    particles = [];
    isOpened = false;
    gsap.to(leftHalf, { x: 0, duration: 1, ease: "elastic.out(1, 0.5)" });
    gsap.to(rightHalf, { x: 0, duration: 1, ease: "elastic.out(1, 0.5)" });
  }

  function switchTab(tabId) {
    if (activeTab === tabId) return;
    const oldContent = document.querySelector(`#${activeTab}`);
    const newContent = document.querySelector(`#${tabId}`);
    
    gsap.to(oldContent, { x: -100, opacity: 0, duration: 0.4, ease: "power2.in" });
    gsap.fromTo(newContent, 
      { x: 100, opacity: 0 }, 
      { x: 0, opacity: 1, duration: 0.4, ease: "power2.out", delay: 0.2 }
    );
    
    activeTab = tabId;
  }
</script>

<canvas id="bg-canvas" style="position: fixed; top: 0; left: 0; z-index: 1;"></canvas>

<main>
  {#if !isOpened}
    <div class="glyph-container">
      <div bind:this={leftHalf} class="glyph-half left" on:click={openGlyph}>
        ⚡⊰ΨΩ
      </div>
      <div bind:this={rightHalf} class="glyph-half right" on:click={openGlyph}>
        ≋⊱⚡
      </div>
    </div>
  {:else}
    <div class="content-wrapper">
      <h1 class="title">TIMOTHY CHAPPELL • AI ENGINEER</h1>
      
      <!-- TABS -->
      <div class="tabs">
        {#each tabs as tab}
          <button 
            class="tab-btn" 
            class:active={activeTab === tab.id}
            on:click={() => switchTab(tab.id)}
          >
            {tab.icon} {tab.label}
          </button>
        {/each}
      </div>

      <!-- NEURAL PROFILE -->
      <div class="tab-content" id="profile" style="display: {activeTab === 'profile' ? 'block' : 'none'}">
        <section class="resume-section">
          <h2>⚡ NEURAL PROFILE</h2>
          <p class="intro">AI Prompt Engineer & Creative Technologist specializing in generative systems, music production AI, and multi-model optimization. 3+ years architecting prompts across GPT, Mistral, Grok, DeepSeek, and OpenRouter platforms.</p>
          
          <div class="highlight-box">
            <p><strong>Core Competency:</strong> Translating creative vision into precise AI instructions that generate professional-grade outputs across audio, visual, and text domains.</p>
          </div>

          <p class="tagline">"Building the bridge between human creativity and machine intelligence."</p>
        </section>
      </div>

      <!-- TRAINING DATA -->
      <div class="tab-content" id="experience" style="display: {activeTab === 'experience' ? 'block' : 'none'}">
        <section class="resume-section">
          <h2>🧠 TRAINING DATA</h2>
          
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
        <section class="resume-section">
          <h2>💻 MODEL ARCHITECTURE</h2>
          
          <div class="skills-grid">
            <div class="skill-category">
              <h3>AI Platforms</h3>
              <ul class="skill-list">
                <li>GPT-4, GPT-3.5, Claude</li>
                <li>Mistral AI</li>
                <li>Grok, DeepSeek</li>
                <li>OpenRouter integrations</li>
                <li>Suno (expert-level)</li>
                <li>Midjourney</li>
              </ul>
            </div>

            <div class="skill-category">
              <h3>Technical Skills</h3>
              <ul class="skill-list">
                <li>Prompt Engineering & Optimization</li>
                <li>Audio Production & Mastering</li>
                <li>Sound Design & MIDI</li>
                <li>GitHub & Version Control</li>
                <li>Python (learning)</li>
                <li>HTML/CSS/JavaScript</li>
                <li>Web Deployment (Vercel, Netlify)</li>
              </ul>
            </div>

            <div class="skill-category">
              <h3>Creative Expertise</h3>
              <ul class="skill-list">
                <li>Music Production (Electronic/Industrial)</li>
                <li>Digital Art & Design</li>
                <li>Creative Writing & Storytelling</li>
                <li>Branding & Visual Identity</li>
                <li>Content Strategy</li>
              </ul>
            </div>
          </div>
        </section>
      </div>

      <!-- DEPLOYMENT HISTORY -->
      <div class="tab-content" id="projects" style="display: {activeTab === 'projects' ? 'block' : 'none'}">
        <section class="resume-section">
          <h2>🚀 DEPLOYMENT HISTORY</h2>
          
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
        <section class="resume-section contact-section">
          <h2>📡 CONTACT PROTOCOL</h2>
          
          <div class="contact-grid">
            <div class="contact-item">
              <span class="contact-icon">329
</span>
              <div>
                <p class="contact-label">Email</p>
                <a href="mailto:tbchappell803@gmail.com" class="contact-link">tbchappell803@gmail.com</a>
              </div>
            </div>

            <div class="contact-item">
              <span class="contact-icon">📱</span>
              <div>
                <p class="contact-label">Phone</p>
                <a href="tel:8033314504" class="contact-link">803-331-4504</a>
              </div>
            </div>

            <div class="contact-item">
              <span class="contact-icon">💼</span>
              <div>
                <p class="contact-label">LinkedIn</p>
                <a href="https://www.linkedin.com/in/timothy-chappell-bb311b390" target="_blank" class="contact-link">timothy-chappell-bb311b390</a>
      

      <div class="contact-item">   <span class="contact-icon">📧</span></div>
</span>
        <div>
          <p class="contact-label">Email</p>
          <a href="mailto:tbchappell803@gmail.com" class="contact-link">tbchappell803@gmail.com</a>
        </div>
      </div>>
          </div>

          <div class="availability">
            <p><strong>🌐 Remote Work:</strong> Available for remote opportunities worldwide</p>
            <p><strong>⏰ Availability:</strong> Immediate start</p>
          </div>
        </section>
      </div>

      <div class="footer-glyph" on:click={resetRitual}>
        ⚡⊰ΨΩ≋⊱⚡ 
        <small>Reset Portal</small>
      </div>
    </div>
  {/if}
</main>

<style>
  :global(body) { margin: 0; background: #000; font-family: 'Courier New', monospace; color: #00ffff; }
  main { position: relative; min-height: 100vh; overflow-y: auto; z-index: 2; }
  
  .glyph-container {
    position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
    display: flex; font-size: 8rem; cursor: pointer; user-select: none; z-index: 100;
  }
  
  .glyph-half {
    color: #00ffff;
    text-shadow: 0 0 30px #00ffff, 0 0 60px #00ffff;
  }
  
  .left { margin-right: -1rem; }
  .right { margin-left: -1rem; }
  
  .content-wrapper { max-width: 900px; margin: 0 auto; padding: 4rem 2rem; }
  .title { font-size: 2.2rem; text-align: center; margin-bottom: 2rem; letter-spacing: 0.3rem; text-shadow: 0 0 20px #00ffff; }
  
  .tabs {
    display: flex; flex-wrap: wrap; gap: 1rem; justify-content: center; margin: 2rem 0;
  }
  
  .tab-btn {
    background: none; border: 1px solid #00ffff; color: #00ffff; padding: 0.8rem 1.2rem;
    font-family: 'Courier New'; cursor: pointer; text-transform: uppercase; font-size: 0.9rem;
    transition: all 0.3s ease;
  }
  
  .tab-btn:hover { background: rgba(0,255,255,0.1); }
  .tab-btn.active { background: #00ffff; color: #000; }
  
  .tab-content { display: none; }
  
  .resume-section { 
    margin: 2rem 0; padding: 2rem; border: 2px solid #00ffff; border-radius: 10px; 
    background: rgba(0,255,255,0.05); 
  }
  
  .resume-section h2 { font-size: 1.8rem; text-align: center; margin-bottom: 1.5rem; color: #00ffff; text-shadow: 0 0 15px #00ffff; }
  
  .intro { font-size: 1.1rem; line-height: 1.8; margin-bottom: 1.5rem; }
  
  .highlight-box { 
    background: rgba(0,255,255,0.1); border-left: 4px solid #00ffff; 
    padding: 1rem; margin: 1.5rem 0; 
  }
  
  .tagline { text-align: center; font-style: italic; color: #0ff; margin-top: 1.5rem; font-size: 1.1rem; }
  
  .experience-item { 
    margin: 2rem 0; padding: 1.5rem; background: rgba(0,255,255,0.03); 
    border-left: 3px solid #00ffff; 
  }
  
  .experience-item h3 { color: #00ffff; margin-bottom: 0.5rem; }
  .date { color: #0ff; font-size: 0.9rem; margin-bottom: 1rem; opacity: 0.8; }
  
  .experience-item ul { margin-left: 1.5rem; }
  .experience-item li { margin: 0.8rem 0; line-height: 1.6; }
  
  .skills-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 2rem; margin-top: 2rem; }
  
  .skill-category h3 { color: #00ffff; margin-bottom: 1rem; border-bottom: 2px solid #00ffff; padding-bottom: 0.5rem; }
  
  .skill-list { list-style: none; padding: 0; }
  .skill-list li { 
    padding: 0.5rem 0; padding-left: 1.5rem; 
    position: relative;
  }
  .skill-list li:before { content: "▸"; position: absolute; left: 0; color: #00ffff; }
  
  .project-item { 
    margin: 1.5rem 0; padding: 1.5rem; background: rgba(0,255,255,0.05); 
    border: 1px solid #00ffff; border-radius: 5px; 
  }
  
  .project-item h3 { color: #00ffff; margin-bottom: 0.8rem; }
  
  .contact-section { text-align: center; }
  
  .contact-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 2rem; margin: 2rem 0; text-align: left; }
  
  .contact-item { 
    display: flex; align-items: center; gap: 1rem; 
    padding: 1rem; background: rgba(0,255,255,0.05); 
    border: 1px solid #00ffff; border-radius: 5px; 
  }
  
  .contact-icon { font-size: 2rem; }
  .contact-label { font-size: 0.9rem; color: #0ff; opacity: 0.8; margin-bottom: 0.3rem; }
  .contact-link { color: #00ffff; text-decoration: none; font-weight: bold; }
  .contact-link:hover { text-decoration: underline; }
  
  .availability { 
    margin-top: 2rem; padding: 1.5rem; background: rgba(0,255,255,0.1); 
    border: 2px solid #00ffff; border-radius: 5px; 
  }
  .availability p { margin: 0.8rem 0; font-size: 1.1rem; }
  
  .footer-glyph { 
    text-align: center; font-size: 3rem; margin-top: 4rem; cursor: pointer; 
    animation: pulse 2s infinite; color: #00ffff;
  }
  .footer-glyph small { font-size: 0.8rem; display: block; margin-top: 0.5rem; color: #0ff; }
  
  @keyframes pulse { 0%, 100% { opacity: 0.7; } 50% { opacity: 1; } }
</style>
