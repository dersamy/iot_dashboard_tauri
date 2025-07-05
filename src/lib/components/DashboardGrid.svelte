<script>
  import { createEventDispatcher } from 'svelte';
  import DashboardCard from './DashboardCard.svelte';

  export let cards = [];

  const dispatch = createEventDispatcher();

  let gridContainer;
  let draggedCard = null;
  let dragOffset = { x: 0, y: 0 };
  let isDragging = false;
  let dragPosition = { x: 0, y: 0 };
  let animationId = null;

  const GRID_SIZE = 280;
  const GAP = 20;

  // Simple collision detection
  function isPositionOccupied(x, y, width, height, excludeCardId = null) {
    return cards.some(card => {
      if (card.id === excludeCardId) return false;
      
      return !(
        x >= card.x + card.w ||
        x + width <= card.x ||
        y >= card.y + card.h ||
        y + height <= card.y
      );
    });
  }

  function findNearestAvailablePosition(targetX, targetY, width, height, excludeCardId = null) {
    if (!isPositionOccupied(targetX, targetY, width, height, excludeCardId)) {
      return { x: targetX, y: targetY };
    }

    // Simple spiral search
    for (let radius = 1; radius <= 5; radius++) {
      for (let dx = -radius; dx <= radius; dx++) {
        for (let dy = -radius; dy <= radius; dy++) {
          if (Math.abs(dx) !== radius && Math.abs(dy) !== radius) continue;
          
          const testX = Math.max(0, targetX + dx);
          const testY = Math.max(0, targetY + dy);
          
          if (!isPositionOccupied(testX, testY, width, height, excludeCardId)) {
            return { x: testX, y: testY };
          }
        }
      }
    }
    
    return { x: targetX, y: targetY };
  }

  function handleCardDrag(event) {
    const { card, startX, startY } = event.detail;
    draggedCard = card;
    isDragging = true;
    
    const rect = gridContainer.getBoundingClientRect();
    dragOffset.x = startX - rect.left - (card.x * (GRID_SIZE + GAP));
    dragOffset.y = startY - rect.top - (card.y * (GRID_SIZE + GAP));
    
    // Initial drag position
    dragPosition = {
      x: card.x * (GRID_SIZE + GAP),
      y: card.y * (GRID_SIZE + GAP)
    };

    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', handleMouseUp);
  }

  function handleMouseMove(event) {
    if (!draggedCard || !isDragging) return;

    if (animationId) {
      cancelAnimationFrame(animationId);
    }

    animationId = requestAnimationFrame(() => {
      const rect = gridContainer.getBoundingClientRect();
      const mouseX = event.clientX - rect.left - dragOffset.x;
      const mouseY = event.clientY - rect.top - dragOffset.y;

      // Update drag position for smooth visual feedback
      dragPosition = { x: mouseX, y: mouseY };

      // Calculate grid position for collision detection
      const gridX = Math.max(0, Math.round(mouseX / (GRID_SIZE + GAP)));
      const gridY = Math.max(0, Math.round(mouseY / (GRID_SIZE + GAP)));

      // Check for collision
      const wouldOverlap = isPositionOccupied(
        gridX, 
        gridY, 
        draggedCard.w, 
        draggedCard.h, 
        draggedCard.id
      );

      // Update only the dragged card's logical position WITHOUT triggering reactive updates
      const cardIndex = cards.findIndex(card => card.id === draggedCard.id);
      if (cardIndex !== -1) {
        cards[cardIndex] = { 
          ...cards[cardIndex], 
          x: gridX, 
          y: gridY, 
          isInvalidPosition: wouldOverlap 
        };
      }
    });
  }

  function handleMouseUp() {
    if (!draggedCard || !isDragging) return;

    if (animationId) {
      cancelAnimationFrame(animationId);
      animationId = null;
    }

    const currentCard = cards.find(card => card.id === draggedCard.id);
    
    const isValidPosition = !isPositionOccupied(
      currentCard.x,
      currentCard.y,
      currentCard.w,
      currentCard.h,
      draggedCard.id
    );

    let finalPosition;
    
    if (isValidPosition) {
      finalPosition = { x: currentCard.x, y: currentCard.y };
    } else {
      finalPosition = findNearestAvailablePosition(
        currentCard.x,
        currentCard.y,
        currentCard.w,
        currentCard.h,
        draggedCard.id
      );
    }

    // Create new cards array with updated position
    const updatedCards = cards.map(card => 
      card.id === draggedCard.id 
        ? { 
            ...card, 
            x: finalPosition.x, 
            y: finalPosition.y, 
            isInvalidPosition: false 
          }
        : card
    );

    // Dispatch update to parent
    dispatch('update', { cards: updatedCards });

    // Cleanup
    isDragging = false;
    draggedCard = null;
    dragPosition = { x: 0, y: 0 };
    document.removeEventListener('mousemove', handleMouseMove);
    document.removeEventListener('mouseup', handleMouseUp);
  }

  function handleCardDelete(event) {
    const cardId = event.detail.id;
    const updatedCards = cards.filter(card => card.id !== cardId);
    dispatch('delete', { id: cardId });
    // Don't modify positions when deleting
  }

  // Grid calculations - ensure minimum size
  $: maxX = Math.max(4, ...cards.map(card => card.x + card.w));
  $: maxY = Math.max(3, ...cards.map(card => card.y + card.h));
  $: gridWidth = maxX * (GRID_SIZE + GAP);
  $: gridHeight = maxY * (GRID_SIZE + GAP);

  function getCardDimensions(card) {
    return {
      width: card.w * GRID_SIZE + (card.w - 1) * GAP,
      height: card.h * GRID_SIZE + (card.h - 1) * GAP
    };
  }

  export function addCard(newCard) {
    const position = findNearestAvailablePosition(0, 0, newCard.w, newCard.h);
    const cardWithPosition = { ...newCard, ...position };
    
    // Don't modify existing cards when adding new ones
    const updatedCards = [...cards, cardWithPosition];
    dispatch('update', { cards: updatedCards });
  }
