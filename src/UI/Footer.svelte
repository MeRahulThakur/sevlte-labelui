<script lang="ts">
  interface IFooter {
    isSticky?:boolean;
  }
  let {isSticky = false} = $props();
</script>

<style>
  .footer {
      background: #333;
      color: white;
      text-align: center;
      padding: 10px;
      width: 100%;
  }

  .sticky {
      position: fixed;
      bottom: 0;
      left: 0;
  }
</style>

<footer class="footer {isSticky ? 'sticky' : ''}">
  <p>&copy; 2024 Your Company. All rights reserved.</p>
</footer>
