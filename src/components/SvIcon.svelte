<script lang="ts">
  import { afterUpdate, onMount } from 'svelte';

  export let spin: boolean = false;
  export let size: 'sm' | 'md' | 'lg' | 'xl' = 'md';

  let icon: HTMLSpanElement;
  let hidden: boolean = true;

  onMount(async () => {
    icon.innerHTML = await getSvg();
    hidden = false;
  });

  function getIcon() {
    if (icon) {
      return icon.textContent?.trim();
    }
    return null;
  }

  async function getSvg() {
    if (getIcon()) {
      const svg = (await import(`/static/${getIcon()}.svg?src`)).default;
      return svg;
    }
    return null;
  }
</script>

<span bind:this={icon} class="icon {size}" class:spin class:hidden>
  <slot />
</span>

<style lang="scss">
  .hidden {
    opacity: 0;
  }
  :global {
    .icon {
      color: inherit;
      display: inline-block;

      &.sm {
        height: 1.2rem;
        width: 1.2rem;
      }
      &.md {
        height: 2rem;
        width: 2rem;
      }
      &.lg {
        height: 2.7rem;
        width: 2.7rem;
      }
      &.xl {
        height: 3.2rem;
        width: 3.2rem;
      }
      > svg {
        height: 100%;
        width: 100%;
        font-size: inherit;
        * {
          opacity: 1;
        }
        path,
        rect {
          fill: currentColor !important;
          stroke: none;
        }
        line {
          stroke: currentColor !important;
        }
      }
      &.spin {
        > svg {
          animation-name: spin;
          animation-duration: 750ms;
          animation-iteration-count: infinite;
          animation-timing-function: linear;
        }
      }
    }
    @-ms-keyframes spin {
      from {
        -ms-transform: rotate(0deg);
      }
      to {
        -ms-transform: rotate(360deg);
      }
    }
    @-moz-keyframes spin {
      from {
        -moz-transform: rotate(0deg);
      }
      to {
        -moz-transform: rotate(360deg);
      }
    }
    @-webkit-keyframes spin {
      from {
        -webkit-transform: rotate(0deg);
      }
      to {
        -webkit-transform: rotate(360deg);
      }
    }
    @keyframes spin {
      from {
        transform: rotate(0deg);
      }
      to {
        transform: rotate(360deg);
      }
    }
  }
</style>
