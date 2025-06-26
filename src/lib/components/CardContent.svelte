<script>
  import { onMount } from 'svelte';

  export let card;
  export let cardWidth = 240;
  export let cardHeight = 240;

  let currentValue = 0;
  let status = 'active';
  let lastUpdate = new Date();
  let history = []; // Store last 10 values for trend

  // Responsive sizing based on card dimensions
  $: isSmallCard = cardWidth < 200 || cardHeight < 200;
  $: isMediumCard = cardWidth >= 200 && cardWidth < 300 && cardHeight >= 200 && cardHeight < 300;
  $: isLargeCard = cardWidth >= 300 || cardHeight >= 300;
  $: isWideCard = cardWidth > cardHeight * 1.5;
  $: isTallCard = cardHeight > cardWidth * 1.5;

  // Dynamic text sizes
  $: mainValueSize = isSmallCard ? 'text-lg' : 
                     isMediumCard ? 'text-2xl' : 
                     isLargeCard ? 'text-4xl' : 'text-3xl';
  
  $: statusTextSize = isSmallCard ? 'text-xs' : 
                      isMediumCard ? 'text-sm' : 'text-base';
  
  $: labelTextSize = isSmallCard ? 'text-xs' : 
                     isMediumCard ? 'text-sm' : 'text-base';
  
  $: infoTextSize = isSmallCard ? 'text-sm' : 
                    isMediumCard ? 'text-base' : 'text-lg';

  // Dynamic spacing
  $: contentPadding = isSmallCard ? 'p-2' : 
                      isMediumCard ? 'p-3' : 'p-4';
  
  $: sectionSpacing = isSmallCard ? 'mb-2' : 
                      isMediumCard ? 'mb-4' : 'mb-6';

  // Dynamic indicator sizes
  $: statusIndicatorSize = isSmallCard ? 'w-2 h-2' : 
                           isMediumCard ? 'w-3 h-3' : 'w-4 h-4';
  
  $: pirIndicatorSize = isSmallCard ? 'w-6 h-6' : 
                        isMediumCard ? 'w-10 h-10' : 'w-14 h-14';
  
  $: pirDotSize = isSmallCard ? 'w-3 h-3' : 
                  isMediumCard ? 'w-5 h-5' : 'w-7 h-7';

  // Progress bar height
  $: progressBarHeight = isSmallCard ? 'h-1' : 
                         isMediumCard ? 'h-2' : 'h-3';

  // Simulate real-time data updates
  onMount(() => {
    const interval = setInterval(() => {
      updateSensorData();
    }, 2000 + Math.random() * 3000);

    for (let i = 0; i < 10; i++) {
      history.push(generateRandomValue(card.type));
    }

    return () => clearInterval(interval);
  });

  function generateRandomValue(type) {
    switch (type) {
      case 'temperature':
        return 15 + Math.random() * 20;
      case 'humidity':
        return 30 + Math.random() * 40;
      case 'pir':
        return Math.random() > 0.8 ? 1 : 0;
      case 'gas':
        return Math.random() * 1024;
      case 'ldr':
        return Math.random() * 1024;
      case 'potentiometer':
        return Math.random() * 1024;
      default:
        return 0;
    }
  }

  function updateSensorData() {
    lastUpdate = new Date();
    status = Math.random() > 0.1 ? 'active' : 'warning';
    currentValue = generateRandomValue(card.type);
    history = [...history.slice(-9), currentValue];
  }

  function formatValue(value, type) {
    switch (type) {
      case 'temperature':
        return `${value.toFixed(1)}°C`;
      case 'humidity':
        return `${value.toFixed(0)}%`;
      case 'pir':
        return value ? 'Motion Detected' : 'No Motion';
      case 'gas':
        return `${Math.round(value)}/1024`;
      case 'ldr':
        return `${Math.round(value)}/1024`;
      case 'potentiometer':
        return `${Math.round(value)}/1024`;
      default:
        return value.toString();
    }
  }

  function getStatusColor(status) {
    switch (status) {
      case 'active':
        return 'text-green-400';
      case 'warning':
        return 'text-yellow-400';
      case 'error':
        return 'text-red-400';
      default:
        return 'text-gray-400';
    }
  }

  function getMaxValue(type) {
    switch (type) {
      case 'temperature':
        return 50;
      case 'humidity':
        return 100;
      case 'pir':
        return 1;
      case 'gas':
        return 1024;
      case 'ldr':
        return 1024;
      case 'potentiometer':
        return 1024;
      default:
        return 100;
    }
  }

  function getValueColor(value, type) {
    switch (type) {
      case 'temperature':
        if (value < 18) return 'text-blue-400';
        if (value > 28) return 'text-red-400';
        return 'text-green-400';
      case 'humidity':
        if (value < 40 || value > 60) return 'text-yellow-400';
        return 'text-green-400';
      case 'pir':
        return value ? 'text-red-400' : 'text-green-400';
      case 'gas':
        if (value > 700) return 'text-red-400';
        if (value > 400) return 'text-yellow-400';
        return 'text-green-400';
      case 'ldr':
        if (value < 200) return 'text-blue-400';
        if (value > 800) return 'text-yellow-400';
        return 'text-green-400';
      case 'potentiometer':
        return 'text-blue-400';
      default:
        return 'text-white';
    }
  }

  function getProgressBarColor(value, type) {
    switch (type) {
      case 'temperature':
        if (value < 18) return 'from-blue-400 to-blue-500';
        if (value > 28) return 'from-red-400 to-red-500';
        return 'from-green-400 to-green-500';
      case 'humidity':
        if (value < 40 || value > 60) return 'from-yellow-400 to-yellow-500';
        return 'from-green-400 to-green-500';
      case 'pir':
        return value ? 'from-red-400 to-red-500' : 'from-green-400 to-green-500';
      case 'gas':
        if (value > 700) return 'from-red-400 to-red-500';
        if (value > 400) return 'from-yellow-400 to-yellow-500';
        return 'from-green-400 to-green-500';
      case 'ldr':
        return 'from-yellow-400 to-orange-500';
      case 'potentiometer':
        return 'from-blue-400 to-purple-500';
      default:
        return 'from-blue-400 to-green-400';
    }
  }

  // Initialize with random data
  updateSensorData();
