<script lang="ts">
    import { onMount } from "svelte";

  interface IImageContainerProps {
    imageSrc?: string | null;
    labelDataSet?: { x0: number, y0: number, x1: number, y1: number }[] | null;
    onLabelBoxClick?:(rect: { x0: number, y0: number, x1: number, y1: number }) => void;
  }
  let {imageSrc = null, labelDataSet = null, onLabelBoxClick}: IImageContainerProps= $props();
  let canvas = $state<HTMLCanvasElement | null>(null);

  function drawRectangles(ctx: CanvasRenderingContext2D) {
    ctx.strokeStyle = "red";
    ctx.lineWidth = 2;
    labelDataSet?.forEach(({ x0, y0, x1, y1 }) => {
        ctx.strokeRect(x0, y0, x1 - x0, y1 - y0);
    });
  }

  function renderImage() {
    if (!canvas || !imageSrc || !labelDataSet) return;
    const ctx = canvas.getContext("2d");
    if (!ctx) return;
    const img = new Image();
    img.src = imageSrc;
    img.onload = () => {
      if (!canvas) return;
      canvas.width = img.width;
      canvas.height = img.height;
      ctx.drawImage(img, 0, 0);
      drawRectangles(ctx);
    };
  }

  function handleCanvasClick(event: MouseEvent) {
    if (!canvas || !labelDataSet) return
    const rect = canvas.getBoundingClientRect();
    const clickX = event.clientX - rect.left;
    const clickY = event.clientY - rect.top
    for (const eachLabel of labelDataSet) {
        const { x0, y0, x1, y1 } = eachLabel;
        if (clickX >= x0 && clickX <= x1 && clickY >= y0 && clickY <= y1) {
            console.log('clicked label-',eachLabel)
            onLabelBoxClick && onLabelBoxClick(eachLabel); // Call the parent-provided callback function
            break;
        }
    }
  }

  onMount(() => {
    if (labelDataSet) {
        renderImage();
    }
  });

  $effect(() => {
    if (imageSrc && labelDataSet) {
      renderImage();
    }
  });
</script>

<style>
  .image-container {
    background: lightgray;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    overflow: hidden;
  }

  .image-container {
    overflow: auto;
    height: var(--dynamic-height, auto);
  }
  .canvas-container, img {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
  }
</style>

<div class="image-container">
  {#if labelDataSet}
    <canvas bind:this={canvas} class="canvas-container" onclick={handleCanvasClick}></canvas>
  {:else if imageSrc}
    <img src={imageSrc} alt="Uploaded">
  {:else}
    Image Preview Area
  {/if}
</div>
