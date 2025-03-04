<script lang="ts">
    import Header from './UI/Header.svelte';
    import ImageContainer from './components/ImageContainer.svelte';
    import DataTable from './components/DataTable.svelte';

    const state = $state({
        imageSrc: null as string | null
    });

    function previewImage(event: Event) {
        const target = event.target as HTMLInputElement;
        const file = target.files?.[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = (e) => state.imageSrc = e.target?.result as string;
            reader.readAsDataURL(file);
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
    }

    @media (max-width: 768px) {
        .main {
            grid-template-columns: 1fr;
        }
    }
</style>

<Header onUpload={previewImage} />

<main class="main">
    <ImageContainer imageSrc={state.imageSrc} />
    <DataTable />
</main>
