<script lang="ts">
  export let value: boolean;
  export let disabled: boolean = false;

  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();
  let checked: boolean = value;

  // TODO: find a better way
  $: toggleName = `toggle_${Math.trunc(Math.random() * 10000)}`;

  function activate() {
    if (!disabled) {
      checked = !checked;
      dispatch('change', checked);
    }
  }
</script>

<div
  on:click={activate}
  class="toggle {checked ? 'checked' : 'unchecked'}"
  class:disabled
  class:clickable={!disabled}
>
  <input
    on:click={activate}
    id="toggle-on-{toggleName}"
    class="toggle"
    name="toggle"
    value="false"
    type="radio"
  />
  <label
    for="toggle-on-{toggleName}"
    class="label {checked ? 'checked' : 'unchecked'}"
    class:clickable={!disabled}
  />
</div>

<style lang="scss">
  $primary: #6cfc65 !default;
  $white: white !default;
  $text-muted: #1a1a1a !default;

  .toggle {
    width: 50px;
    height: 22px;
    border-radius: 100px;
    position: relative;
    transition: background-color 0.4s;
    display: inline-block;

    &.unchecked {
      background-color: #b5b5b5;
    }

    &.checked {
      background-color: $primary;
    }

    &.disabled {
      background-color: $text-muted;
    }

    input {
      display: none;
    }

    label {
      display: block;
      width: 18px;
      height: 18px;
      background-color: $white;
      z-index: 100;
      border-radius: 100%;
      position: absolute;
      top: 50%;
      transform: translate(0, -50%);
      transition: all 0.35s ease-in-out;

      &.unchecked {
        margin-left: 2px;
      }

      &.checked {
        margin-left: calc(100% - 2px);
        transform: translate(-100%, -50%);
      }
    }
  }

  .clickable {
    cursor: pointer;
  }
</style>
