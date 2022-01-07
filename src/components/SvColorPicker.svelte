<script lang="ts" context="module">
  declare namespace svelte.JSX {
    interface HTMLAttributes<T> {
      onclick_outside: () => void;
    }
  }
</script>

<script lang="ts">
  import { createEventDispatcher, onMount } from 'svelte';
  import { clickOutside } from '../directives/ClickOutside';
  import { chunkArrayInGroups } from '../utils/chunkArrayInGroups';
  import SvIcon from './SvIcon.svelte';

  export let colors: string[] = [];
  export let color: Readonly<string> = '#ff0000';
  export let columns: Readonly<number> = 1;

  const dispatch = createEventDispatcher();

  let open = false;
  let currentColor = '';

  onMount(() => {
    currentColor = color;
  });

  const closeDropdown = () => {
    open = false;
  };

  const onColorClick = (color: string) => {
    currentColor = color;
    dispatch('change', color);
  };

  $: colorChunks = chunkArrayInGroups(colors, columns);

  const handleClickOutside = () => {
    closeDropdown();
  };
</script>

<div use:clickOutside on:clickOutside={handleClickOutside} class="dropdown" v-click-outside="closeDropdown">
  <div class="d-flex h-100">
    <div class="my-auto clickable ml-3 h-100" on:click={() => (open = !open)}>
      <div class="color-indicator" style="background: {currentColor}" />
    </div>
  </div>
  <div class="dropdown-menu dropdown-menu-left px-4 {open ? 'show' : ''}" aria-labelledby="navbarDropdown">
    {#each colorChunks as chunk}
      <div class="row mb-2">
        {#each chunk as chunkColor}
          <div class="col p-0">
            <div
              class="color-picker-indicator clickable d-flex m-1 p-1"
              style="background: {chunkColor}"
              on:click={() => onColorClick(chunkColor)}
            >
              {#if currentColor == chunkColor}
                <div class="m-auto icon icon-white">
                  <SvIcon size="sm">check</SvIcon>
                </div>
              {/if}
            </div>
          </div>
        {/each}
      </div>
    {/each}
  </div>
</div>

<style>
  .color-indicator {
    width: 13px;
    height: 13px;
    border-radius: 100%;
  }
  .color-picker-indicator {
    width: 23px;
    height: 23px;
    border-radius: 100%;
  }
  .icon {
    height: 23px;
  }
</style>
