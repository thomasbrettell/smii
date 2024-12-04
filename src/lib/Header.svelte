<script lang="ts">
  export let hideControls: boolean;
</script>

<header>
  <div>smii</div>
  <div class="controls">
    <label>
      <input bind:checked={hideControls} type="checkbox" />
      <span> hide controls </span>
    </label>
    <button>download</button>
    <button>randomise</button>
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
