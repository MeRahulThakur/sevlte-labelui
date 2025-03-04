<script>
  export let imageSrc;
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

  img {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
  }
</style>

<div class="image-container">
  {#if imageSrc}
    <img src={imageSrc} alt="Uploaded">
  {:else}
    Image Preview Area
  {/if}
</div>
