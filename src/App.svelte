<script lang="ts">
  import Header from './UI/Header.svelte';
  import ImageContainer from './components/ImageContainer.svelte';
  import DataTable from './components/DataTable.svelte';
  import Footer from './UI/Footer.svelte';

  const state = $state({
    imageSrc: null as string | null,
    selectedFile: null as File | null
  });

  function previewImage(event: Event) {
    const target = event.target as HTMLInputElement;
    const file = target.files?.[0];
    if (file) {
      state.selectedFile = file;
      const reader = new FileReader();
      reader.onload = (e) => state.imageSrc = e.target?.result as string;
      reader.readAsDataURL(file);
    }
  }

  async function uploadImage() {
    if (!state.selectedFile) {
      alert("Please select an image first.");
      return;
    }

    const formData = new FormData();
    formData.append("file", state.selectedFile);

    try {
      const response = await fetch("https://your-api-endpoint.com/upload", {
        method: "POST",
        body: formData,
      });

      if (!response.ok) {
        throw new Error("Upload failed");
      }

      const result = await response.json();
      state.imageSrc = result.imageUrl; // Assuming API returns an image URL
    } catch (error) {
      console.error("Error uploading image:", error);
      alert("Upload failed. Please try again.");
    }
  }
</script>

<style>
  .main {
    display: grid;
    grid-template-columns: 3fr 1fr;
    gap: 10px;
    padding: 10px;
    flex: 1;
    margin-top: 5rem;
  }

  @media (max-width: 768px) {
    .main {
      grid-template-columns: 1fr;
    }
  }
</style>

<Header isSticky onUpload={previewImage} onUploadClick={uploadImage} />

<main class="main">
  <ImageContainer imageSrc={state.imageSrc} />
  <DataTable />
</main>

<Footer isSticky={true} />
