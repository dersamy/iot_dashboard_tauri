<script>
  import { createEventDispatcher } from 'svelte';
  import DashboardCard from './DashboardCard.svelte';

  export let cards = [];

  const dispatch = createEventDispatcher();

  let gridContainer;
  let draggedCard = null;
  let dragOffset = { x: 0, y: 0 };
  let isDragging = false;

  const GRID_SIZE = 280;
  const GAP = 20;

  function handleCardDrag(event) {
    const { card, startX, startY } = event.detail;
    draggedCard = card;
    isDragging = true;
    
    const rect = gridContainer.getBoundingClientRect();
    dragOffset.x = startX - rect.left - (card.x * (GRID_SIZE + GAP));
    dragOffset.y = startY - rect.top - (card.y * (GRID_SIZE + GAP));

    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', handleMouseUp);
  }

  function handleMouseMove(event) {
    if (!draggedCard || !isDragging) return;

    const rect = gridContainer.getBoundingClientRect();
    const x = event.clientX - rect.left - dragOffset.x;
    const y = event.clientY - rect.top - dragOffset.y;

    // Snap to grid
    const gridX = Math.max(0, Math.round(x / (GRID_SIZE + GAP)));
    const gridY = Math.max(0, Math.round(y / (GRID_SIZE + GAP)));

    // Update card position
    const updatedCards = cards.map(card => 
      card.id === draggedCard.id ? { ...card, x: gridX, y: gridY } : card
    );

    dispatch('update', { cards: updatedCards });
  }

  function handleMouseUp() {
    isDragging = false;
    draggedCard = null;
    document.removeEventListener('mousemove', handleMouseMove);
    document.removeEventListener('mouseup', handleMouseUp);
  }

  function handleCardDelete(event) {
    dispatch('delete', event.detail);
  }

  // Calculate grid dimensions
  $: maxX = Math.max(0, ...cards.map(card => card.x + card.w));
  $: maxY = Math.max(0, ...cards.map(card => card.y + card.h));
  $: gridWidth = Math.max(4, maxX) * (GRID_SIZE + GAP) - GAP;
  $: gridHeight = Math.max(3, maxY) * (GRID_SIZE + GAP) - GAP;

  // Calculate actual card dimensions
  function getCardDimensions(card) {
    return {
      width: card.w * GRID_SIZE + (card.w - 1) * GAP,
      height: card.h * GRID_SIZE + (card.h - 1) * GAP
    };
  }
</script>

<div 
  bind:this={gridContainer}
  class="relative backdrop-blur-md bg-white/5 rounded-2xl border border-white/10 p-6 shadow-xl min-h-96"
  style="width: {gridWidth}px; height: {gridHeight}px;"
>
  <!-- Grid background -->
  <div class="absolute inset-6 opacity-20">
    <svg width="100%" height="100%" class="pointer-events-none">
      <defs>
        <pattern id="grid" width="{GRID_SIZE + GAP}" height="{GRID_SIZE + GAP}" patternUnits="userSpaceOnUse">
          <rect width="{GRID_SIZE}" height="{GRID_SIZE}" fill="none" stroke="rgba(255,255,255,0.1)" stroke-width="1"/>
        </pattern>
      </defs>
      <rect width="100%" height="100%" fill="url(#grid)" />
    </svg>
  </div>

  <!-- Cards with responsive dimensions -->
  {#each cards as card (card.id)}
    {@const cardDimensions = getCardDimensions(card)}
    <div
      class="absolute transition-all duration-200 ease-out"
      class:z-50={draggedCard?.id === card.id}
      class:z-10={draggedCard?.id !== card.id}
      style="
        left: {card.x * (GRID_SIZE + GAP)}px;
        top: {card.y * (GRID_SIZE + GAP)}px;
        width: {cardDimensions.width}px;
        height: {cardDimensions.height}px;
      "
    >
      <DashboardCard 
        {card}
        isDragging={draggedCard?.id === card.id}
        cardWidth={cardDimensions.width}
        cardHeight={cardDimensions.height}
        on:dragstart={handleCardDrag}
        on:delete={handleCardDelete}
      />
    </div>
  {/each}

  <!-- Empty state -->
  {#if cards.length === 0}
    <div class="flex items-center justify-center h-full">
      <div class="text-center text-white/50">
        <div class="text-6xl mb-4">📊</div>
        <h3 class="text-xl font-semibold mb-2">No cards yet</h3>
        <p>Click the "Add Card" button to get started</p>
      </div>
    </div>
  {/if}
</div>