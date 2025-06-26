<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  function handleClick() {
    dispatch('click');
  }
</script>

<button
  on:click={handleClick}
  class="group backdrop-blur-md bg-white/10 hover:bg-white/20 text-white font-semibold py-2 sm:py-3 px-4 sm:px-6 rounded-lg sm:rounded-xl border border-white/20 hover:border-white/40 transition-all duration-300 shadow-lg hover:shadow-xl transform hover:scale-105 active:scale-95 flex items-center gap-2 sm:gap-3 w-full sm:w-auto justify-center"
>
  <!-- Plus icon with animation -->
  <div class="relative">
    <div class="w-4 sm:w-5 h-4 sm:h-5 flex items-center justify-center">
      <div class="absolute w-2.5 sm:w-3 h-0.5 bg-white rounded-full transition-all duration-300 group-hover:w-3 sm:group-hover:w-4"></div>
      <div class="absolute w-0.5 h-2.5 sm:h-3 bg-white rounded-full transition-all duration-300 group-hover:h-3 sm:group-hover:h-4"></div>
    </div>
  </div>
  
  <span class="text-xs sm:text-sm">Add Card</span>
  
  <!-- Glow effect -->
  <div class="absolute inset-0 rounded-lg sm:rounded-xl bg-gradient-to-r from-blue-400/20 to-purple-400/20 opacity-0 group-hover:opacity-100 transition-opacity duration-300 -z-10 blur-sm"></div>
</button>

<style>
  button {
    position: relative;
  }
</style>