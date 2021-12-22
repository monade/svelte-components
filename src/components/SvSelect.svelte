<script context="module" lang="ts">
  import { onMount, createEventDispatcher, tick } from 'svelte';

  export interface VSelectOption {
    id: string;
    text: string;
    editable?: boolean;
    new?: boolean;
  }
</script>

<script lang="ts">
  export let value: string | null | undefined = null;
  export let placeholder: string = 'Select an option';
  export let options: VSelectOption[];
  export let required = false;
  export let addable = false;
  export let editable = false;
  export let expanded = false;
  export let autoSort = true;
  export let addableAlign = 'top';
  export let disabled = false;
  export let scrollable = true;
  export let loading = false;
  export let inputClass = '';

  const dispatch = createEventDispatcher();

  let dropdownVisible = false;

  let editing: VSelectOption | null = null;
  let selected: VSelectOption | null = null;
  let selectableOptions: VSelectOption[] = [];
  let adding = {
    enabled: false,
    text: ""
  };

  let inputRef: HTMLInputElement | null = null;
  let editInputRefs: HTMLInputElement[] = [];
  let dropdownRef: HTMLDivElement | null = null;

  $: updateSelected(value)
  $: {
    selectableOptions = [...options];
    updateSelected(value)
  }

  $: rootClasses = parseClasses({ 'v-select--open': showDropdown, 'v-select--expanded': expanded });
  $: optionsClasses = parseClasses({ 'flex-column-reverse': addableAlign === 'bottom' });

  $: showDropdown = dropdownVisible || expanded;

  $: label = (() => {
    if (selected == null) {
      if (loading) {
        return "";
      }
      return placeholder || "";
    }
    return selected.text;
  })()

  onMount(() => {
    updateSelected(value);
  })

  function parseClasses(classes: Record<string, boolean>) {
    return Object.keys(classes).filter(c => classes[c]).join(' ');
  }

  function updateSelected(value: string | null | undefined) {
    if (value == null) {
      selected = null;
    } else {
      selected =
        selectableOptions.find(element => element.id === value) ||
        null;
    }
  }

  function toggleDropdown() {
    dropdownVisible = !dropdownVisible;
  }

  function closeDropdown() {
    dropdownVisible = false;
  }

  function select(option: VSelectOption) {
    selected = option;
    closeDropdown();
    dispatch("input", selected.id);
  }

  function add() {
    if (adding.text === "") {
      return;
    }
    const option: VSelectOption = {
      id: `new_${selectableOptions.length + 1}`,
      text: adding.text,
      new: true
    };
    if (editable) {
      option.editable = true;
    }
    selectableOptions.push(option);
    sortOptions();
    selected = option;
    stopAdding();
    closeDropdown();
    dispatch("added", option);
  }

  async function startAdding() {
    adding.enabled = true;
    await tick();
    inputRef?.focus();
  }

  function stopAdding() {
    adding.enabled = false;
    adding.text = "";
  }

  function onAddingKeyUp(event: KeyboardEvent) {
    if (event.key === 'Enter') {
      add();
    } else if (event.key === 'Escape') {
      stopAdding();
    }
  }

  function onEditingKeyUp(event: KeyboardEvent) {
    if (event.key === 'Enter') {
      edit();
    } else if (event.key === 'Escape') {
      stopEditing();
    }
  }


  function edit() {
    closeDropdown();
    dispatch("edited", editing);
    stopEditing();
  }

  async function startEditing(option: VSelectOption) {
    editing = option;
    await tick();
    return editInputRefs[0]?.focus();
  }

  function stopEditing() {
    editing = null;
  }

  function sortOptions() {
    if (!autoSort) {
      return;
    }
    selectableOptions.sort((a: VSelectOption, b: VSelectOption) => {
      if (a.text.toLowerCase() < b.text.toLowerCase()) {
        return -1;
      } else if (a.text.toLowerCase() > b.text.toLowerCase()) {
        return 1;
      }
      return 0;
    });
  }

  function isSelected(option: VSelectOption) {
    if (selected == null) {
      return false;
    }
    return selected.id === option.id;
  }
</script>

<span
  class="v-select {rootClasses}"
  v-click-outside="closeDropdown"
