<script lang="ts" context="module">
  export interface PaginationMeta {
    lastPage: number;
    currentPage: number;
    totalPages: number;
    previousPage: number | null;
    nextPage: number | null;
  }
</script>

<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { onMount } from 'svelte';

  const dispatch = createEventDispatcher();

  export let meta: Readonly<PaginationMeta> = { lastPage: 0, currentPage: 0, totalPages: 0, previousPage: 0, nextPage: 0 };
  export let range: Readonly<number> = 5;
  export let align: Readonly<'center' | 'right' | 'left' | null> = null;
  
  let page = 1;
  let ready = false;

  onMount(() => {
    if (meta.currentPage) {
      page = meta.currentPage;
      ready = true;
    }
  });

  $: isReady = ready && meta.totalPages > 1;

  $: pages = () => {
    const step = Math.floor(range / 2);
    let first = page - step;
    if (first <= 0) {
      first = 1;
    }
    let last = first + range - 1;
    if (last > meta.lastPage) {
      last = meta.lastPage;
    }
    if (last === meta.lastPage) {
      const count = last - first + 1;
      first = first - (range - count);
      if (first <= 0) {
        first = 1;
      }
    }
    const items: number[] = [];
    for (let i = first; i <= last; i++) {
      items.push(i);
    }
    return items;
  };

  $: previousDisabled = () => {
    return meta.previousPage == null;
  };

  $: nextDisabled = () => {
    return meta.nextPage == null;
  };

  $: alignClass = () => {
    switch (align) {
      case 'center':
        return 'justify-content-center';
      case 'right':
        return 'justify-content-end';
      case 'left':
      default:
        return null;
    }
  };

  const goTo = (item: number) => {
    page = item;
    dispatch('change', page);
  };

  const goToPrevious = () => {
    let previous = page - 1;
    if (previous <= 0) {
      previous = 1;
    }
    goTo(previous);
  };

  const goToNext = () => {
    let next = page + 1;
    if (next > meta.lastPage) {
      next = meta.lastPage;
    }
    goTo(next);
  };

  const goToFirst = () => {
    goTo(1);
  };

  const goToLast = () => {
    goTo(meta.lastPage);
  };

  const isCurrent = (item: number) => {
    return item === page;
  };
</script>

{#if isReady}
  <nav aria-label="Page navigation">
    <ul class="pagination {alignClass()}">
      <li class="page-item {previousDisabled() ? 'disabled' : ''}">
        <a class="page-link" href="#" aria-label="First" on:click|preventDefault={goToFirst}>
          <slot name="first-arrow">&lt;&lt;</slot>
        </a>
      </li>
      <li class="page-item {previousDisabled() ? 'disabled' : ''}">
        <a class="page-link" href="#" aria-label="Previous" on:click|preventDefault={goToPrevious}>
          <slot name="left-arrow">⇦</slot>
        </a>
      </li>

      {#each pages() as item}
        <li class="page-item {isCurrent(item) ? 'active' : ''}">
          <a class="page-link" href="#" on:click|preventDefault={() => goTo(item)}>{item}</a>
        </li>
      {/each}

      <li class="page-item {nextDisabled() ? 'disabled' : ''}">
        <a class="page-link" href="#" aria-label="Next" on:click|preventDefault={goToNext}>
          <slot name="right-arrow">⇨</slot>
        </a>
      </li>
      <li class="page-item {nextDisabled() ? 'disabled' : ''}">
        <a class="page-link last-link" href="#" aria-label="Last" on:click|preventDefault={goToLast}>
          <slot name="last-arrow">&gt;&gt;</slot>
        </a>
      </li>
    </ul>
  </nav>
{/if}

<style>
  .page-item {
    min-width: 52px;
    text-align: center;
    user-select: none;
  }
</style>
