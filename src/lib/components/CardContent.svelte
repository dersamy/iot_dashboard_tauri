<script>
  import { onMount, onDestroy } from 'svelte';
  import { fetch as tauriFetch } from '@tauri-apps/plugin-http';

  export let card;
  export let cardWidth = 240;
  export let cardHeight = 240;

  let currentValue = 0;
  let status = 'connecting';
  let lastUpdate = new Date();
  let updateInterval;
  let errorCount = 0;
  let isOnline = false;
  let timestamp = 0;
  let isSending = false;

  // Control states
  let sliderValue = 50;
  let toggleValue = false;
  let colorValue = '#ff8000';
  let textValue = '';

  const API_BASE = 'http://192.168.4.1:80';
  const MAX_RETRIES = 3;
  const UPDATE_INTERVAL = 5000;

  // API endpoint mapping for sensors
  const SENSOR_ENDPOINTS = {
    temperature: '/temperature',
    humidity: '/humidity',
    gas: '/gas',
    potentiometer: '/pot',
    pir: '/pir',
    ldr: '/ldr'
  };

  // Control card types
  const CONTROL_TYPES = ['slider', 'toggle', 'color', 'text'];

  // Responsive calculations
  let responsiveCache = {};
  
  function getResponsiveProps(width, height) {
    const key = `${width}x${height}`;
    if (responsiveCache[key]) return responsiveCache[key];
    
    const isSmallCard = width < 200 || height < 200;
    const isMediumCard = width >= 200 && width < 300 && height >= 200 && height < 300;
    const isLargeCard = width >= 300 || height >= 300;
    
    const props = {
      isSmallCard,
      isMediumCard,
      isLargeCard,
      mainValueSize: isSmallCard ? 'text-lg' : isMediumCard ? 'text-2xl' : 'text-4xl',
      statusTextSize: isSmallCard ? 'text-xs' : isMediumCard ? 'text-sm' : 'text-base',
      labelTextSize: isSmallCard ? 'text-xs' : isMediumCard ? 'text-sm' : 'text-base',
      contentPadding: isSmallCard ? 'p-2' : isMediumCard ? 'p-3' : 'p-4',
      sectionSpacing: isSmallCard ? 'mb-2' : isMediumCard ? 'mb-4' : 'mb-6',
      statusIndicatorSize: isSmallCard ? 'w-2 h-2' : isMediumCard ? 'w-3 h-3' : 'w-4 h-4'
    };
    
    responsiveCache[key] = props;
    return props;
  }

  $: responsive = getResponsiveProps(cardWidth, cardHeight);
  $: isControlCard = CONTROL_TYPES.includes(card.type);

  // Send control data to endpoint
  async function sendControlData(value) {
    if (!card.endpoint) return;
    
    isSending = true;
    try {
      const endpoint = card.endpoint.startsWith('/') ? card.endpoint : `/${card.endpoint}`;
      
      const response = await tauriFetch(`${API_BASE}${endpoint}`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        },
        body: JSON.stringify({ value }),
        timeout: 3000
      });

      if (response.ok) {
        status = 'active';
        isOnline = true;
        errorCount = 0;
        lastUpdate = new Date();
        console.log(`Control sent to ${endpoint}:`, value);
      } else {
        throw new Error(`HTTP ${response.status}`);
      }
    } catch (error) {
      console.error(`Control send error:`, error);
      errorCount++;
      status = 'error';
      isOnline = false;
    } finally {
      isSending = false;
    }
  }

  // Fetch sensor data (for sensor cards only)
  async function fetchSensorData() {
    if (isControlCard) return;
    
    const endpoint = SENSOR_ENDPOINTS[card.type];
    if (!endpoint) return;

    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 3000);

      const response = await tauriFetch(`${API_BASE}${endpoint}`, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'Cache-Control': 'no-cache'
        },
        signal: controller.signal
      });

      clearTimeout(timeoutId);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      const data = await response.json();
      const newValue = parseValue(data, card.type);
      
      currentValue = newValue;
      timestamp = data.timestamp || Date.now();
      lastUpdate = new Date();
      status = 'active';
      errorCount = 0;
      isOnline = true;

    } catch (error) {
      errorCount++;
      isOnline = false;
      
      if (error.name === 'AbortError') {
        status = 'timeout';
      } else if (error.message.includes('Failed to fetch')) {
        status = 'offline';
      } else {
        status = 'error';
      }

      if (errorCount >= MAX_RETRIES) {
        status = 'error';
      }
    }
  }

  function parseValue(data, type) {
    switch (type) {
      case 'temperature': return parseFloat(data.temperature);
      case 'humidity': return parseFloat(data.humidity);
      case 'gas': return parseInt(data.gas);
      case 'potentiometer': return parseInt(data.pot);
      case 'pir': return Boolean(data.pir || data.motion_detected);
      case 'ldr': return parseInt(data.ldr || data.light_level);
      default: return data.value || 0;
    }
  }

  const formatCache = new Map();
  function formatValue(value, type) {
    const key = `${value}-${type}`;
    if (formatCache.has(key)) return formatCache.get(key);
    
    let formatted;
    switch (type) {
      case 'temperature': formatted = `${value.toFixed(1)}°C`; break;
      case 'humidity': formatted = `${value.toFixed(0)}%`; break;
      case 'pir': formatted = value ? 'Motion Detected' : 'No Motion'; break;
      case 'gas': formatted = `${Math.round(value)} ppm`; break;
      case 'ldr': formatted = `${Math.round(value)} lux`; break;
      case 'potentiometer': formatted = `${Math.round(value)}/4095`; break;
      default: formatted = value.toString();
    }
    
    formatCache.set(key, formatted);
    return formatted;
  }

  function getStatusColor(status) {
    switch (status) {
      case 'active': return 'text-green-400';
      case 'connecting': return 'text-blue-400';
      case 'timeout': return 'text-yellow-400';
      case 'offline': return 'text-orange-400';
      case 'error': return 'text-red-400';
      default: return 'text-gray-400';
    }
  }

  function getStatusIndicator(status) {
    switch (status) {
      case 'active': return 'bg-green-400 animate-pulse';
      case 'connecting': return 'bg-blue-400 animate-ping';
      case 'timeout': return 'bg-yellow-400 animate-pulse';
      case 'offline': return 'bg-orange-400';
      case 'error': return 'bg-red-400 animate-pulse';
      default: return 'bg-gray-400';
    }
  }

  // Control event handlers with proper event handling
  function handleSliderChange(event) {
    event.stopPropagation();
    sliderValue = parseInt(event.target.value);
    sendControlData(sliderValue);
  }

  function handleSliderMouseDown(event) {
    event.stopPropagation();
  }

  function handleToggleChange(event) {
    event.stopPropagation();
    toggleValue = !toggleValue;
    sendControlData(toggleValue);
  }

  function handleColorChange(event) {
    event.stopPropagation();
    colorValue = event.target.value;
    // Convert hex to RGB
    const r = parseInt(colorValue.slice(1, 3), 16);
    const g = parseInt(colorValue.slice(3, 5), 16);
    const b = parseInt(colorValue.slice(5, 7), 16);
    sendControlData({ r, g, b });
  }

  function handleTextSubmit(event) {
    event.stopPropagation();
    if (textValue.trim()) {
      sendControlData(textValue.trim());
    }
  }

  function handleTextKeydown(event) {
    event.stopPropagation();
    if (event.key === 'Enter') {
      handleTextSubmit(event);
    }
  }

  function handleTextInput(event) {
    event.stopPropagation();
  }

  // Initialize control cards
  function initializeControlCard() {
    status = 'active';
    isOnline = true;
    lastUpdate = new Date();
    
    // Set default values based on card type
    switch (card.type) {
      case 'slider':
        sliderValue = 50;
        break;
      case 'toggle':
        toggleValue = false;
        break;
      case 'color':
        colorValue = '#ff8000';
        break;
      case 'text':
        textValue = '';
        break;
    }
  }

  // Lifecycle management
  onMount(() => {
    if (isControlCard) {
      initializeControlCard();
    } else {
      fetchSensorData();
      updateInterval = setInterval(fetchSensorData, UPDATE_INTERVAL);
    }
  });

  onDestroy(() => {
    if (updateInterval) {
      clearInterval(updateInterval);
    }
    formatCache.clear();
    responsiveCache = {};
  });
