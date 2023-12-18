<script lang="ts">
  import Generator from "./lib/Generator.svelte";
  import FileInput from "./lib/FileInput.svelte";

  let image: File | null = null;
</script>

{#if image}
  <div class="flex flex-col md:flex-row w-full min-h-full md:h-full p-4 gap-2">
    <Generator {image} />
  </div>
{:else}
  <FileInput bind:input={image} />
{/if}
