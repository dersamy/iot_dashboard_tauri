<script>
  import DashboardGrid from '$lib/components/DashboardGrid.svelte';
  import AddCardButton from '$lib/components/AddCardButton.svelte';
  import AddCardModal from '$lib/components/AddCardModal.svelte';
  import { onMount } from 'svelte';

  let showModal = false;
  let cards = [];

  // Updated card types to match CardContent.svelte sensor types
  const cardTypes = [
    { id: 'temperature', name: 'Temperature Sensor', icon: '🌡️' },
    { id: 'humidity', name: 'Humidity Sensor', icon: '💧' },
    { id: 'pir', name: 'Motion Detector (PIR)', icon: '🚶' },
    { id: 'gas', name: 'Gas Sensor', icon: '💨' },
    { id: 'ldr', name: 'Light Sensor (LDR)', icon: '💡' },
    { id: 'potentiometer', name: 'Potentiometer', icon: '🎚️' }
  ];

  // Load cards from localStorage on mount
  onMount(() => {
    const savedCards = localStorage.getItem('dashboard-cards');
    if (savedCards) {
      cards = JSON.parse(savedCards);
    } else {
      // Default cards for demo - updated to use correct sensor types
      cards = [
        { id: 'temp-1', type: 'temperature', title: 'Room Temperature', x: 0, y: 0, w: 1, h: 1 },
        { id: 'hum-1', type: 'humidity', title: 'Room Humidity', x: 1, y: 0, w: 1, h: 1 },
        { id: 'pir-1', type: 'pir', title: 'Motion Detector', x: 2, y: 0, w: 1, h: 1 },
        { id: 'gas-1', type: 'gas', title: 'Air Quality', x: 0, y: 1, w: 1, h: 1 },
        { id: 'ldr-1', type: 'ldr', title: 'Ambient Light', x: 1, y: 1, w: 1, h: 1 },
        { id: 'pot-1', type: 'potentiometer', title: 'Control Knob', x: 2, y: 1, w: 1, h: 1 }
      ];
    }
  });

  // Save cards to localStorage whenever cards change
  $: if (cards.length > 0) {
    localStorage.setItem('dashboard-cards', JSON.stringify(cards));
  }

  function handleAddCard() {
    showModal = true;
  }

  function handleModalClose() {
    showModal = false;
  }

  function handleCardAdd(event) {
    const { cardType, title } = event.detail;
    
    // Find a suitable position for the new card
    const existingPositions = new Set(cards.map(card => `${card.x},${card.y}`));
    let newX = 0, newY = 0;
    
    // Find the first available position
    for (let y = 0; y < 10; y++) {
      for (let x = 0; x < 6; x++) {
        if (!existingPositions.has(`${x},${y}`)) {
          newX = x;
          newY = y;
          break;
        }
      }
      if (newX !== 0 || newY !== 0) break;
    }
    
    const newCard = {
      id: `${cardType.id}-${Date.now()}`,
      type: cardType.id,
      title: title || cardType.name,
      x: newX,
      y: newY,
      w: 1,
      h: 1
    };
    
    cards = [...cards, newCard];
    showModal = false;
  }

  function handleCardDelete(event) {
    const cardId = event.detail.id;
    cards = cards.filter(card => card.id !== cardId);
    
    // Update localStorage after deletion
    if (cards.length === 0) {
      localStorage.removeItem('dashboard-cards');
    } else {
      localStorage.setItem('dashboard-cards', JSON.stringify(cards));
    }
  }

  function handleGridUpdate(event) {
    cards = event.detail.cards;
  }
</script>

<svelte:head>
  <title>IoT Educational Dashboard</title>
  <meta name="description" content="Monitor and interact with educational IoT sensors in real-time" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

<main class="min-h-screen bg-gradient-to-br from-indigo-900 via-purple-900 to-pink-800 p-2 sm:p-4">
  <!-- Background overlay for glassmorphism effect -->
  <div class="fixed inset-0 bg-black/20 backdrop-blur-sm pointer-events-none"></div>
  
  <!-- Header -->
  <header class="relative z-10 mb-4 sm:mb-6">
    <div class="backdrop-blur-md bg-white/10 rounded-xl sm:rounded-2xl border border-white/20 p-3 sm:p-4 md:p-6 shadow-xl">
      <div class="flex items-start sm:items-center justify-between flex-col sm:flex-row gap-3 sm:gap-4">
        <div class="flex-1 min-w-0">
          <h1 class="text-xl sm:text-2xl md:text-3xl font-bold text-white mb-1 sm:mb-2">
            IoT Educational Dashboard
          </h1>
          <p class="text-white/70 text-sm sm:text-base">
            Monitor and control your educational IoT sensors in real-time
          </p>
          <div class="flex items-center gap-2 sm:gap-4 mt-2 text-xs sm:text-sm text-white/60 flex-wrap">
            <span>📊 {cards.length} active sensors</span>
            <span>🔄 Live updates</span>
            <span class="hidden sm:inline">📱 Drag & drop layout</span>
          </div>
        </div>
        <div class="flex-shrink-0 w-full sm:w-auto">
          <AddCardButton on:click={handleAddCard} />
        </div>
      </div>
    </div>
  </header>

  <!-- Dashboard Grid -->
  <div class="relative z-10">
    <DashboardGrid 
      {cards} 
      on:delete={handleCardDelete}
      on:update={handleGridUpdate}
    />
  </div>

  <!-- Add Card Modal -->
  {#if showModal}
    <AddCardModal 
      {cardTypes}
      on:close={handleModalClose}
      on:add={handleCardAdd}
    />
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    overflow-x: auto;
  }
  
  :global(*) {
    box-sizing: border-box;
  }

  /* Ensure modal doesn't cause horizontal scroll */
  :global(html, body) {
    overflow-x: hidden;
  }
</style>