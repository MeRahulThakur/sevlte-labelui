<script lang="ts">
  import { onMount } from "svelte";

  interface IImageContainerProps {
    imageSrc?: string | null;
    labelDataSet?: { x0: number, y0: number, x1: number, y1: number }[] | null;
    onLabelBoxClick?: (rect: { x0: number, y0: number, x1: number, y1: number }) => void;
    onBoxSelectAnimate?: boolean;
    selectedLabelBox?: { x0: number, y0: number, x1: number, y1: number } | null;
  }

  let { imageSrc = null, labelDataSet = null, onLabelBoxClick, onBoxSelectAnimate = true, selectedLabelBox = null }: IImageContainerProps = $props();
  let canvas = $state<HTMLCanvasElement | null>(null);
  //let selectedLabelBox = $state<{ x0: number, y0: number, x1: number, y1: number } | null>(null);
  let dashPhase = 0;
  const dashSpeed = 0.5; // Adjust for smoother animation
  let firstRenderDone = false;

  let animationFrameId: number | null = null;

  function drawRectangles(ctx: CanvasRenderingContext2D) {
    if (!canvas || !imageSrc) return;

    const img = new Image();
    img.src = imageSrc;
    img.onload = () => {
      if (!canvas) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      
      ctx.strokeStyle = "red";
      ctx.lineWidth = 2;
      ctx.setLineDash([]);

      labelDataSet?.forEach(({ x0, y0, x1, y1 }) => {
        ctx.strokeRect(x0, y0, x1 - x0, y1 - y0);
      });

      // Keep selected rectangle highlighted with animation
      if (selectedLabelBox && onBoxSelectAnimate) {
        animateDashedRectangle(ctx);
      }
    };
  }

  function animateDashedRectangle(ctx: CanvasRenderingContext2D) {
    if (!canvas || !selectedLabelBox || !onBoxSelectAnimate) return;

    // Ensure the base image and static rectangles are drawn only once
    if (!firstRenderDone) {
      const img = new Image();
      img.src = imageSrc!;
      img.onload = () => {
        if (!canvas) return;
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        // Draw static bounding boxes
        ctx.strokeStyle = "red";
        ctx.lineWidth = 2;
        ctx.setLineDash([]);
        labelDataSet?.forEach(({ x0, y0, x1, y1 }) => {
          ctx.strokeRect(x0, y0, x1 - x0, y1 - y0);
        });
        firstRenderDone = true;
      };
    } else {
      // Instead of clearing everything, only redraw the selected bounding box
      const img = new Image();
      img.src = imageSrc!;
      img.onload = () => {
        if (!canvas) return;
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        // Draw static bounding boxes again
        ctx.strokeStyle = "red";
        ctx.lineWidth = 2;
        ctx.setLineDash([]);
        labelDataSet?.forEach(({ x0, y0, x1, y1 }) => {
            ctx.strokeRect(x0, y0, x1 - x0, y1 - y0);
        });
        // Animate only the selected bounding box
        ctx.strokeStyle = "blue";
        ctx.lineWidth = 3;
        ctx.setLineDash([6, 6]);
        ctx.lineDashOffset = -dashPhase;
        const { x0, y0, x1, y1 } = selectedLabelBox!;
        ctx.strokeRect(x0, y0, x1 - x0, y1 - y0);
      };
    }

    dashPhase = (dashPhase + dashSpeed) % 12; // Creates smooth animation loop
    animationFrameId = requestAnimationFrame(() => animateDashedRectangle(ctx));
  }

  function stopAnimation() {
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId);
      animationFrameId = null;
    }
  }

  function renderImage() {
    if (!canvas || !imageSrc) return;
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
    if (!canvas || !labelDataSet) return;
    const rect = canvas.getBoundingClientRect();
    const clickX = event.clientX - rect.left;
    const clickY = event.clientY - rect.top;

    let clickedRect = null;
    for (const eachLabel of labelDataSet) {
      const { x0, y0, x1, y1 } = eachLabel;
      if (clickX >= x0 && clickX <= x1 && clickY >= y0 && clickY <= y1) {
          clickedRect = eachLabel;
          break;
      }
    }

    if (clickedRect) {
      selectedLabelBox = clickedRect;
      stopAnimation();
      const ctx = canvas.getContext("2d");
      if (ctx) animateDashedRectangle(ctx);
      onLabelBoxClick && onLabelBoxClick(clickedRect);
    } else {
      selectedLabelBox = null;
      stopAnimation();
      const ctx = canvas.getContext("2d");
      if (ctx) drawRectangles(ctx); // Redraw without animation
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

  $effect(() => {
    if (selectedLabelBox) {
      if (animationFrameId) {
        cancelAnimationFrame(animationFrameId);
      }
      const ctx = canvas?.getContext("2d");
      if (ctx) {
        animateDashedRectangle(ctx);
      }
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
    cursor: pointer;
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