</script>

<div 
  bind:this={gridContainer}
  class="relative backdrop-blur-md bg-white/5 rounded-2xl border border-white/10 p-6 shadow-xl min-h-96"
  style="width: {gridWidth}px; height: {gridHeight}px;"
>
  <!-- Simple grid background -->
  <div class="absolute inset-6 opacity-10 pointer-events-none">
    <div 
      class="w-full h-full"
      style="
        background-image: 
          linear-gradient(rgba(255,255,255,0.1) 1px, transparent 1px),
          linear-gradient(90deg, rgba(255,255,255,0.1) 1px, transparent 1px);
        background-size: {GRID_SIZE + GAP}px {GRID_SIZE + GAP}px;
      "
    ></div>
  </div>

  <!-- Cards -->
  {#each cards as card (card.id)}
    {@const cardDimensions = getCardDimensions(card)}
    {@const isDraggedCard = draggedCard?.id === card.id}
    
    <div
      class="absolute"
      class:transition-all={!isDraggedCard}
      class:duration-300={!isDraggedCard}
      class:ease-out={!isDraggedCard}
      class:z-50={isDraggedCard}
      class:z-10={!isDraggedCard}
      style="
        left: {isDraggedCard ? dragPosition.x : card.x * (GRID_SIZE + GAP)}px;
        top: {isDraggedCard ? dragPosition.y : card.y * (GRID_SIZE + GAP)}px;
        width: {cardDimensions.width}px;
        height: {cardDimensions.height}px;
        transform: {isDraggedCard ? 'scale(1.05)' : 'scale(1)'};
      "
    >
      <DashboardCard 
        {card}
        isDragging={isDraggedCard}
        isInvalidPosition={card.isInvalidPosition || false}
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