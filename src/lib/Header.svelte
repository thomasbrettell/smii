<script lang="ts">
  export let hideControls: boolean;
  export let randomise: () => void;
</script>

<header>
  <div>smii - create simple faces</div>
  <div class="controls">
    <label>
      <input bind:checked={hideControls} type="checkbox" />
      <span> hide controls </span>
    </label>
    <button>download</button>
    <button on:click={randomise}>randomise</button>
  </div>
</header>

<style>
  header {
    position: fixed;
    display: flex;
    top: 0;
    padding: 10px;
    justify-content: space-between;
    width: 100%;
    z-index: 2;
  }

  .controls {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  label {
    display: flex;
    align-items: center;
  }
</style>
