<script lang="ts">
  import type { Writable } from "svelte/store";

  export let point: Writable<number[]>;
  export let r: number;
  export let controlledPointRef: Writable<number[]> | null;
  export let mouseDown: boolean;
</script>

<circle
  cx={$point[0]}
  cy={$point[1]}
  {r}
  fill="red"
  class="anchor"
  on:mousedown={() => {
    controlledPointRef = point;
    mouseDown = true;
  }}
  role="button"
  tabindex={-1}
/>

<style>
  circle {
    cursor: pointer;
    outline: none;
  }
</style>
