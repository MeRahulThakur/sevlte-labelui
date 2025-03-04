<script lang="ts">
  interface IHeader {
    onUpload: (event: Event) => void;
  }

  let {onUpload}: IHeader = $props();
  let fileName = $state<string>("Choose Image");
   
  function handleFileChange(event: Event) {
    const target = event.target as HTMLInputElement;
    const file = target.files?.[0];

    if (file) {
      fileName = file.name; // Store the selected file name
      if (onUpload) {
        onUpload(event);
      }
    }
  }
</script>

<style>
  .header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 10px;
      background: #333;
      color: white;
  }

  .left {
      display: flex;
      align-items: center;
      gap: 10px;
  }

  .center {
      display: flex;
      gap: 10px;
  }

  input[type="file"] {
      display: none;
  }

  label {
      background: #555;
      padding: 8px 12px;
      border-radius: 5px;
      cursor: pointer;
  }

  button {
      background: #007BFF;
      color: white;
      border: none;
      padding: 8px 12px;
      cursor: pointer;
      border-radius: 5px;
  }
</style>

<header class="header">
  <div class="left">
      <img src="logo.png" alt="Logo" width="40">
      <h1>Site Title</h1>
  </div>
  <div class="center">
      <label for="fileInput">{fileName}</label>
      <input type="file" id="fileInput" accept="image/*" onchange={handleFileChange}>
      <button>Upload</button>
  </div>
</header>
