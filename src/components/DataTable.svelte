<script lang="ts">
  interface IDataTableProps {
    recordsDataSet?: { mappingKey: string, column1: string, column2: string }[] | null;
    selectedMappingKey?: string | null;
    onRowSelect?: ((mappingKey: string) => void) | null;
  }

  let { recordsDataSet = null, selectedMappingKey = null, onRowSelect}: IDataTableProps = $props();
  let selectedRowKey = $state<string | null>(selectedMappingKey);

  function handleRowClick(mappingKey: string) {
    selectedRowKey = mappingKey;
    onRowSelect?.(mappingKey);
  }

  // Auto-scroll to selected row
  $effect(() => {
    if (!selectedRowKey) return;
    const row = document.querySelector(`[data-mappingKey="${selectedRowKey}"]`);
    row?.scrollIntoView({ behavior: "smooth", block: "center" });
  });

  $effect(() => {
    selectedRowKey = selectedMappingKey;
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
    background-color: light-dark(rgb(255, 255, 255), #555);
  }

  th, td {
    border: 1px solid #ccc;
    padding: 8px;
    text-align: center;
  }

  th {
    background-color: light-dark(rgb(255, 255, 255), #242424);
  }

  thead {
    position: sticky;
    top: 0;
  }

  .no-data-message {
    align-self: center;
  }

  .selected {
    background-color: #e3aa47;
    font-weight: bold;
  }
</style>

<div class="table-container">
  {#if recordsDataSet && recordsDataSet.length > 0}
    <table>
      <thead>
        <tr>
          <th>Column 1</th>
          <th>Column 2</th>
        </tr>
      </thead>
      <tbody>
        {#each recordsDataSet as { mappingKey, column1, column2 }}
          <tr 
            data-mappingKey={mappingKey}
            class:selected={selectedRowKey === mappingKey}
            onclick={() => handleRowClick(mappingKey)}
          >
            <td>{column1}</td>
            <td>{column2}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  {:else}
    <p class="no-data-message">No data available</p>
  {/if}
</div>
