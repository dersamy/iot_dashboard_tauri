<script>
  import { onMount } from 'svelte';
  
  let isLoading = false;
  let mouseTrails = [];
  let trailId = 0;

  function enterDashboard() {
    isLoading = true;
    
    // Simulate loading and redirect
    setTimeout(() => {
      // Replace with your actual dashboard URL or use SvelteKit navigation
      window.location.href = '/testbis';
    }, 1000);
  }

  function handleMouseMove(event) {
    const trail = {
      id: trailId++,
      x: event.clientX,
      y: event.clientY
    };
    
    mouseTrails = [...mouseTrails, trail];
    
    // Remove trail after animation
    setTimeout(() => {
      mouseTrails = mouseTrails.filter(t => t.id !== trail.id);
    }, 1000);
  }

  onMount(() => {
    // Clean up any remaining trails on unmount
    return () => {
      mouseTrails = [];
    };
  });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<div class="welcome-page">
  <div class="floating-elements">
    {#each Array(5) as _, i}
      <div class="floating-element" style="--delay: {i * 1.2}s"></div>
    {/each}
  </div>

  {#each mouseTrails as trail (trail.id)}
    <div 
      class="mouse-trail" 
      style="left: {trail.x}px; top: {trail.y}px;"
    ></div>
  {/each}

  <div class="welcome-container">
    <h1 class="welcome-title">Welcome to MobotIOT Dashboard</h1>
    <p class="welcome-subtitle">Monitor sensors and control devices in real-time</p>
    
    <button 
      class="enter-button" 
      class:loading={isLoading}
      on:click={enterDashboard}
      disabled={isLoading}
    >
      {isLoading ? 'Loading...' : 'Enter Dashboard'}
    </button>
    
    <div class="features-hint">
      <div class="feature-item">
        <div class="feature-icon">📊</div>
        <span>Live Updates</span>
      </div>
      <div class="feature-item">
        <div class="feature-icon">🎛️</div>
        <span>Interactive Controls</span>
      </div>
      <div class="feature-item">
        <div class="feature-icon">🔗</div>
        <span>Real-time Monitoring</span>
      </div>
    </div>
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    overflow: hidden;
  }

  .welcome-page {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    overflow: hidden;
  }

  .welcome-container {
    text-align: center;
    max-width: 800px;
    padding: 2rem;
    position: relative;
    z-index: 2;
  }

  .welcome-title {
    font-size: 4rem;
    font-weight: 300;
    color: white;
    margin-bottom: 1rem;
    text-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
    animation: fadeInUp 1s ease-out;
  }

  .welcome-subtitle {
    font-size: 1.5rem;
    color: rgba(255, 255, 255, 0.9);
    margin-bottom: 3rem;
    font-weight: 300;
    animation: fadeInUp 1s ease-out 0.3s both;
  }

  .enter-button {
    background: linear-gradient(45deg, #ff6b6b, #ee5a24);
    border: none;
    padding: 1.5rem 3rem;
    font-size: 1.3rem;
    color: white;
    border-radius: 50px;
    cursor: pointer;
    box-shadow: 0 8px 30px rgba(255, 107, 107, 0.4);
    transition: all 0.3s ease;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 1px;
    position: relative;
    overflow: hidden;
    animation: fadeInUp 1s ease-out 0.6s both;
  }

  .enter-button:hover:not(:disabled) {
    transform: translateY(-3px);
    box-shadow: 0 12px 40px rgba(255, 107, 107, 0.6);
  }

  .enter-button:active:not(:disabled) {
    transform: translateY(-1px);
  }

  .enter-button:disabled {
    cursor: not-allowed;
    opacity: 0.7;
  }

  .enter-button.loading {
    transform: scale(0.95);
  }

  .enter-button::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
    transition: left 0.6s ease;
  }

  .enter-button:hover:not(:disabled)::before {
    left: 100%;
  }

  .floating-elements {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    pointer-events: none;
    overflow: hidden;
  }

  .floating-element {
    position: absolute;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 50%;
    animation: float 6s ease-in-out infinite;
    animation-delay: var(--delay);
  }

  .floating-element:nth-child(1) {
    width: 80px;
    height: 80px;
    top: 20%;
    left: 10%;
  }

  .floating-element:nth-child(2) {
    width: 120px;
    height: 120px;
    top: 60%;
    right: 15%;
  }

  .floating-element:nth-child(3) {
    width: 60px;
    height: 60px;
    bottom: 30%;
    left: 20%;
  }

  .floating-element:nth-child(4) {
    width: 100px;
    height: 100px;
    top: 10%;
    right: 30%;
  }

  .floating-element:nth-child(5) {
    width: 140px;
    height: 140px;
    bottom: 10%;
    right: 40%;
  }

  .mouse-trail {
    position: fixed;
    width: 4px;
    height: 4px;
    background: rgba(255, 255, 255, 0.6);
    border-radius: 50%;
    pointer-events: none;
    z-index: 1000;
    animation: fadeOut 1s ease-out forwards;
    transform: translate(-50%, -50%);
  }

  .features-hint {
    margin-top: 2rem;
    display: flex;
    justify-content: center;
    gap: 2rem;
    animation: fadeInUp 1s ease-out 0.9s both;
  }

  .feature-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: rgba(255, 255, 255, 0.8);
    font-size: 0.9rem;
  }

  .feature-icon {
    width: 20px;
    height: 20px;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  @keyframes fadeInUp {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @keyframes float {
    0%, 100% {
      transform: translateY(0px) rotate(0deg);
    }
    50% {
      transform: translateY(-20px) rotate(180deg);
    }
  }

  @keyframes fadeOut {
    0% { 
      opacity: 1; 
      transform: translate(-50%, -50%) scale(1); 
    }
    100% { 
      opacity: 0; 
      transform: translate(-50%, -50%) scale(0); 
    }
  }

  @media (max-width: 768px) {
    .welcome-title {
      font-size: 2.5rem;
    }
    
    .welcome-subtitle {
      font-size: 1.2rem;
    }
    
    .enter-button {
      padding: 1.2rem 2.5rem;
      font-size: 1.1rem;
    }
    
    .features-hint {
      flex-direction: column;
      gap: 1rem;
    }
  }
</style>