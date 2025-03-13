<script lang="ts">
  import { onMount } from "svelte";

  interface IImageContainerProps {
    imageSrc?: string | null;
    labelDataSet?: { mappingKey: string, x0: number, y0: number, x1: number, y1: number }[] | null;
    onLabelBoxClick?: (rects: { mappingKey: string, x0: number, y0: number, x1: number, y1: number }) => void;
    onBoxSelectAnimate?: boolean;
    selectLabelBoxes?: { mappingKey: string, x0: number, y0: number, x1: number, y1: number }[] | null;
  }

  let { imageSrc = null, labelDataSet = null, onLabelBoxClick, onBoxSelectAnimate = false, selectLabelBoxes = [] }: IImageContainerProps = $props();
  let canvas = $state<HTMLCanvasElement | null>(null);
  let dashPhase = 0;
  const dashSpeed = 0.5;
  let firstRenderDone = false;
  let animationFrameId: number | null = null;

  function drawImageWithRectangles(ctx: CanvasRenderingContext2D, dashed = false) {
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

      if (selectLabelBoxes?.length) {
        if (dashed) {
          // Animate selected boxes
          ctx.strokeStyle = "blue";
          ctx.lineWidth = 3;
          ctx.setLineDash([6, 6]);
          ctx.lineDashOffset = -dashPhase;
        } else {
          // Static highlight (if animation is off)
          ctx.strokeStyle = "green";
          ctx.lineWidth = 3;
          ctx.setLineDash([]); // Solid line
        }
        selectLabelBoxes.forEach(({ x0, y0, x1, y1 }) => {
          ctx.strokeRect(x0, y0, x1 - x0, y1 - y0);
        });
      }
    };
  }

  function animateDashedRectangles(ctx: CanvasRenderingContext2D) {
    if (!canvas || selectLabelBoxes?.length === 0 || !onBoxSelectAnimate) return;

    if (!firstRenderDone) {
      drawImageWithRectangles(ctx);
      firstRenderDone = true;
    } else {
      drawImageWithRectangles(ctx, true);
    }

    dashPhase = (dashPhase + dashSpeed) % 12;
    animationFrameId = requestAnimationFrame(() => animateDashedRectangles(ctx));
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
      drawImageWithRectangles(ctx);
    };
  }

  function handleCanvasClick(event: MouseEvent) {
    if (!canvas || !labelDataSet) return;
    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / rect.width; // Scale factor for X
    const scaleY = canvas.height / rect.height; // Scale factor for Y

    // Convert mouse coordinates to match canvas scaling
    const clickX = (event.clientX - rect.left) * scaleX;
    const clickY = (event.clientY - rect.top) * scaleY;

    let clickedRects = labelDataSet.filter(({ x0, y0, x1, y1 }) =>
      clickX >= x0 && clickX <= x1 && clickY >= y0 && clickY <= y1
    );

    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    if (clickedRects.length > 0) {
      selectLabelBoxes = clickedRects;
      stopAnimation();
      if (onBoxSelectAnimate) {
        animateDashedRectangles(ctx);
      } else {
        drawImageWithRectangles(ctx, false); // Ensure static highlight is drawn
      }
      onLabelBoxClick && onLabelBoxClick(clickedRects[0]);
    } else {
      selectLabelBoxes = [];
      stopAnimation();
      drawImageWithRectangles(ctx);
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
    if (selectLabelBoxes && selectLabelBoxes.length > 0) {
    stopAnimation();
    const ctx = canvas?.getContext("2d");
    if (ctx) {
      if (onBoxSelectAnimate) {
        animateDashedRectangles(ctx);
      } else {
        drawImageWithRectangles(ctx, false);
      }
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
