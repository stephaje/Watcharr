<script lang="ts">
  import { onMount } from "svelte";

  export let type: "wrapped" | "vertical" = "wrapped";
  export let label: string = "";

  let ulEl: HTMLUListElement;

  onMount(() => {
    if (ulEl) {
      ulEl.classList.add(type);
    }
  });
</script>

<div class="column">
  <h2>{label}</h2>
  <div class="row">
    <ul bind:this={ulEl}>
      <slot />
    </ul>
  </div>
</div>

<style lang="scss">
  div.column {
    display: flex;
    justify-content: left;
    max-width: 1200px;
    flex-direction: column;
    margin-inline: 50px;
  }

  h2 {
    display: flex;
  }

  div.row {
    display: flex;
    justify-content: center;
  }

  ul {
    display: flex;
    flex-flow: row;
    justify-content: center;
    gap: 10px;
    list-style: none;
    flex-wrap: wrap;
    margin: 20px 10px;

    &:global(.vertical) {
      flex-wrap: nowrap;
      justify-content: unset;
      overflow-x: auto;
      padding: 15px 8px;
      margin: 5px 0;
    }
  }
</style>
