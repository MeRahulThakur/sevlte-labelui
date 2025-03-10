<script lang="ts">
  import Header from './UI/Header.svelte';
  import ImageContainer from './components/ImageContainer.svelte';
  import DataTable from './components/DataTable.svelte';
  import Footer from './UI/Footer.svelte';
  import DataTableAg from './components/DataTableAG.svelte';

  const predefinedKeys = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L"]; // Example predefined set of keys

  const state = $state({
    imageSrc: null as string | null,
    selectedFile: null as File | null,
    labelDataSet: [] as { mappingKey: string, x0: number, y0: number, x1: number, y1: number }[],
    recordsDataSet: [] as { mappingKey: string, column1: string, column2: string }[],
    selectedLabelBoxes: [] as { mappingKey: string, x0: number, y0: number, x1: number, y1: number }[] | null, // Track the selected record
    selectedMappingKey: null as string | null,
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

  function getRandomDataSet(count: number) {
    return Array.from({ length: count }, (_, i) => {
      const mappingKey = predefinedKeys[Math.floor(Math.random() * predefinedKeys.length)]; // Assign random key
      const x0 = Math.floor(Math.random() * 400);
      const y0 = Math.floor(Math.random() * 400);
      const x1 = x0 + Math.floor(Math.random() * 100) + 20;
      const y1 = y0 + Math.floor(Math.random() * 100) + 20;
      return { mappingKey, x0, y0, x1, y1 };
    });
  }

  function generateRecordsData(labelData: { mappingKey: string }[]) {
    const uniqueRecords = new Map();
    labelData.forEach(({ mappingKey }) => {
      if (!uniqueRecords.has(mappingKey)) {
        uniqueRecords.set(mappingKey, {
          mappingKey,
          column1: `Data for ${mappingKey}`,
          column2: `More info ${mappingKey}`
        });
      }
    });
    return Array.from(uniqueRecords.values());
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
      //state.imageSrc = "https://www.howardsgroup.co.uk/assets/media/evolutionofthecar_1.jpg"
      // Simulating received rectangle data from response
      state.labelDataSet = getRandomDataSet(20);
      state.recordsDataSet = generateRecordsData(state.labelDataSet);
      
    } catch (error) {
      console.error("Error uploading image:", error);
      alert("Upload failed. Please try again.");
    }
  }

  function handleLabelBoxClick(row:{ mappingKey: string, x0: number, y0: number, x1: number, y1: number }) {
    console.log('Selected Label Box:', row);
    state.selectedMappingKey = row.mappingKey; // Highlight all boxes with this key
  }

  function handleRowSelect(mappingKey: string) {
    console.log('Selected Row mapping key:', mappingKey);
    state.selectedLabelBoxes = state.labelDataSet.filter(labelData => labelData.mappingKey === mappingKey); // Select corresponding bounding boxes
    console.log(state.selectedLabelBoxes)
  }
</script>

<style>
  .main {
    display: grid;
    grid-template-columns: 3fr 1fr;
    gap: 10px;
    padding: 10px;
    flex: 1;
    margin-top: 4.5rem;
  }

  @media (max-width: 768px) {
    .main {
      grid-template-columns: 1fr;
    }
  }
</style>

<Header 
  isSticky 
  onUpload={previewImage} 
  onUploadClick={uploadImage} 
  resetLabelDataSet={() => {
    state.labelDataSet = [];
    state.recordsDataSet = [];
    state.selectedMappingKey = null;
  }} 
/>

<main class="main">
  <ImageContainer
   imageSrc={state.imageSrc}
   labelDataSet={state.labelDataSet.length > 0 ? state.labelDataSet : null}
   onLabelBoxClick={handleLabelBoxClick}
   selectLabelBoxes={state.selectedLabelBoxes}
   --dynamic-height="calc(100dvh - 4.5rem - 4.5rem)"
  />
  <!-- {--dynamic-height="calc(100dvh - 4.5rem - 4.5rem)" } -->
  <!-- <DataTable
   recordsDataSet={state.recordsDataSet} 
   selectedMappingKey={state.selectedMappingKey}
   onRowSelect={handleRowSelect}
   --dynamic-height="calc(100dvh - 4.5rem - 4.5rem)"
  /> -->

  <DataTableAg
    recordsDataSet={state.recordsDataSet} 
    selectedMappingKey={state.selectedMappingKey}
    onRowSelect={handleRowSelect}
    --dynamic-height="calc(100dvh - 4.5rem - 4.5rem)"
  />
</main>

<Footer isSticky={true} />
