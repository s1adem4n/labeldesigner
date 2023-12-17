<script lang="ts">
  import Generator from "./lib/Generator.svelte";
  import FileInput from "./lib/FileInput.svelte";

  let image: File | null = null;
</script>

<div class="flex w-full h-full p-4 gap-4 overflow-hidden">
  {#if image}
    <Generator {image} />
  {:else}
    <FileInput bind:input={image} />
  {/if}
</div>
