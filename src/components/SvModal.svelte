<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { tick } from 'svelte';
  import { fade } from 'svelte/transition';

  export let contentClasses = '';
  export let headerClasses = '';
  export let bodyClasses = '';
  export let footerClasses = '';
  export let hidesOnClickOut = true;
  export let size = 'md';

  export const show = async () => {
    visible = true;
    dispatch('shown');
    await tick();
    hideable = true;
    registerKeyDownEvents();
  };

  export const hide = () => {
    visible = false;
    hideable = false;
    dispatch('hidden');
    unregisterKeyDownEvents();
  };

  const dispatch = createEventDispatcher();

  let visible = false;
  let hideable = false;

  function eventListener(e: KeyboardEvent) {
    if (e.keyCode === 27 && hidesOnClickOut) {
      hide();
    }
  }

  function registerKeyDownEvents() {
    window.addEventListener('keydown', eventListener);
  }

  function unregisterKeyDownEvents() {
    window.removeEventListener('keydown', eventListener);
  }

  function hasHeader() {
    return $$slots.header;
  }

  function hasFooter() {
    return $$slots.footer;
  }

  function onkeypress() {
    if (visible) {
      hide();
    }
  }
</script>

{#if visible}
  <div class="modal-wrapper" class:active={visible}>
    <div transition:fade class="modal modal-{size}" tabindex="-1" role="dialog" aria-hidden="true">
      {#if hasHeader()}
        <div class="modal__header {headerClasses}">
          <slot name="header" />
        </div>
      {/if}

      <div class="modal__body {bodyClasses}">
        <slot />
      </div>

      {#if hasFooter()}
        <div class="modal__footer {footerClasses}">
          <slot name="footer" />
        </div>
      {/if}
    </div>
    <div
      transition:fade
      class="modal-backdrop"
      on:click|preventDefault={hidesOnClickOut ? hide : null}
    />
  </div>
{/if}

<style lang="scss" scoped>
  $bg-color: #eee;
  $radius: 1rem;

  .modal-wrapper {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    justify-content: center;
    align-items: center;
    z-index: 100;

    &.active {
      display: flex;
    }
  }

  .modal-backdrop {
    position: absolute;
    left: 0;
    right: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(#000, 0.7);
    backdrop-filter: blur(3px);
  }

  .modal {
    z-index: 200;
    width: 40rem;
    background-color: $bg-color;
    border-radius: $radius;

    &__body {
      border-top: 1px solid #bababa;
      border-bottom: 1px solid #bababa;
    }
  }
</style>
