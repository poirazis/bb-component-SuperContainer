<script>
  export let grabberPosition;
  export let state;
  export let resizing;
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
  class="grabber"
  class:resizing
  class:top={grabberPosition == "top"}
  class:right={grabberPosition == "right"}
  class:bottom={grabberPosition == "bottom"}
  class:left={grabberPosition == "left"}
  style:--pointer={grabberPosition == "left" || grabberPosition == "right"
    ? "col-resize"
    : "row-resize"}
  on:mousedown={state.startResizing}
/>

<style>
  .grabber {
    position: absolute;
    background-color: var(--spectrum-global-color-gray-200);
    cursor: var(--pointer);
    --grabber-thickness: 4px;
  }

  .resizing {
    background-color: var(--spectrum-global-color-blue-500);
  }

  .bottom {
    bottom: 0px;
    left: 0px;
    height: var(--grabber-thickness);
    width: 100%;
  }
  .right {
    top: 0px;
    right: 0px;
    width: var(--grabber-thickness);
    height: 100%;
    z-index: 2;
  }
</style>
