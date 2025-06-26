<script>
  import { createEventDispatcher } from 'svelte';
  import { fly, fade } from 'svelte/transition';

  export let cardTypes = [];

  const dispatch = createEventDispatcher();

  let selectedCardType = null;
  let customTitle = '';
  let modalElement;

  function handleClose() {
    dispatch('close');
  }

  function handleBackdropClick(event) {
    if (event.target === event.currentTarget) {
      handleClose();
    }
  }

  function handleCardTypeSelect(cardType) {
    selectedCardType = cardType;
    customTitle = cardType.name;
  }

  function handleAddCard() {
    if (selectedCardType) {
      dispatch('add', {
        cardType: selectedCardType,
        title: customTitle.trim() || selectedCardType.name
      });
    }
  }

  function handleKeydown(event) {
    if (event.key === 'Escape') {
      handleClose();
    }
  }

  // Get the appropriate icon based on the sensor type
  function getCardIcon(cardType) {
    switch (cardType.id) {
      case 'temperature':
        return '🌡️';
      case 'humidity':
        return '💧';
      case 'pir':
        return '🚶';
      case 'gas':
        return '💨';
      case 'ldr':
        return '💡';
      case 'potentiometer':
        return '🎚️';
      default:
        return cardType.icon || '📱';
    }
  }

  // Get sensor description
  function getSensorDescription(cardType) {
    switch (cardType.id) {
      case 'temperature':
        return 'Measures ambient temperature';
      case 'humidity':
        return 'Monitors air humidity levels';
      case 'pir':
        return 'Detects motion and movement';
      case 'gas':
        return 'Monitors air quality and gas levels';
      case 'ldr':
        return 'Measures light intensity';
      case 'potentiometer':
        return 'Variable resistance control';
      default:
        return `${cardType.name} monitoring`;
    }
  }

  // Get expected value range for preview
  function getValueRange(cardType) {
    switch (cardType.id) {
      case 'temperature':
        return '15-35°C';
      case 'humidity':
        return '30-70%';
      case 'pir':
        return 'Motion/No Motion';
      case 'gas':
        return '0-1024';
      case 'ldr':
        return '0-1024';
      case 'potentiometer':
        return '0-1024';
      default:
        return 'Variable';
    }
  }
</script>

<svelte:window on:keydown={handleKeydown} />

<!-- Modal backdrop - Increased opacity -->
<div
  class="fixed inset-0 bg-black/70 backdrop-blur-sm z-50 flex items-center justify-center p-2 sm:p-4"
  on:click={handleBackdropClick}
  transition:fade={{ duration: 200 }}
