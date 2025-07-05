<script>
  import { createEventDispatcher } from 'svelte';
  import CardContent from './CardContent.svelte';

  export let card;
  export let isDragging = false;
  export let isInvalidPosition = false;
  export let cardWidth = 280;
  export let cardHeight = 280;

  const dispatch = createEventDispatcher();

  let showDeleteButton = false;

  // Memoized responsive calculations
  $: isSmallCard = cardWidth < 200 || cardHeight < 200;
  $: headerHeight = isSmallCard ? 60 : cardWidth < 300 ? 70 : 80;
  $: contentHeight = cardHeight - headerHeight;

  // Icon mapping
  const iconMap = {
    temperature: '🌡️',
    humidity: '💧',
    pir: '🚶',
    gas: '💨',
    ldr: '💡',
    potentiometer: '🎚️',
    light: '💡',
    motion: '🚶',
    pressure: '📊',
    power: '⚡',
    camera: '📹',
    audio: '🔊',
    slider: '🎛️',
    toggle: '🔘',
    color: '🎨',
    text: '📝'
  };

  function handleMouseDown(event) {
    // Prevent drag when interacting with control elements
    if (event.target.closest('.delete-button') || 
        event.target.closest('.card-control') ||
        event.target.closest('input') || 
        event.target.closest('button:not(.drag-handle)') ||
        event.target.closest('textarea') ||
        event.target.closest('select')) {
      return;
    }
    
    dispatch('dragstart', {
      card,
      startX: event.clientX,
      startY: event.clientY
    });
  }

  function handleDelete() {
    dispatch('delete', { id: card.id });
  }
</script>

<div
  class="h-full w-full select-none group"
  class:cursor-move={!isDragging}
  class:cursor-grabbing={isDragging}
  on:mousedown={handleMouseDown}
  on:mouseenter={() => showDeleteButton = true}
  on:mouseleave={() => showDeleteButton = false}
  role="button"
  tabindex="0"
>
  <!-- Card background -->
  <div class="
    relative h-full w-full backdrop-blur-lg rounded-xl border shadow-xl overflow-hidden flex flex-col
    {isInvalidPosition ? 'bg-red-500/20 border-red-400/50' : 'bg-white/10 border-white/20'}
    {isDragging ? 'shadow-2xl' : 'hover:bg-white/15 hover:border-white/30'}
    {isDragging ? '' : 'transition-colors duration-200'}
  ">
    
    <!-- Delete button -->
    {#if showDeleteButton && !isDragging}
      <div class="delete-button absolute top-2 right-2 z-20 opacity-100 transition-opacity duration-200">
        <button
          on:click|stopPropagation={handleDelete}
          class="bg-red-500/80 hover:bg-red-500 text-white rounded-full w-8 h-8 flex items-center justify-center font-bold backdrop-blur-sm border border-red-400/50 transition-all duration-200 hover:scale-110"
          title="Delete card"
        >
          ×
        </button>
      </div>
    {/if}

    <!-- Card header with drag handle -->
    <div class="p-4 border-b border-white/10 flex-shrink-0 drag-handle cursor-move">
      <div class="flex items-center gap-3">
        <div class="text-2xl flex-shrink-0">
          {iconMap[card.type] || '📱'}
        </div>
        <div class="min-w-0 flex-1">
          <h3 class="text-white font-semibold text-base truncate">
            {card.title}
          </h3>
          <p class="text-white/60 text-sm capitalize truncate">
            {card.type === 'slider' || card.type === 'toggle' || card.type === 'color' || card.type === 'text' ? 'control' : 'sensor'}
          </p>
        </div>
      </div>
    </div>

    <!-- Card content -->
    <div class="flex-1 min-h-0">
      <CardContent 
        {card} 
        cardWidth={cardWidth}
        cardHeight={contentHeight}
      />
    </div>
  </div>
</div>