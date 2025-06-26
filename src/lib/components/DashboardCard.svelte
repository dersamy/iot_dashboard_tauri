<script>
  import { createEventDispatcher } from 'svelte';
  import CardContent from './CardContent.svelte';

  export let card;
  export let isDragging = false;
  export let cardWidth = 280;
  export let cardHeight = 280;

  const dispatch = createEventDispatcher();

  let cardElement;
  let showDeleteButton = false;

  // Calculate actual content dimensions (minus padding and header)
  $: headerHeight = isSmallCard ? 60 : cardWidth < 300 ? 70 : 80;
  $: contentWidth = cardWidth;
  $: contentHeight = cardHeight - headerHeight; // Subtract header height

  // Responsive header sizing
  $: isSmallCard = cardWidth < 200 || cardHeight < 200;
  $: headerPadding = isSmallCard ? 'p-3' : cardWidth < 300 ? 'p-4' : 'p-5';
  $: contentPadding = isSmallCard ? 'p-3' : cardWidth < 300 ? 'p-4' : 'p-5';
  $: iconSize = isSmallCard ? 'text-xl' : cardWidth < 300 ? 'text-2xl' : 'text-3xl';
  $: titleSize = isSmallCard ? 'text-sm' : cardWidth < 300 ? 'text-base' : 'text-lg';
  $: subtitleSize = isSmallCard ? 'text-xs' : 'text-sm';
  $: deleteButtonSize = isSmallCard ? 'w-6 h-6 text-xs' : 'w-8 h-8 text-sm';

  function handleMouseDown(event) {
    if (event.target.closest('.card-controls')) return;
    
    dispatch('dragstart', {
      card,
      startX: event.clientX,
      startY: event.clientY
    });
  }

  function handleDelete() {
    dispatch('delete', { id: card.id });
  }

  function handleMouseEnter() {
    showDeleteButton = true;
  }

  function handleMouseLeave() {
    if (!isDragging) {
      showDeleteButton = false;
    }
  }
</script>

<div
  bind:this={cardElement}
  class="card-container h-full w-full cursor-move select-none group"
  class:dragging={isDragging}
  on:mousedown={handleMouseDown}
  on:mouseenter={handleMouseEnter}
  on:mouseleave={handleMouseLeave}
  role="button"
  tabindex="0"
>
  <!-- Card background with glassmorphism -->
  <div class="relative h-full w-full backdrop-blur-lg bg-white/10 rounded-xl border border-white/20 shadow-xl overflow-hidden transition-all duration-300 hover:bg-white/15 hover:border-white/30 flex flex-col">
    
    <!-- Delete button - Responsive size -->
    <div class="card-controls absolute top-2 right-2 z-20 opacity-0 group-hover:opacity-100 transition-opacity duration-200">
      <button
        on:click|stopPropagation={handleDelete}
        class="bg-red-500/80 hover:bg-red-500 text-white rounded-full {deleteButtonSize} flex items-center justify-center font-bold backdrop-blur-sm border border-red-400/50 transition-all duration-200 hover:scale-110"
        title="Delete card"
      >
        ×
      </button>
    </div>

    <!-- Card header - Responsive sizing -->
    <div class="{headerPadding} border-b border-white/10 flex-shrink-0">
      <div class="flex items-center gap-2 sm:gap-3">
        <div class="{iconSize} flex-shrink-0">
          {#if card.type === 'temperature'}🌡️
          {:else if card.type === 'humidity'}💧
          {:else if card.type === 'pir'}🚶
          {:else if card.type === 'gas'}💨
          {:else if card.type === 'ldr'}💡
          {:else if card.type === 'potentiometer'}🎚️
          {:else if card.type === 'light'}💡
          {:else if card.type === 'motion'}🚶
          {:else if card.type === 'pressure'}📊
          {:else if card.type === 'power'}⚡
          {:else if card.type === 'camera'}📹
          {:else if card.type === 'audio'}🔊
          {:else}📱{/if}
        </div>
        <div class="min-w-0 flex-1">
          <h3 class="text-white font-semibold {titleSize} truncate">{card.title}</h3>
          <p class="text-white/60 {subtitleSize} capitalize truncate">{card.type} sensor</p>
        </div>
      </div>
    </div>

    <!-- Card content - Responsive -->
    <div class="flex-1 min-h-0">
      <CardContent 
        {card} 
        cardWidth={contentWidth}
        cardHeight={contentHeight}
      />
    </div>

    <!-- Drag indicator -->
    {#if isDragging}
      <div class="absolute inset-0 bg-blue-400/20 border-2 border-blue-400/50 rounded-xl"></div>
    {/if}
  </div>
</div>

<style>
  .card-container.dragging {
    transform: rotate(5deg) scale(1.05);
    z-index: 1000;
  }

  .card-container:active {
    cursor: grabbing;
  }
</style>