<script lang="ts" context="module">
  export interface ContentLaodingConfig {
    notch: {
      size: '3x' | '2x' | '1x';
      color: string;
      class: any;
    };
    overlay: {
      opacity: number;
      color: string;
      class: string;
    };
  }
</script>

<script lang="ts">
  import Fa from 'svelte-fa/src/fa.svelte';
  import { faCircleNotch } from '@fortawesome/free-solid-svg-icons';

  const defaultConfig: ContentLaodingConfig = {
    notch: { size: '3x', color: 'red', class: '' },
    overlay: {
      color: 'black',
      opacity: 0.6,
      class: '',
    },
  };

  export let config: Readonly<ContentLaodingConfig> = defaultConfig;

  $: overlayOpacity = config?.overlay?.opacity || defaultConfig?.overlay?.opacity;
  $: overlayStyle = `background: ${config?.overlay?.color || defaultConfig?.overlay?.color}; 'z-index': 100; opacity: ${overlayOpacity}`;

  $: notchSize = config?.notch?.size || defaultConfig?.notch?.size;
  $: notchClass = config?.notch?.class || defaultConfig?.notch?.class;
  $: notchStyle = `color: ${config?.notch?.color || defaultConfig?.notch?.color}`;
</script>

<div class="position-absolute h-100 d-flex w-100" style={overlayStyle}>
  <Fa icon={faCircleNotch} size={notchSize} class="m-auto {notchClass}" style={notchStyle} spin />
</div>
