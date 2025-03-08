<script lang="ts">
  import { onMount } from "svelte";
  import {
    type GridApi,
    type GridOptions,
    type ColDef,
    ModuleRegistry,
    type GridReadyEvent,
    type RowClickedEvent,
    ClientSideRowModelModule,
    AllCommunityModule,
    createGrid,
    type RowStyle,
  } from "ag-grid-community";
  import "ag-grid-community/styles/ag-grid.css";
  import "ag-grid-community/styles/ag-theme-alpine.css";

  interface IRow { 
    mappingKey: string;
    column1: string; 
    column2: string
  }

  // Register the required module
  ModuleRegistry.registerModules([AllCommunityModule]);

  interface IDataTableProps {
    recordsDataSet?: IRow[] | null;
    selectedMappingKey?: string | null;
    onRowSelect?: (mappingKey: string) => void;
  }

  let { recordsDataSet = [], selectedMappingKey = null, onRowSelect }: IDataTableProps = $props();
  let gridApi: GridApi | null = null;
  let gridElement: HTMLDivElement | null = null;

  let gridOptions: GridOptions<IRow> = {
    defaultColDef: {
      sortable: true,
      filter: true,
      resizable: true,
      floatingFilter: true, // Enables column searching
      flex: 1,
    },
    columnDefs: [
      { headerName: "Column 1", field: "column1", flex: 1 },
      { headerName: "Column 2", field: "column2", flex: 1 },
    ],
    rowSelection: "single",
    onRowClicked: (event: RowClickedEvent) => handleRowClick(event),
  };

  function handleRowClick(event: RowClickedEvent) {
    selectedMappingKey = event.data.mappingKey;
    if (onRowSelect) {
      onRowSelect(event.data.mappingKey);
    }
    gridApi?.redrawRows(); // Ensures rowStyle updates when selection changes
  }

  onMount(() => {
    if (gridElement) {
      gridApi = createGrid(gridElement, gridOptions);
      console.log("AG Grid Initialized:", gridApi);
      if (recordsDataSet && recordsDataSet.length > 0) {
        gridApi.setGridOption("rowData", [...recordsDataSet]);
      } else {
        gridApi.setGridOption("rowData", []); // Ensures empty state
      }
    }
  });

  // Auto-update row data when recordsDataSet changes
  $effect(() => {
    if (gridApi) {
      gridApi.setGridOption("rowData", recordsDataSet ? [...recordsDataSet] : []);
    }
  });

  // Scroll to selected row and apply highlighting
  $effect(() => {
    console.log("Selected Row Key:", selectedMappingKey);
    if (gridApi && selectedMappingKey) {
      gridApi.forEachNode((node) => {
        if (node.data.mappingKey === selectedMappingKey) {
          node.setSelected(true);
          gridApi?.ensureNodeVisible(node, "middle");
        }
      });
      gridApi.redrawRows(); // Ensures rowStyle updates
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

</style>

<div class="table-container">
  <div bind:this={gridElement} class="ag-theme-alpine {recordsDataSet?.length === 0 ? 'hide-headers' : ''}" style="width: 100%; height: 100%;"></div>
</div>