</script>

<div class="h-full flex flex-col {responsive.contentPadding}">
  <!-- Status indicator -->
  {#if !responsive.isSmallCard}
    <div class="flex items-center justify-between mb-2 text-xs text-white/60">
      <span>
        {#if isControlCard}
          Endpoint: {card.endpoint || '/default'}
        {:else}
          ESP32: {API_BASE.replace('http://', '')}
        {/if}
      </span>
      <div class="flex items-center gap-1">
        <div class="{responsive.statusIndicatorSize} rounded-full {getStatusIndicator(status)}"></div>
        <span class="{getStatusColor(status)}">
          {#if isSending}
            Sending...
          {:else}
            {status === 'active' ? 'Ready' : status === 'connecting' ? 'Connecting' : 'Offline'}
          {/if}
        </span>
      </div>
    </div>
  {/if}

  <!-- Main content -->
  <div class="flex-1 flex flex-col justify-center">
    
    {#if card.type === 'slider'}
      <!-- Slider Control -->
      <div class="text-center mb-4 card-control">
        <div class="{responsive.mainValueSize} font-bold text-blue-400 mb-2">
          {sliderValue}%
        </div>
        <div class="px-2">
          <input
            type="range"
            min="0"
            max="100"
            bind:value={sliderValue}
            on:input={handleSliderChange}
            on:mousedown={handleSliderMouseDown}
            class="w-full h-3 bg-white/20 rounded-lg appearance-none cursor-pointer slider"
            disabled={isSending}
          />
        </div>
        <div class="flex justify-between text-xs text-white/60 mt-2">
          <span>0%</span>
          <span>100%</span>
        </div>
      </div>

    {:else if card.type === 'toggle'}
      <!-- Toggle Control -->
      <div class="text-center card-control">
        <div class="{responsive.mainValueSize} font-bold mb-4 {toggleValue ? 'text-green-400' : 'text-gray-400'}">
          {toggleValue ? 'ON' : 'OFF'}
        </div>
        <button
          on:click={handleToggleChange}
          on:mousedown={handleSliderMouseDown}
          disabled={isSending}
          class="relative inline-flex h-12 w-20 items-center rounded-full transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50
                 {toggleValue ? 'bg-green-500' : 'bg-gray-600'}"
        >
          <span class="inline-block h-8 w-8 transform rounded-full bg-white transition-transform duration-200 {toggleValue ? 'translate-x-10' : 'translate-x-1'}"></span>
        </button>
      </div>

    {:else if card.type === 'color'}
      <!-- Color Control -->
      <div class="text-center card-control">
        <div class="{responsive.mainValueSize} font-bold mb-4 text-white">
          RGB Color
        </div>
        <div class="flex flex-col items-center space-y-4">
          <div 
            class="w-16 h-16 rounded-full border-2 border-white/30 shadow-lg"
            style="background-color: {colorValue}"
          ></div>
          <input
            type="color"
            bind:value={colorValue}
            on:input={handleColorChange}
            on:mousedown={handleSliderMouseDown}
            disabled={isSending}
            class="w-full h-12 rounded-lg border-2 border-white/30 bg-transparent cursor-pointer disabled:opacity-50"
          />
          <div class="text-sm text-white/70">
            {colorValue.toUpperCase()}
          </div>
        </div>
      </div>

    {:else if card.type === 'text'}
      <!-- Text Input Control -->
      <div class="text-center card-control">
        <div class="{responsive.labelTextSize} text-white/70 mb-2">
          Text Input
        </div>
        <div class="space-y-3">
          <input
            type="text"
            bind:value={textValue}
            on:keydown={handleTextKeydown}
            on:input={handleTextInput}
            on:mousedown={handleSliderMouseDown}
            disabled={isSending}
            placeholder="Enter text..."
            class="w-full px-3 py-2 rounded-lg bg-white/20 border border-white/30 text-white placeholder-white/50 focus:outline-none focus:border-blue-400 focus:bg-white/25 transition-all duration-200 disabled:opacity-50"
          />
          <button
            on:click={handleTextSubmit}
            on:mousedown={handleSliderMouseDown}
            disabled={isSending || !textValue.trim()}
            class="w-full px-4 py-2 bg-blue-500/80 hover:bg-blue-500 disabled:bg-gray-600 disabled:opacity-50 text-white rounded-lg transition-all duration-200 font-medium disabled:cursor-not-allowed"
          >
            {isSending ? 'Sending...' : 'Send'}
          </button>
        </div>
      </div>

    {:else}
      <!-- Sensor Display -->
      <div class="text-center">
        <div class="{responsive.mainValueSize} font-bold text-white mb-1 sm:mb-2 leading-tight">
          {formatValue(currentValue, card.type)}
        </div>
        
        <div class="flex items-center justify-center gap-2">
          <div class="{responsive.statusIndicatorSize} rounded-full {getStatusIndicator(status)}"></div>
          <span class="{responsive.statusTextSize} {getStatusColor(status)} capitalize">
            {status === 'active' ? 'Online' : status}
          </span>
        </div>
      </div>
    {/if}
  </div>

  <!-- Footer -->
  <div class="mt-auto pt-2 sm:pt-4 border-t border-white/10">
    <div class="flex justify-between items-center {responsive.labelTextSize} text-white/60">
      <span>Last {isControlCard ? 'sent' : 'update'}</span>
      <span>{lastUpdate.toLocaleTimeString()}</span>
    </div>
  </div>
</div>

<style>
  .slider::-webkit-slider-thumb {
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #60a5fa;
    cursor: pointer;
    border: 2px solid white;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  }

  .slider::-moz-range-thumb {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #60a5fa;
    cursor: pointer;
    border: 2px solid white;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  }

  .slider::-webkit-slider-track {
    background: rgba(255, 255, 255, 0.2);
    border-radius: 5px;
  }

  .slider::-moz-range-track {
    background: rgba(255, 255, 255, 0.2);
    border-radius: 5px;
  }
</style>