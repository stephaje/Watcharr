<script lang="ts">
  import type { HotOrNot } from "@/types";
  import Icon from "../Icon.svelte";

  export let hotOrNot: HotOrNot | undefined = undefined;
  export let handleHotOrNot: (hotOrNot: HotOrNot) => void;

  function handleClickWrapper(s: HotOrNot) {
    if (s === hotOrNot) {
      handleHotOrNot("null");
      return;
    }
    handleHotOrNot(s);
  }
</script>

<div class="hotOrNot">
  <button
    class="not {hotOrNot && hotOrNot == 'hot' ? 'not-active' : ''}"
    on:click={(event) => {
      event.stopPropagation();
      handleClickWrapper("not");
    }}
    on:mouseleave={(event) => {
      event.currentTarget.blur();
    }}
  >
    <Icon i="not" />
  </button>
  <button
    class="hot {hotOrNot && hotOrNot == 'not' ? 'not-active' : ''}"
    on:click={(event) => {
      event.stopPropagation();
      handleClickWrapper("hot");
    }}
    on:mouseleave={(event) => {
      event.currentTarget.blur();
    }}
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
