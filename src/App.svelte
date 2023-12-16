<script lang="ts">
  import { jsPDF } from "jspdf";

  interface Props {
    height: number;
    width: number;
    count: number;
    gap: number;
    margin: number;
    image: File | null;
  }

  let props: Props = {
    height: 50,
    width: 0,
    count: 5,
    gap: 10,
    margin: 10,
    image: null,
  };

  $: generatePdf(props);

  let pdfResult: string | null = null;

  const generatePdf = async ({
    width,
    height,
    count,
    margin,
    gap,
    image,
  }: Props) => {
    if (!image) {
      return;
    }
    const pdf = new jsPDF({
      orientation: "portrait",
      unit: "mm",
    });
    const img = new Image();
    img.src = URL.createObjectURL(image);
    // await img loaded
    await new Promise((resolve) => {
      img.onload = resolve;
    });

    let size = {
      width: width,
      height: height,
    };

    if (props.width === 0 && height === 0) {
      return;
    }

    let ratio = img.width / img.height;
    if (width === 0) {
      size.width = size.height * ratio;
      console.log(size.height, ratio);
    }
    if (height === 0) {
      size.height = size.width / ratio;
      console.log(size);
    }

    console.log(size);

    let pos = {
      x: margin,
      y: margin,
    };

    for (let i = 0; i < count; i++) {
      pdf.addImage(img, "JPEG", pos.x, pos.y, size.width, size.height);
      pos.x += size.width + gap;
      if (pos.x + size.width >= pdf.internal.pageSize.getWidth() - margin) {
        pos.x = props.margin;
        pos.y += size.height + gap;
      }
      if (pos.y + size.height >= pdf.internal.pageSize.getHeight() - margin) {
        pdf.addPage();
        pos.x = margin;
        pos.y = margin;
      }
    }

    pdfResult = pdf.output("datauristring");
  };
</script>

<div class="flex w-full h-full p-4 gap-4 overflow-hidden">
  <div class="flex flex-col gap-2 overflow-y-auto w-full sm:w-auto">
    <div class="flex flex-col">
      <label for="imageFile">Bild</label>
      <input
        id="imageFile"
        type="file"
        on:input={(e) => {
          if (e.currentTarget && e.currentTarget.files) {
            props.image = e.currentTarget.files[0];
          }
        }}
      />
    </div>

    <div class="flex flex-col">
      <label for="height">Höhe</label>
      <input
        id="height"
        placeholder="Höhe"
        type="number"
        bind:value={props.height}
      />
    </div>

    <div class="flex flex-col">
      <label for="width">Breite</label>
      <input
        id="width"
        placeholder="Breite"
        type="number"
        bind:value={props.width}
      />
    </div>

    <div class="flex flex-col">
      <label for="count">Anzahl</label>
      <input
        id="count"
        type="number"
        placeholder="Anzahl"
        bind:value={props.count}
      />
    </div>

    <div class="flex flex-col">
      <label for="gap">Abstand</label>
      <input
        id="gap"
        type="number"
        placeholder="Abstand"
        bind:value={props.gap}
      />
    </div>

    <div class="flex flex-col">
      <label for="margin">Abstand (Rand)</label>
      <input
        id="margin"
        type="number"
        placeholder="Rand"
        bind:value={props.margin}
      />
    </div>

    <a href={pdfResult} download="result.pdf">
      <button
        class="py-1 px-2 w-full text-white bg-blue-500 rounded-md hover:bg-blue-600 border border-blue-400 hover:border-blue-500"
        disabled={!pdfResult}
      >
        Download
      </button>
    </a>
  </div>
  <hr class="rounded-full h-full w-px hidden sm:inline bg-gray-200" />

  <embed
    src={pdfResult}
    type="application/pdf"
    class="w-full h-full rounded-md hidden sm:inline"
  />
</div>

<style lang="postcss">
  input:focus {
    @apply outline-none;
  }

  input {
    @apply border border-gray-300 rounded-md px-2 py-1;
  }
</style>
