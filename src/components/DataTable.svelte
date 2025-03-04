<script lang="ts">
  interface IDataTableProps {
    labelDataSet?: { x0: number, y0: number, x1: number, y1: number }[] | null;
    selectedLabelBox?: { x0: number, y0: number, x1: number, y1: number } | null;
  }

  let { labelDataSet = null, selectedLabelBox = null }: IDataTableProps = $props();

  // Auto-scroll to the selected row when selectedRect changes
  $effect(() => {
    if (!selectedLabelBox) return;
    const row = document.querySelector(
      `[data-x0="${selectedLabelBox.x0}"][data-y0="${selectedLabelBox.y0}"][data-x1="${selectedLabelBox.x1}"][data-y1="${selectedLabelBox.y1}"]`
    );

    if (row) {
      row.scrollIntoView({ behavior: "smooth", block: "center" });
    }
  });

</script>

<style>
  .table-container {
    overflow-y: auto;
    background: lightgray;
    display: flex;
    align-items: flex-start;
    justify-content: center;
    padding: 0 3px;
    border: 1px solid #ccc;
  }

  .table-container {
    overflow-x: hidden;
    height: var(--dynamic-height, auto);
  }

  table {
    width: 100%;
    border-collapse: collapse;
    background-color: light-dark(rgb(255, 255, 255),#555);
  }

  th, td {
    border: 1px solid #ccc;
    padding: 8px;
    text-align: center;
  }

  th {
    background-color: light-dark(rgb(255, 255, 255),#242424);
  }

  thead {
    position: sticky;
    top: 0;
  }

  .selected {
    background-color: #e3aa47;
    font-weight: bold;
  }
</style>

<div class="table-container">
  {#if labelDataSet && labelDataSet.length > 0}
  <table>
    <thead>
      <tr>
        <th>(x0, y0)</th>
        <th>(x1, y1)</th>
      </tr>
    </thead>
    <tbody>
      {#each labelDataSet as { x0, y0, x1, y1 }}
        <tr 
          data-x0={x0} data-y0={y0} data-x1={x1} data-y1={y1}
          class:selected={selectedLabelBox && selectedLabelBox.x0 === x0 && selectedLabelBox.y0 === y0 && selectedLabelBox.x1 === x1 && selectedLabelBox.y1 === y1}
        >
          <td>({x0}, {y0})</td>
          <td>({x1}, {y1})</td>
        </tr>
      {/each}
    </tbody>
  </table>
  {:else}
    <p>No data available</p>
  {/if}
</div>
