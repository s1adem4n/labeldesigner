<script lang="ts">
  import Generator from "./lib/Generator.svelte";
  import FileInput from "./lib/FileInput.svelte";

  let image: File | null = null;
</script>

<div class="flex flex-col md:flex-row w-full min-h-full p-4 gap-4">
  {#if image}
    <Generator {image} />
  {:else}
    <FileInput bind:input={image} />
  {/if}
</div>
