<script lang="ts">
    import "../app.css";
    let { children } = $props();
</script>

<div  style="display:flex; flex-direction: column; gap: 1rem;">
<div class="bg-blue-300">
<a href="/">home</a>
<a href="/test">test</a>
<a href="/dndtest">dnd</a>
<a href="/testbis">bis</a>
</div>

{@render children()}
</div>
