<script lang="ts">
  import { jsPDF } from "jspdf";
  import * as pdfjsLib from "pdfjs-dist";
  import * as pdfjsWorker from "pdfjs-dist/build/pdf.worker.min.mjs";
  import { onDestroy, onMount } from "svelte";

  function dataURItoBlob(dataURI: string): Blob {
    const base64String = dataURI.split(",")[1];
    const byteString = atob(base64String);

    const mimeString = dataURI.split(",")[0].split(":")[1].split(";")[0];

    const ab = new ArrayBuffer(byteString.length);
    const ia = new Uint8Array(ab);

    for (let i = 0; i < byteString.length; i++) {
      ia[i] = byteString.charCodeAt(i);
    }

    const blob = new Blob([ab], { type: mimeString });
    return blob;
  }

  pdfjsLib.GlobalWorkerOptions.workerSrc = pdfjsWorker;
  const worker = new pdfjsLib.PDFWorker();
  onDestroy(() => {
    worker.destroy();
  });

  interface Props {
    height: number;
    width: number;
    count: number;
    gap: number;
    margin: number;
  }

  export let image: File;

  let props: Props = {
    height: 50,
    width: 0,
    count: 5,
    gap: 10,
    margin: 10,
  };

  let pdfCanvas: HTMLCanvasElement;
  let pdfPagesCount: number;
  let pdfPageCurrent: number = 1;

  let ctx: CanvasRenderingContext2D;

  onMount(async () => {
    ctx = pdfCanvas.getContext("2d")!;
  });

  const renderPdf = async (src: string) => {
    const pdf = await pdfjsLib.getDocument(src).promise;
    pdfPagesCount = pdf.numPages;
    if (pdfPageCurrent > pdfPagesCount) {
      pdfPageCurrent = pdfPagesCount;
    }
    const page = await pdf.getPage(pdfPageCurrent);

    const viewport = page.getViewport({ scale: 1 });
    pdfCanvas.height = viewport.height;
    pdfCanvas.width = viewport.width;

    page.render({
      canvasContext: ctx,
      viewport,
    }).promise;
  };

  $: generatePdf(props);
  $: if (pdfResult && pdfPageCurrent) {
    renderPdf(pdfResult);
  }

  let pdfResult: string | null = null;

  const generatePdf = async ({ width, height, count, margin, gap }: Props) => {
    if (!image) {
      return;
    }
    const pdf = new jsPDF({
      orientation: "portrait",
      unit: "mm",
    });
    const img = new Image();
    img.src = URL.createObjectURL(image);

    await new Promise((resolve) => {
      img.onload = resolve;
    });

    let size = {
      width,
      height,
    };

    if (props.width === 0 && height === 0) {
      return;
    }

    let ratio = img.width / img.height;
    if (width === 0) {
      size.width = size.height * ratio;
    }
    if (height === 0) {
      size.height = size.width / ratio;
    }

    let pos = {
      x: margin,
      y: margin,
    };

    for (let i = 0; i < count; i++) {
      pdf.addImage(
        img,
        pos.x,
        pos.y,
        size.width,
        size.height,
        undefined,
        "FAST"
      );
      pos.x += size.width + gap;
      if (pos.x + size.width >= pdf.internal.pageSize.getWidth() - margin) {
        pos.x = props.margin;
        pos.y += size.height + gap;
      }
      if (
        pos.y + size.height >= pdf.internal.pageSize.getHeight() - margin &&
        i < count - 1
      ) {
        pdf.addPage();

        pos.x = margin;
        pos.y = margin;
      }
    }

    pdfResult = pdf.output("datauristring");
  };

  const downloadPdfResult = () => {
    if (pdfResult) {
      const blob = dataURItoBlob(pdfResult);
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "result.pdf";
      a.click();
    }
  };
</script>

<div
  class="bg-gray-100 p-2 rounded-md flex flex-col gap-2 overflow-y-auto w-full sm:w-auto"
>
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

  <button
    class="py-1 rounded-md px-2 w-full text-white bg-blue-500 hover:bg-blue-600 border border-blue-400 hover:border-blue-500"
    disabled={!pdfResult}
    on:click={downloadPdfResult}
  >
    Speichern
  </button>
</div>

<div class="flex flex-col w-full h-full gap-2">
  <div class="flex gap-2 items-center justify-center">
    <span>Seite</span>
    <input
      type="number"
      value="1"
      on:input={(e) => {
        const input = e.currentTarget.valueAsNumber;
        if (input > 0 && input <= pdfPagesCount) {
          pdfPageCurrent = input;
        } else {
          e.currentTarget.value = pdfPageCurrent.toString();
        }
      }}
      class="w-16"
    />
    <span>von {pdfPagesCount}</span>
  </div>
  <canvas
    id="pdfCanvas"
    class="mx-auto border border-gray-300 rounded-md w-full h-auto md:w-auto md:h-full"
    bind:this={pdfCanvas}
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