>
  <div class="flex h-full relative">
    <div class="flex absolute h-full">
      {#if loading}
        <slot name="loading">
          loading...
          <!-- <v-icon spin={true}>spinner</v-icon> -->
        </slot>
      {/if}
    </div>
    {#if !expanded}
      <input
        type="text"
        class="form-control my-auto {inputClass}"
        readonly
        value={label}
        on:click|preventDefault={toggleDropdown}
        disabled={disabled}
      />
    {/if}
  </div>
  <input class="v-select__value" required={required} bind:value={value} />

  <div
    class="v-select__dropdown {inputClass} { showDropdown ? '' : 'hidden' }"
    v-show="showDropdown"
    bind:this={dropdownRef}
  >
    <div
      class="v-select__options flex flex-col {optionsClasses}"
    >
      {#if addable}
        <div
          class="v-select__option v-select__option--add {adding ? 'v-select__option--adding' : '' }"
          on:click|stopPropagation={startAdding}
        >
          {#if adding.enabled}
            <input
              type="text"
              class="form-control"
              bind:value={adding.text}
              bind:this={inputRef}
              on:keyup={onAddingKeyUp}
              placeholder="Inserisci descrizione..."
            />

            <button class="btn btn-link text-primary" on:click|stopPropagation={add}>
              <slot name="add-confirm"></slot>
            </button>
          {:else}
            <slot name="add">Add</slot>
          {/if}
        </div>
      {/if}

      <div
        class={scrollable ? 'v-select__options--scrollable' : 'v-select__options'}
      >
        {#each selectableOptions as option, index (option.id)}
          <div
            class="v-select__option {parseClasses({ 'v-select__option--selected': isSelected(option) })} {inputClass}"
            on:click|stopPropagation={() => select(option)}
          >
            {#if editing && editing.id == option.id}
              <input
                type="text"
                class=" v-select__option--editing"
                bind:value={editing.text}
                bind:this={editInputRefs[index]}
                on:keyup={onEditingKeyUp}
                placeholder="Inserisci descrizione..."
              />
            {:else}
              { option.text }
              {#if option.editable}
                <button
                  class="btn btn-link text-primary p-0 float-right"
                  on:click|stopPropagation={() => startEditing(option)}
                >
                  <slot name="edit">Edit</slot>
                </button>
              {/if}
            {/if}
          </div>
        {/each}
      </div>
    </div>
  </div>
</span>

<style lang="scss">
$primary: --color-primary;
$zindex-dropdown: 100;
$input-border-radius: 10px;
$gray-100: #f0f0f0;
$white: #fff;
$border-color: #ff0000;

.form-control {
  @apply p-3.5 rounded-md w-full;
}

$v-select-min-width: 200px !default;
$v-select-scrollable-max-heigh: 155px !default;
%v-select-input {
  background-color: transparent;
  border: 0;
  padding: 0;
  height: auto;
  border-radius: 0;
  caret-color: $primary;
  &::placeholder {
    padding-left: 6px;
  }
  &:focus {
    background-color: transparent;
  }
}
.v-select {
  display: block;
  position: relative;
  min-width: $v-select-min-width;
  &__value {
    width: 0;
    height: 1px;
    display: block;
    border: 0;
    margin: 0;
    padding: 0;
    opacity: 0;
  }
  &:after {
    position: absolute;
    content: " ";
    right: 1rem;
    top: 0;
    width: 9px;
    height: 100%;
    margin-top: 2px; // Fix svg render issue
    // @include background-image("../assets/icons/chevron-down.svg", contain);
  }
  &--open {
    &:after {
      transform: rotate(180deg);
      -ms-transform: rotate(180deg);
      -webkit-transform: rotate(180deg);
      margin-top: 0; // Fix svg render issue
    }
  }
  &--expanded {
    .v-select__dropdown {
      position: initial;
    }
    &:after {
      display: none;
    }
  }
  input {
    padding-right: 2rem;
    cursor: pointer;
    user-select: none;
  }
  &__dropdown {
    position: absolute;
    z-index: $zindex-dropdown;
    margin-top: 5px;
    padding: 0;
    border-radius: $input-border-radius;
    border: 1px solid $gray-100;
    box-shadow: 0px 3px 6px $gray-100;
    background-color: $white;
    min-width: $v-select-min-width;
    width: 100%;
  }
  &__options {
    width: 100%;
    &--scrollable {
      width: 100%;
      max-height: $v-select-scrollable-max-heigh;
      overflow-y: auto;
    }
  }
  &__option {
    cursor: pointer;
    padding: 10px;
    border-bottom: 1px solid $border-color;
    &:last-child {
      border-bottom: 0;
    }
    &:hover {
      background-color: $gray-100;
    }
    &--add {
      color: $primary;
    }
    &--editing {
      @extend %v-select-input;
    }
    &--adding {
      display: flex;
      input {
        @extend %v-select-input;
      }
    }
    &--selected {
      background-color: $gray-100;
    }
  }
}
</style>
