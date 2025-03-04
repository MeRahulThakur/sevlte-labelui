<script lang="ts">
  import Header from './UI/Header.svelte';
  import ImageContainer from './components/ImageContainer.svelte';
  import DataTable from './components/DataTable.svelte';
  import Footer from './UI/Footer.svelte';

  const state = $state({
    imageSrc: null as string | null,
    selectedFile: null as File | null,
    labelDataSet: [] as { x0: number, y0: number, x1: number, y1: number }[],
    selectedLabelBox: null as { x0: number, y0: number, x1: number, y1: number } | null 
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

  function getRandomDataSet(count: number): { x0: number, y0: number, x1: number, y1: number }[] {
    return Array.from({ length: count }, () => {
      const x0 = Math.floor(Math.random() * 400);
      const y0 = Math.floor(Math.random() * 400);
      const x1 = x0 + Math.floor(Math.random() * 100) + 20;
      const y1 = y0 + Math.floor(Math.random() * 100) + 20;
      return { x0, y0, x1, y1 };
    });
  }

  async function uploadImage() {
    if (!state.selectedFile) {
      alert("Please select an image first.");
      return;
    }

    const formData = new FormData();
    formData.append("file", state.selectedFile);

    try {
      /*const response = await fetch("https://your-api-endpoint.com/upload", {
        method: "POST",
        body: formData,
      });

      if (!response.ok) {
        throw new Error("Upload failed");
      }

      const result = await response.json();
      state.imageSrc = result.imageUrl; // Assuming API returns an image URL*/
      state.imageSrc = "https://www.howardsgroup.co.uk/assets/media/evolutionofthecar_1.jpg"
      // Simulating received rectangle data from response
      state.labelDataSet = getRandomDataSet(20);
    } catch (error) {
      console.error("Error uploading image:", error);
      alert("Upload failed. Please try again.");
    }
  }

  function handleLabelBoxClick(box: { x0: number, y0: number, x1: number, y1: number }){
    console.log('selectedLabelBox',box)
    state.selectedLabelBox = box
  }

  function handleRowSelect(box: { x0: number, y0: number, x1: number, y1: number }) {
    console.log('handleRowSelect',box)
    state.selectedLabelBox = box;
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

<Header isSticky onUpload={previewImage} onUploadClick={uploadImage} resetLabelDataSet={() => (state.labelDataSet = [])} />

<main class="main">
  <ImageContainer
   imageSrc={state.imageSrc}
   labelDataSet={state.labelDataSet.length>0?state.labelDataSet:null}
   onLabelBoxClick={handleLabelBoxClick}
   selectedLabelBox={state.selectedLabelBox}
   --dynamic-height="calc(100dvh - 4.5rem - 4.5rem)"
  />
  <!-- {--dynamic-height="calc(100dvh - 4.5rem - 4.5rem)" } -->
  <DataTable
   labelDataSet={state.labelDataSet}
   selectedLabelBox={state.selectedLabelBox}
   onRowSelect={handleRowSelect}
   --dynamic-height="calc(100dvh - 4.5rem - 4.5rem)"
  />
</main>

<Footer isSticky={true} />
