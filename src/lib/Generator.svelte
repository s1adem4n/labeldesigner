<script lang="ts">
  import { jsPDF } from "jspdf";
  import * as pdfjsLib from "pdfjs-dist";
  import * as pdfjsWorker from "pdfjs-dist/build/pdf.worker.min.mjs";
  import { onDestroy, onMount } from "svelte";

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

  const renderPage = async (page: pdfjsLib.PDFPageProxy) => {
    const viewport = page.getViewport({ scale: 1 });
    pdfCanvas.height = viewport.height;
    pdfCanvas.width = viewport.width;

    await page.render({
      canvasContext: ctx,
      viewport,
    }).promise;
  };

  $: generatePdf(props);
  $: if (pdfResult) {
    pdfjsLib
      .getDocument({
        url: pdfResult,
        worker,
      })
      .promise.then((pdf) => {
        pdfPagesCount = pdf.numPages;
        if (pdfPageCurrent > pdfPagesCount) {
          pdfPageCurrent = 1;
        }
        pdf.getPage(pdfPageCurrent).then(renderPage);
      });
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

  <a href={pdfResult} download="result.pdf" class="rounded-md overflow-hidden">
    <button
      class="py-1 px-2 w-full text-white bg-blue-500 hover:bg-blue-600 border border-blue-400 hover:border-blue-500"
      disabled={!pdfResult}
      tabindex="-1"
    >
      Speichern
    </button>
  </a>
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
    class="mx-auto border border-gray-300 rounded-md h-full w-auto"
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
