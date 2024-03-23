<script lang="ts">
  import Icon from "./Icon.svelte";
  import type { HotOrNot } from "../types";
  import tooltip from "./actions/tooltip";

  export let hotOrNot: HotOrNot | undefined;
  export let onChange: (newStatus: HotOrNot | undefined) => void;

  function handleHotOrNotClick(s: HotOrNot) {
    if (s === hotOrNot) {
      onChange("null");
      return;
    }
    onChange(s);
  }
</script>

<div class="hotOrNot">
  <button
    class="not {hotOrNot && hotOrNot == 'hot' ? 'not-active' : ''}"
    on:click={() => handleHotOrNotClick("not")}
    use:tooltip={{ text: "not", pos: "top" }}
  >
    <Icon i="not" />
  </button>
  <button
    class="hot {hotOrNot && hotOrNot == 'not' ? 'not-active' : ''}"
    on:click={() => handleHotOrNotClick("hot")}
    use:tooltip={{ text: "hot", pos: "top" }}
  >
    <Icon i="hot" />
  </button>
</div>

<style lang="scss">
  .hotOrNot {
    display: flex;
    flex-flow: row;
    gap: 20px;
    width: 100%;

    button {
      font-size: 10px;
      padding-inline: 10%;

      &.hot:hover,
      &.hot:not(.not-active) {
        background-color: firebrick;
      }

      &.not:hover,
      &.not:not(.not-active) {
        background-color: cornflowerblue;
      }
    }
  }
</style>
