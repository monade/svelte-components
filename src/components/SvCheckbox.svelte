<script lang="ts">
  import { createEventDispatcher, onMount } from 'svelte';

  export let value: boolean;
  export let label: string | null = null;
  export let inputValue: string | null = null;
  export let disabled: boolean = false;

  onMount(() => {
    console.log(label);
  });

  let selected: boolean = value || false;
  let slot: HTMLLabelElement;
  const dispatch = createEventDispatcher();

  $: onValueChange(value);
  function onValueChange(value: boolean) {
    selected = value;
  }

  $: id = `checkbox-${randomValue()}`;
  $: checkboxValue = inputValue ? inputValue : slot?.textContent!.trim() || label;

  function randomValue() {
    return Math.random().toString(36).substring(2, 9);
  }

  function onChange() {
    console.log(label);
    selected = !selected;
    dispatch('input', { selected });
  }
</script>

<div class="custom-control custom-checkbox">
  <input
    type="checkbox"
    class="custom-control-input"
    {id}
    value={checkboxValue}
    on:change={onChange}
    checked={selected}
    {disabled}
  />
  {#if $$slots.default}
    <label bind:this={slot} class="custom-control-label" for={id}><slot /></label>
  {:else}
    <label class="custom-control-label" for={id}>{label}</label>
  {/if}
</div>

<style lang="scss">
  label {
    user-select: none;
  }
  .custom-checkbox {
    cursor: pointer;
    .custom-control-label {
      font-size: 0.6875rem;
      &:before,
      &:after {
        top: 0;
      }
    }
  }
</style>