</script>

<div class="h-full flex flex-col {contentPadding}">
  <!-- Main value display - Responsive sizing -->
  <div class="flex-1 flex items-center justify-center">
    <div class="text-center">
      <div class="{mainValueSize} font-bold {getValueColor(currentValue, card.type)} mb-1 sm:mb-2 leading-tight">
        {#if isWideCard && !isSmallCard}
          <!-- Wide card layout - horizontal -->
          <div class="flex items-center justify-center gap-4">
            <span>{formatValue(currentValue, card.type)}</span>
            <div class="flex items-center gap-2">
              <div class="{statusIndicatorSize} rounded-full {status === 'active' ? 'bg-green-400' : status === 'warning' ? 'bg-yellow-400' : 'bg-red-400'} animate-pulse"></div>
              <span class="{statusTextSize} {getStatusColor(status)} capitalize">{status}</span>
            </div>
          </div>
        {:else}
          <!-- Standard layout - vertical -->
          {formatValue(currentValue, card.type)}
        {/if}
      </div>
      
      {#if !isWideCard || isSmallCard}
        <div class="flex items-center justify-center gap-2">
          <div class="{statusIndicatorSize} rounded-full {status === 'active' ? 'bg-green-400' : status === 'warning' ? 'bg-yellow-400' : 'bg-red-400'} animate-pulse"></div>
          <span class="{statusTextSize} {getStatusColor(status)} capitalize">{status}</span>
        </div>
      {/if}
    </div>
  </div>

  <!-- Sensor-specific additional info - Responsive -->
  {#if !isSmallCard}
    {#if card.type === 'pir'}
      <div class="{sectionSpacing} text-center">
        <div class="{labelTextSize} text-white/60 mb-2">Detection Status</div>
        <div class="flex justify-center">
          <div class="{pirIndicatorSize} rounded-full border-2 {currentValue ? 'border-red-400 bg-red-400/20' : 'border-green-400 bg-green-400/20'} flex items-center justify-center">
            <div class="{pirDotSize} rounded-full {currentValue ? 'bg-red-400' : 'bg-green-400'} {currentValue ? 'animate-pulse' : ''}"></div>
          </div>
        </div>
      </div>
    {:else if card.type === 'gas'}
      <div class="{sectionSpacing} text-center">
        <div class="{labelTextSize} text-white/60 mb-2">Air Quality</div>
        <div class="{infoTextSize} font-medium {currentValue > 700 ? 'text-red-400' : currentValue > 400 ? 'text-yellow-400' : 'text-green-400'}">
          {currentValue > 700 ? 'Poor' : currentValue > 400 ? 'Moderate' : 'Good'}
        </div>
      </div>
    {:else if card.type === 'ldr'}
      <div class="{sectionSpacing} text-center">
        <div class="{labelTextSize} text-white/60 mb-2">Light Condition</div>
        <div class="{infoTextSize} font-medium {getValueColor(currentValue, card.type)}">
          {currentValue < 200 ? 'Dark' : currentValue > 800 ? 'Bright' : 'Normal'}
        </div>
      </div>
    {:else if card.type === 'temperature'}
      <div class="{sectionSpacing} text-center">
        <div class="{labelTextSize} text-white/60 mb-2">Comfort Level</div>
        <div class="{infoTextSize} font-medium {getValueColor(currentValue, card.type)}">
          {currentValue < 18 ? 'Cold' : currentValue > 28 ? 'Hot' : 'Comfortable'}
        </div>
      </div>
    {:else if card.type === 'humidity'}
      <div class="{sectionSpacing} text-center">
        <div class="{labelTextSize} text-white/60 mb-2">Humidity Level</div>
        <div class="{infoTextSize} font-medium {getValueColor(currentValue, card.type)}">
          {currentValue < 40 ? 'Dry' : currentValue > 60 ? 'Humid' : 'Optimal'}
        </div>
      </div>
    {/if}
  {/if}

  <!-- Progress bar and info - Responsive -->
  <div class="mt-auto pt-2 sm:pt-4 border-t border-white/10">
    {#if !isSmallCard || isTallCard}
      <div class="flex justify-between items-center {labelTextSize} text-white/60 mb-2 sm:mb-3">
        <span>Last update</span>
        <span>{lastUpdate.toLocaleTimeString()}</span>
      </div>
    {/if}
    
    <!-- Progress bar for analog sensors -->
    {#if card.type !== 'pir'}
      <div class="{progressBarHeight} bg-white/10 rounded-full overflow-hidden">
        <div 
          class="h-full bg-gradient-to-r {getProgressBarColor(currentValue, card.type)} rounded-full transition-all duration-1000"
          style="width: {Math.min(100, (currentValue / getMaxValue(card.type)) * 100)}%"
        ></div>
      </div>
    {/if}

    <!-- Range indicator for analog sensors -->
    {#if !isSmallCard && card.type !== 'pir'}
      {#if ['gas', 'ldr', 'potentiometer'].includes(card.type)}
        <div class="flex justify-between {labelTextSize} text-white/40 mt-1 sm:mt-2">
          <span>0</span>
          <span>1024</span>
        </div>
      {:else if card.type === 'humidity'}
        <div class="flex justify-between {labelTextSize} text-white/40 mt-1 sm:mt-2">
          <span>0%</span>
          <span>100%</span>
        </div>
      {:else if card.type === 'temperature'}
        <div class="flex justify-between {labelTextSize} text-white/40 mt-1 sm:mt-2">
          <span>-10°C</span>
          <span>50°C</span>
        </div>
      {/if}
    {/if}
  </div>
</div>