>
  <!-- Modal content - Increased background opacity -->
  <div
    bind:this={modalElement}
    class="w-full max-w-xs sm:max-w-md md:max-w-2xl lg:max-w-4xl xl:max-w-5xl h-full max-h-[95vh] sm:max-h-[90vh] backdrop-blur-lg bg-white/20 rounded-xl sm:rounded-2xl border border-white/30 shadow-2xl overflow-hidden flex flex-col"
    transition:fly={{ y: 50, duration: 300 }}
  >
    <!-- Modal header - Fixed -->
    <div class="flex-shrink-0 p-3 sm:p-4 md:p-6 border-b border-white/20">
      <div class="flex items-center justify-between">
        <div>
          <h2 class="text-lg sm:text-xl md:text-2xl font-bold text-white">
            Add New Sensor Card
          </h2>
          <p class="text-white/80 mt-1 text-xs sm:text-sm">
            Choose a sensor type for your IoT dashboard
          </p>
        </div>
        <button
          on:click={handleClose}
          class="text-white/70 hover:text-white text-xl sm:text-2xl font-light transition-colors duration-200 hover:bg-white/20 rounded-full w-6 h-6 sm:w-8 sm:h-8 flex items-center justify-center flex-shrink-0 ml-2"
        >
          ×
        </button>
      </div>
    </div>

    <!-- Modal body - Scrollable -->
    <div class="flex-1 overflow-y-auto">
      <div class="p-3 sm:p-4 md:p-6">
        <!-- Card type selection -->
        <div class="mb-4 sm:mb-6">
          <h3 class="text-sm sm:text-base md:text-lg font-semibold text-white mb-3 sm:mb-4">
            Select Sensor Type
          </h3>
          <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-2 sm:gap-3 md:gap-4 mb-4 sm:mb-6">
            {#each cardTypes as cardType (cardType.id)}
              <button
                on:click={() => handleCardTypeSelect(cardType)}
                class="p-2 sm:p-3 md:p-4 rounded-lg sm:rounded-xl border transition-all duration-200 text-left group hover:scale-105 {selectedCardType?.id === cardType.id ? 'bg-blue-500/30 border-blue-400/60' : 'bg-white/15 border-white/30 hover:bg-white/25 hover:border-white/40'}"
              >
                <div class="flex items-start gap-2 sm:gap-3">
                  <div class="text-xl sm:text-2xl md:text-3xl group-hover:scale-110 transition-transform duration-200 flex-shrink-0">
                    {getCardIcon(cardType)}
                  </div>
                  <div class="flex-1 min-w-0">
                    <div class="text-white text-xs sm:text-sm font-medium mb-1 truncate">
                      {cardType.name}
                    </div>
                    <div class="text-white/70 text-xs mb-1 sm:mb-2 line-clamp-2">
                      {getSensorDescription(cardType)}
                    </div>
                    <div class="text-white/50 text-xs">
                      Range: {getValueRange(cardType)}
                    </div>
                  </div>
                </div>
              </button>
            {/each}
          </div>
        </div>

        <!-- Custom title input -->
        {#if selectedCardType}
          <div class="mb-4 sm:mb-6" transition:fly={{ y: 20, duration: 200 }}>
            <label for="cardTitle" class="block text-white font-medium mb-2 text-sm sm:text-base">
              Card Title
            </label>
            <input
              id="cardTitle"
              type="text"
              bind:value={customTitle}
              placeholder="Enter custom title..."
              class="w-full px-3 sm:px-4 py-2 sm:py-3 rounded-lg sm:rounded-xl backdrop-blur-md bg-white/20 border border-white/30 text-white placeholder-white/60 focus:outline-none focus:border-blue-400/60 focus:bg-white/25 transition-all duration-200 text-sm sm:text-base"
            />
            <div class="text-white/60 text-xs mt-1">
              Leave empty to use default: "{selectedCardType.name}"
            </div>
          </div>

          <!-- Enhanced Preview -->
          <div class="mb-4 sm:mb-6 p-3 sm:p-4 rounded-lg sm:rounded-xl backdrop-blur-md bg-white/15 border border-white/20">
            <h4 class="text-white font-medium mb-3 text-sm sm:text-base">Card Preview</h4>
            
            <!-- Mock card preview -->
            <div class="bg-white/20 rounded-lg sm:rounded-xl border border-white/30 p-3 sm:p-4 max-w-full sm:max-w-sm mx-auto">
              <!-- Card header -->
              <div class="flex items-center gap-2 sm:gap-3 mb-3 pb-3 border-b border-white/20">
                <div class="text-lg sm:text-xl md:text-2xl flex-shrink-0">
                  {getCardIcon(selectedCardType)}
                </div>
                <div class="min-w-0 flex-1">
                  <div class="text-white font-semibold text-xs sm:text-sm truncate">
                    {customTitle || selectedCardType.name}
                  </div>
                  <div class="text-white/70 text-xs capitalize">
                    {selectedCardType.id} sensor
                  </div>
                </div>
              </div>
              
              <!-- Mock sensor data -->
              <div class="text-center py-3 sm:py-4">
                <div class="text-lg sm:text-xl font-bold text-white mb-2">
                  {#if selectedCardType.id === 'temperature'}
                    22.5°C
                  {:else if selectedCardType.id === 'humidity'}
                    45%
                  {:else if selectedCardType.id === 'pir'}
                    No Motion
                  {:else if selectedCardType.id === 'gas'}
                    250/1024
                  {:else if selectedCardType.id === 'ldr'}
                    512/1024
                  {:else if selectedCardType.id === 'potentiometer'}
                    750/1024
                  {:else}
                    --
                  {/if}
                </div>
                <div class="flex items-center justify-center gap-2 mb-3">
                  <div class="w-2 h-2 rounded-full bg-green-400 animate-pulse"></div>
                  <span class="text-xs text-green-400">Active</span>
                </div>
                
                <!-- Mock additional info -->
                {#if selectedCardType.id === 'temperature'}
                  <div class="text-xs sm:text-sm text-green-400">Comfortable</div>
                {:else if selectedCardType.id === 'humidity'}
                  <div class="text-xs sm:text-sm text-green-400">Optimal</div>
                {:else if selectedCardType.id === 'gas'}
                  <div class="text-xs sm:text-sm text-green-400">Good</div>
                {:else if selectedCardType.id === 'ldr'}
                  <div class="text-xs sm:text-sm text-green-400">Normal</div>
                {/if}
              </div>
              
              <!-- Mock progress bar -->
              {#if selectedCardType.id !== 'pir'}
                <div class="mt-3 pt-3 border-t border-white/20">
                  <div class="h-1 bg-white/20 rounded-full overflow-hidden">
                    <div class="h-full bg-gradient-to-r from-green-400 to-green-500 rounded-full" style="width: 45%"></div>
                  </div>
                </div>
              {/if}
            </div>
          </div>

          <!-- Sensor specifications -->
          <div class="bg-white/15 rounded-lg sm:rounded-xl p-3 sm:p-4 border border-white/20">
            <h5 class="text-white font-medium mb-2 text-sm sm:text-base">Sensor Specifications</h5>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-2 sm:gap-3 text-xs sm:text-sm">
              <div class="flex flex-col sm:flex-row sm:items-center">
                <span class="text-white/70 sm:mr-2">Type:</span>
                <span class="text-white capitalize">{selectedCardType.id}</span>
              </div>
              <div class="flex flex-col sm:flex-row sm:items-center">
                <span class="text-white/70 sm:mr-2">Range:</span>
                <span class="text-white">{getValueRange(selectedCardType)}</span>
              </div>
              <div class="flex flex-col sm:flex-row sm:items-center">
                <span class="text-white/70 sm:mr-2">Update Rate:</span>
                <span class="text-white">2-5 seconds</span>
              </div>
              <div class="flex flex-col sm:flex-row sm:items-center">
                <span class="text-white/70 sm:mr-2">Status:</span>
                <span class="text-green-400">Real-time</span>
              </div>
            </div>
          </div>
        {/if}
      </div>
    </div>

    <!-- Modal footer - Fixed -->
    <div class="flex-shrink-0 p-3 sm:p-4 md:p-6 border-t border-white/20 flex gap-2 sm:gap-3 justify-end">
      <button
        on:click={handleClose}
        class="px-3 sm:px-4 md:px-6 py-2 rounded-lg sm:rounded-xl text-white/80 hover:text-white hover:bg-white/20 transition-all duration-200 font-medium text-sm sm:text-base"
      >
        Cancel
      </button>
      <button
        on:click={handleAddCard}
        disabled={!selectedCardType}
        class="px-3 sm:px-4 md:px-6 py-2 rounded-lg sm:rounded-xl bg-blue-500/80 hover:bg-blue-500 disabled:bg-white/20 disabled:text-white/50 text-white font-medium transition-all duration-200 disabled:cursor-not-allowed flex items-center gap-1 sm:gap-2 text-sm sm:text-base"
      >
        <span>Add Card</span>
        {#if selectedCardType}
          <span class="text-base sm:text-lg">{getCardIcon(selectedCardType)}</span>
        {/if}
      </button>
    </div>
  </div>
</div>

<style>
  .line-clamp-2 {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
</style>