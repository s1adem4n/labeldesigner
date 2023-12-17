<script lang="ts">
  export let input: File | null = null;
  let inputElement: HTMLInputElement;

  const drop = (e: DragEvent) => {
    e.preventDefault();
    const file = e.dataTransfer?.files[0];
    // check if file is an image
    if (file && file.type.startsWith("image")) {
      input = file;
    } else {
      alert("Bitte wähle ein Bild aus");
    }
  };
</script>

<div class="w-full h-full p-4">
  <button
    class="flex w-full h-full justify-center items-center bg-gray-100 rounded-xl border-4 border-gray-300 border-dashed"
    on:drop={drop}
    on:dragover={(e) => {
      e.preventDefault();
    }}
    on:click={() => {
      inputElement.click();
    }}
  >
    <svg
      class="w-8 h-8 text-gray-400"
      fill="none"
      stroke="currentColor"
      viewBox="0 0 24 24"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
        d="M12 6v6m0 0v6m0-6h6m-6 0H6"
      ></path>
    </svg>
    <p class="text-gray-400">Bild auswählen</p>
    <input
      type="file"
      accept="image/*"
      class="hidden"
      bind:this={inputElement}
      on:change={() => {
        if (inputElement.files) {
          input = inputElement.files[0];
        }
      }}
    />
  </button>
</div>
