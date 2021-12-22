<script context="module" lang="ts">
  import type { SvelteComponent } from 'svelte';

  export interface ColumnConfig {
    sortable?: boolean;
    sort?: string;
    name: string;
    class?: string;
    default?: string;
    label: string;
    component?: SvelteComponent;
  }

  export interface ColumnData {
    id: number | string;
    [key: string]: any;
  }
</script>

<script lang="ts">
  import { onMount, createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();
  export let columns: ColumnConfig[] = [];
  export let tableClasses = 'table table-sm table-hover';
  export let values: ColumnData[] = [];
  export let sort = '';
  export let sortable = false;
  export let loading = false;

  let sortingMap: Map<string, string> = new Map();

  $: columnsTotal = $$slots.actions ? columns.length + 1 : columns.length;
  $: {
    parseSort(sort);
  }

  onMount(() => {
    parseSort(sort);
  })

  function select(value: ColumnData) {
    dispatch('selected', value);
  }

  function parseSort(sort: string) {
    sortingMap = new Map();
    sort.split(',').map(item => {
      const order = item[0] === '-' ? '-' : '';
      const key = order ? item.substr(1) : item;
      sortingMap.set(key, order)
    })
    console.log('parseSlot', sort, sortingMap)
  }

  function sortBy(column: ColumnConfig) {
    if (!sortable) {
      return;
    }
    const name = column.sort || column.name;
    let order = sortingMap.get(name);
    if (typeof order === 'string') {
      order = order === '-' ? '' : '-';
    } else {
      order = '';
    }
    const sort = `${order}${name}`;
    dispatch('sorted', sort);
  }

  function isSortedBy(column: ColumnConfig) {
    const name = column.sort || column.name;
    return !!sortingMap.get(name);
  }

  function isSortedByDesc(column: ColumnConfig) {
    const name = column.sort || column.name;
    return sortingMap.get(name) === '-';
  }

  function getColumnClass(column: ColumnConfig) {
    const columnClass: string[] = [];
    if (isSortedBy(column)) {
      columnClass.push('col-order-by');
    }
    columnClass.push(`col-data__${column.name}`);
    if (column.class) {
      columnClass.push(column.class);
    }
    return columnClass.join(' ');
  }

  function getValue(column: ColumnConfig, value: ColumnData) {
    if (column.name.indexOf('.') !== -1) {
      const methodChain = column.name.split('.');
      let result = value;
      for (const method of methodChain) {
        result = result ? result[method] : null;
      }
      return result || column.default;
    }
    return value[column.name] || column.default;
  }
</script>

<div class="data-table-container">
  <div class="table-responsive overflow-x-auto max-w-100">
    <table class="bg-red-100 {tableClasses}">
      <thead>
        <tr>
          {#each columns as column (column.name)}
            <th class={getColumnClass(column)}>
              {#if sortable && column.sortable !== false }
                <a href="#" on:click|preventDefault={() => sortBy(column)}>
                  { column.label }
                  {#if isSortedBy(column) }
                    {#if isSortedByDesc(column)}
                      <slot name="sort-icon-down">⇩</slot>
                    {:else}
                      <slot name="sort-icon-up">⇧</slot>
                    {/if}
                  {/if}
                </a>
              {:else}
                { column.label }
              {/if}
            </th>
          {/each}
          {#if $$slots.actions }
            <th>
              <slot name="actions-label">Actions</slot>
            </th>
          {/if}
        </tr>
      </thead>
      <tbody>
        {#if values.length }
          {#each values as value (value.id)}
            <tr class="border-red-500 border-1">
              {#each columns as column (column.name)}
                <td class={getColumnClass(column)} on:click={() => select(value)}>
                  {#if column.component }
                    <svelte:component this={column.component} value={value} column={column} />
                  {:else}
                    <slot name="cell" value={getValue(column, value)} column={column}>
                      { getValue(column, value) }
                    </slot>
                  {/if}
                </td>
              {/each}
              {#if $$slots.actions }
                <td class="col-actions">
                  <slot name="actions" value={value}></slot>
                </td>
              {/if}
            </tr>
          {/each}
        {:else}
          <tr>
            <td colspan={columnsTotal} class="col-no-results">
              {#if !loading}
                <span>
                  <slot name="no-results-label">No result</slot>
                </span>
              {/if}
            </td>
          </tr>
        {/if}
      </tbody>
    </table>
    {#if loading}
      <div class="loading-label">
        <slot name="loading">Loading...</slot>
      </div>
    {/if}
  </div>
  <slot></slot>
</div>

<style>
  .table-responsive {
    min-height: 320px;
  }
  .col-no-results {
    height: 270px;
    vertical-align: middle;
    border-bottom: 0;
  }
</style>
